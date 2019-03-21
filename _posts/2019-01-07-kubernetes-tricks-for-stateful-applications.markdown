---
layout: post
title: "Kubernetes for stateful applications: Scaling macroservices"
date: 2019-01-07 11:10:21 +0100
comments: true
published: true
categories: 
  - k8s
  - sysadmin
---
I recently got to proctor an [Openhack](https://openhack.microsoft.com/) event on modern containerization. It ended up an excuse to dig deep on one of the corner cases that we all encounter, but no one likes to talk about.

[Kubernetes](https://kubernetes.io/) is one of the greatest **orchestration** and **scaling** tools ever built, designed for modern **decoupled**, **stateless** architectures. Kubernetes tutorials abound to show you these strong use cases. **But in the real world where you don't get to build "green field" every time, there are a lot of applications that don't fit that model**.

Lots of people out there are still writing tightly-coupled monoliths, in many cases for good reason. In some use cases microservices style scalability isn't even useful - you actually *prefer* stateful applications with tight coupling. For example a game server, where you don't want to scale player capacity per-game, you want to add more games (server instances). 

So today I'm writing about **stateful, non-scalable applications in kubernetes.** 

There are a few different approaches to coupling appliciation components:

Multi-container pods
---

Level 0 is to simply specify multiple components (containers) in your deployment.

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: php
        image: php:fpm
        ports:
        - containerPort: 9000
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```
This specifies 3 copies of the same application, with the same two containers in each replica. This is a coupled application, but it's still stateless. Let's add a volume - that's where we get into trouble.

The problem: If you add a Volume the normal way ([persistentVolumeClaim](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)), each of your replicas will try and connect to the same volume. It'll act like a network shared drive. Maybe that's OK for your application, but not if it's our super-stateful example! And depending on your volume class, the volume may reject multiple connections like ([Azure Disk](https://docs.microsoft.com/en-us/azure/aks/azure-disks-dynamic-pv) does, for example).

So how do we get around this limitation? I want a separate volume for each instance of the application. 

Kubernetes supports a different object type for this use case, called a [StatefulSet](https://kubernetes.io/docs/tutorials/stateful-application/basic-stateful-set/). This is exactly what it sounds like: a set of objects that define a stateful application. It's a template for creating multiple copies of *all resources* defined therein.

A statefulset will create replicas similar to a deployment, but it will set up separate Volumes and VolumeClaims for each one. The replicas will be identical except for an index number at the end of the labels. The first one might be called `nginx-deployment-0`, the second: `nginx-deployment-1`, and so on.  The result is a set of tightly coupled components, which can be individually addressed, and scaled using normal Kubernetes tools.

``` yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  serviceName: "nginx"
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
      - name: php
        image: php:fpm
        volumeMounts:
        - mountPath: "/var/www/html"
          name: data
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        storageClassName: default
        accessModes:
          - ReadWriteOnce
        resources:
          requests:
            storage: 5Gi
---
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
    - port: 80
      name: http
  clusterIP: None
  selector:
    app: nginx
```
There are a few details to notice here.

Yes, we've replaced *Deployment* with *StatefulSet*. You get a shiny gold star if you noticed that one.

The interesting part is the *VolumeClaimTemplates* section, below the containers definition. This keyword only exists inside a StatefulSet, and it's just what it sounds like: a template for creating Persistent Volume Claims.

If you apply this config, you'll see three PVs created, with three PVCs, attached to three Pods. You can apply HPA rules to scale these up and down just like you would with deployments.

There's also that weird Service at the bottom. A naked service with no clusterIP? What's the point? The point is as a helper for Kubernetes' internal DNS. All of those nice StatefulSet pods will come under a neat subdomain, eg nginx-0.nginx, nginx-1.nginx, etc. Additionally you can connect to active members of the StatefulSet by using that nginx domain component. A dns lookup on it will show all the IPs of the active members in the CNAME record.

"But what about external access?" I hear you cry. Yes, we've built a great stateful application that can scale instances, but it's only internally addressable! Good luck hosting those games...

External access and metacontroller
---

Normally you would put a LoadBalancer service in front of your application. But a Kubernetes load balancer will grab all of these StatefulSet members - so you can't address them externally one-by-one. What you *really* want to do, is create an external IP address for each statefulset member. 

One solution is to use a reverse proxy like nginx or HAProxy, configured to differentiate based on hostnames. But this is a blog post about Kubernetes, so we're going to do this the Kubernetes way!

Kubernetes is very extensible. If Pods, Services, etc don't make sense for your application or domain, you can define custom object types and behaviors, through [custom resources and controllers](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/). That's pretty edge case, but as we've seen, some kubernetes edge cases are mainstream cases in the real world. 

In our super-stateful application, we don't need a custom resource type. But we do want to attach [custom behaviors](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/#custom-controllers) to our StatefulSet: every time we start up a pod we should create a LoadBalancer for it. We should be nice and tear them down when the pods are scaled down, of course.

We'll use the [Metacontroller](https://github.com/GoogleCloudPlatform/metacontroller) add-on to make our lives easier. Metacontroller makes it "easy" to add custom behaviors. Just write a short script, stick it into a ConfigMap or FaaS, and let Metacontroller work its magic!

Metacontroller project comes with several well documented examples, including one that's very close to our requirement: [service-per-pod](https://github.com/GoogleCloudPlatform/metacontroller/tree/master/examples/service-per-pod).  

Step 1 is to install Metacontroller, of course:

``` bash
# Create 'metacontroller' namespace, service account, and role/binding.
kubectl apply -f https://raw.githubusercontent.com/GoogleCloudPlatform/metacontroller/master/manifests/metacontroller-rbac.yaml
# Create CRDs for Metacontroller APIs, and the Metacontroller StatefulSet.
kubectl apply -f https://raw.githubusercontent.com/GoogleCloudPlatform/metacontroller/master/manifests/metacontroller.yaml
```

Then we'll add some new metadata to our existing StatefulSet. The metacontroller script will use these values to configure the load balancers.

``` yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  annotations:
    service-per-pod-label: "pod-name"
    service-per-pod-ports: "80:80"
...
```

We also need to tell Kubernetes to decorate each StatefulSet with a pod-name label. We do this in the StatefulSet's pod template. 

``` yaml
...
spec:
  template:
    metadata:
      annotations:
        pod-name-label: "pod-name"
...
```

Note: this only works in k8s 1.9+ - if you're stuck with a lower version, you can script this action with Metacontroller, too. :). 

Now you're going to need two hooks. Put them in a directory together so they're easy to apply at once. These ones are written in jsonnet, but you could write this in whatever language you like.

The first hook actually creates the LoadBalancer for each Pod.

``` c#
function(request) {
  local statefulset = request.object,
  local labelKey = statefulset.metadata.annotations["service-per-pod-label"],
  local ports = statefulset.metadata.annotations["service-per-pod-ports"],

  // Create a service for each Pod, with a selector on the given label key.
  attachments: [
    {
      apiVersion: "v1",
      kind: "Service",
      metadata: {
        name: statefulset.metadata.name + "-" + index,
        labels: {app: "service-per-pod"}
      },
      spec: {
        type: "LoadBalancer",
        selector: {
          [labelKey]: statefulset.metadata.name + "-" + index
        },
        ports: [
          {
            local parts = std.split(portnums, ":"),
            port: std.parseInt(parts[0]),
            targetPort: std.parseInt(parts[1]),
          }
          for portnums in std.split(ports, ",")
        ]
      }
    }
    for index in std.range(0, statefulset.spec.replicas - 1)
  ]
}
```
The other hook is the "finalizer" - it responds to changes or deletions in pods by tearing down the corresponding LoadBalancers.

``` c# 
function(request) {
  // If the StatefulSet is updated to no longer match our decorator selector,
  // or if the StatefulSet is deleted, clean up any attachments we made.
  attachments: [],
  // Mark as finalized once we observe all Services are gone.
  finalized: std.length(request.attachments['Service.v1']) == 0
}
```

Add those into a subdirectory, and put them into a configmap together. Metacontroller will run them from there.

``` bash
kubectl create configmap service-per-pod-hooks -n metacontroller --from-file=hooks
```

Now apply the actual decorator controller which will run those functions. Note that you have to identify your hook jsonnet files by (file) name! Get the name wrong, and the finalizer will hang forever, [preventing you from deleting your statefulset](https://github.com/kubernetes/kubernetes/issues/72598). In my case, the files were called `create-lb-per-pod.jsonnet` and `finalizer.json`.

``` yaml
apiVersion: metacontroller.k8s.io/v1alpha1
kind: DecoratorController
metadata:
  name: service-per-pod
spec:
  resources:
  - apiVersion: apps/v1beta1
    resource: statefulsets
    annotationSelector:
      matchExpressions:
      - {key: service-per-pod-label, operator: Exists}
      - {key: service-per-pod-ports, operator: Exists}
  attachments:
  - apiVersion: v1
    resource: services
  hooks:
    sync:
      webhook:
        url: http://service-per-pod.metacontroller/create-lb-per-pod
    finalize:
      webhook:
        url: http://service-per-pod.metacontroller/finalizer
---
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: service-per-pod
  namespace: metacontroller
spec:
  replicas: 1
  selector:
    matchLabels:
      app: service-per-pod
  template:
    metadata:
      labels:
        app: service-per-pod
    spec:
      containers:
      - name: hooks
        image: metacontroller/jsonnetd:0.1
        imagePullPolicy: Always
        workingDir: /hooks
        volumeMounts:
        - name: hooks
          mountPath: /hooks
      volumes:
      - name: hooks
        configMap:
          name: service-per-pod-hooks
---
apiVersion: v1
kind: Service
metadata:
  name: service-per-pod
  namespace: metacontroller
spec:
  selector:
    app: service-per-pod
  ports:
  - port: 80
    targetPort: 8080

```

That's it! Now you can scale complete replicas of a very-stateful application with a simple `kubectl scale sts nginx --replicas=900`. 

Enjoy bragging to your friends about your "macroservices architecture", pushing the limits of Kubernetes to run and replicate a stateful monolith!

*Everyone hates writing YAML. Check out the [sample code for this post on Github](https://github.com/ohthehugemanatee/kubernetes-stateful-example)*

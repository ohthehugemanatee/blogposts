---
layout: post
title: "What Crell doesn't want you to know: how to automate letsencrypt on platform.sh"
date: 2017-02-21 22:33:08 +0100
comments: true
categories: 
 - hosting
 - sysadmin
 - letsencrypt
 - drupalplanet
---

If you believe the [docs](https://docs.platform.sh/development/going-live.html#prerequisites) and the [twitters](https://twitter.com/damz/status/672559665377501184), there is no way to automate [letsencrypt](https://letsencrypt.org/) certificates updates on [platform.sh](https://platform.sh/). You have to create the certificates manually, upload them manually, and maintain them manually.

But as readers of this blog know, the docs are only the start of the story. I've really enjoyed working with platform.sh with one of my private clients, and I couldn't believe that with all the flexibility - all the POWER - letsencrypt was really out of reach. I found a few attempts to script it, and one really great [snippet on gitlab](https://gitlab.com/snippets/27467). But no one had ever really synthesized this stuff into an easy howto. So here we go.

### 1) Add some writeable directories where platform.sh CLI and letsencrypt need them.

Normally when Platform deploys your application, it puts it all in a read-only filesystem. We're going to mount some special directories read-write so all the letsencrypt/platform magic can work.

Edit your application's `.platform.app.yaml` file, and find the `mounts:` section. At the bottom, add these three lines. Make sure to match the indents with everything else under the `mounts:` section!

```
    "/web/.well-known": "shared:files/.well-known"
    "/keys": "shared:files/keys"
    "/.platformsh": "shared:files/.platformsh"
```

Let's walk through each of these:

* /web/.well-known: In order to confirm that you actually control example.com, letsencrypt drops a file somewhere on your website, and then tries to fetch it. This directory is where it's going to do the drop and fetch. My webroot is `web`, you should change this to match your own environment. You might use `public` or `www` or something.
* /keys: You have to store your keyfiles SOMEWHERE. This is that place.
* /.platformsh: Your master environment needs a bit of configuration to be able to login to platform and update the certs on your account. This is where that will go.

### 2) Expose the .well-known directory to the Internet

I mentioned above that letsencrypt test your control over a domain by creating a file which it tries to fetch over the Internet. We already created the writeable directory where the scripts can drop the file, but platform.sh (wisely) defaults to hide your directories from the Internet. We're going to add some configuration to the "web" app section to expose this .well-known directory. Find the `web:` section of your `.platform.app.yaml` file, and the `locations:` section under that. At the bottom of that section, add this:

```
      '/.well-known':
            # Allow access to all files in the public files directory.
            allow: true
            expires: 5m
            passthru: false
            root: 'web/.well-known'
            # Do not execute PHP scripts.
            scripts: false
```

Make sure you match the indents of the other location entries! In my (default) `.platform.app.yaml` file, I have 8 spaces before that `'/.well-known':` line. Also note that the `root:` parameter there also uses my webroot directory, so adjust that to fit your environment.

### 3) Download the binaries you need during the application "build" phase

In order to do this, we're going to need to have the platform.sh CLI tool, and a let's encrypt CLI tool called lego. We'll download them during the "build" phase of your application. Still in the `platform.app.yaml` file, find the `hooks:` section, and the `build:` section under that. Add these steps to the bottom of the build:

```
      cd ~
      curl -sL https://github.com/xenolf/lego/releases/download/v0.3.1/lego_linux_amd64.tar.xz | tar -C .global/bin -xJ --strip-components=1 lego/lego
      curl -sfSL -o .global/bin/platform.phar https://github.com/platformsh/platformsh-cli/releases/download/v3.12.1/platform.phar
```

We're just downloading reasonably recent releases of our two tools. If anyone has a better way to get the latest release of either tool, please let me know. Otherwise we're stuck keeping this up to date manually.

### 4) Configure the platform.sh CLI

In order to configure the platform.sh CLI on your server, we have to deploy the changes from steps 1-3. Go ahead and do that now. I'll wait.

Now connect to your platform environment via SSH (`platform ssh -e master` for most of us). First we'll add a config file for platform. Edit a file in `.platformsh/config.yaml` with the editor of choice. You don't have to use vi, but it will win you some points with me. Here are the contents for that file:

```
updates:
    check: false
api:
    token_file: token
```

Pretty straightforward: this tells platform not to bother updating the CLI tool automatically (it can't - read-only filesystem, remember?). It then tells it to login using an API token, which it can find in the file `.platformsh/token`. Let's create that file next.

Log into the platform.sh web UI (you can launch it with `platform web` if you're feeling sassy), and navigate to your account settings > api tokens. That's at `https://accounts.platform.sh/user/12345/api-tokens` (with your own user ID of course). Add an API token, and copy its value into `.platformsh/token` on the environment we're working on. The token should be the only contents of that file.

Now let's test it by running `php /app/.global/bin/platform.phar auth:info`. If you see your account information, congratulations! You have a working platform.sh CLI installed.

### 5) Request your first certificate by hand

Still SSH'ed into that environment, let's see if everything works.

```
lego --email="support@example.com" --domains="www.example.com" --webroot=/app/public/ --path=/app/keys/ -a run
csplit -f /app/keys/certificates/www.example.com.crt- /app/keys/certificates/www.example.com.crt '/-----BEGIN CERTIFICATE-----/' '{1}' -z -s
php /app/.global/bin/platform.phar domain:update -p $PLATFORM_PROJECT --no-wait --yes --cert /app/keys/certificates/www.example.com.crt-00 --chain /app/keys/certificates/www.example.com.crt-01 --key /app/keys/certificates/www.example.com.key example.com
```

This is three commands: register the cert with letsencrypt, then split the resulting file into it's components, then register those components with platform.sh. If you didn't get any errors, go ahead and test your site - it's got a certificate! (yay)

### 6) Set up automatic renewals on cron

Back to `.platform.app.yaml`, look for the `crons:` section. If you're running drupal, you probably have a drupal cronjob in there already. Add this one at the bottom, matching indents as always.

```
    letsencrypt:
        spec: '0 0 1 * *'
        cmd: '/bin/sh /app/scripts/letsencrypt.sh'
```

Now let's create the script. Add the file `scripts/letsencrypt.sh` to your repo, with this content:

``` bash
#!/usr/bin/env bash

# Checks and updates the letsencrypt HTTPS cert.

set -e

if [ "$PLATFORM_ENVIRONMENT" = "master-7rqtwti" ]
  then
    # Renew the certificate
    lego --email="example@example.org" --domains="example.org" --webroot=/app/web/ --path=/app/keys/ -a renew
    # Split the certificate from any intermediate chain
    csplit -f /app/keys/certificates/example.org.crt- /app/keys/certificates/example.org.crt '/-----BEGIN CERTIFICATE-----/' '{1}' -z -s
    # Update the certificates on the domain
    php /app/.global/bin/platform.phar domain:update -p $PLATFORM_PROJECT --no-wait --yes --cert /app/keys/certificates/example.org.crt-00 --chain /app/keys/certificates/example.org.crt-01 --key /app/keys/certificates/example.org.key example.org
fi
```

Obviously you should replace all those `example.org`s and email addresses with your own domain. Make the file executable with `chmod u+x scripts/letsencrypt.sh`, commit it, and push it up to your platform.sh environment.

### 7) Send a bragging email to Crell

Technically this isn't supposed to be possible, but YOU DID IT! Make sure to rub it in.

{% img center /images/larry-garfield.jpg "Larry is waiting to hear from you. (photo credit Jesus Manuel Olivas)" %}

Good luck!

PS - I'm just gonna link one more time to the guy whose snippet made this all possible: [Ariel Barreiro](https://www.drupal.org/u/hanoii) did the hardest part of this. I'm grateful that he made his notes public!

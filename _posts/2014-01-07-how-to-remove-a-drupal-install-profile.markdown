---
layout: post
title: "How to remove a Drupal install profile"
date: 2014-01-07 13:23:45 +0100
comments: true
categories: 
 - Drupal
 - howto
 - drupalplanet
---

[Install profiles](https://drupal.org/project/install_profile_api) are a great way to throw together a functional Drupal site really quickly. In Drupal 6, an Install Profile was just a blueprint for setting up a site really quickly. What you did after the site was installed was your own business! But in Drupal 7 profiles are much more integrated with core. The assumption is that when you use an install profile, you want to rely on the profile's maintainer for all your updates. This is not always the case.

Very often your site will diverge from the install profile as it takes on a life of its own, and it will be useful to convert it to "vanilla" Drupal. Today I'll use a relatively simple example of a musician site which is moving away from the [Pushtape](https://drupal.org/project/pushtape) distribution. Later I'll return to this subject with the much more in-depth example of moving a community site away from [Drupal Commons](https://drupal.org/project/commons).

Move things around
-----------------

Install profiles have all their files stored in the site root's *profiles/* directory. The first step is going to be moving everything out of there. In the case of pushtape, we have libraries, modules, and a theme stored in there. We're going to move them to a more normal location.

``` console 
# mkdir sites/all/libraries
# mv profiles/pushtape/libraries/* sites/all/libraries

# mkdir sites/all/modules/custom
# mv profiles/pushtape/modules/pushtape_* sites/all/modules/custom
# mv profiles/pushtape/modules/* sites/all/modules

# mkdir sites/all/themes
# mv profiles/pushtape/themes/* sites/all/themes
```

Next we need to see if there are any variables set in the install profile which really depend on the profile directory. If there are, we'll have to set them again with the new path.

``` console
# cd profiles/pushtape
# grep 'profiles/pushtape' * -R
pushtape.install:  variable_set('sm2_path', 'profiles/pushtape/libraries/soundmanager2');
```

In this case, we see one variable_set which tells the system where to find the soundmanager2 library. We can update that easily enough with drush:

``` console
# drush vset sm2_path 'sites/all/libraries/soundmanager2'
```

Now we have to update Drupal's setting for which install profile was used to create the site.

``` console
# drush vset install_profile standard
```

In some cases this will be enough to work. Personally I like to keep my modules folder more organized, so I go the extra mile:

``` console
# cd sites/all/modules
# mkdir contrib
# mv !(custom|contrib) contrib
```

I also separated out the custom code from the features. You can figure out which custom modules implement features with *find . |grep features*, and move them into a separate directory manually.

Clearing caches
---------------

Once you're done moving things around, CLEAR CACHES. Drupal keeps an index of module, library, and theme directories, and you just broke it.

```
drush cc all
```

The only problem is, in many cases you will have moved a module that is required for drupal bootstrap. So you'll have to get the handy drush tool [Registry Rebuild](https://drupal.org/project/registry_rebuild), and run that before your cache clear:

``` console
# drush dl registry_rebuild
# drush rr
# drush cc all
```

Extra Cleanup
-------------

As commenter @ericaitala notes, you may need some followup cleanup to really get all traces out. The easiest way to do this is from the SQL command line, which you can access via drush:

```
drush sqlq "DELETE FROM `system` WHERE filename LIKE 'profiles/profilename/profilename.profile"
drush sqlq "UPDATE `system` SET status=1 WHERE filename LIKE 'profiles/standard/standard.profile'"
```

Technically these should both be covered by the registry_rebuild operation, but we're doing it by hand because it seems to be missed in some operations. The first command removes the entry for the profile from Drupal's system table - it removes any knowledge Drupal has that there was an install profile there. The second tells Drupal that the "standard" install profile is active, and should be checked for updates. 

That's it - your site is now officially a vanilla Drupal install. Test by removing the profiles/pushtape directory, clearing caches, and browsing around your site.


*NOTE: With a more complex install profile I expect to encounter more difficulty. Stay tuned for the post on extricating yourself from Commons later this year!*

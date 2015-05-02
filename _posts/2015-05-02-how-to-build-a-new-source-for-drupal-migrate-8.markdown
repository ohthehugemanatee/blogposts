---
layout: post
title: "How to build a new source for Drupal Migrate 8"
date: 2015-05-02 10:10:36 -0400
comments: true
categories:
  - drupal
  - drupalplanet
  - drupal 8
  - howto
---
This week I wanted to accomplish a task in Drupal 8 that would be simple in Drupal 7: Import several CSV files, each one related to the others by taxonomy terms. Most importantly, I wanted to do it with [Migrate module](https://drupal.org/project/migrate).

Migrate in Drupal 7 is a fantastic piece of code. It is not designed to be used from the GUI, rather, it provides a framework of "source", "destination", and "migration" classes so that even the most convoluted migration is 90% written for you. To create a migration in Drupal 7, you create a custom module, declare your migrations in a hook_info, and then extend the built in "migration" class. You instantiate one of the given classes for the source material (is it a CSV? JSON? Direct connection to a custom DB?), then one of the classes for the destination (is it a content type? Taxonomy term?). Then you add one simple line of code mapping each field from source to destination. If you know what you're doing, the task I had in mind shouldn't take more than 15 minutes per source.

It's not quite so easy in Drupal 8. First of all, with Migrate in core, we had to greatly simplify the goals for the module. The version of Migrate that is really functional and stable is specifically and _only_ the basic framework. There is a separate migrate_drupal module to provide everything you need for migrating from Drupal 6 or 7. This has been a laser-tight focus on just the essentials, which means there's no UI, very little drush support, and definitely no nice extras like the ability to read non-Drupal sources.

My task this week became to write the first CSV source for Drupal 8 Migrate.

Drupal 8 Migrate Overview
===

Drupal 8 Migrations, when you're using classes that already exist, are actually even easier than Migrate 7. All you do is write a single YAML file for each kind of data you're transferring, and put it in a custom module's _config/install_ directory. Your YAML file gives your migration a name and a group, tells us what the source is for data, maps source fields to destination fields, and tells us what the destination objects are. Here's an example Migration definition file from core. See if you can understand what's being migrated here.

``` yaml
id: d6_system_site
label: Drupal 6 site configuration
migration_groups:
  - Drupal 6
source:
  plugin: variable
  variables:
    - site_name
    - site_mail
    - site_slogan
    - site_frontpage
    - site_403
    - site_404
    - drupal_weight_select_max
    - admin_compact_mode
process:
  name: site_name
  mail: site_mail
  slogan: site_slogan
  'page/front': site_frontpage
  'page/403': site_403
  'page/404': site_404
  weight_select_max: drupal_weight_select_max
  admin_compact_mode: admin_compact_mode
destination:
  plugin: config
  config_name: system.site
```

You probably figured it out: this migration takes the system settings (variables) from a Drupal 6 site, and puts them into the Drupal 7 configuration. Not terribly hard, right? You can even do data transformations from the source field value to the destination.

Unfortunately, the only sources we have so far are for Drupal 6 and 7 sites, pulling directly from the database. If you want to use Migrate 8 the way we used Migrate 7, as an easy way to pull in data from arbitrary sources, you'll have to contribute.


Writing a source plugin in Migrate_plus
===

Enter [Migrate Plus module](https://www.drupal.org/sandbox/mikeryan/migrate_plus). This is the place in contrib, where we can fill out all the rest of the behavior we want from Migrate, that's not necessarily a core requirement. This is where we'll be writing our source plugin.

To add a source plugin, just create a .php file in migrate_plus/src/Plugins/migrate/source . Drupal will discover the new plugin automatically the next time you rebuild the cache. The filename has to be the same as the name of the class, so choose carefully! My file is called CSV.php . Here's the top of the file you need for a basic :

```php
<?php
/**
 * @file
 * Contains \Drupal\migrate_plus\Plugin\migrate\source\csv.
 */

namespace Drupal\migrate_plus\Plugin\migrate\source;

use Drupal\migrate\Plugin\migrate\source\SourcePluginBase;

/**
 * Source for CSV files.
 *
 * @MigrateSource(
 *   id = "csv"
 * )
 */
class CSV extends SourcePluginBase {
```

I'm calling this out separately because for newbies to Drupal 8, this is the hard part. This is all the information that Drupal needs to be able to find your class when it needs it. The @file comment is important. That and the namespace below have to match the actual location of the .php file.

Then you declare any other classes that you need, with their full namespace. To start with all you need is SourcePluginBase.

Finally you have to annotate the class with that @MigrateSource(id="csv"). This is how Migrate module knows that this is a MigrateSource, and the name of your Plugin. Don't miss it!

Inside the class, you must have the following methods. I'll explain a bit more about each afterwards.

* initializeIterator() : Should return a valid Iterator object.
* getIds() : Should return an array that defines the unique identifiers of your data source.
* __toString() : Should return a simple, string representation of the source.
* fields() : Should return a definitive list of fields in the source.
* __construct() : You don't NEED this method, but you probably will end up using it.

initializeIterator()
---

An Iterator is a complicated sounding word for an Object that contains everything you need to read from a data source, and go through it one line at a time. Maybe you're used to fopen('path/to/file', 'r') to open a file, and then you write code for every possible operation with that file. An iterator takes care of all that for you. In the case of most file-based sources, you can just use the SplFileObject class that comes with PHP.

Any arguments that were passed in the source: section of the YAML file will be available under $this->configuration. So if my YAML looks like this:

```yaml
source:
  plugin: csv
  path: '/vagrant/import/ACS_13_1YR_B28002_with_ann.csv'
```

My initializeIterator( ) method can look like this:

```php

public function initializeIterator() {
  // File handler using our custom header-rows-respecting extension of SPLFileObject.
  $file = new SplFileObject($this->configuration['path']);
  $file->setFlags(SplFileObject::READ_CSV);
  return $file;
}
```

Not too complicated, right? This method is called right at the beginning of the migration, the first time Migrate wants to get any information out of your source. The iterator will be stored in $this->iterator.


getIds()
---

This method should return an array of all the unique keys for your source. A unique key is some value that's unique for that row in the source material. Sometimes there's more than one, which is why this is an array. Each key field name is also an array, with a child "type" declaration. This is hard to explain in English, but easy to show in code:

```php
public function getIDs() {
  $ids = array();
  foreach ($this->configuration['keys'] as $key) {
    $ids[$key]['type'] = 'string';
  }
  return $ids;
}
```

We rely on the YAML author to tell us the key fields in the CSV, and we just reformat them accordingly. Type can be 'string', 'float', 'integer', whatever makes sense.

__toString()
---

This method has to return a simple string explanation of the source query. In the case of a file-based source, it makes sense to print the path to the file, like this:

```php
public function __toString() {
  return (string) $this->query;
}
```

fields()
---

This method returns an array of available fields on the source. The keys should be the machine names, the values are descriptive, human-readable names. In the case of the CSV source, we look for headers at the top of the CSV file and build the array that way.

__construct()
---
The constructor method is called whenever a class is instantiated. You don't technically HAVE to have a constructor on your source class, but you'll find it helpful. For the CSV source, I used the constructor to make sure we have all the configuration that we need. Then I try and set sane values for fields, based on any header in the file.

```php
public function __construct(array $configuration, $plugin_id, $plugin_definition, MigrationInterface $migration) {
  parent::__construct($configuration, $plugin_id, $plugin_definition, $migration);

  // Path is required.
  if (empty($this->configuration['path'])) {
    return new MigrateException('You must declare the "path" to the source CSV file in your source settings.');
  }

  // Key field(s) are required
  if (empty($this->configuration['keys'])) {
    return new MigrateException('You must declare the "keys" the source CSV file in your source settings.');
  }

  // Set header rows from the migrate configuration.
  $this->headerRows = !empty($this->configuration['header_rows']) ? $this->configuration['header_rows'] : 0;

  // Figure out what CSV columns we have.
  // One can either pass in an explicit list of column names to use, or if we have
  // a header row we can use the names from that
  if ($this->headerRows && empty($this->configuration['csvColumns'])) {
    $this->csvColumns = array();

    // Skip all but the last header
    for ($i = 0; $i < $this->headerRows - 1; $i++) {
      $this->getNextLine();
    }

    $row = $this->getNextLine();
    foreach ($row as $key => $header) {
      $header = trim($header);
      $this->getIterator()->csvColumns[] = array($header, $header);
    }
  }
  elseif ($this->configuration['csvColumns']) {
    $this->getIterator()->csvColumns = $this->configuration['csvColumns'];
  }
}
```

Profit!
---

That's it! Four simple methods, and you have a new source type for Drupal 8 Migrate. Of course, you will probably find complications that require a bit more work. For CSV, supporting a header row turned out to be a real pain. Any time Migrate tried to "rewind" the source back to the first line, it would try and migrate the header row! I ended up extending SplFileObject just to handle this issue.

Here's the YAML file I used to test this, importing a list of states from some US Census data.

```yaml
id: states
label: States
migration_groups:
  - US Census

source:
  plugin: csv
  path: '/vagrant/import/ACS_13_1YR_B28002_with_ann.csv'
  header_rows: 2
  fields:
    - Id2
    - Geography
  keys:
    - Id2

process:
  name: Geography
  vid:
    -
      plugin: default_value
      default_value: state

destination:
  plugin: entity:taxonomy_term
```

You can see my CSV source and Iterator in the [issue queue for migrate_plus](https://www.drupal.org/node/2458003). Good luck, and happy migrating!


Thanks
---

I learned a lot this week. Big thanks to the [Migrate Documentation](https://www.drupal.org/node/2127611), but especially to [chx](https://www.drupal.org/u/chx), [mikeryan](https://www.drupal.org/u/mikeryan), and the other good folks in #drupal-migrate who helped set me straight.

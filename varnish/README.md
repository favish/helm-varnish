# Varnish

This chart is intended to exist in front of the Drupal chart maintained by Favish.

It will create two varnish containers in a stateful set with a headless service to access them.  The service is intended
to be used by modules or in-cluster services to clear specific parts of the cache during operation.

# Varnish Purgers

Without a purger, Varnish is configured to hold on to cached items for their entire TTL value.

Normal setups should have a rather long TTL in Drupal so that cache is warm for as long as possible, but that means
your users cannot update anything for several hours.

Enter purgers. There are a series of Drupal modules that will send BAN requests to Varnish immediately after
edits are made so that your users can make important updates quickly.

## Setting up

1. Run `composer require drupal/purge drupal/varnish_purge`. Purge provides several additional sub-modules.
1. Install them with drush: `drush en purge purge_drush purge_ui purge_processor_lateruntime purge_queuer_coretags varnish_purger varnish_purge_tags`
1. Follow the documentation [here](https://lagoon.readthedocs.io/en/latest/using_lagoon/drupal/services/varnish/#install-purge-and-varnish-purge-modules) 
(just the **Configure Varnish Purge** section), 
but change the Name to **varnish-0**. Repeat creating a purger with identical settings but name it **varnish-1**. The hostname is not important here.
1. Run `drush cex` and then navigate to your config directory. Note the hash after your new purgers - it will be in the filename and look similar to
`varnish_purger.settings.104fbb7449`. Create two lines in your settings.php with corresponding hashes, like this:
```php
$config['varnish_purger.settings.104fbb7449']['hostname'] = 'varnish-0.' . getenv("VARNISH_STATEFULSET_DOMAIN");
$config['varnish_purger.settings.9c32454b45']['hostname'] = 'varnish-1.' . getenv("VARNISH_STATEFULSET_DOMAIN");
```
This uses [the stateful set domain](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#stable-network-id) to set the 
hostname to match the DNS record that corresponds to both of your varnish services, so your Drupal php pods can make requests to them.

That's it! Now the `Late runtime processor` will trigger BAN requests to each of your varnish heads when Drupal tags are invalidated
in a request. For example, when you submit a node form, that invalidates your node's related cache tags and Varnish will be notified!

### Further Reading

[Original blog post introducing varnish_purge](https://digitalist-tech.se/blogg/purge-cachetags-varnish)
[The lagoon docs](https://lagoon.readthedocs.io/en/latest/using_lagoon/drupal/services/varnish/#install-purge-and-varnish-purge-modules)

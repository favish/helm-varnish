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
2. Install them with drush: `drush en purge purge_drush purge_ui purge_processor_lateruntime purge_queuer_coretags varnish_purger varnish_purge_tags`
3. Follow the documentation [here](https://dev.docs.agile.coop/processes/varnish-purge/) 
(just the **Configure Varnish Purge** section), 
but change the Name to **varnish-0**. Repeat creating a purger with identical settings but name it **varnish-1**. The hostname is not important here.
   1. In addition to the `Cache-Tag` header, you need to add a `X-Varnish-Purge` header the purger request headers with the same value as `.Values.secret`. The purpose of this is to only authorize BAN requests containing the right credentials.
4. Run `drush cex` and then navigate to your config directory. Note the hash after your new purgers - it will be in the filename and look similar to
`varnish_purger.settings.104fbb7449`. Create two lines in your settings.php with corresponding hashes, like this:
```php
$config['varnish_purger.settings.104fbb7449']['hostname'] = 'varnish-0.' . getenv("VARNISH_STATEFULSET_DOMAIN");
$config['varnish_purger.settings.9c32454b45']['hostname'] = 'varnish-1.' . getenv("VARNISH_STATEFULSET_DOMAIN");
```
5. Test out caching and invalidation:
   1. Use curl to make a request and notice the hits and misses and from which varnish pod they originate:
    ```
    curl -i 'http://kitco-cms-drupal.local.favish.com/graphql' \
      -H 'Content-Type: application/json' \
      --data-raw '{"query":"query {\n  nodeByUrlAlias(urlAlias: \"/news/article/2023-04-21/gold-prices-set-soar-us-deficit-widens-felder-report\") {\n    title\n  }\n}","variables":null}' \
      --compressed \
      --insecure
    ```

   2. Make a BAN request with for the node in question:

    ```
    curl -i http://<hostname> -X BAN -H 'Cache-Tags: node:<nid>' -H 'X-Varnish-Purge: <secret>'`
    ```
    3. In each of the Varnish pods, run `varnishlog` and notice the BAN request and subsequent MISS and HIT requests. This can also be helpful to see the `Cache-Tag` and `X-Varnish-Purge` headers and ensure everything is set like you expect.
       1. Use this version of the command to pare down the output. But it's just one example of many options:
       ```
       varnishlog -i VCL_call,ReqMethod,BereqMethod,ReqURL,BereqURL,VCL_Log
       ```

This uses [the stateful set domain](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#stable-network-id) to set the 
hostname to match the DNS record that corresponds to both of your varnish services, so your Drupal php pods can make requests to them.

That's it! Now the `Late runtime processor` will trigger BAN requests to each of your varnish heads when Drupal tags are invalidated
in a request. For example, when you submit a node form, that invalidates your node's related cache tags and Varnish will be notified!

### Further Reading

[Configuring Varnish_Purge](https://dev.docs.agile.coop/processes/varnish-purge/)
[Caching POST requests](https://docs.varnish-software.com/tutorials/caching-post-requests)

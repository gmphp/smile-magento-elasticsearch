Behavorial Search Features
==========================

The module comes with mechanism which allows the engine to analyze behavior of the users of the website to modify search results scoring.

This section explains how to configure this feature and extend the available models.

Data collector
--------------

### Prepare ElasticSearch

In order to make the feature working, you will need to install the ES tracking indexer shipped with the module (es/plugins/tracking-indexer/tracking-indexer-current.jar).

The module is normally installed with ES if you have used the automated install script (see [Installing the module](install.md)). 

You can check the module is correctly installed by running the following command from the shell :

```bash
/usr/share/elasticsearch/bin/plugin --list
```

If not installed, you can rerun the install script after you have check you have the last version of the module.
You can also run the following command to install the plugin from the source :

```bash
/usr/share/elasticsearch/bin/plugin -i tracking-indexer -u file:///SOURCE_ROOT/es/tracking-indexer/tracking-indexer-current.jar
```

When installed the http://localhost:9200/tracker/hit URL of the search engine should respond with an empty PNG image and the search engine is ready to index events sent through this URL.

> **Note :** 
> When using SPBuilder, make sure the es directory of your project is part of your delivery package. It is not the case bu default, but you can add the path to the svn-components (More details at https://wiki.smile.fr/view/Dirtech/Projets/SpBuilderProperties)

### Apache configuration

You will need a new domain name to collect tracking.

For a site named **www.mysite.com**, you can use a new domain called **t.mysite.com** or **hit.mysite.com** by example.
 
This domain will be proxied to ES tracking plugin through an Apache vhost :

``` conf
<VirtualHost *:80>

    ServerName t.mysite.com

    ProxyPreserveHost On
    <Proxy balancer://esnodes>
      BalancerMember http://localhost:9200/tracker/hit
      # Place all your ES nodes as balancer member
      # If you have 2 nodes (es.node1.mywebsite.com and es.node2.mywebsite.com),
      # you will have the following configuration :
      #
      # BalancerMember http://es.node1.mywebsite.com:9200/tracker/hit
      # BalancerMember http://es.node1.mywebsite.com:9200/tracker/hit
    </Proxy>

    RewriteEngine On
    RewriteRule (.*) balancer://esnodes [P]
    
    ErrorLog  /var/log/apache2/smile-tracker.log
    CustomLog /var/log/apache2/smile-tracker.log combined
    
</VirtualHost>
```

Before restarting apache, ensure the required Apache modules are correctly loaded :

```bash
a2enmode proxy proxy_http proxy_balancer headers
```

> **Note :**
> * If using Varnish, you have to exclude the hit domain from the cache.
> * If using SSL on your website, you will need **to duplicate this configuration on the SSL port (443)** in order your website respond to https://t.mysite.com correctly. **You will need a valid certificate for this domain.**
> * Use the same domain name for SSL and non-SSL (a limitation into the tracking module does not allow different domain name).


### Smile Tracker



Optimizer models
----------------
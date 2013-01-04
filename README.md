ops-platform-setup
==================
This repository is designed to collect all information needed to set up the Open Phacts platform. The aim is to be able to pull directly pull from github and in one go setup a running version of Open Phacts. Hopefully, this will include not only the core cache but also the other required services Something all the lines of how [Heroku](https://devcenter.heroku.com/articles/git ) leverages git. In the meantime, using github will allow us to keep our documentation reasonably up-to-date as we develop.

Core Platform TODOs
-------------------
In JIRA at <https://openphacts2011.atlassian.net/secure/IssueNavigator.jspa?mode=hide&requestId=10701>

Code
--------------

Source code needed

- <https://larkc.svn.sourceforge.net/svnroot/larkc/trunk>
- <https://github.com/openphacts/OpsPlatform>
- <https://github.com/openphacts/coreGUI>
- <https://github.com/openphacts/OPS_LinkedDataApi>


Software
----------------
- Tomcat 6
- Apache 2
- [Sesame 2.6.8](http://www.openrdf.org/)

Data
---------------
- Data loading script is in scripts/seasame-load-list.txt
- Run using console.sh < sesame-load-list.txt
- Each dataset is loaded into a separate named graph 

#### Hierarchies
For each hierarchy we have three named graphs

1. the direct hierarchy - the named graph is denoted by /direct
2. the full closure - the named graph is denoted by /inference
3. other info (e.g. rdfs:label) - the named graph without a suffix

Environment Setup on ops.few.vu.nl (Debian 6.0)
-----------------

We use screen

Directory setup

    /var/www                  #lda is here
    /var/www/api-config-files # sparql files for lda located here
    ~/
    ~/log/                    # logs are stored here
    ~/production/             # contains OpsPlatform code for production
    ~/develop/                # contains OpsPlatform code for develop
    ~/coreGUI/                # contains core GUI code for running the explorer

- One RDF store (sesame) running on 9090
    - http://ops.few.vu.nl:9090/openrdf-workbench/
- data is located at ~/develop/openphacts/datasets
    - <https://github.com/openphacts/ops-platform-setup/tree/master/data-sources>
- We are currently running all data off one endpoint

#### Building Base Platform
    OPS@ops:~/production/openphacts/ops-platform/scripts$ source  ExportOPSVariables.sh
    OPS@ops:~/production/openphacts/ops-platform/scripts$ source BuildLarKC.sh 
    OPS@ops:~/develop/openphacts/ops-platform/scripts$ source  ExportOPSVariables.sh
    OPS@ops:~/develop/openphacts/ops-platform/scripts$ source BuildLarKC.sh 

#### Core API LarKC instance
     
     OPS@ops:~$ screen
     OPS@ops:~/production/openphacts/ops-platform/scripts$ source RunLarKC.sh &> ~/log/production.log
     # exit out of screen
     # and launch the workflow
     OPS@ops:~/production/openphacts/ops-platform/scripts$ source  ExportOPSVariables.sh
     OPS@ops:~/production/openphacts/ops-platform/scripts$ source LaunchOPSAPIWorkflow.sh

Note that ~/production/larkc-endpoints/endpoint.opsapi/src/main/java/eu/larkc/endpoint/opsapi/sparql/ contains the sparql queries being run by the core api. These are before expansion.


#### Linked Data API LarKC instance

    OPS@ops:~$ screen
    OPS@ops:~/develop/openphacts/ops-platform/scripts$ source RunLarKC.sh &> ~/log/develop.log
    # exit out of screen
    # and launch the workflow
    OPS@ops:~/develop/openphacts/ops-platform/scripts$ source  ExportOPSVariables.sh
    OPS@ops:~/develop/openphacts/ops-platform/scripts$ source Launch_LDA_Workflow.sh

###### LDA queries
- all queries are in api-config-files
- the difference between github and server is the port of where the lark endpoint is located
- sparqlEndpoint  api:sparqlEndpoint <http://localhost:9183/sparql/> ;
- to update the lda queries you just need to modify the queries
- can test by checking out and using the ops machine
- by convention, in the lda in sparql variables names ?ops_item are replaced by the uri of post variable
- ops:input tells the expander what to expand on 

#### Launch GUI
  
     OPS@ops:~/coreGUI$ screen 
     rails s -e production  &> ~/log/gui.log

### Platform set-up on ops2 (CentOS 6.2), 10 Aug 2012

#### Load data into Virtuoso
    
#### Set up LDA

    [antonis@ops2 ~]$ sudo yum install php
    [antonis@ops2 ~]$ sudo yum install php-xml
    [antonis@ops2 ~]$ sudo chmod -R go+rw /var/www
    [antonis@ops2 ~]$ cd /var/www/
    [antonis@ops2 www]$ rmdir html
    [antonis@ops2 www]$ git clone https://github.com/openphacts/OPS_LinkedDataApi.git html

    Edit: /etc/httpd/conf/httpd.conf
    You need .htaccess files enabled and allowed on DOCUMENTROOT

    AccessFileName .htaccess

    <Directory "/var/www/html">
        Options Indexes FollowSymLinks
        AllowOverride All
        Order allow,deny
        Allow from all
    </Directory>

    [antonis@ops2 ~]$ service httpd start

#### Steps to enable caching from RAM instead of disk
1. Edit deployment.settings.php from the root of LDA and change:

  define('PUELIA_SERVE_FROM_CACHE', true);

to

  define('PUELIA_SERVE_FROM_CACHE', true);

2. yum install php-pecl-memcache

3. yum install memcached

4. Add to php.ini:

  extension=memcache.so

5. Add to/Create /etc/httpd/conf.d/memcached.conf 

  <IfModule mod_memcached_cache.c>
        CacheEnable memcached /
        MemcachedCacheServer localhost:11211
        MemcachedMaxServers 50
        MemcachedMinConnections 10
        MemcachedSMaxConnections 100
        MemcachedMaxConnections 100
        MemcachedConnectionTTL 10
        MemcachedCacheMinFileSize 1
        MemcachedCacheMaxFileSize 20971520
        CacheDisable /admin/
  </IfModule>

6. Make sure the request Header does not contain the option 'cache-control=no-cache'

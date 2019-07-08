# Sugar Kubernetes 
This repository will help you deploy a Kubernetes based local enviornment. We'll be using `minikube` to simulate a Kubernetes cluser on local.

# Getting started
Start `minikube`:
`minikube start --memory 8192`

### Types of stacks
There are mainly two types of stack:
* Single Apache web server - Initial development
* An Apache load balancer with a cluster of two Apache web servers behind it - A more real-life environment to verify that everything works correctly

### All stack components
There are multiple stack components as docker containers, that perform different duties. Not all the stack components might be used on a specific stack setup.
* Apache load balancer - Load balances requests between the cluster of Apache PHP web servers, round robin
* Apache PHP web server - Web server(s)
* MySQL database - Database
* Elasticsearch - Sugar search engine
* Redis - Two purposes: Sugar object caching service and PHP Session storage/sharing service
* Cron - Sugar background scheduler processing. Note that this is enabled immediately and it will run `cron.php` as soon as the file is available, and it will attempt to do so every 60 seconds since its last run. This container is used for any other CLI execution required during development
* Permission - Sets Sugar instance permissions correctly and then terminates
* LDAP - LDAP testing server if needed with authentication

## Get the system up and running
* The first step for everything to work smoothly, is to add on your computer's host file /etc/hosts the entry "docker.local" to point to your machine's ip (it might be 127.0.0.1 if running the stack locally or within the VM running Docker)
* Clone the repository with `git clone https://github.com/esimonetti/SugarDockerized.git sugardocker` and enter sugardocker with `cd sugardocker`
* Run the utility `./utilities/setownership.sh` to set the correct ownership of the data directory
* Select the stack combination to run by choosing the correct yml file within the subdirectories inside [stacks](stacks/). See next step for more details and an example.
* Run `docker-compose -f <stack yml filename> up -d` for the selected <stack yml filename>. As an example if we selected `stacks/sugar8/php71.yml`, you would run `docker-compose -f stacks/sugar8/php71.yml up -d`

## Current version support
The main stacks work with [Sugar version 9.0 and all its platform requirements](http://support.sugarcrm.com/Resources/Supported_Platforms/Sugar_9.0.x_Supported_Platforms/). Additional stacks are aligned with the platform requirements of version [8.0](http://support.sugarcrm.com/Resources/Supported_Platforms/Sugar_8.0.x_Supported_Platforms/), [7.9](http://support.sugarcrm.com/Resources/Supported_Platforms/Sugar_7.9.x_Supported_Platforms/) and the Sugar Cloud only versions: 7.10/7.11, 8.1, 8.2 and 8.3.

## Starting and stopping the desired stack
* Run the stack with `docker-compose -f <stack yml filename> up -d`
* Stop the stack with `docker-compose -f <stack yml filename> down`

To facilitate the task of starting and stopping stacks and to switch between them, is available [this utility script](https://github.com/esimonetti/SugarDockerized#stacksh).

## System's details

### Stack components hostnames
* Apache load balancer: sugar-lb
* Apache PHP web server: On single stack: sugar-web1 On cluster stack: sugar-web1 and sugar-web2
* MySQL database: sugar-mysql
* Elasticsearch: sugar-elasticsearch
* Redis - sugar-redis
* Cron - sugar-cron
* Permission - sugar-permissions
* LDAP - sugar-ldap

To verify all components hostnames just run `docker ps` when the stack is up and running.

Please note that on this setup, only the web server or the load balancer (if in single web server or cluster stack) and the database can be accessed externally. Everything else is only allowed within the stack components.

### Core stack components
* Linux
* Apache
* MySQL
* PHP
* Redis
* Elasticsearch

### Sugar Setup details
* Browser url: http://docker.local/sugar/ - Based on the host file entry defined above on the local machine
* MySQL hostname: sugar-mysql
    * MySQL user: root
    * MySQL password: root
* Elasticsearch hostname: sugar-elasticsearch
* Redis hostname: sugar-redis

### Apache additional information
Apache web servers have enabled:
* mod_headers
* mod_expires
* mod_deflate
* mod_rewrite

### PHP additional information
Apache web servers have PHP with enabled:
* Zend OPcache - Configured for Sugar with the assumption the files will be located within the correct path
* XHProf and Tideways profilers
* Xdebug is installed but it is not enabled by default (due to its performance impact). If there is the need to enable it, you would have to uncomment the configuration option on the PHP Dockerfile of choice, and leverage the stack configuration with local build.
    * If you use an IDE such as PHPStorm, you can setup DBGp Proxy under the menus Preference -> Language & Framework -> PHP -> Debug -> DBGp Proxy. Example settings are available in the screenshot below:

      <img width="1026" alt="PHPStorm xdebug settings" src="https://user-images.githubusercontent.com/361254/38972661-d48661f6-4356-11e8-9245-ad598239fe94.png">

        * Debug with Xdebug Helper

          If you use Chrome as a browser, you can install the extension [Xdebug helper](https://chrome.google.com/webstore/detail/xdebug-helper/eadndfjplgieldjbigjakmdgkmoaaaoc). When ready to debug, click the debug button on the Xdebug helper, and click on "Start listening for PHP Debug Connections" within PHPStorm

          <img width="146" alt="xdebughelper" src="https://user-images.githubusercontent.com/361254/43093912-5a7a3bf2-8e66-11e8-9c11-811316d8f2ee.png">
          <img  width="50" alt="Start listening for PHP Debug Connections" src="https://user-images.githubusercontent.com/361254/43093985-8d4aa724-8e66-11e8-946c-5ccc83b62560.png">

        * Debug with Postman

          It is possible to debug a specific API endpoint through Postman leveraging a similar approach.
          In this example, we are going to debug the login authentication api endpoint rest/v11_1/oauth2/token. The first step is to add the cookie "XDEBUG_SESSION" in Postman. The same cookie is set through the Xdebug helper, and the keyword is referenced on the PHPstorm settings and on Xdebug PHP server side settings as well.
          See screenshots below:

          <img width="948" alt="Debug with Postman" src="https://user-images.githubusercontent.com/361254/43094521-0cf97058-8e68-11e8-8fc3-303c513dc1e9.png">
          <img width="679" alt="Postman cookie setting for remote debug" src="https://user-images.githubusercontent.com/361254/43094713-9190640c-8e68-11e8-95e0-11b866e452d4.png">


Session storage is completed leveraging the Redis container.

## Elasticsearch additional information
If you notice that your Elasticsearch container is not running (check with `docker ps`), it might be required to tweak your Linux host settings.
To be able to run Elasticsearch version 5 and above, it is needed an [increase of the maximum mapped memory a process can have](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/vm-max-map-count.html). To complete this change permanently run:

`echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf`

Alternatively the limit can be increased runtime with:

`sudo sysctl -w vm.max_map_count=262144`

### Docker images
* `images/php/XY/apache/` - Image for Apache with PHP version X.X
* `images/php/XY/cron/` - Image for PHP version X.Y, for background jobs and any CLI need
* `images/mysql/XY/` - Image for MySQL version X.Y
* `images/elasticsearch/XY/` - Image for Elasticsearch X.Y
* `images/permissions/` - Image for permissions fixing container
* `images/loadbalancer/` - Image for Apache load balancer
* `images/jmeter/` - Image for Jmeter
* `images/sidecar-build/` - Image for building Sidecar's javascript
* `images/traefik/` - Traefik image to expose Sugar within the local network when using a VM
* `images/ldap/` - OpenLDAP image


Most images are currently leveraging Debian linux.

### Persistent storage locations
All persistent storage is located within the `./data` directory tree within your local checkout of this git repository.
* The Sugar application files served from the web servers and leveraged by the cronjob server have to be located in `./data/app/sugar/`. Within the web servers and the cronjob server the location is `/var/www/html/sugar/`. Everything within `./data/app/` can be accessed through the browser, but the Sugar instance files have to be within `./data/app/sugar/`
* MySQL files are located in `./data/mysql/XY/`
* Elasticsearch files are located in `./data/elasticsearch/XY/`
* Redis files are located in `./data/redis/`
* LDAP files are located in `./data/ldap/`

Do not change the permissions of the various data subdirectories, as it might cause the system to not work correctly.

#### Sugar single instance application files - important notes
This setup is designed to run only a single Sugar instance. It also requires the application files to be exactly on the right place for the following three reasons:
1. File system permissions settings
2. PHP OPcache settings (eg: blacklisting of files that should not be cached)
3. Cronjob background process running

For the above reasons the single instance Sugar's files have to be located inside `./data/app/sugar/` (without subdirectories), for the stack setup to be working as expected.
If you do need multiple instances, as long as they are not running at the same time, you can leverage the provided tools to replicate and swap the data directories.

## Tips
### Utilities
To help with development, there are a set of tools provided within the `utilities` [directory of this repository](utilities). Some of the scripts are mentioned below.
#### setownership.sh
```./utilities/setownership.sh```
```
All directories and files within "data" are now owned by uid:gid 1000:1000
```
It sets the correct ownership of the data directories
#### stack.sh
```./utilities/stack.sh 80 down```
```
./utilities/stack.sh 80 down
stacks/sugar8/php71.yml down
Stopping sugar-cron          ... done
Stopping sugar-web1          ... done
Stopping sugar-redis         ... done
Stopping sugar-mysql         ... done
Stopping sugar-elasticsearch ... done
Removing sugar-cron          ... done
Removing sugar-web1          ... done
Removing sugar-redis         ... done
Removing sugar-mysql         ... done
Removing sugar-permissions   ... done
Removing sugar-elasticsearch ... done
Removing network sugar8_default
No stopped containers
```
It helps to take the default stack for the sugar version passed as a parameter, up or down. It expects two parameters: version number (eg: 80, 90 etc) and up/down.
Have a look at the configuration file `./utilities/stacks.conf`, to know all the available stack combinations for the script. For some of the main stacks is available the "local" version of the stack, that allows local modification of settings and local docker image building.
#### runcli.sh
```./utilities/runcli.sh "php ./bin/sugarcrm password:weak"```
It helps to execute a command within the CLI container. It requires the stack to be on

#### backup.sh
```./utilities/backup.sh 802_2018_11_21```
```
Backing up sugar to "backups/backup_802_2018_11_21"
[sudo] password for docker: 
Application files backed up on backups/backup_802_2018_11_21/sugar
Database backed up on backups/backup_802_2018_11_21/sugar.sql
```
It takes a snapshot of sugar files on `backups/backup_802_2018_11_21/sugar` and a MySQL database dump on `backups/backup_802_2018_11_21/sugar.sql`.
The script assumes that the database name is sugar and the web directory is sugar as well. The script does not take backups of Elasticsearch and/or Redis.
#### restore.sh
```./utilities/restore.sh 802_2018_11_21```
```
Restoring sugar from "backups/backup_802_2018_11_21"
sugar-permissions
Application files restored
Database "sugar" dropped
Database restored
Debug: Entering directory .
Repairing...
Repair completed in 9 seconds.
System repaired
```
It restores a previous snapshot of sugar files from `backups/backup_802_2018_11_21/sugar` and of MySQL from `backups/backup_802_2018_11_21/sugar.sql`
The script assumes that the database name is sugar and the web directory is sugar as well. The script does not restore Elasticsearch and/or Redis.
#### copysystem.sh
```./utilities/copysystem.sh data_80_clean data_80_clean_copy```
```
Copying "data_80_clean" to "data_80_clean_copy"
Copying data_80_clean to data_80_clean_copy
Copy completed, you can now swap or start the system
```
It helps to replicate a full `data_80_clean` content to another backup directory of choice (`data_80_clean_copy`). It requires the stack to be off (and it will check for it). This is most useful when there is the need to copy over also the content of Elasticsearch, Redis etc.
#### swapsystems.sh
```./utilities/swapsystems.sh backup_2018_06_28 data_80_clean```
```
Moving "data" to "backup_2018_06_28" and "data_80_clean" to "data"
Moving data to backup_2018_06_28
Moving data_80_clean to data
You can now start the system with the content of data_80_clean
```
It helps to move the current `data` directory to `backup_2018_06_28` and then `data_80_clean` to `data`, effectively swapping the current data in use. It requires the stack to be off (and it will check for it)

### Detect web server PHP error logs
To be able to achieve this consistently, it is recommended to leverage the single web server stack.
By running the command `docker logs -f sugar-web1` it is then possible to tail the output from the access and error log of Apache and/or PHP

### Fix Sugar permissions
You would just need to run again the permissions docker container with `docker start sugar-permissions`. The container will fix the permissions and ownership of files for you and then terminate its execution.
Apache and Cron run as the `sugar` user. Se the following options on `config_override.php`

```
$sugar_config['default_permissions']['user'] = 'sugar';
$sugar_config['default_permissions']['group'] = 'sugar';
```

### Run the included command line Repair
The application contains few scripts built to facilitate the system's repair from command line. The scripts will wipe the various caches (including OPcache and Redis if used). It will also warm-up as much as possible the application, to improve the browser experience on first load. The cron container from which the repair runs, has also been optimised to speed up the repairing processing.
To run the repair from the docker host, assuming that the repository has been checked out on sugardocker execute:

```
cd sugardocker
./repair
```
### Setup Sugar instance to leverage Redis object caching
Add on `config_override.php` the following options:
```
$sugar_config['external_cache_disabled'] = false;
$sugar_config['external_cache_disabled_redis'] = false;
$sugar_config['external_cache']['redis']['host'] = 'sugar-redis';
```
Make sure there are no other caching mechanism enabled on your config/config_override.php combination, otherwise set them as disabled = true.

### Run command line command or script
To run a PHP script execute something like the following sample commands:
```
docker@docker:~/sugardocker$ ./utilities/runcli.sh "php ../repair.php --instance ."
Debug: Entering directory .
Repairing...
Completed in 8 seconds
```

```
docker@docker:~/sugardocker$ ./utilities/runcli.sh "whoami"
sugar
```

```
docker@docker:~/sugardocker$ ./utilities/runcli.sh "pwd"
/var/www/html/sugar
```

If needed, sudo is available as well without the need of entering a password. Just make sure the permissions and ownership (user `sugar`) is respected.

### XHProf / Tideways profiling data collection

Both XHProf extension and Tideways extensions are configured in most stacks.

To enable profiling:
* If you choose to use Tideways, add [this custom code](https://github.com/esimonetti/SugarTidewaysProfiling) to your Sugar installation and repair the system.
* Configure `config_override.php` specific settings (see below based on the stack extension)

XHProf Sugar `config_override.php` configuration:
```
$sugar_config['xhprof_config']['enable'] = true;
$sugar_config['xhprof_config']['log_to'] = '../profiling';
$sugar_config['xhprof_config']['sample_rate'] = 1;
$sugar_config['xhprof_config']['flags'] = 0;
```

Tideways Sugar `config_override.php` configuration:
```
$sugar_config['xhprof_config']['enable'] = true;
$sugar_config['xhprof_config']['manager'] = 'SugarTidewaysProf';
$sugar_config['xhprof_config']['log_to'] = '../profiling';
$sugar_config['xhprof_config']['sample_rate'] = 1;
$sugar_config['xhprof_config']['flags'] = 0;
```

Make sure new files are created on `./data/app/profiling/` when navigating Sugar. If not, ensure that the directory permissions are set correctly so that the `sugar` user can write on the directory.

Please note that profiling degrades user performance, as the system is constantly writing to disk profiling information and tracking application stats. Use profiling only on replica of the production environment.

### XHProf / Tideways profiling data analysis

* Download [XHProf viewer](https://github.com/sugarcrm/xhprof-viewer/releases/latest) zip file
* Unzip its files content within `./data/app/performance/`
* Make sure the `config_override.php` settings available on `./data/app/performance/` are kept as is (`<?php
$config['profile_files_dir'] = '../profiling';`)
* Access the viewer on http://docker.local/performance/ and verify that the collected data is viewable

### Typical Sugar config_override.php options for real-life development
```
$sugar_config['external_cache_disabled'] = false;
$sugar_config['external_cache_disabled_redis'] = false;
$sugar_config['external_cache_force_backend'] = 'redis';
$sugar_config['external_cache']['redis']['host'] = 'sugar-redis';
$sugar_config['external_cache_disabled_wincache'] = true;
$sugar_config['external_cache_disabled_db'] = true;
$sugar_config['external_cache_disabled_smash'] = true;
$sugar_config['external_cache_disabled_apc'] = true;
$sugar_config['external_cache_disabled_zend'] = true;
$sugar_config['external_cache_disabled_memcache'] = true;
$sugar_config['external_cache_disabled_memcached'] = true;
$sugar_config['cache_expire_timeout'] = 600; // default: 5 minutes, increased to 10 minutes
$sugar_config['disable_vcr'] = true; // bwc module only
$sugar_config['disable_count_query'] = true; // bwc module only
$sugar_config['save_query'] = 'populate_only'; // bwc module only
$sugar_config['collapse_subpanels'] = true; // 7.6.0.0+
$sugar_config['hide_subpanels'] = true; // bwc module only
$sugar_config['hide_subpanels_on_login'] = true; // bwc module only
$sugar_config['logger']['level'] = 'fatal';
$sugar_config['logger']['file']['maxSize'] = '10MB';
$sugar_config['developerMode'] = false;
$sugar_config['dump_slow_queries'] = false;
$sugar_config['import_max_records_total_limit'] = '2000';
$sugar_config['verify_client_ip'] = false;
```
Tweak the above settings based on your specific needs.

## Mac users notes
These stacks have been built on a Mac platform, that is known to not perform well with [Docker mounted volumes](https://github.com/docker/for-mac/issues/77).
Personally I run Docker on a Debian based minimal VirtualBox VM with fixed IP, running a NFS server. I either mount NFS on my Mac when needed or SSH directly into the VM. [The Debian Docker VirtualBox VM for Mac is available here](https://github.com/esimonetti/DebianDockerMac) with its [latest downloadable version here](https://github.com/esimonetti/DebianDockerMac/releases/latest).

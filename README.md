![Atlassian Confluence Server](https://wac-cdn.atlassian.com/dam/jcr:5d1374c2-276f-4bca-9ce4-813aba614b7a/confluence-icon-gradient-blue.svg?cdnVersion=696)

Confluence Server is where you create, organise and discuss work with your
team. Capture the knowledge that's too often lost in email inboxes and shared
network drives in Confluence - where it's easy to find, use, and update. Give
every team, project, or department its own space to create the things they need,
whether it's meeting notes, product requirements, file lists, or project plans,
you can get more done in Confluence.

Learn more about Confluence Server: <https://www.atlassian.com/software/confluence>

You can find the repository for this Dockerfile at <https://hub.docker.com/r/atlassian/confluence-server>

# Contents

[TOC]

# Overview

This Docker container makes it easy to get an instance of Confluence up and
running.

*NOTE*: This Docker image is published as both `atlassian/confluence` and
`atlassian/confluence-server`. These are the same image, but the `-server`
version is deprecated and only kept for backwards-compatibility; for new
installations it is recommended to use the shorter name.

# Quick Start

For the directory in the environmental variable `CONFLUENCE_HOME` that is used
to store Confluence data (amongst other things) we recommend mounting a host
directory as a [data volume][1]:

Additionally, if running Confluence in Data Center mode it is required that a
shared filesystem is mounted. The mountpoint (inside the container) can be
configured with `CONFLUENCE_SHARED_HOME`.

Start Atlassian Confluence Server:

    docker run -v /data/your-confluence-home:/var/atlassian/application-data/confluence --name="confluence" -d -p 8090:8090 -p 8091:8091 atlassian/confluence


**Success**. Confluence is now available on <http://localhost:8090>*

Please ensure your container has the necessary resources allocated to it.  We
recommend 2GiB of memory allocated to accommodate the application server.  See
[Supported Platforms][3] for further information.

_* Note: If you are using `docker-machine` on Mac OS X, please use `open http://$(docker-machine ip default):8090` instead._

# Configuring Confluence

This Docker image is intended to be configured from its environment; the
provided information is used to generate the application configuration files
from templates. This allows containers to be repeatably created and destroyed
on-the-fly, as required in advanced cluster configurations. Most aspects of the
deployment can be configured in this manner; the necessary environment variables
are documented below. However, if your particular deployment scenario is not
covered by these settings, it is possible to override the provided templates
with your own; see the section _Advanced Configuration_ below.

## Memory / Heap Size

If you need to override Confluence Server's default memory allocation, you can
control the minimum heap (Xms) and maximum heap (Xmx) via the below environment
variables.

* `JVM_MINIMUM_MEMORY` (default: 1024m)

   The minimum heap size of the JVM

* `JVM_MAXIMUM_MEMORY` (default: 1024m)

   The maximum heap size of the JVM

* `JVM_RESERVED_CODE_CACHE_SIZE` (default: 256m)

    The reserved code cache size of the JVM

## Tomcat and Reverse Proxy Settings

If Confluence is run behind a reverse proxy server (e.g. a load-balancer or
nginx server), then you need to specify extra options to make Confluence aware
of the setup. They can be controlled via the below environment variables.

* `ATL_PROXY_NAME` (default: NONE)

   The reverse proxy's fully qualified hostname. `CATALINA_CONNECTOR_PROXYNAME`
   is also supported for backwards compatability.

* `ATL_PROXY_PORT` (default: NONE)

   The reverse proxy's port number via which Confluence is
   accessed. `CATALINA_CONNECTOR_PROXYPORT` is also supported for backwards
   compatability.

* `ATL_TOMCAT_PORT` (default: 8090)

   The port for Tomcat/Confluence to listen on. Depending on your container
   deployment method this port may need to be
   [exposed and published][docker-expose].

* `ATL_TOMCAT_SCHEME` (default: http)

   The protocol via which Confluence is accessed. `CATALINA_CONNECTOR_SCHEME` is also
   supported for backwards compatability.

* `ATL_TOMCAT_SECURE` (default: false)

   Set 'true' if `ATL_TOMCAT_SCHEME` is 'https'. `CATALINA_CONNECTOR_SECURE` is
   also supported for backwards compatability.

* `ATL_TOMCAT_CONTEXTPATH` (default: NONE)

   The context path the application is served over. `CATALINA_CONTEXT_PATH` is
   also supported for backwards compatability.

* `ATL_TOMCAT_ACCESS_LOG` (default: false)

   Whether to enable Tomcat access logging; set to `true` to enable. *NOTE*:
   These logs are written to the Container internal volume by default (under
   `/opt/atlassian/confluence/logs/`); these are rotated but not removed, and
   will grow indefinitely. If you enable this functionality it is recommended
   that you map the directory to a volume and perform log ingestion/cleanup with
   external tools.

The following Tomcat/Catalina options are also supported. For more information,
see https://tomcat.apache.org/tomcat-7.0-doc/config/index.html

* `ATL_TOMCAT_MGMT_PORT` (default: 8000)
* `ATL_TOMCAT_MAXTHREADS` (default: 48)
* `ATL_TOMCAT_MINSPARETHREADS` (default: 10)
* `ATL_TOMCAT_CONNECTIONTIMEOUT` (default: 20000)
* `ATL_TOMCAT_ENABLELOOKUPS` (default: false)
* `ATL_TOMCAT_PROTOCOL` (default: org.apache.coyote.http11.Http11NioProtocol)
* `ATL_TOMCAT_REDIRECTPORT` (default: 8443)
* `ATL_TOMCAT_ACCEPTCOUNT` (default: 10)
* `ATL_TOMCAT_DEBUG` (default: 0)
* `ATL_TOMCAT_URIENCODING` (default: UTF-8)
* `ATL_TOMCAT_MAXHTTPHEADERSIZE` (default: 8192)

## JVM configuration

If you need to pass additional JVM arguments to Confluence such as specifying a
custom trust store, you can add them via the below environment variable

* `JVM_SUPPORT_RECOMMENDED_ARGS`

   Additional JVM arguments for Confluence

Example:

    docker run -e JVM_SUPPORT_RECOMMENDED_ARGS=-Djavax.net.ssl.trustStore=/var/atlassian/application-data/confluence/cacerts -v confluenceVolume:/var/atlassian/application-data/confluence --name="confluence" -d -p 8090:8090 -p 8091:8091 atlassian/confluence

## Confluence-specific settings

* `ATL_AUTOLOGIN_COOKIE_AGE` (default: 1209600; two weeks, in seconds)

   The maximum time a user can remain logged-in with 'Remember Me'.

* `CONFLUENCE_HOME`

   The confluence home directory. This may be on an mounted volume; if so it
   should be writable by the user `confluence`. See note below about UID
   mappings.

* `ATL_LUCENE_INDEX_DIR`

  The directory where [Lucene](https://lucene.apache.org/) search indexes should
  be stored. Defaults to `index` under the Confluence home directory.

* `ATL_LICENSE_KEY` (from Confluence 7.9 onwards)

  The Confluence license string. Providing this will remove the need to supply it through the web startup screen.
  
* *use with caution* `CONFLUENCE_LOG_STDOUT` `[true, false]`  (from Confluence 7.9 onwards)

  Prior to Confluence version 7.9.0, the log files are always stored in the `logs` folder in Confluence home. From version 
  7.9.0, the logs can be printed directly to the `stdout` and don't use the file at all. This makes it possible to fetch the log messages
  via `docker logs <CONTAINER_ID>`. In this setup we recommend using some log aggregation tooling (e.g. AWS Cloudwatch or ELK stack).
  
  **Beware, if enabled, the support ZIP produced by the Troubleshooting and Support plugin doesn't contain the application logs.**
  
## Database configuration

It is optionally possible to configure the database from the environment,
avoiding the need to do so through the web startup screen.

The following variables are all must all be supplied if using this feature:

* `ATL_JDBC_URL`

   The database URL; this is database-specific.

* `ATL_JDBC_USER`

   The database user to connect as.

* `ATL_JDBC_PASSWORD`

   The password for the database user.

* `ATL_DB_TYPE`

   The type of database; valid supported values are:

   * `mssql`
   * `mysql`
   * `oracle12c` (Confluence 7.3.0 or earlier only)
   * `oracle` (Confluence 7.3.1 or later only. Compatible with Oracle 12c and Oracle 19c)
   * `postgresql`

Note: Due to licensing restrictions Confluence does not ship with a MySQL or
Oracle JDBC drivers. To use these databases you will need to copy a suitable
driver into the container and restart it. For example, to copy the MySQL driver
into a container named "confluence", you would do the following:

    docker cp mysql-connector-java.x.y.z.jar confluence:/opt/atlassian/confluence/confluence/WEB-INF/lib
    docker restart confluence

For more information see the [Database JDBC Drivers](https://confluence.atlassian.com/doc/database-jdbc-drivers-171742.html)
page.

### Optional database settings

The following variables are for the database connection pool, and are
optional.

* `ATL_DB_POOLMINSIZE` (default: 20)
* `ATL_DB_POOLMAXSIZE` (default: 100)
* `ATL_DB_TIMEOUT` (default: 30)
* `ATL_DB_IDLETESTPERIOD` (default: 100)
* `ATL_DB_MAXSTATEMENTS` (default: 0)
* `ATL_DB_VALIDATE` (default: false)
* `ATL_DB_ACQUIREINCREMENT` (default: 1)
* `ATL_DB_VALIDATIONQUERY` (default: "select 1")

## Data Center configuration

This docker image can be run as part of a [Data Center][4] cluster. You can
specify the following properties to start Confluence as a Data Center node,
instead of manually configuring a cluster. See [Installing Confluence Data
Center][5] for more information.

### Cluster configuration

Confluence Data Center allows clustering via various methods. For more
information on the setting for each type see [this page][6].

**NOTE:** The underlying network should be set-up to support the Confluence
clustering type you are using. How to do this depends on the container
management technology, and is beyond the scope of this documentation.

#### Common cluster settings

* `ATL_CLUSTER_TYPE`

   The cluster type. Setting this effectively enables clustering. Valid values
   are `aws`, `multicast`, and `tcp_ip`.

* `ATL_CLUSTER_NAME`

   The cluster name; this should be common across all nodes.

* `ATL_PRODUCT_HOME_SHARED`

   The location of the shared home directory for all Confluence nodes. **Note**:
   This must be real shared filesystem that is mounted inside the
   container. Additionally, see the note about UIDs.

* `ATL_CLUSTER_TTL`

   The time-to-live for cluster packets. Primarily of use in multicast clusters.

#### AWS cluster settings

   The following should be populated from the AWS environment.

* `ATL_HAZELCAST_NETWORK_AWS_IAM_ROLE`
* `ATL_HAZELCAST_NETWORK_AWS_IAM_REGION`
* `ATL_HAZELCAST_NETWORK_AWS_HOST_HEADER`
* `ATL_HAZELCAST_NETWORK_AWS_SECURITY_GROUP`
* `ATL_HAZELCAST_NETWORK_AWS_TAG_KEY`
* `ATL_HAZELCAST_NETWORK_AWS_TAG_VALUE`

#### TCP cluster settings

* `ATL_CLUSTER_PEERS`

   A comma-separated list of peer IPs.

#### Multicast cluster settings

* `ATL_CLUSTER_ADDRESS`

   The multicast address the cluster will communicate on.

## Container Configuration

* `SET_PERMISSIONS` (default: true)

   Define whether to set home directory permissions on startup. Set to `false` to disable
   this behaviour.

## Advanced Configuration

As mentioned at the top of this section, the settings from the environment are
used to populate the application configuration on the container startup. However
in some cases you may wish to customise the settings in ways that are not
supported by the environment variables above. In this case, it is possible to
modify the base templates to add your own configuration. There are three main
ways of doing this; modify our repository to your own image, build a new image
from the existing one, or provide new templates at startup. We will briefly
outline this methods here, but in practice how you do this will depend on your
needs.

#### Building your own image

* Clone the Atlassian repository at https://bitbucket.org/atlassian-docker/docker-atlassian-confluence-server/
* Modify or replace the [Jinja](https://jinja.palletsprojects.com/) templates
  under `config`; _NOTE_: The files must have the `.j2` extensions. However you
  don't have to use template variables if you don't wish.
* Build the new image with e.g: `docker build --tag my-confluence-image --build-arg CONFLUENCE_VERSION=6.x.x .`
* Optionally push to a registry, and deploy.

#### Build a new image from the existing one

* Create a new `Dockerfile`, which starts with the line e.g: `FROM atlassian/confluence:latest`.
* Use a `COPY` line to overwrite the provided templates.
* Build, push and deploy the new image as above.

#### Overwrite the templates at runtime

There are two main ways of doing this:

* If your container is going to be long-lived, you can create it, modify the
  installed templates under `/opt/atlassian/etc/`, and then run it.
* Alternatively, you can create a volume containing your alternative templates,
  and mount it over the provided templates at runtime
  with `--volume my-config:/opt/atlassian/etc/`.

# Shared directory and user IDs

By default the Confuence application runs as the user `confluence`, with a UID
and GID of 2002. Consequently this UID must have write access to the shared
filesystem. If for some reason a different UID must be used, there are a number
of options available:

* The Docker image can be rebuilt with a different UID.
* Under Linux, the UID can be remapped using
  [user namespace remapping][7].

To preserve strict permissions for certain configuration files, this container starts as
`root` to perform bootstrapping before running Confluence under a non-privileged user
account. If you wish to start the container as a non-root user, please note that Tomcat
configuration, and the bootstrapping of seraph-config.xml (SSO) &
confluence-init.properties (overriding `$CONFLUENCE_HOME`) will be skipped and a warning
will be logged. You may still apply custom configuration in this situation by mounting a
custom file directly, e.g. by mounting your own server.xml file directly to
`/opt/atlassian/confluence/conf/server.xml`

Database and Clustering bootstrapping will work as expected when starting this container
as a non-root user.

# Upgrade

To upgrade to a more recent version of Confluence Server you can simply stop the
`Confluence` container and start a new one based on a more recent image:

    docker stop confluence
    docker rm confluence
    docker run ... (see above)

As your data is stored in the data volume directory on the host, it will still
be available after the upgrade.

_Note: Please make sure that you **don't** accidentally remove the `confluence`
container and its volumes using the `-v` option._

# Backup

For evaluating Confluence you can use the built-in database that will store its
files in the Confluence Server home directory. In that case it is sufficient to
create a backup archive of the directory on the host that is used as a volume
(`/data/your-confluence-home` in the example above).

Confluence's [automatic backup][8] is currently supported in the Docker
setup. You can also use the [Production Backup Strategy][9] approach if you're
using an external database.

Read more about data recovery and backups: [Site Backup and Restore][10]

# Shutdown

Confluence allows a grace period of 20s for active operations to finish before
termination. If sending a `docker stop` this should be taken into account with
the `--time` flag.

Alternatively, the script `/shutdown-wait.sh` is provided, which will initiate a
clean shutdown and wait for the process to complete. This is the recommended
method for shutdown in environments which provide for orderly shutdown,
e.g. Kubernetes via the `preStop` hook.

# Versioning

The `latest` tag matches the most recent official release of Atlassian Confluence Server.
So `atlassian/confluence:latest` will use the newest stable version of
Confluence Server available.

Alternatively, you can use a specific minor version of Confluence Server by
using a version number tag: `atlassian/confluence:6.13`. This will
install the latest `6.13.x` version that is available.

We also publish docker images for our [EAP releases](https://www.atlassian.com/software/confluence/download-eap) (not
supported for use in production). The tag for EAP releases is the EAP version.
For example to get the `7.8.0-beta1` EAP release, use `atlassian/confluence:7.8.0-beta1`.

For example, `atlassian/confluence:6.13-ubuntu-18.04-adoptopenjdk8` will
install the latest 6.13.x version with AdoptOpenJDK 8.

# Supported JDK versions

All the Atlassian Docker images are now JDK11 only, and generated from the
[official AdoptOpenJDK Docker images](https://hub.docker.com/r/adoptopenjdk/openjdk11).

The Docker images follow the [Atlassian Support end-of-life
policy](https://confluence.atlassian.com/support/atlassian-support-end-of-life-policy-201851003.html);
images for unsupported versions of the products remain available but will no longer
receive updates or fixes.

Historically, we have also generated other versions of the images, including
JDK8, Alpine, and 'slim' versions of the JDK. These legacy images still exist in
Docker Hub, however they should be considered deprecated, and do not receive
updates or fixes.

If for some reason you need a different version, see "Building your own image"
above.

# Troubleshooting

These images include built-in scripts to assist in performing common JVM diagnostic tasks.

## Thread dumps

`/opt/atlassian/support/thread-dumps.sh` can be run via `docker exec` to easily trigger the collection of thread
dumps from the containerized application. For example:

    docker exec my_container /opt/atlassian/support/thread-dumps.sh

By default this script will collect 10 thread dumps at 5 second intervals. This can
be overridden by passing a custom value for the count and interval, by using `-c` / `--count`
and `-i` / `--interval` respectively. For example, to collect 20 thread dumps at 3 second intervals:

    docker exec my_container /opt/atlassian/support/thread-dumps.sh --count 20 --interval 3

Thread dumps will be written to `$APP_HOME/thread_dumps/<date>`.

Note: By default this script will also capture output from top run in 'Thread-mode'. This can
be disabled by passing `-n` / `--no-top`

## Heap dump

`/opt/atlassian/support/heap-dump.sh` can be run via `docker exec` to easily trigger the collection of a heap
dump from the containerized application. For example:

    docker exec my_container /opt/atlassian/support/heap-dump.sh

A heap dump will be written to `$APP_HOME/heap.bin`. If a file already exists at this
location, use `-f` / `--force` to overwrite the existing heap dump file.

## Manual diagnostics

The `jcmd` utility is also included in these images and can be used by starting a `bash` shell
in the running container:

    docker exec -it my_container /bin/bash

# Support

For product support, go to
[support.atlassian.com](https://support.atlassian.com/confluence-server/).

You can also visit the [Atlassian Data Center on
Kubernetes](https://community.atlassian.com/t5/Atlassian-Data-Center-on/gh-p/DC_Kubernetes)
forum for discussion on running Atlassian Data Center products in containers.

# Contribution

See the [contributing guideline](CONTRIBUTING.md) if you are contributing from outside Atlassian.

# License

Copyright © 2020 Atlassian Corporation Pty Ltd.
Licensed under the Apache License, Version 2.0.

[1]: https://docs.docker.com/userguide/dockervolumes/#mount-a-host-directory-as-a-data-volume
[3]: https://confluence.atlassian.com/display/DOC/Supported+platforms
[4]: https://confluence.atlassian.com/doc/confluence-data-center-technical-overview-790795847.html
[5]: https://confluence.atlassian.com/doc/installing-confluence-data-center-203603.html
[6]: https://confluence.atlassian.com/doc/change-node-discovery-from-multicast-to-tcp-ip-or-aws-792297728.html#ChangeNodeDiscoveryfromMulticasttoTCP/IPorAWS-TochangefromTCP/IPtomulticast
[7]: https://docs.docker.com/engine/security/userns-remap/
[8]: https://confluence.atlassian.com/display/DOC/Configuring+Backups
[9]: https://confluence.atlassian.com/display/DOC/Production+Backup+Strategy
[10]: https://confluence.atlassian.com/display/DOC/Site+Backup+and+Restore
[12]: https://confluence.atlassian.com/doc/confluence-6-13-release-notes-959288785.html

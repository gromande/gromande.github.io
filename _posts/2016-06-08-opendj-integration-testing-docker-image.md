---
layout: post
title: OpenDJ Docker Image for Integration Testing
date: 2016-06-08 22:40:48.000000000 -05:00
password: ''
categories: []
tags: []
---
As I mentioned in my [previous post]({{ site.baseurl }}/opendj-java-sdk-testing-part-1/)&nbsp;I've been playing with Docker lately trying to build an OpenDJ image that I could use for integration testing. Before digging deeper into my testing strategy I decided to publish this short post to explain how such image can be created using a simple Dockerfile.  
<!--more-->

You can go ahead and [download](https://github.com/gromande/opendj-docker-image) the project from Github if you want to play with it. This is what the Dockerfile looks like:

```
FROM java:8 WORKDIR /opt EXPOSE 1389 1636 4444 ADD opendj.zip /var/tmp/ RUN unzip /var/tmp/opendj.zip -d /opt/ && rm -fr /var/tmp/opendj.zip ENV JAVA\_HOME /usr/lib/jvm/java-8-openjdk-amd64/ RUN /opt/opendj/setup --cli -p 1389 --ldapsPort 1636 --enableStartTLS --generateSelfSignedCertificate --baseDN "dc=example,dc=com" -h localhost --rootUserPassword password --acceptLicense --no-prompt --doNotStart ADD 99-custom-schema.ldif /opt/opendj/config/schema/ ADD custom-dit.ldif /var/tmp/ RUN /opt/opendj/bin/import-ldif --backendID userRoot --ldifFile /var/tmp/custom-dit.ldif && rm -fr /var/tmp/custom-dit.ldif CMD ["/opt/opendj/bin/start-ds", "--nodetach"]
```

Let's look at this file line by line.

```
FROM java:8
```

The base image is _java:8&nbsp;_which is available from the public [Docker Hub](https://hub.docker.com/_/java/) and, as the name implies, provides us with an image that has Java 8 already installed.

```
EXPOSE 1389 1636 4444
```

We need to expose the OpenDJ ports so that we can connect&nbsp;to them once the container is running:

- 1389: LDAP port
- 1636: LDAPS port
- 4444: Admin port

```
ADD opendj.zip /var/tmp/
```

By default the Dockerfile expect to find the OpenDJ binaries in a file called _opendj.zip_. You can download any OpenDJ version from the [Fogerock Backstage](https://backstage.forgerock.com/#!/downloads/OpenDJ/OpenDJ%20Enterprise#browse)&nbsp;page, rename it to _opendj.zip_&nbsp;and move it to the project folder.

If you prefer to use the nightly build you can comment out the previous line and uncomment the following one:

```
RUN echo `curl -s https://forgerock.org/djs/opendjrel.js | grep -o "http://.*\.zip" | tail -1` | xargs curl -o /var/tmp/opendj.zip
```

Once the OpenDJ zip file has been added we need to extract the contents to the installation directory:

```
RUN unzip /var/tmp/opendj.zip -d /opt/ && rm -fr /var/tmp/opendj.zip
```

And then run the installation script:

```
RUN /opt/opendj/setup --cli \ -p 1389 \ --ldapsPort 1636 \ --enableStartTLS \ --generateSelfSignedCertificate \ --baseDN "dc=example,dc=com" \ -h localhost \ --rootUserPassword password \ --acceptLicense \ --no-prompt \ --doNotStart
```

Besides specifying ports and hostname we also have to choose a _base DN_, password for the _"cn=Directory Manager"_ user, &nbsp;and tell the script not to start OpenDJ just yet.

I decided to add the following lines at the end to be able to load a custom schema file and some sample data to initialize our directory:

```
ADD 99-custom-schema.ldif /opt/opendj/config/schema/ ADD custom-dit.ldif /var/tmp/ RUN /opt/opendj/bin/import-ldif \ --backendID userRoot \ --ldifFile /var/tmp/custom-dit.ldif \ && rm -fr /var/tmp/custom-dit.ldif
```

And finally, this line&nbsp;tells Docker which command to run when the container starts:

```
CMD ["/opt/opendj/bin/start-ds", "--nodetach"]
```

We need to start OpenDJ in no-detach mode to keep the container running afterwards.

And that's it! You can build and spin up a container by running the following commands (feel free to use your own image name and port mappings):

```
docker build -t groman/opendj-test:1.0 . docker run -d -p 1389:1389 1636:1636 444:4444 groman/opendj-test:1.0
```

Be advised that any changes&nbsp;made after the container was started (via _ldapmodify_ for example) will be lost once the container is stopped. This was put in place intentionally to make sure our integration tests always work with the same pre-define data.

Enjoy!


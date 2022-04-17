# Open Liberty Buildpack

Experimental Cloud Foundry for Open Liberty.

# NOTE

The normal Java buildpack can be used in Cloud Foundry to deploy Open Liberty.

Use a manifest like this:

```
---
applications:
- name: my-app-name
  buildpacks:
  - java_buildpack
  command: JAVA_HOME=~/.java-buildpack/open_jdk_jre PATH=$PATH:$JAVA_HOME/bin:~/wlp/bin server run
```

When using Maven, for a single project (not a multi-project, so EAR projects cannot do this) the following command sequence will build the Zip file for deployment:

```
$ mvn clean package liberty:create liberty:install-feature liberty:deploy liberty:package -Dinclude=minify
```

For multi-projects, it appears that the Open Liberty command-line (`server`) tools need to be employed something like this:

```
$ cd ~/tmp
$ java -version
openjdk version "1.8.0_312"
OpenJDK Runtime Environment (build 1.8.0_312-8u312-b07-0ubuntu1~20.04-b07)
OpenJDK 64-Bit Server VM (build 25.312-b07, mixed mode)
# ^^ Just to verify the current version of Java is being used. The Cloud Foundry java_buildpack defaults to Java 1.8.
$ git clone git@github.com:OpenLiberty/guide-maven-multimodules.git
<snip>
$ cd guide-maven-multimodules/finish
$ mvn clean package
<snip>
$ cd ~/tmp
# Download current version of Open Liberty (22.0.0.4 in this example)
$ unzip ~/Downloads/openliberty-22.0.0.4.zip
<snip>
$ wlp/bin/server create

Server defaultServer created.
$ cp guide-maven-multimodules/finish/ear/target/guide-maven-multimodules-ear.ear wlp/usr/servers/defaultServer/dropins/
$ cp guide-maven-multimodules/finish/ear/src/main/liberty/config/server.xml .
# EDIT: server.xml to have a host entry like this (or httpPort="${PORT}"):
#    <httpEndpoint host="*" httpPort="8080"
#        httpsPort="-1" id="defaultHttpEndpoint" />
$ cp server.xml wlp/usr/servers/defaultServer/server.xml
$ wlp/bin/server package --include=minify

Packaging server defaultServer.
Querying server defaultServer for content.
Launching defaultServer (Open Liberty 22.0.0.4/wlp-1.0.63.cl220420220328-1303) on OpenJDK 64-Bit Server VM, version 1.8.0_312-8u312-b07-0ubuntu1~20.04-b07 (en_US)
[AUDIT   ] CWWKE0001I: The server defaultServer has been launched.
[AUDIT   ] CWWKF0012I: The server installed the following features: [expressionLanguage-4.0, pages-3.0, servlet-5.0].
[AUDIT   ] CWWKF0026I: The server defaultServer is ready to build a smaller package.
[AUDIT   ] CWWKE1100I: Waiting for up to 30 seconds for the server to quiesce.
[AUDIT   ] CWWKE0036I: The server defaultServer stopped after 0.918 seconds.
Building archive for server defaultServer.
Server defaultServer package complete in ~/tmp/wlp/usr/servers/defaultServer/defaultServer.zip.
$ cat > manifest.yml
---
applications:
- name: guide-maven-multimodules
  buildpacks:
  - java_buildpack
  command: JAVA_HOME=~/.java-buildpack/open_jdk_jre PATH=$PATH:$JAVA_HOME/bin:~/wlp/bin server run
<Type control+D>
$ cf push -f manifest.yml -p wlp/usr/servers/defaultServer/defaultServer.zip
<snip>
name:              guide-maven-multimodules
requested state:   started
routes:            guide-maven-multimodules.mydomain.com
last uploaded:     Sun 17 Apr 13:47:25 CDT 2022
stack:             cflinuxfs3
buildpacks:        
	name             version                                                         detect output   buildpack name
	java_buildpack   v4.42-git@github.com:cloudfoundry/java-buildpack.git#91aefacf   java            java

type:            web
sidecars:        
instances:       1/1
memory usage:    1024M
start command:   JAVA_HOME=~/.java-buildpack/open_jdk_jre PATH=$PATH:$JAVA_HOME/bin:~/wlp/bin server run
     state     since                  cpu     memory        disk           details
#0   running   2022-04-17T18:48:10Z   17.2%   58.4M of 1G   422.6M of 1G   
<snip>
```

# Using the Buildpack

## Building the Buildpack

> Note: the buildpack currently only works as a static buildpack.

```
$ buildpack-packager build -cached -any-stack
```

Upload the resulting buildpack as usual.

## Deployment

> The package structure is subject to change.

Package the deployment in a Zip file. Contents are as follows:

```
apps/
  WAR/EAR files
config/
  server.xml
```

`cf push` the app as usual.

## Resources

* [About Buildpacks](https://docs.cloudfoundry.org/buildpacks/understand-buildpacks.html)
* [Creating Custom Buildpacks](https://docs.cloudfoundry.org/buildpacks/custom.html)
* [Buildpack Packager](https://github.com/cloudfoundry/libbuildpack/tree/master/packager)
* [Open Liberty Docs](https://openliberty.io/docs/22.0.0.4/overview.html)

## See Also 

* [Paketo Buildpacks](https://github.com/paketo-buildpacks/liberty)

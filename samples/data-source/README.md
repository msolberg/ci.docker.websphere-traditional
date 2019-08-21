# Hello World

A simple `Hello World` application that demonstrate a basic pattern for deploying an application into IBM Cloud Private using the official helm chart for IBM WebSphere Application Server traditional Base edition.

## Building the application image
Dockerfile adds three things to build application image
1. application EAR file
2. application installation script (Jython)
3. sample properties file that increases container thread pool to 100

Run following command inside this directory:

`docker build -t app .`

## Customizing the app server using properties

An easy way to customize the application server is to log into the admin console locally, make changes to the environment, export the properties and then add them to the docker build. These properties then can also be modified at run time with a config map.

1. Start the application server image you've built:

`docker run -p 9080:9080 -p 9043:9043 app:1`

2. Extract the password so that you can log into the console:

`docker exec <CONTAINERID> cat /tmp/PASSWORD`

3. Start a terminal on the running container and dump the default properties into a file:

```
$ docker exec -it <CONTAINERID> /bin/bash
$ /opt/IBM/WebSphere/AppServer/bin/wsadmin.sh -lang jython -conntype NONE
wsadmin> AdminTask.extractConfigProperties('-propertiesFileName /tmp/ws.props')
```

4. Log into the console using the password extracted above and make changes. Save the configuration in the console.

5. Dump the modified properties to another file and find differences between the original config and the modified config:

```
wsadmin> AdminTask.extractConfigProperties('-propertiesFileName /tmp/ws-modified.props')
$ diff -rup /tmp/ws.props /tmp/ws-modified.props > /tmp/ws.props.diff
```

6. That diff file can be copied to a local machine for editing using `docker cp`

```
$ docker cp <CONTAINERID>:/tmp/ws.props.diff ws.props.diff
```

7. Testing the new file during the build can be difficult. To test, stop the running container, start a new one, copy the properties file, and then apply it.

```
$ docker cp changes.props <CONTAINERID>:/tmp/changes.props
$ docker exec -it <CONTAINERID> /bin/bash
$ /opt/IBM/WebSphere/AppServer/bin/wsadmin.sh -lang jython -conntype NONE
wsadmin> AdminTask.applyConfigProperties(['-propertiesFileName /tmp/changes.props -reportFileName /tmp/changes.txt'])
wsadmin> AdminConfig.save()
```




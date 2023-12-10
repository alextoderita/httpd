# Apache Container

An Apache container loading static html files.

## Getting Started

These instructions will cover usage information for the docker container

### Prerequisities for Local Deployment

In order to run this container you'll need docker installed.

* [Windows](https://docs.docker.com/windows/started)
* [OS X](https://docs.docker.com/mac/started/)
* [Linux](https://docs.docker.com/linux/started/)

### Usage

#### Build and run the container locally

Build the httpd docker image:

```shell
$ docker build -t httpd .
[+] Building 1.2s (8/8) FINISHED                                                                                            docker:desktop-linux
 => [internal] load .dockerignore                                                                                                           0.0s
 => => transferring context: 2B                                                                                                             0.0s
 => [internal] load build definition from Dockerfile                                                                                        0.0s
 => => transferring dockerfile: 145B                                                                                                        0.0s
 => [internal] load metadata for docker.io/library/httpd:2.4                                                                                1.0s
 => [auth] library/httpd:pull token for registry-1.docker.io                                                                                0.0s
 => [internal] load build context                                                                                                           0.0s
 => => transferring context: 148B                                                                                                           0.0s
 => [1/2] FROM docker.io/library/httpd:2.4@sha256:04551bc91cc03314eaab20d23609339aebe2ae694fc2e337d0afad429ec22c5a                          0.0s
 => CACHED [2/2] COPY ./public-html/index.html /usr/local/apache2/htdocs/                                                                   0.0s
 => exporting to image                                                                                                                      0.0s
 => => exporting layers                                                                                                                     0.0s
 => => writing image sha256:d4c8d5200ffd39bf4d8c4c01db0646b74789e84222c49e2f40b801d3b903f066                                                0.0s
 => => naming to docker.io/library/httpd                                                                                                    0.0s
```

Start the httpd container locally:

```shell
$ docker run -dit --name httpd -p 8080:80 httpd
3ef3a8ed56bcf8f0bdbd9af3d8d6dc4362f23ad1ba8a68ee25a623b5fdf6e75a
```

Check the response of the container running locally:
```shell
$ curl -v http://localhost:8080
*   Trying 127.0.0.1:8080...
* Connected to localhost (127.0.0.1) port 8080 (#0)
> GET / HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/8.1.2
> Accept: */*
> 
< HTTP/1.1 200 OK
< Date: Sun, 10 Dec 2023 07:42:32 GMT
< Server: Apache/2.4.58 (Unix)
< Last-Modified: Sat, 09 Dec 2023 20:26:55 GMT
< ETag: "31-60c19859cbc5b"
< Accept-Ranges: bytes
< Content-Length: 49
< Content-Type: text/html
< 
<html><body><h1>Alex Toderita</h1></body></html>
* Connection #0 to host localhost left intact
```

### Prerequisites for Cloud deployments - we're using Red Hat OpenShift Service on AWS (ROSA) in this example

* [OpenShift](https://docs.aws.amazon.com/ROSA/latest/userguide/getting-started.html)
* [OpenShift CLI](https://docs.openshift.com/container-platform/4.8/cli_reference/openshift_cli/getting-started-cli.html)

#### Publish the docker image and run the application in OpenShift

Tag the docker image to be published to an external docker registry:
```shell
docker tag httpd:latest default-route-openshift-image-registry.apps.rosa.rosa-hcp-public.gbk1.p3.openshiftapps.com/default/httpd:latest
```

Push the previously tagged image to an external docker registry:
```shell
$ docker push default-route-openshift-image-registry.apps.rosa.rosa-hcp-public.gbk1.p3.openshiftapps.com/default/httpd:latest
The push refers to repository [default-route-openshift-image-registry.apps.rosa.rosa-hcp-public.gbk1.p3.openshiftapps.com/default/httpd]
99a043a1c1af: Pushed 
b9bf93af811f: Pushed 
28f18e7dc61c: Pushed 
27496babc700: Pushed 
fad7e2250d8f: Pushed 
92770f546e06: Pushed 
latest: digest: sha256:18236a491ffadb9e7ae3fdfc68cc0131148fa5f9004bc059c844a432106b3b72 size: 1573
```

Deploy a new OpenShift app called httpd:
```shell
$ oc new-app httpd --name=httpd
--> Found image d4c8d52 (4 minutes old) in image stream "default/httpd" under tag "latest" for "httpd"
--> Creating resources ...
    deployment.apps "httpd" created
    service "httpd" created
--> Success
```

Application is not exposed. You can expose services to the outside world by executing one or more of the commands below:
```shell
$ oc expose service/httpd
route.route.openshift.io/httpd exposed
```

Identify the route/FQDN port number for the exposed application:
```shell
$ oc get route httpd
NAME    HOST/PORT                                                           PATH   SERVICES   PORT     TERMINATION   WILDCARD
httpd   httpd-default.apps.rosa.rosa-hcp-public.gbk1.p3.openshiftapps.com          httpd      80-tcp                 None
```

Check the response of the publicly exposed app:

```shell
$ curl -v http://httpd-default.apps.rosa.rosa-hcp-public.gbk1.p3.openshiftapps.com
*   Trying 54.73.18.112:80...
* Connected to httpd-default.apps.rosa.rosa-hcp-public.gbk1.p3.openshiftapps.com (54.73.18.112) port 80 (#0)
> GET / HTTP/1.1
> Host: httpd-default.apps.rosa.rosa-hcp-public.gbk1.p3.openshiftapps.com
> User-Agent: curl/8.1.2
> Accept: */*
> 
< HTTP/1.1 200 OK
< date: Sun, 10 Dec 2023 08:07:10 GMT
< server: Apache/2.4.58 (Unix)
< last-modified: Sat, 09 Dec 2023 20:26:55 GMT
< etag: "31-60c19859341c0"
< accept-ranges: bytes
< content-length: 49
< content-type: text/html
< set-cookie: f0a236c69785b2fbbed70ccb030e4ffe=d789a1bb5b46f2593673d7598c0ca087; path=/; HttpOnly
< cache-control: private
< 
<html><body><h1>Alex Toderita</h1></body></html>
* Connection #0 to host httpd-default.apps.rosa.rosa-hcp-public.gbk1.p3.openshiftapps.com left intact
```

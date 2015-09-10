---
title: Using ASP.NET with Rackspace Container Service
slug: deploy-from-visual-studio
description: How to deploy an ASP.NET web application to Rackspace Container Service using Visual Studio
topics:
  - docker
  - windows
---

<!--ASP.NET 5 can run on Linux but most .NET devs aren't familiar enough with Linux to know how to deploy or run their server. RCS would work well in this situation as the entire process can be mediated through familiar tools Visual Studio and VS Online.

* How to test on a local docker container running linux from your windows dev box
* Deploy from Visual Studio to your local machine
* Deploy directly to RCS
* CI/CD
 * Build a docker image using VSO
 * Publish to docker hub
 * Deploy to staging
 * Deploy to production
-->

Visual Studio is tightly integrated with Microsoft Azure and with Visual Studio 2015
you can now deploy directly from Visual Studio to Docker containers in Azure. But did you
know that you can deploy your ASP.NET 5 application to other container hosts? Follow along
and learn to deploy your application to Rackspace Container Service using Visual Studio.

* [Requirements](#Requirements)
* [A Note on Docker Versions]()

## <a name="Requirements"></a> Requirements ##

* [Visual Studio 2015 Community Edition or Higher][get-vs]
* [Microsoft ASP.NET and Web Tools 2015][vs-webtools-plugin] Extension
* [Visual Studio 2015 Tools for Docker][vs-docker-plugin] Extension. *This requires the ASP.NET and Web Tools, so make sure to install that first*

## <a name="DockerVersions"></a>"A Note on Docker Versions ##

In general when using Docker, the client and server versions must match.
The Visual Studio Tools for Docker extension ships with its own docker.exe,
instead of relying upon a system install. This means that when using a custom
docker host, that you may need to replace docker.exe with a compatible version.

If you see an error message when publishing about mis-matched Docker versions,
follow the steps below to update the extension with the correct docker client.

<!-- TODO: Add example message about versions needing to match -->

1. Identify the version of Docker used by the server from the error message.
2. Download the appropriate docker client. Below are direct links from the [Docker releases page][docker-releases].
  * [Docker Client - 1.6.1](https://get.docker.com/builds/Windows/x86_64/docker-1.6.1.exe)
3. Open Windows Explorer and browse to the Docker Tools extension directory:
        C:\Program Files (x86)\Microsoft Visual Studio 14.0\Common7\IDE\Extensions\Microsoft\Windows Azure Tools\Docker
        cd /d "%VS140ComnTools%\..\IDE\Extensions\Microsoft\Windows Azure Tools\Docker"
4. In that directory, rename `docker.exe` to `docker-1.8.1.exe` for safe-keeping.
5. Move the docker client that you downloaded in step 2 into this directory and rename it to `docker.exe`.

[vs-webtools-plugin]: https://www.microsoft.com/en-us/download/details.aspx?id=48738
[vs-docker-plugin]: https://visualstudiogallery.msdn.microsoft.com/0f5b2caa-ea00-41c8-b8a2-058c7da0b3e4
[get-vs]: https://www.visualstudio.com/downloads/download-visual-studio-vs
[docker-releases]: https://github.com/docker/docker/releases

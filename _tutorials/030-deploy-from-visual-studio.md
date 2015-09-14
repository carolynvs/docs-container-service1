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

Visual Studio is tightly integrated with Microsoft Azure and now with Visual Studio 2015
you can deploy directly from Visual Studio to Docker containers in Azure. But did you
know that you can deploy your ASP.NET 5 application to other container hosts? Follow along
and learn to deploy your application to Rackspace Container Service using Visual Studio.

1. [Requirements](#Requirements)
  * [A Note on Docker Versions](#DockerVersions)
2. [Create an ASP.NET 5 Application](#CreateApp)
3. [Publish](#Publish)
  * [Publish from Visual Studio](#VisualStudioPublish)
  * [Publish from Command Line](#CommandLinePublish)
  * [Publish from MS Build](#MSBuildPublish)

## <a name="Requirements"></a> Requirements ##

* [Visual Studio 2015 Community Edition or Higher][get-vs]
* [Microsoft ASP.NET and Web Tools 2015][vs-webtools-plugin] Extension
* [Visual Studio 2015 Tools for Docker][vs-docker-plugin] Extension <br/>
*This requires the ASP.NET and Web Tools*
* [Rackspace Cluster Credentials][get-cluster-creds]

[vs-webtools-plugin]: https://www.microsoft.com/en-us/download/details.aspx?id=48738
[vs-docker-plugin]: https://visualstudiogallery.msdn.microsoft.com/0f5b2caa-ea00-41c8-b8a2-058c7da0b3e4
[get-vs]: https://www.visualstudio.com/downloads/download-visual-studio-vs

## <a name="CreateApp"></a> Create an ASP.NET 5 Application ##
Create a new ASP.NET 5 application.

1. Open Visual Studio
2. Go to File &rarr; New &rarr; Project then select the ASP.NET Web Application template.
![New Project Window]({% asset_path 030-deploy-from-visual-studio/new-project.png %})

* Kestrel command should look like this:

        "kestrel": "Microsoft.AspNet.Hosting --server Microsoft.AspNet.Server.Kestrel --config hosting.ini"
* Hosting.ini should look like this:
  * server.urls=http://localhost:5000

## <a name="Publish"></a> Publish ##

### <a name="VisualStudioPublish"></a> Publish from Visual Studio ###

Right click on project and select publish

* Choose custom docker host
* The url is the docker_host environment variable from docker.env (e.g. tcp://104.130.0.124:2376)
* Host port is 80
* Container port should match the port used in your hosting.ini, e.g. 5000
* Specify the authentication in the auth options advanced setting

        --tls --tlscert="Y:\23aed633-dab0-4a97-a888-fc4e681a990d\cert.pem" --tlskey="Y:\23aed633-dab0-4a97-a888-fc4e681a990d\key.pem"
* verify


### <a name="CommandLinePublish"></a> Publish from Command Line ###

### <a name="MSBuildPublish"></a> Publish from MS Build ###

## <a name="DockerVersions"></a>A Note on Docker Versions ##

In general when using Docker, the client and server versions must match.
The Visual Studio Tools for Docker extension first tries to use the docker client
that is accessible via the current PATH environment variable and then falls back to
using the docker client in the extension installation directory. This means that when using a custom
docker host, that you may need to tweak your publish profile to use a different version of the docker client.

If you see an error message like `Error response from daemon: client and server don't have same version (client : 1.20, server: 1.18)`
when publishing about mis-matched Docker versions, follow the steps below to
explicitly specify which version of the docker client to use.

1. Identify the version of Docker used by the server from the error message.
2. Download the appropriate docker client. Below are direct links from the [Docker releases page][docker-releases].
  * [Docker Client - 1.6.1](https://get.docker.com/builds/Windows/x86_64/docker-1.6.1.exe)
3. Open Windows Explorer and browse to the Docker Tools extension directory, e.g. `C:\Program Files (x86)\Microsoft Visual Studio 14.0\Common7\IDE\Extensions\Microsoft\Windows Azure Tools\Docker`.

        cd /d "%VS140ComnTools%\..\IDE\Extensions\Microsoft\Windows Azure Tools\Docker"
4. Create a new directory and name it after the downloaded Docker client version.

        mkdir 1.6.1
5. Move the docker client that you downloaded in step 2 into this directory and rename it to `docker.exe`.
6. Now, in the Visual Studio Solution Explorer browse to the Properties/PublishProfiles directory. Open the PowerShell script for your publish profile, e.g. MyProfile-publish.ps1.
7. Go to the `Ensure-DockerCommand` function and update it to use the new directory you created.

    ```bash
    # Original Line
    # This uses the default docker.exe client in the extension directory
    $dockerExtensionPath = Join-Path $vsInstallPath ".\Extensions\Microsoft\Windows Azure Tools\Docker"
    # Updated Line
    # This uses the alternate docker client located in the version directory
    $dockerExtensionPath = Join-Path $vsInstallPath ".\Extensions\Microsoft\Windows Azure Tools\Docker\1.6.1"
    ```

Now Visual Studio will use the appropriate Docker client when publishing this
project and other projects will continue to use the default docker client provided by the extension.

[docker-releases]: https://github.com/docker/docker/releases

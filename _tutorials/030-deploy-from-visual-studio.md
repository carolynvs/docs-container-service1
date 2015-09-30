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
and learn how to deploy your application to Rackspace Container Service using Visual Studio.

* [Requirements](#Requirements)
* [Create an ASP.NET 5 Application](#CreateApp)
* [Deploy to Docker](#Publish)
  * [Publish from Visual Studio](#VisualStudioPublish)
  * [Publish from MS Build](#MSBuildPublish)
* [Troubleshooting](#Troubleshooting)

## <a name="Requirements"></a> Requirements ##

* [Visual Studio 2015 Community Edition or Higher][get-vs]
* [Microsoft ASP.NET and Web Tools 2015][vs-webtools-plugin] Extension
* [Visual Studio 2015 Tools for Docker][vs-docker-plugin] Extension <br/>
*This requires ASP.NET and Web Tools*
* Your [Rackspace Cluster Credentials][get-cluster-creds]

[vs-webtools-plugin]: https://www.microsoft.com/en-us/download/details.aspx?id=48738
[vs-docker-plugin]: https://visualstudiogallery.msdn.microsoft.com/0f5b2caa-ea00-41c8-b8a2-058c7da0b3e4
[get-vs]: https://www.visualstudio.com/downloads/download-visual-studio-vs
[get-cluster-creds]: /
<!-- TODO: Link to instructions -->

## <a name="CreateApp"></a> Create an ASP.NET 5 Application ##
First create an ASP.NET 5 application which we can deploy to Linux via a Docker container. This is possible because ASP.NET 5 now supports running on Linux with the [Kestrel web server][kestrel]. To learn more, checkout Microsoft's [ASP.NET 5 tutorial][aspnet-tutorial].

1. Open Visual Studio, go to File &rarr; New &rarr; Project, then select the `ASP.NET Web Application` template.

    ![New Project Window]({% asset_path 030-deploy-from-visual-studio/new-project.png %})

2. Verify that the project builds successfully.

3. Edit `project.json` and add kestrel as a dependency.

    ```json
    "dependencies": {
      "Microsoft.AspNet.Server.Kestrel": "1.0.0-beta7"
    }
    ```

4. Edit `project.json` and update the `kestrel` command to specify the server argument. This will allow you to use the hosting.ini file with both IIS and Kestrel.

        "commands": {
          "web": "Microsoft.AspNet.Hosting --server Microsoft.AspNet.Server.WebListener --config hosting.ini",
          "kestrel": "Microsoft.AspNet.Hosting --server Microsoft.AspNet.Server.Kestrel --config hosting.ini"
        }

5. Edit `hosting.ini` and specify the port on which your application should be hosted, e.g. 5000.

        server.urls=http://localhost:5000

6. Edit your project properties and select Kestrel as the default debug profile. Set the Launch URL to the same url in your `hosting.ini` file.
![Project Properties]({% asset_path 030-deploy-from-visual-studio/project-properties.png %})

7. Verify that your application runs as expected before attempting to publish to a Docker container. Select Debug &rarr; Start Debugging to start the Kestrel web server and open your web application in the browser.
![Web Page Screenshot]({% asset_path 030-deploy-from-visual-studio/app-screenshot.png %})

[kestrel]: https://github.com/aspnet/KestrelHttpServer
[aspnet-tutorial]: http://docs.asp.net/en/latest/tutorials/your-first-aspnet-application.html#create-a-new-asp-net-5-project

## <a name="Publish"></a> Publish ##
Now that you have an ASP.NET 5 application which works locally, the next step is
to publish it to a Docker container. You should have already signed-up for Rackspace Container Service,
downloaded and unzipped your cluster's credentials to your local machine.

See [Managing your cluster credentials][get-cluster-creds] for detailed instructions.

### <a name="VisualStudioPublish"></a> Publish from Visual Studio ###
Before you can automate publishing your application,
you first must use Visual Studio to setup and configure your publish profile.
This profile will contain the necessary credentials and settings to publish your
application to Rackspace Container Service.

One of the files downloaded is docker.env which contains your your Docker Host,
the ip address and port on which your Docker cluster is running, e.g. `DOCKER_HOST=tcp://104.130.0.124:2376`.
Remember this value as it will be required later.

1. Right click on your project in the Solution Explorer and select Publish.
2. On the Profile page, create a new custom profile and name it.
![Name Publish Profile]({% asset_path 030-deploy-from-visual-studio/publish-target.png %})
3. Click on Docker Containers for your publish target. In the Select Docker Virtual Machine prompt check Custom Docker Host and click OK.
![Custom Docker Host]({% asset_path 030-deploy-from-visual-studio/publish-host.png %})
5. On the Connection Settings page, enter the following settings:
    * Server Url: {your-docker-host}, e.g. `tcp://104.130.0.124:2376`. See the instructions to locate this value at the beginning of this section.
    * Image Name: {your-docker-image}, e.g. `MyProject`.
    * Dockerfile: Keep the default value.
    * Host Port: `80`
    * Container Port: `5000`. This should match the port specified in your hosting.ini file.
    * Auth Options: --tls --tlscert="{path-to-rackspace-credentials}\cert.pem" --tlskey="{path-to-rackspace-credentials}\key.pem", e.g. `--tls --tlscert="Y:\myproj\cert.pem" --tlskey="Y:\myproj\key.pem"`. The key and cert paths are located in the credentials you downloaded previously.
![Connection Settings]({% asset_path 030-deploy-from-visual-studio/publish-connection.png %})
6. Click Validate Connection to verify your settings.
7. Click Publish to deploy your application to the Rackspace Container Service. The deploy progress and any output is displayed in the Azure App Service Activity window.

### <a name="MSBuildPublish"></a> Publish from MS Build ###
*If you have not already generated a publish profile in Visual Studio, go back to the previous section and do that first.*


## <a name="Troubleshooting"></a> Troubleshooting ##

* [unable to find a node with port 80 available](#PortsFull)
* [client and server don't have same version](#DockerVersions)


### <a name="PortsFull"></a>unable to find a node with port 80 available ###

`Error response from daemon: unable to find a node with port X available`

This usually happens after a few failed deployments. It indicates that your Docker
cluster has run out of free nodes from which it can host the desired port. For example,
your application is exposed externally on port 80 and your cluster has two host nodes. After two failed deployments, both of your nodes is hosting a container with port 80 taken. When you try to deploy a third time, there are no nodes available with port 80 free.

To resolve this, use the Docker client to identity the nodes containing the failed
deployments and remove the containers. You can identify them in the docker ps output
by looking for containers which are running the kestrel command.

    $ docker.exe ps -a

    CONTAINER ID        IMAGE               COMMAND
    4a91d3218d21        myproject:latest    "./kestrel"
    f1e00223eda3        myproject:latest    "./kestrel"

    $ docker.exe rm 4a91d3218d21 f1e00223eda3

### <a name="DockerVersions"></a>client and server don't have same version ###

`Error response from daemon: client and server don't have same version (client : X, server: Y)`

In general when using Docker, the client and server versions must match.
The Visual Studio Tools for Docker extension first tries to use the docker client
that is accessible via the current PATH environment variable and then falls back to
using the docker client in the extension installation directory. This means that when using a custom
docker host, that you may need to tweak your publish profile to use a different version of the docker client.

If you see an error message like the one above, follow the steps below to
explicitly specify which version of the docker client to use.

1. Check if the Docker client is in your PATH. If it is, remove it and reboot. If that doesn't correct the error, move on to step 2.
2. Identify the version of Docker used by the server from the error message.
3. Download the appropriate docker client. Below are direct links from the [Docker releases page][docker-releases].
  * [Docker Client - 1.6.1](https://get.docker.com/builds/Windows/x86_64/docker-1.6.1.exe)
4. Open Windows Explorer and browse to the Docker Tools extension directory, e.g. `C:\Program Files (x86)\Microsoft Visual Studio 14.0\Common7\IDE\Extensions\Microsoft\Windows Azure Tools\Docker`.

        cd /d "%VS140ComnTools%\..\IDE\Extensions\Microsoft\Windows Azure Tools\Docker"
5. Create a new directory and name it after the downloaded Docker client version.

        mkdir 1.6.1
6. Move the docker client that you downloaded in step 2 into this directory and rename it to `docker.exe`.
7. Now, in the Visual Studio Solution Explorer browse to the Properties/PublishProfiles directory. Open the PowerShell script for your publish profile, e.g. MyProfile-publish.ps1.
8. Go to the `Ensure-DockerCommand` function and update it to use the new directory you created.

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

# DevOps as Code workshop

This workshop will teach you:

* How to start up the XL DevOps Platform with the devops-as-code features installed.
* How to install the XL CLI.
* How to import and deploy a Docker application with XL Deploy.
* How to import and run a pipeline with XL Release.

## Prerequisites

You'll need to have Docker installed on your machine before you begin:
* Mac: https://docs.docker.com/docker-for-mac/
* Windows: https://docs.docker.com/docker-for-windows/
* Linux: Refer to the instructions for your Linux distribution on how to install Docker

# Get the workshop

1) Download and extract the workshop zip:
```
$ curl -LO https://github.com/xebialabs/devops-as-code-demo/archive/workshop-2.zip
$ unzip workshop-2.zip
$ cd devops-as-code-demo-workshop-2
```

# Start up the XL DevOps Platform

1) If you are already running XL Deploy or XL Release on your local machine, please stop them.

2) If you are running the workshop on Windows, execute the following commmand to be able to run the Docker Compose file:

```
> set COMPOSE_CONVERT_WINDOWS_PATHS=1
```

For more information on this environment variable, read [the documentation for Docker Compose](https://docs.docker.com/compose/reference/envvars/#compose_convert_windows_paths)

3) Start up the XL DevOps Platform:

```
$ docker-compose up --build
```

4) Wait for XL Deploy and XL Release to have started up. This will have occurred when the following line is shown in the logs:
```
devops-as-code-demo-workshop-2_xl-cli_1 exited with code 0
```

5) Open the XL Deploy GUI at http://localhost:4516/ and login with the username `admin` and password `admin`. Verify that the about box reports the version to be **8.5.0-alpha.3**.

6) Open the XL Release GUI at http://localhost:5516/ and login with the username `admin` and password `admin`. Verify that the about box reports the version to be **8.5.0-alpha.3**.

# Install the XL CLI

1) Open a new terminal window and install the XL command line client:

## Mac
```
$ curl -LO https://s3.amazonaws.com/xl-cli/bin/8.5.0-alpha.5/darwin-amd64/xl
$ chmod +x xl
$ sudo mv xl /usr/local/bin
```

## Linux
```
$ curl -LO https://s3.amazonaws.com/xl-cli/bin/8.5.0-alpha.5/linux-amd64/xl
$ chmod +x xl
$ sudo mv xl /usr/local/bin
```

## Windows
Download https://s3.amazonaws.com/xl-cli/bin/8.5.0-alpha.5/windows-amd64/xl.exe
and place it somewhere on your `%PATH%`


2) Test the CLI by running the following the following command:
```
$ xl help
```

The output should look something like this:
```
The xl command line tool provides a fast and straightforward method for provisioning
XL Release and XL Deploy with YAML files. The files can include items like
releases, pipelines, applications and target environments.

Usage:
  xl [command]

Available Commands:
  apply       Apply configuration changes
  help        Help about any command

Flags:
      --config string   config file (default: $HOME/.xebialabs/config.yaml)
  -h, --help            help for xl
  -v, --verbose         verbose output

Use "xl [command] --help" for more information about a command.
```

# Exercise 1: Review the XL DevOps Platform running on Docker

When the XL DevOps Platform was started up by [the Docker Compose file](https://github.com/xebialabs/devops-as-code-demo/blob/workshop-2/docker-compose.yaml), four containers were started:
* `xl-deploy` runs XL Deploy.
* `xl-release` runs XL Release.
* `dockerproxy` is where XL Deploy will deploy to. It's a proxy for the Docker engine on your local machine, the same that runs XL Release and XL Deploy..
* `xl-cli` runs the XL CLI to apply the [`configure-xl-devops-platform.yaml`](https://github.com/xebialabs/devops-as-code-demo/blob/workshop-2/config/configure-xl-devops-platform.yaml) YAML file. This XL YAML file adds two configurations:
  * It adds an XL Deploy configuration to XL Release so that the latter can find the former.
  * It adds a `docker.Engine` configuration to XL Deploy so that XL Deploy can deploy to the Docker engine (via the Docker proxy).

1) Open the XL Deploy GUI, find the **local-docker** entry in the Infrastructure tree and run the **Check Connection** control task.


# Exercise 2: Set up the environment

Now that we have the XL DevOps Platform up and runnning and connected to our local Docker instance, let's create an environment that contains it.

1) Open a new terminal window and go to the directory where you checked out the `devops-as-code-workshop` repository.

2) Create the environment that contains the Docker engine by applying its XL YAML file:
```
$ xl apply -f workshop/exercise-2/docker-environment.yaml
```
3) Open the XL Deploy GUI and review the environment that you just created.

# Exercise 3: Import a simple package and deploy it

Let's try and deploy something to our local Docker instance with XL Deploy. We'll start with a single container; the backend part of the REST-o-rant application. It's called **rest-o-rant-api** because it serves up the API.

1) Import the REST-o-rant-api package:

```
$ xl apply -f workshop/exercise-3/rest-o-rant-api-docker.yaml
```

2) Open the XL Deploy GUI and review version **1.0** of the **rest-o-rant-api-docker** application. Compare it with the contents of the XL YAML file that you applied in the previous step.

3) Deploy version **1.0** of the **rest-o-rant-api-docker** application to the **Local Docker Engine** environment.

4) When the deployment has finished, check whether the **rest-o-rant-api** container is running:
```
$ docker ps
```

The output should look something like this:
```
$ docker ps
CONTAINER ID        IMAGE                                           COMMAND                  CREATED             STATUS              PORTS                    NAMES
b0611ff9ffbb        xebialabsunsupported/rest-o-rant-api            "java -Djava.securit…"   11 seconds ago      Up 10 seconds       8080/tcp                 rest-o-rant-api
f80c2b00a88e        xebialabsunsupported/xl-release:8.5.0-alpha.2   "/opt/xebialabs/tini…"   3 days ago          Up About an hour    0.0.0.0:5516->5516/tcp   devops-as-code-demo_xl-release_1
a99c31ddd458        tecnativa/docker-socket-proxy:latest            "/docker-entrypoint.…"   3 days ago          Up About an hour    2375/tcp                 devops-as-code-demo_dockerproxy_1
68a5c6439540        xebialabsunsupported/xl-deploy:8.5.0-alpha.2    "/opt/xebialabs/tini…"   3 days ago          Up About an hour    0.0.0.0:4516->4516/tcp   devops-as-code-demo_xl-deploy_1
```

# Exercise 4: Deploy a (slightly) more complex package

Serving a REST API is all nice and dandy, but it's pretty useless without a UI. So let's deploy the **rest-o-rant-web** container. Because it needs to access the **rest-o-rant-api** application to get its data, we'll also need to define a Docker network to allow the two containers to communicate. Version **1.1** of the **rest-o-rant-api** application will define that network.

1) Import the new version of the **rest-o-rant-api-docker** package, as well as the **rest-o-rant-web-docker** package:
```
$ xl apply -f workshop/exercise-4/rest-o-rant-docker.yaml
```

2) Open the XL Deploy GUI and review the two packages that you just imported and compare them with the XL YAML file. Notice the net **rest-o-rant-network** deployable in the **rest-o-rant-api-docker** package.

3) Deploy version **1.1** of the **rest-o-rant-api-docker** package to the **Local Docker Engine** environment.

4) Deploy version **1.0** of the **rest-o-rant-web-docker** package to the **Local Docker Engine** environment.

5) When the deployment has finished, open a new browser tab and access it at http://localhost/. You should see a text saying "Find the best restaurants near you!".
Type "Cow" in the search bar and click "Search" to find the "Old Red Cow" restaurant.

# Exercise 5: Import a pipeline

OK, we've deployed the application, but how do we get rid of it? Well, let's do that manually for one last time:

1) Undeploy the **rest-o-rant-web** and **rest-o-rant-api** applications from the **Local Docker Engine** environment.

But let's make sure that you don't forget next time that you run this workshop. Let's create a simple pipeline.

2) Import that REST-o-rant pipeline:
```
$ xl apply -f workshop/exercise-5/rest-o-rant-docker-pipeline.yaml
```

3) Open the XL Release GUI and review the **REST-o-rant on Docker** pipeline that you've just imported and compare it to the XL YAML file.

4) Start a new release from that template and follow the instructions.

# Exercise 6: Simplify the package and the pipeline by using dependencies

You'll have noticed that the pipeline has two separate steps to deploy and later undeploy the **rest-o-rant-api-docker** and the **rest-o-rant-web-docker** packages. If we make the frontend depend on the backend, we can simplify the pipeline _and_ ensure that the frontend is not deployed without the backend.

In this exercise we will make the necessary modification in the package and the pipeline and then teach you how to export them back to YAML.

1) In XL Deploy, open **Applications/rest-o-rant-web-docker/1.0** and add the following entry to the **Application Dependencies** map:

| Key | Value	|
|------------------------	|-----	|
| rest-o-rant-api-docker 	| 1.1 	|

and change **Undeploy Unused Dependencies** to `true`.

2) In XL Release, remove the **Deploy REST-o-rant application backend** task from the first phase and the **Undeploy REST-o-rant application backend** step from the last phase.

3) Run the pipeline to verify that the application dependencies and the updated pipeline function correctly. When the pipeline has completed, go to the XL Deploy UI and check that the **rest-o-rant-api-docker** application was undeployed.

4) Export the XL YAML files for the changes:

```
$ xl export -s xl-deploy -p Applications/rest-o-rant-web-docker -f workshop/exercise-6/rest-o-rant-web-docker-with-dependencies.yaml
$ xl export -s xl-release -p REST-o-rant -f workshop/exercise-6/rest-o-rant-pipeline-with-dependencies.yaml
```

5) Review the exported XL YAML files and compare them with the originals from exercises 4 and 5 respectively.

# Exercise 7: Export an XL YAML file with artifacts

The XL YAML format also allows you to use artifacts. To see what that looks like, let's export the PetClinic sample package:

1) In the XL Deploy UI, import the **PetClinic-ear/1.0** sample package from the XL Deploy server.

2) Export the XL YAML file for the package you just imported to see how to specify artifacts:
```
$ xl export -s xl-deploy -p Applications/PetClinic-ear/1.0 -f workshop/exercise-7/petlinic-ear.yaml
```

3) Review the exported XL YAML file and the artifact.

# Exercise 8: Shutting down the XL DevOps Platform

1) Shut down the XL DevOps Platform:

```
$ docker-compose down
```

Not only will this stop the XL DevOps Platform, it will also remove any data stored on it. Therefore you should make sure that all releases and deployments have finished and that you've undeployed any applications you've deployed with it before you shut down the XL DevOps Platform.

# Bonus exercise: putting it all together

OK, that was cool and all. But you had to run the `xl apply` command line four times to get everything into XL Deploy and XL Release. That's nice for a workshop, but not so nice for a demo.

1) Create a single XL YAML file that combines all the exercises we've done so that you can get the demo up and running with two simple commands:
```
$ docker-compose up
$ xl apply -f rest-o-rant-demo.yaml
```

**N.B. 1:** If the XL YAML file contains multiple documents (blocks), they will be executed in order.

**N.B. 2:** Make sure you've undeployed the REST-o-rant application before trying again. Otherwise you might run into issues with there being two Docker networks named **rest-o-rant-network**. If you do, roll back your deployment and remove both (by ID)

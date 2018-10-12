# Datadog Summit London 2019 Kubernetes Workshop

Welcome! In this workshop we're going to learn about Kubernetes.

## Pre-requisites

* MacOS (with [brew](https://brew.sh/) _or_ a common Linux distribution (64-bit)
* [VirtualBox](https://www.virtualbox.org/) preferred; other virtualisation techniques are possible but are **unsupported**.
* [Docker](https://www.docker.com/install/) (CE is fine)
* `git`

All of the above must have been recently updated. 👍

### Datadog account

In order to enjoy the full scope of this workshop, you'll want to [sign up for a free trial Datadog account](https://www.datadoghq.com/lpg6/); you probably don't want to use a pre-existing account, as we'll be playing around with lots of dummy data.

## Overview

In the first part of the workshop we'll install [Minikube](https://github.com/kubernetes/minikube).
> Minikube is a tool that makes it easy to run Kubernetes locally. Minikube runs a single-node Kubernetes cluster inside a VM on your laptop for users looking to try out Kubernetes or develop with it day-to-day.

In the second part of the workshop we'll deploy a basic application stack via Kubernetes. We'll also see how to deploy the Datadog Agent and explore some of the features of Datadog itself.

# Part one: Minikube and friends

The goal here is to get a functioning Kubernetes cluster up and running as efficiently as possible. We'll explore Minikube itself, take a first look at the Kubernetes CLI tool, and learn some key terminology and concepts.

## Install `minikube` and `kubectl`

(The following instructions are adapted from the [Minikube README](https://github.com/kubernetes/minikube/blob/master/README.md).)

Both the Minikube (`minikube`) and the Kubernetes (`kubectl`) binaries are self-contained and of moderate size.

#### MacOS

Brew is by far the most straightforward way to install the tooling required for this workshop, and Minikube is no exception.

```shell
$ brew cask install minikube
```

This will _also_ install the `kubernetes-cli` formula, which provides `kubectl`. Hooray!

#### Linux

Your distribution's package manager _may_ include Minikube; most likely, you will need to install it manually:

```shell
$ curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
$ sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

`kubectl` it can also be installed manually, though more [mature package manager support](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-binary-using-native-package-management) is available for many Linux flavours.

```shell
$ KUBECTL_STABLE=$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt) 
$ curl -LO https://storage.googleapis.com/kubernetes-release/release/$KUBECTL_STABLE/bin/linux/amd64/kubectl
$ sudo install kubectl /usr/local/bin/kubectl
```

In the case of manual installation, feel free to put the binaries anywhere you like - just don't forget to have them in your `$PATH` for ease of use.

### Test it out!

First, _refresh your shell session_ in order to make sure the paths and aliases are all set up. Next, test the binaries:

```
$ minikube version
minikube version: v0.30.0

$ kubectl version
Client Version: version.Info{Major:"1", Minor:"12", GitVersion:"v1.12.0", [...]
```

We just learned our first commands for both `minikube` and `kubectl` - congratulations!

## Our first cluster

Let's take a closer look at Minikube. The `status` command gives us a very brief status overview of the Minikube Kubernetes cluster:

```
$ minikube status
minikube:
cluster:
kubectl:
```

All of the status fields are black because we don't actually have a cluster yet, so let's fire up our first Minikube instance! This will download a smallish ISO, perform some configuration steps, then instantiate a virtual machine. (In other words: it will take a few minutes.)

```
$ minikube start
Starting local Kubernetes v1.10.0 cluster...
Starting VM...
Downloading Minikube ISO
 170.78 MB / 170.78 MB [============================================] 100.00% 0s
Getting VM IP address...
Moving files into cluster...
Downloading kubelet v1.10.0
Downloading kubeadm v1.10.0
Finished Downloading kubelet v1.10.0
Finished Downloading kubeadm v1.10.0
Setting up certs...
Connecting to cluster...
Setting up kubeconfig...
Starting cluster components...
Kubectl is now configured to use the cluster.
Loading cached images from config file.
```

Our status command is more interesting now:

```
$ minikube status
minikube: Running
cluster: Running
kubectl: Correctly Configured: pointing to minikube-vm at 192.168.99.100
```

(The IP address may vary depending on the local network.)

## Our first application

Let's set up our first application: a sort of "Hello, World!" for Kubernetes. This introduces a new command: `run`.

```
$ kubectl run hello-minikube --image=k8s.gcr.io/echoserver:1.4 --port=8080
kubectl run --generator=deployment/apps.v1beta1 is DEPRECATED and will be removed in a future version. Use kubectl create instead.
deployment.apps/hello-minikube created
```

There's a lot of information packed into those lines, so let's break it down. First, the command line:

```
          command            container image     container name  port to expose
          ⬇                 ⬇                  ⬇               ⬇
$ kubectl run hello-minikube --image=k8s.gcr.io/echoserver:1.4 --port=8080
  ⬆          ⬆                      ⬆                    ⬆
  CLI tool    name of instance       source                container version
```

The deprecation warning is relatively new and can safely be ignored for now (we'll touch on it later).

Lastly, the result of the command:

```
                resource name
                ⬇
deployment.apps/hello-minikube created
⬆                             ⬆
resource type                  action
```

There are a number of actions and resource types in Kubernetes. For now, just get a feel for the output format, as this is what using `kubectl` feels like: commands with options, and the resulting structured output.

Now we'll need to a queryable endpoint available - in the Kubernetes parlance, we'll "expose a deployment":

```
$ kubectl expose deployment hello-minikube --type=NodePort
service/hello-minikube exposed
```

Breaking that down:

```
          command           resource name
          ⬇                ⬇
$ kubectl expose deployment hello-minikube --type=NodePort
  ⬆             ⬆                         ⬆
  CLI tool       resource type              service type
```

With the notable exception of "service type", the contents of that invocation are relatively straightforward (don't worry about that last option for now).

### Ask a question…

Next we'll learn about a fairly common command: `get`. As the name implies, `get` allows you to make queries regarding Kubernetes _resources_.

```
$ kubectl get pod
NAME                             READY   STATUS    RESTARTS   AGE
hello-minikube-6c47c66d8-95vxs   1/1     Running   0          1m
```

### …get an answer

Instead of explaining the previous command, let's take a look at the help pages available directly in `kubectl`. To get help about pretty much anything, just prepend "help" to your command invocation. For example:

```
$ kubectl help get pod
Display one or many resources

Prints a table of the most important information about the specified resources. [...]
```

The output has been truncated here, but you get the idea. And what about that service type from the previous invocation? Try getting some help for that, too!

### Test it out!

We're going to query the little echoserver application that we've deployed, but first, we need a URL. This URL is generated when we exposed the resource previously.

```
$ minikube service hello-minikube --url
http://192.168.99.100:32521
```

We can also leverage some shell tricks:
```
$ curl $(minikube service hello-minikube --url)
CLIENT VALUES:
client_address=172.17.0.1
command=GET
real path=/
query=nil
request_version=1.1
request_uri=http://192.168.99.100:8080/

[...]
```

It works! What a glorious, modern age we live in.

### And scene.

As this is just a toy application, there isn't much more for us to do, so let's learn about another important command: `delete`.

```
$ kubectl delete service hello-minikube
service "hello-minikube" deleted
$ kubectl delete deployment hello-minikube
deployment "hello-minikube" deleted
```

If you want to stop now and call it a day, you can also stop Minicube too - but _don't do this_ if you plan to keep going with the next section as it'll just slow us down. 🚀

```
$ minikube stop     # Info only! Don't do this now!
Stopping local Kubernetes cluster...
Machine stopped.
```

# Part two: Deploying an application with Kubernetes

(This section is based on [Z's Minikube-Datadog tutorial](https://github.com/ziquanmiao/minikube_datadog).)

The goal here is to use what we've just learned to deploy an actual application stack, consisting of a simple Flask-based web app, a Postgres database, and the Datadog agent for monitoring _everything_. You'll learn more about Kubernetes and `kubectl` as well as how to use the Datadog product with Kubernetes.

## A wild Docker appears!

In order to keep things simple, we're going to roll Docker images for our application and database, respectively. This next part is _very important_ so take note: we're going to use **Minikube's Docker environment**, _not_ the default environment on your machine.
```
$ eval $(minikube docker-env)
```

This just sets some environment variables (go ahead and run the encapsulated command yourself to see), but without these variables in our session, we'll run into confusing problems later on.

Now it's time to build those images. 


Store the API key in a kubernetes secret so its not directly in the deployment code
```
kubectl create secret generic datadog-api --from-literal=token=___INSERT_API_KEY_HERE
```
The key is then referenced in the Daemon file [here](https://github.com/ziquanmiao/minikube_datadog/blob/8b48b62278dc52f4f8d2834bc6df3ae8f955acaf/agent_daemon.yaml#L28-L32)

## build images

Then, build the images based off the provided Dockerfiles
```
docker build -t sample_flask:007 ./flask_app/
docker build -t sample_postgres:007 ./postgres/
```

## deploy things

Deploy the postgres container
```
kubectl create -f postgres_deployment.yaml
```

Deploy the application container and turn it into a service
Also create a configMap for the logs product
```
kubectl create -f app_deployment.yaml
```



Deploy the Datadog agent container

```
kubectl create -f agent_daemon.yaml
```

Deploy a nonfunction pause container to demonstrate Datadog [AutoDiscovery](https://docs.datadoghq.com/agent/autodiscovery/) via a simple HTTP check against www.google.com
```
kubectl create -f pause.yaml
```

Deploy kubernetes state files to demonstrate [kubernetes_state check](https://docs.datadoghq.com/integrations/kubernetes/#setup-kubernetes-state)

```
kubectl create -f kubernetes
```

And we are done!

# Use the Flask App

The Flask App offers 3 endpoints that returns some text `FLASK_SERVICE_IP:5000/`, `FLASK_SERVICE_IP:5000/query`, `FLASK_SERVICE_IP:5000/bad`

Run ```kubectl get services``` to find the [FLASK_SERVICE_IP](https://cl.ly/a344b20d5481) address of the flask application service

You can then access the endpoints within the minikube vm:
```
minikube ssh
```

then hit one of the following:
```
curl FLASK_SERVICE_IP:5000/
curl FLASK_SERVICE_IP:5000/query
curl FLASK_SERVICE_IP:5000/bad
```
to see the Flask application at work

# Some points of interest

The Datadog agent container should now be deployed and is acting as a collector and middleman between the services and Datadog's backend. Through actions -- curling the endpoints -- and doing nothing, metrics will be generated and directed to the corresponding Datadog Account based off your supplied API key

Below is a quick discussion on some points of interest

## Infrastructure Product
This part pertains to the ingestion of timeseries data, status checks, and events. 

By deploying the agent referencing the [Datadog Container Image](https://github.com/ziquanmiao/minikube_datadog/blob/ba94f6072fbfccbaaf8595020690df9b2f6ebdfb/agent_daemon.yaml#L13) in the agent_daemon.yaml file, the check automatically comes prepackaged with system level (CPU, Mem, IO, Disk), [Kubernetes](https://docs.datadoghq.com/agent/kubernetes/), and [Docker](https://docs.datadoghq.com/integrations/docker_daemon/) level checks.

The gist of the setup portion is:
1. Deploy the agent daemon the proper environment variables, volume and volumeMount arguments
2. Deploy relevant applications with annotations
3. Validate metrics go to agent and ends up in our application

### System Metric Requirements

[volumeMount](https://github.com/ziquanmiao/minikube_datadog/blob/ba94f6072fbfccbaaf8595020690df9b2f6ebdfb/agent_daemon.yaml#L65-L67) and [volume](https://github.com/ziquanmiao/minikube_datadog/blob/ba94f6072fbfccbaaf8595020690df9b2f6ebdfb/agent_daemon.yaml#L93-L95) for the proc directory is required from the host level

In the Datadog web application you can reference the [host map](datadoghq.com/infrastructure/map), and filter on the particular hostname to see what is going on.

### Kubernetes/Docker
docker.sock and cgroup [volumeMounts](https://github.com/ziquanmiao/minikube_datadog/blob/ba94f6072fbfccbaaf8595020690df9b2f6ebdfb/agent_daemon.yaml#L63-L70) and [volumes](https://github.com/ziquanmiao/minikube_datadog/blob/ba94f6072fbfccbaaf8595020690df9b2f6ebdfb/agent_daemon.yaml#L90-L98) are required to be attached in the daemonset

### Autodiscovery
Application/Service Pods and Containers go up and down. The Datadog agent traditionally requires a modification of the hardcoded host/port values of corresponding configuration files (example with [postgres](https://github.com/DataDog/integrations-core/blob/fd4414ed3d85a6ad835f6440f4bd091a4cf1a0f2/postgres/datadog_checks/postgres/data/conf.yaml.example#L4-L5)) and an agent restart to collect Data for installed software.

Rather than having a Mechanical Turk sit on standy ready to make the changes, the containerized accommodates makes this process automagic using [Autodiscovery](https://docs.datadoghq.com/agent/autodiscovery/) where the agent has the capability to monitor the annotations of deployments and automatically establish checks as pods come and go.

To set up autodiscovery, you will need to set up [volumes](https://github.com/ziquanmiao/minikube_datadog/blob/67c0d53f3d9fa2c55dc47986c1ab82625445a70e/agent_daemon.yaml#L109-L111) and [mountPaths](https://github.com/ziquanmiao/minikube_datadog/blob/67c0d53f3d9fa2c55dc47986c1ab82625445a70e/agent_daemon.yaml#L81-L82) to put the ethemeral configuration files

#### Postgres Example
Autodiscovery of the postgres pod in this container is straightforward, [simply annotation](https://github.com/ziquanmiao/minikube_datadog/blob/ba94f6072fbfccbaaf8595020690df9b2f6ebdfb/postgres_deployment.yaml#L14-L17) in the configuration file by adding in the typical check sections required.

Note the annotation arguments [here](https://cl.ly/d77e73e9786d) must be identical for the agent to properly connect to the container.

### Prometheus
Many services (like kubernetes itself) utilizes Prometheus as an enhancement to reveal custom internal metrics specific to the service. 

You can see what the structure of the prometheus metrics look like by running:

```
minikube ssh
curl localhost:10255/metrics
```

Datadog has the innate capability to read the log structure of prometheus produced metrics and turn it into custom metrics.
The scope of this repo doesn't really touch on it too much, but collecting prometheus metrics can simply be done via annotations done at the deployment level as seen [here](https://github.com/ziquanmiao/minikube_datadog/blob/67c0d53f3d9fa2c55dc47986c1ab82625445a70e/app_deployment.yaml#L15-L17) -- note this example fails on purpose so you can see what an error looks like in agent status

Read more about it via our [documentation](https://docs.datadoghq.com/agent/prometheus/)

## Live Processes/Containers
[Live Process/Container Monitoring](https://www.datadoghq.com/blog/live-process-monitoring/) is the capability to get container and process level granularity for all monitored systems.
This feature provides not only standard system level metrics at the process/container level, but also on the initial run commands used to set up the process/container.

Simply add [DD_PROCESS_AGENT_ENABLED](https://github.com/ziquanmiao/minikube_datadog/blob/ba94f6072fbfccbaaf8595020690df9b2f6ebdfb/agent_daemon.yaml#L38-L39) env variable in the daemonset to turn on this feature

### Requirements
Sometimes passwords are revealed in the initial run commands, the agent comes equipped with [passwd](https://github.com/ziquanmiao/minikube_datadog/blob/ba94f6072fbfccbaaf8595020690df9b2f6ebdfb/agent_daemon.yaml#L72-L74) to remove a [standard set of arguments](https://docs.datadoghq.com/graphing/infrastructure/process/#process-arguments-scrubbing)

We again need [docker.sock](https://docs.datadoghq.com/graphing/infrastructure/process/#kubernetes-daemonset) to get container information.

### Validation

run ``` kubectl get pods``` to get the pod name of the agent container.

run ```kubectl exec -it POD_NAME bash ``` to hop into the container

run ```agent status``` to see the status summary and look for the integrations section to see agent is collecting metrics

run ``` cat /var/log/datadog/agent.log``` to see logs pertaining to the agent

## APM Tracing
The same agent that handles infrastructure metrics can also accommodate receiving [Trace Data](https://www.datadoghq.com/blog/tag/apm/) from a designated [APM module](datadoghq.com/apm/docs) -- these modules sit on top of your applications and forward [payloads](https://docs.datadoghq.com/api/?lang=bash#send-traces) to a local Datadog agent to middle man to our backend.

Applications are spinning up in pods and we need payloads being fired to the sidecar agent pod. In this example, we set up a route between the pods with a port going through the host level.

### Requirements

#### From the agent daemon side
Enable agent to receive traces from the Agent deployment side via [environment variables](https://github.com/ziquanmiao/minikube_datadog/blob/ba94f6072fbfccbaaf8595020690df9b2f6ebdfb/agent_daemon.yaml#L26-L27)

Create a [port connection to host](https://github.com/ziquanmiao/minikube_datadog/blob/ba94f6072fbfccbaaf8595020690df9b2f6ebdfb/agent_daemon.yaml#L21-L24) via the 8126 port

#### From the application Side
Provide the deploy file with a [link to the host level](https://github.com/ziquanmiao/minikube_datadog/blob/ba94f6072fbfccbaaf8595020690df9b2f6ebdfb/app_deployment.yaml#L27-L32) for port 8126 via environmental variables, so that applications can reference the host/port values to fire traces to

##### Flask specific
Have the [ddtrace module](https://github.com/ziquanmiao/minikube_datadog/blob/ba94f6072fbfccbaaf8595020690df9b2f6ebdfb/flask_app/requirements.txt#L2)

In the app.py code, [import ddtrace module](https://github.com/ziquanmiao/minikube_datadog/blob/ba94f6072fbfccbaaf8595020690df9b2f6ebdfb/flask_app/app.py#L18-L25) and patch both [sqlalchemy](https://github.com/ziquanmiao/minikube_datadog/blob/ba94f6072fbfccbaaf8595020690df9b2f6ebdfb/flask_app/app.py#L27) and the [Flask app object](https://github.com/ziquanmiao/minikube_datadog/blob/ba94f6072fbfccbaaf8595020690df9b2f6ebdfb/flask_app/app.py#L37).

**Note**: the trace module is an implementation as all modules are. If certain spans are not being captured, you can always [hardcode](https://github.com/ziquanmiao/minikube_datadog/blob/ba94f6072fbfccbaaf8595020690df9b2f6ebdfb/flask_app/app.py#L102) them in.

### Validation

#### Agent Side
run ``` kubectl get pods``` to get the pod name of the agent container.

run ```kubectl exec -it POD_NAME bash ``` to hop into the container

run ```agent status``` to see the status summary and look for the tracing section to see agent has tracing turned on

run ``` cat /var/log/datadog/trace-agent.log``` to see logs pertaining to the trace agent

#### Datadog Side

head over to [Trace Services Page](datadoghq.com/apm/services) and look for your service level metrics and traces!

## Logs

The same agent polling for metrics periodically for infrastructure metrics, live process metrics, and middle manning trace transcations can also be set up to tail log instances.

Simply turn on [DD_LOGS_ENABLED](https://github.com/ziquanmiao/minikube_datadog/blob/67c0d53f3d9fa2c55dc47986c1ab82625445a70e/agent_daemon.yaml#L44-L45) via the environmental variable in the agent daemon file.

### Tailing Flask logs via Config Maps

Tailing logs from flask is pretty easy using kubernetes config maps.

#### Agent Side
Simply set up the [mountPath directory](https://github.com/ziquanmiao/minikube_datadog/blob/67c0d53f3d9fa2c55dc47986c1ab82625445a70e/agent_daemon.yaml#L79-L80) connected via the host and the [volume](https://github.com/ziquanmiao/minikube_datadog/blob/67c0d53f3d9fa2c55dc47986c1ab82625445a70e/agent_daemon.yaml#L106-L108) in the agent daemon

#### Flask side
Set up the corresponding mounts for [volume and mountPath](https://github.com/ziquanmiao/minikube_datadog/blob/67c0d53f3d9fa2c55dc47986c1ab82625445a70e/app_deployment.yaml#L37-L43) so we can connect the flask pod to the relevant agent pod via the host

In the app, set up the app level [logging configurations](https://github.com/ziquanmiao/minikube_datadog/blob/67c0d53f3d9fa2c55dc47986c1ab82625445a70e/flask_app/app.py#L55-L64) so logs are properly pushed to the right log file when routines are run ([example](https://github.com/ziquanmiao/minikube_datadog/blob/67c0d53f3d9fa2c55dc47986c1ab82625445a70e/flask_app/app.py#L73-L79))

### Validation

#### Agent Side
run ``` kubectl get pods``` to get the pod name of the agent container.

run ```kubectl exec -it POD_NAME bash ``` to hop into the container

run ```agent status``` to see the status summary and look for the logging section to see agent has logs turned on

run ``` cat /var/log/datadog/agent.log``` to see relevant log instances pertaining to the log agent

#### Datadog Side

Navigate to the [logs explorer page](datadoghq.com/logs) and look for flask logs!





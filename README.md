# Datadog Kubernetes Workshop (Datadog Summit London 2018 edition)

# https://v.gd/dd_k8s_workshop

Welcome! In this workshop we're going to learn about Kubernetes and Datadog.

## Pre-requisites

* MacOS (with [brew](https://brew.sh/) _or_ a common Linux distribution (64-bit)
* [VirtualBox](https://www.virtualbox.org/) preferred; other virtualisation techniques are possible but are **unsupported**.
* [Docker](https://www.docker.com/install/) (CE is fine)
* `git`

All of the above must have been recently updated. 👍

### Datadog account

In order to enjoy the full scope of this workshop, you'll want to [sign up for a free trial Datadog account](https://www.datadoghq.com/lpg6/); you probably don't want to use a pre-existing account, as we'll be playing around with lots of dummy data.

## Overview

In the first part of the workshop, we'll install [Minikube](https://github.com/kubernetes/minikube).
> Minikube is a tool that makes it easy to run Kubernetes locally. Minikube runs a single-node Kubernetes cluster inside a VM on your laptop for users looking to try out Kubernetes or develop with it day-to-day.

In the second part of the workshop, we'll deploy a basic application stack via Kubernetes. We'll also see how to deploy the Datadog Agent and explore some of the features of Datadog itself.

In the third and final part of the workshop, we'll see how to gain knowledge and insight into the Kubernetes cluster via Datadog.

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

Let's set up our first application: a sort of "Hello, World!" for Kubernetes. This introduces a new command: `run`. Before we go any further, it's important to note that `run` is great for testing things locally, but really isn't meant for production use - we're just using it here because it's expedient.

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

#### One quick thing

If you haven't already, now is the time to [clone this repo locally](https://github.com/phrawzty/datadog_k8s_workshop), as we'll be using it extensively going forward.

## A wild Docker appears!

In order to keep things simple, we're going to roll Docker images for our application and database, respectively. This next part is _very important_ so take note: we're going to use **Minikube's Docker environment**, _not_ the default environment on your machine.
```
$ eval $(minikube docker-env)
```

This just sets some environment variables (go ahead and run the encapsulated command yourself to see), but without these variables in our session, we'll run into confusing problems later on.

Now it's time to build those images. This will take a while, so while these images are building, open a _new_ terminal session and _move on_ to the next steps.

```
$ cd datadog_k8s_workshop
$ docker build -t sample_flask:007 ./flask_app/ && docker build -t sample_postgres:007 ./postgres/
[...]
Successfully tagged sample_flask:007
[...]
```

## Kubernetes can keep a secret

We're going to explore a neat feature of Kubernetes: secrets. Secrets are a native resource that can be queried and used within managed environments. They are especially useful for things like login credentials, access tokens, and keys. In this case, we're going to insert the Datadog API key as a secret so that it can be used later on.

Make sure you're logged into the temporary workshop Datadog account that you created previously (see: [Pre-requisites](#pre-requisites)), then use the [API key](https://app.datadoghq.com/account/settings#api) in the following invocation:

```
$ kubectl create secret generic datadog-api --from-literal=token=__INSERT_API_KEY_HERE__
```

There are a few different types of secrets (see `kubectl help create secret` for more details), but in this case, all we need is a simple key/value pair. Note the syntax:

```
                                                           key
                                                           ⬇
$ kubectl create secret generic datadog-api --from-literal=token=__INSERT_API_KEY_HERE__
                                ⬆                               ⬆
                                name of the resource             value
```

The Kubernetes resource name and the key don't need to be the same. As an aside, generic-type keys can contain any number of key/value pairs (like a hash) - but we only need one for now.

## Configuration files

During part one of this workshop we encountered an invocation that produced a [deprecation notice](#our-first-application), (which we conveniently ignored). The warning implored us to use `create` instead of `run` for setting up a new deployment. As previously mentioned, `run` is fine for local testing - that's because it's shorthand for "take this container, apply all the defaults, and execute". In the Real World™️, `run` is _never used_.

The preferred format for new deployments is to build them from configuration files. In actual operating environmments, there are a tonne of flags, preferences, keys, and settings that need to be specified everytime a deployment is touched. All of that configuration data must be captured, tracked, and revisioned in configuration files, which we'll now encounter for the first time.

### Our first configuration file

Let's take a look at the most simple configuration file in this repository: `postgres_deployment.yaml`. There's a fair amount going on here, so let's concentrate on some key elements.

```yaml
kind: Deployment
```

`kind` defines the resource type. There are two instances of `kind` in this configuration file, which indicates that two resources are being configured. Since this is YAML, the `---` string is used to separate the blocks as well.

```yaml
spec:
  replicas: 1
```

That defines the number of replicas for the deployment. Neat!

```yaml
spec:
  template:
    spec:
      containers:
        [...]
```

In the `Deployment` kind, this section should be familiar from the world of Docker. It defines the container-specific information, including image name, port, and so forth.

```yaml
spec:
  ports:
```

In the `Service` kind, this defines what port to expose for the service.

### More configuration files!

We're going to be deploying a bunch of different resources, and from a learning perspective it's definitely worth stepping through each line of each file; however, for the same of this workshop, let's just skip to the good stuff.

In `app_deployment.yaml`, note the container spec section, as it contains a number of environment variables which relate to the Datadog Agent. Note also the `ConfigMap` kind, which sets up a log location that will be useful to us later.

In `agent_daemon.yaml`, look for the `DD_API_KEY` string. This is where the secret that we defined previously will be used. Instead of having to hardcode this into the configuration file, we can leverage Kubernetes' internal datastore for great success. 🏆 While you're there, note the other `DD_*` variables that have been set to `true` - these all activate various Datadog features that we'll explore later on.

While we won't be diving into it, take a look at the `kubernetes/` directory. It's full of configuration files - this is another way to organise resource definitions (as opposed to cramming them all into a single file).

### The "pause" container

The final configuration file is `pause.yaml`, which defines a Pause container. I'm going to be honest here: this is a bit of an arcane concept. Briefly stated, it's a container that sits around and waits (pauses) to clean things up. It also acts as a container to share namespaces (which it needs to do in order to act as a garbage collector). That shared namespace feature can be leveraged for other uses, such as auto-discovery.

If you want to learn more, [Ian Lewis' explanation](https://www.ianlewis.org/en/almighty-pause-container) is both approachable and comprehensive - for now, just bookmark that page and forge ahead.

## Back to the deployment

With the Docker image builds out of the way, let's move on to deployment - make sure that you're using the Minikube Docker [environment](#a-wild-docker-appears) though!

Deploy the Postgres database:

```
$ kubectl create -f postgres_deployment.yaml
deployment.apps/postgres created
service/postgres created
```

Deploy the Application:

```
$ kubectl create -f app_deployment.yaml
deployment.apps/flaskapp created
service/flaskapp created
configmap/cm-datadog-confd created
```

Deploy the Datadog Agent:

```
$ kubectl create -f agent_daemon.yaml
daemonset.extensions/datadog-agent created
```

Deploy the Pause container:

```
$ kubectl create -f pause.yaml
deployment.apps/datadog-pause-monitoring created
```

Deploy Kubernetes state configuration:

```
$ kubectl create -f kubernetes
clusterrolebinding.rbac.authorization.k8s.io/kube-state-metrics created
clusterrole.rbac.authorization.k8s.io/kube-state-metrics created
[...]
```

We're off to the races! 🐎

### Test it out!

Let's use `get` to query the services provided by our Kubernetes cluster:

```
$ kubectl get services
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
flaskapp     ClusterIP   10.109.187.61   <none>        5000/TCP   7m
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP    30m
postgres     ClusterIP   10.109.235.5    <none>        5432/TCP   8m
```

Note that none of these services have _external_ IPs. This will be relevant later on.

## Kubernetes Dashboard

Kubernetes has a built-in dashboard. Real talk: it's not a general-purpose monitoring tool - it's just for looking at the internals of the cluster - but if that's your use case, it's totally reasonable. In our case, we can load it up via Minikube:

```
$ minikube dashboard
Opening http://127.0.0.1:54437/api/v1/namespaces/kube-system/services/http:kubernetes-dashboard:/proxy/ in your default browser...
```

Spend some time exploring here - it's easy and sort of fun.

## Interacting with the application

Remember that `services` output from above, and how there weren't any external IPs? We didn't create any for the sake of simplicity; however, it does have the side effect of making it a little trickier to actually interact with these services. Luckily, they can be accessed from _within_ the cluster:

```
$ minikube ssh
```

This opens up a shell within the Minikube container itself. From here, the services can be accessed via their internal IPs. Let's take a look at the Flask app (note the IP for `flaskapp` in the services output):

```
$ curl -sI 10.109.187.61:5000 | grep HTTP
HTTP/1.0 200 OK

$ curl -sI 10.109.187.61:5000/query | grep HTTP
HTTP/1.0 200 OK

$ curl -sI 10.109.187.61:5000/bad | grep HTTP
HTTP/1.0 500 INTERNAL SERVER ERROR
```

The actual body of the responses aren't interesting - the important thing is to know that of the three, two will result in nice, clean responses, and the third will produce a traceable error.

# Part three: Datadog

The Datadog Agent is now deployed and is acting as a collector and middle-man between the Kubernetes-managed services and Datadog's backend. Infrastructural data, including both host and container statistics, are being monitored and collected by Datadog. Furthermore, all of the interactions with the application endpoints previously noted will generate metrics, traces, and all sorts of data.

## Testing the Agent

It can sometimes be useful to interact with the Datadog Agent directly. Since the Agent is deployed as a pod, the first step is to determine the pod name:

```
$ kubectl get pods
NAME                                       READY   STATUS    RESTARTS   AGE
datadog-agent-gdjv6                        1/1     Running   0          1h
datadog-pause-monitoring-7d8ff6bf4-tm9dd   1/1     Running   0          1h
flaskapp-d5c7cc988-qhbnn                   1/1     Running   0          2h
postgres-6555948bc6-l2lw5                  1/1     Running   0          2h
```

Then, you can open a shell session within that pod:

```
$ kubectl exec -it datadog-agent-gdjv6 bash
```

This will give you a `root` shell. From there, you can examine the Agent configuration, or query it directly via `agent status`. For more information, see the [Datadog Agent documentation](https://docs.datadoghq.com/agent/faq/agent-commands/).

## Have fun!

Go ahead and fire off a bunch of queries to the application endpoints. It might be a little laggy on the command line, but that's fine. Concentrate `/` and `/query`, but throw some `/bad` in there once in a while too. If you're feeling adventerous, you could even set up a little shell script to do this for you…

Once you've got a few minutes worth of data, head over to the Datadog application and start clicking around. Be sure to check out the following views:
* [Infrastructure](https://app.datadoghq.com/infrastructure), including the Host and Container maps.
* [Container (detail)](https://app.datadoghq.com/containers) 
* [Process list](https://app.datadoghq.com/process)
* [APM (tracing)](https://app.datadoghq.com/apm/services)
* [Logs](https://app.datadoghq.com/logs)

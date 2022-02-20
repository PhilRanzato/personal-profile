---
title: Microservices REST API Application
date: 2022-02-19T07:00:13+00:00
img: "/images/projects/cloud-native-app.webp"
description: "A Cloud native, microservices application with Golang backend and ReactJS frontend"
github: https://github.com/PhilRanzato/kubensure
featured: https://medium.com/@philranzato/deploying-a-cloud-native-kubernetes-dashboard-application-on-kubernetes-f9fafa9fb03d
---

## A Full-Stack application on Kubernetes

![ReactJS frontend + Golang backend REST API application in Kubernetes](https://miro.medium.com/max/1400/1*B1b7x5QgjAxRtz5Iyrp1-Q.png)

ReactJS frontend + Golang backend REST API application in Kubernetes

## **What is Kubernetes**

[Kubernetes](https://kubernetes.io) is an open-source container orchestration platform by Google. The purpose of this container management system is to ease the deliver of applications in a consistent and flexible way.

## **What are we going to do**

In this post we are going to develop a **REST API web application** by completely splitting the frontend from the backend.

This web application **will provide a simple **Kubernetes dashboard****, where we can see our running Kubernetes resources and run commands in our Pod’s containers.

I choose to develop the backend using [Golang](https://golang.org), from Google, while the frontend using [ReactJS](https://reactjs.org), from Facebook.

## **Why Golang and ReactJS**

Well, the answer is really straightforward, just to learn them.

****Go****, (or **Golang**), is an open source programming language that makes it easy to build simple, reliable, and efficient software. Plus, since Kubernetes itself is written in this language, I was always interested in learning it.

****React**** is a JavaScript library that makes it painless to create interactive UIs, by designing simple views for each state in your application in a declarative and component-based way. It efficiently updates and renders just the right components when your data changes. Its declarative views make your code more predictable and easier to debug.

## Prerequisites

* A Kubernetes environment reachable by your browser (you can start it with [kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/), [minikube](https://kubernetes.io/docs/setup/learning-environment/minikube/) or any other tool)
* General knowledge of Kubernetes [deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/), [services](https://kubernetes.io/docs/concepts/services-networking/service/) and [configmaps](https://kubernetes.io/docs/concepts/configuration/configmap/)
* General knowledge of Golang and JavaScript

Plus, if you want to test them locally on your machine instead on a Kubernetes cluster:

* Go installed (you can test it by running `go version`).
* Node installed (you can test it by running `node --version`).
* [Docker for Desktop](https://docs.docker.com/desktop/) with Kubernetes plugin enabled.

## Writing the application

Deploying an application on Kubernetes assumes, obviously, writing the code of your application.

In addition, this application must be containerized, meaning that you have to create an image of this application.

To create an image you need a [CRI](https://kubernetes.io/docs/setup/production-environment/container-runtimes/) (Container Runtime Interface) like [Docker](https://www.docker.com). Once created, we will push these images to a private Docker Registry (don’t worry, we will deploy it with a single command).

Finally, we need to deploy the Kubernetes resources in our cluster.

And this is exactly what we are going to do.

Wrapping up:

* We will write the backend in Golang using the [kubernetes-client](https://kubernetes.io/docs/reference/using-api/client-libraries/#officially-supported-kubernetes-client-libraries).
* We will write the frontend in React.
* We will create a Docker image of the backend.
* We will create a Docker image of the frontend.
* We will create an instance of a Docker Registry and push there our images.
* We will create and apply the Kubernetes resources to deploy our backend and frontend.

## Writing the backend

Our backend will consist of three files:

* `main.go`.
* `handlers.go` a file that contains all handlers called by the main.
* `k8s-functions.go` a file that contains all utility functions that we are going to use to fill the API calls written in the handlers file.

In order to be concise, I will explain the easiest API endpoints. If you want to see all the endpoints and related logic, I suggest looking at the bottom of the post where I have included the GitHub repository with all the code.

We will start creating the `main.go` file of our backend.

{{< gist PhilRanzato afb840b5665890b934b589986767f049 >}}

As we can see our `main()` function create a new router (using the dependency `gorilla/mux`) that routes traffic based on the call made.

To allow the calls in the web browser we need to enable the [CORS](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing).

Our server is now listening on port 80.

Let’s see the `podHandler` function:

{{< gist PhilRanzato 476a9b1f1a72ed69c139051d53df56b3 >}}

This function calls the creation of a clients (needed to retrieve the information about the cluster) and encodes in the http response the resources obtained by another functions that list the pods.

Let’s see these utility functions:

{{< gist PhilRanzato cc44b361269a16cdc007a0cc75ed3429 >}}

In this way, when the backend is queried on the endpoint `/api/pods` will encode in the http response the list of pods found in the Kubernetes cluster related to the `/root/.kube/config` on the deployment machine. In the same way, it will return in the http response the list of services at `/api/services`.

## Test the backend locally

We can test the backend locally by running `go run .` in the directory of these three files.

Now we can query the backend from the command line with a simple `curl localhost/api/pods`.

> Note: in order to run a go project, you should move the directory of the project in `$GO_PATH/src/<project>` and run the project from there. Remember that you must have a Kubernetes cluster up and running, so if you are doing this on your machine, be sure to start Docker for Desktop and its Kubernetes standalone cluster, otherwise you will receive: `curl: (52) Empty reply from server`.

## Writing the frontend

The first thing we need to do is to create our React application. We can use the command `npx create-react-app <AppName>`. This command will create our standard application with the classic React logo, listening on port `3000`.

Once the frontend is ready, we can start customizing our application.

We will use the `react-pro-sidebar` to allow navigation between pages. In order to use it we need to install it: `npm install react-pro-sidebar`.

To enable routing inside our application we install the `react-router-dom` with the following command: `npm install react-router-dom`.

We will customize the main file named `App.js` and we will create more files for all our subpages in a directory that I named `components`:

* `components/pods.js`
* `components/services.js`

Let’s start by the components.

{{< gist PhilRanzato 22ef4faff8a3c4c7b6ffc3678056605a >}}

In this component, we are creating a simple table, with no CSS, where we list the name, the namespace and the status of all pods we receive from the backend.

In the same way we write the second component where we list services’ name and namespace.

{{< gist PhilRanzato e3d025a79b7f74212ddd6b89930bf8fc >}}

Let’s put them together in the main file.

{{< gist PhilRanzato 094bd3e5f3bf1e7825a821f8abbe174b >}}

To explain the file above, the best thing to do is to split it into little pieces. We can start from the constant defined at top, in line 14:

`let endpoint = process.env.REACT_APP_BACKEND_SERVER`

In order to make this frontend more portable we define an environmental variable `REACT_APP_BACKEND_SERVER`. All React environment variables have to be named like `REACT_APP_<var>`. To set these variables we can use a file that must be called `.env` at the root of the project.

{{< gist PhilRanzato b0c38f2311aed551adf0dc22713def4e >}}

We can simply map the React environmental variable to another one that we can set to the value we want.

We then define the state that will be updated by our API calls to the backend:

{{< gist PhilRanzato a32956e84ddfb4a4fbd4980b2101ea11 >}}

This state is updated by the http response to these API calls:

{{< gist PhilRanzato 6ad2d3f2150b0796e32ba4f90e36b1ae >}}

We take the `.json()` parameter of the http response obtained by the call to `endpoint/api/pods` and `/endpoint/api/services` and we parse it into the correct parameter of `state`, filling our pods and services.

{{< gist PhilRanzato 1f4e2e59ff5b7a4e8a266a494c3bcbea >}}

The only thing left is to render the updated state into the web page. To do this we use Routes to our custom object: `Pod` and `Service`, defined as functions as below:

{{< gist PhilRanzato 8b1a9b9bf44747ad78ab076a144007f8 >}}

In this way we have added to sub-pages listing pods and services.

## Test the frontend locally

Firstly, let’s set the `BACKEND_SERVER` environment variable to `[http://localhost](http://localhost)` since our backend server is listening to port `80`.

* Ensure Docker for Desktop Kubernetes is running.
* Ensure the backend is listening on port `80`.
* Start the frontend by typing: `npm start` inside the root of the project

The result should be like this:

![Frontend](https://miro.medium.com/max/1400/1*lZr25MNRTk9H__mCRryyyQ.png)

Left: dashboard — center: pods list — right: services list

## Create the images

Once both the frontend and the backend are ready we can create their Docker images.

To do so, we have to write a Dockerfile for each of them.

Let’s start with the backend.

{{< gist PhilRanzato 5d1342567cd58f969ad23dcb8b5e188a >}}

Starting from the `golang` official image we add and build our project exposing the port that we have chosen.

Pretty easy, right?

Now let’s see the frontend.

{{< gist PhilRanzato 0a1a708ae8fe6dbeab5eecfda3b0c166 >}}

As before, we just start from an official image, `node` this time, and add our project and dependencies.

> Note: these Dockerfiles must be in the location of the project since we are copying its current directory as project.

### Create the private registry and push the images

Creating a private docker registry is just pretty straightforward: `docker run -d -p 5000:5000 --restart always --name docker-registry registry:latest`.

But we cannot use this registry until we set our docker daemon with the insecure registries key. Edited the `/etc/docker/daemon.json` with the following:

{{< gist PhilRanzato 10646c97103aa3e4bba5aa71ce470e70 >}}

We do not need any login since the registry image does not use any user.

Restart the docker engine to get the new daemon:

`systemctl daemon-reload`

`systemctl restart docker`

Once the container is ready we can build and tag an image (we can do this in a single step but it is cleaner in two in my opinion)

Move all the backend code into a `./backend` folder and the frontend into `./frontend` (also the respective Dockerfiles).

`cd backend`

`docker build -t go-backend .`

`docker tag go-backend localhost:5000/backend-go:1.0.0`

`docker push localhost:5000/backend-go:1.0.0`

Once the backend is ready, we can build the frontend:

`cd ../frontend`

`docker build -t react-frontend .`

`docker tag react-frontend localhost:5000/frontend-react:1.0.0`

`docker push localhost:5000/frontend-react:1.0.0`

Now we have our images on the docker registry.

> Note: if you are using a single node cluster, this step can be redundant since you can actually use the images on the host.

## Create the Kubernetes resources for the application

For both backend and frontend, we need to write Kubernetes resources in order to deploy on our cluster.

Let’s see what we need for the frontend:

* Deployment
* Service

That’s it. We don’t need anything else for our application since we don’t use any storage or configurations.

The backend is a little more complicated:

* Deployment
* Service
* Service Account
* Cluster Role
* Cluster Role Binding
* Config Map

We can start from the frontend, being the easiest to configure.

First of all, create the namespace:

`kubectl create ns k8s-utils`

Then we can apply the deployment and the service, defined as below:

{{< gist PhilRanzato 0d050a9746feba92b87e2649f749e98d >}}

A `kubectl apply -f react.yaml` applies the deployment and the service.

The frontend is ready!!

Now we have to write the kubernetes resources for the backend.

Since the application we have built lists all pods and services (among other things) we need to ensure that the kubeconfig that the application is using is somehow restricted to what the application will do. To be able to do this we will create a ServiceAccount named `go` with **only** the capabilities it needs to run the application. And how we can restrict its capabilities? With a Role, of course! To be more specific, a ClusterRole since we want to handle kubernetes resources on all namespaces. To add these capabilities to the Service Account we are creating a ClusterRoleBinding. Here below the resources just explained:

{{< gist PhilRanzato 15c3a47ee6b0323cc3556c96fc3f81c5 >}}

The next step is to create the kubeconfig for this ServiceAccount, in order to use it in the backend. To do so I have customised [this script](https://docs.armory.io/docs/spinnaker-install-admin-guides/manual-service-account/):

{{< gist PhilRanzato 0c88246a2927ab749e544a3dee13d88c >}}

We will create a configMap using the produced by the script above (named `config`):

`./create-sa-config.sh`

`kubectl create cm kubeconfig --from-file=config`

Now we are ready to create the deployment and the service to expose our backend:

{{< gist PhilRanzato 706c21f271a376c7bc3d8f713e709267 >}}

As you can see, both the backend and the frontend services have ClusterIP type. This means that you can’t reach them from in a web browser. But don’t worry! We will expose them using a ****reverse proxy****!

## Nginx reverse proxy

In order to expose our frontend to the outside world we will use an [**nginx**](https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/) reverse proxy configured to redirect traffic to our frontend, with the below configuration:

{{< gist PhilRanzato 49a6d18bc21104f89d1aca036d058563 >}}

But we need to proxy also our backend API calls!! Let’s see the full configuration of our nginx inside the configMap we will use:

{{< gist PhilRanzato a8e1902b6fe3da0f978c1a65a842f3c6 >}}

This nginx configuration will redirect everything on `/` to our frontend and every calls to the `/api/` endpoint will be redirected to our backend.

It’s time to deploy our nginx using this configMap!

{{< gist PhilRanzato f58b3d197e3c0c1a7cc5e3b0af4785e0 >}}

We expose the nginx deployment in NodePort to make it accessible from the outside world.

## Access the application

If you are deploying the application on your machine using Docker for Desktop with Kubernetes, you can access your application at `[http://localhost:32700.](http://localhost:32700.)`

Otherwise if you are deploying the application inside a virtual machine you should use its IP address ( `http://<VM_IP>:32700` ).

Before in the backend code, to avoid being verbose, I omitted the other features of the application:

* The sub-page **Exec into Pod** allows you to send a command to a pod and see the result (with green background for a successful command, while red if it fails)
* The sub-page **Test Connection** will test the connection from a pod another pod by using a service (it runs a `wget` command, be sure to use a pod with this executable)

## Conclusions

And this is how to deploy a full-stack, cloud-native, application on Kubernetes.

Check out the GitHub repositories related to this tutorial:

* Kubernetes Resources: [https://github.com/PhilRanzato/k8s-utils-kubernetes-resources](https://github.com/PhilRanzato/k8s-utils-kubernetes-resources)
* Go Backend: [https://github.com/PhilRanzato/k8s-utils-backend-golang](https://github.com/PhilRanzato/k8s-utils-backend-golang)
* React Frontend: [https://github.com/PhilRanzato/k8s-utils-frontend-react](https://github.com/PhilRanzato/k8s-utils-frontend-react)

And the Docker images too:

* [pranzato/k8s-utils-backend-golang:1.0.0](https://hub.docker.com/repository/docker/pranzato/k8s-utils-backend-golang)
* [pranzato/k8s-utils-frontend-react:1.0.0](https://hub.docker.com/repository/docker/pranzato/k8s-utils-frontend-react)

In the next episode, we will change the backend language and see how we can easily switch them.

See you next episode!

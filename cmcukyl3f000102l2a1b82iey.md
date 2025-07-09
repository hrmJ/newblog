---
title: "My Kubernetes journey so far"
datePublished: Tue Jul 08 2025 13:43:09 GMT+0000 (Coordinated Universal Time)
cuid: cmcukyl3f000102l2a1b82iey
slug: my-kubernetes-journey-so-far
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1751982803206/d10da8b6-812d-4425-a3bf-14ad19ced715.png
tags: web-development, kubernetes

---

Just finished reading William Deniss’s book [Kubernetes for developers](https://www.manning.com/books/kubernetes-for-developers) and thought this might be a good place for some recapping of not just what I’ve learned by reading but also how my understanding of Kubernetes has evolved over the last couple of years of me actually working with the technology.

Let’s kick things off with a network graph based on my Kubernetes-related notes:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1732884931874/deb86575-6b1e-48b8-b6f4-ed4c203a5f78.png align="center")

I guess that’s a pretty accurate description of how I view this rather massive field at the moment. The difference to 2 or three years ago are **the connections** (now, hopefully, existing in my mind as well) mapping the concepts to each other and to my other general knowledge of developing web applications.

## How it used to look like

I remember first noticing the term Kubernetes when listening to *Linux unplugged* and raising my eyebrows hearing an out-of-place Greek word amidst all the chatter about containers. I got the usual explanation about container orchestration and such, but didn’t quite understand how this relates to docker, containers and was quite baffled by the question *how is this different from just running docker compose*? Interestingly enough, I bought a version of Denniss’s book when it was still at a very early stage and when I had no practical experience with Kubernetes - somewhere around 2021. I was intrigued by the *for developers* part in the title, correctly assuming it might make things understandable enough for a js dev like me without going too deep into details I would be unlikely to care about.

Quite early on I started to make the docker compose comparisons for myself, trying, first of all, to understand why “pods” instead of “containers” and if there is a difference (3 years on I feel I can finally say yes and understand why). When I first stepped into a codebase deployed into a Kubernetes based environment my idea of the big K changed into the reality reflected by this AI generated image:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1732886819846/352e2a31-2983-449d-a395-9b68053b56d8.png align="center")

I had never seen so much yaml in my life. Then I learned about the enormous ecosystem of tools, often named using words involving a more or less creative use of the letter K ( *Kustomize, Istio, Karpenter, kubectl, skaffold, helm, minikube, microk8s, rancher,helm*….) . Trying to have something run on Kubernetes on my linux box seemed outright impossible and I admit feeling quite desparate.

## How it all started to make sense

Like with any technology, getting involved in a real project utilizing the tech helped a ton. With some assistance from more experienced colleagues it isn’t that hard to start working in a kubernetes based project, run some magic commands (or just push code into a branch), and see your code applied in a running web application. At some stage you learn, what files to touch when a certain env var needs updating, how to connect to another microservice, how to move between different environments… Still, Kubernetes can be a gigantic black box for a long time even if you are actually working and immersing yourself in a kube-based environment. This is, I think, a perfect place to actually read a book like Kubernetes for developers and see how that helps connecting the dots and understanding how the magic works. Here’s a screenshot of the book's TOC from Manning’s product page:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1732887966292/006bb2b9-7e19-47e3-b04f-9b40581276a9.png align="center")

Let me use the TOC to briefly highlight some of the key things I’ve learned. To be honest, my reading of the book came somewhat late: most of what I read I had already realized the hard way through trial, error, asking for help and watching the real experts do their thing with some apparently magical kubectl commands. But having done the reading I feel like all the pieces now form a much clearer picture in my head. Here’s a simple list of three key areas that I have started to think about when trying to define Kubernetes and answering the question of “why use Kubernetes when there’s docker compose?”

1. Scaling
    
    * Instead of just spinning up microservices Kubernetes gives you a way to ensure your services are up, are restarted if needed and are dynamically assigned more resources if needed.
        
    * What I didn't originally understand and why I found it hard to start experimenting with Kubernetes was the concept of *nodes*. When developing locally on a physical machine you're more or less stuck with a single node to run the cluster. Denniss actually emphasizes that a good way to start learning is to use a cloud provider from the get-go which would have been good advice for me in the beginning. That being said, a local cluster is, in my opinion, not as hard to setup as it used to be: docker desktop and Rancher desktop have that functionality built-in.
        
    * Another key concept I didn’t know about early on was that of *deployments.* Instead of thinking, by default, about *pods* or *containers* , it’s almost always better to start with deployments when answering the question about what comprises a microservice in your app. A deployment is the thing you want to have up and running and you can use pods and containers to help you with the task of keeping it running. For instance, when you’re rolling out a new version of the microservice a deployment let’s you gracefully shut down the old pods with the old version after having made sure pods with the new version are ready to serve the users of the app. And for each pod of, say, an express api with a database connection, you can have one or more initialization containers that first ensure that all the necessary migrations have been applied and only after that spin up an actual api container. And then there is the concept of replicas and other scaling related factors determined on a deployment level: how many instances of an api do I want to have up receiving the traffic? How much resources should they be given? When is an api considered ready for taking in “customers”?
        
2. Networking
    
    * In Kubernetes the word *service* is purely a network-related concept. At first I was confused when looking at the service.yaml files in the first Kubernetes project I became a part of: why are these so small? Why only the port the app is running on? After having understood that the actual heavy lifting is done with deployments the role of services as a description of a microservice’s connections inside it’s host environment started to make sense.
        
    * A trickier task was to know how to describe the traffic from outside the environment. For that I found *ingresses* to be the concept easiest to grasp. A simple `kubectl describe ingress some-ingress` actually tells a whole lot about an app’s structure! Alas, ingresses are not the only tool for controlling subdomains and connecting services with the outside world. In production environments I was quickly faced with concepts like Istio, which definitely bring another level of complexity into the game.
        
3. GitOps
    
    * Even though the amount of yaml files can be overwhelming, think about the mess that a purely imperative way of configuring an app creates: instead of cleanly structured, standardized yamls you would have just… well, your bash history, I guess, or a complex-as-heck `deploy.sh` that you would be scared to death to ever run yourself.
        
    * The concept of GitOps only became clear to me with Kuberenetes: having all that yaml means being able to do all the necessary configuration with descriptive files that can be committed to version control. And once the configuration is in place, it becomes astonishingly easy to deploy the exact same app to multiple environments, like separate feature environments in the cloud, a dedicated on-premises environment or even your local dev environment.
        
    * The idea of controlling everything related to a specific environment with a simple utility like `kubectl` is a powerful one. Even more powerful it became once I realized how a kubeconfig file can hold enough information for you to access all the necessary environments from your own machine, especially with the help of a utility like [https://github.com/ahmetb/kubectx](https://github.com/ahmetb/kubectx). That’s like continuously running gazillions of ssh sessions inside dozens of tmux panes and terminal windows!
        
    * Coming back to the idea of using Kubernetes to run a local dev setup: I found using [https://tilt.dev/](https://tilt.dev/) not only gave me the possibility of mirroring a production environment’s structure but also gave me a nice interface for controlling the pieces of the app I am developing. Kubernetes truly is a tool that creates an endless need for new tools!
        

## What’s next?

Life’s busy! It took me months to come back to this blog posting after the initial idea of summing up my Kubernetes journey. Interestingly enough, as rapidly evolving as the web dev world is, the core concepts of Kubernetes have, to me at least, stayed pretty stable all the while. I haven’t had to re-learn yet alone unlearn many things, and the muscle memory you gain each day when debugging, running and deploying kube-based apps feels more and more valueable every day. The next step on my Kubernetes journey might be to dig deeper on the node level and move further down from the actual app logic into the underlying infrastructure that keeps everything running. But do I want to go there? After all, nothing beats the thrill of making a small change in your code and seeing the actual UI change, right?
# Overview
This is a demo project intended to explore a solution for MicroVM-based FaaS platform. 

Contact: Liming Liu liming.liu@campus.tu-berlin.de

## 1. Architecture Design
Since the goal of this demo is to explore a potential solution for MicroVM-based FaaS platform, a simplified architecture is being used. Compare with a full-fledged platform, the demo mainly focuses on the function handler part of FaaS service and temporarily not implementing client gateway and automated management modules. An overview of the simplified architecture is shown below:
![Architecture overview](https://i.ibb.co/kKz02Wm/architecture.png)

The workflow of the simplified architecture is:

1: The platform deploys Docker environment, then run Weaveworks Ignite in a docker container  
2: The developer uploads function code  
3: Deploy new function ( this step is now by manual, with further development it should be automatically managed).   

 - Ignite creates a new Firecracker microVM instance for the new function
 - The platform copys the Http server and function code into the new VM
 - The platform runs the Http server in the new VM

4: The client invokes the function by sending HTTP request to the Http server, the server will then call the function and pass the output back to client

## 2. Prerequisites

To run this demo you need to install docker and Weaveworks Ignite on the host machine. You can install the prerequisites by following their official docs:
Docker:  [Docker Installation guide](https://docs.docker.com/get-docker/)
Weaveworks Ignite:  [Install guide Weaveworks Ignite](https://github.com/weaveworks/ignite/blob/master/docs/installation.md)
After you finish you should be able to run docker and Weaveworks Ignite. You can check your Docker and Ignite version, here are the result on my host machine:

![Docker version](https://i.ibb.co/j64DVKN/Docker-version.png)

![Ignite Version](https://i.ibb.co/5r96c0t/Ignite-version.png)

## 3. Deployment

To deploy the demo project firstly clone the repo to your local

    
    mkdir ./demo
    cd ./demo
    git clone https://github.com/limingliu-eit-kth-tub/faas-microvm-proof-of-concept-demo.git
    cd ./faas-microvm-proof-of-concept-demo/
    
   As you can see there are four files in the repository, I will explain their functions one by one.
   

 - The **hello-world<span>.py** is the example function code in this demo, it prints out a simpel HTML page, if you like you can replace with any function you want to play with.
 - The **localfunction<span>.sh** is the command to execute the function code, in our case it executes hello-world<span>.py
 - The **server<span>.py** was intended to serve as the http server like in tinyFaaS architecture or the watch dog in OpenFaaS architecture, it receives the HTTP trigger from gateway and execute the function. However since this is a simple demo, I did not build a client gateway, so this extra layer of proxy is now transparent. The client( browser) need to send request directly to this server to access the function.
 - Finally the  **func-handler<span>.yml**  is the configuration file of the MicroVM machine when being created by Ignite. Here I use a template file from Ignite website. It declares the new VM to be assigned 1 CPU, 2GB disk and 800MB memory. Addition to that the "ssh" specification is added to securely transfer file from host machine to the Firecracker MicroVM.
 
 Since the focus in this demo is to show how the function handler works, we temporarily will ignore the automation process and do things manually( just for now). 
 Let's suppose hello-world<span>.py is the function code that developers just uploaded. Then the first thing the platform should do is to create a new microVM instance for it. So we run this:
 

    sudo ignite run --config func-handler.yml 

A new microVM is created, you will see the INFO message. Then we can check the new vm by:

    sudo ignite vm ls | grep function-handler-vm
Now the new microVM instance is created, we will deploy our function on it. To do this we need to copy the files to new VM instance then start the server from the host machine( not the VM):

    sudo ignite cp ./ function-handler-vm:./
    sudo ignite exec function-handler-vm python3 server.py
   Now you will see some info saying the httpd is started. This shows that the server on the microVM is up. However we have one more step to access the fuction. Since we do not have a proxy or gateway yet, we need to know the VM's IP in order to send request to our newly extablished server. We can do this:
   

    sudo ignite vm ls
   This command will list all VMs created by Ignite and you need to find "function-handler-vm" instance, copy its IP and then open a browser, request [copied ip]:8082, and we are getting there! ( The reason I set the server port as 8082 is because I have other project running on 8080, you can manually set it to any port you want in server<span>.py script)
   
![The Hello Function Page](https://i.ibb.co/BLKX3tG/Final.png)
## 4. Future work
Up to now our FaaS platform feels very unhandy to use. Yes indeed it is. But with further development we can automate the management as well as adding a client proxy( or a gateway as you might like to call it). Ideally, the developer will simply need to select a programming language, upload their code and then the web deployment will be handled automatically. 

There are lots of add-ons the mainstream FaaS platforms provide, kind of making them must-haves. For example auto-scaling, load-balancing and performance monitoring, etc. Some of them we also will look into in future.



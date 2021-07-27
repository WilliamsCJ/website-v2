+++
date = 2021-07-28T11:00:00Z
linktitle = "StudentLayer Part 3 - Deploying our service"
series = ["StudentLayer"]
title = "StudentLayer Part 3 - Deploying our service"
type = ["posts", "post"]
weight = 10
[author]
name = "CJ Williams"

+++
In our last post, we created a basic Hello World service. Now we'll look at Docker-ising our service before deploying it to a VPS.

> The code for this post can be found [here](https://github.com/WilliamsCJ/studentlayer/commit/fe7e2b93f2a2ae0e8ca3a7ecd664520aa609a741)

## Creating our Dockerfile

First, we'll start by setting the base image for our single-stage build. This base image will use the same version of Go that we installed previously:

```docker
FROM golang:1.16-alpine
```

Next, we'll create a working directory in which to build (and run) our binary. The _WORKDIR_ command creates the directory app and sets the working directory to it:

```docker
WORKDIR /app
```

Now we'll copy over the Go modules files and install our dependencies:

```docker
COPY go.mod ./
COPY go.sum ./
RUN go mod download
```

Next, we can copy over our program files and build our binary:

```docker
COPY *.go ./
RUN go build -o /studentlayer
```

Notice that we've used the -o flag to name the output binary, which isn't strictly necessary. However, this is desirable for our Dockerfile, as it prevents module name changes in our project from breaking our Dockerfile.

Another important feature of our Dockerfile is that we must expose the relevant port on our Docker container. Our router currently listens for traffic on port 8080; however, we must expose this port on our container to allow traffic to reach our router. We do this as follows:

```docker
EXPOSE 8080
```

The last remaining step of our Dockerfile is to execute our binary. We can do that with this command:

```docker
CMD [ "/studentlayer" ]
```

## Building and Running our Dockerfile

We can now build our Docker image using the following command (make sure to run this in the same directory as the Dockerfile we created):

    docker build --tag studentlayer .

Gives our built image the tag _studentlayer_. This allows us always to run the latest version of our image using:

    docker run -p 8080:8080 studentlayer:latest	

Note the use of the flag `-p 8080:8080`. This instructs Docker to map port 8080 of our local machine to port 8080 on our local machine, allowing us to access our service at _localhost:8080_.

## Setting up our VPS

For this project, I'll be using a VPS from [Fasthosts](https://www.fasthosts.co.uk). I've chosen to use a basic Linux (Ubuntu 20.04 specifically) server instead of serverless or other more managed options, so we can really explore what is going on.

Once our VPS is online, we can start by SSH'ing into the server using the supplied root credentials. Once we have access, we should immediately create a new user with a strong password, add it to the sudoers group and disable the root user.

To create a new user, follow [this article](https://www.digitalocean.com/community/tutorials/how-to-create-a-new-sudo-enabled-user-on-ubuntu-20-04-quickstart). As this box is exposed to the internet, we must make sure we set a strong password.

The root user can subsequently be disabled using:

    sudo passwd -dl root

While we have set a strong password and disabled root, we probably want to take one more measure to help secure our box - setting up certificate-based authentication and subsequently disabling password-based authentication. This mitigates the risk from a bad actor attempting to brute force our SSH login. To do this, follow this [helpful guide](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-2) from DigitalOcean.

Lastly, we need to make sure we have the tools we need on our box. Currently, these are _Git_ and _Docker_. To install them:

    sudo apt-get update
    sudo apt-get install git docker -y

## Deploying our service

Now that everything is set up, we can deploy our service. Let's start by cloning our _Git_ repo:

    git clone git@github.com:WilliamsCJ/studentlayer.git
    cd studentlayer

Now we can build and run our container:

    sudo docker build --tag studentlayer .
    sudo docker run --name studentlayer -p 80:8080 -d --restart unless-stopped studentlayer:latest 

Our service is now up and running - amazing! Before we test it, let's examine some differences in building and running our Docker container. Firstly, notice that we used `sudo` to run our Docker commands - on our system, most Docker commands are privileged and require root. Secondly, notice that our port mapping has changed to `-p 80:8080` - this is so we can expose our container to the outside world on port 80, the port number assigned for HTTP traffic. Lastly, we have added the `--restart unless-stopped` flag. This directs Docker to restart our container if it ever stops, except if we stop it ourselves - useful in case our service crashes.

## Visiting our service

Now, if we visit `<OUR VPS IP ADDRESS>/studentlayer`, we will get a Hello World response from our server. However, providing our service from an IP address is not particularly user-friendly. IP addresses are not easy to remember and, in some cases, can change. A better solution is to set up a hostname to point to our server.

We can do this by creating an ALIAS record that maps a hostname to the IP of our VPS. For my DNS provider, Cloudflare, we can follow [this guide](https://support.cloudflare.com/hc/en-us/articles/360019093151-Managing-DNS-records-in-Cloudflare).

I've set up my service at _api.studentlayer.cjwilliams.io_, as I don't want to buy a separate domain, but I also want to leave room for a landing page or API docs.

We can visit our service from a web browser or by sending an HTTP GET request using cURL:

    curl http://api.studentlayer.cjwilliams.io/studentlayer

_Note_: You may need to make sure that the URL includes _http://_, as some web browsers will default to _https://_ and we haven't set this up for our API yet.

## Wrapping up

Wow! That was a lot of information, but we have now deployed our service. In the next post, we'll look at making our service actually do something useful - student email verification!
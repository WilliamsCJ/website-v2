+++
date = 2021-07-25T11:00:00Z
linktitle = "StudentLayer Part 2 - A Hello World Service"
series = "StudentLayer"
title = "StudentLayer Part 2 - A Hello World Service"
type = []
weight = 1
[author]
name = "CJ Williams"

+++
# 2 - A Hello World Service

In this post, we will set up our development environment and create a basic, Hello World style, service. While we will go through how we set up our project, this post won't be a step-by-step tutorial - there are many superior tutorials for all of the topics we will cover. 

**Side note**: From herein, I will only reference commands for a Unix-based system (macOS specifically), though these should work for any POSIX-compliant OS.

> The code for this post can be found [here](https://github.com/WilliamsCJ/studentlayer/commit/644ce50a7eae09baad14a5035a8f59b2dfc07e08)

## Installing Go

For the time being, we will use the latest stable version of Go - 1.16.6. As the project goes on we will attempt to stay up to date, dealing with any dependency maintenance issues that may arise - a potentially painful but very relevant endeavour.

To speed things up, we'll just download the macOS installer from the official Golang [site](https://golang.org/dl/),
but feel free to download and install the archive if, for instance, you are running Ubuntu.

Verify you have 1.16.6 by running:

```shell
go version
```

If you have had previous installations, you may need to uninstall any previous versions. Problems can also arise if your GOPATH and GOROOT are not set correctly.

## Go Modules

We will use Go Modules to manage dependencies. We will start by creating a module for our project, with a name that
matches our GitHub repository. We'll run this within our Git repository:

```shell
go mod init github.com/WilliamsCJ/studentlayer
```

## Creating a basic router

Now we can get down to creating a basic HTTP request router that will form the basis of our service. For this, we will use _net/http_ from the Go standard library. Whilst our service is in its infancy, we don't need a complexity web framework or router, such as Gin or Chi. _net/http_ serves our needs and allows us to really understand what is going on.

We can create a basic Hello World router as follows:

```go
package main

import(
	"fmt"
	"log"
	"net/http"
)

func main() {
	router := http.NewServeMux()
	router.HandleFunc("/studentlayer", func(w http.ResponseWriter, r *http.Request) {
		log.Printf("Serving %s request for %s", r.Method, r.URL)
		_, err := fmt.Fprintf(w, "Hello world")
		if err != nil {
			panic(err)
		}
	})

	log.Println("Running on localhost:1000...")
	log.Fatal(http.ListenAndServe(":1000", router))
}
```

This program creates a router that handles a single route _/studentlayer_. As we only have one (small) handler function, we can get away with using an anonymous function. However, as things get more complicated, we will want to define separate handler functions. Currently, all our handler does is log some request information received from the _http.Request_ struct and writes our Hello World response to the _http.ResponseWriter_ using _fmt.Fprintf_. 

Note however that we're using _log_ and not _fmt_ for logging output. This is for the following reasons:

1. _log_ is concurrency safe (useful for later). _fmt_ is not.
2. _log_ outputs to _stderr_. _fmt_ outputs to _stdout_. Read [this](https://www.gnu.org/software/libc/manual/html_node/Standard-Streams.html) if you don't understand why this matters. N.B. we would consider logging to be 'diagnostic output'. The 'normal output' of our program is our API responses.
3. _log_ adds timestamps automatically.

## Running our router

We can start serving requests from the router by running:

    go run main.go

This will give us an output like so:

```s
2021/07/24 17:06:45 Running on localhost:8080...
2021/07/24 18:27:49 Serving GET request for /studentlayer
```

We can also compile our code into a binary that we can subsequently run. This is done as such:

    go build -o studentlayer
    ./studentlayer

_Note:_ The `-o` flag is not strictly necessary in this instance as our binary will automatically be named `studentlayer` because our module name (`github.com/WilliamsCJ/studentlayer`) ends with `studentlayer`.

## Wrapping up

While our router doesn't do much yet, it will form the basis of our service. In the next post, we will look at Docker-ising and deploying our service.
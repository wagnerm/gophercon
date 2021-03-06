# Philosophies of Go
Simplicity. Go programming is simple as well as the intended function of the program.

## Other things I learned that seem to be good practices?
* If something is hard to do, make someone else do it (push complex functionality up the stack)
  * Makes the library simple
  * return errors, don't deal with them within the library
  * channels and goroutinues are very rarely used in a public api - allow someone else to make you program concurrent or deal with the error
* Your library should serve a singular purpose - make it simple enough anyone else can plug and play it into their library.
* You're bad if you make a heap of scripts

# Talks

## Understanding nil
* nil is the default value for a variable that is uninitalized. nil can be different types.
* Defines the 'zero' value for a specific type
  * int is 0
  * string is ""
* vars are not generally set to nil
https://speakerdeck.com/campoy/understanding-nil
https://tour.golang.org/basics/12

## Go Without the Operating System
http://atmanos.org/talks/go-without-the-os-gophercon-2016/#1
Allows you to compile and run Go programs on cloud platforms like Amazon (for now) without a traditional operation system.
Uses a __Unikernel__: the minimal set of libraries to run the Go program.

## Go Datascience
https://github.com/gophercon/2016-talks/tree/master/DanielWhitenack-GoForDataScience

Contrived comparison from python and go to find Max value
```
$ cat example.csv
1,testval1
2,testval2
,testval

Python
$ python example.py
2.0

Go
$ go run example.go
$ go run example2.go
2016/05/12 13:23:53 Parse
failed: strconv.ParseInt:
parsing "": invalid
syntax
exit status 1
```

Integrity & Deployment
Dockerfiles for python are huge
```
FROM ubuntu:latest
RUN apt-get -y update && apt-get install -y wget nano locales curl unzip wget openssl libhdf5-dev libpq-dev \
    && apt-get clean
RUN pip install -r /requirements.txt && rm -rf /root/.cache/pip/*
ADD . /workspace
WORKDIR /workspace
ADD notebook.sh /notebook.sh
RUN chmod a+x /notebook.sh
EXPOSE 8888
CMD ....
```
Go: Static Binaries
```
FROM scratch
ADD myservice /myservice
CMD ["/myservice"]
```

Tools to data science:
* GoNum: https://github.com/gonum
  * Graphs, plots, etc
* Pachyderm - data lake, git for data: https://github.com/pachyderm/pachyderm
  * Data lake good for storing persistent data in a consistent state
  * Git for data: view commits, file, diffs

## Crypto for Go
https://github.com/gophercon/2016-talks/tree/master/GeorgeTankersley-CryptoForGoDevelopers

We should probably be using some type of crypto for password
Provided code: https://github.com/gtank/cryptopasta

## Errors (the proverb thing)
https://github.com/gophercon/2016-talks/tree/master/DaveCheney-DontCheckErrorsHandleThemGracefully
Dave Cheney's blog: http://dave.cheney.net/
* Errors are just values - any value that implements the `error` interface is itself an error
* Three ways to handle errors:
  * Sentiel Errors: signifies no futher processing is possible, least flexible
    ```
    if err == ErrSomething { ... }
    ```
    * __Never__ inspect error.Error()
    ```
    if strings.Contains(error.Error(), "not found")) {...}
    ```
    * Sentiel errors become part of your public API (returns a sentiel value, value must be exposed)
    * Create a dependency between two packages
  * Error Types
  ```
  if err, ok := err.(SomeType); ok {...}
  ```
    * Errors types must be public/exposed!
  * Opaque Errors: let someone else deal with it
  ```
  if err != nil {
    return err
  }
  ```
4th option:
https://github.com/pkg/errors: Wrap errors with original error and stack trace.

## Reflection
https://github.com/gophercon/2016-talks/tree/master/EmilyGu-PowerAndEleganceOfReflection
Allows program to examine its own structure and even alter it at runtime.
Laws: https://blog.golang.org/laws-of-reflection

## Visualizing Concurrency in Go
https://divan.github.io/talks/2016/gophercon/
Native support for concurrency. Just because the goroutines and channels are available to you doesn't mean you have to use them in your program.

Demo: https://divan.github.io/posts/go_concurrency_visualize/

## Inside the Map Implementation
https://docs.google.com/presentation/d/1CxamWsvHReswNZc7N2HMV7WPFqS8pvlPVZcDegdC_T4/edit
* hash and store into buckets, will double in size and copy elements to the new
memory location on each new entry

## Go assembler
Can't find slides :(

Implementation recently changed - 1.6 replaced a suite of C programs, one per architecture, with a single binary, written in Go, that supports all architectures.

The usual variables, GOOS and GOARCH, are sufficient to configure it for any environment.

Go on other architectures?

## Practical Advice for Go Developers
https://github.com/cep21/go-talks

Main topics:
* Consider package name in struct names
  Stuttering not acceptable:
  Not:
   var c client.Client
  Optimal:
   var v http.Client

* Object construction
  Using the struct
   x := http.Client()
  Zero value considerations
   var c http.Client
   y := bytes.Buffer{}
Bad: Zero values are viral, need nil considerations

* New() constructor functions that return and interface are not usually good
  * Really big interfaces are hard to extend.
  * Interfaces describe behavior, not large piles of junk

* Logging
  * (BAD) log.Println prints to std streams, many logging freamworks, cannot turn off
  * (Good) push error up the stack

* Interfaces
  * Accept interfaces, return structs
  * Export interfaces, keep all structs private
  * Usually no need to include interfaces from outside your package/stdlib
  * Avoid large interfaces - turn large interfaces into small interfaces with wrapper logic in the structs

* Never panic, especially in a gorutine. In the Init might a good place to panic

* don't construct for the user

* don't make the dependency, take the dependency

## Vendoring
https://github.com/gophercon/2016-talks/blob/master/WisdomOmuya-GoVendoringDeconstructed/GoVendoringDeconstructed.pdf

Vendoring: Go 1.5 includes experimental support for using local copies of external dependencies to satisfy imports of those dependencies. So Go stores it's dependencies within the source of library!

This has recently changed with newer versions of Go:
* Import resolution happens in `$GOPATH/vendor` then in the parent's `vendor` and so on..
* Vendoring inclusion:
  * Go 1.4 no vendoring
  * 1.5 Experimental through GO15VENDOREXPERIMENT env var
  * 1.6 Default to have vendoring
  * 1.7 Vendor is on, point of no return

__Literally__ someone else's code is in your repository:
```
Example Layout
/home/projects/
    mongoproxy/
        main.go
        modules/
          mongod/
              proxy.go
          vendor/
              src/
                  github.com/go-mgo/mgo <--- Depedency from someone else
```
Code?
```
import "github.com/go-mgo/mgo"
```
Build? Easy
```
export GOPATH=/home/projects/mongoproxy
go build
```

Good things?
* Depedencies don't change unless I update them.
* Don't have to install dependencies like python
  * No pip

Bad things?
* Automate dependency update?
* Vendoring is now a requirement
* No pip, npm... use 'go get' instead

Impacts for BTI?
* No pip!
* Smaller Docker images
* We don't care what dependencies you have, it's your code.

### Examples of new Go Vendoring:
https://github.com/drone/drone
```
vendor/
  github.com/
  gopkg.in/
```

### Examples of using 3rd party vendoring tools like Godep
https://github.com/HewlettPackard/oneview-golang
```
Godeps/
  Godeps.json
  _workspace/
    src/
      github.com
vendor/ <-- symlink
```

## Binaries vs Dockerfiles?
Statically compiled binaries vs Dockerfiles that build go? Hmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm... Both?

#### Docker?
* Easy deployment
* Static compile make small Docker images
```
go build -v .

FROM scratch
COPY ./cmd /cmd
CMD ["/cmd"]
```
* _very_ small containers

## Kubernetes Lightning talk (no slides)
* Pre-setup a kubernetes cluster with 3 nodes
* Custom scheduler written in Go
  * Can customize the scheduler to deploy the containers based on binpack, cpu, etc
  * Custom scheduler runs in a container in the kub cluster
  * Attaches to the default scheduler in kubernetes so that there is schedule failover (2 schedules).
* Created a deployment.yml file
  * Specified custom scheduler
  * Containers/Apps to run in the cluster (Some web apps with a load balancer)
* Ran the deployment.yml file in the kub cluster, spun up 8 containers and scheduler
* Can start a rolling update with the deployment.yml file for all the containers/apps

Kelsey mentioned something about a credentials manager was running as a pob in the cluster.
New services would bind to the credentials manager so your app doesn't have to have credentials.

Kub workshop: https://github.com/kelseyhightower/intro-to-kubernetes-workshop
Kub dockerfiles: https://github.com/kelseyhightower/kubernetes-docker-files
The hard way https://github.com/kelseyhightower/kubernetes-the-hard-way

# Links
GopherCon 2016 Slides from most of the speakers: https://github.com/gophercon/2016-talks

Go Proverbs by Rob Pike: https://go-proverbs.github.io/
Don't just read the proverbs - you must understand the meaning behind them.

For something warm and fuzzy (not programming) - Renee French: The Go Gopher A Character Study: https://github.com/gophercon/2016-talks/blob/master/ReneeFrench-TheGoGopherACharacterStudy/gopherconfinal.pdf

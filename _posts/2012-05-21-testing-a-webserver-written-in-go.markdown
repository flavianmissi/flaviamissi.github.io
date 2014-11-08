Testing a webserver written in Go
I’ve been recently working on an api that needed to be super fast and made async calls to Canonical’s Juju. For this job, my team and I choosed to use [Golang](http://golang.org/), wich’s aim is to be fast and easy to learn.

We’ve found some difficulty searching for samples of how to test the api handlers that we wrote, reading Juju’s source code I found some tests that started a test webserver and used it to make the requests. Based on that sample, I made some very clear tests, on top of some handlers that I wrote for that porpose.

Those tests served as a base for all Tsuru’s api tests, that you can find here: [http://github.com/globocom/tsuru](http://github.com/globocom/tsuru)

The original base tests can be found here: [https://github.com/flaviamissi/go-webserver-sample/blob/master/handlers/handlers_test.go](https://github.com/flaviamissi/go-webserver-sample/blob/master/handlers/handlers_test.go)

Hope it helps ;)

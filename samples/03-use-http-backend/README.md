# Using an HTTP backend service

This scenario demonstrates the use of an HTTP cache backend service. You can use this approach to set up a local cache for testing (although you can also use the [built-in local cache implementation](../01-simple-local-caching) if you want to run some simple tests). It is also possible to share the HTTP cache between multiple machines running Gradle builds.

Gradle comes with built-in support for HTTP cache backends.
Gradle Enterprise ships with its own [build cache backend](https://gradle.com/build-cache) to make it easier to use this across an entire organization.

## Preparations

We'll need an HTTP server that is capable of storing files uploaded via `PUT` requests.
We have a ready-to-use [build cache node](https://hub.docker.com/r/gradle/build-cache-node/) Docker container that you can fire up with little extra hassle.
The build cache node operates as an HTTP Gradle build cache, and can connect with [Gradle Enterprise](https://gradle.com/enterprise) for centralized management.
Run it via:

    $ docker run --detach --publish 8885:80 gradle/build-cache-node

## Running builds using the backend

The HTTP cache is already pre-configured in [`settings.gradle`](http-test/settings.gradle). You can override the URL to the cache via `-Dcache.url=...`. Note that the URL's path must end in `/`.

In the below examples we are assuming you set up the Docker container locally. If that's not the case make sure the URI points to the HTTP server.

```text
$ cd http-test
$ ./gradlew --build-cache clean run
Task output caching is an incubating feature.
:clean
:compileJava
:processResources UP-TO-DATE
:classes
:run
Hello World!
```

At this time, the results of `:compileJava` are stored in the cache.

**Note:** We're running every build with `clean` so that the incremental build feature in Gradle would not label some tasks as `UP-TO-DATE`. This is only for demonstration purposes because this allows the new task output cache feature to kick in. For normal use, the build cache will help in situations where task outputs have been created by another build somewhere else (CI) or previously on your machine without a `clean`.

Let's run the build again:

```text
$ ./gradlew --build-cache clean run
Task output caching is an incubating feature.
:clean
:compileJava FROM-CACHE
:processResources UP-TO-DATE
:classes UP-TO-DATE
:run
Hello World!
```

Notice how `:compileJava` is now `FROM-CACHE`.

## Specifying credentials

Currently Gradle's HTTP backend supports only HTTP Basic authentication. You can pass them via `settings.gradle` like this:

```groovy
buildCache {
	remote(HttpBuildCache) {
		url = "http://example.com/"
		credentials {
			username = "username"
			password = "password"
		}
	}
}
```


## Troubleshooting

If running from different computers does not have the desired result (i.e. `:compileJava` being in `FROM-CACHE` state), check if the version of Java is the same on both computers.

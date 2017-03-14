# Simple local caching

This is a test scenario that demonstrates the difference between using the new task output caching vs. relying only on the existing incremental builds feature.

## With caching disabled

Build project, run application:

```text
$ ./gradlew run
:compileJava
:processResources NO-SOURCE
:classes
:run
Hello World!
```

Run it again without changes (`:compileJava` will not be executed):

```text
$ ./gradlew run
:compileJava UP-TO-DATE
:processResources NO-SOURCE
:classes UP-TO-DATE
:run
Hello World!
```

Destroy incremental build data

```text
$ ./gradlew clean
:clean
```

Re-run again (notice that `:compileJava` is re-executed):

```text
$ ./gradlew run
:compileJava
:processResources NO-SOURCE
:classes
:run
Hello World!
```

## With cache enabled

Empty the local cache directory:

```text
$ rm -rf ~/.gradle/caches/3.6-20170314000035+0000/build-cache
```

Destroy incremental build state:

```text
$ ./gradlew clean
:clean
```

Build project with cache enabled, run application:

```text
$ ./gradlew --build-cache run
Build cache is an incubating feature.
Using directory (~/.gradle/caches/3.6-20170314000035+0000/build-cache) as local build cache, push is enabled.
:compileJava
:processResources NO-SOURCE
:classes
:run
Hello World!
```

Works just as before. But let's kill incremental build state again:

```text
$ ./gradlew clean
:clean
```

And if we rebuild again with cache enabled:

```text
$ ./gradlew --build-cache run
Build cache is an incubating feature.
Using directory (~/.gradle/caches/3.6-20170314000035+0000/build-cache) as local build cache, push is enabled.
:compileJava FROM-CACHE
:processResources NO-SOURCE
:classes UP-TO-DATE
:run
Hello World!
```

Notice that `:compileJava` is now `FROM-CACHE`, i.e. it is loaded from the cache.

Check the contents of the cache:

```text
$ ls ~/.gradle/caches/3.6-20170314000035+0000/build-cache
1930b2951f52485ad95c7064087e8301 build-cache.lock

$ tar -tvf ~/.gradle/caches/3.6-20170314000035+0000/build-cache/1930b2951f52485ad95c7064087e8301
  -rw-r--r-- 0/0             370 2017-03-14 18:57 METADATA
  drwxr-xr-x 0/0               0 2017-03-14 18:57 property-destinationDir/
  -rw-r--r-- 0/0             519 2017-03-14 18:57 property-destinationDir/Hello.class
```

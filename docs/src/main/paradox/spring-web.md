# Spring Web

Spring 5.0 introduced compatibility with [Reactive Streams](http://reactive-streams.org), a library interoperability standardization effort co-lead by Lightbend (with Akka Streams) along with Kaazing, Netflix, 
Pivotal, Red Hat, Twitter and many others.

Thanks to adopting Reactive Streams, multiple libraries can now inter-op since the same interfaces are implemented by 
all these libraries. Akka Streams by-design, hides the raw reactive-streams types from end-users, since it allows for
detaching these types from RS and allows for a painless migration to [`java.util.concurrent.Flow`](http://download.java.net/java/jdk9/docs/api/java/util/concurrent/Flow.html) which was introduced in Java 9.

This Alpakka module makes it possible to directly return a `Source` in your Spring Web endpoints.

### Reported issues

[Tagged issues at Github](https://github.com/akka/alpakka/labels/p%3Aspring-web)


## Artifacts

@@dependency [sbt,Maven,Gradle] {
  group=com.lightbend.akka
  artifact=akka-stream-alpakka-spring-web_$scalaBinaryVersion$
  version=$version$
}

## Usage

Using Akka Streams in Spring Web (or Boot for that matter) is very simple, as Alpakka provides autoconfiguration to the
framework, which means that Spring is made aware of Sources and Sinks etc. 

All you need to do is include the above dependency (`akka-stream-alpakka-spring-web`), start your app as usual:

Java
: @@snip ($alpakka$/spring-web/src/test/java/akka/stream/alpakka/spring/web/DemoApplication.java) { #use }


And you'll be able to return Akka Streams in HTTP endpoints directly:


Java
: @@snip ($alpakka$/spring-web/src/test/java/akka/stream/alpakka/spring/web/SampleController.java) { #use }

Both `javadsl` and `scaladsl` Akka Stream types are supported.

In fact, since Akka supports Java 9 and the `java.util.concurrent.Flow.*` types already, before Spring, you could use it
to adapt those types in your applications as well.

### The provided configuration

The automatically enabled configuration is as follows:

Java
: @@snip ($alpakka$/spring-web/src/main/java/akka/stream/alpakka/spring/web/SpringWebAkkaStreamsConfiguration.java) { #configure }

In case you'd like to manually configure it slightly differently.

## Shameless plug: Akka HTTP 

While the integration presented here works, it's not quite the optimal way of using Akka in conjunction with serving HTTP apps.
If you're new to reactive systems and picking technologies, you may want to have a look at [Akka HTTP](https://doc.akka.io/docs/akka-http/current/scala/http/).

If, for some reason, you decided use Spring MVC this integration should help you achieve the basic streaming scenarios though.

# MQTT

The MQTT connector provides an Akka Stream source, sink and flow to connect to MQTT servers. It is based on [Eclipse Paho](https://www.eclipse.org/paho/clients/java/). 

### Reported issues

[Tagged issues at Github](https://github.com/akka/alpakka/labels/p%3Amqtt)

## Artifacts

@@dependency [sbt,Maven,Gradle] {
  group=com.lightbend.akka
  artifact=akka-stream-alpakka-mqtt_$scalaBinaryVersion$
  version=$version$
}

## Setup

@@@ warning { title='Use delayed stream restarts' }
Note that the following examples do not provide any connection management and are designed to get you going quickly. Consider empty client IDs to auto-generate unique identifiers and the use of [delayed stream restarts](https://doc.akka.io/docs/akka/current/stream/stream-error.html?language=scala#delayed-restarts-with-a-backoff-stage). The underlying Paho library's auto-reconnect feature [does not handle initial connections by design](https://github.com/eclipse/paho.mqtt.golang/issues/77).
@@@

First we need to define various settings, that are required when connecting to an MQTT server.

Scala
: @@snip ($alpakka$/mqtt/src/test/scala/akka/stream/alpakka/mqtt/scaladsl/MqttSourceSpec.scala) { #create-connection-settings }

Java
: @@snip ($alpakka$/mqtt/src/test/java/akka/stream/alpakka/mqtt/javadsl/MqttSourceTest.java) { #create-connection-settings }

Here we used @scaladoc[MqttConnectionSettings](akka.stream.alpakka.mqtt.MqttConnectionSettings$) factory to set the address of the server, client ID, which needs to be unique for every client, and client persistence implementation (@extref[MemoryPersistence](paho-api:org/eclipse/paho/client/mqttv3/persist/MemoryPersistence)) which allows to control reliability guarantees.

Most settings are passed on to Paho's @extref[MqttConnectOptions](paho-api:org/eclipse/paho/client/mqttv3/MqttConnectOptions) and documented there. 


## Reading from MQTT

Then let's create a source that is going to connect to the MQTT server upon materialization and receive messages that are sent to the subscribed topics.

Scala
: @@snip ($alpakka$/mqtt/src/test/scala/akka/stream/alpakka/mqtt/scaladsl/MqttSourceSpec.scala) { #create-source }

Java
: @@snip ($alpakka$/mqtt/src/test/java/akka/stream/alpakka/mqtt/javadsl/MqttSourceTest.java) { #create-source }


And finally run the source.

Scala
: @@snip ($alpakka$/mqtt/src/test/scala/akka/stream/alpakka/mqtt/scaladsl/MqttSourceSpec.scala) { #run-source }

Java
: @@snip ($alpakka$/mqtt/src/test/java/akka/stream/alpakka/mqtt/javadsl/MqttSourceTest.java) { #run-source }

This source has a materialized value (@scaladoc[Future](scala.concurrent.Future) in Scala API and @extref[CompletionStage](java-api:java/util/concurrent/CompletionStage) in Java API) which is completed when the subscription to the MQTT broker has been completed.

MQTT automatically acknowledges messages back to the server once they are passed downstream. The `atLeastOnce` source allow users to acknowledge the messages anywhere downstream.
Please note that for manual acks to work `CleanSession` should be set to false and `MqttQoS` should be `AtLeastOnce`.

Scala
: @@snip ($alpakka$/mqtt/src/test/scala/akka/stream/alpakka/mqtt/scaladsl/MqttSourceSpec.scala) { #create-source-with-manualacks }

Java
: @@snip ($alpakka$/mqtt/src/test/java/akka/stream/alpakka/mqtt/javadsl/MqttSourceTest.java) { #create-source-with-manualacks }


The `atLeastOnce` source returns @scaladoc[MqttCommittableMessage](akka.stream.alpakka.mqtt.scaladsl.MqttCommittableMessage) so you can acknowledge them by calling `messageArrivedComplete`.

Scala
: @@snip ($alpakka$/mqtt/src/test/scala/akka/stream/alpakka/mqtt/scaladsl/MqttSourceSpec.scala) { #run-source-with-manualacks }

Java
: @@snip ($alpakka$/mqtt/src/test/java/akka/stream/alpakka/mqtt/javadsl/MqttSourceTest.java) { #run-source-with-manualacks }


## Publishing to MQTT

To publish messages to the MQTT server create a sink and run it.

Scala
: @@snip ($alpakka$/mqtt/src/test/scala/akka/stream/alpakka/mqtt/scaladsl/MqttSourceSpec.scala) { #run-sink }

Java
: @@snip ($alpakka$/mqtt/src/test/java/akka/stream/alpakka/mqtt/javadsl/MqttSourceTest.java) { #run-sink }


The QoS and the retained flag can be configured on a per-message basis.

Scala
: @@snip ($alpakka$/mqtt/src/test/scala/akka/stream/alpakka/mqtt/scaladsl/MqttSourceSpec.scala) { #will-message }

Java
: @@snip ($alpakka$/mqtt/src/test/java/akka/stream/alpakka/mqtt/javadsl/MqttSourceTest.java) { #will-message }


It is also possible to connect to the MQTT server in bidirectional fashion, using a single underlying connection (and client ID). To do that create an MQTT flow that combines the functionalities of an MQTT source and an MQTT sink.

Scala
: @@snip ($alpakka$/mqtt/src/test/scala/akka/stream/alpakka/mqtt/scaladsl/MqttFlowSpec.scala) { #create-flow }

Java
: @@snip ($alpakka$/mqtt/src/test/java/akka/stream/alpakka/mqtt/javadsl/MqttFlowTest.java) { #create-flow }


Run the flow by connecting a source of messages to be published and a sink for received messages.

Scala
: @@snip ($alpakka$/mqtt/src/test/scala/akka/stream/alpakka/mqtt/scaladsl/MqttFlowSpec.scala) { #run-flow }

Java
: @@snip ($alpakka$/mqtt/src/test/java/akka/stream/alpakka/mqtt/javadsl/MqttFlowTest.java) { #run-flow }

### Running the example code

The code in this guide is part of runnable tests of this project. You are welcome to edit the code and run it in sbt.

> Test code requires a MQTT server running in the background. You can start one quickly using docker:
>
> `docker-compose up mqtt`

Scala
:   ```
    sbt
    > mqtt/testOnly *.MqttSourceSpec
    ```

Java
:   ```
    sbt
    > mqtt/testOnly *.MqttSourceTest
    ```

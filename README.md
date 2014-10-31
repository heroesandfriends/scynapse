Welcome to the Scynapse [![Build Status](https://secure.travis-ci.org/thenewmotion/scynapse.png)](http://travis-ci.org/thenewmotion/scynapse)
==================================

This project aims to bring more natural experience with Axon Framework (http://www.axonframework.org) in Scala land.

scynapse-akka
--------------

`scynapse-akka` module provides facilities that make it easier to
integrate Axon components with Akka.

### Subscribing actors to events

`AxonEventBusExtension` allows to subscribe Akka actors to events
published on Axon event bus.  It is implemented as an
[Akka extension](http://doc.akka.io/docs/akka/2.3.6/scala/extending-akka.html)
that manages event bus subscriptions.

In order to subscribe an `ActorRef` to the event bus you should first
initialize an `AxonAkkaBridge`. Here is an example(using Spring):

    import org.axonframework.eventhandling.EventBus
    import com.thenewmotion.scynapse.akka.AxonEventBusExtension

    val eventBus = getBean("eventBus", classOf[EventBus])
    val axonAkkaBridge = AxonEventBusExtension(actorSystem) forEventBus eventBus

Then `axonAkkaBridge` can be used to subscribe actors to the Event bus:

    val eventListener: ActorRef = context.actorOf(...)
    axonAkkaBridge subscribe eventListener

After that `eventListener` actor will receive all events published to
the event bus as simple messages.

To unsubscribe actor from the event bus:

    axonAkkaBridge unsubscribe eventListener

If actor is terminated unsubscription occurs automatically.


### Sending commands from actors

In order to make sending domain commands from Akka components easier a
`CommandGatewayActor` was introduced. It is just a simple actor
interface for the Axon `CommandBus` that dispatches all messages it
receives to a command bus. A result returned by the command handler
(if any) is sent back to the original command sender.

Example usage (again, we're using Spring context here):

    import org.axonframework.commandhandling.CommandBus
    import com.thenewmotion.scynapse.akka.CommandGatewayActor

    val commandBus = getBean("commandBus", classOf[CommandBus])
    val cmdGateway = actorSystem.actorOf(CommandGatewayActor.props(commandBus))

    ...

    cmdGateway ! CreateOrder(...)

# Reactor + Spring Integration Quickstart

This quickstart application demonstrates how to integrate Reactor into a Spring Integration application. It primarily focuses on TCP ingestion and publishing messages into a Channel after decoding in the Reactor `ReactorTcpInboundChannelAdapter`.

### Setup

The classes are javadoc'ed and explain their purpose in the application. This app builds on several important Spring projects:

- Spring Boot powers the app and ties everything together in a simple and efficient way.
- Spring Integration provides the EIP abstractions for processing messages.
- Spring MVC provides a simple @RestController that provides some visibility into the running app.
- Reactor provides the horsepower to process over 1M msgs/sec via the Netty and RingBuffer-based TCP support.

To load test the application, simple send it length-field delimited data. The content of the messages in the default configuration is ignored since there's no message processing being done in this simple example. In your application, you'll want to add a real `Codec` and `MessageHandler` that do more than increment a counter.

There is a class in the tests called `WriteLengthFieldDataFileApp` which will write out a test file to `src/main/resources/data.bin` that is length-field-delimited random data. You can then use the very efficient `LoadTestClient`, also located in the test folder, to send that data to the server using the most efficient method possible. Benchmarking shows that using tools like `netcat`, or even a simple raw Socket-based app, will yield lower throughput than using the included `FileChannel`-based client. It's easy to get into a situation where you're actually load testing your load tester rather than the server-side components.

### Running

__NOTE:__ *If using Java 8, you must be running with ea build 128 or later. Earlier builds may or may not work.*

To run the Spring Integration components, fire up the app using the the `QuickStartApplication` class' Java main method, or by running it from the command line `mvn spring-boot:run`.

Requires __-Dspring.profiles.active=reactor__ or __-Dspring.profiles.active=si__ to run with reactor or native SI TCP adapters respectively.

Also requires one of the following profiles for the type of channel to be used downstream:

- channel.simple
- channel.direct
- channel.ringBuffer
- channel.threadPoolExecutor

If using the `reactor` profile, one of the following profiles must be provided to determine the type of dispatcher within the TCP adapter itself:

- dispatcher.sync
- dispatcher.ringBuffer
- dispatcher.threadPoolExecutor

For example, provide the following to use the Reactor TCP adapter with a RingBuffer dispatcher and a simple Channel (likely the most performant combination):

__-Dspring.profiles.active=reactor,dispatcher.ringBuffer,channel.simple__

### Results

Running on a 2.7GHz Intel quad-core i7 and Mac OS X 10.8.4 and JDK 1.7.0_21-b12, some example results look like this:

Dispatcher | Channel | TCP | messages/sec
-----------|---------|-----|-------------
RingBuffer | Simple | Reactor | 1.91M
RingBuffer | Direct | Reactor | 1.77M
Synchronous | Direct | Reactor | 1.77M
ThreadPoolExecutor | Direct | Reactor | 1.77M
Synchronous | RingBuffer | Reactor | 1.38M
 | Direct | SI | 474k
 | Simple | SI | 537k
 | RingBuffer | SI | 439k
 | ThreadPoolExecutor | SI | 422k

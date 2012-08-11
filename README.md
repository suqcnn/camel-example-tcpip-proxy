Using Fuse ESB to Manage Controller Failover
============================================

Created: June 28, 2012
<a href="https://github.com/markbrooks">Mark Brooks</a>

Updated: August 8, 2012
<a href="https://github.com/scranton">Scott Cranton</a>

This document describes a simple prototype that demonstrates the use of Fuse ESB to manage TCP/IP controller failover.

Here is the design:

![Design Image](/images/design.png "Design Image")

Client sends TCP/IP messages to ESB acting as a proxy. A failover-capable router pickups up messages and sends them
to Controller 1 using a TCP/IP service call. Controller 1 stamps the message as having been processed and returns it
to the router, and then back to the Client application.

If Controller 1 fails, the router fails over to Controller 2. For purposes of a fast prototype, Controller 1 and
Controller 2 were run as simple Camel-Mina services run from the command line (external to the ESB) so they were easy to
kill to demonstrate failover. In practice, additional Fuse ESB containers would be ideal to host controller services.

Software Requirements:

* Fuse ESB
* Fuse IDE (optional)
* JDK 1.6
* Maven 2.2 or 3.0

Download and Install Fuse ESB
-----------------------------

* Download Fuse ESB at http://fusesource.com/products/fuse-esb-enterprise/
* I recommend downloading the .zip or .tar.gz archives rather than the full platform installers.
* Unzip the archive in a directory without spaces in its path.

Download and Install Fuse IDE (optional, not required for this demo)

* Download Fuse IDE at http://fusesource.com/products/fuse-ide/
* Unzip the archive in a directory without spaces in its path.

Project Layout
--------------

There are three modules: applications, controllers and router. Here is the layout in the file system:

* FuseESB-RouterDemo
    * client
    * controller
    * router

All of the projects’ build and deploy steps can be performed from Fuse IDE or from the command line; I’ll show it from
the command line here as it is the simplest to describe, and then show Fuse IDE afterwards.

Start Fuse ESB
--------------

Run the command `bin/fuseesb` in the root directory of Fuse ESB. Here is what the console looks like:

![Console Image](/images/console.png "Console Image")

Building the projects
---------------------

Launch a new terminal session, and run the command `mvn clean install` in the root directory of top level project.
Make sure all projects build successfully.

Deploy the Router into Fuse ESB
-------------------------------

In the Fuse ESB terminal session run the command:

    FuseESB:karaf@root> features:install camel-mina
    FuseESB:karaf@root> features:install camel-groovy
    FuseESB:karaf@root> osgi:install -s mvn:com.fusesource.demo/router/1.0.0-SNAPSHOT

You should see output like this, with a Bundle ID generated by the deployment:

    FuseESB:karaf@root> osgi:install -s mvn:com.fusesource.demo/router/1.0.0-SNAPSHOT
    Bundle ID: 225
    FuseESB:karaf@root>

Launch the Controllers
----------------------

Open two new terminal sessions and switch to the root of the controllers project in both. In one session, execute this
command to launch Controller 1:

    controller$ mvn -P controller1

And in the other execute the command to launch Controller 2:

    controller$ mvn -P controller2

Launch Client
-------------

Open a new terminal sessions and switch to the root of the applications project. In the session, execute this command
to launch the test Client application:

    client$ mvn camel:run

You should see the Client sending and receiving messages like below. Notice the response has a <controller> element,
that initially contains 'Controller 1'

    17:44:47 INFO  Apache Camel 2.9.0.fuse-7-061 (CamelContext: camel-1) started in 0.573 seconds
    17:44:47 INFO  Sending message 1 with data: <data><messageNumber>1</messageNumber><timestamp>17:44:47.348</timestamp></data>
    17:44:47 INFO  Received 1 message: <data><messageNumber>1</messageNumber><timestamp>17:44:47.348</timestamp><controller>Controller 1</controller></data>
    17:44:48 INFO  Sending message 2 with data: <data><messageNumber>2</messageNumber><timestamp>17:44:48.335</timestamp></data>
    17:44:48 INFO  Received 2 message: <data><messageNumber>2</messageNumber><timestamp>17:44:48.335</timestamp><controller>Controller 1</controller></data>
    17:44:49 INFO  Sending message 3 with data: <data><messageNumber>3</messageNumber><timestamp>17:44:49.335</timestamp></data>
    17:44:49 INFO  Received 3 message: <data><messageNumber>3</messageNumber><timestamp>17:44:49.335</timestamp><controller>Controller 1</controller></data>
    17:44:50 INFO  Sending message 4 with data: <data><messageNumber>4</messageNumber><timestamp>17:44:50.334</timestamp></data>
    17:44:50 INFO  Received 4 message: <data><messageNumber>4</messageNumber><timestamp>17:44:50.334</timestamp><controller>Controller 1</controller></data>
    17:44:51 INFO  Sending message 5 with data: <data><messageNumber>5</messageNumber><timestamp>17:44:51.334</timestamp></data>
    17:44:51 INFO  Received 5 message: <data><messageNumber>5</messageNumber><timestamp>17:44:51.334</timestamp><controller>Controller 1</controller></data>
    17:44:52 INFO  Sending message 6 with data: <data><messageNumber>6</messageNumber><timestamp>17:44:52.334</timestamp></data>
    17:44:52 INFO  Received 6 message: <data><messageNumber>6</messageNumber><timestamp>17:44:52.334</timestamp><controller>Controller 1</controller></data>

Kill the Controller 1 session any way you like, and you should see uninterrupted message flow, with Controller 2
handling the messages (note the <controller> changes to 'Controller 2' at message 12):

    17:44:56 INFO  Sending message 10 with data: <data><messageNumber>10</messageNumber><timestamp>17:44:56.334</timestamp></data>
    17:44:56 INFO  Received 10 message: <data><messageNumber>10</messageNumber><timestamp>17:44:56.334</timestamp><controller>Controller 1</controller></data>
    17:44:57 INFO  Sending message 11 with data: <data><messageNumber>11</messageNumber><timestamp>17:44:57.335</timestamp></data>
    17:44:57 INFO  Received 11 message: <data><messageNumber>11</messageNumber><timestamp>17:44:57.335</timestamp><controller>Controller 1</controller></data>
    17:44:58 INFO  Sending message 12 with data: <data><messageNumber>12</messageNumber><timestamp>17:44:58.334</timestamp></data>
    17:44:58 INFO  Received 12 message: <data><messageNumber>12</messageNumber><timestamp>17:44:58.334</timestamp><controller>Controller 2</controller></data>
    17:44:59 INFO  Sending message 13 with data: <data><messageNumber>13</messageNumber><timestamp>17:44:59.334</timestamp></data>
    17:44:59 INFO  Received 13 message: <data><messageNumber>13</messageNumber><timestamp>17:44:59.334</timestamp><controller>Controller 2</controller></data>

Restart Controller 1 and you’ll see it take control back from Controller 2 (this is user-defined behavior in the
configuration of the failover-load balancer)

    17:45:26 INFO  Sending message 40 with data: <data><messageNumber>40</messageNumber><timestamp>17:45:26.333</timestamp></data>
    17:45:26 INFO  Received 40 message: <data><messageNumber>40</messageNumber><timestamp>17:45:26.333</timestamp><controller>Controller 2</controller></data>
    17:45:27 INFO  Sending message 41 with data: <data><messageNumber>41</messageNumber><timestamp>17:45:27.334</timestamp></data>
    17:45:27 INFO  Received 41 message: <data><messageNumber>41</messageNumber><timestamp>17:45:27.334</timestamp><controller>Controller 2</controller></data>
    17:45:28 INFO  Sending message 42 with data: <data><messageNumber>42</messageNumber><timestamp>17:45:28.335</timestamp></data>
    17:45:28 INFO  Received 42 message: <data><messageNumber>42</messageNumber><timestamp>17:45:28.335</timestamp><controller>Controller 1</controller></data>
    17:45:29 INFO  Sending message 43 with data: <data><messageNumber>43</messageNumber><timestamp>17:45:29.335</timestamp></data>
    17:45:29 INFO  Received 43 message: <data><messageNumber>43</messageNumber><timestamp>17:45:29.335</timestamp><controller>Controller 1</controller></data>
    17:45:30 INFO  Sending message 44 with data: <data><messageNumber>44</messageNumber><timestamp>17:45:30.335</timestamp></data>
    17:45:30 INFO  Received 44 message: <data><messageNumber>44</messageNumber><timestamp>17:45:30.335</timestamp><controller>Controller 1</controller></data>

To inspect the projects using Fuse IDE, launch Fuse IDE and import the three projects as “Existing Maven Projects”:

![Import Maven Projects Image](/images/import-maven-projects.jpg "Import Maven Projects Image")

Navigate to the router project’s camel-context.xml file to see a visual editor for the integration:

![Camel Context Image](/images/camel-context.jpg "Camel Context Image")

Switch to the Source tab to see the definition of the integration:

```xml
<camelContext xmlns="http://camel.apache.org/schema/spring">
    <route id="route1">
        <from uri="mina:tcp://0.0.0.0:9000?textline=true&amp;sync=true"/>
        <loadBalance>
            <failover maximumFailoverAttempts="-1" roundRobin="false"/>
            <to uri="mina:tcp://localhost:9001?textline=true&amp;sync=true"/>
            <to uri="mina:tcp://localhost:9002?textline=true&amp;sync=true"/>
        </loadBalance>
    </route>
</camelContext>
```

Links to additional documentation
---------------------------------

* http://fusesource.com/documentation/
* http://fusesource.com/docs/esbent/7.0/camel_eip/IntroToEIP-OP.html
* http://fusesource.com/docs/esbent/7.0/camel_comp_ref/Intro-ComponentsList.html

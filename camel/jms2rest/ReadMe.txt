Jms2Rest
========

Shows how to write an adapter from the Event driven world to the REST world.

The idea is to have a REST Service for persons that is available on HTTP.
We now want to also receive updates from a JMS queue. So camel is used to implement an adapter from
JMS events to REST calls.

The route personJms2rest does the following steps:
- Receive an person XML document on a queue
- Extract the person id from the XML document and store it into a Header
- Build the REST URL using a fixed part and the person id header
- Call the REST Service with the XML Payload

To make testing easier there is a second route "file2jms" that listens to a folder "in" below the karaf directory and sends a message for each file
you put in there. 

Prerequisites
-------------

As the example needs a rest service to call you need to install the server part of the personservice example into karaf first.

Build
-----

Go to the example project directory and type

> mvn clean install

Run in Karaf
------------

features:addUrl mvn:org.apache.activemq/activemq-karaf/5.4.0/xml/features
features:addurl mvn:org.apache.camel.karaf/apache-camel/2.9.0/xml/features
features:install  camel-blueprint camel-jms camel-http
features:install activemq-blueprint
activemq:create-broker --type blueprint 
install -s mvn:net.lr.tutorial.karaf.camel/example-jms2rest/1.0-SNAPSHOT

What did we install
-------------------

We first added the feature files for cmael and activemq.
Then we installed the necessary features for a local ActiveMQ broker and our example.

The create broker command creates a blueprint xml file in the deploy folder that starts and configures a broker. It also initializes a
PoolingConnectionFactory as an OSGi service that we need for our example.

As a last step we install the example project. The example also starts with a blueprint context that loads camel and the Jms2RestRoute RouteBuilder.

Test
----

To test the route we simply copy the file src/test/resources/person1.xml into the "in" folder below that karaf directory. This file will first be sent to the 
jms queue "person". There the second route picks up the message, transforms it into the rest call and calls the rest service.

If the rest service is not installed you will see a 404 http error in the karaf log (log:display).

If the rest service is installed it should print "update request received".

 
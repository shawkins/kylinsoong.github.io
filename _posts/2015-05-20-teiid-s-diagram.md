---
layout: blog
title:  "Teiid Code Analysis - Sequence Diagrams"
date:   2015-05-20 21:20:00
categories: teiid
permalink: /teiid-s-diagram
author: Kylin Soong
duoshuoid: ksoong2015052002
excerpt: Teiid Sequence Diagrams Contains a series Sequence diagrams
---

## How a connection execute the query

The following sequence diagram shows how query sql `SELECT * FROM Marketdata` be send to Teiid Server

![teiid-execute-query]({{ site.baseurl }}/assets/blog/teiid-execute-query.png)

Start from left to right:

* StatementImpl **executeQuery** receive the query sql `SELECT * FROM Marketdata`
* StatementImpl assemble a `org.teiid.client.RequestMessage` base query sql, then invoke DQP Proxy's **executeRequest**
* RemoteInvocationHandler's **invoke** method be invoked
* RemoteInvocationHandler assemble a `org.teiid.net.socket.Message` base on passed `org.teiid.client.RequestMessage`, then SocketServerInstanceImpl's **send** method be invoked
* OioObjectChannel's **write** methd be invoked
* ObjectOutputStream which come from socket **writeObject** `org.teiid.net.socket.Message`

## How Teiid Server handle query request

![teiid-server-hanlde-request]({{ site.baseurl }}/assets/blog/teiid-server-hanlde-request.png)

* SSLAwareChannelHandler

While Teiid Server Started, it will create a SocketListener which listen for new connection requests and create a SocketClientConnection for each connection request.

The SocketListener bootstrap netty server worker threads with port number 31000, SSLAwareChannelHandler is the handler used by netty server side, it's `messageReceived()` can be thought as teiid server side entry point. Once a client request in, this method be first invoked. 

* SocketClientInstance

Sockets implementation of the communication framework class representing the server's view of a client connection. Implements the server-side of the sockets messaging protocol. The client side of the protocol is implemented in SocketServerInstance.

It's `processMessagePacket()` method be invoked for handling income request message.

* DQPWorkContext & ServerWorkItem

ServerWorkItem is a runnable class, it encapulates income request message in it's constructor:

~~~
    public ServerWorkItem(ClientInstance socketClientInstance, Serializable messageKey, Message message, ClientServiceRegistryImpl server) {
                this.socketClientInstance = socketClientInstance;
                this.messageKey = messageKey;
                this.message = message;
                this.csr = server;
        }
~~~

DQPWorkContext is a filed of SocketClientInstance, DQPWorkContext's `runInContext()` method be invoked, the constructed ServerWorkItem passed as a argument. Then ServerWorkItem's `run()` method be invoked.

## How [ConnectorWork](https://github.com/teiid/teiid/blob/master/engine/src/main/java/org/teiid/dqp/internal/datamgr/ConnectorWork.java) interact with Teiid Translator Execution API

Client execute JDBC query sql be convert to a `Command` object, pass it to Engine DQP Process layer as DataTierTupleSource, then translator layer is necessary for translating data as below figure:

![teiid connector logic]({{ site.baseurl }}/assets/blog/connectorworkitem.jpg)

[org.teiid.dqp.internal.datamgr.ConnectorWork](https://github.com/teiid/teiid/blob/master/engine/src/main/java/org/teiid/dqp/internal/datamgr/ConnectorWork.java) is a internal used interface, but it's critical in Teiid Engine, it defined methods as below figure to operate `Teiid Connectors` layer.

![teiid connector work]({{ site.baseurl }}/assets/blog/ConnectorWork.gif)

[org.teiid.dqp.internal.datamgr.ConnectorWorkItem](https://github.com/teiid/teiid/blob/master/engine/src/main/java/org/teiid/dqp/internal/datamgr/ConnectorWorkItem.java) implements [org.teiid.dqp.internal.datamgr.ConnectorWork](https://github.com/teiid/teiid/blob/master/engine/src/main/java/org/teiid/dqp/internal/datamgr/ConnectorWork.java), we will look into it's method's in the following.

### execute()

Its main function is create a translator Execution, then invoke Execution's execute() method, the completed procedure like:

* Create Data Source Connection base on JCA connector implementation
* Create translator Execution with translator ExecutionFactory
* Invoke translator Execution's execute() method

### more()

Usually this methods be invoked after execute(), this method will invoke Execution's next() method repeatedly, return list of rows existed in data source.

# Architecture Diagram

For a constrained Internet of Things (IoT) application such at this one, a publish/subscribe design pattern using the MQTT protocol seems to be a perfect fit.

Quoting the official MQTT 3.1.1 specification:

> MQTT is a Client Server publish/subscribe messaging transport protocol. It is light weight, open, simple, and designed so as to be easy to implement. These characteristics make it ideal for use in many situations, including constrained environments such as for communication in Machine to Machine (M2M) and Internet of Things (IoT) contexts where a small code footprint is required and/or network bandwidth is at a premium.

The publish/subscribe pattern (also known as pub/sub) provides an alternative to traditional client/server architecture. In the client/server model, a client communicates directly with an endpoint. The pub/sub model decouples the client that sends a message (the publisher) from the client or clients that receive the messages (the subscribers). The publishers and subscribers never contact each other directly. In fact, they are not even aware that the other exists. The connection between them is handled by a third component (the broker). The job of the broker is to filter all incoming messages and distribute them correctly to subscribers.

The most important aspect of pub/sub is the decoupling of the publisher of the message from the recipient (subscriber). This decoupling has several dimensions:
- Space decoupling: Publisher and subscriber do not need to know each other (for example, no exchange of IP address and port).
- Time decoupling: Publisher and subscriber do not need to run at the same time.
- Synchronization decoupling: Operations on both components do not need to be interrupted during publishing or receiving.

Applying this design pattern to our use case, and from the architecture diagram above:
1. The dashboard is a client that subscribes to the `vehicle/position` topic
2. The drones publish their geo-location data to the same topic on the broker
3. The broker distribute all the messages to the dashboard, so the dashboard can store and post-process all the data in real-time

```
+ - - - - +                + - - - - - - - - - +        
| vehicle | -- publish --> |                   |       
+ - - - - +                |                   |         + - - - - - +         + - - - - - +
                           |                   |         | Redis DB  |         |           |
+ - - - - +                |    MQTT Broker    |         + - - - - - +         |           |
| vehicle | -- publish --> |                   |                ^              | Dashboard |
+ - - - - +                |   vehicle/position  |                |              |           |
                           |                   | Subscribe + - - - - - +       |           |
+ - - - - +                |                   |---------> |  Server   |<------|           |
| vehicle | -- publish --> |                   |           + - - - - - +       + - - - - - +
+ - - - - +                + - - - - - - - - - +
```

# Technologies 
- MQTT  (Light weight payload)
- NodeJS (Best server framework for async operation)
- Redis (In memeory db)
- ReactJS (Client side framework for developing dashboard)

# Assumptions
- The vehicle firmware exists. I focused on determining how the firmware will send information to the backend (i.e. the Server). They publish their latitude and longitude along other metadata (uuid, name ) to the MQTT broker.
- For the sake of simplicity, I used a free MQTT broker (over HTTP, HTTPS and WebSockets).
- Vehicle will make update request to server in every 10 secs.
- At every event ( vehicle update position) server will check the differnce of position on be behalf of current position and last position. If differnce is not more the 1 meter then will store stop = true in redis.
- Dashboard will make a request to server to get vehicle current postion. 


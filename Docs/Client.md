## This is an extensive docs to the Client side of the Project, for Developers

### Project Definition

This project is a Chat Service simulation, with Whatsapp features. People can "Log in" by running the client side and the talk to other people, connecting to them by their `port number`. 

GUI-wise is a very simple project, must of it's features running on the background. As a Client you can log in(as long as the Server is open), and start a chat with other User that is currently loggeed in.

It has Whatsapp-alike features, for instance, the message `STATUS`(as in "not sent", "sent", "visualized"...).


### Project InfraStructure

#### General
Project is divided in two sub-modules, here we are talking about the Client Side.

We are using MVC as a pattern, since we have the GUI and there are some features that modify it(like the Message Status Update, Openning new screens, changing text...). Don't be confused by the name of the classes, though. The Models folder has the domain classes, our real Model(for the MVC) is in the Client folder in this case. Our view is in 'Resources' and the Controllers are really in 'Controllers'.

![](https://raw.githubusercontent.com/Gui-Lima/ZipZap/master/Res/ProjectStructureClient.png "Project General Structure")


Also for the Messages, we have some important types, that will mold the way the Server and Client respond to Input or Output data it's sending or receiving. This Enum is pretty self-explanatory.

```java
public enum Type{
    MESSAGE_DELETE,
    CONNECT_TO,
    RECEIVE_CONNECTION,
    MESSAGE_SEND,
    STATUS_UPDATE,
    NOTIFICATION,
    FINISH
}
```
When you see the functions that use these Types it should be easy to understand what's doing, without further ado.


#### Running the Client
When you run the client, and click the Button "Connect", it stablishes connection to the server, like a login. What happens in the background is that a Connection is formed. A connection is basically the mid-term between receiving messages and changing the GUI with these messages. Who really receives messages, though, is the Connection_Receiver thread, that is started at the same time the Connection is stablished.

From this point on, the Connection_Receiver is responsible for receiving data via the Input Stream. The most important thing here is the Pattern Observer/Sender-Subscriber. The point is the GUI have to be notified to do something depending on what happens on the Connection. To Notify it, we set it to observe the messages the Connection sends/receives and notify it by the subscriber pattern. Let us explain how this happens.

```java
public interface Observer {
    void notifyConnectionEstablished(int port);

    void notifyUserConnected(int port);

    void notifyMessageReceived(Message message);

    void notifyMessageDeletion(Message message);

    void notifyStatusUpdate(Message message);

    void notifyServerClosed(Message message);
}
```

This interface does the job, by making the controllers inherit it, we can add them as subscribers to the Connection class, that notify every subscriber everytime a message comes

```java
void notifySomethingHappened(Message message) {
        System.out.println("Something happened!");
        for(Observer observer : listeners) {
            if(message.getType() == Type.CONNECT_TO) {
                System.out.println(" It was a connection tryout");
                observer.notifyConnectionEstablished(message.getToPort());
            }
            else if(message.getType()== Type.RECEIVE_CONNECTION) {
                System.out.println(" It was a Connection tryout reciever");
                observer.notifyUserConnected(message.getFromPort());
            }
            else if(message.getType() == Type.MESSAGE_SEND) {
                System.out.println(" It was a Message to" + observer.toString());
                observer.notifyMessageReceived(message);
            }
            else if(message.getType() == Type.MESSAGE_DELETE){
                System.out.println("It was a deletion, wow ");
                observer.notifyMessageDeletion(message);
            }
            else if(message.getType() == Type.STATUS_UPDATE){
                System.out.println("We received a status update");
                observer.notifyStatusUpdate(message);
            }
            else if(message.getType() == Type.FINISH){
                System.out.println("Servidor fechando =(");
                observer.notifyServerClosed(message);
            }
        }
    }
```

With this, we have a robust enough infra so that we can easily pass messages from the InputStream to the GUI's(Messages Received).

The other face of the Client is sending messages. But the heavy load on this is all on the server, we basically just write a Message in the OutputStream in the connection class and the Server deals with it. `Little changes could be made here to allow multiple chats or groups`.

#### Handling Messages 
As said, the client deals more with message receiving. The Connection_Receiver class does this in a very simple way. It is a Thread that starts when the user clicks "Connect" and basically just waits for data in the InputStream, coming from the server.

```java
while(!stopped){
    try {
        String me = this.input.readUTF();
        Message message = new Message(me);
        if(message.getType() == Type.FINISH){
            this.stopped = true;
        }
        this.connection.notifySomethingHappened(message);
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

Everytime some message comes from the InputStream, it sends it back to the Connection and it goes the way we talked on the previous section, notifying the Controllers.

## This is an extensive docs to the Server side of the Project, for Developers

### Project Definition

This project is a Chat Service simulation, with Whatsapp features. People can "Log in" by running the client side and the talk to other people, connecting to them by their `port number`. 

GUI-wise is a very simple project, must of it's features running on the background. As a Server you can only start the server, so people can start logging in and talking to others, and a `"Users Connected Visualizer"`.

It has Whatsapp-alike features, for instance, the message `STATUS`(as in "not sent", "sent", "visualized"...).


### Project InfraStructure

#### General
Project is divided in two sub-modules, here we are talking about the Server Side.

We are using MVC as a pattern, since we have the GUI and there are some features that modify it(like the Message Status Update). Don't be confused by the name of the classes, though. The Models folder has the domain classes, our real Model(for the MVC) is in the Server folder in this case. Our view is in 'Resources' and the Controllers are really in 'Controllers'.

![](https://raw.githubusercontent.com/Gui-Lima/ZipZap/master/Res/ProjectStructure.png "Project General Structure")


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


#### Running the Server
When you run the application(aka click "Start" on the GUI) it basically starts a thread. This thread awaits connections from other people, and it is the Server_Run class.

![](https://raw.githubusercontent.com/Gui-Lima/ZipZap/master/Res/Server_runMain.png "Project General Structure")

Notice how it runs an true-loop, and stops on this line 

```java
clientSocket = this.serverSocket.accept();
```

Until someone connects to the Server. Once someone connects(Aka someone on the Client Side clicks "Connect" button) it starts another thread, the Server_UserConnection class. Each user has it's own thread to communicate with the server. 

This class is a little trickier, as it have the important methods that control the mid-state of messages that are goind back-and-forth. Although it's a single user-server communication, it has the both the User objects of the connected user and the connected-to user.

```java
private User myUser;
private User connectedTo;
```
This is due to the Message Status Update message. It is only a single variable, since there are no `groups allowed` nor `multi-chatting`, but this architeture is easily changeable. It could be a list of connectedTo users, for instance. This class also has a reference to the Main Server that is running, for some utility porpouses such as finding a connected user.

From this point on, it just receives and send messages. Let's see how it does it.

#### Handling Messages
As soon as a Connection to a user is stablished, the new theard awaits on this line: 
```java
this.input = new DataInputStream(clientSocket.getInputStream());
```
Until it gets some input. At this point, the client has still no chats open with another user, it only has clicked the "Connect" button to log in.

The trick is, many user actions trigger the line, not only actual messages sent. For instance, when the user opens a chat, in the background it sends a message to the server telling it he/she is trying to connect to someone. So as soon as we receive someone in the InputStream, we have to deal with a lot of situations:

![](https://raw.githubusercontent.com/Gui-Lima/ZipZap/master/Res/User_ConncetionHandleInput.png "Project General Structure")

From this point, the functions are pretty self-explanatory, each one calling a different function from the server(as told before, each connection has a reference to the main server).

The server then just write in the Output of the due user a Message, depending on what type the message was. Now it's on the Client to read that message and interact with it's GUI.
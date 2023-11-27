# <center>Network & Sockets</center>

## 0. Preface
These notes are wrote during the studies made to achieve the project "WebServer" of the 42 common core. I will skip or mention concepts and
observations unrelated to the matter of the projects. However, there will be present some arguments and concept that aren't strictly
related to the argument of this note (Sockets), the reason for this choice is that I want to report on this page/s the exact steps I've made
during the studies and add things that may have helped me out to achieve a better understanding of the subject. Also, these notes will be mostly theoretical.

## 1. What is a socket?
A socket is a software structure at the "session" layer (look up <a href="https://it.wikipedia.org/wiki/Modello_OSI"> OSI Model </a>) of a network application.
It serves as an endpoint for the machine he is running on by sending and receiving data across a network connected to the machine itself. From the moment a machine
can have more than one socket each of them is identified by their own socket address which is given by the sum of three elements: the
transport protocol (TCP/UDP), the IP address, and the port number. <br>
For now we have talked about what can be defined as Internet Sockets, used by a machine to communicate with another machine inside or even outside
his network; but sockets can also be used as endpoint for inter-process communications.

## 2. OSI Model
To have a better understanding of what is the role of a socket we'll take a look at the <a href="https://it.wikipedia.org/wiki/Modello_OSI"> OSI model</a>.<br>

![OSI_Model](https://github.com/TommyJD93/Notes/assets/59456000/d49f1aba-fc44-4707-9df6-d2ef1a490eab)

In this Model our socket is located in the "Session" layer, as we said before, and it's part of a running program composed by the layers of: Application, Presentation, Session and Transport.
The rest of the model describes the route through which the data passes to reach the other end of the connection.

## 3. Behaviour of the socket
During the execution of a program that uses sockets, for example a web server, the socket passes through different statuses:
- Unbound
- Bound
- Listening
- Connected

### 3.1 Unbound
As we said before in the point 1 a socket is identified by three elements, when these three elements are not set inside the socket itself we call it "unbound" because it
lacks the information to establish a connection for the program using it.

### 3.2 Bound
Once we gave all these information to our socket we call it "bound" and once it's "bound" it is ready to begin listening for
connections. Sticking to the previous example, inside a web server we will give our socket the IP of the machine hosting the
web server, the port through which we want our server to communicate with other networks and the protocol through which
we established a connection with a client (for the WebServer project I will be using the TCP protocol).
In practice what we are doing here is bind the ip address and the port number to the socket.

### 3.3 Listening
After the socket is bound, it is put into the listening state, waiting for incoming connections on the specified port. At this point,
when we are in listening mode, we are listening out for incoming request through the details we gave to the socket; these
incoming requests are made by clients that wants to connect to the socket through that specific IP and port (obviously using
the protocol supported by the socket).

### 3.4 Connected

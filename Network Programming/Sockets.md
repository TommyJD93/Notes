# <div align='center'> Network & Sockets <div>

## 0. Preface
These notes were written during the studies made to achieve the project "WebServer" of the 42 common core. I will skip or mention concepts and
observations unrelated to the matter of the projects. However, there will be present some arguments and concept that aren't strictly
related to the argument of this note (Sockets), the reason for this choice is that I want to report on this page/s the exact steps I've made
during the studies and add things that may have helped me out to achieve a better understanding of the subject. Also, these notes will be mostly theoretical.

## 1. What is a socket?
A socket is a software structure at the "session" layer (look up <a href="https://it.wikipedia.org/wiki/Modello_OSI"> OSI Model </a>) of a network application.
It serves as an endpoint for the machine he is running on by sending and receiving data across a network connected to the machine itself. From the moment a machine
can have more than one socket each of them is identified by their own socket address which is given by the sum of three elements: the
transport protocol (TCP/UDP), the IP address, and the port number. <br>
For now we have talked about what can be defined as Internet Sockets, used by a machine to communicate with another machine inside or even outside
his network; but sockets can also be used as endpoint for inter-process communications. <br>

**Note: Sockets communicate using file descriptors.**

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
- Close/Shutdown

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
Before this status the client has to create a socket of his own that will go through the statuses we've talked before, after that it
will send a request to the server. Note that the client socket doesn't have any control on the port number used to communicate with
the server. Once the connection request from the client is acknowledged and accepted by the server it will duplicate the listening
socket and this copy will get in the connected status, while the original socket will remain in the listening status. In this way
the server can handle multiple clients at the same time by accepting all the incoming connections from the original socket and then
redirecting them to duplicates of that socket that will handle the requests of the connected client.

### 3.5 Close/Shutdown
Once the communication between client and server has reached the end and a request to close the connection is sent, the "life" of
the socket has come to an end, now we have two options: Close the socket, Shutdown the socket; if we proceed with the first option
we will block the connection and destroy the socket. On the other hand we can shut down our socket, this means that we can block
the connection without destroying the socket, however we have to specify how to shut down this socket. For this we have three
different options that we will call with generic macros for simplicity: SHUT_RD, SHUT_WR and SHUT_RDWR. <br>
SHUT_RD will block the socket from receiving further messages **(pending data in the buffer is still readable)**, all requests made
to the socket will return zero-sized messages and the socket is still able to send data. SHUT_WR prevents the socket from sending
messages **(as before all pending messages will be sent before shutting down the socket)** and informs the client of it, similar to
SHUT_RD this operation will not affect the reading capabilities of the socket. At last, we have SHUT_RDWR, the result of this mode
is comparable to the execution of both SHUT_RD and SHUT_WR sequentially.

## 4. Seven steps of a server
Now that we have talked about the "life cycle" of a socket let's take a look at the seven steps that a server needs to start,
establish a connection with a client and exchange messages with it.<br>
First of all let's have a look at these steps:
- Initialize the socket
- Create the socket
- Bind the socket
- Listen on the socket
- Accept connection
- Send and receive data
- Close connection

I'd like to mention also the five steps of the client, pointing out that: the first two are the same of the server, but then
rather than binding the socket the client just attempt to connect to the given IP address (basically it skips the step three
and four of the server).<br>
Here is the list of the client's steps:
- Initialize the socket
- Create the socket
- Connect to the server
- Send and receive data
- Close connection

**Note: from now on there will be some code snippets, since for the WebServer project I'm going to use C++98 and
Ubuntu 22.04 I'll be referring to functions, methods, classes and data-types that works for my setup**

### 4.1 Initialize the socket
For the first step we just need to include the socket library by adding the following line to your
.hpp file.
```
#include <sys/socket.h>
```
I know I've said that I'm going to talk for my specific setup, but I thought it would be nice to specify the
reason why this is a step all by itself and why it isn't included in the creation of a socket. On Windows platform
it's not enough to include a library, but we need to initialize the DLL responsible for sockets (WSA) and use the
appropriate functions to determine whether the DLL has been correctly imported or not.

### 4.2 Create the socket
After that we can call the "socket()" function that will create un unbound socket. The socket function declaration
is the following: <br>
```int socket(int domain, int type, int protocol);``` <br><br>
Let's brake down the parameters.

#### 4.2.1 int domain
This argument specifies the communication domain used by the socket; this selects the protocol family which will
be used for communications. All the available domains are specified in the "<sys/socket.h>" lib.
Here's a few example of domains:
- AF_UNIX used for local interprocess communication
- AF_INET used for IPv4 internet protocols (the protocol we are going to use in this project)
- AF_INET6 used for IPv6 internet protocols. 

#### 4.2.2 int type
This argument specifies the type of socket to be created and with it the communication semantics.
Here's two types that are available in "<sys/socket.h>":
- SOCK_STREAM used for TCP connections
- SOCK_DGRAM used for UDP connections

#### 4.2.3 int protocol
The 'protocol' argument specifies a particular protocol to be used with the socket. Normally only one protocol
exist to support a socket type within a given protocol family, in which case protocol can be specified as '0'.
However, it is possible that more protocol exists for a given family, in this case the intended protocol must
be specified.

_<a href="https://man7.org/linux/man-pages/man2/socket.2.html">source 1</a>_ <br>
_<a href="https://pubs.opengroup.org/onlinepubs/9699919799/functions/socket.html">source 2</a>_

### 4.3 Bind the socket
For the third step we want to bind our IP address and port number to our socket, to do this we use the "bind()"
function. The function declaration is the following:<br>
```int bind(int socketfd, struct sockaddr_in *address, int address_len);``` <br><br>

#### 4.3.1 int socketfd
This argument specifies the file descriptor given by the previous "socket()" call

#### 4.3.2 struct sockaddr_in *address
This parameter is a pointer to the "sockaddr_in" structure which is the following:
```C++
#include <netinet/in.h>

struct sockaddr_in {
    sa_family_t     sin_family;     /* Address Family */
    in_port_t       sin_port;       /* Port number */
    struct in_addr  sin_addr;       /* Socket address */
};

struct in_addr {
   in_addr_t s_addr;        /* Data structure to store IP addresses */
};
```
As you can ses it contains the address family of the socket and the address to be bound to the socket. The sin_family
member has to be coherently with one of the macros we've seen in the point 4.2.1.

#### 4.3.3 int address_len
And for this parameter we simply need to give the size of previous argument, we can do this easily by using ```sizeof(address)```

_<a href="https://man7.org/linux/man-pages/man2/bind.2.html">source 1</a>_ <br>
_<a href="https://pubs.opengroup.org/onlinepubs/7908799/xns/netinetin.h.html">source 2</a>_ <br>
_<a href="https://man7.org/linux/man-pages/man3/sockaddr.3type.html">source 3</a>_ <br>
_<a href="https://pubs.opengroup.org/onlinepubs/009695399/basedefs/sys/socket.h.html">source 4</a>_ <br>

### 4.4 Listen on the socket

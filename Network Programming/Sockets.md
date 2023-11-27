# <center>Network & Sockets</center>

## 0. Preface
These notes are wrote during the studies made to achieve the project "WebServer" of the 42 common core. I will skip or mention concepts and
observations unrelated to the matter of the projects. However, there will be present some arguments and concept that aren't strictly
related to the argument of this note (Sockets), the reason for this choice is that I want to report on this page/s the exact steps I've made
during the studies and add things that may have helped me out to achieve a better understanding of the subject.

## 1. What is a socket?
A socket is a software structure at the "session" layer (look up <a href="https://it.wikipedia.org/wiki/Modello_OSI"> OSI Model </a>) of a network application.
It serves as an endpoint for the machine he is running on by sending and receiving data across a network connected to the machine itself. From the moment a machine
can have more than one socket each of them is identified by their own socket address witch is given by the sum of three elements: the
transport protocol (TCP/UDP), the IP address, and the port number. <br>
For now we have talked about what can be defined as Internet Sockets, used by a machine to communicate with another machine inside or even outside
his network; but sockets can also be used as endpoint for inter-process communications.

## 2. OSI Model
To have a better understanding of what is the role of a socket we'll take a look at the <a href="https://it.wikipedia.org/wiki/Modello_OSI"> OSI model </a> <br>

[Imgur](https://imgur.com/s6l9G19)

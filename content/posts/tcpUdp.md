+++
date = '2026-03-24T22:53:37+05:30'
draft = false
title = 'All about TCP, UDP'
+++

# All about TCP, UDP

There are 7 layers in the OSI model
- Application Layer
- Presentation Layer
- Session Layer
- Transport Layer
- Network Layer
- Data Link Layer
- Physical Layer

out of these, today I’m going to write about the layer I find most interesting.

#### The Transport Layer

Transport layer has one job to do and that is to transport the packets from one process (app) to the other process (app) across the network.
So basically it ensures that the right packet reaches the right process or app on your system.

Otherwise the computer wouldn’t know which process should receive a packet.

It comprises of protocols like TCP and UDP.

While it’s technically possible to send data using only IP, but then you will have to do all the heavy lifting yourself because you will have to implement TCP / UDP properties yourself.

SO what exactly is TCP or UDP for that matter.

![[tcpUdp.svg]]

Lets take a look at the image above... it has a server that is running 3 applications, namely, a Minecraft server, an FTP and a web server ( don't worry if you don't know what those are, just know that they are like apps ).

Now lets assume that I am playing Minecraft on my PC and I want to connect to the Minecraft server to play with my friends online.
I write the IP address of the server in my game which is shown in that image above.. and hit the connect button.
My PC will send data in the form of packets to the IP address of the server, now the question is.. how exactly does the server know that the incoming packets need to be sent to the Minecraft process ( app ) running on the server and not to the FTP or Web or any other process.

Well that is exactly where protocols like TCP or UDP come into action.

A TCP/UDP protocol carries a port number with them in their header ( more on this later ) which tells the server that where a particular packet needs to be sent. Now here is another image with port numbers.

![[tcpHeader.svg]]

Now lets go back to the Minecraft example, this time I will enter the IP and the port number of the Minecraft server in my game and click connect again.
This time the server will know that the packet (data) which I sent to the server needs to be given to the Minecraft process and thus a secure connection is established and now I can happily play with my friends online. LETS GOOO!!!

##### Difference between TCP and UDP

Well if both TCP and UDP does the same thing then what's the difference between the two?
Both carry data, IP and port... but the real difference comes from how they behave while sending data from client to server and vice versa.

So TCP takes a very careful approach. It establishes a connection first.. then keeps track of every piece of data it sends, waits for confirmations, and resends anything that gets lost due to poor connection or some other reason.

Whereas in UDP there’s no connection setup, no guarantee if the data arrived, no retries, no ordering. It just sends packets of data and moves on. If something gets lost, UDP doesn’t stop to fix it... Now you might think that that's a flaw, but in many situations, it’s exactly what you want. Because with UDP you get lower latency as compared to TCP,
for example if you are using VoIP ( Voice over IP ), you wouldn't care if some packets of data gets lost and the receiver on the other end doesn't receive what you said in a particular instant because you can obviously repeat what you said in a call.... lol

#### How does TCP establishes connection between Client and Server

A TCP connection is a 3 way handshake.

Imagine a client ( your browser ) wants to talk to a server.

Step 1: SYN ( Client -> Server )

The client sends a packet with the SYN i.e. synchronize flag to start the conversation.

Step 2: SYN-ACK ( Server -> Client )

Once the server receives the client's SYN request, it then responds with SYN-ACK flags.
Firstly the ACK i.e. acknowledgment confirms that the server has received the request to establish the connection.
Secondly the SYN flag indicates that the server is also ready to initiate connection from it's side.

Step 3: ACK ( Client -> Server )

Finally the client sends back an ACK flag which essentially means that the client has received server's response.

And there you have it, a successful connection has been established between the client and the server for sending and receiving data.

Now you understand why TCP connection takes time to send and confirm data while UDP is faster in that sense because it doesn't do any of this handshake process, instead it directly sends data to the server, as if it's saying,

“Here’s the data. Take it or leave it. 💅”

My favorite game which is Minecraft uses TCP protocol to establish connection between the client and server.

#### TCP / UDP Headers

In layman's terms, a **TCP or UDP header** is label attached to a piece of data (a packet) before it is sent over a network.
It contains information required for reliable transmission of data between client and a server.

![[Untitled-2026-03-23-0026(3).svg|385]]

The image above shows the header part and its composition.

As you can see the TCP protocol has the source port address and destination port address in its header like we talked about earlier.

A TCP header is about 20 bytes in size, minimum.
While a UDP header is about 8 bytes in size, which is another reason why UDP is faster.. because low header size enables faster and lower-overhead transmission.

And the flags part in the image refers to the SYN, ACK and many other flags. But today I told you about SYN and ACK flags because that's more important to know for understanding the 3 way handshake.

Unlike the huge TCP header, UDP only consists of 4 header fields, namely
1. Source Port
2. Destination Port
3. Length
4. Checksum

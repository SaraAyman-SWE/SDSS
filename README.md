# Simple Delay Sync Service (SDSS)

# Intro 

Distributed systems play an important role in our lives today; think about the number of Google servers that exist around the globe or machines in Amazon's cloud. It's often important to measure the delay between nodes in order to distribute a workload for example in the most efficient way. In this lab, we'll get introduced to a simple way to measure delay between nodes (i.e. devices).

# Objective 

This project is a mini introduction to peer-to-peer networking. We'll get to know how connections can be initiated when no single node is the server.
You'll also get yourself comfortable with the notion that any node can either be a client or a server or switch from one to another. 
One other purpose is to make a network with minimal infrastructure, this is useful in emergencies for example or saving bandwidth and many other cases you can read about Ad-hoc & Mesh networks.

# Requirements

Operating System: Linux (required, otherwise the project mostly won't work)
We have a small system that's distributed across a network, our goal is to measure delay between each node and its peers. This will be done in the following steps

   1. Each node will be sending a broadcast message once every second, so other nodes know about its existence.
   
   2. Once a node [A] detects a message from a non-recognized node [B]; it'll initiate a TCP connection to node [B].
   
   3. Node [B] will send a TCP message to node [A] containing [B]'s timestamp.
   
   4. Node [A] will compute the delay by subtracting A's timestamp from B's timestamp, then it'll store that delay in a hashtable containing the node ID as key and its (delay, broadcast_count) as values [NeighborInfo object]. This will be the delay from [B] -> [A], we won't need to compute the delay in the other direction. Each node will know the delay "to" its neighbors only not in both ways.
   
   5. This table will be updated every 10 broadcasts for each node. Use the hash table you have for this purpose.
   
   6. If a node is unreachable when the time of update is due, it's removed from the hashtable.
   
# Peer discovery

You'll need to implement a peer discovery mechanism. Peer discovery means sending messages to everyone in the network to let them know about your node's existence. This is one of the steps in DHCP since once a device is connected to the network, it doesn't know what the router's IP is or where it should send any packets.
Sending messages to everyone can be done using UDP Broadcasting, not over TCP, since TCP needs two "connected" endpoints. This concept is illustrated thoroughly here, please proceed reading there before continuing. We'll have only one constant value which is the broadcasting UDP port.


You'll use broadcasting to share certain information by which; other peers receiving that information will use it to establish a TCP connection with this node. The broadcast port is the single constant known by all nodes, anything else is dynamic.
This conversation represents a high level overview. The square brackets indicate the used protocol.


[UDP] Device 0 -> EVERYBODY : cc89e682 ON 6758

[UDP] Device 1 -> EVERYBODY  : 1ad640cb ON 8768


[TCP] Device 0 connects to port 8768 of device 1

[TCP] Device 0 -> Device 1  : [ cc89e682's TIMESTAMP ]

[TCP] Device 1 -> Device 0  : [ 1ad640cb's TIMESTAMP ]


	Note: Device cc89e682 knew Device 1ad640cb's IP from the return value of recvfrom(). cc89e682 didn't broadcast its IP.
________________

# Protocol Format

All broadcast messages will be sent/received over port number 35498 which is included in the skeleton as get_broadcast_port(), don't change this value.
Broadcast Message [UDP]
You'll always need to broadcast a message to inform peers in the network of your node's existence. Towards that end, the following message format is used.
[NODE_UUID] ON [TCP_SERVER_PORT]

	   * NODE_UUID is a randomly generated 8-char partial-UUID (string) to identify each node. 
     
It's provided in the skeleton and changes every time you run the code.

   * TCP_SERVER_PORT the random TCP server port assigned by the operating system to your socket.
   
   * Each token in a protocol message is followed by a space, except the last one. 
   
   * Encode/Decode messages as UTF-8
   
   * Note that ON is uppercase.
   
# Timestamp [TCP]

Nodes will be exchanging UTC timestamps not the system-wide timestamps. You'll send the "number" as is without converting it to string in the TCP packet. Checkout the resources section. It's also wise to read more about what UTC is and whether it's the same as GMT.
In other words, if we assume that your project runs in 2 different time zones; UTC+3 and UTC-7. It makes no sense to find that the delay between those nodes over the internet is 10 hours, it should remain in the 100ms range. That's why UTC is needed.
We're using TCP for this step because delay sync is a crucial operation, we will never know if UDP dropped the sync packet.

# Implementation Details

> You might want to run the code in two separate terminals on your PC to test it, you'll face the problem of trying to bind the UDP socket to the same port. You can use the SO_REUSEPORT socket option to resolve this issue. 

> Don't forget to set SO_BROADCAST for your UDP socket.

> You'll need to handle ConnectionRefusedError when you attempt to connect to the other node in case it was down. Don't handle the general Exception class, doing so will get you a penalty. Be specific about what you handle.

> Launch timestamp exchange process in a separate thread to avoid blocking any running thread that's used for syncing.

Notes
      * You must use port 0 for your TCP server (so the OS allocates a random port for you). 
      
      * Your broadcast message must contain the port of the TCP server, you can get that from the socket object after you call listen.
      
      * Hardcoding any value will cause the grader to fail. Nobody will check your code to correct the bad values. Program arguments have default values, you can change those, otherwise any hardcoded value will make at least one test case fail, if you're so unlucky it'll fail all the test cases (you hardcoded the TCP port for example) and you'll get a 0.

# Requirements Checklist

> Run every function in the skeleton that ends with _thread in its own thread

Use the daemon_thread_builder() function to make the threads, its signature is similar to python's threading module.


> Setup the TCP socket and start accepting clients (tcp_server_thread) part#1

Make sure you meet the requirements for binding the server address. Make a server loop to accept clients. This step is required at this point so you can know your local TCP port.


> Setup the UDP socket and send broadcasts (send_broadcast_thread)

To confirm that this part is working, run this code and you should find that it's printing data in blue color, because of the print statement in the skeleton code.


> Parse received broadcasts (receive_broadcast_thread)

To confirm that this part is working correctly, run 2 instances of the code, you'll notice that 2 different messages are received each one of them will contain different TCP ports after "ON" word.


> Accept TCP clients and send them the node's timestamp (tcp_server_thread) part#2

After a TCP client is accepted, send them your node's timestamp and close the connection with them.


> Initiate TCP connection to newly discovered nodes and exchange timestamps (exchange_timestamps_thread)

Make sure you run this process in its own thread. Don't initiate a connection each time you receive a broadcast message from another node, this will make the network crammed and is very wasteful, hence it'll be penalized.


> Refresh neighbor information every 10 broadcasts
________________

# Testing Your Code 

Wireshark
Running the code multiple times
This is a slightly better approach, always log what you do in the code by using prints. Debugging isn't very feasible in this project.

Since all of you are implementing the same protocol, you can activate your phone as a hotspot, join the same network, run your codes. All your nodes should be able to detect each other. This is the best test.


# Raw-Socket
README- 

High Level Approach - 

In this project we implemented raw sockets and configured the TCP and IP headers accordingly. Using the URL provided as argument, our code downloads the entire html data of the URL.
the approach follows as-
1) First we imported the python libraries required that are socket, sys, urlparse, random, and struct.
2) We obtained the IP address of the source host by creating a udp socket which connects to a website. And using the socket option sock.getsockname and the destination address from the url by using socket.gethostbyname.
3) Once we have the correct source and destination IP address we create the IP header with the proper IP header values. 
4) Then we create the TCP headers accordingly. The TCP and IP headers are configured refering the silver moon tutorial provided. 
5) Using the pack function the IP and TCP headers were packed in the proper network order big endian format and the headers were created. 
6) The TCP checksum was calculated over a pseudo packet that consisted of the source IP, destination IP, reserved data, protocol value, and data + header length. 
7) The IP checksum was calculated over the IP, TCP header and the data that is to be sent. The checksum function was also written with reference to the silver moon tutorial.
8) In case the data is odd, the checksum function adds padding in the end to calculate the checksum.
9) First we send a SYN packet to the server through the send socket which is of SOCK_RAW/IPPROTO_RAW type. In the SYN packet the TCP flag SYN is set and the port number is a random value between 1-65535.
10) After sending the SYN packet, we receive a SYN_ACK from the receive socket of type SOCK_RAW/IPPROTO_TCP which will filter out only the TCP packets comming to the socket. 
11) After receiving the packet we calculate the checksum of the IP header, if the value is equal to 0 then the checksum is correct. Then unpack the IP headers and the TCP headers and extract the source IP and the Destinaion IP from the IP header and the port number along with sequence number and the acknowledgement number from the TCP header.
12) If the received source IP is equal to the sent destination address and the received destination is equal to the locaol host IP and the port number is same as the random one generated while sending and the TCP flags have SYN and ACK set then this packet is the proper SYN-ACK response from the server and hence we need to send an ACK packet back to complete the three way handshake process and extablish state connection.
13) After validating the SYN-ACK packet we need to again create the IP headers and TCP headers with correct values and pack them, calculate the checksum as before and send the ACK packet. The SYN and ACK packet contain no data in the data field.
14) After the three way handshake is complete we need to send an HTTP request for the URL specified in the argument. 
15) Again we create IP and TCP headers, pack them, compute the checksum values as before and create the packet. But this packet contains the HTTP GET request in the data field. 
16) We create a while loop which keeps on receiving the fragments and stores the data fragments in a list with index as the sequence number. We have a filter here that checks the source and destination IP addresses and the TCP flags value, i.e. if the flag value has ACK or PSH and ACK set then store the data in the data and then send an ACK packet back to the server with the corrosponding sequence numbers and acknowledgement numbers.
17) The acknowledgement numbers here would be the received sequence number plus the data length. So as to acknowledge all the bytes were received in the fragment. 
18) The last packet from the server would have the FIN flag set along with ACK set or ACK and PSH set. In this case we need to store the data and send a FIN ACK packet to the server that will terminate the connection. 
19) After this we need to extract the filename from the given URL, if the URL end with an "/" or the URL has no path along with the hostname then the index.html file should be created, else the file name should be the last part of the URL path. 
20) Using file.open we can open the file in the write mode and then one by one we can write the data from the data list which contains the data indexed according to the sequence numbers. 
21) Before writing the data we need to sort the sequence numbers in the data list so that the data is written sequentially into the file. 

TCP/IP features implemented - 
1) IP headers
2) TCP headers
3) Checksum Validation 
4) Handled out of order packets
5) HTTP get request
6) HTTP status code validation

Challenges faced - 

1) Packing the headers according to the network byte order and identifying the exact order, plus keeping in mind the checksum is not in network byte order and accordingly packing the packet again was a major challenge. 
2) Validating checksum for received IP and TCP packets
3) Storing the data according to the sequence number of the packet.

Testing- 
The downloaded file was compared with the file generated by the wget command and the difference between the two files was calculated using diff. 
Used Wireshark to check outgoing packet header values and incoming packet headers values. 

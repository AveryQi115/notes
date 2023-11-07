# OSI model layer 笔记

- Layer2: Data link (switch using layer1 and layer2)

  eg: mac, vlan

  pack the data into frames, check the data error by checksum

- Layer3: Network (router using layer1, layer2 and layer3)

  Eg: IP

  host addressing, message forwarding

- Layer4: Transport

  Eg: TCP/UDP

  flow control, congestion avoidance

- Layer5: session

  eg: RPC

  Build and control sessions, introduce checkpoints

  (The layer 5 (control of logical connections; also session layer) provides inter-process communication between two systems. Here you can find among others the protocol RPC (Remote Procedure Call). To resolve failures of meeting and similar problems, the session layer services for an organized and synchronized data exchange. To this end, recovery points, so-called fixed points (check points) introduced, where the session can be synchronized after a failure of a transport connection again without the transfer must start from the beginning again.)

- Layer6: presentation

  De/Encryption, Encoding

  


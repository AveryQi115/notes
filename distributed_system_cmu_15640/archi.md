# archi

LSP

(partA checkpoint)

- 建立connection
- 结束

- 接收message by order
  - 维护一个expected sqn

- Sliding window protocol
  - maxunackedMessage(和swp共同控制)(smaller than window size)

- Payload size (first check)

  - size sent before data
  - real payload is larger, truncate the message

  - real payload is smaller, drop it

- checksum (check after payload)
  - checksum different, drop it





- epoch events
  - Every epoch starts, check unacked messages and resend them
  - exponential backoff
  - connect message retansmission handle: one connection for duplicate requests
  - in the past epoch limit epochs, if no data is received from the server, close connection; if no data is to send, send heartbeat
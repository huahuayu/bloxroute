## Problem

Failed to run bloxroute stream go example

after run the example, just get `EOF` in the output

## Reproduce

1). copy go snippet from the document: https://docs.bloxroute.com/streams/newtxs-and-pendingtxs#requests-cloud-api , because I am a non enterprise account type, I use `wss://api.blxrbdn.com/ws` to connect, replace the `Authorization` string to my `secret_hash.txt` string.

```go
package main

import (
	"crypto/tls"
	"fmt"
	"github.com/gorilla/websocket"
	"net/http"
)

func main() {
	// Enterprise users can follow line 12-27 to use wss://eth.feed.blxrbdn.com:28333
	//cert, err := tls.LoadX509KeyPair(
	//	"cert/external_gateway_cert.pem",
	//	"cert/external_gateway_key.pem",
	//)
	//if err != nil {
	//	fmt.Println(err)
	//	return
	//}

	//tlsConfig := &tls.Config{
	//	Certificates:       []tls.Certificate{cert},
	//	InsecureSkipVerify: true,
	//}
	//dialer := websocket.DefaultDialer
	//dialer.TLSClientConfig = tlsConfig
	//wsSubscriber, _, err := dialer.Dial("wss://eth.feed.blxrbdn.com:28333", nil)

	// Non Enterprise users should follow line 30-35 to use wss://api.blxrbdn.com/ws
	tlsConfig := &tls.Config{
		InsecureSkipVerify: true,
	}
	dialer := websocket.DefaultDialer
	dialer.TLSClientConfig = tlsConfig
	wsSubscriber, _, err := dialer.Dial("wss://api.blxrbdn.com/ws", http.Header{"Authorization": []string{"420eccb0f2666a6e189c9f66d2d6678d"}})

	if err != nil {
		fmt.Println(err)
		return
	}

	subRequst := `{"id": 1, "method": "subscribe", "params": ["newTxs", {"include": ["tx_hash"]}]}`
	err = wsSubscriber.WriteMessage(websocket.TextMessage, []byte(subRequst))
	if err != nil {
		fmt.Println(err)
		return
	}

	for {
		_, nextNotification, err := wsSubscriber.ReadMessage()
		if err != nil {
			fmt.Println(err)
		}
		fmt.Println(string(nextNotification)) // or process it generally
	}
}
``` 

2). go run main.go

3). get `EOF` immediately

## more info

I am base in China, if I don't use a proxy I get : 

```
read tcp <my-ip>:42438->39.106.255.190:443: read: connection reset by peer
```

if I use a proxy(Hongkong) in goland I get: EOF

I try to run in an aws server in HongKong

```
go run main.go
go: downloading github.com/gorilla/websocket v1.4.2
read tcp <my-server-ip>:48390->39.106.255.190:443: read: connection reset by peer
```

from my laptop to telnet

```
$ telnet api.blxrbdn.com
Trying 39.105.165.155...
telnet: Unable to connect to remote host: Connection refused

$ telnet api.blxrbdn.com 443
Trying 39.106.255.190...
Connected to api.blxrbdn.com.
Escape character is '^]'.
```

nslookup

```
nslookup api.blxrbdn.com
Server:         127.0.0.53
Address:        127.0.0.53#53

Non-authoritative answer:
Name:   api.blxrbdn.com
Address: 39.105.165.155
```


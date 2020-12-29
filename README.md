# bloxroute go stream api example

## get started

1). Copy go snippet from the document: https://docs.bloxroute.com/streams/newtxs-and-pendingtxs#requests-cloud-api

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
	wsSubscriber, _, err := dialer.Dial("wss://api.blxrbdn.com/ws", http.Header{"Authorization": []string{"$your-authorization"}})

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
2). Copy you account certificate to `cert` dir, if you are enterprise user.

3). Because I am a non enterprise account type, I use `wss://api.blxrbdn.com/ws` to connect, replace the `Authorization` string (find in you blxrbdn account info page https://portal.bloxroute.com/).

3). go run main.go
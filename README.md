# y-taubyte
Use Taubyte's PubSub as a backend for yjs.

# Usage
```js
const ydoc = new Y.Doc()

// Taubyte websocket usage
const provider = new WebsocketProvider("/ws/url", ydoc, {
  room: "roomName",
})
```

# Demo
 - Demo: https://yjs.skelouse.com/
 - Project:
   - Config: https://github.com/taubyte-test/tb_yjsproject
   - Code:  https://github.com/taubyte-test/tb_code_yjsproject
 - Frontend: https://github.com/taubyte-test/tb_website_y_js_website

# Expected project structure
To use this package you will need to have a serverless function on a Taubyte-based Cloud that provisions a WebSocket to a pubsub resource.

## What is Taubyte 
Taubyte is an open-source Web3 Cloud Platform. To learn more head to https://tau.how

## Create a project
Create a project https://tau.how/docs/getting-started/project
## Create a serverless function
Create a serverless function https://tau.how/docs/getting-started/first-serverless-function
## Create a pubsub resource
Configuration refrence https://tau.how/docs/config/messaging

## Generate WebSocket link
Create an HTTP GET function and copy past the code below. Note that the function's entry poing is `getsocketurl`,
```go
package lib

import (
	"net/url"
	"bitbucket.org/taubyte/go-sdk/event"
	"bitbucket.org/taubyte/go-sdk/pubsub"
)

func getChannel(h event.HttpEvent) string {
	room, _ := h.Query().Get("room")

	channelName := "someChannel"
	if len(room) == 0 {
		room = "default"
	}

	return channelName + "/" + room
}

//export getsocketurl
func getsocketurl(e event.Event) uint32 {
	h, err := e.HTTP()
	if err != nil {
		return 1
	}

	url, err := func() (url url.URL, err error) {
		channel, err := pubsub.Channel(getChannel(h))
		if err != nil {
			return
		}

		return channel.WebSocket().Url()
	}()
	if err != nil {
		h, err := e.HTTP()
		if err != nil {
			return 1
		}

		h.Write([]byte(err.Error()))
		return 1
	}

	h.Write([]byte(url.Path))

	return 0
}
```

Your function configuration should look like this:
```yaml
id: QmU8VCp2Yh7vs2KBRfjfzFoxY1dqcqKG2kAfdJagYBHun4
description: Returns a socket url to the matched channel with an optional room query
tags: []
trigger:
    type: https
    method: GET
    paths:
        - /ws/url
domains:
    - YourDomain
includes:
    - /functions/getsocketurl.go
execution:
    timeout: 10s
    memory: 10MB
    call: getsocketurl
```




# License

[The MIT License](./LICENSE)

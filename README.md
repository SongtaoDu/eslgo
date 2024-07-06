# eslgo
[![PkgGoDev](https://pkg.go.dev/badge/github.com/SongtaoDu/eslgo)](https://pkg.go.dev/github.com/SongtaoDu/eslgo)
![GitHub Workflow Status](https://img.shields.io/github/workflow/status/SongtaoDu/eslgo/Go)
[![Go Report Card](https://goreportcard.com/badge/github.com/SongtaoDu/eslgo)](https://goreportcard.com/report/github.com/SongtaoDu/eslgo)
[![Total alerts](https://img.shields.io/lgtm/alerts/g/SongtaoDu/eslgo.svg?logo=lgtm&logoWidth=18)](https://lgtm.com/projects/g/SongtaoDu/eslgo/alerts/)
[![GitHub license](https://img.shields.io/github/license/SongtaoDu/eslgo)](https://github.com/SongtaoDu/eslgo/blob/v1/LICENSE)

eslgo is a [FreeSWITCH™](https://freeswitch.com/) ESL library for GoLang.
eslgo was written from the ground up in idiomatic Go for use in our production products tested handling thousands of calls per second.

## Install
```
go get github.com/SongtaoDu/eslgo
```
```
github.com/SongtaoDu/eslgo v1.4.1
```

## Overview
- Inbound ESL Connection
- Outbound ESL Server
- Event listeners by UUID or All events
  - Unique-Id
  - Application-UUID
  - Job-UUID
- Context support for canceling requests
- All command types abstracted out
  - You can also send custom data by implementing the `Command` interface
    - `BuildMessage() string`
- Basic Helpers for common tasks
  - DTMF
  - Call origination
  - Call answer/hangup
  - Audio playback

## Examples
There are some buildable examples under the `example` directory as well
### Outbound ESL Server
```go
package main

import (
	"context"
	"fmt"
	"github.com/SongtaoDu/eslgo"
	"log"
)

func main() {
	// Start listening, this is a blocking function
	log.Fatalln(eslgo.ListenAndServe(":8084", handleConnection))
}

func handleConnection(ctx context.Context, conn *eslgo.Conn, response *eslgo.RawResponse) {
	fmt.Printf("Got connection! %#v\n", response)

	// Place the call in the foreground(api) to user 100 and playback an audio file as the bLeg and no exported variables
	response, err := conn.OriginateCall(ctx, false, eslgo.Leg{CallURL: "user/100"}, eslgo.Leg{CallURL: "&playback(misc/ivr-to_hear_screaming_monkeys.wav)"}, map[string]string{})
	fmt.Println("Call Originated: ", response, err)
}
```
## Inbound ESL Client
```go
package main

import (
	"context"
	"fmt"
	"github.com/SongtaoDu/eslgo"
	"time"
)

func main() {
	// Connect to FreeSWITCH
	conn, err := eslgo.Dial("127.0.0.1:8021", "ClueCon", func() {
		fmt.Println("Inbound Connection Disconnected")
	})
	if err != nil {
		fmt.Println("Error connecting", err)
		return
	}

	// Create a basic context
	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Minute)
	defer cancel()

	// Place the call in the background(bgapi) to user 100 and playback an audio file as the bLeg and no exported variables
	response, err := conn.OriginateCall(ctx, true, eslgo.Leg{CallURL: "user/100"}, eslgo.Leg{CallURL: "&playback(misc/ivr-to_hear_screaming_monkeys.wav)"}, map[string]string{})
	fmt.Println("Call Originated: ", response, err)

	// Close the connection after sleeping for a bit
	time.Sleep(60 * time.Second)
	conn.ExitAndClose()
}
```
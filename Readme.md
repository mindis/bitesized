# bitesized [![Build Status](https://travis-ci.org/sent-hil/bitesized.svg?branch=master)](https://travis-ci.org/sent-hil/bitesized)

bitesized is a library that uses redis's bit operations to store and calculate analytics. It comes with a http server that can be used as an stand alone api (not implemented yet).

## Motivation

It started when I saw a [blog post](http://blog.getspool.com/2011/11/29/fast-easy-realtime-metrics-using-redis-bitmaps/) about using redis bitmaps to store user retention data. It sounded pretty neat and simple, not to mention fun, to implement.

## Install

`go get github.com/sent-hil/bitesized`

## Usage

Initialize client:

```go
package main

import (
  "github.com/sent-hil/bitesized"
)

func main() {
  redisuri := "localhost:6379"
  client, err := bitesized.NewClient(redisuri)
}
```

Optionally, set intervals you want to track; by default these intervals are tracked: `hourly, daily, weekly and monthly`:

```go
client.Intervals = []Interval{
  bitesized.Hour, bitesized.Daily, bitesized.Week, bitesized.Month,
}
```

Optionally, set prefix to use for ALL keys; defaults to `bitesized`:

```go
client.KeyPrefix = "bitesized"
```

Track an event that an user did:

```go
err = client.TrackEvent("dodge rock", "indianajones", time.Now())
```

Get count of users who did an event on particular interval:

```go
count, err = client.CountEvent("dodge rock", time.Now(), bitesized.Hour)
```

Check if user did an event for particular interval:

```go
didEvent, err := client.DidEvent("dodge rock", "indianajones", time.Now(), bitesized.Hour)
```

Get retention for specified interval:

```go
from := time.Date(2015, time.January, 1, 0, 0, 0, 0, time.UTC)
till := time.Date(2015, time.January, 3, 0, 0, 0, 0, time.UTC)

rs, err := client.GetRetention("dodge rock", bitesized.Daily, from, till)
```

This returns a result like below. The keys are sorted asc by time:

```
[
    { "day:2015-01-01 00:00:00 +0000 UTC": [ 30, 15, 3 ] },
    { "day:2015-01-02 00:00:00 +0000 UTC": [ 50, 25 ] },
    { "day:2015-01-03 00:00:00 +0000 UTC": [ 67 ] }
]
```

Get retention for specified interval in percentages:

```go
rs, err := client.GetRetention("dodge rock", bitesized.Daily, from, till)
```

This returns a result like below. The keys are sorted asc by time. The first entry is total number

```
[
    { "day:2015-01-01 00:00:00 +0000 UTC": [ 30, 50, 10 ] },
    { "day:2015-01-02 00:00:00 +0000 UTC": [ 50, 25 ] },
    { "day:2015-01-03 00:00:00 +0000 UTC": [ 67 ] }
]
```

Get list of events:

```go
// * returns all events
events, err := client.GetEvents("*")

// dodge* returns events with dodge prefix
events, err := client.GetEvents("dodge*")
```

Check if user was seen before:

```go
isUserNew, err := client.IsUserNew("indianajones")
```

Do a bitwise operation on key/keys:

```go
count, err := client.Operation(bitesized.AND, "dodge rock", "dodge nazis")
```

Following operations are support:

* AND
* OR
* XOR
* NOT (only accepts 1 arg)

Get list of users who did an event on particular time/interval:

```go
// returns list of users who did 'dodge rock' event in the last hour
users, err := client.EventUser("dodge rock", time.Now(), Hour)
```

# TODO

* Write blog post explaning bitmaps and this library
* List of events sorted DESC by user count metric
* Http server
* List of users who did an event metric
* List of users who didn't do an event metric
* Identify user with properties
* Total count of users metric
* Optimize bit maps storage?
* Add method to undo an event

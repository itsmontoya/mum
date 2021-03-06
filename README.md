# Enkodo (Marshal, unmarshal) [![GoDoc](https://godoc.org/github.com/mojura/enkodo?status.svg)](https://godoc.org/github.com/mojura/enkodo) ![Status](https://img.shields.io/badge/status-beta-yellow.svg) [![Go Report Card](https://goreportcard.com/badge/github.com/mojura/enkodo)](https://goreportcard.com/report/github.com/mojura/enkodo)

Enkodo is a no-frills encoder/decoder. It is focused around speed and simplicity. 

*Note: Enkodo is still in development and the API could potentially change. Usage is still safe as SEMVER will be respected*

## Benchmarks
```bash
% go test --bench=.
goos: darwin
goarch: amd64
pkg: github.com/mojura/enkodo

# Enkodo
EnkodoEncoding-4                          8989959    126 ns/op      0 B/op    0 allocs/op
EnkodoDecoding-4                          3464836    335 ns/op     16 B/op    1 allocs/op
EnkodoDecoding_no_string-4                3806563    308 ns/op      0 B/op    0 allocs/op
EnkodoEncoding_new_encoder-4              2792838    438 ns/op    296 B/op    6 allocs/op
EnkodoDecoding_new_decoder-4              3024243    393 ns/op     32 B/op    2 allocs/op
EnkodoDecoding_new_decoder_no_string-4    3174868    370 ns/op     16 B/op    1 allocs/op

# JSON
JSONEncoding-4                          843171    1355 ns/op       0 B/op     0 allocs/op
JSONDecoding-4                          201477    5679 ns/op     192 B/op    12 allocs/op
JSONDecoding_no_string-4                207198    5612 ns/op     176 B/op    11 allocs/op
JSONEncoding_new_encoder-4              869494    1379 ns/op       0 B/op     0 allocs/op
JSONDecoding_new_decoder-4              175564    6123 ns/op    1088 B/op    17 allocs/op
JSONDecoding_new_decoder_no_string-4    207910    6001 ns/op    1072 B/op    16 allocs/op

# Gob
# Note: Gob cannot compete in the decoder re-use benchmarks due to it's 
# requirement of initializing a new decoder for every test iteration
GOBEncoding-4                        1395951      827 ns/op       0 B/op      0 allocs/op
GOBEncoding_new_encoder-4             158697     7077 ns/op    1584 B/op     42 allocs/op
GOBDecoding_new_decoder-4              33736    33379 ns/op    8784 B/op    244 allocs/op
GOBDecoding_new_decoder_no_string-4    35460    33105 ns/op    8768 B/op    243 allocs/op

# Specific function benchmarks
EncodeInt64-4       132754987    8.85 ns/op    0 B/op    0 allocs/op
Encoder_Uint64-4    100000000    10.6 ns/op    0 B/op    0 allocs/op
EncodeUint64-4      209906836    5.67 ns/op    0 B/op    0 allocs/op
```

## Usage
```go
package main

import (
	"bytes"
	"log"

	"github.com/mojura/enkodo"
)

func main() {
	var (
		// Original user struct
		u User
		// New user struct (will be used to copy values to)
		nu  User
		err error
	)

	u.Email = "johndoe@gmail.com"
	u.Age = 46
	u.Twitter = "@johndoe"

	buffer := bytes.NewBuffer(nil)
	// Create a writer
	w := enkodo.NewWriter(buffer)
	// Encode user to buffer
	if err = w.Encode(&u); err != nil {
		log.Fatalf("Error encoding: %v", err)
	}

	// Decode new user from buffer
	if err = enkodo.Unmarshal(buffer.Bytes(), &nu); err != nil {
		log.Fatalf("Error decoding: %v", err)
	}

	log.Printf("New user: %v", nu)
}

// User holds the basic information for a user
type User struct {
	Email   string
	Age     uint8
	Twitter string
}

// MarshalEnkodo will marshal a User
func (u *User) MarshalEnkodo(enc *enkodo.Encoder) (err error) {
	enc.String(u.Email)
	enc.Uint8(u.Age)
	enc.String(u.Twitter)
	return
}

// UnmarshalEnkodo will unmarshal a User
func (u *User) UnmarshalEnkodo(dec *enkodo.Decoder) (err error) {
	if u.Email, err = dec.String(); err != nil {
		return
	}

	if u.Age, err = dec.Uint8(); err != nil {
		return
	}

	if u.Twitter, err = dec.String(); err != nil {
		return
	}

	return
}

```

## Features
- [x] Support for basic primitives
- [x] Encoding via helper funcs
- [x] Decoding via helper funcs
- [ ] Encoding helper funcs created by reflection (Next release)
- [ ] Decoding helper funcs created by reflection (Next release)

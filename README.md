# goavro

## Description

Goavro is a golang library that implements encoding and decoding of
Avro data. It provides an interface to encode data directly to
`io.Writer` streams, and decoding data from `io.Reader`
streams. Goavro fully adheres to
[version 1.7.7 of the Avro specification](http://avro.apache.org/docs/1.7.7/spec.html).

## Resources

* [Avro](http://avro.apache.org/)
* [Avro CLI Examples](https://github.com/miguno/avro-cli-examples)
* [JavaScript Object Notation, JSON](http://www.json.org/)
* [http://kafka.apache.org](Kafka)

## Usage

Although the Avro specification defines the terms reader and writer as
library components which read and write Avro data, Go has particular
strong emphasis on what a Reader and Writer are. Namely, it is bad
form to define an interface which shares the same name but uses a
different method signature. In other words, all Reader interfaces
should effectively mirror an `io.Reader`, and all Writer interfaces
should mirror an `io.Writer`. Adherance to this standard is essential
to keep libraries easy to use.

An `io.Reader` reads data from the stream specified at object creation
time into the parameterized slice of bytes and returns both the number
of bytes read and an error. An Avro reader also reads from a stream,
but it is possible to create an Avro reader that can read from one
stream, then read from another, using the same compiled schema. In
other words, an Avro reader puts the schema first, whereas an
`io.Reader` puts the stream first.

To support an Avro reader being able to read from multiple streams,
it's API must be different and incompatible with `io.Reader` interface
from the Go standard. Instead, an Avro reader looks more like the
Unmarshal functionality provided by the Go `encoding/json` library.

### Codec interface

Creating a `goavro.Codec` is fast, but ought to be performed exactly
once per Avro schema to process. Once a `Codec` is created, it may be
used multiple times to either decode or encode data.

The `Codec` interface exposes two methods, one to encode data and one
to decode data. They encode directly into an `io.Writer`, and decode
directly from an `io.Reader`.

A particular `Codec` can work with only one Avro schema. However,
there is no practical limit to how many `Codec`s may be created and
used in a program. Internally a `goavro.codec` is merely a namespace
and two function pointers to decode and encode data. Because `codec`s
maintain no state, the same `Codec` can be concurrently used on
different `io` streams as desired.

```Go
    func (c *codec) Decode(r io.Reader) (interface{}, error)
    func (c *codec) Encode(w io.Writer, datum interface{}) error
```

### Creating a `Codec`

The below is an example of creating a `Codec` from a provided JSON
schema. `Codec`s do not maintain any internal state, and may be used
multiple times on multiple `io.Reader`s, `io.Writer`s, concurrently if
desired.

```Go
    codec, err := goavro.NewCodec(someJsonSchema)
    if err != nil {
        return nil, err
    }
```

### Decoding data

The below is a simplified example of decoding binary data to be read
from an `io.Reader` into a single datum using a previously compiled
`Codec`. The `Decode` method of the `Codec` interface may be called
multiple times, each time on the same or on different `io.Reader`
objects.

```Go
    // uses codec created above, and an io.Reader, definition not shown
    datum, err := codec.Decode(r)
    if err != nil {
        return nil, err
    }
```

### Encoding data

The below is a simplified example of encoding a single datum into the
Avro binary format using a previously compiled `Codec`. The `Encode`
method of the `Codec` interface may be called multiple times, each
time on the same or on different `io.Writer` objects.

```Go
    // uses codec created above, an io.Writer, definition not shown,
    // and some data
    err := codec.Encode(w, datum)
    if err != nil {
        return nil, err
    }
```

Another example, this time leveraging `bufio.Writer`:

```Go
    // Encoding data using bufio.Writer to buffer the writes
    // during data encoding:
 
    func encodeWithBufferedWriter(c Codec, w io.Writer, datum interface{}) error {
     	bw := bufio.NewWriter(w)
     	err := c.Encode(bw, datum)
     	if err != nil {
     		return err
     	}
     	return bw.Flush()
    }
 
    err := encodeWithBufferedWriter(codec, w, datum)
    if err != nil {
        return nil, err
    }
```

## Limitations

Goavro is a fully featured encoder and decoder of binary Avro data. It
fully supports recursive data structures, unions, and namespacing. It
does have a few limitations that have yet to be implemented.

### Aliases

The Avro specification allows an implementation to optionally map a
writer's schema to a reader's schema using aliases. Although goavro
can complile schemas with aliases, it does not yet implement this
feature.

### JSON Encoding

The Avro Data Serialization format describes two encodings: binary and
JSON. Goavro only implements binary encoding of data streams, because
that is what most applications need.

> Most applications will use the binary encoding, as it is smaller and
> faster. But, for debugging and web-based applications, the JSON
> encoding may sometimes be appropriate.

Note that data schemas are always encoded using JSON, as per the
specification.

### Avro Object Container Files

Goavro can decode binary encoded Avro data, and encode data into
Avro's binary encoding. Goavro does not presently handle
[http://avro.apache.org/docs/1.7.7/spec.html#Object+Container+Files](Avro
Object Container Files). The Avro Object Container Files format is a
file encoding schema that sits on top of the Avro Data Serialization
format. This has been implemented but removed until a future release
once the API can be improved.

Recall that Avro Data Serialization allows for two encodings: binary
and JSON. Avro Object Container Files is a higher layer of abstraction
above Avro Data Serialization that allows for `null`, `deflate`, and
optionally the `snappy` codecs for storing data in a file. All of
these compression codecs will be supported once Avro Object Container
Files support is reintegrated.

### Kafka Streams

[http://kafka.apache.org](Kafka) is the reason goavro was
written. Similar to Avro Object Container Files being a layer of
abstraction above Avro Data Serialization format, Kafka's use of Avro
is a layer of abstraction that also sits above Avro Data Serialization
format, but has its own schema. Like Avro Object Container Files, this
has been implemented but removed until the API can be improved.

## License

### Goavro license

Copyright 2015 LinkedIn Corp. Licensed under the Apache License,
Version 2.0 (the "License"); you may not use this file except in
compliance with the License.  You may obtain a copy of the License at
[http://www.apache.org/licenses/LICENSE-2.0](http://www.apache.org/licenses/LICENSE-2.0).

Unless required by applicable law or agreed to in writing, software
 distributed under the License is distributed on an "AS IS" BASIS,
 WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or
implied.Copyright [201X] LinkedIn Corp. Licensed under the Apache
License, Version 2.0 (the "License"); you may not use this file except
in compliance with the License.  You may obtain a copy of the License
at
[http://www.apache.org/licenses/LICENSE-2.0](http://www.apache.org/licenses/LICENSE-2.0).

Unless required by applicable law or agreed to in writing, software
 distributed under the License is distributed on an "AS IS" BASIS,
 WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or
implied.

## Third Party Dependencies

Goavro has no third party dependencies outside of the standard library
included with the Go language.

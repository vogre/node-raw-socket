
# raw-socket - [homepage][homepage]

This module implements raw sockets for [Node.js][nodejs].

*This module has been created primarily to implement another, separate module,
for the purpose of sending ICMP echo (ping) requests to many hosts at once.
That module will be available shortly.*

This module is installed using [node package manager (npm)][npm]:

    # This module contains C++ source code which will be compiled
    # during installation using node-gyp.  A suitable build chain
    # must be configured before installation.
    
    npm install raw-socket

It is loaded using the `require()` function:

    var raw = require ("raw-socket");

Raw sockets can then be created, and data sent using [Node.js][nodejs]
`Buffer` objects:

    var socket = raw.createSocket ({protocol: raw.Protocol.None});

    socket.on ("message", function (buffer, source) {
        console.log ("received " + buffer.length + " bytes from " + source);
    });
    
    socket.send (buffer, 0, buffer.length, "1.1.1.1", function (error, bytes) {
        if (error)
            console.log (error.toString ());
    });

[homepage]: http://re-tool.org "Homepage"
[nodejs]: http://nodejs.org "Node.js"
[npm]: https://npmjs.org/ "npm"

# Network Protocol Support

The raw sockets exposed by this module support IPv4 (soon to include IPv6).

Raw sockets are created using the operating systems `socket()` function, and
the socket type `SOCK_RAW` specified.

# Constants

The following sections describe constants exported and used by this module.

## raw.Protocol

This object contains constants which can be used for the `protocol` option to
the `createSocket()` function exposed by this module.  This option specifies
the protocol number to place in the protocol field of IP headers generated by
the operating system.

The following constants are defined in this object:

 * `None` - protocol number 0
 * `ICMP` - protocol number 1
 * `TCP` - protocol number 6
 * `UDP` - protocol number 17

# Using This Module

Raw sockets are represented by an instance of the `Socket` class.  This
module exports the `createSocket()` function which is used to create
instances of the `Socket` class.

## raw.createSocket ([options])

The `createSocket()` function instantiates and returns an instance of the
`Socket` class:

    // Default options
    var options = {
        protocol: raw.Protocol.ICMP,
        bufferSize: 4096
    };
    
    var socket = raw.createSocket (options);

The optional `options` parameter is an object, and can contain the following
items:

 * `protocol` - Either one of the constants defined in the `raw.Protocol`
   object or the protocol number to use for the socket, defaults to the
   consant `raw.Protocol.None`
 * `bufferSize` - Size, in bytes, of the sockets internal receive buffer,
   defaults to 4096

An exception will be thrown if the underlying raw socket could not be created.
The error will be an instance of the `Error` class.

When data is sent using the instantiated raw socket the operating system will
automatically build IP headers and place them in outgoing packets.

The `protocol` parameter, or its default value of the constant
`raw.Protocol.None`, will be specified in the protocol field of each IP
header.

Upon receiving packets IP headers are **NOT** removed by the operating system.
IP headers in received packets will be included in data presented by this
module.

This module hopes to support the `IP_HDRINCL` socket option in the near
future, which controls the inclusion of IP headers in outgoing packets.

## socket.on ("close", callback)

The `close` event is emitted by the socket when the underlying raw socket
is closed.

No arguments are passed to the callback.

The following example prints a message to the console when the socket is
closed:

    socket.on ("close", function () {
        console.log ("socket closed");
    });

## socket.on ("error", callback)

The `error` event is emitted by the socket when an error occurs sending or
receiving data.

The following arguments will be passed to the `callback` function:

 * `error` - An instance of the `Error` class, the exposed `message` attribute
   will contain a detailed error message.

The following example prints a message to the console when an error occurs,
after which the socket is closed:

    socket.on ("error", function (error) {
        console.log (error.toString ());
        socket.close ();
    });

## socket.on ("message", callback)

The `message` event is emitted by the socket when data has been received.

The following arguments will be passed to the `callback` function:

 * `buffer` - A [Node.js][nodejs] `Buffer` object containing the data
   received, the buffer will be sized to fit the data received, that is the
   `length` attribute of buffer will specify how many bytes were received
 * `address` - Dotted quad formatted source IP address of the message, e.g
   "192.168.1.254"

The following example prints received messages in hexadecimal to the console:

    socket.on ("message", function (message, address) {
        console.log ("received " + buffer.length + " bytes from " + address
                + ": " + buffer.toString ("hex"));
    });

## raw.send (buffer, offset, length, address, callback)

The `send()` method sends data to a remote host.

The `buffer` parameter is a [Node.js][nodejs] `Buffer` object containing the
data to be sent.  The `length` parameter specifies how many bytes from
`buffer`, beginning at offset `offset`, to send.  The `address` parameter
contains the dotted quad formatted IP address of the remote host to send the
data to.  The `callback` function is called once the data has been sent.  The
following arguments will be passed to the `callback` function:

 * `error` - Instance of the `Error` class, or `null` if no error occurred
 * `bytes` - Number of bytes sent

The following example sends a ICMP ping message to a remote host:

    // ICMP echo (ping) request, checksum should be ok
    var buffer = new Buffer ([
            0x08, 0x00, 0x43, 0x52, 0x00, 0x01, 0x0a, 0x09,
            0x61, 0x62, 0x63, 0x64, 0x65, 0x66, 0x67, 0x68,
            0x69, 0x6a, 0x6b, 0x6c, 0x6d, 0x6e, 0x6f, 0x70,
            0x71, 0x72, 0x73, 0x74, 0x75, 0x76, 0x77, 0x61,
            0x62, 0x63, 0x64, 0x65, 0x66, 0x67, 0x68, 0x69]);

    socket.send (buffer, 0, buffer.length, target, function (error, bytes) {
        if (error)
            console.log (error.toString ());
        else
            console.log ("sent " + bytes + " bytes");
    });

# Example Programs

Example programs are included under the modules `example` directory.

# Bugs & Known Issues

None, yet!

Bug reports should be sent to <stephen.vickers.sv@gmail.com>.

# Changes

## Version 1.0.0 - 29/01/2013

 * Initial release

## Version 1.0.1 - ?

 * Move `SOCKET_ERRNO` define from `raw.cc` to `raw.h`
 * Error in exception thrown by `SocketWrap::New` in `raw.cc` stated that two
   arguments were required, this should be one
 * Corrections to the README.md

# Roadmap

In no particular order:

 * Enhance performance by moving the send queue into the C++ raw::SocketWrap
   class
 * Support `IP_HDRINCL` socket option
 * Support IPv6

Suggestions and requirements should be sent to <stephen.vickers.sv@gmail.com>.

# License

This program is free software: you can redistribute it and/or modify it under
the terms of the GNU General Public License as published by the Free Software
Foundation, either version 3 of the License, or (at your option) any later
version.

This program is distributed in the hope that it will be useful, but WITHOUT
ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS
FOR A PARTICULAR PURPOSE.  See the GNU General Public License for more
details.

You should have received a copy of the GNU General Public License along with
this program.  If not, see
[http://www.gnu.org/licenses](http://www.gnu.org/licenses).

# Author

Stephen Vickers <stephen.vickers.sv@gmail.com>

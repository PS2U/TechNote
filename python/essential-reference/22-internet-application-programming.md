# 22.1 `ftplib`

The `ftplib` module implements the client side of the FTP protocol. It’s rarely necessary to use this module directly because the `urllib` package provides a higher-level interface. However, this module may still be useful if you want to have more control over the low-level details of an FTP connection.

`FTP([host [, user [, passwd [, acct [, timeout]]]]])` Creates an object representing an FTP connection.

An instance `f` of FTP has the following methods:   

- `f.abort()` Attempts to abort a file transfer that is in progress.
- `f.close()` Closes the FTP connection.
- `f.connect(host [, port [, timeout]])` Opens an FTP connection to a given host and port.
- `f.cwd(pathname)` Changes the current working directory on the server to pathname.   
- `f.delete(filename)` Removes the file filename from the server.
- `f.dir([dirname [, ... [, callback]]])` Generates a directory listing as produced by the 'LIST' command.
- `f.login([user, [passwd [, acct]]])` Logs in to the server using the specified username, password, and account.
- `f.quit()` Closes the FTP connection by sending the 'QUIT' command to the server.
- `f.retrbinary(command, callback [, blocksize [, rest]])` Returns the results of executing a command on the server using binary transfer mode.
- `f.retrlines(command [, callback])` Returns the results of executing a command on the server using text transfer mode.
- `f.sendcmd(command)` Sends a simple command to the server and returns the server response.


# 22.2 `http`

The `http` package consists of modules for writing HTTP clients and servers as well as support for state management (cookies).

## `http.client`

The `http.client` module provides low-level support for the client side of HTTP. In Python 2, this module is called `httplib`.

`HTTPConnection(host [,port])`   Creates an HTTP connection.

`HTTPSConnection(host [, port [, key_file=kfile [, cert_file=cfile ]]])`   Creates an HTTP connection but uses a secure socket.

- `h.connect()`   Initializes the connection to the host and port given to `HTTPConnection()` or `HTTPSConnection()`.
- `h.close()`   Closes the connection.   
- `h.send(bytes)`   Sends a byte string, bytes, to the server.
- `h.putrequest(method, selector [, skip_host [, skip_accept_encoding]]`)   Sends a request to the server.
- `h.putheader(header, value, ...)`   Sends an RFC-822–style header to the server.
- `h.request(method, url [, body [, headers]])`   Sends a complete HTTP request to the server.
- `h.getresponse()`   Gets a response from the server and returns an HTTPResponse instance that can be used to read data.

An `HTTPResponse` instance, `r`, as returned by the `getresponse()` method, supports the following methods:   

- `r.read([size])`   Reads up to size bytes from the server.
- `r.getheaders()`   Returns a list of `(header, value)` tuples.
- `r.status`   HTTP status code returned by the server.

## `http.server`

The `http.server` module provides various classes for implementing HTTP servers. In Python 2, the contents of this module are split across three library modules: `BaseHTTPServer`, `CGIHTTPServer`, and `SimpleHTTPServer`.

The following class implements a basic HTTP server. In Python 2, it is located in the BaseHTTPServer module.   

`HTTPServer(server_address, request_handler)`   Creates a new HTTPServer object.

`HTTPServer` inherits directly from `TCPServer` defined in the `socketserver` module.

`SimpleHTTPRequestHandler` and `CGIHTTPRequestHandler` Two prebuilt web server handler classes can be used if you want to quickly set up a simple stand-alone web server.

The `BaseHTTPRequestHandler` class is a base class that’s used if you want to define your own custom HTTP server handling. The prebuilt handlers such as `SimpleHTTPRequestHandler` and `CGIHTTPRequestHandler` inherit from this. In Python 2, this class is defined in the `BaseHTTPServer` module.

`BaseHTTPRequestHandler.error_message_format` Format string used to build error messages sent to the client.

`BaseHTTPRequestHandler.responses`   Mapping of integer HTTP error codes to two-element tuples (message, explain) that describe the problem.

When created to handle a connection, an instance, `b`, of `BaseHTTPRequestHandler` has the following attributes:

- `b.send_error(code [, message])` Sends a response for an unsuccessful request.
- `b.send_response(code [, message])` Sends a response for a successful request.
- `b.send_header(keyword, value)` Writes a MIME header entry to the output stream.
- `b.log_error(format, ...)` Logs an error message.

## `http.cookies`

The `http.cookies` module provides server-side support for working with HTTP cookies. In Python 2, the module is called Cookie.

The` http.cookies` module simplifies the task of generating cookie values by providing a special dictionary-like object which stores and manages collections of cookie values known as **morsels**. Each morsel has a name, a value, and a set of optional attributes containing metadata to be supplied to the browser `{expires, path, comment, domain, max-age, secure, version, httponly}`.

`SimpleCookie([input])`   Defines a cookie object in which cookie values are stored as simple strings.

- `c.output([attrs [,header [,sep]]])`   Generates a string suitable for use in setting cookie values in HTTP headers.
- `c.load(rawdata)`   Loads the cookie c with data found in rawdata.

In addition, the morsel m has the following methods and attributes:   

- `m.value`   A string containing the raw value of the cookie
- `m.set(key,value,coded_value)`   Sets the values of m.key, m.value, and m.
- `m.output([attrs [,header]])`   Produces the HTTP header string for this morsel.

## `http.cookiejar`

The `http.cookiejar` module provides client-side support for storing and managing HTTP cookies. In Python 2, the module is called `cookielib`.

`CookieJar()` An object that manages HTTP cookie values, storing cookies received as a result of HTTP requests, and adding cookies to outgoing HTTP requests.

# 22.3 `smtplib`
# 22.4 `urllib`
# 22.5 `xmlrpc`

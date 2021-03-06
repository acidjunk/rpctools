# rpctools

The rpctools package provides RPC clients (currently JSON-RPC, eventually XML-RPC too) with enhancements such
as improved SSL support and connection pooling.

Currently the only protocol implemented is JSON-RPC.  The enhanced SSL support is simply that these libraries
can present client certificates for autentication and can be setup to require a trusted SSL connection with
the server (validating CA and hsotname matches).

This project was created to address the need for better SSL support when using JSON-RPC, in particular to allow for
the use of client certificates in authentication and to validate server certificates.  Python's SSL defaults tend
to forgo security for the sake of "ease of use"; we actually want to use SSL for its security features.

** IMPORTANT **
This library is alpha-quality and should be considered a preview.  It needs unit/functional tests (and will be getting
them).  While there are no plans to change the API for JSON-RPC, but more features will be added and backwards-incompatible
changes may be introduced.

## Known Limitations / Plans

- Unit/functional tests are current #1 priority.
- Currently only JSON-RPC is supported.  We plan to add support for XML-RPC also.
- The connection pooling system should be considered alpha-quality.  We would love feedback, but don't expect
  a bug-free experience.

## Installation

From PyPI using pip:

    pip install rpctools

Or from source checkout:

    git clone https://github.com/appliedsec/rpctools
    cd rpctools
    python setup.py install

## Run tests

Once installed, install pytest:

    pip install pytest

Run tests with:

    py.test tests

## Basic Usage

The JSON-RPC API is modeled after Python's [xmlrpclib](http://docs.python.org/2/library/xmlrpclib.html) API and
also draws some inspiration from other Python clients -- e.g. the [Redis client](https://github.com/andymccurdy/redis-py)
for connection pooling.

### Connecting to an HTTPS Server

```python
from rpctools.jsonrpc import ServerProxy, Fault

proxy = ServerProxy('https://example.com/jsonrpc', ssl_opts={
    'ca_certs': '/path/to/ca-bundle.crt', # PEM-encoded contatenated set of CA certificates
})
try:
    proxy.someServerMethod(param1, param2)
except Fault:
    raise  # Fault instances are used to communicate server-side exceptions.
```

### ... with basic auth

The underlying httplib library supports providing basic auth in the URI:

```python
from rpctools.jsonrpc import ServerProxy, Fault

proxy = ServerProxy('https://foo:pass@example.com/jsonrpc')
try:
    proxy.requiresAuth(param1, param2)
except Fault:
    raise  # Fault instances are used to communicate server-side exceptions.
```

### ... or client certs

```python
from rpctools.jsonrpc import ServerProxy, Fault

proxy = ServerProxy('https://example.com/jsonrpc', ssl_opts={
    keyfile='/path/to/client.key',   # PEM-encoded
    certfile='/path/to/client.crt',  # PEM-encoded
    ca_certs='/path/to/ca-bundle.crt'
})
try:
    proxy.someServerMethod(param1, param2)
except Fault:
    raise  # Fault instances are used to communicate server-side exceptions.
```

### ... connection pooling (ALPHA!)

If you are going to make many repeated calls to the server, you may find it helpful
to use the connection pool feature.

```python
from rpctools.jsonrpc import ServerProxy, Fault

proxy = ServerProxy('http://example.com/jsonrpc', pool_connections=True)
for (param1, param2) in some_params_list:
    try:
        proxy.someServerMethod(param1, param2)
    except Fault:
        raise  # Fault instances are used to communicate server-side exceptions.
```

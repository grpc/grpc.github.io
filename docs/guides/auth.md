---
layout: docs
title: Authentication
---

<h1 class="page-header">Authentication</h1>

<p class="lead">gRPC is designed to plug-in a number of authentication mechanisms. This document provides a quick overview of the various auth mechanisms supported, discusses the API with some examples, and concludes with a discussion of extensibility.</p>

More documentation and examples are coming soon!

<div id="toc"></div>

## Supported auth mechanisms

###SSL/TLS

gRPC has SSL/TLS integration and promotes the use of SSL/TLS to authenticate the server,
and encrypt all the data exchanged between the client and the server. Optional
mechanisms are available for clients to provide certificates to accomplish mutual
authentication.

###OAuth 2.0

gRPC provides a generic mechanism (described below) to attach metadata based credentials
to requests and responses. Additional support for acquiring Access Tokens while
accessing Google APIs through gRPC is provided for certain auth flows, demonstrated
through code examples below.

*WARNING*: Google OAuth2 credentials shall only be used to connect to Google services.
Sending a Google issued OAuth2 token to a non-Google service could result in this token
being stolen and used to impersonate the client to Google services.

## API

To reduce complexity and minimize API clutter, gRPC works with a unified concept of
Credentials objects.

Credentials can be of two types:

- *Channel Credentials*, which are attached to a `Channel` such as SSL credentials.
- *Call Credentials*, which are attached to a call (or `ClientContext` in C++).

Credentials can be composed using `CompositeChannelCredentials`. A
`CompositeChannelCredentials` associates a `ChannelCredentials` and a
`CallCredentials` to create a new `ChannelCredentials`. The result will, for
each call on the channel, send the authentication data associated with the
composed `CallCredentials`.

For example, a `ChannelCredentials` could be created from an `SslCredentials`
and an `AccessTokenCredentials`. The result when applied to a `Channel` would
send the appropriate access token for each call on this channel.

`CallCredentials` can also be composed using `CompositeCallCredentials`. The
resulting `CallCredentials`, when applied to a `ClientContext`, will trigger the
sending of the authentication data associated with the two `CallCredentials`.


###SSL/TLS for server authentication and encryption

This is the simplest authentication scenario, where a client just wants to
authenticate the server and encrypt all data.

```cpp
// Create a default SSL ChannelCredentials object.
auto channel_creds = grpc::SslCredentials(grpc::SslCredentialsOptions());
// Create a channel using the credentials created in the previous step.
auto channel = grpc::CreateChannel(server_name, creds);
// Create a stub on the channel.
std::unique_ptr<Greeter::Stub> stub(Greeter::NewStub(channel));
// Make actual RPC calls on the stub.
grpc::Status s = stub->sayHello(&context, *request, response);
```

For advanced use cases such as modifying the root CA or using client certs,
the corresponding options can be set in the SslCredentialsOptions parameter
passed to the factory method.


### Authenticating with Google

gRPC applications can use a simple API to create a credential that works in
various deployment scenarios.

```cpp
auto creds = grpc::GoogleDefaultCredentials();
// Create a channel, stub and make RPC calls (same as in the previous example)
auto channel = grpc::CreateChannel(server_name, creds);
std::unique_ptr<Greeter::Stub> stub(Greeter::NewStub(channel));
grpc::Status s = stub->sayHello(&context, *request, response);
```

This channel credentials object works for applications using Service Accounts as
well as for applications running in [Google Compute Engine
(GCE)](https://cloud.google.com/compute/).  In the former case, the service
account’s private keys are loaded from the file named in the environment
variable `GOOGLE_APPLICATION_CREDENTIALS`. The keys are used to generate bearer
tokens that are attached to each outgoing RPC on the corresponding channel.

For applications running in GCE, a default service account and corresponding
OAuth2 scopes can be configured during VM setup. At run-time, this credential
handles communication with the authentication systems to obtain OAuth2 access
tokens and attaches them to each outgoing RPC on the corresponding channel.


### Extending gRPC to support other authentication mechanisms

The Credentials plugin API allows developers to plug in their own type of
credentials.

- The `MetadataCredentialsPlugin` abstract class contains the pure virtual
  `GetMetadata` method that needs to be implemented by a sub-class created by
  the developer.
- The `MetadataCredentialsFromPlugin` function which creates a `CallCredentials`
  from the `MetadataCredentialsPlugin`.

Here is example of a simple credentials plugin which sets an authentication
ticket in a custom header.

```cpp
class MyCustomAuthenticator : public grpc::MetadataCredentialsPlugin {
 public:
  MyCustomAuthenticator(const grpc::string& ticket) : ticket_(ticket) {}

  grpc::Status GetMetadata(
      grpc::string_ref service_url, grpc::string_ref method_name,
      const grpc::AuthContext& channel_auth_context,
      std::multimap<grpc::string, grpc::string>* metadata) override {
    metadata->insert(std::make_pair("x-custom-auth-ticket", ticket_));
    return grpc::Status::OK;
  }

 private:
  grpc::string ticket_;
};

auto call_creds = grpc::MetadataCredentialsFromPlugin(
    std::unique_ptr<grpc::MetadataCredentialsPlugin>(
        new MyCustomAuthenticator("super-secret-ticket")));
```

A deeper integration can be achieved by plugging in a gRPC credentials
implementation at the core level. gRPC internals also allow switching out
SSL/TLS with other encryption mechanisms.

## Examples

These authentication mechanisms will be available in all gRPC's supported languages.
The following sections demonstrate how authentication and authorization features
described above appear in each language: more languages are coming soon.

###SSL/TLS for server authentication and encryption (Ruby)

```ruby
# Base case - No encryption
stub = Helloworld::Greeter::Stub.new('localhost:50051', :this_channel_is_insecure)
...

# With server authentication SSL/TLS
creds = GRPC::Core::Credentials.new(load_certs)  # load_certs typically loads a CA roots file
stub = Helloworld::Greeter::Stub.new('localhost:50051', creds)
```

###SSL/TLS for server authentication and encryption (C#)

```csharp
// Base case - No encryption/authentication
var channel = new Channel("localhost:50051", ChannelCredentials.Insecure);
var client = new Greeter.GreeterClient(channel);
...

// With server authentication SSL/TLS
var channelCredentials = new SslCredentials(File.ReadAllText("roots.pem"));  // Load a custom roots file.
var channel = new Channel("myservice.example.com", channelCredentials);
var client = new Greeter.GreeterClient(channel);
```

###SSL/TLS for server authentication and encryption (Python)

```python
from grpc.beta import implementations
import helloworld_pb2

# Base case - No encryption
channel = implementations.insecure_channel('localhost', 50051)
stub = helloworld_pb2.beta_create_Greeter_stub(channel)
...

# With server authentication SSL/TLS
creds = implementations.ssl_channel_credentials(open('roots.pem').read(), None, None)
channel = implementations.secure_channel('localhost', 50051, creds)
stub = helloworld_pb2.beta_create_Greeter_stub(channel)
```

###SSL/TLS for server authentication and encryption (Java)

In Java we recommend that you use OpenSSL when using gRPC over TLS. You can find details about installing and using OpenSSL and other required libraries for both Android and non-Android Java in the gRPC Java [Security](https://github.com/grpc/grpc-java/blob/master/SECURITY.md#transport-security-tls) documentation.

To enable TLS on a server, a certificate chain and private key need to be specified in PEM format. The standard TLS port is 443, but we use 8443 below to avoid needing extra permissions from the OS.

```java
Server server = ServerBuilder.forPort(8443)
    // Enable TLS
    .useTransportSecurity(certChainFile, privateKeyFile)
    .addService(TestServiceGrpc.bindService(serviceImplementation))
    .build();
server.start();
```
If the issuing certificate authority is not known to the client then a properly configured `SslContext` or `SSLSocketFactory` should be provided to the `NettyChannelBuilder` or `OkHttpChannelBuilder`, respectively.

On the client side, server authentication with SSL/TLS looks like this:

```java
// Base case - No encryption
ManagedChannel channel = ManagedChannelBuilder.forAddress("localhost", 50051)
    .usePlaintext(true)
    .build();
GreeterGrpc.GreeterStub stub = GreeterGrpc.newStub(channel);

// With server authentication SSL/TLS
ManagedChannel channel = ManagedChannelBuilder.forAddress("localhost", 50051)
    .build();
GreeterGrpc.GreeterStub stub = GreeterGrpc.newStub(channel);

// With server authentication SSL/TLS; custom CA root certificates; not on Android
ManagedChannel channel = NettyChannelBuilder.forAddress("localhost", 50051)
    .sslContext(GrpcSslContexts.forClient().trustManager(new File("roots.pem")).build())
    .build();
GreeterGrpc.GreeterStub stub = GreeterGrpc.newStub(channel);
```

###Authenticating with Google (Ruby)

####Base case - No encryption/authentication

```ruby
stub = Helloworld::Greeter::Stub.new('localhost:50051', :this_channel_is_insecure)
```

####Authenticate using scopeless credentials (recommended approach)

```ruby
require 'googleauth'  # from http://www.rubydoc.info/gems/googleauth/0.1.0
...
ssl_creds = GRPC::Core::ChannelCredentials.new(load_certs)  # load_certs typically loads a CA roots file
authentication = Google::Auth.get_application_default()
call_creds = GRPC::Core::CallCredentials.new(authentication.updater_proc)
combined_creds = ssl_creds.compose(call_creds)
stub = Helloworld::Greeter::Stub.new('greeter.googleapis.com', combined_creds)
```

####Authenticate using Oauth2 token (legacy approach)

```ruby
require 'googleauth'  # from http://www.rubydoc.info/gems/googleauth/0.1.0
...
ssl_creds = GRPC::Core::ChannelCredentials.new(load_certs)  # load_certs typically loads a CA roots file
scope = 'https://www.googleapis.com/auth/grpc-testing'
authentication = Google::Auth.get_application_default(scope)
call_creds = GRPC::Core::CallCredentials.new(authentication.updater_proc)
combined_creds = ssl_creds.compose(call_creds)
stub = Helloworld::Greeter::Stub.new('greeter.googleapis.com', combined_creds)
```

###Authenticating with Google (Node.js)

####Base case - No encryption/authentication

```js
var stub = new helloworld.Greeter('localhost:50051', grpc.credentials.createInsecure());
```

####Authenticate using scopeless credentials (recommended approach)

```js
// Authenticating with Google
var GoogleAuth = require('google-auth-library'); // from https://www.npmjs.com/package/google-auth-library
...
var ssl_creds = grpc.credentials.createSsl(root_certs);
(new GoogleAuth()).getApplicationDefault(function(err, auth) {
  var call_creds = grpc.credentials.createFromGoogleCredential(auth);
  var combined_creds = grpc.credentials.combineChannelCredentials(ssl_creds, call_creds);
  var stub = new helloworld.Greeter('greeter.googleapis.com', combined_credentials);
});
```

####Authenticate using Oauth2 token (legacy approach)

```js
var GoogleAuth = require('google-auth-library'); // from https://www.npmjs.com/package/google-auth-library
...
var ssl_creds = grpc.Credentials.createSsl(root_certs); // load_certs typically loads a CA roots file
var scope = 'https://www.googleapis.com/auth/grpc-testing';
(new GoogleAuth()).getApplicationDefault(function(err, auth) {
  if (auth.createScopeRequired()) {
    auth = auth.createScoped(scope);
  }
  var call_creds = grpc.credentials.createFromGoogleCredential(auth);
  var combined_creds = grpc.credentials.combineChannelCredentials(ssl_creds, call_creds);
  var stub = new helloworld.Greeter('greeter.googleapis.com', combined_credentials);
});
```

###Authenticating with Google (C#)

####Base case - No encryption/authentication
```csharp
var channel = new Channel("localhost:50051", ChannelCredentials.Insecure);
var client = new Greeter.GreeterClient(channel);
...
```

####Authenticate using scopeless credentials (recommended approach)
```csharp
using Grpc.Auth;  // from Grpc.Auth NuGet package
...
// Loads Google Application Default Credentials with publicly trusted roots.
var channelCredentials = await GoogleGrpcCredentials.GetApplicationDefaultAsync();

var channel = new Channel("greeter.googleapis.com", channelCredentials);
var client = new Greeter.GreeterClient(channel);
...
```

####Authenticate using OAuth2 token (legacy approach)
```csharp
using Grpc.Auth;  // from Grpc.Auth NuGet package
...
string scope = "https://www.googleapis.com/auth/grpc-testing";
var googleCredential = await GoogleCredential.GetApplicationDefaultAsync();
if (googleCredential.IsCreateScopedRequired)
{
    googleCredential = googleCredential.CreateScoped(new[] { scope });
}
var channel = new Channel("greeter.googleapis.com", googleCredential.ToChannelCredentials());
var client = new Greeter.GreeterClient(channel);
...
```

####Authenticate a single RPC call
```csharp
var channel = new Channel("greeter.googleapis.com", new SslCredentials());  // Use publicly trusted roots.
var client = new Greeter.GreeterClient(channel);
...
var googleCredential = await GoogleCredential.GetApplicationDefaultAsync();
var result = client.SayHello(request, new CallOptions(credentials: googleCredential.ToCallCredentials()));
...

```

###Authenticating with Google (PHP)

####Base case - No encryption/authorization

```php
$client = new helloworld\GreeterClient('localhost:50051', [
    'credentials' => Grpc\ChannelCredentials::createInsecure(),
]);
...
```

####Authenticate using scopeless credentials (recommended approach)

```php
function updateAuthMetadataCallback($context)
{
    $auth_credentials = ApplicationDefaultCredentials::getCredentials();
    return $auth_credentials->updateMetadata($metadata = [], $context->service_url);
}
$channel_credentials = Grpc\ChannelCredentials::createComposite(
    Grpc\ChannelCredentials::createSsl(file_get_contents('roots.pem')),
    Grpc\CallCredentials::createFromPlugin('updateAuthMetadataCallback')
);
$opts = [
  'credentials' => $channel_credentials
];
$client = new helloworld\GreeterClient('greeter.googleapis.com', $opts);
````

####Authenticate using Oauth2 token (legacy approach)

```php
// the environment variable "GOOGLE_APPLICATION_CREDENTIALS" needs to be set
$scope = "https://www.googleapis.com/auth/grpc-testing";
$auth = Google\Auth\ApplicationDefaultCredentials::getCredentials($scope);
$opts = [
  'credentials' => Grpc\Credentials::createSsl(file_get_contents('roots.pem'));
  'update_metadata' => $auth->getUpdateMetadataFunc(),
];
$client = new helloworld\GreeterClient('greeter.googleapis.com', $opts);
```

###Authenticating with Google (Python)

####Base case - No encryption/authentication
```python
channel = implementations.insecure_channel('localhost', 50051)
stub = helloworld_pb2.beta_create_Greeter_stub(channel)
...
```

####Authenticate using OAuth2 token (legacy approach)
```python
transport_creds = implementations.ssl_channel_credentials(open('roots.pem').read(), None, None)
def oauth2token_credentials(context, callback):
  try:
    credentials = oauth2client.client.GoogleCredentials.get_application_default()
    scoped_credentials = credentials.create_scoped([scope])
  except Exception as error:
    callback([], error)
    return
  callback([('authorization', 'Bearer %s' % scoped_credentials.get_access_token().access_token)], None)

auth_creds = implementations.metadata_plugin_credentials(oauth2token_credentials)
channel_creds = implementations.composite_channel_credentials(transport_creds, auth_creds)
channel = implementations.secure_channel('localhost', 50051, channel_creds)

stub = helloworld_pb2.beta_create_Greeter_stub(channel)
```

###Authenticating with Google (Java)

####Authenticate using OAuth2 token

The following code snippet shows how you can call the [Google Cloud PubSub API](https://cloud.google.com/pubsub/overview) using gRPC with a service account. The credentials are loaded from a key stored in a well-known location or by detecting that the application is running in an environment that can provide one automatically, e.g. Google Compute Engine. While this example is specific to Google and its services, similar patterns can be followed for other service providers.

```java
// Create a channel to the test service.
ManagedChannel channel = ManagedChannelBuilder.forTarget("pubsub.googleapis.com")
    .build();
// Get the default credentials from the environment
GoogleCredentials creds = GoogleCredentials.getApplicationDefault();
// Down-scope the credential to just the scopes required by the service
creds = creds.createScoped(Arrays.asList("https://www.googleapis.com/auth/pubsub"));
// Intercept the channel to bind the credential
ClientAuthInterceptor interceptor = new ClientAuthInterceptor(creds, someExecutor);
Channel channel = ClientInterceptors.intercept(channelImpl, interceptor);
// Create a stub using the channel that has the bound credential
PublisherGrpc.PublisherBlockingStub publisherStub = PublisherGrpc.newBlockingStub(channel);
publisherStub.publish(someMessage);
```



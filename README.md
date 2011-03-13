ApiAuth
=======

Logins and passwords are for humans. Communication between applications need to 
be protected through different means.

ApiAuth is a Ruby gem designed to be used both in your client and server
HTTP-based applications. It implements the same authentication methods (HMAC-SHA1) 
used by Amazon Web Services.

The gem will sign your requests on the client side and authenticate that 
signature on the server side. If your server resources are implemented as a 
Rails ActiveResource, it will integrate with that. It will even generate the 
secret keys necessary for your clients to sign their requests.

Since it operates entirely using HTTP headers, the server component does not 
have to be written in the same language as the clients.

How it works
------------

1. A canonical string is first created using your HTTP headers containing the 
content-type, content-MD5, request URI and the timestamp. If content-type or 
content-MD5 are not present, then a blank string is used in their place. If the 
timestamp isn't present, a valid HTTP date is automatically added to the 
request. The canonical string string is computed as follows:

    canonical_string = 'content-type,content-MD5,request URI,timestamp'

2. This string is then used to create the signature which is a Base64 encoded 
SHA1 HMAC, using the client's private secret key.

3. This signature is then added as the `Authorization` HTTP header in the form:

    Authorization = APIAuth 'client access id':'signature from step 2'
        
5. On the server side, the SHA1 HMAC is computed in the same way using the 
request headers and the client's secret key, which is known to only 
the client and the server but can be looked up on the server using the client's 
access id that was attached in the header. The access id can be any integer or 
string that uniquely identifies the client.


References
----------

* [Hash functions](http://en.wikipedia.org/wiki/Cryptographic_hash_function)
* [SHA-1 Hash function](http://en.wikipedia.org/wiki/SHA-1)
* [HMAC algorithm](http://en.wikipedia.org/wiki/HMAC)
* [RFC 2104 (HMAC)](http://tools.ietf.org/html/rfc2104)

Usage
-----

### Install ###

    [sudo] gem install api_auth
    
### Supported Request Objects ###

ApiAuth supports most request objects. Support for other request objects can be 
added as a request driver.

Here is the current list of supported request objects:

* Net::HTTP
* ActionController::Request
* Curb (Curl::Easy)
* RestClient
    
### ActiveResource ###

ApiAuth can transparently protect your ActiveResource communications with a 
single configuration line:

    class MyResource < ActiveResource::Base
      with_api_auth(access_id, secret_key)
    end
    
This will automatically sign all outgoing ActiveResource requests from your app.

### Server ###

ApiAuth provides some built in methods to help you generate API keys for your 
clients as well as verifying incoming API requests.

To generate a Base64 encoded API key for a client:

    ApiAuth.generate_secret_key
    
To validate whether or not a request is authentic:
    
    ApiAuth.authentic?(signed_request, secret_key)
    
If your server is a Rails app, the signed request will be the `request` object. 

In order to obtain the secret key for the client, you first need to look up the 
client's access_id. ApiAuth can pull that from the request headers for you:

    ApiAuth.access_id(signed_request)
    
Once you've looked up the client's record via the access id, you can then verify
whether or not the request is authentic. Typically, the access id for the client
will be their records primary key in the DB that stores the record or some other
public unique identifier for the client.

Here's a sample method that can be used in a `before_filter` in a Rails app:

    before_filter :api_authenticate

    def api_authenticate
      @current_account = Account.find_by_access_id(ApiAuth.access_id(request))
      return ApiAuth.authentic?(request, @current_account.secret_key) unless @current_account.nil?
      false
    end
        
Authors
-------

* [Mauricio Gomes](http://github.com/mgomes)

Copyright
---------

Copyright (c) 2011 Gemini SBS LLC. See LICENSE.txt for further details.
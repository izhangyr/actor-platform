# Authentication Key

In rev3 of protocol MTProto v2 introduces a better way of authentication of requests. In previous versions server tell authKey explictly and client apps need to pass it to every request for authentication. This is not a problem as TLS is required and everything is encrypted by default already.
But we think that can be a problem in some cases on the server side as this key flaws around the server and can occasionaly appear in server's logs. On the other side application might need to have some shared secret for performing various security related tasks, for example, sigining voice call requests.

Eventually, in rev4 of MTProto we will eliminate using of TLS for better speed and security.

# Changes in Transport's Package

Adding new ```signature``` field for signing each package. Depends on authId server will use signature or not.
```
Package {
  // unique identifier that is constant thru all application lifetime 
  authId: long
  // random identifier of current session
  sessionId: long
  // message
  message: Message
  // signature 
  signature: bytes
}
```

# New Requesting Authentication Key
Old method of AuthId creation will continue to work, but rev3 introduces new way of more secure way to get AuthId and AuthKey.

First app need to request starting of Auth Key creation:
```
RequestStartAuthKey {
  HEADER = 0xE0
}
```

Server will return truncated to 8 bytes of SHA-256 of available keys
```
ResponseStartAuthKey {
  HEADER = 0xE1
  availableKeys: longs
}
```

Client requests required key:
```
ResponseGetServerKey {
  HEADER = 0xE2
  keyId: long
}
```

Server return raw key data. Client MUST to check received key by comparing FULL hash that is hardcoded inside application. Again, DON'T compare truncated hashes - this is insecure. Client can skip downloading keys if it have built-in keys installed.
```
ResponseGetServerKey {
  HEADER = 0xE3
  keyId: long
  key: bytes
}
```
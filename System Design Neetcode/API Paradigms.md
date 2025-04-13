- what is API in the context of this course?
	- some programming interface that serves user request
- what are the common API paradigms?
	- REST, GraphQL, gRPC
- what is REST?
	- not a protocol
	- built on top of HTTP typically
	- set of  loose restrictions that we apply on HTTP API
	- there should be no state stored by the server itself, whereas it can persist state on to a persistent storage like a db
	- uses json format to communicate data
	- json very readable but performance can be better
- what is pagination?
	- looks like pagination information is sent as part of the URI/URL for a GET request as GET requests do not have a body
	- limit and offset are passed as query parameters as part of the URL
	- pagination eliminates the server to store state
- why do we even need REST, why is there so much fuss about storing state?
	- when the need for a server to store state is eliminated, horizontal scaling becomes possible
	- there will be no need that a user has to communicate to one server and other users will not be blocked in the mean time
- : in a url represents a url parameter to be followed
- question mark in a url represents a query parameter will be followed by it
- what are the problems with REST API?
	- over-fetching
		- when trying to fetch user details of comments, we might end up pull the entire details stored about that user which is not required to display a comment
		- can be dealt with server side change, but it is not scalable
	- endpoints can quickly scale
- what is graphQL?
	- not a protocol
	- built on top of HTTP
	- uses only POST requests
	- body usually consists of what resource we are looking for
	- graphQL addresses the over-fetching problem of REST by allowing to specify which fields do we want 
		- client gets what it needs
		- performance and nw costs saved
	- allows us to bundle multiple consecutive requests as one
		- to fetch comment, the main context is what video had that comment, so we need details about the video, poster, commenter etc..
		- all  this usually requires consecutive requests
		- but graph ql solves this problem by helping us bundle
	- does GrahQL allow for only one endpoing
	- with any graphql end point, we can do:
		- query
		- mutation: any time we update data
	- POST requests are not idempotent
		- so we cannot cache graph ql requests as we would be able to with REST
- in the video, he demonstrated from 18:29 how we can specifically ask for what we want
- unlike REST API, GraphQL has schema
	- schema for what? body of the request?
	- I guess he means schema for each resource which defines what can be requested
	- example: rocket is a resource and it's schema can look something like the following: 'id, name, version' and we have to stick to that
	- grpc built on top of http 2
	- cannot be used from within a browser
		- if we want to make a request to a grpc end point using a browser: we need a proxy
		- because grpc requires some fine grained control over http2 which the browsers do not provide
- common use case is for server to server comms
- faster and efficient
	- as instead of sending information as a string(json) like in rest, grpc sends info in protocol buffers(schema objects)
	- the data put in protocol buffers are serialized and sent
	-  which is lightweight than if we had sent it as string
- since grpc built on top of http2, we might not need other protocols for different use cases
	- ex: we can perform streaming
- http2 makes websockets obsolete
- recent development, less standardized and difficult dev, requires good tooling
- Grpc has error messages and not status codes, so handling might be new
```
## Example of protocol buffer schema
syntax = "proto3"
message SearchRequest { # message is a keyword used to define a schema
	string query = 1; # after equal to, the order is defined
	int32 page_number = 2;
	int32 result_per_page = 3;
}
## so when the binary rep is deserialized, it will be an object
## all this goes into a .proto file
```

```
.proto file
service Greeter {
	# endpoint with name SayHello, which will be passed in a request which has a schema
	rpc SayHello (HelloRequest) returns (HelloReply) {}
}

message HelloRequest {
	string name = 1;
}

message HelloReply {
	string message = 1;
}
```
- every rpc has a action associated with it
- it is as if we are calling functions within the same system
- 
- 53
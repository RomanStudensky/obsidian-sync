Representation state transfer. REST use HTTP as a base. 

For RESTful interfaces need to match requirements:
- modelling relationship between producer and consumer, where P. models resources available for C.
- stateless request from P. to C. non-cacheable for P. which make req.
- single interface 
- multilevel system that abstracts from complexity of the system behind the REST interface

The Richardson maturity model

Level 0 - HTTP/RPC. API builded with HTTP and single URI.
Level 1 - Resources. Example endpoint `/attendees/1` 
Level 2 - Verbs. Decide method of acton for communication
Level 3 - Utils for controlling hypermedia. 
---
title: REST and JSON
draft: true
---

6.0 Automation and Programmability  
6.5 Describe characteristics of REST-based APIs (CRUD, HTTP verbs, and data encoding)  
6.7 Interpret JSON encoded data

The configuration of network devices or report facts about the state of the network.software analyzes data in the form of variables,makes decisions based on that analysis, and then may take action to change the

If communicate the variables used by a program: 
the variable names, their values, and the data structures of those variables.REST provides one standard method of how two automation programs should communicate over a network, JSON then defines how to

REST-Based APIs  
Applications use application programming interfaces (APIs) to communicate.  
one program can learn the variables and data structures used by another program, making logic choices based on those values, changing the values of those variables, creating new variables, and deleting variables  
some applications create an API, with many other applications using (consuming) the API.  
REST-Based (RESTful) APIs  
REST APIs include the six attributes  
▪ Client/server architecture  
▪ Stateless operation  
▪ Clear statement of cacheable/uncacheable  
▪ Uniform interface  
▪ Layered  
▪ Code-on-demand  
Client/Server Architecture  
REST API call, which generates a message sent to the REST server.

### 18 REST and JSON

Tuesday, November 2, 2021 2:46 PM

```
the use of HTTP is not a requirement for an API to be considered RESTful.
Stateless Operation
each API request and reply does not use any other past history considered when processing the request.
For comparison, the TCP protocol uses a stateful approach, whereas UDP uses stateless operation
Cacheable (or Not)
improve performance by retrieving resources less often (cacheable)
cacheable resources are marked with a timeframe so that the client knows when to ask for a new copy of the resource again.
```

List and Dictionary Variables  
a data structure defines a related set of variables and values  
Python uses list variables so that one variable name is assigned a value that is a list of values rather than a single value

```
the dictionary data structure lists paired items as well: keys (like terms) and values (like definitions)
```

a single variable’s value could be a list, with each list element being a dictionary  
REST APIs and HTTP  
APIs exist to allow two programs to exchange data  
Many APIs need to be available to programs that run on other computers, so the API must define the type of networking protocosupported by the API—and many REST-based APIs use the HTTP protocol. ls  
HTTP uses the same principles as REST: it operates with a client/server model; it uses a stateless operational model; and it headers that clearly mark objects as cacheable or not cacheable. It also includes verbs—words that dictate the desired action for a includes  
pair HTTP Request and Reply—which matches how applications like to work.  
Software CRUD Actions and HTTP Verbs  
acronym—CRUD—for the four primary actions performed by an application. Those actions are:  
Create:as kept at the serverAllows the client tocreate some new instances of variables and data structures at the server and initialize their values  
Read:structures, and values at the clientAllows the client to retrieve (read) the current value of variables that exist at the server, storing a copy of the variables,  
Update:Allows the client to change (update) the value of variables that exist at the server  
Delete:Allows the client todelete from the server different instances of data variables

```
HTTP uses verbs that mirror CRUD actions.HTTP defines the concept of an HTTP request and reply
Each request/reply lists an action verb in the HTTP request header, which defines the HTTP action. The HTTP messages also incURI, which identifies the resource being manipulated for this request. lude a
the HTTP message is carried in IP and TCP, with headers and data, as represented
```

Using URIs with HTTP to Specify the Resource  
REST uses URIs to identify what resource the HTTP request acts on. For REST APIs, the resource can be any one of the many resdefined by the API. Each resource contains a set of related variables, defined by the API and identified by a URI. ources

a URI specific to the Cisco DNA Center northbound REST API as an example of some of the components of the URI.

```
HTTPS: The letters before the :// identify the protocol usedencryption). —in this case, HTTP Secure (which uses HTTP with SSL
Hostname or IP Address: This value sits between the // and first /, and identifies the host; if using a hostname, the REST client must perform name resolution to learn the IP address of the REST server.
Path (Resource): This value sits after the first / and finishes either at the end of the URI or before any additional fields (like a parameter query field). HTTP calls this field the path, but for use with REST, the field uniquely identifies the
resource as defined by the API.
Go to see more examples of the resources defined by the Cisco DNA Center API.https://developer.cisco.comand search for “Cisco DNA Center API documentation.” Continue to search for yourself to
```

```
Many of the HTTP request messages need to pass information to the REST server beyond the API
REST APIs use HTTP header fields to encode much of the authentication information for REST calls. Additionally, parameters related to a REST call can be passed as parameters as part of the URI itself.
```

```
the major components of the URIs commonly used with a REST API, with the resource and parameter parts of the URI identifying specifically what the API should supply to the REST client.
```

Example of REST API Call to DNA Center  
sample API calls using a software application called an API development environment tool.  
The examples in this section use an app named Postman. Postman can be downloaded for free (shown in this section. Note that Cisco DevNet makes extensive use of Postman in its many labs and exampleswww.postman.co) and used as

The text follows a data modeling format called JavaScript Object Notation (JSON),

Data Serialization and JSON  
Data serialization languages give us a way to represent variables with text rather than in the internal representation used bparticular programming language. y any  
Each data serialization language enables API servers to return data so that the API client can replicate the same variable nawell as data structures as found on the API server mes as  
JavaScript Object Notation (JSON).  
The Need for a Data Model with APIs  
how to move variables in a program on a server to a client program. First, Figure 18example as a way to identify some of the challenges with copying variable values from one device to another.-10 and surrounding text show aThen Figure 18nonworking -11 and  
its related text show how to use a data serialization language to solve the problems shown around Figure 18-10.

```
The process shown in Figure 18variables in the same ways. First, programs written in different languages use different conventions to store their -10 does not work (and is not attempted) because the REST client programs may not store
variables internally because there is in the same language but with different versions of that language may not store all their variables with the same internal no standard for internal variable storage across languages.In fact, programs written
conventions.
applications need a standard method to represent variables for transmission and storage of those variables outside the prograData serialization languages provide that function. m.
The API converts the internal representation to a data model representing those variables (with JSON shown in the figure).
```

```
while data serialization languages like JSON enable applications to exchange variables over a network, applications can also data in JSON format. store
```

Data Serialization Languages  
The terms data serialization language and data modeling language should be considered equivalent  
different data serialization languages  
JSON  
XML

XML  
XML being a little more challenging to read for the average person  
XML uses beginning and ending tags for each variable,

YAML  
YAML does not attempt to define markup details (while XML does)  
YAML focuses on the data model (structure) details.  
strives to be clean and simple  
the easiest to read for anyone new to data models.  
Ansible makes extensive use of YAML files  
YAML denotes variables in double curly brackets: {{ }}.)

Summary of Data Serialization

qbwn  
Interpreting JSON

Interpreting JSON Key:Value Pairs  
rules about key:value pairs in JSON  
Key:Value Pair:Each and every colon identifies one key:value pair,with thekey before the colon and the value after the colon.  
Key:Text, inside double quotes, before the colon, used as the name that references a value.  
Value: The item after the colon that represents the value of the key, which can be  
Text: Listed in double quotes.  
Numeric: Listed without quotes.  
Array:A special value (more details later).  
Object:A special value (more details later)  
Multiple Pairs:pair) When listing multiple key:value pairs, separate the pairs with a comma at the end of each pair (except the last

The first two key:value pairs end with a comma, meaning that another key:value pair should follow.begin and end the JSON data denote a single JSON object(one pair of curly brackets, so one object). The JSON files, and JSON data curly brackets that  
exchanged over an API, exist first as a JSON object, with an opening (left) and closing (right) curly bracket as shown.  
Interpreting JSON Objects and Arrays  
To communicate data structures beyond a key:value pair with a simple value, JSON uses JSON objects and JSON arrays.be somewhat flexible, but in most uses, they act like a dictionary. Arrays list a series of values. Objects can  
For general conversation, many people refer to the JSON structures as dictionaries and lists rather than as objects and arrays.  
{ } matching right curly bracket.-Object: A series of key:value pairs enclosed in a matched pair of curly brackets,with an opening left curly bracket and its  
[ ] and its matching right square bracket.-Array: A series of values (not key:value pairs) enclosed in a matched pair of square brackets, with an opening left square bracket

Key:value pairs inside objects: All key:value pairs inside an object conform to the earlier rules for key:value pairs.  
Values inside arraysaround numbers). :All values conform to the earlier rules for formatting values (for example, double quotes around text, no quotes

JSON arrays can be used as a value in any key:value pair.

```
It has a matched pair of curly brackets to begin and end the text, encapsulating one object.colons, so there are two key:value pairs inside the object. That object contains two
```

JSON can nest objects and arrays; that is, JSON puts one object or array inside another.  
Minified and Beautified JSON  
When stored in a file or sent in a network, JSON does not use whitespace.  
You might see the pretty version literally called pretty, or beautified, or spaced, while the version with no extra whitespaccalled minified or raw. e might be


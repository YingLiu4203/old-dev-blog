---
layout: post
title: Guide to boto Amazon MWS Python package 
---

## Overview
[boto](https://github.com/boto/boto) is a Python package for Amazon 
web service APIs. It makes it easy to use Amazon marketplace web 
service (MWS) APIs by providing supporting functions such as connection pool, 
auto-retry, response parsing and iterative calls for extra data. 
However, there is no document and the Python code is not well commented. 
Below is an attempt to summarize boto implementation and usage. 

## API Decorators
boto organizes method calls into sections corresponding to Amazon
MWS sections: Feeds, Reports, Orders, Products and Sellers etc. 
In Amazon MWS, each section has independent version, access key (
merchant or seller id, same value so far) and URL api path. 
Sections are defined in api_version_path variable at the top 
of connection.py.

For an API inside a section, there are a set of metadata that are 
defined using decorators. A decorator returns a wrapper function 
that performs validation, sets call parameters and finally 
call the wrapped function. 

A wrapper function has the following attributes that is
set by a decorator.

* action: usually a capitalized wrapped function name
* response: a response 
* section: the section of the api. 
* quota: api throttling quota
* restore: api throttling restore time 
* version: api version 
A decorator sets these attributes in its wrapper function. 

Following are decorators: 

* api_action: This should be the first (innermost) decorator that
specifies the section, version, quota and restore parameters.
Then it create a request argument from a dict; a response 
 argument from ResponseFactory. 
It also put a pair of action and wrapped function name into a 
global variable named `api_call_map`. 
* structured_lists:  is a list of data such as Marketplace Id. Because 
it is a list, a single marketplace id should be passed as `[marketplaceid]`.
* http_body: set HTTP headers and request type used by MWS. It sets two 
keys, 'body' and 'headers' in `**kw`.  
* boolean_argument: add the lower case boolean value to `**kw`.
* requires: this checks that exactly one required parameter is set 
in `**kw`. If not,it raises a KeyError exception.
* exclusive: requires only one or zero parameter from a list. 
* dependent: it checks that dependent parameters are defined in `**kw`. 
* requires_some_of: it requires at least one in a list of fields is defined.
* structured_objects: it breaks objects into individual `**kw` entries.
 
By looking at the decorators associated with an API, one can tell 
the calling conventions and required parameters. For example, 
`submit_feed()` has the following definition:

```python
@requires(['FeedType'])
@boolean_arguments('PurgeAndReplace')
@http_body('FeedContent')
@structured_lists('MarketplaceIdList.Id')
@api_action('Feeds', 15, 120)
def submit_feed(self, request, response, headers=None, body='', **kw):
    ...
```

From the innermost toward outermost, we can tell that the `submit_feed` 
call is in the 'Feeds' section, the quota is 15 and the restoration 
time is 120 seconds. It uses a list of marketplace Ids. The http 
request body has a type of 'FeedContent'. The 'PurgeAndReplace' 
boolean value will be converted to lower case in request. 
The 'FeedType' has to be defined as a keyword argument. 

## Sending Request
In all API calls, boto first creates an HTTPRequest object that contains
all request data. Then it sends the request with a default retry time 
of 6. It uses a connection pool to get and cache connections. 
A connection is identified by three parameters: host, port and is_secure flag.

After adding the required message signature and other authentication data, 
It uses Python httplib.HTTPConnection to send request. If response status
is one of 500, 502, 503, and 504, it tries again. If response 
status is smaller than 300 or bigger than or equals to 400 or not set 
location in header, it returns the response. For all other cases, the
response is a redirecting, it tries sending with new location. 
If an exception happens, it sleeps with an exponential back-off time and 
tries again. After retrying for the specified number of times 
without success, it raise an exception. 

If there is a response returned, it raises BotoServerError with a
response status, a response reason and a response body. Otherwise, 
it raises the exception raised in the request.

When the call completes, if the response status is not OK (200), 
boto raises a customized response exception.

## Parsing Response
If a response content type is not 'text/xml', boto returns the response
body data. Otherwise, boto uses a SAX XML handler to parse the data. 

### Response Result
boto creates an instance of Response to represent XML response. The object
maps each an XML element to an object attribute. An XML element's 
attributes are save to the dictionary of the attribute. 
Except leaf node, boto defines response XML tags by as 
attributes in an instance of `ResponseElement`. 
`ComplexType` is used for XML tag attributes.
 
The attribute value of a `Response` is an instance of an `Element` 
for single elment or `ElementList` for a list of element. 
 
Most request creates a `Response` object that has two attributes:

* XXXResult: the actual response data for XXX API
* ResponseMetadata: the meta data for the request. It has an
`RequestId` attribute.
 
The `XXXResult` attribute has a value that has attributes (and 
attributes of an attribute object) that map the XML response
element structure. For example, the ListOrders API returns an 
XML response with the following structure: 

```xml
<ListOrdersResponse xmlns="https://mws.amazonservices.com/Orders/2013-09-01">
  <ListOrdersResult>
    <Orders>
      <Order>
        <ShipmentServiceLevelCategory>Standard</ShipmentServiceLevelCategory>
        <OrderTotal>
          <Amount>126.74</Amount>
          <CurrencyCode>USD</CurrencyCode>
        </OrderTotal>
        <ShipServiceLevel>Std Cont US Street Addr</ShipServiceLevel>
        ...
      </Order>
      <Order>
        <ShipmentServiceLevelCategory>Standard</ShipmentServiceLevelCategory>
        <OrderTotal>
          <Amount>69.00</Amount>
          <CurrencyCode>USD</CurrencyCode>
        </OrderTotal>
        <ShipServiceLevel>Std Cont US Street Addr</ShipServiceLevel>
        <LatestShipDate>2014-09-04T06:59:59Z</LatestShipDate>
        <MarketplaceId>ATVPDKIKX0DER</MarketplaceId>
        <SalesChannel>Amazon.com</SalesChannel>
        ...
      </Order>
      ...
    <Order>
  <ListOrdersResult>
  <ResponseMetadata>
    <RequestId>a6ac7b47-ddd6-476a-9f0e-ad7e8c9bbf87</RequestId>
  </ResponseMetadata>
</ListOrdersResponse>
```

The response can be accessed using the following code:

```python
# all orders
orders = response.ListOrdersResult.Orders.Order

#the first order amount
firstOrder = orders[0].OrderTotal.Amount

# the request Id
requestId = response.ResponseMetadata.RequestId
```

In the above code, response is the return value of an API call. 
All values are unicode string values.
 
### Response Factory
The following is an incomplete code analysis of the 
implement details of boto response parsing. Ignore it 
if you are not interested in those details. 

When an MWSConnection object is created, a default response factory 
object is created by the code 
`boto.mws.response.ResponseFactory(boto.wms.response)`. 
The response module is passed as a scope of the factory. When a request
is completed, the response factory object is called with an action name 
and a connection. The factory object then search a module attribute
(using `getattr` and `__getitem__`) that has a pattern of 
'${action_name}Response'. If no such attribute is defined in 
the response module, it creates an instance of `DynamicElement` 
that is a subclass of `Response`. This response's  
_name and __name__ attributes are set to '${action_name}Response'.
If the response class doesn't define a `${action_name}Result` 
attribute, it sets the response's  `${action_name}Result` attribute 
to an instance of `Element` that is initialized with an
instance of the `ResponseElement` class. 
 
The `Response` is a subclass of `ResponseElement` that is a subclass
of the `dict` class. The `ResponseElement` defines `startElement` and
`endElement` that are used to parse an XML file. 
The `Response` class defines a `ResponseMetadata`
attribute that is an `Element` object. The response factory
creates a `Response` object with two attributes:

* ResponseMetadata = Element()
* ${action_name}Result = Element(${action_name}Result)

Additionally, the `ResponseElement` constructor calls the `setup`
method of attributes that are a subclass of `DeclarativeType`. 
In `setup`, it creates a clone of the object and set parent 
object's `_name` to the clone. 

The `Element` is a subclass of a `DeclarativeType`. 
The `DeclarativeType` has a `_value` attribute 
and a `_hint` attribute. If `_hint` is not given in an object creation,
it creates a an object of `JITResponse` (another
subclass of `ResponseElement`) as `_hint` attribute value and set 
items in `**kw` as attributes of this newly created `_hint` object.   
 
`ResponseResultList` is a subclass of `Response` that defines one 
class attribute `_ResultClass = ResponseElement` and one 
instance attribute `${action_name}Result = ElementList(self._ResultClass)`.

Some `${action_name}Response` classes are inherited from 
`ResponseResultList` that is used to parse request response. Most 
APIs use a dynamically created `DynamicElement`, a subclass of the 
`Response` class to parse a response.

### Parsing process

boto defines an XmlHandler class, a subclass of xml.sax.ContentHandler, 
as a base class to process XML response. It takes a root node and a 
connection as its constructor parameters. It stores a list of nodes. A 
node is a tuple of ('name', class). In its `startElement` method,
it calls the last node's `startElement` method. If the method creates
a new node, it adds the new node to its node list. In its `endElement` 
method, it calls the last node's `endElement` method and optionally
`endNode` method. If a node's end element is met, the node is popped. 
Therefore, the XMLHandler just call a node's corresponding methods 
with tag name, attrs, connection and content parameters. 

In `Response.startElement` method, it first checks if the XML
tag name equals its `_name` attribute. If true, it updates its 
attributes with the XML tag attributes because the matched name means
that the attributes belong to this response element. If false, this is a 
sub-element, it calls `ResponseElement.startElement` method.
 
In `ResponseElement.startElement` method, it searches if it has an
attribute that has the same name. If true, it call the attribute's 
`start` method. The method set the `_value` of the attribute to 
a new instance of its `_hint` class. If false, it checks if there
is an attributes, if yes, set an attribute with `ComplexType` object
using the attributes. If, no, do nothing. 

In `ResponseElement.startElement` method, if the name is a 
closing tag for the current name, update attributes and call
`teardown` method for the current node. If the name is a
ComplexType attribute name, set its `_value` to tag value. 
Otherwise, this is leaf node with simple value, set an 
attribute with the name and value. 




 
 



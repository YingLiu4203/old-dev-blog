---
layout: post
title: An Example Using boto Amazon MWS Package    
---

## Overview
This article is a follow up of the [Guide to boto Amazon MWS Python
Package.](http://www.mindissoftware.com/2014/10/19/boto-amazon-mws-interface-guide/)
Here we give a complete example of submitting a product feed.

This implementation uses boto 2.33 to call Amazon MWS apis. To create 
a product feed XML, we use the [Jinja2 template engine](http://jinja.pocoo.org/) 
because it has rich features and is widely used by Python developers 
as Web page or Email templates. 

## XML Template of Product Feed
Amazon recommends XML, not a flat file, as the preferred data format to 
send MWS request. According to [a group discussion message]
(https://groups.google.com/forum/#!topic/boto-users/zPjQaQcb1Wo) from 
a boto MWS developer (Andy Davidoff), boto's MWS module does not support 
generating XML from a Python object for two reasons: 

> First, redistribution of the XSD files is (supposedly) not permitted. 
> Second, there are many defects in these files. 

He suggested using an XML template engine and recommended [Genshi]
(http://genshi.edgewall.org/). Because many Python web developers are 
already familiar with Jinja2, we use it as our XML template engine.  
Using Jianja2 template engine is straight forward because its syntax is 
similar to Python code. To make the example simple, the below
Jinja2 template is used to change titles of one or more products.

**Please remove the backslashes in the following code. Without it, I couldn't show Jinja2 code using GitHub markdown**
 
```text
<?xml version="1.0" encoding="utf-8" ?>
<AmazonEnvelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="amzn-envelope.xsd">
    <Header>
        <DocumentVersion>1.01</DocumentVersion>
        <MerchantIdentifier>{{ MerchantId }}</MerchantIdentifier>
    </Header>

    <MessageType>Product</MessageType>
    <PurgeAndReplace>false</PurgeAndReplace>

    \{\% for message in FeedMessages \%\}
    <Message>
        <MessageID>\{\{ loop.index \}\}</MessageID>
        <OperationType>PartialUpdate</OperationType>
        <Product>
            <SKU>\{\{ message.SKU \}\}</SKU>
            <DescriptionData>
                <Title>\{\{ message.Title \}\}</Title>
            </DescriptionData>
        </Product>
    </Message>
    \{\% endfor \%\}

</AmazonEnvelope>
```

The Amazon MWS XML schema is defined in the [Selling on Amazon Guide to XML pdf file]
(https://images-na.ssl-images-amazon.com/images/G/01/rainier/help/XML_Documentation_Intl.pdf)
and links embedded in the file. There are some subtle things that cause
Amazon to reject the a submitted XML feed. For example: 

* There is nothing before the the XML declaration.
* There is a space after the "utf-8" in the XML declaration

## boto MWS API Call
The boto MWS package doesn't have good document and even good code comments.
We had to read the code to figure out the calling conventions. The use is 
not hard once we know where to look in the source code.

The API to submit a product feed are defined in boto.mws.connection module. 
An MWSConnection usually needs three keyword arguments: Merchant, 
aws_access_key_id, and aws_secret_access_key. A connection object is created
using the following code: 

```python
MarketPlaceID = 'Your_Market_Place_ID'
MerchantID = 'Your_Merchant_ID'
AccessKeyID = 'Your_AccessKey_ID'
SecretKey = 'Your_SecreteKey_ID'

conn = connection.MWSConnection(
    aws_access_key_id=AccessKeyID,
    aws_secret_access_key=SecretKey,
    Merchant=MerchantID)
```

To change a product tile, we need to make three calls: the `submit_feed`
call to submit the feed, the `get_feed_submission_list` call to find 
out the call processing status, the `get_feed_submission_result` call to
find out the call processing result. Each call has different requirements. 
For example, the `submit_feed` has the following decorators:

```python
@requires(['FeedType'])
@boolean_arguments('PurgeAndReplace')
@http_body('FeedContent')
@structured_lists('MarketplaceIdList.Id')
@api_action('Feeds', 15, 120)
```

The above decorators specify the following call conventions: 

* It requires a 'FeedType' keyword argument
* It has a boolean  'PurgeAndReplace' keyword argument 
* The HTTP request body is passed as the 'FeedContent' keyword argument
* It is in 'Feed' section with a quota of 15 and restoration time of 120 
seconds.

It seems there is no need to pass any request or header arguments
to make a call. All needed is to provide the required keyword arguments.
The following is the code to call `submit_feed`: 

```python
feed = conn.submit_feed(
    FeedType='_POST_PRODUCT_DATA_',
    PurgeAndReplace=False,
    MarketplaceIdList=[MarketPlaceID],
    content_type='text/xml',
    FeedContent=feed_content
)
```

## Call Response
Though boto doesn't generate request XML content, it parses an XML response
into a Python object. A response XML message usually has two parts. For
the `submit_feed` call, they are: <SubmitFeedResult> and <ResponseMetaData>.
The generated Python object has attributes that have one-to-one mapping 
to the XML response message. The attribute names can be found in 
Amazon MWS API document. For example, [the Amazon SubmitFeed document]
(http://docs.developer.amazonservices.com/en_US/feeds/Feeds_SubmitFeed.html)
describes the XML structure for the boto `submit_feed` method response. 
All attributes can be easily accessed as Python object attribute. All
attribute values are of 'UTF-8' string type. 

## Complete Source Code

There are three files in this example. There are arranged in the 
following structure:

```
boto_test\
    __init__.py
    connnection_test.py
    
    templates\
        product_feed_template.xml        

```

The __init__.py is an empty file used to define a Python package. 
'product_feed_template.xml' is the template described above. 
The following is the source code of 'connection_test.py'.

```python
# -*- coding: utf-8 -*-

from boto.mws import connection
import time
from jinja2 import Environment, PackageLoader

# Amazon US MWS ID
MarketPlaceID = 'ATVPDKIKX0DER'
MerchantID = 'MID'
AccessKeyID = 'AKID'
SecretKey = 'SID'

env = Environment(loader=PackageLoader('boto_test', 'templates'),
                  trim_blocks=True,
                  lstrip_blocks=True)
template = env.get_template('product_feed_template.xml')
class Message(object):
    def __init__(self, sku, title):
        self.SKU = sku
        self.Title = title

feed_messages = [
    Message('SDK1', 'Title1'),
    Message('SDK2', 'Title2'),
]
namespace = dict(MerchantId=MerchantID, FeedMessages=feed_messages)
feed_content = template.render(namespace).encode('utf-8')

conn = connection.MWSConnection(
    aws_access_key_id=AccessKeyID,
    aws_secret_access_key=SecretKey,
    Merchant=MerchantID)

feed = conn.submit_feed(
    FeedType='_POST_PRODUCT_DATA_',
    PurgeAndReplace=False,
    MarketplaceIdList=[MarketPlaceID],
    content_type='text/xml',
    FeedContent=feed_content
)

feed_info = feed.SubmitFeedResult.FeedSubmissionInfo
print 'Submitted product feed: ' + str(feed_info)

while True:
    submission_list = conn.get_feed_submission_list(
        FeedSubmissionIdList=[feed_info.FeedSubmissionId]
    )
    info =  submission_list.GetFeedSubmissionListResult.FeedSubmissionInfo[0]
    id = info.FeedSubmissionId
    status = info.FeedProcessingStatus
    print 'Submission Id: {}. Current status: {}'.format(id, status)

    if (status in ('_SUBMITTED_', '_IN_PROGRESS_', '_UNCONFIRMED_')):
        print 'Sleeping and check again....'
        time.sleep(60)
    elif (status == '_DONE_'):
        feedResult = conn.get_feed_submission_result(FeedSubmissionId=id)
        print feedResult
        break
    else:
        print "Submission processing error. Quit."
        break
```
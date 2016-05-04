---
layout: post
title: Summary of Amazon Marketplace Web Service 
---

# Overview

Here I want to summarize Amazon marketplace web service (MWS or AMWS) that 
can be used for e-commerce data integration. It is based on Amazon 
[online documents][1]. The [developer guide][2] describes the 
basic functions and features of Amazon MWS. 

## Features
Amazon MWS provides the following features:
* Product and Inventory management: you can add/edit products, 
update inventory, price and other product and inventory management tasks. 
* Order management: you can download Amazon order information and payment data. 
You can update order status and shipping information.
* Reports management: you can query and download different reports.  
* Fulfillment by Amazon (FBA) management: you can create inbound shipments,
check shipment status, submit fulfillment orders and manage outbound shipments.

## Authentication
After registering as a developer, you receive three credentials 
to make API calls:

* A Developer Account Identifier that is a 12-digit identifier.
* An Access Key ID that is a 20-character, alphanumeric identifier.
* A Secret Key that is a 40-character identifier.

The access key ID is not a secret. It is used as an Id of a request. 
The secrete key is used to calculate a digital signature from one's 
access key ID. Amazon MWS uses this signature to authenticate a 
seller account.

## Building Robust AMWS apps
Because AMWS AIPs are evolving, it is important to handle response data
gracefully to build a robust AMWS app. 

* Don't break for new element and element values in reports or responses.
* Log unrecognized elements or values to find changes.
* Expect response elements in any order.

## Throttling
AMWS has a limits on how often you can submit API requests. You
should know the limits, check the throttling error, and have a back-off 
re-try implementation.  

A **request quota** represents the number of requests that on can 
submit at one time without throttling. It decreases with each request
and increases at a **restore rate**. The restore rate is the rate
at which the request quota increases over time, up to the maximum
request quota. 

## Using NextToken
Some AMWS APIs have a page size that only returns a limited number of elements. 
You should check the **HasNext** element to find if there are 
more elements. If the HasNext response element returns true, 
use a **NextToken** in response to call a **ByNextToken** operation.
  
Some operations returns a **MoreResultsAvailable** element.
This element should be checked regularly and if it is true,
call the ByNextToken operation with the NextToken to get data till the 
MoreResultsAvailable is false. Then start checking it again. 

Some operations do not return HasNext or MoreResultsAvailable, 
use the existence of the NextToken to find if there are more
data available. 

If a "ByNextToken" operation fails with a "NextTokenCorrupted" 
error, call the original operation. 

## Response
AMWS returns an XML file that contains the results of a request.
For a successful request, it returns a result and a request id.
For an unsuccessful request, it returns an "ErrorResponse"
element that has a list of errors and a request id. 

It is suggested that one should log the request timestamp
and the request id in case an error happens and these
data are used by Amazon support services.

# Feed API and XML Schema
## Feed API
The [MWS Feeds API][3] allows you to submit a feed, cancel a feed and 
check submit status and errors. It supports the following operations:

* [SubmitFeed][4]:  Upload a feed 
* [GetFeedSubmiisionList][5]:  Get a list of feed submissions in the previous 90 days 
* [GetFeedSubmissionListByNextToken][6]: Get a list of feed submissions 
using the **NextToken** parameter 
* [GetFeedSubmissionCount][7]: Get a count of the feeds submitted in the previous 90 days 
* [CancelFeedSubmissions][8]: Cancel one or more feed submissions 
* [GetFeedSubmissionResult][9]: Get the feed processing report and the content MD5 header 

## XML Schemas
MWS defines a core schema and a feed type schema. 
The core schema includes an envelope schema that includes
a head schema and a feed-specific schema. 

All schemas can be downloaded by following the [path of 
`Seller Center help, XML & Data Exchange, Reference, XSDs`][17]. 

### The Envelope Schema
The envelope is used to wrap all other data with message-level
protocol data. The envelope consists of a header and one
or more messages. Each message contains a specific data object
and must be of the same type in the same envelope.  

The envelope schema defines the following elements in the 
root element `<AmazonEnvelope>`:

1. A `<Header>` element
2. A `<MessageType>` element. Can be a values of `FulfillmentCenter, 
Inventory, Listings, OrderAcknowledgement, OrderAdjustment, 
OrderFulfillment, Override, Price, Processing Report, Product,
ProductImage, Relationship, SettlementReport`. 
3. A `<MarketplaceName>` element used only in Override feed
4. A `<PurgeAndReplace>` element. It is not recommended 
because purging existing products loses all history and 
listing positions in Amazon.
5. A `<EffectiveDate>` element used by inventory feed. 
6. Multiple `<Message>` elements. 

Below are elements defined inside the `<Message>` element:
1. A `<MessageID>` element: A unique number inside within the envelop. 
It is used for reconciliation of errors and warnings. It is a 
number that can have 1 to 18 digits. 
2. An `<OperationType>` element: Update, Delete, or PartialUpdate. 
PartialUpdate is only used for product feed.
3. A feed-specific data element that matches the 
`<MessageType>` element value. 

### The Header Schema
This schema only has two elements as shown below: 
 
```xml
<Header>
    <DocumentVersion>1.01</DocumentVersion>
    <MerchantIdentifier>MerchantId</MerchantIdentifier>
</Header>
```

### The Base Schema
The base XSD defines basic data types and values used by other XML
schemas. Basic data types include address, phone number, SKU, 
standard product ID, SKU, currency, time, etc. 

# Product Feeds API
There are six product feeds in AMWS:
1. `Product` feed: product catalog information that allows 
Amazon to assign an ASIN (Amazon Standard Identification Number) for each
SKU. 
2. `Inventory` feed: current stock level, restock date and fulfillment latency.
3. `Price` feed: current price, standard price or sales price
4. `ProductImage` feed: URLs for product images
5. `Relationship` feed: two product relationships: variation and 
accessory. 
6. `Override` feed: setting SKU-level shipping data

## Feed Submission Workflow
The feed submission workflow is depicted in the following diagram. 
![Amazon MWS feed submission workflow]
(http://docs.developer.amazonservices.com/en_US/feeds/Feed_flowchart.png)

AMWS uses a batch processing model that has the following steps: 

1. Submit a feed and get a "FeedSubmissionId". 
2. Check processing status using "GetFeedSubmissionList". 
3. When a feed processing is completed, the "FeedProcessingStatus" element
has a value of `_DONE_`. Use the "GetFeedSubmissionResult" operation to
receive a processing report that describes which records are successful
and which records are wrong. 
4. For unsuccessful records, fix errors and retry. 

## XML Schemas
### `Product` Schema
The product feed is the first step in setting up products. Subsequent feeds
are dependent upon the success of this feed. The `Product` element has
the following elements. If not explicitly marked as required, an 
element is optional. 

* `SKU`: Required. A `SKUType` value is a string with up to 40 chars. 
* `StandardProductID`: 
    * `Type`: a value from `ISBN, UPC, EAN, ASIN, GTIN, GCID, PZN`
    * `Value`: a string with 8 to 16 chars
* `GtinExemptionReason`: a value from `bundle, part`.
* `RelatedProductID`: used for a product bundle or replacement part. 
    * `Type`: a value from `UPC, EAN, GTIN`
    * `Value`: a string with 8 to 16 chars
* `ProductTaxCode`: an Amazon standard code for US-only product tax. A value
from `A_GEN_TAX, A_GEN_NOTAX` for most products. Books have special codes.  
* `LaunchDate`: when the product appears in Amazon site
* `DiscontinueDate`
* `ReleaseDate`: release for sale date
* `ExternalProductUrl`: ??
* `OffAmazonChannel`: ??. a value of `advertise, exclude`
* `OnAmazonChannel`: ??. a value of `sell, advertise, exclude`
* `Condition`: has two elements: <ConditionType> is a value of 
`New, UsedLikeNew, UsedVeryGood, UsedGood, 
UsedAcceptable, CollectibleLikeNew, CollectibleVeryGood, 
CollectibleGood, CollectibleAcceptable, Refurbished, Club`. 
<ConditionNote> is a 2000-char string note. 
* `Rebate`: a rebate with start date, end date, message and name. 
* `ItemPackageQuantity`: number of units in a package.
* `NumberOfItems`: number of discrete items in the product.
* `DescriptionData`: **product description defined below** 
* `PromoTag`: type of promotion, start and end date.
* `DiscoveryData`: browse priority, exclusion
* `ProductData`: **category-specific product data, defined below**
* `ShippedByFreight`: ?? a boolean value
* `EnhancedImageURL`: ?? up to two urls
* `Amazon-Vendor-Only`: ??
* `Amazon-Only`: ??
* `RegisteredParameter`: ?? a value from `PrivateLabel, Specialized,
NonConsumer, PreConfigured`. 

The `DescriptionData` element has the following elements: 

* `Title`: Required. a `LongStringNotNull` value (1-500)
* `Brand`: a `HundredString` value (0-100)
* `Designer`: a `StringNotNull` value (1-50)
* `Description`: `xsd:normalizedString` max length 2000
* 0 to 5 `BulletPoint`: a `LongStringNotNull` value describing features.
* `ItemDimensions`: defined in `Dimensions` Type that has four elements
    * `Length`: type `LengthDimension` adds an attribute to `Dimension` that
    is a type derived from `xsd:decimal`  with total 12 digits and
    2 fraction digits
        * Attribute `unitOfMeasure`:  a `LengthUnitOfMeasure` type value from 
         `MM, CM, M, IN, FT, inches, feet, meters, decimeters, centimeters,
         millimeters, micrometers, nanometers, picometers`.
    * `Width`: a `LengthDimension` value
    * `Height`: a `LengthDimension` value
    * `Weight`: type `WeightDimension` adds an attribute 
        * Attribute `unitOfMeasure`:  a `WeightUnitOfMeasure` type value from 
         `GR, KG< OZ, LB, MG`.
* `PackageDimensions`: dimensions of the package. A `Dimensions` type value
* `PackageWeight`: a `PositiveWeightDimension` value  
* `ShippingWeight`: a `PositiveWeightDimension` value for shipping package
* `MerchantCatalogNumber`: a `FortyStringNotNull` if it is different from `SKU`
* `MSRP`: A `BaseCurrencyAmount` value with `currency` attribute.  
A `BaseCurrencyAmount` value is a number with total 20 digits, 2 are 
fraction digits. A `currency` attribute value is from a list of 
`USD, GBP, EUR, JPY, CAD, CNY, INR`. 
* `MSRPWithTax`: taxed amount of type `BaseCurrencyAmount`
* `MaxOrderQuantity`: a positive integer
* `SerialNumberRequired`: a boolean value indicating if a serial number is required
* `Prop65`: a boolean value for California Prop 65 regulations
* 0 to 4 `CPSIAWarning`: a value from `choking_hazard_balloon, choking_hazard_contains_a_marble
choking_hazard_contains_small_ball, choking_hazard_is_a_marble, choking_hazard_is_a_small_ball
choking_hazard_small_parts, no_warning_applicable`. 
* `CPSIAWarningDescription`: a `TwoFiftyStringNotNull` value
* `LegalDisclaimer`: a `xsd:normalizedString` up to 2500 chars
* `Manufacturer`: a `HundredString` value
* `MfrPartNumber`: a `FortyStringNotNull` value for manufacture part number
* 0 to 5 `SearchTerms`: a `StringNotNull` value for search keywords
* 0 to 20 `PlatinumKeywords`: a `StringNotNull` value to map a product to
nodes in a custom browse structure.
* `Memorabilia`: a boolean value
* `Autographed`: a boolean value
* 0 to 5 `UsedFor`: a `StringNotNull` value. 
* `ItemType`: a `LongStringNotNull` value showing where to put in Amazon
browse structure. 
* 0 to 5 `OtherItemAttributes`: a `LongStringNotNull` value for further 
classification.
* 0 to 4 `TargetAudience`: a `StringNotNull` value for classification
* 0 to 5 `SubjectContent`: a `StringNotNull` value for merchandising ideas
* `IsGiftWrapAvailable`: a boolean value
* `IsGiftMessageAvailable`: a boolean value
* 0 to 10 `PromotionKeywords`: a `StringNotNull` value
* `IsDiscontinuedByManufacturer`: a boolean value for classification
* `DeliveryScheduleGroupID`: ?? a `StringNotNull` value
* 0 to 2 `DeliveryChannel`: a value from `in_store, direct_ship`
* 0 to 2 `PurchasingChannel`: a value from `in_store, online`
* `MaxAggregateShipQuantity`: a positive integer
* `IsCustomizable`: true or false
* `CustomizableTemplateName`: a `StringNotNull` value
* `TSDAgeWarning`: toy safety directive age warning. A value from a 
`not_suitable_under_36_months, not_suitable_under_3_years_supervision, 
..., not_suitable_under_14_years_supervision, 
no_warning_applicable`.
* 0 to 8 `TSDWarning`: a value from `only_domestic_use, adult_supervision_required,
protective_equipment_required, water_adult_supervision_required, toy_inside,
no_protective_equipment, risk_of_entanglement, fragrances_allergy_risk, 
no_warning_applicable`. 
* 0 to 21 `TSDLanguage`: a value from `English, French, ..., Swedish`
* 0 to 2 `OptionalPaymentTypeExclusion`: a value from `cash_on_delivery, 
csv, exclude_none, exclude cod, exclude cvs, exclude cod and cvs`.

There are many types of product data element such as `ClothingAccessories`,
`Clothing`, `Home`, etc. Here is the details of the `Shoes` schema.  
The `Shoes` schema has three elements: `ClothingType`, 
`VariationData`, and `ClassificationData`. 

The required `ClothingType` element is a value from `Accessory, Bag, Shoes,
ShoeAccessory, Handbag, Eyewear`. 

The `VariationData` has the following elements:

* `Parentage`: a value from `parent, child`
* `Size`: a `StringNotNull` value
* `Color`: a `StringNotNull` value
* `VariationTheme`: a value from `Size, Color, SizeColor, 
ColorName-MagnificationStrength, ColorName-LensColor, ColorName-LensWidth, 
Material, SizeStyle, StyleName, PatternName, Size-Material, 
Size-PatternName, LensColor, LensColorShape,
LensColorMaterial, ColorSize`.

The `ClassificationData` has many elements but only a few applicable 
to a specific product. Some sample elements are: 

* `CountryOfOrigin`: two-char country code or `unknown`. 
* `ColorMap`: a a `String` value for color (normalized string up to 50 chars)
* `SizeMap`: a size string 
* `CareInstructions`: a string of care instruction
* `CountryProducedIn`: a string of country that produces it
* `Department`: a `StringNotNull` value
* `FabricType`: a `String` value 
* 0 to 3 `MaterialType`: a `String` value
* `WarrantyType`: a `StringNotNull` value
* `ManufacturerWarrantyType`: a `StringNotNull` value
* `SellerWarrantyDescription`: a `StringNotNull` value
* `StyleName`: a `TwoThousandString` value

### `Inventory` Schema
The `Inventory` schema has the following elements:

* `SKU`: Required. A `SKUType` value
* `FulfillmentCenterID`: A `String` value for seller-defined fulfillment center. 
If use `AFN`, it is `AMAZON_NA`. 
* A choice of `Available, Quantity, Lookup`. Just use `Quantity` that is a 
non-negative integer. If use `AFN`, it is `<Lookup>FulfillmentNetwork</Lookup>`.
* `RestockDate`: a restock date if not currently available
* `FulfillmentLatency`: a number between 1 and 30 showing the days between the 
order date and the ship date. 
* `SwitchFulfillmentTo`: a value from `MFN, AFN`. `MFN` means manufacturer 
fulfillment. `AFN` means Amazon fulfillment. 

### `Price` Schema
The `Price` schema has the following elements: 

* `SKU`: Required. A `SKUType` value
* `StandardPrice`: a `OverrideCurrencyAmount` value. It extends 
the `CurrencyAmountWithDefault` type that has a `currency` attribute 
such as `currency="USD"` and a number that has a total of 20 digits 
in which 4 are faction digits. 
* `MAP`: a `OverrideCurrencyAmount` value for minimum advertised price. 
Both the standard price and sale price must be higher than the MAP value. 
US-only. 
* `DepositAmount`: a `CurrencyAmountWithDefault` value for deposit. 
* `Sale`: sale date and price information that has the following elements: 
    * `StartDate`:
    * `EndDate`: 
    * `SalePrice`: a `OverrideCurrencyAmount` value
* `CompareAt`: a dated price that has a start date, end date and a 
compare at price.
* `Previous`: a `DatedPrice` value that has start, end date, price and 
previous price.
* Rental prices from `Rental_0` to `Rental_9`. 
* `CostPerClickBidPrice`: a `OverrideCurrencyAmount` value. 

### `ProductImage` Schema
The `ProductImage` schema has the following elements: 

* `SKU`: Required. A `SKUType` value
* `ImageType`: Required. a value from `Main, Swatch, PT1, PT2, ..., PT8, Search, ...`
* `ImageLocation`: an HTTP (not HTTPS) url for a JPEG or GIF file. 

### Relationship 
Before uploading relationship feed, all products should be uploaded and 
`VariationData` values are specified. 
The `Relationship` schema has the following elements: 

* `ParentSKU`: Required. A `SKUType` value
* 1 to many `Relation`: Required. Child SKU and relationship information. It 
has the following elements:
    * `SKU`: the child SKU
    * `ChildDetailPageDisplay`: a value from `independently_displayable, 
    display_only_on_parent`. 
    * `Type`: a value from `Variation, Accessory, ...`. 


# Orders API
Order processing involves generating orders reports, submitting 
order acknowledgment feed, shipping feed, order adjustment 
(optional) and being paid. 

The [Orders API][10] allows you to retrieve orders and order times during a 
specified time frame. The Orders API section supports the following operations:

* [ListOrders][11]: Get orders created or updated during a specified
time frame.
* [ListOrdersByNextToke][12]: Use **NextToken** to get the next page of orders.
* [GetOrder][13]: Get orders using **AmazonOrderId**.
* [ListOrderItems][14]: Get order items using **AmazonOrderId**.
* [ListOrderItemsByNextToken][15]: Get the next page of order items.
* [GetServiceStatus][16]: Get the Orders API operational status. 


[1]: https://developer.amazonservices.com/
[2]: http://docs.developer.amazonservices.com/en_US/dev_guide/index.html
[3]: http://docs.developer.amazonservices.com/en_US/feeds/index.html
[4]: http://docs.developer.amazonservices.com/en_US/feeds/Feeds_SubmitFeed.html#Feeds_SubmitFeed
[5]: http://docs.developer.amazonservices.com/en_US/feeds/Feeds_GetFeedSubmissionList.html#Feeds_GetFeedSubmissionList
[6]: http://docs.developer.amazonservices.com/en_US/feeds/Feeds_GetFeedSubmissionListByNextToken.html#Feeds_GetFeedSubmissionListByNextToken
[7]: http://docs.developer.amazonservices.com/en_US/feeds/Feeds_GetFeedSubmissionCount.html#Feeds_GetFeedSubmissionCount
[8]: http://docs.developer.amazonservices.com/en_US/feeds/Feeds_CancelFeedSubmissions.html#Feeds_CancelFeedSubmissions
[9]: http://docs.developer.amazonservices.com/en_US/feeds/Feeds_GetFeedSubmissionResult.html#Feeds_GetFeedSubmissionResult
[10]: http://docs.developer.amazonservices.com/en_US/orders/2013-09-01/index.html
[11]: http://docs.developer.amazonservices.com/en_US/orders/2013-09-01/Orders_ListOrders.html
[12]: http://docs.developer.amazonservices.com/en_US/orders/2013-09-01/Orders_ListOrdersByNextToken.html
[13]: http://docs.developer.amazonservices.com/en_US/orders/2013-09-01/Orders_GetOrder.html
[14]: http://docs.developer.amazonservices.com/en_US/orders/2013-09-01/Orders_ListOrderItems.html
[15]: http://docs.developer.amazonservices.com/en_US/orders/2013-09-01/Orders_ListOrderItemsByNextToken.html
[16]: http://docs.developer.amazonservices.com/en_US/orders/2013-09-01/MWS_GetServiceStatus.html
[17]: https://sellercentral.amazon.com/gp/help/help.html/ref=ag_1611_cont_69042?ie=UTF8&itemID=1611&language=en_US
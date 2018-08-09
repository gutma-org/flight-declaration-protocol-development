# UAV/Operator Flight Declaration Exchange Protocol

- [UAV/Operator Flight Declaration Exchange Protocol](#uavoperator-flight-declaration-exchange-protocol)
	- [1 Introduction](#1-introduction)
	- [2 Abstract](#2-abstract)
	- [3 Background](#3-background)
	- [4 Introduction](#4-introduction)
		- [4.1 Key Assumptions](#41-key-assumptions)
		- [4.2 Definitions](#42-definitions)
		- [4.3 Requirements](#43-requirements)
		- [4.4 Mechanics](#44-mechanics)
		- [4.5 Out of Scope](#45-out-of-scope)
			- [4.5.1 Querying Mechanism](#451-querying-mechanism)
			- [4.5.2 Authorization](#452-authorization)
			- [4.5.3 Message verification](#453-message-verification)
	- [5 Standard Concepts](#5-standard-concepts)
		- [5.1 Security](#51-security)
		- [5.2 Versioning](#52-versioning)
		- [5.3 Dates & Times](#53-dates--times)
		- [5.4 Geospatial Data](#54-geospatial-data)
		- [5.5 Distances](#55-distances)
		- [5.6 Flight Identifier](#56-flight-identifier)
		- [5.7 Contact URL](#57-contact-url)
	- [6 Reliability & Scalability](#6-reliability--scalability)
		- [6.1 Atomic updates](#61-atomic-updates)
		- [6.2 Timestamp and Sequence Number](#62-timestamp-and-sequence-number)
		- [6.3 Deletion](#63-deletion)
		- [6.4 EndPoint Address](#64-endpoint-address)
		- [6.5 Response codes](#65-response-codes)
		- [6.6 Example message flows](#66-example-message-flows)
	- [7 Types](#7-types)
		- [7.1 message](#71-message)
		- [7.1.1 Example](#711-example)
		- [7.2 altitude](#72-altitude)
		- [7.2.1 altitudeDatum enum](#721-altitudedatum-enum)
			- [7.2.2 Example](#722-example)
		- [7.3 flightPart](#73-flightpart)
			- [7.4.1 Notes](#741-notes)
			- [7.4.2 Example](#742-example)
		- [7.5 ident](#75-ident)
		- [7.5.1Notes](#751notes)
		- [7.5.2Example](#752example)
		- [7.6 operationMode enum](#76-operationmode-enum)
		- [7.7 flightDeclaration](#77-flightdeclaration)
			- [7.7.1 Notes](#771-notes)
			- [7.7.2 Examples](#772-examples)
		- [7.8 error](#78-error)
	- [8 References](#8-references)
	- [9 Revision History](#9-revision-history)
	- [10 Attribution](#10-attribution)

## 1 Introduction
_A protocol designed to facilitate the secure exchange of flight situation data between UTM Providers, 
while allowing each UTM Provider to retain ownership of their customer data._

## 2 Abstract

With the increasing number of drone flights, a core challenge for any UTM Provider is having access to 
sufficient data to provide a comprehensive air-situation picture.

While much of the data offered by UTM Providers is, today, largely proprietary, it is widely believed that 
for UTM Providers to be able to function going forwards, they must have a mechanism to share safety-related 
data between them.

For the purposes of this document, "safety-related" data is generally taken to mean data that describes drone 
activity taking place (or scheduled to take place) known only to the originating UTM Provider, but that could 
benefit **all** UTM Providers and other interested parties by sharing key elements of that data.

For example:

- In a common scenario, Provider A has a customer base which is busy planning/conducting drone flights and tracking these in Provider A's system. Provider B offers a similar service within the same geography, however neither Party A nor Party B can offer their customers a comprehensive air picture because each hold their own data in private silos.

- It is the proposal put forward in this protocol specification to describe a mechanism through which Flight-related safety data can be exchanged between UTM Providers in a way that allows core safety data to be exchanged, while permitting each individual UTM Provider to retain ownership and control of their data.


## 3 Background

The Global UTM Association is committed to the advancement and adoption of technical systems and standards which promote the safe integration of drones into the airspace.

As part of the association's commitment to this goal, we have prepared this protocol specification to solve a vital safety problem affecting all UTM Providers today: the ability to securely share important 
safety information with each other, while retaining strict privacy controls.

## 4 Introduction

### 4.1 Key Assumptions

In the specification of this protocol, we have made several key assumptions:

- It is not the primary concern of each individual UTM Provider to build reliable, distributed infrastructure to facilitate the exchange of flight data between competing providers, however it is recognized that access to such data would enhance the proposition of Interested Parties;

- The number of drone flights is set to rise dramatically over the coming years, and as such not every UTM Provider will have the capability and/or desire to implement a notification-based geospatial system.

- Many UTM Providers will either not want to, or be unable to, invest the significant time required in building systems which can compute the air-situation picture as changes occur in real-time and would prefer to receive computed data, rather than poll for it.

To facilitate a scalable and reliable data exchange, this document describes an event-based notification system where the Originating Party notifies Interested Parties of a flight or changes to a previously announced flight.

As core design objectives, we have sought to base the protocol on an architecture which better facilitates low-latency and high-scale, and permit the exchange of critical flight safety data between UTM Providers who wish to retain ownership and control of their user data.

### 4.2 Definitions

In this specification, we introduce and propose the following definitions:

| Term | Meaning |
| --- | --- |
| Originating Party | The UTM Provider who wishes to publish data to other members |
| Interested Party | Any party receiving a flight declaration that is not the Originating Party. For the purposes of this specification, this party is essentially the "consumer" of data published by Originating Parties. |
| Flight declaration | Information, originating from a user, that describes their intent to fly or that they are flying within the notified region. Only one drone can participate in a flight declaration at any one time. |
| Operating Area/Notified Area | The area that a drone pilot is or is planning to operate in. Unless the operator defines a more specific area, this can be assumed to be a circle around the operator's location with a radius equal to the maximum distance the operator is permitted to fly. |
| Route | One or more consecutive straight lines the drone will follow. |
| User/Pilot | The party declaring the flight (i.e. a customer of an Originating Party). |
| Very low altitude | Describes the airspace below which General Aviation is not permitted to operate – typically below 500 feet AGL. |

### 4.3 Requirements

Additionally, the words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in the remainder of this document are to be interpreted as described in RFC 2119 [[l]](#Ref-1).

### 4.4 Mechanics

For each flight declaration received from their Users, the Originating Party makes an HTTPS POST, containing the Flight Declarations data, to all the Interested Parties' registered endpoints. This POST contains all the details of the flight declaration in a JSON [[2]](#Ref-2) serialized message object.

### 4.5 Out of Scope

The following areas are regarded as out of scope from this specification, as their definition will necessarily depend on implementing parties' business models they are effectively implementation details, where practical, the authors have attempted to address any "out of scope" items by providing prototype implementations  that early adopters/testers of this specification can use to practically  verify the suitability of this protocol against its objectives.

#### 4.5.1 Querying Mechanism

The mechanism where interested parties subscribe for updates. This specification does not define a programmatic method to subscribe to notifications. The expectation is that the Interested and Originating parties will either have a commercial relationship or use a common software (e.g. InterUSS) to query the data. 

The authors note there are some implementation challenges associated with a point-to-point implementation of this protocol, notably:

- Interested Parties need seek, identify and establish commercial relationships with each UTM Provider operating within a specific geographic region

- Originating Parties need maintain separate systems and commercial relationships with a changing (growing) list of Interested Parties.

#### 4.5.2 Authorization

The allocation of credentials to interested parties is regarded as part of the subscription process resulting in the Authorization and Authentication for the endpoint to regarded as out of scope. Organizations implementing this standard 
should be aware of the reputational damage that data breaches and unauthorized modification of data will result in and must ensure their endpoints are secured to a level that they feel appropriate.

#### 4.5.3 Message verification

The protocol recommends – but does not presently mandate – that the parties perform verification of each message. It is expected that this could take the form of a cryptographically-signed hash included as part of the HTTP headers.

## 5 Standard Concepts

### 5.1 Security

While this document does not mandate how implementers perform authentication and authorization, it does mandate that HTTP over a minimum of TLS 1.2 [[3]](#Ref-3) must be used for all flight data notifications between originating and interested parties.

### 5.2 Versioning

All version numbers must follow the Semantic Versioning 2.0.0 standard [[4]](#Ref-4)

> _Given a version number MAJOR.MINOR.PATCH, increment the:_
> 
> 1) _MAJOR version when you make incompatible API changes,_
> 
> 2) _MINOR version when you add functionality in a backwards-compatible manner, and_
> 
> 3) _PATCH version when you make backwards-compatible bug fixes._
> 
> _Additional labels for pre-release and build metadata are available as extensions to the MAJOR.MINOR.PATCH format._

### 5.3 Dates & Times

Dates and times will follow the **ISO-8601** **[[5]](#Ref-5)** formatting standard. Local times are not supported; all times must be in UTC or have a time zone offset specified.

No support is made for just a date or just a time. For comparison of time ranges, the start time is regarded as included in the time range, where the end time is excluded.

The format pattern for date times is `YYYY-MM-DDTHH:mm:ss.sssZ` where Z is either the character _Z_ to represent UTC, _or_ the +/- timezone offset from UTC. If a message is received with no timezone offset, it should be regarded as an error 
and the correct error code returned.

The requirement to specify times or dates does not apply to the specification, so all temporal data will be defined as a _datetime_.

No guarantees that a timezone offset will be preserved should be made.

### 5.4 Geospatial Data

Geospatial data must be described using a geometry object as defined in the _GeoJSON_ specification [[6]](#Ref-6) with the following specific requirements:

- Using the default CRS - geographic coordinate reference system, using the WGS84 datum, and with longitude and latitude units of decimal degrees

- Bounding box is not expected

Latitude and Longitude should not be specified to more than 8 decimal places. This gives an accuracy of approximately 1.1 mm at the equator making any further digits superfluous.

### 5.5 Distances

All distances (both horizontal and vertical) are specified in metres.

### 5.6 Flight Identifier

This is a unique identifier generated by the Originating Party that should provide an _anonymous_ identifier for this flight. The Originating Party must be able to use this identifier to identify the original records that resulted in this flight. Together with knowledge of who the Originating Party is, the flight identifier constitutes a globally unique identifier for this flight. i.e. Identifiers may be shared across originators, but the must be unique within an originator.

### 5.7 Contact URL

As part of the flight declaration, the Originating Party must provide a URL that can be used to initiate contact with the end User. This mechanism ensures that the drone pilot can be contacted (subject to the correct legal provisions) 
without the Originating Party having to distribute any personally identifiable information outside of their own system boundary.

For clarity, the provision of the Contact URL means that an Interested Party who is displaying data received from an Originating Party within its products is able to provide their users with a mechanism to interact with the pilot through a system boundary. This mechanism allows UTM Providers to satisfy proposed regulatory requirements to deliver this functionality, without requiring the UTM Provider to also publish contact data for its users.

It is expected that this URL would resolve in a way that the operator can be reached. It is expected that the method of contact will depend on the jurisdiction and the URL could resolve to a phone call or SMS or other methods (e.g. in App screens). It is not expected that this URL would be shared with any of an interested party's users without some form of authentication and auditing.

## 6 Reliability & Scalability

To ensure that this protocol is applicable in scalable, asynchronous distributed systems, it has been designed to push notifications to Interested Parties rather than requiring them to continuously poll for flights updates.

The protocol provides support for missing messages, out-of-order delivery of messages and multiple sends through the following mechanisms.

### 6.1 Atomic updates

When a Flight Declaration is updated by an Originating Party, that _entire_ Flight Declaration record will be sent. This allows the receiving service to replace the record in its entirety and allows the system to self-heal in the 
event of a missing message. This also removes the need to differentiate from a create and update – both are handled in the same way.

### 6.2 Timestamp and Sequence Number

To account for the possibility of messages being delivered out-of-order, a combination of timestamp and sequence number must be used. A timestamp of the creation of the message must be added at every declaration. When a flight declaration is updated, the sequence number must also be increased to make it numerically greater than the previous update. This allows the receiver to ensure that they are not overwriting more recent data if they process a delayed update message. The timestamp and the sequence numeber used in combination serve as the primary method of verification of sequence. The timestamp should be considered a **primary** method while the sequence number should be considered as a **secondary** form of verification. 

Sequence numbers do not have to be consecutive, nor do they need to start from zero, however they do need to be an unsigned integer – i.e. a whole number greater than or equal to zero.

### 6.3 Deletion

To delete a Flight Declaration a null flightDeclaration element should be sent in the [message](#message). The flightId must be provided and the sequence number increased. It is regarded as invalid to update a previously deleted record, once a flight declaration is deleted it must not be re-activated.


### 6.4 EndPoint Address

The address of the notification endpoint will be provided by the Interested Party at subscription time. For this reason, no URL is provided in this spec.

### 6.5 Response codes

This specification does not preclude the use of, or override any HTTP response codes, but the following codes should be returned for these reasons:

| Header | Reason |
| --- | --- |
| 201 Created | When this is the first message for this Originating Party with this flightId that the Interested Party has received. |
| 200 OK | Upon a successful update or deletion of a Flight Declaration. |
| 303 Not Modified | If the sequence number of the passed message is less than or the same as the record that the Interested Party already has. |
| 400 Bad Request | If the message fails validation, or -If the message is of a version that is not supported by the endpoint, or -Any dates transmitted do not have a time zone offset. |

A JSON-serialized error message should be returned in the response body with the shouldRetry property set to the 
appropriate value – e.g. set to false if the message failed validation.

### 6.6 Example message flows

**Example 1**

Below is a description of a set of messages that would be sent by an Originating Party when one of its users planned a drone operation in-advance of the actual flight time, revised once, then actually executed the flight:

- Initial declaration of intent to fly. The Sequence number is 0, the start _datetime_ is in the future.
- Closer to commencement of the operation as declared in the original Flight Declaration, the operator notifies their UTM Provider of a revision to the flight plan. The Originating Party issues a subsequent Flight Declaration which is then sent on to all the Interested Parties, and the sequence number is then incremented to 1.
- At take-off, the updated message now includes a take-off time and has the sequence number incremented to 2, and subsequently distributed to all parties.
- At the conclusion of the operation, the update message has a landing time and the sequence number set to 3.

**Example 2**

Below is a set of messages that would be sent for a flight that is planned in advance and then cancelled
- Initial declaration of intent to fly. The sequence number is 0, the start date is in the future.
- Deletion of the flight – the update message has a null value for flightDeclaration and the sequence number is incremented to 1.

**Example 3**

Below is a set of messages that would be sent for a flight that is not planned in advance planned in advance

- Initial declaration of flight. The sequence number is 0, the start date and the take-off time are set to that actual time of take-off.

## 7 Types
### 7.1 message

The primary entity exchanged between Originating and Interested Parties.

| Name | Description | Type |
| --- | --- | --- |
| **flightId** | Identifier provided by the Originating Party that uniquely identifies this declaration from other declarations provided by the same Originating Party. | string |
| **sequenceNumber** | A number that represents the version of this message data. When a record is modified, the sequence number must be numerically greater than the previous update. | number (uint64) |
| **timeStamp** |  **ISO-8601** **[[5]](#Ref-5)** formatting standard. Local times are not supported; all times must be in UTC or have a time zone offset specified. | YYYY-MM-DDTHH:mm:ss.sssZ |
| **flightDeclaration** | A flightDeclaration object describing this proposed flight. To delete a flight, this field should be null. | flightDeclaration |
| **version** | The version of this protocol that the message has been implemented from. | string - currently "0.2.0" |

### 7.1.1 Example

    {
        "flightId": "5a7f3377-b991-4cc8-af2d-379d57f786d1",
        "sequenceNumber": 0,
        "timeStamp": ""2017-02-01T15:00:00.000Z",
        "flightDeclaration": { ... },
        "version": "1.0.0"
    }

### 7.2 altitude

Altitude is specified in Metres above the specified datum. The altitude type combines both values.

| Name | Description | Type |
| --- | --- | --- |
| **metres** | The height above the specified datum in metres | number |
| **datum** | The datum that describes what the altitude measurement is relative to | altitudeDatum { "agl", "amsl", "sps", "wgs84" } |


### 7.2.1 altitudeDatum enum

This specification defines the following altitude datums, however specific messages may not allow altitudes to be defined using certain datums (e.g. it makes no sense to declare the maximum altitude of a pre-planned very low altitude flight relative to the SPS datum). The following string values are supported for datums effectively creating an enumeration.

| Datum | Description |
| --- | --- |
| **agl** | Above Ground Level |
| **amsl** | Above Mean Sea Level. This value is included for completeness. As this datum is not valid for any of the messages in this specification, the issue of defining the tidal datum for Mean Sea Level has not been included. |
| **sps** | Altitude where a barometric altimeter would be set to the Standard Pleasure Setting. This is effectively of the Flight Level multiplied by 100 and converted to metres. |
| **wgs84** | Distance above the WGS 84 datum. |


#### 7.2.2 Example

An altitude 152.4 metres above ground level would be represented by the following entity:

    {
        "metres": 152.4,
        "datum": "agl"
    }
### 7.3 flightPart

A flight consists of one or more parts and these parts must be declared as a GeoJSON FeatureCollection. Each part has a start and end time as well as a geography and maximum altitude. In addition to describing the the Geogrpahic area of operation in a Polygon or LineString format. The following properties must be declared for each format. 

| Name | Description | Type |
| --- | --- | --- |
| **id** | An identifier that uniquely identifies this part within this flight. | string |
| **geography** | A Polygon or LineString describing the planned operating area or route. | geometry |
| **startTime** | The time that the flight is expected to start. | datetime |
| **endTime** | The time that the flight is expected to be completed by. This must always be greater than startTime. | datetime |
| **maxAltitude** | The maximum altitude that the drone will achieve during the _flightPart_. | altitude |

#### 7.4.1 Notes

- No parts of a declared flight can have overlapping start and end times.
- For the geography, a:
  - _Polygon_ is used to define the area where a drone is expected to operate.
  - _LineString_ should be used when there is a defined route that the drone is expected to follow.
  - No other GeoJSON types are supported.
- When the geography is a _Polygon_, multiple linear rings representing the exterior ring (and optionally 'holes') should be supported. This allows the geography to define parts of the area where the drone will not operate.
- When the geography is a _LineString_, it may have more than two points.
- For version 1.0 of the specification, neither sps nor amsl are supported datums for maxAltitude.

#### 7.4.2 Example

The following example shows a flight part defining a 30-minute flight with a polygonal area of operation.

	{
		  "type": "FeatureCollection",
		  "features": [
		    {
		      "type": "Feature",
		      "properties": {
		      		"id:"1",
			      	"startTime": "2017-02-01T15:00:00+00:00",
	        		"endTime": "2017-02-01T15:30:00+00:00",
			        "maxAlt": {
			            "metres": 152.4,
			            "datum": "agl"
			        }
				},
		      "geometry": {
		        "type": "Polygon",
		        "coordinates": [
		          [
		            [
		              -6.314907073974609,
		              53.252030291779924
		            ],
		            [
		              -6.3126325607299805,
		              53.252030291779924
		            ],
		            [
		              -6.3126325607299805,
		              53.253301239242234
		            ],
		            [
		              -6.314907073974609,
		              53.253301239242234
		            ],
		            [
		              -6.314907073974609,
		              53.252030291779924
		            ]
		          ]
		        ]
		      }
		    }
		  ]
	}

   
### 7.5 ident

To allow a flight to be correlated with positional data from other sources (e.g. ADSB or RADAR) a flight declaration can 
include one or more _idents_ that will be associated with this flight.

| Name | Description | Type |
| --- | --- | --- |
| **method** | The name of the technology providing the ident | string { "adsb", "flarm" } |
| **ident** | The identifier in the specified data source that will identify this flight. | string |

### 7.5.1Notes

It is recognised that not all idents that a drone may be allocated will be available at the time of flight declaration. 
If a drone is allocated an ident it should update the flight declaration to include the new ident.

It must not be treated as an error if the Interested Party does not recognise a method that is provided.

### 7.5.2Example

    {
        "method": "adsb",
        "ident": "4840D6"
    }

### 7.6 operationMode enum

| Datum | Description |
| --- | --- |
| **vlos** | The drone is being flown by a human pilot within visual line of sight |
| **evlos** | The drone is being flown by a human pilot with extended visual line of sight – typically enabled by the use of observers. |
| **bvlos** | The drone is being flown by a human pilot beyond visual line of sight |
| **automated** | The drone does not have a human pilot |



### 7.7 flightDeclaration

| Name | Description | Type |
| --- | --- | --- |
| **parts** | One or more part that make up this flight | array |
|**purpose** | A human readable description of the reason the flight is being conducted. This field can be omitted if the end user chooses not to share the purpose of their flight. (_See notes_)| string |
| **expectTelemetry** | A flag indicting whether it is expected that telemetry will be available during the flight. | boolean |
| **originatingParty** | The name of the party that the flight was originally declared with. | string |
| **contactUrl** | The URL to be use to initiate contact with the user. This can be used to make nuisance report about this flight or for law enforcement to start the process of identifying a drone operator. It is expected that the flightId will be used as part of this Url as the Url must be standalone and not require any other information. | string |
| **operationMode** | The mode that the drone is being operated in. | operationMode { "vlos", "evlos", "bvlos", "automated" } |
| **idents** | Any idents that are associated with this flight | array _[optional]_ |
| **actualTakeOffTime** | The time the flight took off. This value can be null or omitted if the take-off time is not known | datetime _[optional]_ |
| **actualLandingTime** | The time the flight completed. This value can be null or omitted if the landing time is not known | datetime _[optional]_ |

#### 7.7.1 Notes

It is expected that a future version of this specification will include a telemetryEndPoint field. This is an endpoint 
that an interest party would be able to call to get live telemetry while the flight was in progress.

In future, we recommend moving to a more structured taxonomy of flight purposes however this is outside the scope of our 
initial draft and we would welcome input focused in this area.

#### 7.7.2 Examples

**Example 1:** A VLOS survey flight with a polygonal area of operation.

    {
        "parts": {
			  "type": "FeatureCollection",
			  "features": [
			    {
			      "type": "Feature",
			      "properties": {
		            "startTime": "2017-02-01T15:00:00+00:00",
		            "endTime": "2017-02-01T15:30:00+00:00",
		            "maxAlt": {
		                "metres": 152.4,
		                "datum": "agl"
		            }
		        	},
			      "geometry": {
			        "type": "Polygon",
			        "coordinates": [
			          [
			            [
			              -6.290252208709717,
			              53.21870259514697
			            ],
			            [
			              -6.285746097564697,
			              53.21870259514697
			            ],
			            [
			              -6.285746097564697,
			              53.22109226443749
			            ],
			            [
			              -6.290252208709717,
			              53.22109226443749
			            ],
			            [
			              -6.290252208709717,
			              53.21870259514697
			            ]
			          ]
			        ]
			      }
			    }
			  ]
			},
        "purpose": "Agricultural survey",
        "expectTelemetry": false,
        "originatingParty": "Originating Party, Inc.",
        "contactUrl": "https://utm.originatingparty.com/contact?d6c8cec9-2d57-43f6-8301-53efee5702b4",
        "operationMode": "vlos"
    }

**Example 2:** A BVLOS delivery flight with an out bound leg and a return leg, the return leg being flown an hour 
after the return leg. This drone is expecting to provide telemetry during the flight


		{
			"parts": {
			  "type": "FeatureCollection",
			  "features": [
			    {
			      "type": "Feature",
			      "properties": {
							"id":"0"
							"startTime": "2017-02-01T15:00:00+00:00",
							"endTime": "2017-02-01T15:30:00+00:00",
							"maxAlt": {
								"metres": 152.4,
								"datum": "agl"
							}
					},
			      "geometry": {
			        "type": "LineString",
			        "coordinates": [
			          [
			            -6.294693946838379,
			            53.21507929385216
			          ],
			          [
			            -6.284565925598144,
			            53.22237697776495
			          ]
			        ]
			      }
			    },
			    {
			      "type": "Feature",
			      "properties": {
							"id":"1",
							"startTime": "2017-02-01T16:00:00+00:00",
							"endTime": "2017-02-01T16:30:00+00:00",
							"maxAlt": {
								"metres": 152.4,
								"datum": "agl"
							}
					},
			      "geometry": {
			        "type": "LineString",
			        "coordinates": [
			          [
			            -6.28392219543457,
			            53.22213288519814
			          ],
			          [
			            -6.294286251068114,
			            53.2148865564753
			          ]
			        ]
			      }
			    }
			  ]
			},
			"purpose": "Delivery",
			"expectTelemetry": true,
			"originatingParty": "Originating Party, Inc.",
			"contactUrl": "https://utm.originatingparty.com/contact?5a7f3377-b991-4cc8-af2d-379d57f786d1",
			"operationMode": "bvlos"
		}

### 7.8 error

| Name | Description | Type |
| --- | --- | --- |
| **errorDescription** | A human-readable description of what the error was. | string |
| **shouldRetry** | A flag to indicate that the whether the message should be queued for resending. | bool |
| **paramName** | If the error was caused due to validation failure, the name of the parameter should be provided in this field. | string _[optional]_ |

**Example 1: Retry not required**

    {
        "errorDescription": "Server doesn't require sender to retry",
        "shouldRetry": false
    }
    
## 8 References
|||
| --- | --- |
| [1] | "RFC 2119: Key words for use in RFCs to Indicate Requirement Levels" - [https://www.ietf.org/rfc/rfc2119.txt](#https://www.ietf.org/rfc/rfc2119.txt) |
| [2] | "RFC 7159: The JavaScript Object Notation (JSON) Data Interchange Format" - [https://tools.ietf.org/html/rfc7159](#https://tools.ietf.org/html/rfc7159) |
| [3] | "RFC 2818: HTTP Over TLS" - [https://tools.ietf.org/html/rfc2818](#https://tools.ietf.org/html/rfc2818) |
| [4] | "Semantic Versioning 2.0.0" - [http://semver.org/](#http://semver.org/) |
| [5] | "ISO 8601 - Date and time format" - [http://www.iso.org/iso/home/standards/iso8601.htm](#http://www.iso.org/iso/home/standards/iso8601.htm) |
| [6] | "RFC 7946: The GeoJSON Format" - [https://tools.ietf.org/html/rfc7946](#https://tools.ietf.org/html/rfc7946)|

## 9 Revision History
**REVISION HISTORY**

| Version | Date | Comments |
| --- | --- | --- |
| 0.2.1-draft | 01/07/18 | Revisions from GUTMA |
| 0.1.0-draft | 01/09/16 | Initial draft specification by Altitude Angel |
| 0.1.1-draft | 03/11/16 | Updates to terminology and inclusion of example message flows |
| 0.2.0-draft | 16/01/17 | Version 0.2 ready for public release and comment |

## 10 Attribution

**ATTRIBUTION NOTICE**

| Author | Company | Role |
| --- | --- | --- |
| H. Ballal | GUTMA | Revisions |
| N. Kidd | Altitude Angel | Original author |
| R. Garfoot | Altitude Angel | Contributor |
| R. Parker | Altitude Angel | Contributor |

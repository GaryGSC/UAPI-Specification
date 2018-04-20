# BYU University API Standard

Version 1.0

The BYU University API Standard is licensed under [The Apache License, Version 2.0](http://www.apache.org/licenses/LICENSE-2.0.html).

* Note - Unless otherwise specified the URLs contained in this document are for example purposes only. The actual URLs to any UAPI resource can be found [here](https://developer.byu.edu).


## 1.0 Introduction

The University API (UAPI) Specification is an effort intended to bring standardization to consumers and producers of [API](#glossary)s. It presents a standard for URL design, request and response format, and HTTP methods and return codes. The UAPI standard is required for any API designated as part of the University API and recommended for other APIs whenever feasible. 

### 1.1 University APIs vs Domain APIs

The University has designated a number of common [resources](#glossary) as part of the UAPI implementation such as students, employees, and persons. Other resources represent  specific constructs useful within a specific [business domain](#glossary). It is recommended where possible to apply the UAPI standard to domain specific APIs. Detailed information for applying the UAPI standard to domain APIs can be found [here](#x.0 Domain APIs and the UAPI Standard)

## 2.0 URLs

URLs provide a unique address space for resources within the UAPI. The UAPI specifies a format for URLs to ensure consistency across resources. 

### 2.1 URL Design

A URL that is part of the UAPI implementation will have the following format:

`https://api.byu.edu/byuapi/resource[/identifier]`

If the resource has sub-resources (e.g. persons have addresses) they are addressed as follows:

```
    https://api.byu.edu/byuapi/<top-level-resource>/<top-level-resource-identifier>/<sub-resource>/<sub-resource-identifier>
```

#### 2.1.1 Top-level Resource

The resource portion of the URL represents an entity within the UAPI. For example, the resource "persons" contains  data related to a person. The resource `https://api.byu.edu/byuapi/persons` is the URL for the Persons resource. 

Resources that appear first in the URL structure are considered top-level resources.

All resources are written in their plural form. There are a few exceptions where the API represents a single instance of a resource but these are rare. 

#### 2.1.2 Sub-Resource

Resources are "sub-resources" if they are part of a top-level resource. For example, addresses are sub-resources of a persons resource because a person has addresses associated with them. 

A sub-resource can be addressed as part of the top-level resource by adding the name of the sub-resource to the URL. For example,

`https://api.byu.edu/byuapi/persons/123456789/addresses`

is the URL for the addresses associated with the person identified by the byu_id of 123456789.

#### 2.1.3 Resource (and Sub-Resource) Identifier

If a resource represents a collection of data an item in that resource collection should be addressable by adding its identifier to the URL. For example, the URL `https://api.byu.edu/byuapi/persons/123456789` represents the person with the byu_id of 123456789. An HTTP GET to this URL would return the data related to this particular person.   

Sub-resources can also have identifiers. The URL for the work address of a person would be `https://api.byu.edu/byuapi/persons/123456789/addresses/WRK`

#### 2.1.4 Composite Resource Identifier

Some resources and sub-resources must be identified by using more than one piece of information. This type of identifier is termed a [composite identifier](#glossary). A composite identifier uniquely identifies a resource by using multiple pieces of information. The UAPI specification states that values in a composite identifier be separated by a comma. 

Composite identifiers should conform to the pattern:

`https://api.byu.edu/byuapi/resource/{key}[,{additional_keys}]`

An example of a URL using a composite identifier is: 

`https://api.byu.edu/byuapi/classes/2016Fall,MATH,110,001`

## 3.0 Resources 

Resources have properties associated with them. Some properties are simple name / value pairs and others may be complex objects. Sub-resources are an example of complex objects that are properties of a top-level resource.  

### 3.1 Resource Representation

When a consumer of an API interacts with a resource the representation of the properties of that resource are dependent upon the [mime-type](#glossary) used. All text based resources will be represented as [JSON](#glossary) using the `application/json` mime-type. Other text representations (XML, etc) can be provided if the API is capable. The HTTP Accept header should be used to indicate to the API which representation(s) the consumer is capable of processing and the HTTP Content-Type header indicating the mime-type of the data being represented.  Binary resource properties such as images should use the appropriate mime-type for the type of data being represented. 

### 3.1 Representing a Single Resource

Single resources are addressed by URLs that end in identifiers (e.g. `https://api.byu.edu/byuapi/persons/123456789`). 

Top level resources should return a default set of properties when no sub-resources have been requested (See [field\_sets](#field_sets) for more information on requesting multiple sub-resources in a single request). These properties should be included in a single JSON object with the name `basic`. The `basic` object and all sub-resource objects should always include the following elements:

* Links
* Metadata
* Properties 

#### 3.1.1 Links

The links property is an object that represents possible future actions that can be performed against this resource. The contents of the links section may vary depending upon the context in which the resource is being accessed (date, user authorizations, etc) See [HATEOAS](#hateoas) for a more complete discussion of HATEOS links and their purpose.  

#### 3.1.2 Metadata

The metadata property contains data about the request for this resource including any errors the request has generated. In the case of a request that returns a single resource the metadata may include the following:

|Property|Required|Description
|-----|-----|-----
|validation_response|yes|This is an object that contains two required properties: *code* and *message*.
|validation_information|no|This is an array of strings that provide information about errors correlated to the validation_response.code and HTTP response code. See [here](#post) for more information. 
|cache|required if the result is a cached value|This is an object that contains one required property: *date_time*. The date_time value is when the data was updated in the cache.
|

Metadata related to [fieldsets](#fieldsets) and [contexts](#contexts) along with any custom metadata about the resource may also be included.

The following metadata object would indicate a successful request for this resource: 

```
    "metadata": {
      "validation_response": {
        "code": 200,
        "message": "Success"
      }
    }
```

The following metadata would indicate an error occurred when processing the request:

```
"metadata": {
  "validation_response": {
    "code": 403,
    "message": "Not Authorized"
  },
  "validation_information": [
    "Not authorized"
  ]
 }
```

#### 3.1.4 Properties

Resource properties are name value pairs containing information about the state of a resource. The persons resource has properties such as byu\_id, net\_id, preferred name, etc. Property values are returned in a JSON  object that provides additional information for the consumer about the property. The JSON object has the following elements:

|Property|Required|Description
|-----|-----|-----
value|Required if the api_type is not a value communicating an error.|Contains the data value to be processed.
|api_type|Yes|Describes how this value may be used in this API. For a list of the possible values for api_type and the rules associated with each value see they list below.
|key|Required if the property is the identifier or one of the composite identifiers of the resource.|Designates that the property is one of the key elements for this resource. Key fields are required to have values - they are not allowed to be blank or null. 
|description|No|Explains the data value in human-friendly terms.
|display_label|No|Provides a suggested string to use when creating a label for this property in the user interface.
|domain|Required if the value property is part of a set of allowable values.|Contains the URL that can be used to retrieve the set of allowable values. The result of invoking the URL could be used to populate the UI's. For example the value `"domain": "https://api.byu.edu/byuapi/meta/year_terms"` may return a set of the valid year terms. See [Meta](#meta) for more information. 
|long_description|No|Explains the data value in human-friendly terms; contains more information than the (short) description.
|related_resource|Required if the api_type property is `related`.|If the api_type is `related` this property will contain the resource-name that "owns" this property. The resource-name can be used to find a HATEOAS link that can access this property.

The `api_type` field is required for all properties. It indicates more information about the value being returned such as if it is modifiable by the consumer, a derived field, a system generated value, etc. 

The possible values for `api_type` are:

|api_type|Modifiable by the consumer through this API|Description
|-----|-----|-----
read-only|No|There are either business rules or authorization considerations that do not allow the property to be changed at this time by the consumer.
|modifiable|Yes|The property may be modified by the consumer.
|system|No|The value was assigned by the system. A date-time field that tracks the last time the resource was updated is a good example of a system data value.
|derived|No|The property was derived by a "calculation" on other properties. An example might be GPA.
|unauthorized|No|The consumer is not authorized to retrieve this property. **This api\_type is deprecated. It should not be used for ANY new development.** See [Authorization](#authorization) for more information. 
|related|No|The responsibility for manipulating this property doesn't belong to this API. The business logic to modify the property exists in the "related_resource".

A property would look something like this:  

```
"byu_id": {
  "api_type": "system",
  "display_label": "BYU ID",
  "key": true,
  "value": "123456789"
},
```

#### 3.1.5 Representing Sub-Resources

Single value sub-resources (i.e. those retrieved by using an identifier in the URL) are represented much the same way the `basic` object is represented. They require the same links, metadata, and properties elements. All properties are represented just under the root of the returned object (i.e. they do not have a `basic` object). 

#### 3.1.6 Example Single Resource Response

A single top level resource representation would look like:

```
{ 
    "basic": {
        "links": {
            "persons__info": {
                "rel": "self",
                "href": "https://api.byu.edu/byuapi/persons/123456789",
                "method": "GET"
            },
            "students__info": {
                "rel": "students__info",
                "href": "https://api.byu.edu/byuapi/students/123456789",
                "method": "GET"
            }
        },
        "metadata": {
            "validation_response": {
                "code": 200,
                "message": "Successful"
            }
        },
        "byu_id": {
            "value": "123456789",
            "api_type": "system",
            "key": true
        },
        "person_id": {
            "value": "987654321",
            "api_type": "system"
        },
        "net_id": {
            "value": "joe",
            "api_type": "related",
            "related_resource": "https://api.byu.edu/byuapi/persons/123456789/credentials/NET_ID,joe"
        },
        "personal_email_address": {
            "value": "joe@byu.edu",
            "api_type": "related",
            "related_resource": "https://api.byu.edu/byuapi/persons/123456789/email_addresses/PERSONAL"
        },
        "primary_phone_number": {
            "value": "",
            "api_type": "related",
            "related_resource": "https://api.byu.edu/byuapi/persons/123456789/phones"
        },
        "date_time_updated": {
            "value": "2016-09-21T09:03:18.000Z",
            "api_type": "system"
        },
        "updated_by_id": {
            "value": "323232323",
            "description": "Joe Admin",
            "api_type": "system"
        },
        "date_time_created": {
            "value": "1997-02-07T12:22:32.000Z",
            "api_type": "system"
        }
    }
}
```

#### 3.1.7 Single Sub-resource Representation 

A single sub-resource representation would look like:

```
{
    "links": {
        "group_memberships__info": {
            "rel": "self",
            "href": "https://api.byu.edu/byuapi/persons/123456789/group_memberships/ADMINISTRATIVE",
            "method": "GET"
        },
        "group_memberships__modify": {
            "rel": "self",
            "href": "https://api.byu.edu/byuapi/persons/123456789/group_memberships/ADMINISTRATIVE",
            "method": "PUT"
        },
        "group_memberships__delete": {
            "rel": "self",
            "href": "https://api.byu.edu/byuapi/persons/123456789/group_memberships/ADMINISTRATIVE",
            "method": "DELETE"
        }
    },
    "metadata": {
        "validation_response": {
            "code": 200,
            "message": "Success"
        },
        "validation_information": [
            "No additional information"
        ]
    },
    "group_id": {
        "value": "ADMINISTRATIVE",
        "description": "Administrative",
        "api_type": "read-only",
        "key": true
    },
    "group_type": {
        "value": "A",
        "api_type": "read-only"
    },
    "byu_id": {
        "value": "123456789",
        "description": "Joe Admin",
        "api_type": "system",
        "key": true
    },
    "department": {
        "value": "OIT- Administration",
        "api_type": "related",
        "related_resource": "https://api.byu.edu/byuapi/employees",
        "domain": "https://api.byu.edu/byuapi/meta/departments"
    }
}
```

### 3.2 Representing a Collection of Resources

If a UAPI resource (top level or sub-resource) is accessed with no identifier or via a filter (see [filters](#filters) a collection of individual resource representations is returned. This collection will have its own set of links and metadata with values that pertain to the collection. The individual resource representations as defined in [3.1](#3.1) are returned in a JSON array named `values`. 

#### 3.2.1 Collection Links

The links object for a collection will contain [HATEOAS](#hateoas) links related to processing the collection. They may include links for paging, etc.  

#### 3.2.2 Collection Metadata

The metadata associated with collections includes data about the collection that has been returned. 

Metadata related to [field_sets](#field_sets) and [contexts](#contexts) along with any custom metadata about the resource collection may also be included. 

|Property|Required|Description
|-----|-----|-----
|validation_response|yes|This is an object that contains two required properties: *code* and *message*.
|validation_information|no|This is an array of strings that provide information about errors correlated to the validation_response.code and HTTP response code.
|cache|required if the result is a cached value|This is an object that contains one required property: *date_time*. The date_time value is when the data was updated in the cache.
|collection_size|required|The number of items of the resource which exist in the entire collection.
|default\_page\_size|required if API implements support for paging|Unless overridden in query parameters, this endpoint will return this number of resources in each request.
|max\_page\_size|required if API implements support for paging|The largest number of resources this API can return in one request.
|page_size|required if the API implements support for paging|The number of resources which were returned in this request.
|page_start|required if the API implements support for paging|The resource of the collection on which the returned "page" of the collection starts
|page_end|required if the API implements support for paging|The resource of the collection on which the returned "page" of the collection ends

The metadata returned for a resource collection that supports paging would look like:

```
"metadata": {
  "validation_response": {
    "code": 200,
    "message": "Success"
  },
  "collection_size": 150,
  "default_page_size": 50,
  "max_page_size": 1000,
  "page_size": 50,
  "page_start": 51,
  "page_end": 100
 }
```

See [paging](#paging) for more information about implementing paging in an API. 

#### 3.2.3 Values Array
The `values` array contains an entry for each individual resource representation. The representations must follow the specification in [3.1](#3.1) including containing the `links` and `metadata` properties. 

#### 3.2.4 Empty Collections
If the collection returned is empty (e.g. no resources matched a filter) the collection metadata should still be returned. Paging data should still be valid - `page_start` should be `0`, `page_end` should also be `0` since no values were returned. The `values` array should be empty. 

<a name=hateoas></a> 
## 4.0 HATEOAS Links
Hypertext As The Engine Of Application State (HATEOAS) is one of the fundamental design considerations of a REST based API. HATEOAS uses hypertext links to provide the consumers of an API with information regarding the next allowable operations the consumer is allowed to perform. 

### 4.1 Links

HATEOAS links are intended to convey to the consumer the next possible actions that can be performed on a resource or collection of resources. In the case of a collection of resources the `links` property may include links for retrieving the first, next, and last resources in the collection and adding a new value into the collection along with paging through the collection. For a single resource the `links` property may include links to modify or delete the resource along with a link to retrieve this specific resource (useful if the resource is returned as part of a collection). Links should only be provided if the consumer is allowed to perform the specific action represented by the link. For example if the consumer is allowed to modify but not delete the current person resource the `persons__modify` link will be present in the `links` property while the `persons__delete` will not. This way the consumer does not need to understand the business logic behind authorization, it can assume an operation is possible by the presence of the corresponding link. 

### 4.2 Link Format

The UAPI standard varies from traditional HATEOAS implementations in order to make things easier for consumers to utilize the `links` property. HATEOAS links in the UAPI have the following format:

```
 "persons__info": {
                "rel": "self",
                "href": "https://api.byu.edu/byuapi/persons/123456789",
                "method": "GET"
            },
```

The name each of link object and the `rel` property follows the defined pattern `resource-name__business-action`.   Business actions should convey operations that make sense within the business domain. Consumers can use the name of the links to determine which operations can be performed on the resource. 

The properties of the link are defined as follows:  

|Property|Description
|-----|-----
|rel|The `rel` property defines the relationship the link has relative to the resource. The UAPI specification requires the value of `rel` either be `self` or match the name of the HATEOAS link object name. `self` is a special value for `rel` that indicates this link can be used to retrieve the current resource. 
|href|The href contains the URL used to perform the HATEOAS link action (this may also be a templated URL according to RFC 6570).
|method|The method contains the HTTP method used when invoking the URL in the href property.

The UAPI specification requires a `self` link be included. This link should be the link that can be used to retrieve this specific resource (or collection). For a top-level resource it should be named `resource-name__info`. If the link is for a portion of the result (such as a sub-resource) it should be named `sub-resource-name__info`.

An example links object for the person resource at `https://api.byu.edu/byuapi/persons/123456789` would look something like:

          "links": {
            "basic__info": {
                "rel": "self",
                "href": "https://api.byu.edu/byuapi/persons/123456789",
                "method": "GET"
            },
            "basic__modify": {
                "rel": "self",
                "href": "https://api.byu.edu/byuapi/persons/123456789",
                "method": "PUT"
            },
            "basic__delete": {
                "rel": "self",
                "href": "https://api.byu.edu/byuapi/persons/123456789",
                "method": "DELETE"
            }


<a name=field_sets></a> 
## 5.0 Sub-resources, Field\_sets, and Contexts

Access to individual sub-resources is accomplished by adding the desired sub-resource name and an optional identifier to the URL of the top level resource. There are cases where access to more than one sub-resource in a single request is necessary or desirable. The UAPI standard provides two query string parameters to make accessing multiple sub-resources in one call possible: `field\_sets` and `contexts`.

<a name=field_sets></a> 
### 5.1 Field_sets 

The `field_sets` query string parameter contains a comma separated list of the sub-resources the consumer would like to have returned in the response. Since field\_sets are merely a convenient way to request multiple sub-resources the names of field\_sets and sub-resources must be the same. If an API supports sub-resources it must also support field\_sets. 

The API should document the field\_sets supported in the metadata of the top level resource. The metadata properties are:

|Property|Required|Description
|-----|-----|-----
|field\_sets\_returned|only if specific field\_sets have been requested|JSON array of all the field\_sets represented in this result. May be different from the list of field\_sets requested in the query string.
|field\_sets\_available|only if field\_sets are supported|JSON array of all available field\_sets available to request for this resource. A consumer may not have access to all field\_sets in this array.  
|field\_sets\_default|only if field\_sets are supported| The field\_sets that will be returned if no `field_sets` query parameter is specified. 
 
 Example `metadata`  from a top-level resource that supports field\_sets looks like:
 
```
    "metadata": {
        "field_sets_returned": [
            "basic"
        ],
        "field_sets_available": [
            "basic",
            "addresses",
            "email_addresses",
            "languages",
            "phones",
            "relationships"
        ],
        "default_field_sets": [
            "basic"
        ],
    },
```

#### 5.1.1 The 'basic' Field\_set

The UAPI specification defines a special field\_set that doesn't directly correspond to a sub-resource. The  `basic` field\_set is used to represent a set of properties the resource will return when no field\_sets are requested. These properties can include items from sub-resources as well as the top-level resource. If the query string for the request includes the `field_sets` or `contexts` parameter the `basic` field\_set will NOT be included in the result unless it is part of the list of requested field\_sets. 

#### 5.1.2 Field\_set Representation

Sub-resources requested using the `field\_set` query string parameter are represented as JSON objects at the top level of the result. The name of the object is the name of the sub-resource. The representation of each sub-resource is exactly the same as if the sub-resource was accessed directly from the URL. 

A request to `https://api.byu.edu/byuapi/persons/123456789?field_sets=basic,addresses` would look like: 

```
{
    "basic": {
        "links": {
            "basic__info": {
                "rel": "self",
                "href": "https://api.byu.edu/byuapi/persons/123456789",
                "method": "GET"
            },
            "basic__modify": {
                "rel": "self",
                "href": "https://api.byu.edu/byuapi/persons/123456789",
                "method": "PUT"
            },
            "basic__delete": {
                "rel": "self",
                "href": "https://api.byu.edu/byuapi/persons/123456789",
                "method": "DELETE"
            }
        },
        "metadata": {
            "validation_response": {
                "code": 200,
                "message": "Success"
            },
            "validation_information": [
                "No additional information"
            ],
            "cache": {
                "date_time": "2018-02-21T22:26:57.480Z"
            }
        },
        "byu_id": {
            "value": "1234567890",
            "api_type": "system",
            "key": true
        },
        "person_id": {
            "value": "987654321",
            "api_type": "system"
        },
        "net_id": {
            "value": "adddrop",
            "api_type": "related",
            "related_resource": "https://api.byu.edu/byuapi/persons/1234567890/credentials/NET_ID,adddrop"
        },
        "date_time_updated": {
            "value": "2016-09-21T09:03:18.000Z",
            "api_type": "system"
        },
        "date_time_created": {
            "value": "1997-02-07T12:22:32.000Z",
            "api_type": "system"
        },
        "first_name": {
            "value": "John",
            "api_type": "modifiable"
        },
        "middle_name": {
            "value": "D",
            "api_type": "modifiable"
        },
        "surname": {
            "value": "Doe",
            "api_type": "modifiable"
        },
        "rest_of_name": {
            "value": "John D",
            "api_type": "derived"
        },
        "name_lnf": {
            "value": "Doe, John D",
            "api_type": "derived"
        }
    },
    "addresses": {
        "links": {
            "addresses__info": {
                "rel": "self",
                "href": "https://api.byu.edu/byuapi/persons/123456789/addresses",
                "method": "GET"
            }
        },
        "metadata": {
            "collection_size": 2,
            "page_start": 1,
            "page_end": 2,
            "page_size": 2,
            "default_page_size": 1,
            "maximum_page_size": 100,
            "validation_response": {
                "code": 200,
                "message": "Success"
            },
            "validation_information": [
                "No additional information"
            ],
            "cache": {
                "date_time": "2018-02-21T22:26:57.520Z"
            }
        },
        "values": [
            {
                "links": {
                    "addresses__info": {
                        "rel": "self",
                        "href": "https://api.byu.edu/byuapi/persons/123456789/addresses/MAL",
                        "method": "GET"
                    },
                    "addresses__modify": {
                        "rel": "self",
                        "href": "https://api.byu.edu/byuapi/persons/123456789/addresses/MAL",
                        "method": "PUT"
                    },
                    "addresses__delete": {
                        "rel": "self",
                        "href": "https://api.byu.edu/byuapi/persons/123456789/addresses/MAL",
                        "method": "DELETE"
                    }
                },
                "metadata": {
                    "validation_response": {
                        "code": 200,
                        "message": "Success"
                    },
                    "validation_information": [
                        "No additional information"
                    ],
                    "cache": {
                        "date_time": "2018-02-21T22:26:57.518Z"
                    }
                },
                "byu_id": {
                    "value": "123456789",
                    "description": "John Doe",
                    "api_type": "system",
                    "key": true
                },
                "address_type": {
                    "value": "MAL",
                    "api_type": "modifiable",
                    "key": true
                },
                "date_time_updated": {
                    "value": "2012-09-18T09:42:54.000Z",
                    "api_type": "system"
                },
                "date_time_created": {
                    "value": "1997-02-07T00:00:00.000Z",
                    "api_type": "system"
                },
                "address_line_1": {
                    "value": "1300 N University Ave",
                    "api_type": "modifiable"
                },
                "address_line_2": {
                    "value": "PROVO, UT  84602",
                    "api_type": "modifiable"
                },
                "address_line_3": {
                    "value": " ",
                    "api_type": "modifiable"
                },
                "address_line_4": {
                    "value": " ",
                    "api_type": "modifiable"
                },
                "building": {
                    "value": " ",
                    "api_type": "modifiable"
                },
                "room": {
                    "value": " ",
                    "api_type": "modifiable"
                },
                "country_code": {
                    "value": "USA",
                    "description": "United States of America",
                    "api_type": "modifiable"
                },
                "city": {
                    "value": "PROVO",
                    "api_type": "modifiable"
                },
                "state_code": {
                    "value": "UT",
                    "description": "Utah",
                    "api_type": "modifiable"
                },
                "postal_code": {
                    "value": "84602",
                    "api_type": "modifiable"
                }
            },
           {
                "links": {
                    "addresses__info": {
                        "rel": "self",
                        "href": "https://api.byu.edu/byuapi/persons/123456789/addresses/WRK",
                        "method": "GET"
                    },
                    "addresses__modify": {
                        "rel": "self",
                        "href": "https://api.byu.edu/byuapi/persons/123456789/addresses/WRK",
                        "method": "PUT"
                    },
                    "addresses__delete": {
                        "rel": "self",
                        "href": "https://api.byu.edu/byuapi/persons/123456789/addresses/WRK",
                        "method": "DELETE"
                    }
                },
                "metadata": {
                    "validation_response": {
                        "code": 200,
                        "message": "Success"
                    },
                    "validation_information": [
                        "No additional information"
                    ],
                    "cache": {
                        "date_time": "2018-02-21T22:26:57.519Z"
                    }
                },
                "byu_id": {
                    "value": "123456789",
                    "description": "John Doe",
                    "api_type": "system",
                    "key": true
                },
                "address_type": {
                    "value": "WRK",
                    "api_type": "modifiable",
                    "key": true
                },
                "date_time_updated": {
                    "value": "2015-06-09T10:37:00.000Z",
                    "api_type": "system"
                },
                "date_time_created": {
                    "value": "2003-05-06T12:23:14.000Z",
                    "api_type": "system"
                },
                "address_line_1": {
                    "value": "2019 ITB",
                    "api_type": "modifiable"
                },
                "address_line_2": {
                    "value": "Provo, UT  84602",
                    "api_type": "modifiable"
                },
                "address_line_3": {
                    "value": " ",
                    "api_type": "modifiable"
                },
                "address_line_4": {
                    "value": " ",
                    "api_type": "modifiable"
                },
                "building": {
                    "value": "ITB",
                    "description": "Information Tec",
                    "long_description": "Information Technology Bldg",
                    "api_type": "modifiable"
                },
                "room": {
                    "value": "2033",
                    "api_type": "modifiable"
                },
                "country_code": {
                    "value": "USA",
                    "description": "United States of America",
                    "api_type": "modifiable"
                },
                "city": {
                    "value": "Provo",
                    "api_type": "modifiable"
                },
                "state_code": {
                    "value": "UT",
                    "description": "Utah",
                    "api_type": "modifiable"
                },
                "postal_code": {
                    "value": "84602",
                    "api_type": "modifiable"
                }
            }
        ]
    }
}
```

<a name=contexts></a> 
### 5.2 Contexts
Contexts are an optional part of the UAPI specification that allow APIs with a large number of field\_sets to group them together into logical groups. This allows for simplification of the URL query string. Contexts should reflect the business domain related to the top-level resource. Field\_sets can be included in multiple contexts. 

#### 5.2.1 Context Metadata

If a top-level resource supports contexts it should add the `contexts_avaiable` metadata to the field\_set metadata. The value of this property is a JSON object consisting of a JSON array for each of the available contexts. The JSON array will contain the field\_sets included in that context. 

An example of the context metadata is as follows: 

        "contexts_available": {
            "all": [
                "basic",
                "addresses",
                "email_addresses",
                "languages",
                "phones",
                "relationships"
            ],
            "contact": [
                "basic",
                "addresses",
                "email_addresses",
                "phones"
            ],
            "person_bio": [
                "basic",
                "languages"
            ]

#### 5.2.2 Context Query String Parameter

To specify a context or contexts to be returned the `contexts` query string parameter is used to list the contexts being requested. For example, using the above example context definitions, the query string parameter `contexts=contact,person_bio` would return the `basic`, `addresses`, `email_addresses`, `phones`,  and `languages` field\_sets. 

#### 5.2.3 Context Representation

Responses generated by using the `contexts` query string parameter are represented in exactly the same way as if the consumer had requested the included field\_sets using the `field_sets` query string parameter. If the consumer specifies both the `contexts` and `field_sets` query string parameters the result should be a union of the field\_sets requests. A field\_set will only be included once in the result. 

<a name=paging></a> 
## 6.0 Collection Paging

Some resources and sub-resources represent a large number of instances. Returning a large number of entries in a collection can be both resource intensive and time consuming. APIs that support large number of instances should implement collection paging in order to provide the consumer a way to move through the collection in a reasonable way. It may be possible that the underlying data has changed between invocations so instances may be repeated or skipped. 

To implement collection paging a resource must include paging specific `metadata` and `links` with the collection in the response. 

### 6.1 Paging Metadata

The following `metadata` properties are required if collection paging is supported by the resource: 

|Property|Description
|-----|-----
|default\_page\_size|Unless overridden in query parameters, this endpoint will return this number of resources in each request.
|max\_page\_size|The largest number of resources this API can return in one request.
|page\_size|The number of resources which were returned in this request.
|page\_start|The resource of the collection on which the returned "page" of the collection starts
|page\_end|The resource of the collection on which the returned "page" of the collection ends

### 6.2 Paging Query Parameters

In order to implement the collection paging the following query string parameters must be supported:

|Parameter|Description
|-----|-----
|page\_start|The first resource in the collection to be returned with this page
|page\_size|The number of resources in the collection to return (optional)

If the `page_size` parameter is not included in a request the default specified in the `metadata` should be used. 

### 6.3 Paging HATEOAS Links

Additional `links` are included that provide the link to use in order to move back and forth between pages in the collection:

|Link|Content
|-----|-----
|*resource-name*\_\_first|Link to the first page of the collection
|*resource-name*\_\_current|Link to the current page in the collection
|*resource-name*\_\_last|Link to the last page of the collection
|*resource-name*\_\_next|Link to the next page in the collection

An example of the collection paging links for the persons resource would be as follows:

```
        "persons__first": {
            "rel": "persons__first",
            "href": "https://api.byu.edu/byuapi/persons/?page_start=1,page_size=100",
            "method": "GET"
        },
        "persons__last": {
            "rel": "self",
            "href": "https://api.byu.edu/byuapi/persons/?page_start=345600,page_size=100",
            "method": "GET"
        },
        "persons__current": {
            "rel": "self",
            "href": "https://api.byu.edu/byuapi/persons/?page_start=300,page_size=100",
            "method": "GET"
        },
        "persons__next": {
            "rel": "self",
            "href": "https://api.byu.edu/byuapi/persons/?page_start=400,page_size=100",
            "method": "GET"
        },
```


<a name=filters></a> 
## 7.0 Filters
 
 A collection of resources can be filtered using query string parameters. Which of the properties of a resource can be used as filters in the query string is left to the API to define and should follow the business use cases of the top-level resource. For example our `persons` top level resource uses `byu_id` as its primary key. Unfortunately `byu_id` is only one of three keys in common use to access the `persons` resource. If a consumer wants to find a person with a specific `net_id` then a query string parameter of `net_id` should be supported by the API. Likewise it makes sense to allow for filtering on other properties such as `email_address` or `visa_status`.  Information in the Swagger definition for the API should include which properties can be used for filtering. 
 
#### 7.1 Filter Parameters 
 
Filter parameters given in the query string should correspond to properties provided by the resource and should be defined in the Swagger definition of the resource. The `field_sets` and `contexts` query string parameters can be combined with filter parameters to customize the resulting representation. 
 
#### 7.2 Filtering Sub-resources
 
 Filters apply to the lowest level resource specified on the URL. If the URL only specifies a top level resource a collection of top level resources that match the filter will be returned. If the URL includes a sub-resource the filter will be applied to that sub-resource and the collection returned will be instances of the sub-resource. For example, the URL `https://api.byu.edu/byuapi/persons?net_id=johndoe` will return a collection of persons that have a `net_id` of `johndoe`. The URL `https://api.byu.edu/byuapi/persons/123456789/addresses?address_type=MAL` will return a collection of all mailing addresses associated to the person with the byu_id `123456789`.
 
#### 7.3 Dot notation (filtering with sub-resources)

There are times when it makes sense to query a top level resource by a value in one of the sub resources. For example, if I want a list of persons whose home address is in a certain zip code. Resources can support this type of query by implementing dot (`.`) notation. Dot notation allows for a filter parameter to specify the relationship between a top level and a sub-resource property. In order to implement the above example the query string would look something like `https://api.byu.edu/byuapi/persons?persons.addresses.address_type=HOM&persons.addresses.zip_code=84604`  would return a collection of persons with a home address in the `84604` zip code.

#### 7.4 Wildcards

Individual resources can choose to support a wildcard in their query string parameters where they make sense. The asterisk (`*`) should be used as the wildcard character for consistency across resources.  
 
 <a name=meta></a> 
## 8.0 Meta URL Namespaces and APIs

Resources often have related datasets that are necessary for a client to properly use the API of that resource. Those datasets may be such things as lists of accepted state and country names and their abbreviations, possible values for etc. The `meta` URL namespace specification help to clarify where these types of data sets should be located. APIs should be provided to access these datasets.

Each meta dataset should have a corresponding API with the following characteristics:

- The API should be located under `https://api.byu.edu/byuapi/meta/<top-level-resource>`.   
- Meta APIs should provide HTTP GET support only. Updating of the values in the dataset represented by the meta API is left to the domain APIs supporting the dataset.   
- Meta API responses do not include `metadata` or `links` sections like regular UAPI top level resource responses. 
- Meta APIs provide two variations of the GET verb support: 
    -  An HTTP GET on the URL  `https://api.byu.edu/byuapi/meta/<top-level-resource>/<meta api>/<key>` will return an individual entry in the dataset as a JSON set of properties. For example, an HTTP GET on the `state_code` dataset addressed at the URL `https://api.byu.edu/byuapi/meta/student/state_code/UT` would return the following:  
    
    ```
    {
        "value": "UT" ,
        "description": "Utah"
    }
    ```  
    
    - An HTTP GET on the URL  `https://api.byu.edu/byuapi/meta/<top-level-resource>/<meta api>` will return a collection of the individual values defined in the previous section represented in a JSON array, e.g. an HTTP GET on `https://api.byu.edu/byuapi/meta/student/state_code` would return the following:  
    
    ```
    {
    "values": [
        {
            "value": "ID" ,
            "description": "Idaho"
        },
        {
            "value": "NV" ,
            "description": "Nevada"
        },
        {
            "value": "UT" ,
            "description": "Utah"
        },
        {
            "value": "VT" ,
            "description": "Vermont"
        },
        ...
    ]
    }
    ```  
    
    **Note:** Meta APIs do not support paging as defined in [6.0 Collection Paging](#paging)


<a name=binary></a> 
## 9.0 Binary Data 

TBD

<a name=post></a> 
## 10.0 HTTP POST, PUT, DELETE
The UAPI spec defines the HTTP PUT and POST verbs (respectively) for modifying and creating new top level and sub-resources. The resource body used in the request is field\_set specific. Generally it is a JSON structure containing all properties as name/value pairs that can be modified for that field\_set.

### 10.1 PUT
The HTTP PUT verb is used to modify a resource. PUTs against the top level resource will update the `basic` field\_set. All other PUTs will be to the URLs of sub-resources associated with the field\_set. 

For example, to update the `basic` field\_set of the persons resource a PUT is sent to the URL `https://api.byu.edu/byuapi/persons/123456789` with the request body as follows: 

```
{
    "first_name": "Abernathy",
    "high_school_code": "050714",
    "home_country_code": "USA",
    "home_state_code": "UT",
    "home_town": "Provo",
    "middle_name": "Cosmo",
    "preferred_first_name": "Cosmo",
    "preferred_surname": "Cougar",
    "restricted": false,
    "sex": "M",
    "suffix": "II",
    "surname": "Cougar-Rodriguez"
}                            
```

To update the work mailing address of a person resource a PUT is sent to the URL `https://api.byu.edu/byuapi/persons/123456789/addresses/WRK` with the request body as follows:

```{
    "address_line_1": "1234 Milky Way",
    "address_line_2": "Highland, UT 84003",
    "address_line_3": " ",
    "address_line_4": " ",
    "building": " ",
    "city": "Highland",
    "country_code": "USA",
    "postal_code": "84003",
    "room": " ",
    "state_code": "UT",
    "unlisted": false,
    "verified_flag": true
}          
```

### 10.2 POST

The HTTP POST verb is used to insert a new resource or sub-resource. Due to the complexities of creating a new resource the UAPI spec does not specify a format or restrictions on the request body. When a new resource is created an HTTP Status Code of `201` should be returned along with the `Location` HTTP header containing the URL for the new resource. 

### 10.3 DELETE

The UAPI spec also defines the HTTP DELETE verb for deleting top level and sub-resources. If the DELETE is successful an HTTP Status Code of `204` should be returned. 

### 10.4 Return representation 

Upon successful completion of a PUT or POST the return representation should be the resource that was updated in the standard UAPI format. 

If errors in the request prevent the POST or PUT from happening the appropriate HTTP Status Code should be returned along with a minimal UAPI standard result representation. Specifically, the `metadata` properties `validation_response` and `validation_information` should be used to convey additional information to the client about the errors that occurred. Notice `validation_information` is an array in order for information about all errors that occurred when processing the request can be returned to the client. 

For example, the following response could be returned when there are multiple validation errors.

```
"metadata": {
    "validation_response": {
            "code": 400,
            "message": "Bad Request"
            },
    "validation_information": [
         "Invalid home_country_code",
         "Invalid home_state_code",
         "first_name required"
         ]
    },
```

 For more information about handling errors see [Errors](#errors)

<a name=authorization></a> 
## 11.0 Authorization
Authorization for access to to UAPIs falls back to the authorization schemes that individual domains employ to protect their resources from unauthorized access. UAPI resources should gain their identity information about the client and end user making the request from the JSON Web Token (JWT) contained in the `x-jwt-assertion` HTTP header and should pass this header and value to any domain APIs the resource uses to process the request. The UAPI resource should also validate the JWT signature and expiration properties. 

Authorization failures should be communicated via standard HTTP Status Codes, typically using the `403` status code. 

It is highly recommended that data level authorization be handled at the field\_set level. 

Basic unit of authorization is the field\_set? 


<a name=errors></a> 
## 12.0 Errors 


------
------
------
## Appendix A -  Domain APIs and the UAPI Standard
### URLs
#### Namespaces
The namespace designated for Domain APIs is:

```
https://api.byu.edu/domains/{domain_name}
```

The *domain_name* portion of the Domain API namespace is determined by the domain team. For example, if a domain team had chosen the domain_name "identity," its namespace would be:

```
https://api.byu.edu/domains/identity
```


<a name=glossary></a> 
## Glossary

|Term|Definition
|-----|-----
|consumer|
|API|
|domain| 
|mime-type|
|JSON|
|Contract|A published definition of a resource and its associated representation and URL design. Contracts only change when a new major version of an API is published.
|HATEOAS|[Hypermedia As The Engine Of Application State](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_1_5). A series of links associated with a resource that indicate the next permissible actions that can be performed on that resource. 
|HTTP|Hypertext Transfer Protocol](https://tools.ietf.org/html/rfc2616)|
|Property|Key/value pairs defined in a representation.**fix this**
|Representation|The structure of a domain-model object in the context of a specified mime-type. A representation is what is returned in response to a request or what is submitted in the body of a request.
|Resource|A domain-model object that has a structural representation and set of allowable HTTP methods.
|REST|[Representational State Transfer](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm)
|URL|[Uniform Resource Locator](https://tools.ietf.org/html/rfc1738)

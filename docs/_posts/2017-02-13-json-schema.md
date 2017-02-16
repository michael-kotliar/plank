---
layout: post
title: "JSON Schema"
---

JSON Schema is a powerful tool for validating the structure of JSON data. In practice, these schemas can be used to create validators, code generators and other useful tools to handle complex and/or tedious issues. There is a great online overview of JSON-Schema and its specifics here: [http://spacetelescope.github.io/understanding-json-schema/#](http://spacetelescope.github.io/understanding-json-schema/#). For the purposes of this document we will only be concerned with version 4 of JSON-Schema.

## JSON Schema Basics

Here is a simple schema and overview of the fields listed.

    {
        "id": "schemas/user.json",
        "title": "user",
        "description" : "Schema definition of a User",
        "$schema": "http://json-schema.org/schema#",
        "type": "object",
        "properties": { "id" : { "type": "string" },},
        "required": [] 
    }
| id (string)                      | The id property identifies where this resource can be found. This can either be a relative or absolute path. In addition, schemas that are accessed remotely can be accessed by specifying the correct URI. This value will become more important when we discuss JSON Pointers below. |
| -------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| title (string)                   | Title is used to identify the name of the object. The convention we use is all lowercase with underscores (“_”) to separate words (i.e. “offer_summary”).                                                                                                                              |
| description (string)             | Description is a helpful place to specify more detail about the current model object or property.                                                                                                                                                                                      |
| $schema (string, URI formatted)  | This is a URI to the json-schema version this document is based on. This will be the default schema URI for now: "[http://json-schema.org/schema#](http://json-schema.org/schema#)"                                                                                                    |
| type (string)                    | Specifies the type, currently this is always “object” when declared outside of the properties map. Valid types are “string”, “boolean”, “null”, “number”, “integer”, “array”, “object”.                                                                                                |
| properties (map<string, object>) | Properties are where most of your editing will be focused. This area allows us to specify the property names (as the key) as well as their expected type.                                                                                                                              |
| required                         | List of property names that are required to be present in the JSON response. This is currently unused but eventually could be utilized to provide tighter validation of schema responses.                                                                                              |


**Property fields** 
Properties are where most of your editing will be focused. This area allows us to specify the fields that are available on this model. The properties declaration is a map from the property name to a simple schema that describes the property. 

The keys follow the same naming conventions as title field (lowercase, underscore separated) and should map directly to the key that will be used in the JSON response. The value will be an object that can be one of the types specified earlier or a reference to another JSON-schema file (via JSON Pointer `$ref` ). 

In addition, there is syntax for providing concrete subtypes such as dates, URIs, and emails as shown below. A full list can be seen under the JSON-Schema type-specific documentation [here](http://spacetelescope.github.io/understanding-json-schema/reference/type.html).
Type-specific examples

| String Property                     | `"about" : { "type" : "string" },`                                                                                                                                                                                                                                                               |
| ----------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| String Enum                         | ```
{
   …
  "email_interval" : {
    "type" : "string",
    "enum": [
        { "default" : "unset", "description" : "unset" },
        { "default" : "immediate", "description" : "immediate" },
        { "default" : "daily", "description" : "daily" }
    ],
    "default" : "unset"
}
``` |
| Boolean Property                    | `"blocked_by_me" : { "type" : "boolean" },`                                                                                                                                                                                                                                                      |
| Integer Property                    | `"board_count" : { "type" : "integer" },`                                                                                                                                                                                                                                                        |
| Integer Enum                        | "in_stock" : {
            "type": "integer",
            "enum": [
                { "default" : -1, "description" : "unknown" },
                { "default" : 0, "description" : "out_of_stock" },
                { "default" : 1, "description" : "in_stock" }
            ]
        },     |
| Date-time Property (String variant) | "created_at" : { "type" : "string" , "format" : "date-time"},                                                                                                                                                                                                                                    |
| Email Property (String variant)     | "email" : { "type" : "string" , "format" : "email"},                                                                                                                                                                                                                                             |
| URI Property (String variant)       | "image_large_url" : { "type" : "string", "format": "uri" },                                                                                                                                                                                                                                      |
| JSON Pointer Property ($ref)        | "verified_identity" : { "$ref" : "verified_identity.json" },                                                                                                                                                                                                                                     |
| Array Property                      | "pin_thumbnail_urls" : {
            "type": "array",
         }                                                                                                                                                                                                                                 |
| Array Property with Item types      | "pin_thumbnail_urls" : {
            "type": "array",
            "items": {
                 "type": "string",
                 "format": "uri"
             }
         }                                                                                                                       |
| Object Property                     | "some_map" : {
            "type": "object"
         }                                                                                                                                                                                                                                           |
| Object Property with Item types     | "some_map" : {
            "type": "object",
            “additionalProperties”: { $ref : “user.json” }
         }                                                                                                                                                                               |


**JSON Pointers**

****Most of these property declarations should be straightforward to understand with the exception of JSON Pointer. This is a specific syntax that is used to reference the location of other JSON files. 
The key for a JSON pointer is “$ref” and the value is a path relative to the base location which was specified by the “id” key.

Here’s an example of how the pointers destination is resolved.

1. The schema declares an **id** property:
2. "id": "[http://foo.bar/schemas/address.json](http://foo.bar/schemas/address.json)"
3. There is a property defined with a JSON pointer as its value.
4. “some_property_name” : { "$ref": "person.json" }
5. When the pointers destination is resolved, it will be:
6. http://foo.bar/schemas/person.json
# Java callout for JSONPath

This directory contains the Java source code required to compile a Java
callout for Apigee Edge that does JSONPath. There's a built-in ExtractVariables policy that
can evaluate json-path; this callout is a bit more flexible.

* the source can be a string or a message
* The jsonpath processor is more current


## Disclaimer

This example is not an official Google product, nor is it part of an official Google product.

## Using this policy

You need to build the source code in order to use the policy in Apigee Edge, in order to download all the dependencies.

The instructions to do so are at the bottom of this README.

After you build it,

1. copy the jar file, available in  target/edge-callout-jsonpath-20191216.jar , if you have built the jar, or in [the repo](bundle/apiproxy/resources/java/edge-callout-jsonpath-20191216.jar) if you have not, to your apiproxy/resources/java directory. Also copy all the required dependencies. (See below) You can do this offline, or using the graphical Proxy Editor in the Apigee Edge Admin Portal.

2. include a Java callout policy in your
   apiproxy/resources/policies directory. It should look
   like this:
   ```xml
   <JavaCallout name="Java-JSON-Path-Multiple-Fields">
     <Properties>
       <Property name='jsonpath'>$[*]['id','name']</Property>
       <Property name='source'>contrivedMessage.content</Property>
     </Properties>
     <ClassName>com.google.apigee.edgecallouts.jsonpath.JsonPathCallout</ClassName>
     <ResourceURL>java://edge-callout-jsonpath-20191216.jar</ResourceURL>
   </JavaCallout>
   ```

5. use the Edge UI, or a command-line tool like [importAndDeploy.js](https://github.com/DinoChiesa/apigee-edge-js/blob/master/examples/importAndDeploy.js) or similar to
   import the proxy into an Edge organization, and then deploy the proxy .

6. use a client to generate and send http requests to tickle the proxy.



## Notes

There is one callout class, com.google.apigee.edgecallouts.jsonpath.JsonPathCallout ,
which performs a JSON Path read . This class depends on [the jayway jsonpath
library for Java](https://github.com/json-path/JsonPath), v2.4.0

You must configure the callout with Property elements in the policy
configuration.

| Property           | Description                                                                                                         |
|--------------------|---------------------------------------------------------------------------------------------------------------------|
| jsonpath             | required. a string representing the query.                                                                        |
| source               | optional. name of a string variable that contains json, or name of a Message that has a json payload.            |
| return-first-element | optional. when the jsonpath returns a list, this property gets the first element of that list. (See notes below) |


Regarding `return-first-element`, the jsonpath spec allows for predicates, but
does not allow for indexers following predicates.  Given this source json:
```json
{
  "records" : [
  {
    "type" : "A",
    "value" : 1000
  },
  {
    "type" : "A",
    "value" : 8000
  },
  {
    "type" : "B",
    "value" : 900
  }
 ]
}
```

A json path of `$.records[?(@.type=='A')]` yields:
```json
[
   {
      "type" : "A",
      "value" : 1000
   },
   {
      "type" : "A",
      "value" : 8000
   }
]
```

This is an array. One might thing that appending a `[0]` to the above jsonpath
would yield the first element of that array, but [that is not correct](https://github.com/json-path/JsonPath/issues/272).

The `return-first-element` property allows you to retrieve just the first
element of the returned array.  If you need the 2nd element or ... something
else, then... you need to evaluate a 2nd jsonpath query.


## Building

Building from source requires Java 1.8, and Maven.

1. unpack (if you can read this, you've already done that).

2. Before building _the first time_, configure the build on your machine by loading the Apigee jars into your local cache:
  ```
  ./buildsetup.sh
  ```

3. Build with maven.
  ```
  mvn clean package
  ```
  This will build the jar and also run all the tests.


Pull requests are welcomed!


## Build Dependencies

- Apigee Edge expressions v1.0
- Apigee Edge message-flow v1.0
- jayway json-path 2.4.0
- json-smart 2.3


## License

This material is Copyright (c) 2019, Google LLC.  and is licensed under
the [Apache 2.0 License](LICENSE). This includes the Java code as well
as the API Proxy configuration.


## Support

This callout is open-source software, and is not a supported part of Apigee Edge.
If you need assistance, you can try inquiring on
[The Apigee Community Site](https://community.apigee.com).  There is no service-level
guarantee for responses to inquiries regarding this callout.


## Bugs

* The tests are incomplete.


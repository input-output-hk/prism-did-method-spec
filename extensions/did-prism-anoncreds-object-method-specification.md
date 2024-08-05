# did:prism AnonCreds Method v1.0

## Objective of this specification

The primary goal of this document is to describe a way to identify and retrieve resources using the [PRISM DID method](https://github.com/input-output-hk/prism-did-method-spec/blob/main/w3c-spec/PRISM-method.md) as a building block.
As a secondary set of goals we want our approach to be re-usable by other DID methods that share some basic functionalities. Finally, we will present a component we identify as useful in the interoperability of [AnonCred Methods](https://hyperledger.github.io/anoncreds-methods-registry/).

## Definitions

In this section we will present a way to classify the resources we want to treat, and then define the properties we want to satisfy with our AnonCred method.

Within the context of SSI, different resources are involved in the creation, sharing and verification of [Verifiable Credentials (VCs)](https://www.w3.org/TR/vc-data-model-2.0/) (including [AnonCreds](https://hyperledger.github.io/anoncreds-spec/)). Some of these resources can be described as _static_ due to the fact that they do not change once associated to a VC. Examples of these resources are schema objects and credential definitions. Conversely, other resources could be described as _dynamic_ in nature, as VCs refer to them, but the resources per se are updated over time. Examples of these resources are status lists or revocation entries. This classification will help us later in this document to describe how to identify, retrieve and validate resources in accordance to this specification.

As a whole, this AnonCred method aims to provide:
- Integrity assurance for static resources: When a user retrieves a resource based on its identifier, he can validate on his own that the resource hasn't been tampered with.
- Storage layer extensibility: after resource creation, the user controlling the resource identifier can update the storage location and/or retrieval protocols without affecting the original resource identifier.

### Underlying DID method properties

As we mentioned in the objectives section, we will define the AnonCred method in terms of the PRISM DID method. However, the AnonCred method could be constructed on top of any [DID method](https://www.w3.org/TR/did-core/) that simply supports for [services](https://www.w3.org/TR/did-core/#services).

In the remaining of this document we will take the following conventions:

- For each resource, the user has an intended URL where the resource should be retrieved. We will call this URL `baseURL`.
- Each resource has an associated DID, `userDID`, that is used to create the resource identifier.
- The `userDID` has a service we will call `baseService`.
  + The `baseService` has `baseURL` as its only service endpoint.
  + The `baseService` has type `LinkedResourceV1`.
- The resource has an optional associated path `resourcePath` relative to its `baseURL`.

As an illustrative example we can show the following DID document where:
- `userDID` is `did:prism:123...abc`
- `baseService` is `did:prism:123...abc#service1`
- `baseURL` is `https://example.com`

Based on those values, we can present the following DID document:

```json
{
  "id" : "did:prism:123...abc",
  ...
  "service" : [
    { 
      "id" : "did:prism:123...abc#service1",
      "type : "LinkedResourceV1",
      "serviceEndpoint" : "https://example.com"
    }
  ]
}
```

## Identifiers

In this section we will define the identifiers structure for any given resource. 
We will start describing a basic identifier where each resource uses its own service, and later explain how to optionally share the same service between multiple resources.
We will also distinguish the identifier construction for static and dynamic resources.

### Dynamic resources

For dynamic resources such as revocation lists, we define the identifiers in this AnonCred method as follows. 
Given a resource `r` with associated `userDID` and base service `baseService`, we can define `r`'s identifier as follows:

```
  userDID ? resourceService= baseService
```


### Static resources

For static resources, like a schema or a credential definition, we don't usually have an integrity check at application layer. For this reason, we add to identifiers a `resourceHash` section that will serve for this purpose. 
Hence, given a resource `r` with associated `userDID` and base service `baseService`, we can define `r`'s identifier as follows:

```
  userDID ? resourceService= baseService & resourceHash = encoded_hash(r)
```

where `encoded_hash` is the hex encoded SHA256 hash of the base64URL encoded resource `r`.

#### A comment about resource authenticity

We want to remark that, in practice, dynamic resources in SSI are signed by a key associated to the issuer of an associated VC. For this reason, this AnonCred method does not add any integrity nor authenticity check for dynamic resources, and leaves it to the application layer. In this way, we avoid an overhead for a second authenticity/integrity check at this stage.

On a related note, we want to point out that we are also not performing an authenticity checks for static resources inside the AnonCred method. The reason is that in the context of VCs, the identifiers we are generating are attached in signed objects (the VCs). By attaching an integrity check to the identifier, we are inheriting the authenticity of the expected resource from the signature on the VC that contained the identifier in the first place. Future versions of this AnonCred method could add more flexibility by embedding a signature in the resource envelope (see below). However, we argue that the relevant authenticity property is attached to the authenticity of the identifier, and not the authenticity of the indirect resource it represents (see Future Work section).

### Sharing a common `baseURL`

There are situations where a user controlling multiple resources would prefer to share the same service to support many of those resources. The advantage is to require less services in the `userDID` document, where the trade off is that updating the service for one resource will imply updating the location of all the resources. We consider that this trade off is use case dependent, and we added the following structure to support both scenarios.

If a user wants to share the same `resourceService` and consequently `baseURL` for multiple resources, they have the option to distinguish specific resources by relative path from the `baseURL`. The relative path is specified with the `resourcePath` query parameter. As an example, if we imagine resources `r1` and `r2` located at `https://example.com/resources/r1.txt` and `https://example.com/resources/r2.txt` respectively. We can use the `baseURL` `https://example.com` and relative paths `/resource/r1.txt` and `/resource/r2.txt` respectively to obtain the following identifiers.

```
  userDID ?resourceService= resourceService & resourcePath= /resource/r1.txt
```
and 

```
  userDID ?resourceService= resourceService & resourcePath= /resource/r2.txt
```

If the resources require a hash, the corresponding query parameter would also be included. Namely:

```
  userDID ?resourceService= resourceService & resourcePath= /resource/r2.txt & resourceHash= encoded_hash(r2.txt)
```

where `encoded_hash`, once again, represents the hex encoded sha256 hash of the base64URL encodedresource.

## Resources retrieval

In this section, we will describe how to obtain a resource from a given identifier in this AnonCred method.
We start by briefly displaying some example identifiers using the DIDs from the PRISM DID method.

```
// Example 1
did:prism:abc...123?resourceService="service1"&resourcePath="/schemas/123.json"&resourceHash="hffa...31aef3"

// Example 2
did:prism:abc...123?resourceService="service1"&resourcePath="/schemas/123.json"

//Example 3: long form DID with all parameters
did:prism:abc...123:abfha...1d4s8fr?resourceService="service1"&resourcePath="/schemas/123.json"&resourceHash="hffa...31aef3"
```

An implementation of this specification will behave as follows:
- First, extract the PRISM DID from the URI. In examples 1 and 2, this is `did:prism:abc...123`, while in example 4, this is `did:prism:abc...123:abfha...1d4s8fr`
- Then, resolve the extracted DID from the previous step.
- The resolution will return a DID document. The implementation should validate that:
  - Given the value of `resourceService` parameter, lets call it `serviceId`. There must be a service with name `<prism-did>#serviceId` in the resolved DID document. In examples 1 and 2, this is a service with id equal to `did:prism:abc...123#service1`. See example DID document at the end of this section.
  - the identified service type must be `LinkedResourceV1`.
- The `serviceEndpoint` of the service previously identified will contain a URL, lets call it `baseURL` in the next steps.
- If present, attach to `baseURL` the value in `resourcePath`. For the examples shown above, this would lead to a URL `baseURL/schemas/123.json`
- Perform a GET call to that URL to obtain the resource, expecting a JSON response.
  The resource will be encoded as a base64url string inside a JSON object of the form: 

  ```
   { 
      “resource“ : <base64url string> 
   }
  ```
- if the initial identifier contains a `resourceHash` query parameter, validate that the hex encoded sha256 hash of the string located in the `“response“` field of the JSON, matches the string in the `resourceHash` value.
- If all the validations in the previous steps were correct, then return the JSON that it received from the server that generated the JSON response.

Example DID document to illustrate an expected service:

```
  {
    "id": "did:prism:abc...123",
    ...
    "service" : [
      {
        "id" : "did:prism:abc...123#service1",
        "type": "LinkedResourceV1",
        "serviceEndpoint" : baseUrl
      }
    ]
  }
```

## Universal AnonCred method resolver

We want to dedicate a section to discuss an additional component, which is not exactly part of this AnonCred Method, but sits on top of it.

Different methods in the AnonCreds method registry have described custom formats to represent resources when they are queried. Some methods return the resource as a string in a verbatim form. Other methods, including this one, create an envelope JSON object that contains the resources under some encoding (Base58, Base64URL, etc). From an application point of view, this format discrepancies hinders interoperability. We would like to describe here a simple layer that most application will have to implement in one form or another. We can call this component AnonCred method universal resolver. The sole purpose of this layer is to receive a URI that identifies a resources, and return the string resource without any envelope. The input URI can represent an identifier from any AnonCred method. The universal resolver would determine from the URI which specific AnonCred method it should call, and after receiving the response, it would do any needed validation (e.g. integrity checks), and remove the potential specific envelope that any method may be adding.

We want to remark that URIs would need to contain enough information to distinguish which AnonCred method needs to be called. We could suggest to have in the URIs a reserved query parameter to identify the method (see Future Work section). 

## Future Work

This section is kept to list possible extensions for this AnonCred method

### Add a `resourceMethod` query parameter

As mentioned in the previous section, we have observed the lack of clear ways to identify how to resolve a resource based solely on an identifier. Several AnonCred methods use a DID URL as resource identifier. However, they all follow slightly different resource resolution steps. We consider useful, if not actually needed, to add a `resourceMethod` query parameter to identify the AnonCred method that the identifier is representing. This allows a clear mechanism to take an identifier and determine how to treat it. It is also a good tool to identify a version of the AnonCred method the identifier is associated with. In the future of this specification, we may add new features where version names like `DIDURLsV1`, `DIDURLsV2`, etc. may become handy.

### More flexible `serviceEndpoint` values

In this first conservative version, our method allows for a single URL for `serviceEndpoint` in the associated `resourceService`. However, one could see many alternatives to this:
- Allow a list to represent a fallback semantics.
- Allow other URIs for resources. For example, IPFS identifiers for static resources.
- Allow a map to represent richer data. For example, a way to specify holder to verifier direct transmission of a resource.

### More flexible authenticity checks

In order to add another layer for authenticity checks, the resource identifier would need to add a reference to a key, while the resource envelope would need a signature field.
For example, the resource identifier could have query parameters `signingKey` and `signingAlgorithm`. Where `signingKey` could be either an encoded key or a DID URL that refers to a verification method. Key encodings and signing algorithms should be standardize. The JSON response that serves as resource envelope would add an adequate `proof` field containing necessary signature information.

In that way, the user would be able to validate expected source for the resource. We want to note that the information related to the signing key is added in the identifier, in order to avoid the server side of the resolution process to tamper the key. However, we also need to remark that the resource identifier itself is susceptible to be modified, meaning that a user trying to dereference an identifier needs to trust the identifier's source in the first place. This implies that the authenticity of the resource is strongly tight to the authenticity of the identifier, and this is why we consider that supporting an integrity check between an static resource and its identifier is enough for this AnonCred method. Adding the authenticity at resource level does not solve the more relevant identifier authenticity, which is the likely intended property to preserve. 

For the case of dynamic resources, we remind the reader that existing SSI protocols already enforce a signature checks at application layer, but we would find it adequate to have an optional authenticity check for the case of dynamic resources in the future version of this AnonCred method.

### Passing query parameters to the underlying server

For the case of dynamic resources, some use cases could benefit from the addition of historical versions of a resource. If the underlying server hosting the resources keeps their historical changes and exposes them through query parameters in an API, we find it relevant to support in future versions of this method a mechanism to forward that query to the server. We point out, that this additional query string is part of the identifier per se, as each user may decide to specify different parameters. What would be needed are two main parts:
- An standard query string to send to an underlying server. For instance, `resourceVersion`.
- A way to indicate this AnonCred method that the query parameter should be attached to the final URL before making the resource request.


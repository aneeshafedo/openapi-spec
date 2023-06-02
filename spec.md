# Specification: Ballerina OpenAPI Tool

_Owners_:  
_Reviewers_:  
_Created_: 
_Updated_:   
_Edition_: Swan Lake


## Introduction

This is the specification for the Ballerina OpenAPI Tool of the [Ballerina language](https://ballerina.io/), which generates a Ballerina service or a client from an OpenAPI contract and vice versa.

The Open API Tool specification has evolved and may continue to evolve in the future. The released versions of the specification can be found under the relevant GitHub tag. 

If you have any feedback or suggestions about the tool, start a discussion via a [GitHub issue](https://github.com/ballerina-platform/ballerina-standard-library/issues) or in the [Discord server](https://discord.gg/ballerinalang). Based on the outcome of the discussion, the specification and implementation can be updated. Community feedback is always welcome. Any accepted proposal, which affects the specification is stored under `/docs/proposals`. Proposals under discussion can be found with the label `type/proposal` in GitHub.

The conforming implementation of the specification is released and included in the distribution. Any deviation from the specification is considered a bug.

## Contents


## 1. Overview

This module provides the Ballerina OpenAPI tooling, which will make it easy to start the development of a service documented in an OpenAPI contract in Ballerina by generating the Ballerina service and client skeletons.

The OpenAPI tools provide the following capabilities.

1. Generate the Ballerina service or client code for a given OpenAPI definition.
2. Export the OpenAPI definition of a Ballerina service.
3. Validate the service implementation of a given OpenAPI contract.

The openapi command in Ballerina is used for OpenAPI to Ballerina and Ballerina to OpenAPI code generations. Code generation from OpenAPI to Ballerina can produce ballerina service stubs and ballerina client stubs. The OpenAPI compiler plugin will allow you to validate a service implementation against an OpenAPI contract during the compile time. This plugin ensures that the implementation of a service does not deviate from its OpenAPI contract.


## 2. OpenAPI to Ballerina

Generate Service and Client Stub from an OpenAPI Contract

```
bal openapi -i <openapi-contract-path> 
               [--tags: tags list]
               [--operations: operationsID list]
               [--mode service|client ]
               [(-o|--output): output file path]
```

Generates both the Ballerina service and Ballerina client stubs for a given OpenAPI file.

This `-i <openapi-contract-path>` parameter of the command is mandatory. It will get the path to the OpenAPI contract file (i.e., `my-api.yaml` or `my-api.json`) as an input.

You can give the specific tags and operations that you need to document as services without documenting all the operations using these optional `--tags` and `--operations` commands.

The `(-o|--output)` is an optional parameter. You can use this to give the output path of the generated files. If not, it will take the execution path as the output path.


**Modes**

If you want to generate a service only, you can set the mode as `service` in the OpenAPI tool.

```
bal openapi -i <openapi-contract-path> --mode service [(-o|--output) output file path]
```

If you want to generate a client only, you can set the mode as `client` in the OpenAPI tool. This client can be used in client applications to call the service defined in the OpenAPI file.


### 2.1. Ballerina Client Generation

Generates a Ballerina client from a given OpenAPI file. This include generating 

```
bal openapi -i <openapi-contract-path> --mode client [(-o|--output) output file path]
```

#### 2.1.1. Generated Client Object

By default the generated client name is “Client”. 

##### Client Initialization

The client init method has following parameters

   1. Service URL (`string serviceUrl`)
   - The server URL is identified from the `server` section of the given OpenAPI file. 
   - If there are multiple servers given, the first URL is used as the default service URL. 
   - If the server URL is parameterized, the default value for service URL is constructed using the default value of each variable. 
     - Ex: 
       ```yaml
       servers:
        url: https://{customerId}.saas-app.com:{port}/v2
        variables:
        customerId:
            default: demo
            description: Customer ID assigned by the service provider
        port:
            enum:
            - '443'
            - '8443'
            default: '443'
       ```
       The constructed default service URL for above OpenAPI server definition is `https://demo.saas-app.com:443/v2`

   - Currently tool does not provide the support for [overriding servers](https://swagger.io/docs/specification/api-host-and-base-path/#:~:text=%C2%A0%C2%A0%C2%A0%C2%A0%C2%A0%C2%A0%C2%A0%C2%A0%C2%A0%20%2D%20southeastasia-,Overriding%20Servers,-The%20global%20servers)
   - If there is no servers defined in the OpenAPI file, the parameter `serviceURL` is generated as a required parameter. Otherwise the `serviceURL` parameter is a defaultable parameter. 
  
   2. Client Configuration (`ConnectionConfig config`)
   - `ConnectionConfig` record is based on the `http:ClientConfiguration`. The changes between the `http:ClientConfiguration` and `ConnectionConfig` is as below. 
     - The type of the `auth` record is changed according to the `securitySchemes` defined in the OpenAPI file. 
     - The `ClientHttp1Settings` record is generated within the connector initializing default values for each fields. 

Within the client init method, using the parameters provided by the user, an http:Client object is initialized to use across the client's resource/remote functions. 

<TODO: Do we need to give an init sample code ???>


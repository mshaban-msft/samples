# Track-II SDK Development (Onboarding)
This document gives you detailed explanation on how to develop SDKs for Azure services (or any web service).

## Contents

### Development Pipeline
SDK development typically goes through the following stages
- [Service Release](#service-release)
- [Understanding the Service](#understanding-the-service)
- [Design Phase (LLC or conveniece layer)](#design-phase)
- [Client generation](#client-generation)
- Samples
- Testing
- Final touches
    - Updating main readme, samples readme, release changelog
- User studies

### Resources
- [Azure-Core library](#azure-core-library)
- [Coding Guidelines](#coding-guidelines)
- [Internal Tutorials](#internal-tutorials)
- [Repositories](#repositories)

### Language-specific Instructions
- [Python-specific instructions](#python-specific-instructions)



## Service Release

At this stage, the service team should release the service swagger definitions to [azure-rest-api-specs](https://github.com/Azure/azure-rest-api-specs) repo, and they should also provide at least a working PPE env for us to use for SDK development.

The [azure-rest-api-specs](https://github.com/Azure/azure-rest-api-specs) repo also has CI checks to make sure swagger files submitted are valid syntactically and complies with OpenApi standards (a requirement for [Autorest]([azure-rest-api-specs](https://github.com/Azure/azure-rest-api-specs)) client generation).

There are practices for preview and GA swagger releases on [azure-rest-api-specs](https://github.com/Azure/azure-rest-api-specs). For example, preview swaggers are released to main branch, while GA swaggers are released to main branch.

One of the issues the CI has at the moment, is that there might be discrepancies between the actual service APIs and the published swagger. The team is currently working on some validations to make sure that never happens.



## Understanding the Service

At this stage, we try to familiarize ourselves with the service swagger and APIs. We usually meet with someone from the service team. They explain how the service works, walk us through newly added features, breaking changes, and provide postman collection we can use for experimenting with the service.

We then start experimenting with the APIs using postman, making sure we fully understand the Ins and Outs od the service.

Usually the service should be stable, having already been tested by the service team through multiple bug bashes.

## Design Phase
At this stage, we decide on how we should proceed with SDK design. we have 2 options to go with
- LLC client
- Client with a conveniece layer

We factor-in things like
- How many APIs are there
- Customization level we need
- Previous user studies (if any)

### LLC client
This stands for Low-Level Client. This SDK is purely generated. We use generation parameters and directives to customize the SDK.

### Client with Conveniece Layer
Sometimes, the client prompts adding a conveniece layer.
This layer is made for more customization/control over the client, and it is built on top of the LLC client.
For example, we might want to add our own simplified methods and parameters, or add some handling for LROs.
For this type of clients, we start with the design on a platform like [apiview.dev](https://apiview.dev/), and then we go with multiple rounds of design/review with the SDK archboard until we get a final design.


## Client Generation
At this stage, we generate LLC client using the service swagger definitions using the [Autorest](https://github.com/Azure/autorest) tool.

### Generation Schema (swagger.md)
In order to generate LLC client for any service in any programming language, we need a schema to perfectly describe to Autorest what generation options to use, in addition to directives to customize our client.

The whole point of this file is to get the same result when generating client by anyone, and on any platform.

You need to understand you can't make changes to generated client code, because regeneration overrides previous changes.
- [Generation parameters](#generation-parameters)
- [Generation directives](#generation-directives)
- [Code customization](#generated-code-customization-patch-file)

#### Generation Parameters

There's an article describing how to use Autorest to generate client from swagger, starting with simple CLI parameters, all the way up to grouping generation options into single unified file (swagger.md).
Read article [here](https://github.com/Azure/autorest/blob/main/docs/generate/readme.md). 

#### Generation Directives
When generating client, we sometimes need to do additional customizations ranging from fixing swagger and generation errors, renaming methods or models, to deleting unwanted or redundant apis.
Read articles [here](https://github.com/Azure/autorest/blob/main/docs/generate/directives.md), and [here](https://github.com/Azure/autorest/blob/main/docs/extensions/readme.md).

#### Generated Code Customization (_patch file)
In order to further customize your client, you can actually override generated code using special _patch file.
Read article [here](https://github.com/Azure/autorest.python/blob/autorestv3/docs/customizations.md).  

## Testing
At this stage, we write tests to make sure our client and methods are working perfectly. 

### secrets management. (tests.yml file)
we can run tests in 2 modes: live and recorded modes.
In live mode, you actually make calls to api endpoint. But in recorded mode, python simply mock network traffic.
We have recorded mode because when submitting a PR or pushing new changes, CI/CD pipeline will only run tests in recorded mode.

## Resources

### Azure-Core library
You need to familiarize yourself with the underlying library for all SDKs, which is Azure-Core. You need to be familiar with concepts like Pipelines, LROs, Paging, .. etc.

- Azure-Core .Net: https://docs.microsoft.com/en-us/dotnet/api/overview/azure/core-readme

- Azure-Core Python: https://docs.microsoft.com/en-us/python/api/overview/azure/core-readme?view=azure-python

### Coding Guidelines
- coding guidelines: https://azure.github.io/azure-sdk/general_introduction.html

### Internal Tutorials
- Internal Tutorials:           https://dev.azure.com/azure-sdk/internal/_wiki/wikis/internal.wiki/346/Azure-SDK-Development-Training
- Azure SDK Internal Training:  https://github.com/Azure/azure-sdk-pr/tree/master/training   

### Repositories
- Azure-api-specs: https://github.com/Azure/azure-rest-api-specs
- Python SDKs repository: https://github.com/Azure/azure-sdk-for-python/
- .NET SDKs repository: https://github.com/Azure/azure-sdk-for-net/    
- Autorest repository: https://www.npmjs.com/package/autorest


## Language-specific Instructions

### Python-specific Instructions

#### Prepare development environment
Create conained environment
```bash
python -m venv "path/to/venv"
```

Install development dependencies
```bash
cd "path/to/sdk"
pip install -r dev_requirements.txt
```

Install client as package (to run samples and tests)
```bash
cd "path/to/sdk"
pip install -e .
```

#### Running Tests
We can run tests in 2 modes: live and recorded modes. By default, tests run in recorded mode. You need to add the following file to run tests in live mode 
```yml
- directory: "azure-sdk-for-python/tools/azure-sdk-tools/devtools_testutils"
- file-name: testsettings_local.cfg
- file-content: "live-mode: true"
```


## Ramp-up tasks
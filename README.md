# MAS AH Integration ([Arrowhead Framework 4.3.0](https://github.com/arrowhead-f/core-java-spring#arrowhead-framework-430))

## Table of Contents
- [Introduction](#introduction)
- [MAS Integration](#mas-integration)
- [MAS - Translator Integration](#mas-translator-integration)
- [MAS - IGS Integration](#mas-igs-integration)
- [Scenarios](#scenarios)

## Introduction

This project is the continuance of the [Mismatch Analysis System](https://github.com/fernand0labra/mismatch-analysis-system) or MAS. The system was not integrated as a support member within the framework, instead it was built as a provider system when interacting with the core components. 

The extension includes its integration with the framework as well as the use of two support systems, the Translator and the Interface Generator System (IGS) for the handling of mismatches in the service definitions of the service contracts. The Orchestrator code has been edited for handling requests of consumer systems that have different protocol, encoding or semantics than the provided services.

The results show that it is possible to perform an exchange of information between two systems that do not share the same service definition by the use of other support systems that act as a middleware layer. However, improvements can be made for reducing waiting times in the MAS and IGS.

## MAS Integration

The integration has been performed by allowing the consumer systems to indicate with a flag if an analysis on the service contracts should be run. When the flag is up, the Orchestrator requests the analysis to the MAS and responds with a flag displaying the different possibilities (OK, NOT_OK, ALTER_T or ALTER_G). For the first two flags there is no use of any support system as it would mean respectively that the communication can be performed or that there is no possible solution to address the mismatches.

When the flag received is ALTER_T, the Orchestrator builds a request object and sends it to the Translator that in turn would generate a translation hub and return its address. The Orchestration result object is edited so that the address is from the hub.

When the flag received is ALTER_G, the Orchestrator builds a request object and sends it to the IGS. This one would generate a jar and execute it in parallel for handling the consuming requests and return its address to the Orchestrator that would update the orchestration result responded to the consumer.

## MAS - Translator Integration ([Link to Video](https://youtu.be/aTH2UHaNN6Y))

Due to the use of the AH version 4.3.0, the Translator code was not complete and did not follow the compliant structure of the AH support systems. Moreover, some issues regarding query and protocol support as well as url building led to the editing of its code so that it would correctly work when called. The orchestrator was edited for performing Translator requests and handling its responses.

<p align="center">
  <img src="https://user-images.githubusercontent.com/70638694/178143445-9b7b5f93-bdaa-403c-8587-9b9ed32b3ad7.png"/>
</p>

The previous figure displays the different stages performed during the exchange of communciation between a consumer and a provider:
1. Consumer requests service consumption (with an MAS flag).
2. Orchestrator requests SC analysis.
3. MAS returns an **ALTER_T** flag.
4. Orchestrator requests hub creation to the Translator and receives endpoint.
5. Orchestrator returns orchestration result to the consumer updated with the hub's endpoint.
6. Consumer requests service consumption (COAP).
7. Translator hub requests service consumption (HTTP).
8. Provider returns response (HTTP).
9. Translator hub returns response (COAP).

## MAS - IGS Integration ([Link to Video](https://youtu.be/UbYFUnEFCoc))

The Interface Generator System was still at an early stage of coding structure. Due to this factor, cleaning and project reshaping was performed in order to easily understand its code and follow the compliant structure of an AH system. The cleaning included mostly absolute path transformation and naming conventions while the structure reshaping included the generation of code within the project for reducing the complexity of the process. As with the Translator, the Orchestrator was edited for performing requests and handling its responses.

<p align="center">
  <img src="https://user-images.githubusercontent.com/70638694/178144248-8e70d1fd-cc60-4778-86a2-0d2187b46fd8.png"/>
</p>

The previous figure displays the different stages performed during the exchange of communciation between a consumer and a provider:
1. Consumer requests service consumption (with an MAS flag).
2. Orchestrator requests SC analysis.
3. MAS returns an **ALTER_G** flag.
4. Orchestrator requests interface generation to the IGS.
5. The IGS compiles the new interface, executes a server on a parallel thread and returns endpoint.
6. Orchestrator returns orchestration result to the consumer updated with the generated interface endpoint.
7. Consumer requests service consumption (HTTP/JSON).
8. The generated interface requests service consumption (COAP/XML).
9. Provider returns response (COAP/XML).
10. The generated interface returns response (HTTP/JSON).

## Scenarios

There are two tested scenarios for consumer and provider systems that effectively allow an exchange of information during runtime:
* The first one emulates a difference in the protocol of the communication, where the consumer performs a request using COAP while the provider expects an HTTP request. The MAS after analysing the service contracts of both systems, responds the Orchestrator with an ALTER_T flag what sequentially makes it call the Translator. This one would create a translation hub and return its address, which in turn would be the one received by the consumer. 
  - **Consumer**: COAP
  - **Provider**: HTTP
  
* The second one emulates a difference in the protocol and the encoding of the communication. The consumer system performs a request with HTTP and JSON while the provider expects COAP and XML. The MAS again analyses both service contracts, but this time responds an ALTER_G flag making the Orchestrator call the IGS. This one would dynamically create the consumer code for a correct request and return an address to the executing created server that would handle the communication.
  - **Consumer**: HTTP/JSON
  - **Provider**: COAP/XML

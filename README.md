# Product Hub – GraphQL Contract Testing with Specmatic

This repository demonstrates **GraphQL contract testing and service virtualization using Specmatic.**

The project follows a **schema-first approach**, where a GraphQL SDL is used as the single source of truth to generate a stub server with deterministic responses.
It showcases how consumers can validate their integrations against a virtualized GraphQL API without relying on a live backend.

All commands and examples were executed using **Windows PowerShell**, but the same concepts apply to any OS.

---

## Why This Project Exists
This project aims to demonstrate:
* How to virtualize a GraphQL API using contracts
* How to define deterministic responses using Specmatic examples
* How to validate consumer behavior using headers, variables, and query shapes
* How to run GraphQL contract stubs locally and in CI pipelines

This setup is especially useful for:
* Early consumer development
* Integration testing
* Contract-driven development (CDD)
* CI smoke validation without backend dependencies

---

## Prerequisites

- Docker installed and running
- Git (optional, for cloning the repository)

---

## Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/BernardoSJ/product-graphql-specmatic.git
```

### 2. Navigate to the project folder and start the GraphQL stub

```bash
cd product-graphql-specmatic

docker run --rm -v "${PWD}\contracts:/sandbox" -p 9000:9000 specmatic/specmatic-graphql virtualize /sandbox/product-api.graphql
```

This command will:

* Load the GraphQL SDL (```product-api.graphql```)
* Read all example files from ```product-api_examples```
* Start a GraphQL stub server on port **9000**
* Expose the ```/graphql``` endpoint for POST requests
 💡 To use a different port, map it as ```PORT:9000```.

In case you want to validate the one with specmatic.yaml in the root folder use this command:

```bash
docker run -v "$PWD/product-api-v2.graphql:/usr/src/app/product-api-v2.graphql" -v "$PWD/examples:/usr/src/app/examples" -v "$PWD/specmatic.yaml:/usr/src/app/specmatic.yaml" -p 9000:9000 specmatic/specmatic-graphql virtualize --port=9000 --examples=examples
```

## Verify the Stub is Running (PowerShell)

Use ```Invoke-RestMethod``` to send a GraphQL POST request: 

```bash
$payload = @{
  query = 'query { findAvailableProducts(type: gadget, pageSize: 10) { id name inventory type } }'
} | ConvertTo-Json -Compress

$resp = Invoke-RestMethod -Method Post `
  -Uri "http://localhost:9000/graphql" `
  -ContentType "application/json" `
  -Body $payload

$resp.data.findAvailableProducts | Format-List *
```
You should receive a **deterministic response** based on the defined Specmatic examples.

## Verify Using curl (CMD)

If you prefer using ```curl``` from **Command Prompt (CMD)**: 

```bash
curl -X POST http://localhost:9000/graphql ^
  -H "Content-Type: application/json" ^
  -d "{\"query\":\"query { findAvailableProducts(type: gadget, pageSize: 10) { id name inventory type } }\"}"
```

In case for the example in the root folder please use this command:

```bash
curl -X POST http://localhost:9094/graphql ^
     -H "Content-Type: application/json"  ^
     -d "{\"query\":\"query { findAvailableProductsV2(type: gadget, pageSize: 10) { id name } }\"}"
```

## Requests with Header-Based Matching

Specmatic can return different responses depending on HTTP headers.

```bash
$payload = @{
  query = 'query { findProductById(id: "P-300") { id name inventory type } }'
} | ConvertTo-Json -Compress

$resp = Invoke-RestMethod -Method Post `
  -Uri "http://localhost:9000/graphql" `
  -Headers @{ "X-Tenant" = "demo" } `
  -ContentType "application/json" `
  -Body $payload

$resp.data.findProductById | Format-List *
```

This demonstrates **header-based contract matching.**

## Requests Using GraphQL Variables

GraphQL queries with variables are also supported:

```bash
$payload = @{
  query = 'query($id: ID!) { findProductById(id: $id) { id name inventory type } }'
  variables = @{ id = "P-300" }
} | ConvertTo-Json -Compress

$resp = Invoke-RestMethod -Method Post `
  -Uri "http://localhost:9000/graphql" `
  -Headers @{ "X-Tenant" = "demo" } `
  -ContentType "application/json" `
  -Body $payload

$resp.data.findProductById | Format-List *
```
Although the client uses variables, Specmatic matches the request against example-based contracts.

## Partial Field Selection

GraphQL field selection is flexible and does not break contract matching:

```bash
$payload = @{
  query = 'query { findAvailableProducts(type: gadget, pageSize: 50) { id name } }'
} | ConvertTo-Json -Compress

$resp = Invoke-RestMethod -Method Post `
  -Uri "http://localhost:9000/graphql" `
  -ContentType "application/json" `
  -Body $payload

$resp.data.findAvailableProducts | Format-List *

```

## Delayed Responses (Latency Simulation)

```bash
$payload = @{ query = 'query { findAvailableProducts(type: gadget, pageSize: 5) { id name inventory type } }' } | ConvertTo-Json -Compress

Measure-Command {
  Invoke-RestMethod -Method Post `
    -Uri "http://localhost:9000/graphql" `
    -ContentType "application/json" `
    -Body $payload
}

```
This request will intentionally delay the response (e.g. 3 seconds) as defined in the example file.

## Error Handling Strategy

Due to current limitations in the ```specmatic/specmatic-graphql``` container:

* Native GraphQL ```errors {}``` blocks are **not emitted**
* Negative scenarios are modeled using **schema-compliant sentinel responses** (e.g. ```NOT_FOUND``` objects)
This allows consumers to test failure paths while remaining contract-compliant.

---

## GitHub Actions - CI Integration
This repository includes a **GitHub Actions workflow** that runs a smoke validation against the GraphQL stub.

**Workflow details:**

* Name: **Specmatic GraphQL Stub Smoke**
* Triggers:
   * push
   * pull_request
   * workflow_dispatch

**Artifacts generated:**

* Stub server logs
* JSON responses from smoke requests

---

## Project Structure

```text
product-graphql-specmatic
 ├──contracts/
 ├──product-api.graphql
 ├──product-api_examples/
     ├──findAvailableProducts.yaml
     ├──findProductById.yaml
     ├──findProductById_with_headers.yaml
```

## Key Concepts Demonstrated
* GraphQL service virtualization
* Schema-first contract testing
* Deterministic stub responses
* Header-based matching
* Variable-based GraphQL queries
* Latency simulation
* Consumer-side CI validation

## Scope and Limitations
This project focuses on **stub-based contract testing** for consumers.
* Provider-side contract verification is out of scope
* Real backend integration is intentionally excluded
* Designed for learning, demos, and portfolio usage

Provider verification can be added as a future enhancement.
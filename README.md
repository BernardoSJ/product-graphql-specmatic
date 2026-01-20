# Product Hub – GraphQL Contract Testing with Specmatic

This project is an example of **contract testing using GraphQL and Specmatic**.  
It demonstrates how to virtualize a GraphQL API using a schema-first approach and how to test different request scenarios such as headers, variables, and deterministic responses.

All commands and examples were created and executed using **Windows PowerShell**.

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

This command:

* Loads the GraphQL SDL (product-api.graphql)
* Starts a stub server on port 9000
* Automatically uses the examples located in product-api_examples

## Verify the Stub is Running (PowerShell)

Use ```text Invoke-RestMethod``` to send a GraphQL POST request: 

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
You should receive a deterministic response based on the defined Specmatic examples.

## Verify Using curl (CMD)

If you prefer using ```text curl``` from **Command Prompt (CMD)**: 

```bash
curl -X POST http://localhost:9000/graphql ^
  -H "Content-Type: application/json" ^
  -d "{\"query\":\"query { findAvailableProducts(type: gadget, pageSize: 10) { id name inventory type } }\"}"
```

## Performing Requests with Headers

This example demonstrates how Specmatic can differentiate responses based on HTTP headers.

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

## Performing Requests with Variables

This example shows how GraphQL requests using variables are handled by the stub:

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
Even though the client sends variables, Specmatic matches the request against examples defined with inline values.

## Performing Multi Field Selection
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
* GraphQL service virtualization using Specmatic
* Schema-first contract testing
* Deterministic responses using examples
* Header-based request matching
* Variable-based GraphQL queries

## Notes
This project focuses on stub-based contract testing and virtualization.
Provider-side contract tests can be added as a next step.
# Product Hub GraphQL + Specmatic

This project contains a example of contract testing implementing GraphQL + Specmatic. All the project was created with powershell. In order to see this working you need to follow the next steps:

Preconditions:
* Have Docker installed within your computer

1. **Clone or download the repository:**
   ```bash
   git clone https://github.com/BernardoSJ/product-graphql-specmatic.git
   ```
2. **Navigate to the project folder and start the stub with Docker:**
   ```bash
   cd gadgethub-contract-testing-project
   
   docker run --rm -v "${PWD}\contracts:/sandbox" -p 9000:9000 specmatic/specmatic-graphql virtualize /sandbox/product-api.graphql
   ```
3. **Make sure the stub is executing with the Invoke-RestMethod**
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
3.1 **Execute a call using curl command withing CMD**
    ```bash
    curl -X POST http://localhost:9090/graphql -H "Content-Type: application/json" -d "{ \"query\": \"query { findAvailableProducts(type: gadget, pageSize: 10) { id name inventory type } }\" }"
    ```


## Perform requests with Headers
   ```bash
   $payload = @{ query = 'query { findProductById(id: "P-300") { id name inventory type } }' } | ConvertTo-Json -Compress

   $resp = Invoke-RestMethod -Method Post `
   -Uri "http://localhost:9000/graphql" `
   -Headers @{ "X-Tenant" = "demo" } `
   -ContentType "application/json" `
   -Body $payload

   $resp.data.findProductById | Format-List *
   ```

## Perform requests with Variables
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
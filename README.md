# Cloud Pricing API

This project aims to create a GraphQL cloud pricing API. Currently supports AWS, Google (GCP) and Azure; future support for other cloud vendors is planned.

Infracost has a hosted version of this service. To get an API key [download infracost](https://www.infracost.io/docs/#quick-start) and run `infracost register`.

## Table of Contents

* [Example requests](#example-requests)
* [Getting Started](#getting-started)
  * [Prerequisites](#prerequisites)
  * [Installation](#installation)
* [Usage](#usage)
  * [Running](#running)
* [License](#license)

## Example requests

Get all t3.micro prices in US East, this returns 30+ results. Try it yourself by pasting the following query into [https://pricing.api.infracost.io/graphql](https://pricing.api.infracost.io/graphql) - you'll need to use something like the [modheader](https://bewisse.com/modheader/) browser extension so you can set the custom HTTP header `x-api-key` to your Infracost API key.

```graphql
query {
  products(
    filter: {
      vendorName: "aws",
      service: "AmazonEC2",
      productFamily: "Compute Instance",
      region: "us-east-1",
      attributeFilters: [
        { key: "instanceType", value: "t3.micro" }
      ]
    },
  ) {
    attributes { key, value }
    prices { USD }
  }
}
```

Get the hourly on-demand price of a Linux EC2 t3.micro instance in US East:

Request:

```graphql
query {
  products(
    filter: {
      vendorName: "aws",
      service: "AmazonEC2",
      productFamily: "Compute Instance",
      region: "us-east-1",
      attributeFilters: [
        { key: "instanceType", value: "t3.micro" },
        { key: "tenancy", value: "Shared" },
        { key: "operatingSystem", value: "Linux" },
        { key: "capacitystatus", value: "Used" },
        { key: "preInstalledSw", value: "NA" }
      ]
    },
  ) {
    prices(
      filter: {
        purchaseOption: "on_demand"
      },
    ) { USD }
  }
}
```

Response:

```json
{
  "products": [
    {
      "pricing": [
        {
          "USD": "0.0104000000"
        }
      ]
    }
  ]
}
```

> More examples can be found in `./examples/queries`

## Getting started

### Prerequisites

 * Node.js >= 14.15.0
 * MongoDB >= 4.4

### Installation

1. Clone the repo

  ```sh
  git clone https://github.com/infracost/cloud-pricing-api.git
  cd cloud-pricing-api
  ```

2. Add a `.env` file to point to your MongoDB server, e.g.

  ```
  MONGODB_URI=mongodb://localhost:27017/cloudPricing

  # If using GCP
  GCP_PROJECT=
  GCP_API_KEY=<GCP API Key> # An API key with access to the GCP Cloud Billing API
  GCP_KEY_FILE=gcp_creds.json # Credentials for a service account that has read-only access to Compute Engine.
  ```

3. Install the npm packages

  ```sh
  npm install
  ```

4. Run npm build

  ```sh
  npm run-script build
  ```

5. Update the pricing data
   **Note: this downloads several GB of data**

  ```sh
  npm run update
  ```

  If you only want to download from a single source you can run:

  `Example: AWS Bulk`

  ```sh
  npm run update:dev -- --only=aws:bulk
  ```

  `Example: Azure Retail`
  
  ```sh
  npm run update -- --only=azure:retail
  ```

## Usage

### Running locally

```
npm run dev
```

### Running in production

```
npm run-script build
npm run start
```

You can now access the GraphQL Playground at [http://localhost:4000/graphql](http://localhost:4000/graphql).

## License

[Apache License 2.0](https://choosealicense.com/licenses/apache-2.0/)

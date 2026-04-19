---
title: API Access
---

# API Access

Pixee Enterprise Server provides a REST API for programmatic access to your Pixee installation. This page explains how to authenticate with the API and access available resources.

<Note>
The current API authentication uses a global system API key. Improved authentication mechanisms are planned for future releases.
</Note>

## Retrieving Your API Key

To access the Pixee API, you need to retrieve your API key from the admin console.

### Step 1: Access the Admin Console

Navigate to your admin console:

- **Embedded Cluster**: `https://<your-domain>:30000`
- **Helm Deployment**: Access via the configured admin console endpoint

Log in with your admin credentials.

### Step 2: Enable and Retrieve the API Key

1. Select the **Config** tab in the admin console
2. Scroll to the **Basic Settings** section
3. Check the **Enable Pixee API key** checkbox to enable API authentication
4. Copy the value from the **Pixee API Key** field

[![Pixee API Key configuration in admin console]({{asset "assets/api/pixee-api-key-config.png"}})]({{asset "assets/api/pixee-api-key-config.png"}})

<Tip>
Store your API key securely. Treat it like a password and avoid committing it to version control or sharing it in insecure channels.
</Tip>

## Authentication

The Pixee API uses Bearer token authentication. Include your API key in the `Authorization` header of each request.

### Example: cURL Request

<CommandBlock>
curl -H "Authorization: Bearer <your-api-key>" \
  https://<your-pixee-server>/api/v1/openapi
</CommandBlock>

### Example: Python Request

<CommandBlock>
import requests

headers = {
    "Authorization": "Bearer <your-api-key>"
}

response = requests.get(
    "https://<your-pixee-server>/api/v1/openapi",
    headers=headers
)
</CommandBlock>

## API Resources

Pixee Enterprise Server provides two resources for exploring and integrating with the API:

### HAL Browser

**URL**: `/api/browser`

The HAL Browser provides an interactive web interface for exploring the API with real data from your Pixee installation. Use it to:

- Discover available API endpoints
- Understand the structure of API responses
- Test API calls interactively

### OpenAPI Specification

**URL**: `/api/v1/openapi`

The OpenAPI specification provides a machine-readable description of the API. Use it to:

- Generate client code in your preferred programming language
- Import into API testing tools like Postman or Insomnia
- Build integrations with automated tooling

To download the specification:

<CommandBlock>
curl -H "Authorization: Bearer <your-api-key>" \
  https://<your-pixee-server>/api/v1/openapi > openapi.json
</CommandBlock>

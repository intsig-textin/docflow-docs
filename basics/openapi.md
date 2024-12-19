---
icon: network-wired
---

# OpenAPI

You can sync GitBook pages with an OpenAPI or Swagger file or a URL to include auto-generated API methods in your documentation.

### OpenAPI block

GitBook's OpenAPI block is powered by [Scalar](https://scalar.com/), so you can test your APIs directly from your docs.

{% swagger src="openapi.yaml" path="/api/app-api/sip/platform/v2/files/upload" method="post" %}
[openapi.yaml](openapi.yaml)
{% endswagger %}

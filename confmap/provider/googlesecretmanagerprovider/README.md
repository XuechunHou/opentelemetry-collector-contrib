# Google Secrets Provider
<!-- status autogenerated section -->
| Status        |           |
| ------------- |-----------|
| Stability     | [alpha]  |
| Distributions | [contrib] |
| Issues        | [![Open issues](https://img.shields.io/github/issues-search/open-telemetry/opentelemetry-collector-contrib?query=is%3Aissue%20is%3Aopen%20label%3Aprovider%2Fgooglesecretmanagerprovider%20&label=open&color=orange&logo=opentelemetry)](https://github.com/open-telemetry/opentelemetry-collector-contrib/issues?q=is%3Aopen+is%3Aissue+label%3Aprovider%2Fgooglesecretmanagerprovider) [![Closed issues](https://img.shields.io/github/issues-search/open-telemetry/opentelemetry-collector-contrib?query=is%3Aissue%20is%3Aclosed%20label%3Aprovider%2Fgooglesecretmanagerprovider%20&label=closed&color=blue&logo=opentelemetry)](https://github.com/open-telemetry/opentelemetry-collector-contrib/issues?q=is%3Aclosed+is%3Aissue+label%3Aprovider%2Fgooglesecretmanagerprovider) |
| Code coverage | [![codecov](https://codecov.io/github/open-telemetry/opentelemetry-collector-contrib/graph/main/badge.svg?component=provider_googlesecretmanager)](https://app.codecov.io/gh/open-telemetry/opentelemetry-collector-contrib/tree/main/?components%5B0%5D=provider_googlesecretmanager&displayType=list) |
| [Code Owners](https://github.com/open-telemetry/opentelemetry-collector-contrib/blob/main/CONTRIBUTING.md#becoming-a-code-owner)    | [@aabmass](https://www.github.com/aabmass), [@dashpole](https://www.github.com/dashpole), [@jsuereth](https://www.github.com/jsuereth), [@psx95](https://www.github.com/psx95), [@braydonk](https://www.github.com/braydonk), [@ridwanmsharif](https://www.github.com/ridwanmsharif) |

[alpha]: https://github.com/open-telemetry/opentelemetry-collector/blob/main/docs/component-stability.md#alpha
[contrib]: https://github.com/open-telemetry/opentelemetry-collector-releases/tree/main/distributions/otelcol-contrib
<!-- end autogenerated section -->

## Summary

This Provider component offers a secure way to reference secrets or sensitive information in collector configurations using [Google Secret Manager](https://cloud.google.com/security/products/secret-manager). Use a placeholder in the format `${googlesecretmanagerprovider:projects/<project Id>/secrets/<secret Id>/versions/<version Id>}` within your configuration. The actual secrets will then be fetched dynamically from [Google Secret Manager](https://cloud.google.com/security/products/secret-manager) during collector initialization.
## Usage

- Simply replace plaintext secrets within your collector configuration with the placeholder: `${googlesecretmanagerprovider:projects/<project Id>/secrets/<secret Id>/versions/<version Id>}`

An example collector configuration:

```
receivers:
  otlp:
    protocols:
      grpc:
      http:
processors:
  batch:

exporters:
  logging:
    loglevel: debug
  http:
    endpoint: "https://example.com/api/metrics"
    headers:
      X-API-Key: ${googlesecretmanagerprovider:projects/12345/secrets/my-secret/versions/1}
service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [batch]
      exporters: [logging, http]
    metrics:
      receivers: [otlp]
      processors: [batch]
      exporters: [logging, http]
    logs:
      receivers: [otlp]
      processors: [batch]
      exporters: [logging, http]

```

### Prerequisites
1. Make sure to enable access to the [Secret Manager API](https://cloud.google.com/secret-manager/docs/accessing-the-api).
2. Make sure to [add the secret entries to Google Secret Manager](https://cloud.google.com/secret-manager/docs/create-secret-quickstart) before referencing them in the collector configurations. 
3. This Provider interacts with Google Secret Manager using the Secret Manager client library. This library uses [Application Default Credentials (ADC)](https://cloud.google.com/docs/authentication/application-default-credentials) to locate authentication credentials for Secret Manager. Therefore, if you run your collector in a local environment, execute the [`gcloud auth application-default login`](https://cloud.google.com/secret-manager/docs/authentication#client-libs) command to generate the necessary credential file to provide to ADC.
4. However, if your collector runs on Google Compute Engine (GCE) or Google Kubernetes Engine (GKE), running `gcloud auth application-default login` is optional. This is because ADC can retrieve credentials via [the metadata server](https://cloud.google.com/docs/authentication/application-default-credentials#order). However, ensure that your GKE or GCE instance [has enabled the cloud-platform OAuth scope](https://cloud.google.com/secret-manager/docs/accessing-the-api#oauth-scopes). Additionally, verify that the Service Account attached to the GCE or GKE instance has been granted at least the [roles/secretmanager.secretAccessor](https://cloud.google.com/secret-manager/docs/access-control#secret-manager-roles) IAM role to access secret entries in Google Secret Manager.


---
title: References
permalink: /v1/reference/
redirect_from: /reference/
toc: true
---

## HTML Reference
[API v1 Reference](https://gateway.production.netfoundry.io/rest/v1/docs/index.html){: .btn .btn--info .btn--x-large}
[API v2 Reference](https://gateway.production.netfoundry.io/core/v2/docs/index.html){: .btn .btn--info .btn--x-large}

## OpenAPI

OpenAPI is a specification for describing a REST API as a data structure and is represented as JSON or YAML. With this you could generate a client for your preferred programming language or import it as a collection of requests in [Postman](https://learning.postman.com/docs/postman/collections/working-with-openAPI/).

<!-- [JSON](/assets/openapi-3.0.json){: .btn .btn--info .btn--x-large} &nbsp; -->
[JSON](https://gateway.production.netfoundry.io/rest/v1/docs/openapi-3.0.json){: .btn .btn--info .btn--x-large} &nbsp;

[YAML](https://gateway.production.netfoundry.io/rest/v1/docs/openapi-3.0.yml){: .btn .btn--info .btn--x-large}

### Client Generators
[OpenAPITools Generator](https://openapi-generator.tech/){: .btn .btn--info .btn--x-large} &nbsp;
[Swagger Codegen](https://swagger.io/tools/swagger-codegen/){: .btn .btn--info .btn--x-large} &nbsp;

## Quick Reference

### Life Cycle Status Codes

The numeric value appears in the attribute `status` for many types of resources.

100
: NEW

200
: PROVISIONING

300
: PROVISIONED

400
: REGISTERED

500
: ERROR

600
: UPDATING

700
: REPLACING

800
: DELETING

900
: DELETED

### Endpoint Availability Status Codes

The numeric value of `status` indicates the realtime availability of an endpoint.

100
: REGISTERING

200
: OFFLINE

300
: ONLINE

### NetFoundry Authorization Session Status Codes

The numeric value of `status` indicates the realtime session state (active login to Console) of a user.

100
: INACTIVE

200
: ACTIVE

### Endpoint Types

CL
: Non-Ziti Client Endpoint

ZTCL
: Ziti Client Endpoint

ZTGW
: NF-Hosted Ziti Bridge Gateway. Ziti Client Endpoints require one separate "Ziti bridge gateway" for each AppWAN.

ZTNHGW
: Customer-hosted Ziti Bridge Gateway. Ziti Client Endpoints require one separate "Ziti bridge gateway" for each AppWAN.

GW
: NF-Hosted Cloud Gateway in AWS

AWSCPEGW
: Customer-hosted Cloud Gateway in AWS launched from the AWS Marketplace

```bash
# look up the latest version of the marketplace offer
❯ aws --region us-east-1 \
    ec2 describe-images \
      --owners aws-marketplace \
      --filters "Name=product-code,Values=a4h9d0h56e7x0q6hlbii33kl8" \
      --query 'sort_by(Images, &CreationDate)[-1]'
```

VCPEGW
: Generic cloud gateway. Use this for on-premises hypervisors e.g. VMWare. (Virtual Customer Provided Equipment Gateway)

AVWGW
: Azure Virtual WAN Branch Gateway. on-premises gateway that provides a branch-to-cloud on-demand IPSec via Azure Virtual WAN.

AZSGW
: Cloud Gateway from the Azure Stack Marketplace offer.

AZCPEGW
: Cloud Gateway from the Azure Marketplace offer.

HAGW
: Logical pair of cloud gateways in a primary, backup configuration.

GCPCPEGW
: Cloud Gateway from the Google Compute Platform marketplace offer.

ALICPEGW
: Cloud Gateway from the Alibaba marketplace offer.

## Experimental HTML References
[OpenAPITools HTML2](/v1/reference/html2/){: .btn .btn--info .btn--x-large} &nbsp;
[Spectacle](/v1/reference/spectacle/){: .btn .btn--info .btn--x-large} &nbsp;
[ReDoc](/v1/reference/redoc/){: .btn .btn--info .btn--x-large} &nbsp;

<!--innocuous commit-->

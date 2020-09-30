---
permalink: /v1/guides/rapidapi/
title: Hello, RapidAPI
sidebar:
    nav: v1guides
toc: true
classes: wide
---

<script type="text/javascript" src="/assets/scripts/gifffer.min.js"></script>
<script>
    window.onload = function() {
    Gifffer();
    }
</script>

## RapidAPI Vs Direct

You may subscribe to the NetFoundry API through RapidAPI. The HTTP requests you send via RapidAPI are the same as you would send directly to NetFoundry, but the URL and headers are different. This replaces the client code, authentication token, and billing relationship you would otherwise have with NetFoundry; and enables
1. lower or no-cost entry to building NetFoundry AppWANs, and
2. building integrations by accessing multiple RapidAPI marketplace offers with the same authentication token.

## RapidAPI Marketplace

RapidAPI provides code samples for many programming languages for all of the APIs, such as the NetFoundry API, in their marketplace. Some developers find this is an accelerated path to leveraging just one or several of the numerous available APIs in their own application.

**It's easier than ever to get an AppWAN up and running!**
{: .text-center}

<img class="align-center" data-gifffer="/assets/images/LetsGoExcited.gif" data-gifffer-alt="Best Day Ever"/>

## Subscribe Now

You may click below to connect through RapidAPI to NetFoundry's API and paste code snippets they provide in your IDE to start building right away. After you subscribe through RapidAPI you'll use [RapidAPI's guides](https://docs.rapidapi.com/) to authenticate and wrap your HTTP requests for the NetFoundry API with the appropriate RapidAPI headers. Beyond that you'll use the same concepts and order of operations to manage your AppWANs as described in [Hello, AppWAN](/v1/guides/hello-appwan/).

As a RapidAPI subscriber you will not need a separate API account from NetFoundry.
{: .notice--success}

[![RapidAPI Marketplace](/assets/images/connect-on-rapidapi.png)](https://rapidapi.com/netfoundryinc-netfoundryinc-default/api/netfoundry-programmable-zero-trust-connectivity)

## How to Build an AppWAN with RapidAPI

The result of these examples is functioning AppWAN which publishes an HTTP weather service to your client. All of the components except the client are public and hosted by NetFoundry. You could self-host any or all of these components to achieve other goals such as making your server invisible to the internet.

You may safely ignore caveats in info cards such as this. These are slightly more advanced alternatives and details you may find helpful the second time through when you are beginning to customize your AppWAN topology.
{: .notice--success}

### Workspace

These examples make use of three tools that are available for all major OSs:
1. [HTTPie (command-line HTTP client)](https://httpie.org/) and
1. [`jq` (command-line JSON processor)](https://stedolan.github.io/jq/) running in
1. [BASH](https://www.gnu.org/software/bash/).

With these installed and your RapidAPI token assigned to environment variable `RAPID_API_KEY` you are ready to go! You may obtain your token from the code snippets shown in RapidAPI after you subscribe to NetFoundry.

You don't have to use these shell commands if you prefer to work with [another programming language](https://docs.rapidapi.com/docs/getting-started-with-rapidapi-sdks) that is supported by RapidAPI.

### Discover the RapidAPI Network

We'll need this ID later to compose requests.

```bash
❯ http GET https://netfoundry-programmable-zero-trust-connectivity.p.rapidapi.com/networks \
        x-rapidapi-host:netfoundry-programmable-zero-trust-connectivity.p.rapidapi.com \
        x-rapidapi-key:${RAPID_API_KEY} | jq '._embedded.networks[0].id'

"4a566244-d9d6-4a92-b40d-385570cfa3d1"

❯ NF_NETWORK=4a566244-d9d6-4a92-b40d-385570cfa3d1
```

### Discover Georegions

Multi-cloud `GENERIC` regions are preferable for availability and performance unless you have a specific need to bias for one cloud provider.
{: .notice--info}

Choose a region that is near the server you will share with endpoints.

```bash
❯ http GET https://netfoundry-programmable-zero-trust-connectivity.p.rapidapi.com/geoRegions \
    x-rapidapi-host:netfoundry-programmable-zero-trust-connectivity.p.rapidapi.com \
    x-rapidapi-key:${RAPID_API_KEY} | jq '._embedded.geoRegions[11]|{id:.id,name:.name}'
{
  "id": "1d824744-0b38-425a-b1d3-6c1dd69def26",
  "name": "GENERIC Canada East2"
}
❯ NF_SERVICE_REGION=1d824744-0b38-425a-b1d3-6c1dd69def26
```

Choose a region that is near the client with which you will access the service.

```bash
❯ http GET https://netfoundry-programmable-zero-trust-connectivity.p.rapidapi.com/geoRegions \
    x-rapidapi-host:netfoundry-programmable-zero-trust-connectivity.p.rapidapi.com \
    x-rapidapi-key:${RAPID_API_KEY} | jq '._embedded.geoRegions[14]|{id:.id,name:.name}'
{
  "id": "20d5c5b8-0006-43b9-99aa-503fd3931fea",
  "name": "GENERIC US West"
}
❯ NF_CLIENT_REGION=20d5c5b8-0006-43b9-99aa-503fd3931fea
```

### Create Terminating Endpoint

Use the discovered network ID and georegion ID to create the terminating endpoint that will access the server on behalf of your clients and gateways.

```bash
❯ http POST https://netfoundry-programmable-zero-trust-connectivity.p.rapidapi.com/networks/${NF_NETWORK}/endpoints \
    x-rapidapi-host:netfoundry-programmable-zero-trust-connectivity.p.rapidapi.com \
    x-rapidapi-key:${RAPID_API_KEY} \
    name=exampleTerminatingEndpoint \
    geoRegionId=${NF_SERVICE_REGION} \
    endpointType=GW | jq .id
"588bfd0d-f561-4427-bddd-e7aa9de8883d"
❯ NF_TERMINATING_ENDPOINT=588bfd0d-f561-4427-bddd-e7aa9de8883d
```

The terminating endpoint is a "gateway" type of endpoint. Traffic flows through gateways from clients to services. For the sake of simplicity; you could use a public, hosted endpoint as shown in this example; or you could use `endpointType=VCPEGW` and self-host your own terminating endpoint with [the virtual machine images](https://netfoundry.io/resources/support/downloads/networkversion6/#gateways) that we provide. If you self-host then you'll need to log in as `nfadmin` and run the registration command on your VM using the one-time key that is an attribute of your terminating endpoint like `sudo nfnreg {one time key}`. Here's [an article in our Support Hub](https://support.netfoundry.io/hc/en-us/articles/360016129312-Create-a-NetFoundry-Gateway-VM-on-Your-Own-Equipment) about self-hosted gateway registration. First-boot registration is automated for all hosted gateways and for some cloud providers when launching a self-hosted gateway through the web console.
{: .notice--info}

### Create a Service

A service is a description of a server. You could use any server that is reachable by the terminating endpoint. The intercept IP can be the same as the real address or you could use a fictitious intercept. Clients will intercept traffic that matches this address and send it to the real network address.

```bash
❯ http POST https://netfoundry-programmable-zero-trust-connectivity.p.rapidapi.com/networks/${NF_NETWORK}/services \
    x-rapidapi-host:netfoundry-programmable-zero-trust-connectivity.p.rapidapi.com \
    x-rapidapi-key:${RAPID_API_KEY} \
    name=exampleService \
    serviceClass=CS \
    serviceInterceptType=IP \
    serviceType=TCP \
    interceptDnsHostname=wttr.in \
    interceptDnsPort=80 \
    interceptFirstPort=80 \
    interceptLastPort=80 \
    networkIp=5.9.243.187 \
    networkFirstPort=80 \
    networkLastPort=80 \
    interceptIp=5.9.243.187 \
    endpointId=${NF_TERMINATING_ENDPOINT} | jq .id
"085349ac-aece-4cfc-aacb-2562f87413fb"
❯ NF_SERVICE=085349ac-aece-4cfc-aacb-2562f87413fb
```

### Create the Bridge Gateway

Every AppWAN needs a bridge gateway to enable Ziti clients. Choose a georegion near the client for best performance. A gateway is a type of endpoint through which traffic may flow from clients to services. You could skip this step if you're not following the example to use a Ziti client.

```bash
❯ http POST https://netfoundry-programmable-zero-trust-connectivity.p.rapidapi.com/networks/${NF_NETWORK}/endpoints \
    x-rapidapi-host:netfoundry-programmable-zero-trust-connectivity.p.rapidapi.com \
    x-rapidapi-key:${RAPID_API_KEY} \
    name=exampleBridgeGw \
    endpointType=ZTGW \
    geoRegionId=${NF_CLIENT_REGION} | jq .id
"6c96b27c-f5eb-4027-aad0-1a9dc5cb176a"
❯ NF_BRIDGE_GW=6c96b27c-f5eb-4027-aad0-1a9dc5cb176a
```

### Create Your Ziti Client

Ziti is open-source software that works with NetFoundry. Ziti clients are endpoints. Endpoints send traffic to services. You could build your own app with a Ziti endpoint SDK to enable it as a client, or you could download Ziti Tunneler and use it as an intercepting proxy running on the same computer as your own app(s).

```bash
❯ http POST https://netfoundry-programmable-zero-trust-connectivity.p.rapidapi.com/networks/${NF_NETWORK}/endpoints \
    x-rapidapi-host:netfoundry-programmable-zero-trust-connectivity.p.rapidapi.com \
    x-rapidapi-key:${RAPID_API_KEY} \
    name=exampleTunneler \
    endpointType=ZTCL \
    geoRegionId=${NF_CLIENT_REGION} | jq .id
"09baa7c3-869d-4816-86f0-ef7260ba1648"
❯ NF_CLIENT=09baa7c3-869d-4816-86f0-ef7260ba1648
```

Use `endpointType=CL` if creating a non-Ziti standard client, which is outside the scope of this guide.
{: .notice--info}

### Initialize an AppWAN

An AppWAN is a policy that works like a permission group. Next we'll add the endpoints and the service to the empty AppWAN.

```bash
❯ http POST https://netfoundry-programmable-zero-trust-connectivity.p.rapidapi.com/networks/${NF_NETWORK}/appWans \
    x-rapidapi-host:netfoundry-programmable-zero-trust-connectivity.p.rapidapi.com \
    x-rapidapi-key:${RAPID_API_KEY} \
    name=exampleAppWAN | jq .id
"a088efef-f7a4-4b9a-b4de-80b6cc4025ee"
❯ NF_APPWAN=a088efef-f7a4-4b9a-b4de-80b6cc4025ee
```

### Authorize the Endpoints

Add the ID of the Ziti client and the bridge gateway to the empty AppWAN.

```bash
❯ http POST https://netfoundry-programmable-zero-trust-connectivity.p.rapidapi.com/networks/${NF_NETWORK}/appWans/${NF_APPWAN}/endpoints \
    x-rapidapi-host:netfoundry-programmable-zero-trust-connectivity.p.rapidapi.com \
    x-rapidapi-key:${RAPID_API_KEY} \
    ids:='["'${NF_CLIENT}'","'${NF_BRIDGE_GW}'"]' | jq .updatedAt
"2020-06-14T16:04:16.000+0000"

# optionally compare the updated timestamp to the current time with GNU date
❯ date -d 2020-06-14T16:04:16.000+0000
Sun 14 Jun 2020 12:04:16 PM EDT

❯ date
Sun 14 Jun 2020 12:04:56 PM EDT
```

### Authorize the Service

Add the ID of the service to the AppWAN with the endpoints.

```bash
❯ http POST https://netfoundry-programmable-zero-trust-connectivity.p.rapidapi.com/networks/${NF_NETWORK}/appWans/${NF_APPWAN}/services \
    x-rapidapi-host:netfoundry-programmable-zero-trust-connectivity.p.rapidapi.com \
    x-rapidapi-key:${RAPID_API_KEY} \
    ids:='["'${NF_SERVICE}'"]' | jq .updatedAt
"2020-06-14T16:08:42.000+0000"
```

### Download the Enrollment Token

```bash
❯ http --download --output exampleTunneler.jwt GET \
    https://netfoundry-programmable-zero-trust-connectivity.p.rapidapi.com/networks/${NF_NETWORK}/endpoints/${NF_CLIENT}/downloadRegistrationKey \
    x-rapidapi-host:netfoundry-programmable-zero-trust-connectivity.p.rapidapi.com \
    x-rapidapi-key:${RAPID_API_KEY}
```

Alternatively, for a non-Ziti client, you will be prompted for the value of endpoint attribute `registrationKey` when you run the client app [you downloaded here](https://netfoundry.io/resources/support/downloads/networkversion6/#clients).
{: .notice--info}

### Enroll the Client

Your Ziti Tunneler needs an identity .json file. The enrollment token downloaded above is used by a utility `ziti-enroller` to generate the identity .json file.

{% include ziti-enroller-lts.md %}

```bash
❯ ./ziti-enroller version
0.5.8-2554

❯ ./ziti-enroller --jwt exampleTunneler.jwt
```

### Run the Client

Download and Run Ziti Tunneler.

{% include ziti-tunneler-lts.md %}

```bash
❯ ./ziti-tunneler version
0.5.8-2554

# when running Ziti LTS in proxy mode you must provide an identifier like "{AppWAN name}-{service name}:{localport}"
❯ ./ziti-tunnel proxy exampleAppWAN-exampleService:8080 --identity exampleTunneler.json --verbose
```

The effect of this command is for Tunneler to bind to localhost:8080 and begin listening for connections. We'll test this by sending a request to that port along with a `Host` header so that the responding service will know which web site we're asking for.

```bash
❯ http GET http:localhost:8080 "Host: wttr.in"
```

{% include wttr.in.md %}

We used Tunneler's `proxy` mode for the sake of simplicity, which replaces Tunnelers "intercept" capability with a TCP proxy which includes a built-in nameserver for domain name interception. Know more about Ziti Tunnler by reading [the Tunneler manual](https://openziti.github.io/ziti/clients/tunneler.html). NetFoundry [`/rest/v1`](/v1/) works with Ziti LTS and RapidAPI, and NetFoundry [`/core/v2`](/v2/) works with Ziti (latest). RapidAPI support for NetFoundry v2 is coming soon!
{: .notice--success}

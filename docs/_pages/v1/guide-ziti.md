---
title: OpenZiti
permalink: /v1/guides/ziti/
redirect_to: /guides/

---

NetFoundry is built on Ziti. Ziti is open-source software for building programmable, zero-trust networks. You can use Ziti "endpoints" with your NetFoundry network. For example, you can embed Ziti in your app with the endpoint SDK so it can participate in a NetFoundry network directly without any external software or gateways. In that case your app will communicate with the Ziti fabric hosted by NetFoundry through the built-in Ziti libraries. If your app is a service this means it can be reached anywhere, from anywhere, by any endpoint you allow without a static public IP address. You Zitified service is invisible except to your allowed endpoints.

Visit the [Ziti web site](https://ziti.dev) to learn more about the Open Ziti project.

See [Hello, AppWAN!](/v1/guides/hello-appwan/) for a step-by-step example using Ziti LTS with NetFoundry.
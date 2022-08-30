---
title: Load Balancing
content-type: tutorial
book: get-started
chapter: 6
---

Load balancing is a method of distributing API request traffic across
multiple upstream services. Load balancing improves overall system responsiveness
and reduces failures by preventing overloading of individual resources. 

In the following example, you’ll use an application deployed across two different servers, or upstream targets. 
{{site.base_gateway}} needs to load balance across both servers, so that if one of the servers is unavailable, 
it automatically detects the problem and routes all traffic to the working server.

An [upstream](/gateway/latest/understanding-kong/key-concepts/upstreams/) 
refers to the service applications sitting behind {{site.base_gateway}}, 
to which client requests are forwarded. In {{site.base_gateway}}, an upstream represents a virtual hostname and can be 
used to health check, circuit break, and load balance incoming requests over multiple [target](gateway/latest/admin-api/#target-object) backend services.

In this section, you’ll re-configure the service created earlier, (`example_service`) to point to an upstream 
instead of a specific host. For the purposes of our example, the upstream will point to two different targets, 
`httpbin.org` and `mockbin.org`. More commonly, targets will be instances of the same backend service running on different host systems.

Here is a diagram illustrating the setup:

![Upstream targets](/assets/images/docs/getting-started-guide/upstream-targets.png)

## Enable load balancing

In this section, you will create an upstream named `example_upstream` and add two targets to it.

### Prerequisites

This chapter is part of the *Get Started with Kong* series. For the best experience, it is recommended that you follow the
series from the beginning. 

Start with the introduction, [Get Kong](/gateway/latest/get-started/get-kong), which includes
a list of prerequisites and instructions for running a local {{site.base_gateway}}.

Step two of the guide, [Services and Routes](/gateway/latest/get-started/services-and-routes),
includes instructions for installing a mock service used throughout this series. 

If you haven't completed these steps already, complete them before proceeding.

1. **Create an upstream** 

   Use the Admin API to create an upstream named `example_upstream`:

   ```sh
   curl -X POST http://localhost:8001/upstreams \
     --data name=example_upstream
   ```

1. **Create upstream targets**

   Create two targets for `example_upstream`. Each request creates a new target, and 
   sets the backend service connection endpoint:
   
   ```sh
   curl -X POST http://localhost:8001/upstreams/example_upstream/targets \
     --data target='mockbin.org:80'
   curl -X POST http://localhost:8001/upstreams/example_upstream/targets \
     --data target='httpbin.org:80'
   ```

1. **Update the service**

   In the [services and routes](/gateway/latest/get-started/services-and-routes/) section of this guide, you created `example_service` which pointed
   to an explicit host, `http://mockbin.org`. Now you'll modify that service to point to the upstream instead:
   
   ```sh
   curl -X PATCH http://localhost:8001/services/example_service \
     --data host='example_upstream'
   ```

   You now have an upstream with two targets, `httpbin.org` and `mockbin.org`, and a service pointing to that upstream.

1. **Validate**

   Validate that the upstream you configured is working by visiting the route `http://localhost:8000/mock` using a web browser or CLI.
   Continue pinging the endpoint and the site should change from `httpbin` to `mockbin`.

# Open Policy Agent (OPA) Overview and Capabilities

## Overview

Open Policy Agent (OPA) is an open-source, general-purpose policy engine that unifies policy enforcement across a wide range of systems and services. OPA allows you to define policies using a high-level, declarative language called Rego. These policies can be used to control access, enforce compliance, and manage resource usage within applications, microservices, Kubernetes clusters, and other environments.

## Key Features

### Policy as Code

* **Rego Language** : OPA uses Rego, a purpose-built policy language that enables you to express complex logic in a clear and concise manner.
* **Version Control** : Policies can be managed and version-controlled alongside application code, ensuring consistency and traceability.

### Decoupled Policy Decision Making

* **Separation of Concerns** : OPA decouples policy decision-making from policy enforcement, allowing you to manage policies centrally and apply them consistently across your infrastructure.

### Flexible Deployment Options

* **Standalone Service** : OPA can run as a standalone service, making it accessible via HTTP API calls.
* **Embedded** : OPA can be embedded directly into applications as a library.

### Wide Integration Support

* **Kubernetes** : OPA integrates seamlessly with Kubernetes to enforce admission control policies.
* **Microservices** : OPA can be used with microservices architectures to manage access control and other policies.
* **API Gateways** : OPA works with API gateways to enforce authorization policies at the edge.

### High Performance

* **Efficient Evaluation** : OPA is designed to evaluate policies quickly, making it suitable for high-throughput environments.
* **Caching** : Policies and data can be cached to improve performance.

## Deploying OPA as an Independent Service

OPA can be deployed as an independent service, allowing other services to call it explicitly for policy evaluations. This approach centralizes policy management and simplifies the integration of policy enforcement across different parts of your system.

### Deployment Steps

1. **Docker Deployment** :

<pre><div class="dark bg-gray-950 rounded-md border-[0.5px] border-token-border-medium"><div class="flex items-center relative text-token-text-secondary bg-token-main-surface-secondary px-4 py-2 text-xs font-sans justify-between rounded-t-md"><span>bash</span><div class="flex items-center"><span class="" data-state="closed"><button class="flex gap-1 items-center"><svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24" class="icon-sm"><path fill="currentColor" fill-rule="evenodd" d="M7 5a3 3 0 0 1 3-3h9a3 3 0 0 1 3 3v9a3 3 0 0 1-3 3h-2v2a3 3 0 0 1-3 3H5a3 3 0 0 1-3-3v-9a3 3 0 0 1 3-3h2zm2 2h5a3 3 0 0 1 3 3v5h2a1 1 0 0 0 1-1V5a1 1 0 0 0-1-1h-9a1 1 0 0 0-1 1zM5 9a1 1 0 0 0-1 1v9a1 1 0 0 0 1 1h9a1 1 0 0 0 1-1v-9a1 1 0 0 0-1-1z" clip-rule="evenodd"></path></svg>Copy code</button></span></div></div><div class="overflow-y-auto p-4 text-left undefined" dir="ltr"><code class="!whitespace-pre hljs language-bash">docker run -d --name opa -p 8181:8181 openpolicyagent/opa:latest run --server
   </code></div></div></pre>

1. **Kubernetes Deployment** :

<pre><div class="dark bg-gray-950 rounded-md border-[0.5px] border-token-border-medium"><div class="flex items-center relative text-token-text-secondary bg-token-main-surface-secondary px-4 py-2 text-xs font-sans justify-between rounded-t-md"><span>yaml</span><div class="flex items-center"><span class="" data-state="closed"><button class="flex gap-1 items-center"><svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24" class="icon-sm"><path fill="currentColor" fill-rule="evenodd" d="M7 5a3 3 0 0 1 3-3h9a3 3 0 0 1 3 3v9a3 3 0 0 1-3 3h-2v2a3 3 0 0 1-3 3H5a3 3 0 0 1-3-3v-9a3 3 0 0 1 3-3h2zm2 2h5a3 3 0 0 1 3 3v5h2a1 1 0 0 0 1-1V5a1 1 0 0 0-1-1h-9a1 1 0 0 0-1 1zM5 9a1 1 0 0 0-1 1v9a1 1 0 0 0 1 1h9a1 1 0 0 0 1-1v-9a1 1 0 0 0-1-1z" clip-rule="evenodd"></path></svg>Copy code</button></span></div></div><div class="overflow-y-auto p-4 text-left undefined" dir="ltr"><code class="!whitespace-pre hljs language-yaml">apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: opa
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: opa
     template:
       metadata:
         labels:
           app: opa
       spec:
         containers:
         - name: opa
           image: openpolicyagent/opa:latest
           ports:
           - containerPort: 8181
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: opa
   spec:
     ports:
     - port: 8181
       targetPort: 8181
     selector:
       app: opa
   </code></div></div></pre>

## Calling OPA to Evaluate Policies

Once OPA is deployed as an independent service, you can call its REST API to evaluate policies. Here’s an example of how to structure such calls:

### Example Policy (policy.rego)

<pre><div class="dark bg-gray-950 rounded-md border-[0.5px] border-token-border-medium"><div class="flex items-center relative text-token-text-secondary bg-token-main-surface-secondary px-4 py-2 text-xs font-sans justify-between rounded-t-md"><span>rego</span><div class="flex items-center"><span class="" data-state="closed"><button class="flex gap-1 items-center"><svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24" class="icon-sm"><path fill="currentColor" fill-rule="evenodd" d="M7 5a3 3 0 0 1 3-3h9a3 3 0 0 1 3 3v9a3 3 0 0 1-3 3h-2v2a3 3 0 0 1-3 3H5a3 3 0 0 1-3-3v-9a3 3 0 0 1 3-3h2zm2 2h5a3 3 0 0 1 3 3v5h2a1 1 0 0 0 1-1V5a1 1 0 0 0-1-1h-9a1 1 0 0 0-1 1zM5 9a1 1 0 0 0-1 1v9a1 1 0 0 0 1 1h9a1 1 0 0 0 1-1v-9a1 1 0 0 0-1-1z" clip-rule="evenodd"></path></svg>Copy code</button></span></div></div><div class="overflow-y-auto p-4 text-left undefined" dir="ltr"><code class="!whitespace-pre hljs language-rego">package authz

default allow = false

allow {
  input.method == "GET"
  input.path == ["salary", username]
  input.user == username
}</code></div></div></pre>


### Sending a Policy Query

You can query OPA by sending an HTTP POST request with the input data to be evaluated against the policy.


### Java Example

<pre><div class="dark bg-gray-950 rounded-md border-[0.5px] border-token-border-medium"><div class="flex items-center relative text-token-text-secondary bg-token-main-surface-secondary px-4 py-2 text-xs font-sans justify-between rounded-t-md"><span>java</span><div class="flex items-center"><span class="" data-state="closed"><button class="flex gap-1 items-center"><svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24" class="icon-sm"><path fill="currentColor" fill-rule="evenodd" d="M7 5a3 3 0 0 1 3-3h9a3 3 0 0 1 3 3v9a3 3 0 0 1-3 3h-2v2a3 3 0 0 1-3 3H5a3 3 0 0 1-3-3v-9a3 3 0 0 1 3-3h2zm2 2h5a3 3 0 0 1 3 3v5h2a1 1 0 0 0 1-1V5a1 1 0 0 0-1-1h-9a1 1 0 0 0-1 1zM5 9a1 1 0 0 0-1 1v9a1 1 0 0 0 1 1h9a1 1 0 0 0 1-1v-9a1 1 0 0 0-1-1z" clip-rule="evenodd"></path></svg>Copy code</button></span></div></div><div class="overflow-y-auto p-4 text-left undefined" dir="ltr"><code class="!whitespace-pre hljs language-java">import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.http.HttpEntity;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpMethod;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.client.RestTemplate;

import java.util.HashMap;
import java.util.Map;

public class OpaClient {

    private static final String OPA_URL = "http://localhost:8181/v1/data/authz/allow";

    public static void main(String[] args) {
        RestTemplate restTemplate = new RestTemplate();
        ObjectMapper objectMapper = new ObjectMapper();

        Map<String, Object> input = new HashMap<>();
        input.put("method", "GET");
        input.put("path", new String[]{"salary", "alice"});
        input.put("user", "alice");

        Map<String, Object> request = new HashMap<>();
        request.put("input", input);

        try {
            HttpHeaders headers = new HttpHeaders();
            headers.setContentType(MediaType.APPLICATION_JSON);

            HttpEntity<String> entity = new HttpEntity<>(objectMapper.writeValueAsString(request), headers);

            ResponseEntity<JsonNode> response = restTemplate.exchange(OPA_URL, HttpMethod.POST, entity, JsonNode.class);
            JsonNode result = response.getBody();

            if (result != null && result.get("result").asBoolean()) {
                System.out.println("Access granted");
            } else {
                System.out.println("Access denied");
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
</code></div></div></pre>

### Explanation:

1. **Dependencies** :

* Ensure you have the necessary dependencies in your `pom.xml` for `spring-web` and `jackson-databind`.

1. **Setting up the Request** :

* Create a `RestTemplate` instance to send the HTTP request.
* Create an `ObjectMapper` instance to convert Java objects to JSON.

1. **Preparing the Input** :

* Prepare the input data to be evaluated by OPA.
* Structure it as a map, with `method`, `path`, and `user` fields.

1. **Sending the Request** :

* Set the content type to `application/json` using `HttpHeaders`.
* Create an `HttpEntity` with the input JSON and headers.
* Send a POST request to the OPA endpoint and get the response.

1. **Handling the Response** :

* Check the result from OPA's response and print "Access granted" or "Access denied" based on the decision.

This example demonstrates how to interact with OPA's REST API from a Java application, making it straightforward to integrate OPA for policy evaluations in your services.

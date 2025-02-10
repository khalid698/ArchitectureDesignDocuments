
## RTGS API - Account API Specification

The **SWIFT API Specification** file is available at: https://docs.developer.swift.com/docs/api-guides/boe-rtgs-apis/accounts-api-reference

### **Base URLs**
The API provides multiple environments for different stages:

- **Non-Production (NP4)**: `https://api-test.swiftnet.sipn.swift.com/boe-rtgs-api-pilot-i1/acct-mgmt/np4/v1`
- **Non-Production (NP5)**: `https://api-test.swiftnet.sipn.swift.com/boe-rtgs-api-pilot-i1/acct-mgmt/np5/v1`
- **Pre-Production (PP2)**: `https://api-test.swiftnet.sipn.swift.com/boe-rtgs-api-pilot-i2/acct-mgmt/pp2/v1`
- **Pre-Production (PP2 - Another Instance)**: `https://api-test.swiftnet.sipn.swift.com/boe-rtgs-api-pilot-i3/acct-mgmt/pp2/v1`
- **Production (Live Environment)**: `https://api.swiftnet.sipn.swift.com/boe-rtgs-api/acct-mgmt/v1`

---

### **API Endpoints**
Below are the available endpoints for managing RTGS accounts:

1. **Retrieve a List of Accounts**  
   ```
   GET /accounts
   ```

2. **Retrieve Details of a Specific Account**  
   ```
   GET /accounts/{accountId}
   ```
   - Replace `{accountId}` with a valid account identifier.

3. **Retrieve Account Balance Allocations**  
   ```
   GET /accounts/{accountId}/balanceAllocations
   ```
   - Retrieves balance allocation details for a given account.

4. **Retrieve Reports Associated with an Account**  
   ```
   GET /accounts/{accountId}/reports
   ```
   - Fetches key information about reports for a given account.

5. **Retrieve a Specific Report Associated with an Account**  
   ```
   GET /accounts/{accountId}/reports/{reportId}
   ```
   - Replace `{reportId}` with a valid report identifier.

---

### **Notes**
- **Authentication**: These APIs require OAuth 2.0 authentication using bearer tokens.
- **Headers Required**: Some APIs require custom headers such as `x-userContext`, `x-journeyId`, and `X-Forwarded-Host/Path`.
- **Filtering and Pagination**: The `/accounts` endpoint supports query parameters like `limit`, `offset`, and various filters such as `participantId`, `accountStatusCode`, and `accountTypeCode`.

---

### Account API Client
Here's a **Spring Boot REST API client** implementation to call the SWIFT API endpoints. The client will have separate methods for each endpoint, using an **OAuth 2.0 Bearer Token** provided by the caller.

---

## **1. Project Setup**
### **Dependencies (Add to `pom.xml`)**
```xml
<dependencies>
    <!-- Spring Boot Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- Spring Boot Security (for OAuth if needed later) -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>

    <!-- Spring Boot Configuration Processor -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-configuration-processor</artifactId>
        <optional>true</optional>
    </dependency>

    <!-- Jackson for JSON Parsing -->
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
    </dependency>

    <!-- Spring Boot Actuator (Optional for Monitoring) -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>

    <!-- Apache HttpClient (For REST Calls) -->
    <dependency>
        <groupId>org.apache.httpcomponents.client5</groupId>
        <artifactId>httpclient5</artifactId>
    </dependency>
</dependencies>
```

---

## **2. API Configuration**
We define the **base URL** for the SWIFT API in a properties file.

### **`application.yml`**
```yaml
swift:
  base-url: https://api.swiftnet.sipn.swift.com/boe-rtgs-api/acct-mgmt/v1
```

---

## **3. Create a DTO for API Responses**
We create a generic response class to handle API responses.

### **`ApiResponse.java`**
```java
import com.fasterxml.jackson.annotation.JsonIgnoreProperties;

@JsonIgnoreProperties(ignoreUnknown = true)
public class ApiResponse<T> {
    private T data;

    public T getData() {
        return data;
    }

    public void setData(T data) {
        this.data = data;
    }
}
```

---

## **4. Implement the SWIFT API Client**
The client will handle all API calls and include error handling.

### **`SwiftApiClient.java`**
```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.http.*;
import org.springframework.stereotype.Service;
import org.springframework.web.client.HttpStatusCodeException;
import org.springframework.web.client.RestTemplate;

import java.util.HashMap;
import java.util.Map;

@Service
public class SwiftApiClient {

    private final RestTemplate restTemplate;
    private final String baseUrl;

    public SwiftApiClient(@Value("${swift.base-url}") String baseUrl) {
        this.restTemplate = new RestTemplate();
        this.baseUrl = baseUrl;
    }

    private HttpHeaders createHeaders(String token) {
        HttpHeaders headers = new HttpHeaders();
        headers.set("Authorization", "Bearer " + token);
        headers.setContentType(MediaType.APPLICATION_JSON);
        return headers;
    }

    private <T> ResponseEntity<T> makeApiCall(String url, HttpMethod method, String token, Class<T> responseType) {
        HttpHeaders headers = createHeaders(token);
        HttpEntity<String> entity = new HttpEntity<>(headers);

        try {
            return restTemplate.exchange(url, method, entity, responseType);
        } catch (HttpStatusCodeException ex) {
            System.err.println("API call failed: " + ex.getStatusCode() + " - " + ex.getResponseBodyAsString());
            throw new RuntimeException("API call error: " + ex.getMessage());
        }
    }

    // 1. Retrieve a List of Accounts
    public ResponseEntity<String> getAccounts(String token) {
        String url = baseUrl + "/accounts";
        return makeApiCall(url, HttpMethod.GET, token, String.class);
    }

    // 2. Retrieve Details of a Specific Account
    public ResponseEntity<String> getAccountDetails(String accountId, String token) {
        String url = baseUrl + "/accounts/" + accountId;
        return makeApiCall(url, HttpMethod.GET, token, String.class);
    }

    // 3. Retrieve Account Balance Allocations
    public ResponseEntity<String> getAccountBalanceAllocations(String accountId, String token) {
        String url = baseUrl + "/accounts/" + accountId + "/balanceAllocations";
        return makeApiCall(url, HttpMethod.GET, token, String.class);
    }

    // 4. Retrieve Reports Associated with an Account
    public ResponseEntity<String> getAccountReports(String accountId, String token) {
        String url = baseUrl + "/accounts/" + accountId + "/reports";
        return makeApiCall(url, HttpMethod.GET, token, String.class);
    }

    // 5. Retrieve a Specific Report Associated with an Account
    public ResponseEntity<String> getSingleAccountReport(String accountId, String reportId, String token) {
        String url = baseUrl + "/accounts/" + accountId + "/reports/" + reportId;
        return makeApiCall(url, HttpMethod.GET, token, String.class);
    }
}
```

---

## **5. Implement a REST Controller**
This controller allows external applications to trigger API calls.

### **`SwiftApiController.java`**
```java
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/swift")
public class SwiftApiController {

    private final SwiftApiClient swiftApiClient;

    public SwiftApiController(SwiftApiClient swiftApiClient) {
        this.swiftApiClient = swiftApiClient;
    }

    @GetMapping("/accounts")
    public ResponseEntity<String> getAccounts(@RequestHeader("Authorization") String token) {
        return swiftApiClient.getAccounts(token.replace("Bearer ", ""));
    }

    @GetMapping("/accounts/{accountId}")
    public ResponseEntity<String> getAccountDetails(@PathVariable String accountId, @RequestHeader("Authorization") String token) {
        return swiftApiClient.getAccountDetails(accountId, token.replace("Bearer ", ""));
    }

    @GetMapping("/accounts/{accountId}/balanceAllocations")
    public ResponseEntity<String> getAccountBalanceAllocations(@PathVariable String accountId, @RequestHeader("Authorization") String token) {
        return swiftApiClient.getAccountBalanceAllocations(accountId, token.replace("Bearer ", ""));
    }

    @GetMapping("/accounts/{accountId}/reports")
    public ResponseEntity<String> getAccountReports(@PathVariable String accountId, @RequestHeader("Authorization") String token) {
        return swiftApiClient.getAccountReports(accountId, token.replace("Bearer ", ""));
    }

    @GetMapping("/accounts/{accountId}/reports/{reportId}")
    public ResponseEntity<String> getSingleAccountReport(@PathVariable String accountId, @PathVariable String reportId, @RequestHeader("Authorization") String token) {
        return swiftApiClient.getSingleAccountReport(accountId, reportId, token.replace("Bearer ", ""));
    }
}
```

---

## **6. Running and Testing**
1. **Start the Spring Boot application**
   ```sh
   mvn spring-boot:run
   ```

2. **Test API Calls using Postman or Curl**

   - Fetch all accounts:
     ```sh
     curl -X GET "http://localhost:8080/swift/accounts" -H "Authorization: Bearer YOUR_TOKEN"
     ```
   - Fetch account details:
     ```sh
     curl -X GET "http://localhost:8080/swift/accounts/ACCOUNT_ID" -H "Authorization: Bearer YOUR_TOKEN"
     ```
   - Fetch account balance allocations:
     ```sh
     curl -X GET "http://localhost:8080/swift/accounts/ACCOUNT_ID/balanceAllocations" -H "Authorization: Bearer YOUR_TOKEN"
     ```
   - Fetch reports:
     ```sh
     curl -X GET "http://localhost:8080/swift/accounts/ACCOUNT_ID/reports" -H "Authorization: Bearer YOUR_TOKEN"
     ```

---

### **Next Steps**
- Implement OAuth Token Fetching dynamically.
- Add more advanced error handling with custom exception classes.
- Secure API access with Spring Security.

Would you like me to extend this further with automated tests or OAuth token management? 🚀

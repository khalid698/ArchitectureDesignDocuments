
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

---
### Update version with specific DTOs

```
package com.bankofengland.rtgs;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;
import org.springframework.web.reactive.function.client.WebClient;
import reactor.core.publisher.Mono;
import java.math.BigDecimal;
import java.util.List;

@Service
public class RtgsAccountsApiClient {

    private final WebClient webClient;

    public RtgsAccountsApiClient(@Value("${rtgs.api.baseUrl}") String baseUrl, WebClient.Builder webClientBuilder) {
        this.webClient = webClientBuilder.baseUrl(baseUrl).build();
    }

    public AccountSingleResponse getSingleAccount(String accountId, String userContext, String journeyId,
                                                    String forwardedHost, String forwardedPath) throws ApiException {
        return webClient.get()
                .uri(uriBuilder -> uriBuilder.path("/accounts/{accountId}").build(accountId))
                .headers(headers -> {
                    if (userContext != null) headers.add("x-userContext", userContext);
                    if (journeyId != null) headers.add("x-journeyId", journeyId);
                    if (forwardedHost != null) headers.add("X-Forwarded-Host", forwardedHost);
                    if (forwardedPath != null) headers.add("X-Forwarded-Path", forwardedPath);
                })
                .retrieve()
                .onStatus(status -> !status.is2xxSuccessful(),
                        response -> response.bodyToMono(APIErrorResponseWrapperDTO.class)
                                .flatMap(error -> Mono.error(new ApiException("Error retrieving account: " + error))))
                .bodyToMono(AccountSingleResponse.class)
                .block();
    }

    public AccountBalanceAllocationResponseList getAccountBalanceAllocations(String accountId,
                                                                             String balanceAllocationSuffix,
                                                                             String balanceAllocationName,
                                                                             Double negativeBalance,
                                                                             Double currentBalance,
                                                                             Double availableBalance,
                                                                             String balanceAllocationStatusCode,
                                                                             String balanceAllocationTypeCode,
                                                                             String variant,
                                                                             Integer limit,
                                                                             Integer offset,
                                                                             String sort,
                                                                             String userContext,
                                                                             String journeyId,
                                                                             String forwardedHost,
                                                                             String forwardedPath) throws ApiException {
        return webClient.get()
                .uri(uriBuilder -> {
                    uriBuilder.path("/accounts/{accountId}/balanceAllocations");
                    if (balanceAllocationSuffix != null)
                        uriBuilder.queryParam("balanceAllocationSuffix", balanceAllocationSuffix);
                    if (balanceAllocationName != null)
                        uriBuilder.queryParam("balanceAllocationName", balanceAllocationName);
                    if (negativeBalance != null)
                        uriBuilder.queryParam("negativeBalance", negativeBalance);
                    if (currentBalance != null)
                        uriBuilder.queryParam("currentBalance", currentBalance);
                    if (availableBalance != null)
                        uriBuilder.queryParam("availableBalance", availableBalance);
                    if (balanceAllocationStatusCode != null)
                        uriBuilder.queryParam("balanceAllocationStatusCode", balanceAllocationStatusCode);
                    if (balanceAllocationTypeCode != null)
                        uriBuilder.queryParam("balanceAllocationTypeCode", balanceAllocationTypeCode);
                    if (variant != null)
                        uriBuilder.queryParam("variant", variant);
                    if (limit != null)
                        uriBuilder.queryParam("limit", limit);
                    if (offset != null)
                        uriBuilder.queryParam("offset", offset);
                    if (sort != null)
                        uriBuilder.queryParam("sort", sort);
                    return uriBuilder.build(accountId);
                })
                .headers(headers -> {
                    if (userContext != null) headers.add("x-userContext", userContext);
                    if (journeyId != null) headers.add("x-journeyId", journeyId);
                    if (forwardedHost != null) headers.add("X-Forwarded-Host", forwardedHost);
                    if (forwardedPath != null) headers.add("X-Forwarded-Path", forwardedPath);
                })
                .retrieve()
                .onStatus(status -> !status.is2xxSuccessful(),
                        response -> response.bodyToMono(APIErrorResponseWrapperDTO.class)
                                .flatMap(error -> Mono.error(new ApiException("Error retrieving balance allocations: " + error))))
                .bodyToMono(AccountBalanceAllocationResponseList.class)
                .block();
    }

    public AccountResponseList getAccounts(String participantId, String schemeCode, String accountStatusCode,
                                             String accountBookCode, String accountTypeCode, String accountSubTypeCode,
                                             String accountNumber, String transactionSubTypeCode, Integer limit,
                                             Integer offset, String sort, String userContext, String journeyId,
                                             String forwardedHost, String forwardedPath) throws ApiException {
        return webClient.get()
                .uri(uriBuilder -> {
                    uriBuilder.path("/accounts");
                    if (participantId != null)
                        uriBuilder.queryParam("participantId", participantId);
                    if (schemeCode != null)
                        uriBuilder.queryParam("schemeCode", schemeCode);
                    if (accountStatusCode != null)
                        uriBuilder.queryParam("accountStatusCode", accountStatusCode);
                    if (accountBookCode != null)
                        uriBuilder.queryParam("accountBookCode", accountBookCode);
                    if (accountTypeCode != null)
                        uriBuilder.queryParam("accountTypeCode", accountTypeCode);
                    if (accountSubTypeCode != null)
                        uriBuilder.queryParam("accountSubTypeCode", accountSubTypeCode);
                    if (accountNumber != null)
                        uriBuilder.queryParam("accountNumber", accountNumber);
                    if (transactionSubTypeCode != null)
                        uriBuilder.queryParam("transactionSubTypeCode", transactionSubTypeCode);
                    if (limit != null)
                        uriBuilder.queryParam("limit", limit);
                    if (offset != null)
                        uriBuilder.queryParam("offset", offset);
                    if (sort != null)
                        uriBuilder.queryParam("sort", sort);
                    return uriBuilder.build();
                })
                .headers(headers -> {
                    if (userContext != null) headers.add("x-userContext", userContext);
                    if (journeyId != null) headers.add("x-journeyId", journeyId);
                    if (forwardedHost != null) headers.add("X-Forwarded-Host", forwardedHost);
                    if (forwardedPath != null) headers.add("X-Forwarded-Path", forwardedPath);
                })
                .retrieve()
                .onStatus(status -> !status.is2xxSuccessful(),
                        response -> response.bodyToMono(APIErrorResponseWrapperDTO.class)
                                .flatMap(error -> Mono.error(new ApiException("Error retrieving accounts: " + error))))
                .bodyToMono(AccountResponseList.class)
                .block();
    }

    public AccountReportResponseList getAccountReports(String accountId, String reportTypeCode, String valueDate,
                                                         Integer limit, Integer offset, String sort,
                                                         String userContext, String journeyId,
                                                         String forwardedHost, String forwardedPath) throws ApiException {
        return webClient.get()
                .uri(uriBuilder -> {
                    uriBuilder.path("/accounts/{accountId}/reports");
                    if (reportTypeCode != null)
                        uriBuilder.queryParam("reportTypeCode", reportTypeCode);
                    if (valueDate != null)
                        uriBuilder.queryParam("valueDate", valueDate);
                    if (limit != null)
                        uriBuilder.queryParam("limit", limit);
                    if (offset != null)
                        uriBuilder.queryParam("offset", offset);
                    if (sort != null)
                        uriBuilder.queryParam("sort", sort);
                    return uriBuilder.build(accountId);
                })
                .headers(headers -> {
                    if (userContext != null) headers.add("x-userContext", userContext);
                    if (journeyId != null) headers.add("x-journeyId", journeyId);
                    if (forwardedHost != null) headers.add("X-Forwarded-Host", forwardedHost);
                    if (forwardedPath != null) headers.add("X-Forwarded-Path", forwardedPath);
                })
                .retrieve()
                .onStatus(status -> !status.is2xxSuccessful(),
                        response -> response.bodyToMono(APIErrorResponseWrapperDTO.class)
                                .flatMap(error -> Mono.error(new ApiException("Error retrieving account reports: " + error))))
                .bodyToMono(AccountReportResponseList.class)
                .block();
    }

    public AccountReport getSingleAccountReport(String accountId, String reportId, String userContext,
                                                  String journeyId, String forwardedHost, String forwardedPath) throws ApiException {
        return webClient.get()
                .uri(uriBuilder -> uriBuilder.path("/accounts/{accountId}/reports/{reportId}").build(accountId, reportId))
                .headers(headers -> {
                    if (userContext != null) headers.add("x-userContext", userContext);
                    if (journeyId != null) headers.add("x-journeyId", journeyId);
                    if (forwardedHost != null) headers.add("X-Forwarded-Host", forwardedHost);
                    if (forwardedPath != null) headers.add("X-Forwarded-Path", forwardedPath);
                })
                .retrieve()
                .onStatus(status -> !status.is2xxSuccessful(),
                        response -> response.bodyToMono(APIErrorResponseWrapperDTO.class)
                                .flatMap(error -> Mono.error(new ApiException("Error retrieving account report: " + error))))
                .bodyToMono(AccountReport.class)
                .block();
    }

    public static class AccountSingleResponse {
        private String accountId;
        private String accountName;
        private String accountStatus;
        private BigDecimal currentBalance;

        public String getAccountId() { return accountId; }
        public void setAccountId(String accountId) { this.accountId = accountId; }
        public String getAccountName() { return accountName; }
        public void setAccountName(String accountName) { this.accountName = accountName; }
        public String getAccountStatus() { return accountStatus; }
        public void setAccountStatus(String accountStatus) { this.accountStatus = accountStatus; }
        public BigDecimal getCurrentBalance() { return currentBalance; }
        public void setCurrentBalance(BigDecimal currentBalance) { this.currentBalance = currentBalance; }

        @Override
        public String toString() {
            return "AccountSingleResponse{accountId='" + accountId + "', accountName='" + accountName +
                    "', accountStatus='" + accountStatus + "', currentBalance=" + currentBalance + "}";
        }
    }

    public static class AccountBalanceAllocationResponseList {
        private List<BalanceAllocation> allocations;
        public List<BalanceAllocation> getAllocations() { return allocations; }
        public void setAllocations(List<BalanceAllocation> allocations) { this.allocations = allocations; }
    }

    public static class BalanceAllocation {
        private String balanceAllocationId;
        private String balanceAllocationSuffix;
        private BigDecimal currentBalance;
        private BigDecimal availableBalance;
        public String getBalanceAllocationId() { return balanceAllocationId; }
        public void setBalanceAllocationId(String balanceAllocationId) { this.balanceAllocationId = balanceAllocationId; }
        public String getBalanceAllocationSuffix() { return balanceAllocationSuffix; }
        public void setBalanceAllocationSuffix(String balanceAllocationSuffix) { this.balanceAllocationSuffix = balanceAllocationSuffix; }
        public BigDecimal getCurrentBalance() { return currentBalance; }
        public void setCurrentBalance(BigDecimal currentBalance) { this.currentBalance = currentBalance; }
        public BigDecimal getAvailableBalance() { return availableBalance; }
        public void setAvailableBalance(BigDecimal availableBalance) { this.availableBalance = availableBalance; }
    }

    public static class AccountResponseList {
        private List<AccountSingleResponse> accounts;
        public List<AccountSingleResponse> getAccounts() { return accounts; }
        public void setAccounts(List<AccountSingleResponse> accounts) { this.accounts = accounts; }
    }

    public static class AccountReportResponseList {
        private List<AccountReport> reports;
        public List<AccountReport> getReports() { return reports; }
        public void setReports(List<AccountReport> reports) { this.reports = reports; }
    }

    public static class AccountReport {
        private String reportId;
        private String reportTypeCode;
        private String reportCreatedAt;
        public String getReportId() { return reportId; }
        public void setReportId(String reportId) { this.reportId = reportId; }
        public String getReportTypeCode() { return reportTypeCode; }
        public void setReportTypeCode(String reportTypeCode) { this.reportTypeCode = reportTypeCode; }
        public String getReportCreatedAt() { return reportCreatedAt; }
        public void setReportCreatedAt(String reportCreatedAt) { this.reportCreatedAt = reportCreatedAt; }
    }

    public static class APIErrorResponseWrapperDTO {
        private APIError error;
        public APIError getError() { return error; }
        public void setError(APIError error) { this.error = error; }
        @Override
        public String toString() { return "APIErrorResponseWrapperDTO{error=" + error + "}"; }
    }

    public static class APIError {
        private String status;
        private String id;
        private String detail;
        private List<APIErrorDetail> errors;
        public String getStatus() { return status; }
        public void setStatus(String status) { this.status = status; }
        public String getId() { return id; }
        public void setId(String id) { this.id = id; }
        public String getDetail() { return detail; }
        public void setDetail(String detail) { this.detail = detail; }
        public List<APIErrorDetail> getErrors() { return errors; }
        public void setErrors(List<APIErrorDetail> errors) { this.errors = errors; }
        @Override
        public String toString() {
            return "APIError{status='" + status + "', id='" + id + "', detail='" + detail + "', errors=" + errors + "}";
        }
    }

    public static class APIErrorDetail {
        private String errorCode;
        private String detail;
        public String getErrorCode() { return errorCode; }
        public void setErrorCode(String errorCode) { this.errorCode = errorCode; }
        public String getDetail() { return detail; }
        public void setDetail(String detail) { this.detail = detail; }
        @Override
        public String toString() {
            return "APIErrorDetail{errorCode='" + errorCode + "', detail='" + detail + "'}";
        }
    }

    public static class ApiException extends Exception {
        public ApiException(String message) { super(message); }
        public ApiException(String message, Throwable cause) { super(message, cause); }
    }
}
```

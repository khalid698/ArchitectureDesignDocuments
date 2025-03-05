## Maven‐based Spring Boot microservice for a bank account API. 
  
This sample exposes two HTTP endpoints:  
- GET /accounts – returns all existing accounts   
- POST /account – creates a new account   
  
### 1. Maven Project Setup (pom.xml)
   
```
<project xmlns="http://maven.apache.org/POM/4.0.0" 
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
                             http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>bank-account-api</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>
    <name>Bank Account API</name>
    <description>Microservice for bank account API using Spring Boot</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.5</version>
    </parent>

    <properties>
        <java.version>11</java.version>
    </properties>

    <dependencies>
        <!-- Spring Boot Starter Web -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- Jackson for JSON -->
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
        </dependency>

        <!-- Lombok (Optional, helps reduce boilerplate) -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <!-- Spring Boot Starter Test (for unit testing) -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <!-- Spring Boot Maven Plugin -->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

### 2. Main Application Class  
```
package com.example.bankaccountapi;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class BankAccountApiApplication {
    public static void main(String[] args) {
        SpringApplication.run(BankAccountApiApplication.class, args);
    }
}

```

### 3. Account Model 
```
package com.example.bankaccountapi.model;

public class Account {
    
    private Long id;
    private String accountName;
    private String accountNumber;
    
    // Default constructor
    public Account() { }
    
    // Constructor with fields (excluding id, which is generated)
    public Account(String accountName, String accountNumber) {
        this.accountName = accountName;
        this.accountNumber = accountNumber;
    }
    
    // Getters and setters
    public Long getId() {
        return id;
    }
    public void setId(Long id) {
        this.id = id;
    }
    
    public String getAccountName() {
        return accountName;
    }
    public void setAccountName(String accountName) {
        this.accountName = accountName;
    }
    
    public String getAccountNumber() {
        return accountNumber;
    }
    public void setAccountNumber(String accountNumber) {
        this.accountNumber = accountNumber;
    }
}

```

### 4. In‑Memory Account Repository  
```
package com.example.bankaccountapi.repository;

import com.example.bankaccountapi.model.Account;
import org.springframework.stereotype.Repository;

import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.atomic.AtomicLong;

@Repository
public class AccountRepository {

    // Thread-safe storage for accounts
    private final Map<Long, Account> accountStorage = new ConcurrentHashMap<>();
    private final AtomicLong idCounter = new AtomicLong();

    public List<Account> findAll() {
        return new ArrayList<>(accountStorage.values());
    }

    public Account save(Account account) {
        Long id = idCounter.incrementAndGet();
        account.setId(id);
        accountStorage.put(id, account);
        return account;
    }
}

```

### 5. REST Controller  
```
package com.example.bankaccountapi.controller;

import com.example.bankaccountapi.model.Account;
import com.example.bankaccountapi.repository.AccountRepository;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
public class AccountController {

    private final AccountRepository accountRepository;

    public AccountController(AccountRepository accountRepository) {
        this.accountRepository = accountRepository;
    }

    // Endpoint to return all accounts
    @GetMapping("/accounts")
    public ResponseEntity<List<Account>> getAllAccounts() {
        List<Account> accounts = accountRepository.findAll();
        return ResponseEntity.ok(accounts);
    }

    // Endpoint to create a new account
    @PostMapping("/account")
    public ResponseEntity<Account> createAccount(@RequestBody Account account) {
        // In a real application, you’d perform validations here.
        Account savedAccount = accountRepository.save(account);
        return new ResponseEntity<>(savedAccount, HttpStatus.CREATED);
    }
}

```

### 6. Running the Application  
```
mvn spring-boot:run

curl -X GET http://localhost:8080/accounts

curl -X POST http://localhost:8080/account \
     -H "Content-Type: application/json" \
     -d '{"accountName": "John Doe Savings", "accountNumber": "12345678"}'
```

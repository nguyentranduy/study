# Example: Banking System với DDD

Đây là một mini project Banking System được thiết kế theo DDD. Tập trung vào **kiến trúc và cấu trúc code**, không đi sâu vào implementation chi tiết.

## Tổng quan Domain

**Banking Domain** bao gồm:
- **Account Management**: Quản lý tài khoản ngân hàng
- **Transaction Processing**: Xử lý giao dịch (nạp, rút, chuyển tiền)
- **Customer Management**: Quản lý khách hàng

## Bounded Contexts

```
┌───────────────────────────┐     ┌───────────────────────────┐
│   Account Context         │     │   Customer Context        │
│                           │     │                           │
│   - BankAccount           │────▶│   - Customer              │
│   - Transaction           │     │   - CustomerProfile       │
│   - Balance               │     │                           │
└───────────────────────────┘     └───────────────────────────┘
             │
             │
             ▼
┌───────────────────────────┐
│   Fraud Detection Context │
│                           │
│   - FraudCheck            │
│   - RiskAssessment        │
└───────────────────────────┘
```

Trong ví dụ này, chúng ta tập trung vào **Account Context**.

## Kiến trúc tổng quan

```
┌─────────────────────────────────────────────────────┐
│           Interfaces Layer                          │
│  (REST Controllers, Web UI, CLI)                    │
└─────────────┬───────────────────────────────────────┘
              │
┌─────────────▼───────────────────────────────────────┐
│         Application Layer                           │
│  (Use Cases, Application Services, DTOs)            │
└─────────────┬───────────────────────────────────────┘
              │
┌─────────────▼───────────────────────────────────────┐
│           Domain Layer                              │
│  (Entities, Value Objects, Aggregates,              │
│   Domain Services, Domain Events, Repositories)     │
└─────────────┬───────────────────────────────────────┘
              │
┌─────────────▼───────────────────────────────────────┐
│       Infrastructure Layer                          │
│  (Database, External APIs, Message Queue)           │
└─────────────────────────────────────────────────────┘
```

## Cấu trúc thư mục

```
src/main/java/com/bank/
├── domain/
│   ├── account/
│   │   ├── model/
│   │   │   ├── BankAccount.java          # Aggregate Root
│   │   │   ├── Transaction.java          # Entity
│   │   │   ├── AccountId.java            # Value Object
│   │   │   ├── Money.java                # Value Object
│   │   │   ├── AccountType.java          # Enum
│   │   │   └── AccountStatus.java        # Enum
│   │   ├── service/
│   │   │   └── TransferService.java      # Domain Service
│   │   ├── repository/
│   │   │   └── AccountRepository.java    # Repository Interface
│   │   ├── event/
│   │   │   ├── MoneyDeposited.java       # Domain Event
│   │   │   ├── MoneyWithdrawn.java       # Domain Event
│   │   │   └── MoneyTransferred.java     # Domain Event
│   │   └── exception/
│   │       ├── InsufficientFundsException.java
│   │       └── AccountNotFoundException.java
│   └── customer/
│       └── model/
│           ├── CustomerId.java           # Value Object
│           └── Customer.java             # Entity (simplified)
├── application/
│   ├── account/
│   │   ├── service/
│   │   │   └── AccountApplicationService.java
│   │   ├── command/
│   │   │   ├── CreateAccountCommand.java
│   │   │   ├── DepositMoneyCommand.java
│   │   │   ├── WithdrawMoneyCommand.java
│   │   │   └── TransferMoneyCommand.java
│   │   └── query/
│   │       ├── AccountQuery.java
│   │       └── AccountDTO.java
│   └── ...
├── infrastructure/
│   ├── persistence/
│   │   ├── jpa/
│   │   │   ├── JpaAccountRepository.java
│   │   │   └── AccountEntity.java        # JPA Entity
│   │   └── mapper/
│   │       └── AccountMapper.java
│   └── messaging/
│       └── EventPublisher.java
└── interfaces/
    ├── rest/
    │   └── AccountController.java
    └── dto/
        ├── CreateAccountRequest.java
        └── TransferMoneyRequest.java
```

## 1. Domain Layer - Value Objects

### Money.java
```java
package com.bank.domain.account.model;

import java.math.BigDecimal;
import java.util.Currency;
import java.util.Objects;

/**
 * Value Object đại diện cho số tiền với currency
 */
public final class Money {
    private final BigDecimal amount;
    private final Currency currency;
    
    private Money(BigDecimal amount, Currency currency) {
        validateAmount(amount);
        this.amount = amount;
        this.currency = Objects.requireNonNull(currency, "Currency cannot be null");
    }
    
    public static Money of(BigDecimal amount, String currencyCode) {
        return new Money(amount, Currency.getInstance(currencyCode));
    }
    
    public static Money zero(String currencyCode) {
        return new Money(BigDecimal.ZERO, Currency.getInstance(currencyCode));
    }
    
    private void validateAmount(BigDecimal amount) {
        if (amount == null) {
            throw new IllegalArgumentException("Amount cannot be null");
        }
        if (amount.compareTo(BigDecimal.ZERO) < 0) {
            throw new IllegalArgumentException("Amount cannot be negative");
        }
    }
    
    // Arithmetic operations - return new object (immutable)
    public Money add(Money other) {
        validateSameCurrency(other);
        return new Money(this.amount.add(other.amount), this.currency);
    }
    
    public Money subtract(Money other) {
        validateSameCurrency(other);
        if (this.amount.compareTo(other.amount) < 0) {
            throw new IllegalArgumentException("Result would be negative");
        }
        return new Money(this.amount.subtract(other.amount), this.currency);
    }
    
    // Comparison
    public boolean isGreaterThan(Money other) {
        validateSameCurrency(other);
        return this.amount.compareTo(other.amount) > 0;
    }
    
    public boolean isGreaterThanOrEqual(Money other) {
        validateSameCurrency(other);
        return this.amount.compareTo(other.amount) >= 0;
    }
    
    private void validateSameCurrency(Money other) {
        if (!this.currency.equals(other.currency)) {
            throw new IllegalArgumentException("Cannot operate on different currencies");
        }
    }
    
    // Getters
    public BigDecimal getAmount() { return amount; }
    public Currency getCurrency() { return currency; }
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Money)) return false;
        Money money = (Money) o;
        return amount.equals(money.amount) && currency.equals(money.currency);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(amount, currency);
    }
    
    @Override
    public String toString() {
        return amount + " " + currency.getCurrencyCode();
    }
}
```

### AccountId.java
```java
package com.bank.domain.account.model;

import java.util.Objects;
import java.util.UUID;

/**
 * Value Object đại diện cho Account Identity
 */
public final class AccountId {
    private final String value;
    
    private AccountId(String value) {
        if (value == null || value.trim().isEmpty()) {
            throw new IllegalArgumentException("AccountId cannot be null or empty");
        }
        this.value = value;
    }
    
    public static AccountId of(String value) {
        return new AccountId(value);
    }
    
    public static AccountId generate() {
        return new AccountId(UUID.randomUUID().toString());
    }
    
    public String getValue() {
        return value;
    }
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof AccountId)) return false;
        AccountId accountId = (AccountId) o;
        return value.equals(accountId.value);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(value);
    }
    
    @Override
    public String toString() {
        return value;
    }
}
```

## 2. Domain Layer - Entities & Aggregate

### Transaction.java (Entity)
```java
package com.bank.domain.account.model;

import java.time.LocalDateTime;
import java.util.Objects;

/**
 * Entity trong Account Aggregate
 */
public class Transaction {
    private final String transactionId;
    private final TransactionType type;
    private final Money amount;
    private final LocalDateTime timestamp;
    private final String description;
    
    private Transaction(String transactionId, TransactionType type, 
                       Money amount, String description) {
        this.transactionId = Objects.requireNonNull(transactionId);
        this.type = Objects.requireNonNull(type);
        this.amount = Objects.requireNonNull(amount);
        this.description = description;
        this.timestamp = LocalDateTime.now();
    }
    
    public static Transaction deposit(String id, Money amount, String description) {
        return new Transaction(id, TransactionType.DEPOSIT, amount, description);
    }
    
    public static Transaction withdrawal(String id, Money amount, String description) {
        return new Transaction(id, TransactionType.WITHDRAWAL, amount, description);
    }
    
    public static Transaction transfer(String id, Money amount, String description) {
        return new Transaction(id, TransactionType.TRANSFER, amount, description);
    }
    
    // Getters
    public String getTransactionId() { return transactionId; }
    public TransactionType getType() { return type; }
    public Money getAmount() { return amount; }
    public LocalDateTime getTimestamp() { return timestamp; }
    public String getDescription() { return description; }
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Transaction)) return false;
        Transaction that = (Transaction) o;
        return transactionId.equals(that.transactionId);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(transactionId);
    }
}

enum TransactionType {
    DEPOSIT, WITHDRAWAL, TRANSFER
}
```

### BankAccount.java (Aggregate Root)
```java
package com.bank.domain.account.model;

import com.bank.domain.account.event.*;
import com.bank.domain.account.exception.*;
import com.bank.domain.customer.model.CustomerId;
import java.time.LocalDateTime;
import java.util.*;

/**
 * Aggregate Root - BankAccount
 * Quản lý tất cả business logic liên quan đến account
 */
public class BankAccount {
    private final AccountId id;
    private final CustomerId ownerId;
    private final AccountType accountType;
    private Money balance;
    private AccountStatus status;
    private final LocalDateTime createdAt;
    private final List<Transaction> transactions;
    
    // Domain Events chờ publish
    private final List<DomainEvent> domainEvents;
    
    // Constructor - private, chỉ dùng qua factory method
    private BankAccount(AccountId id, CustomerId ownerId, AccountType accountType, 
                       Money initialBalance) {
        this.id = Objects.requireNonNull(id);
        this.ownerId = Objects.requireNonNull(ownerId);
        this.accountType = Objects.requireNonNull(accountType);
        this.balance = initialBalance;
        this.status = AccountStatus.ACTIVE;
        this.createdAt = LocalDateTime.now();
        this.transactions = new ArrayList<>();
        this.domainEvents = new ArrayList<>();
    }
    
    // Factory method
    public static BankAccount open(AccountId id, CustomerId ownerId, 
                                  AccountType accountType, Money initialDeposit) {
        validateInitialDeposit(accountType, initialDeposit);
        
        BankAccount account = new BankAccount(id, ownerId, accountType, initialDeposit);
        
        // Record initial transaction
        if (initialDeposit.getAmount().compareTo(java.math.BigDecimal.ZERO) > 0) {
            Transaction initialTransaction = Transaction.deposit(
                UUID.randomUUID().toString(),
                initialDeposit,
                "Initial deposit"
            );
            account.transactions.add(initialTransaction);
        }
        
        // Raise domain event
        account.addDomainEvent(new AccountOpened(id, ownerId, accountType, initialDeposit));
        
        return account;
    }
    
    private static void validateInitialDeposit(AccountType accountType, Money initialDeposit) {
        Money minimumDeposit = accountType.getMinimumBalance();
        if (initialDeposit.getAmount().compareTo(minimumDeposit.getAmount()) < 0) {
            throw new IllegalArgumentException(
                "Initial deposit must be at least " + minimumDeposit
            );
        }
    }
    
    // Business method: Deposit money
    public void deposit(Money amount, String description) {
        validateAccountActive();
        validatePositiveAmount(amount);
        
        // Update balance
        this.balance = this.balance.add(amount);
        
        // Record transaction
        Transaction transaction = Transaction.deposit(
            UUID.randomUUID().toString(),
            amount,
            description
        );
        this.transactions.add(transaction);
        
        // Raise domain event
        addDomainEvent(new MoneyDeposited(this.id, amount, this.balance));
    }
    
    // Business method: Withdraw money
    public void withdraw(Money amount, String description) {
        validateAccountActive();
        validatePositiveAmount(amount);
        validateSufficientBalance(amount);
        validateMinimumBalanceAfterWithdrawal(amount);
        
        // Update balance
        this.balance = this.balance.subtract(amount);
        
        // Record transaction
        Transaction transaction = Transaction.withdrawal(
            UUID.randomUUID().toString(),
            amount,
            description
        );
        this.transactions.add(transaction);
        
        // Raise domain event
        addDomainEvent(new MoneyWithdrawn(this.id, amount, this.balance));
    }
    
    // Business method: Transfer money (debit side)
    public void debit(Money amount, AccountId beneficiaryAccountId) {
        validateAccountActive();
        validatePositiveAmount(amount);
        validateSufficientBalance(amount);
        
        // Update balance
        this.balance = this.balance.subtract(amount);
        
        // Record transaction
        Transaction transaction = Transaction.transfer(
            UUID.randomUUID().toString(),
            amount,
            "Transfer to " + beneficiaryAccountId
        );
        this.transactions.add(transaction);
        
        // Raise domain event
        addDomainEvent(new MoneyTransferred(this.id, beneficiaryAccountId, amount, this.balance));
    }
    
    // Business method: Transfer money (credit side)
    public void credit(Money amount, AccountId senderAccountId) {
        validateAccountActive();
        validatePositiveAmount(amount);
        
        // Update balance
        this.balance = this.balance.add(amount);
        
        // Record transaction
        Transaction transaction = Transaction.transfer(
            UUID.randomUUID().toString(),
            amount,
            "Transfer from " + senderAccountId
        );
        this.transactions.add(transaction);
        
        // No event needed - handled by debit side
    }
    
    // Business method: Close account
    public void close() {
        if (status == AccountStatus.CLOSED) {
            throw new IllegalStateException("Account is already closed");
        }
        
        if (balance.getAmount().compareTo(java.math.BigDecimal.ZERO) > 0) {
            throw new IllegalStateException("Cannot close account with positive balance");
        }
        
        this.status = AccountStatus.CLOSED;
        addDomainEvent(new AccountClosed(this.id));
    }
    
    // Validations
    private void validateAccountActive() {
        if (status != AccountStatus.ACTIVE) {
            throw new AccountNotActiveException("Account is not active");
        }
    }
    
    private void validatePositiveAmount(Money amount) {
        if (amount.getAmount().compareTo(java.math.BigDecimal.ZERO) <= 0) {
            throw new IllegalArgumentException("Amount must be positive");
        }
    }
    
    private void validateSufficientBalance(Money amount) {
        if (balance.getAmount().compareTo(amount.getAmount()) < 0) {
            throw new InsufficientFundsException(
                "Insufficient funds. Balance: " + balance + ", Required: " + amount
            );
        }
    }
    
    private void validateMinimumBalanceAfterWithdrawal(Money withdrawalAmount) {
        Money balanceAfter = balance.subtract(withdrawalAmount);
        Money minimumBalance = accountType.getMinimumBalance();
        
        if (balanceAfter.getAmount().compareTo(minimumBalance.getAmount()) < 0) {
            throw new MinimumBalanceViolationException(
                "Withdrawal would violate minimum balance requirement of " + minimumBalance
            );
        }
    }
    
    // Domain events management
    private void addDomainEvent(DomainEvent event) {
        this.domainEvents.add(event);
    }
    
    public List<DomainEvent> getDomainEvents() {
        return Collections.unmodifiableList(domainEvents);
    }
    
    public void clearDomainEvents() {
        this.domainEvents.clear();
    }
    
    // Getters
    public AccountId getId() { return id; }
    public CustomerId getOwnerId() { return ownerId; }
    public AccountType getAccountType() { return accountType; }
    public Money getBalance() { return balance; }
    public AccountStatus getStatus() { return status; }
    public LocalDateTime getCreatedAt() { return createdAt; }
    
    public List<Transaction> getTransactions() {
        return Collections.unmodifiableList(transactions);
    }
    
    public List<Transaction> getRecentTransactions(int limit) {
        return transactions.stream()
            .sorted(Comparator.comparing(Transaction::getTimestamp).reversed())
            .limit(limit)
            .toList();
    }
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof BankAccount)) return false;
        BankAccount that = (BankAccount) o;
        return id.equals(that.id);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(id);
    }
}

// Enums
enum AccountType {
    SAVINGS(Money.of(new java.math.BigDecimal("100"), "USD")),
    CHECKING(Money.of(new java.math.BigDecimal("0"), "USD")),
    BUSINESS(Money.of(new java.math.BigDecimal("1000"), "USD"));
    
    private final Money minimumBalance;
    
    AccountType(Money minimumBalance) {
        this.minimumBalance = minimumBalance;
    }
    
    public Money getMinimumBalance() {
        return minimumBalance;
    }
}

enum AccountStatus {
    ACTIVE, SUSPENDED, CLOSED
}
```

## 3. Domain Layer - Domain Service

### TransferService.java
```java
package com.bank.domain.account.service;

import com.bank.domain.account.model.*;
import com.bank.domain.account.repository.AccountRepository;
import com.bank.domain.account.exception.*;

/**
 * Domain Service - xử lý logic liên quan đến nhiều Aggregate
 */
public class TransferService {
    
    private final AccountRepository accountRepository;
    
    public TransferService(AccountRepository accountRepository) {
        this.accountRepository = accountRepository;
    }
    
    /**
     * Transfer money giữa hai accounts
     * Domain Service vì liên quan đến 2 Aggregates
     */
    public void transfer(AccountId fromAccountId, AccountId toAccountId, Money amount) {
        // Validate
        if (fromAccountId.equals(toAccountId)) {
            throw new IllegalArgumentException("Cannot transfer to the same account");
        }
        
        // Load both aggregates
        BankAccount fromAccount = accountRepository.findById(fromAccountId)
            .orElseThrow(() -> new AccountNotFoundException(fromAccountId));
        
        BankAccount toAccount = accountRepository.findById(toAccountId)
            .orElseThrow(() -> new AccountNotFoundException(toAccountId));
        
        // Validate currency
        if (!fromAccount.getBalance().getCurrency()
                .equals(toAccount.getBalance().getCurrency())) {
            throw new IllegalArgumentException("Cannot transfer between different currencies");
        }
        
        // Execute transfer - domain logic ở Entity
        fromAccount.debit(amount, toAccountId);
        toAccount.credit(amount, fromAccountId);
        
        // Repository sẽ save trong Application Service
    }
}
```

## 4. Domain Layer - Repository Interface

### AccountRepository.java
```java
package com.bank.domain.account.repository;

import com.bank.domain.account.model.*;
import com.bank.domain.customer.model.CustomerId;
import java.util.List;
import java.util.Optional;

/**
 * Repository interface - ở Domain layer
 * Implementation ở Infrastructure layer
 */
public interface AccountRepository {
    
    Optional<BankAccount> findById(AccountId id);
    
    List<BankAccount> findByOwnerId(CustomerId ownerId);
    
    List<BankAccount> findByStatus(AccountStatus status);
    
    void save(BankAccount account);
    
    void delete(BankAccount account);
    
    boolean exists(AccountId id);
}
```

## 5. Domain Layer - Domain Events

### DomainEvent.java (Base)
```java
package com.bank.domain.account.event;

import java.time.LocalDateTime;
import java.util.UUID;

public abstract class DomainEvent {
    private final String eventId;
    private final LocalDateTime occurredOn;
    
    protected DomainEvent() {
        this.eventId = UUID.randomUUID().toString();
        this.occurredOn = LocalDateTime.now();
    }
    
    public String getEventId() { return eventId; }
    public LocalDateTime getOccurredOn() { return occurredOn; }
}
```

### MoneyDeposited.java
```java
package com.bank.domain.account.event;

import com.bank.domain.account.model.*;

public class MoneyDeposited extends DomainEvent {
    private final AccountId accountId;
    private final Money amount;
    private final Money newBalance;
    
    public MoneyDeposited(AccountId accountId, Money amount, Money newBalance) {
        super();
        this.accountId = accountId;
        this.amount = amount;
        this.newBalance = newBalance;
    }
    
    public AccountId getAccountId() { return accountId; }
    public Money getAmount() { return amount; }
    public Money getNewBalance() { return newBalance; }
}
```

### MoneyTransferred.java
```java
package com.bank.domain.account.event;

import com.bank.domain.account.model.*;

public class MoneyTransferred extends DomainEvent {
    private final AccountId fromAccountId;
    private final AccountId toAccountId;
    private final Money amount;
    private final Money newBalance;
    
    public MoneyTransferred(AccountId fromAccountId, AccountId toAccountId, 
                          Money amount, Money newBalance) {
        super();
        this.fromAccountId = fromAccountId;
        this.toAccountId = toAccountId;
        this.amount = amount;
        this.newBalance = newBalance;
    }
    
    // Getters...
}
```

## 6. Application Layer - Commands

### CreateAccountCommand.java
```java
package com.bank.application.account.command;

import java.math.BigDecimal;

/**
 * Command - immutable DTO từ presentation layer
 */
public record CreateAccountCommand(
    String customerId,
    String accountType,
    BigDecimal initialDeposit,
    String currency
) {
    public CreateAccountCommand {
        // Validation ở đây nếu cần
        if (customerId == null || customerId.isBlank()) {
            throw new IllegalArgumentException("CustomerId is required");
        }
    }
}
```

### TransferMoneyCommand.java
```java
package com.bank.application.account.command;

import java.math.BigDecimal;

public record TransferMoneyCommand(
    String fromAccountId,
    String toAccountId,
    BigDecimal amount,
    String currency
) {
    public TransferMoneyCommand {
        if (fromAccountId == null || toAccountId == null) {
            throw new IllegalArgumentException("Account IDs are required");
        }
        if (amount == null || amount.compareTo(BigDecimal.ZERO) <= 0) {
            throw new IllegalArgumentException("Amount must be positive");
        }
    }
}
```

## 7. Application Layer - Application Service

### AccountApplicationService.java
```java
package com.bank.application.account.service;

import com.bank.domain.account.model.*;
import com.bank.domain.account.repository.AccountRepository;
import com.bank.domain.account.service.TransferService;
import com.bank.application.account.command.*;
import com.bank.infrastructure.messaging.EventPublisher;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

/**
 * Application Service - điều phối use cases
 * Không chứa business logic, chỉ orchestration
 */
@Service
@Transactional
public class AccountApplicationService {
    
    private final AccountRepository accountRepository;
    private final TransferService transferService;
    private final EventPublisher eventPublisher;
    
    public AccountApplicationService(AccountRepository accountRepository,
                                    TransferService transferService,
                                    EventPublisher eventPublisher) {
        this.accountRepository = accountRepository;
        this.transferService = transferService;
        this.eventPublisher = eventPublisher;
    }
    
    /**
     * Use Case: Open new bank account
     */
    public String openAccount(CreateAccountCommand command) {
        // 1. Convert command to domain objects
        AccountId accountId = AccountId.generate();
        CustomerId customerId = CustomerId.of(command.customerId());
        AccountType accountType = AccountType.valueOf(command.accountType());
        Money initialDeposit = Money.of(command.initialDeposit(), command.currency());
        
        // 2. Execute domain logic (ở Entity)
        BankAccount account = BankAccount.open(
            accountId, 
            customerId, 
            accountType, 
            initialDeposit
        );
        
        // 3. Persist
        accountRepository.save(account);
        
        // 4. Publish events
        publishEvents(account);
        
        return accountId.getValue();
    }
    
    /**
     * Use Case: Deposit money
     */
    public void depositMoney(DepositMoneyCommand command) {
        // 1. Load aggregate
        AccountId accountId = AccountId.of(command.accountId());
        BankAccount account = accountRepository.findById(accountId)
            .orElseThrow(() -> new AccountNotFoundException(accountId));
        
        // 2. Execute domain logic
        Money amount = Money.of(command.amount(), command.currency());
        account.deposit(amount, command.description());
        
        // 3. Save
        accountRepository.save(account);
        
        // 4. Publish events
        publishEvents(account);
    }
    
    /**
     * Use Case: Withdraw money
     */
    public void withdrawMoney(WithdrawMoneyCommand command) {
        // Similar pattern
        AccountId accountId = AccountId.of(command.accountId());
        BankAccount account = accountRepository.findById(accountId)
            .orElseThrow(() -> new AccountNotFoundException(accountId));
        
        Money amount = Money.of(command.amount(), command.currency());
        account.withdraw(amount, command.description());
        
        accountRepository.save(account);
        publishEvents(account);
    }
    
    /**
     * Use Case: Transfer money between accounts
     */
    public void transferMoney(TransferMoneyCommand command) {
        // 1. Convert to domain objects
        AccountId fromAccountId = AccountId.of(command.fromAccountId());
        AccountId toAccountId = AccountId.of(command.toAccountId());
        Money amount = Money.of(command.amount(), command.currency());
        
        // 2. Execute domain service (vì liên quan 2 aggregates)
        transferService.transfer(fromAccountId, toAccountId, amount);
        
        // 3. Load và save cả 2 accounts
        BankAccount fromAccount = accountRepository.findById(fromAccountId)
            .orElseThrow(() -> new AccountNotFoundException(fromAccountId));
        BankAccount toAccount = accountRepository.findById(toAccountId)
            .orElseThrow(() -> new AccountNotFoundException(toAccountId));
        
        accountRepository.save(fromAccount);
        accountRepository.save(toAccount);
        
        // 4. Publish events
        publishEvents(fromAccount);
        publishEvents(toAccount);
    }
    
    /**
     * Use Case: Close account
     */
    public void closeAccount(String accountIdStr) {
        AccountId accountId = AccountId.of(accountIdStr);
        BankAccount account = accountRepository.findById(accountId)
            .orElseThrow(() -> new AccountNotFoundException(accountId));
        
        account.close();
        
        accountRepository.save(account);
        publishEvents(account);
    }
    
    // Helper method
    private void publishEvents(BankAccount account) {
        account.getDomainEvents().forEach(eventPublisher::publish);
        account.clearDomainEvents();
    }
}
```

## 8. Application Layer - Query & DTO

### AccountQuery.java
```java
package com.bank.application.account.query;

import com.bank.domain.account.model.*;
import com.bank.domain.account.repository.AccountRepository;
import com.bank.domain.customer.model.CustomerId;
import org.springframework.stereotype.Service;

import java.util.List;

/**
 * Query Service - read-only operations
 * Có thể query trực tiếp database để tối ưu performance
 */
@Service
public class AccountQuery {
    
    private final AccountRepository accountRepository;
    
    public AccountQuery(AccountRepository accountRepository) {
        this.accountRepository = accountRepository;
    }
    
    public AccountDTO getAccount(String accountId) {
        BankAccount account = accountRepository.findById(AccountId.of(accountId))
            .orElseThrow(() -> new AccountNotFoundException(AccountId.of(accountId)));
        
        return AccountDTO.from(account);
    }
    
    public List<AccountDTO> getCustomerAccounts(String customerId) {
        List<BankAccount> accounts = accountRepository
            .findByOwnerId(CustomerId.of(customerId));
        
        return accounts.stream()
            .map(AccountDTO::from)
            .toList();
    }
}
```

### AccountDTO.java
```java
package com.bank.application.account.query;

import com.bank.domain.account.model.*;
import java.math.BigDecimal;
import java.time.LocalDateTime;

/**
 * DTO để trả về presentation layer
 */
public record AccountDTO(
    String accountId,
    String customerId,
    String accountType,
    BigDecimal balance,
    String currency,
    String status,
    LocalDateTime createdAt
) {
    public static AccountDTO from(BankAccount account) {
        return new AccountDTO(
            account.getId().getValue(),
            account.getOwnerId().getValue(),
            account.getAccountType().name(),
            account.getBalance().getAmount(),
            account.getBalance().getCurrency().getCurrencyCode(),
            account.getStatus().name(),
            account.getCreatedAt()
        );
    }
}
```

## 9. Infrastructure Layer - Repository Implementation

### JpaAccountRepository.java
```java
package com.bank.infrastructure.persistence.jpa;

import com.bank.domain.account.model.*;
import com.bank.domain.account.repository.AccountRepository;
import com.bank.domain.customer.model.CustomerId;
import org.springframework.stereotype.Repository;

import jakarta.persistence.EntityManager;
import jakarta.persistence.PersistenceContext;
import java.util.List;
import java.util.Optional;

/**
 * JPA implementation của AccountRepository
 */
@Repository
public class JpaAccountRepository implements AccountRepository {
    
    @PersistenceContext
    private EntityManager entityManager;
    
    private final AccountMapper mapper;
    
    public JpaAccountRepository(AccountMapper mapper) {
        this.mapper = mapper;
    }
    
    @Override
    public Optional<BankAccount> findById(AccountId id) {
        AccountEntity entity = entityManager.find(AccountEntity.class, id.getValue());
        return Optional.ofNullable(entity)
            .map(mapper::toDomain);
    }
    
    @Override
    public List<BankAccount> findByOwnerId(CustomerId ownerId) {
        List<AccountEntity> entities = entityManager
            .createQuery("SELECT a FROM AccountEntity a WHERE a.ownerId = :ownerId", 
                        AccountEntity.class)
            .setParameter("ownerId", ownerId.getValue())
            .getResultList();
        
        return entities.stream()
            .map(mapper::toDomain)
            .toList();
    }
    
    @Override
    public List<BankAccount> findByStatus(AccountStatus status) {
        // Implementation...
        return List.of();
    }
    
    @Override
    public void save(BankAccount account) {
        AccountEntity entity = mapper.toEntity(account);
        
        if (exists(account.getId())) {
            entityManager.merge(entity);
        } else {
            entityManager.persist(entity);
        }
    }
    
    @Override
    public void delete(BankAccount account) {
        AccountEntity entity = entityManager.find(
            AccountEntity.class, 
            account.getId().getValue()
        );
        if (entity != null) {
            entityManager.remove(entity);
        }
    }
    
    @Override
    public boolean exists(AccountId id) {
        Long count = entityManager
            .createQuery("SELECT COUNT(a) FROM AccountEntity a WHERE a.id = :id", Long.class)
            .setParameter("id", id.getValue())
            .getSingleResult();
        return count > 0;
    }
}
```

## 10. Interfaces Layer - REST Controller

### AccountController.java
```java
package com.bank.interfaces.rest;

import com.bank.application.account.service.AccountApplicationService;
import com.bank.application.account.command.*;
import com.bank.application.account.query.*;
import com.bank.interfaces.dto.*;
import org.springframework.http.*;
import org.springframework.web.bind.annotation.*;

import java.util.List;

/**
 * REST Controller - presentation layer
 */
@RestController
@RequestMapping("/api/accounts")
public class AccountController {
    
    private final AccountApplicationService applicationService;
    private final AccountQuery accountQuery;
    
    public AccountController(AccountApplicationService applicationService,
                            AccountQuery accountQuery) {
        this.applicationService = applicationService;
        this.accountQuery = accountQuery;
    }
    
    @PostMapping
    public ResponseEntity<CreateAccountResponse> createAccount(
            @RequestBody CreateAccountRequest request) {
        
        CreateAccountCommand command = new CreateAccountCommand(
            request.customerId(),
            request.accountType(),
            request.initialDeposit(),
            request.currency()
        );
        
        String accountId = applicationService.openAccount(command);
        
        return ResponseEntity
            .status(HttpStatus.CREATED)
            .body(new CreateAccountResponse(accountId));
    }
    
    @PostMapping("/{accountId}/deposit")
    public ResponseEntity<Void> deposit(
            @PathVariable String accountId,
            @RequestBody DepositRequest request) {
        
        DepositMoneyCommand command = new DepositMoneyCommand(
            accountId,
            request.amount(),
            request.currency(),
            request.description()
        );
        
        applicationService.depositMoney(command);
        
        return ResponseEntity.ok().build();
    }
    
    @PostMapping("/{accountId}/withdraw")
    public ResponseEntity<Void> withdraw(
            @PathVariable String accountId,
            @RequestBody WithdrawRequest request) {
        
        WithdrawMoneyCommand command = new WithdrawMoneyCommand(
            accountId,
            request.amount(),
            request.currency(),
            request.description()
        );
        
        applicationService.withdrawMoney(command);
        
        return ResponseEntity.ok().build();
    }
    
    @PostMapping("/transfer")
    public ResponseEntity<Void> transfer(
            @RequestBody TransferMoneyRequest request) {
        
        TransferMoneyCommand command = new TransferMoneyCommand(
            request.fromAccountId(),
            request.toAccountId(),
            request.amount(),
            request.currency()
        );
        
        applicationService.transferMoney(command);
        
        return ResponseEntity.ok().build();
    }
    
    @GetMapping("/{accountId}")
    public ResponseEntity<AccountDTO> getAccount(@PathVariable String accountId) {
        AccountDTO account = accountQuery.getAccount(accountId);
        return ResponseEntity.ok(account);
    }
    
    @GetMapping("/customer/{customerId}")
    public ResponseEntity<List<AccountDTO>> getCustomerAccounts(
            @PathVariable String customerId) {
        List<AccountDTO> accounts = accountQuery.getCustomerAccounts(customerId);
        return ResponseEntity.ok(accounts);
    }
    
    @DeleteMapping("/{accountId}")
    public ResponseEntity<Void> closeAccount(@PathVariable String accountId) {
        applicationService.closeAccount(accountId);
        return ResponseEntity.noContent().build();
    }
}
```

## Tổng kết

### Các điểm chính trong thiết kế DDD:

1. **Separation of Concerns**: 
   - Domain layer: business logic
   - Application layer: orchestration
   - Infrastructure: technical details
   - Interfaces: presentation

2. **Rich Domain Model**:
   - Entity có business methods
   - Value Object immutable
   - Aggregate Root bảo vệ invariants

3. **Ubiquitous Language**:
   - `deposit()`, `withdraw()`, `transfer()`
   - `BankAccount`, `Money`, `Transaction`
   - Tên class/method phản ánh domain

4. **Aggregate Pattern**:
   - `BankAccount` là Aggregate Root
   - `Transaction` là Entity bên trong
   - Chỉ truy cập qua Root

5. **Domain Events**:
   - `MoneyDeposited`, `MoneyTransferred`
   - Decouple các bounded context

6. **Repository Pattern**:
   - Interface ở Domain layer
   - Implementation ở Infrastructure
   - Che giấu persistence details

## Nguồn tham khảo

!!! info "Tài liệu tham khảo"
    - **Vaughn Vernon** - [Implementing Domain-Driven Design](https://www.amazon.com/Implementing-Domain-Driven-Design-Vaughn-Vernon/dp/0321834577) - Chapter 4-11: Architecture and Implementation
    - **Eric Evans** - [Domain-Driven Design Reference](http://domainlanguage.com/ddd/reference/) - Pattern summaries
    - **Microsoft** - [.NET Microservices: Architecture for Containerized .NET Applications](https://docs.microsoft.com/en-us/dotnet/architecture/microservices/) - DDD patterns and practices

---

[← Quay lại Building Blocks](building-blocks.md)

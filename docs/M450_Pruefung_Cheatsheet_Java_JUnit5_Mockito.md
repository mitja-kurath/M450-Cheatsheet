# **M450 Prüfung - Praktisches Cheatsheet (Java: JUnit 5 & Mockito)**

## **Inhaltsverzeichnis**
1. [AAA-Pattern](#aaa-pattern)  
2. [JUnit 5 Grundlagen](#junit-5-grundlagen)  
3. [Assert Methoden](#assert-methoden)  
4. [Test Doubles - Eigene Implementierung](#test-doubles---eigene-implementierung)  
5. [Mockito Framework](#mockito-framework)  
6. [Äquivalenzklassen](#äquivalenzklassen)  
7. [Grenzwertanalyse](#grenzwertanalyse)  
8. [Zustandsbasierte Tests](#zustandsbasierte-tests)  
9. [Exception Testing](#exception-testing)  
10. [Best Practices](#best-practices)  

---

## **AAA-Pattern**

**Arrange - Act - Assert**: Strukturierung jedes Tests in 3 Phasen

```java
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;

class CalculatorTests {

    @Test
    void methodName_scenario_expectedBehavior() {
        // Arrange - Setup: Objekte erstellen, Daten vorbereiten
        Calculator calculator = new Calculator();
        int a = 5;
        int b = 3;
        int expected = 8;

        // Act - Ausführung: Die zu testende Methode aufrufen
        int result = calculator.add(a, b);

        // Assert - Überprüfung: Ergebnis validieren
        assertEquals(expected, result);
    }
}
```

### **Naming Convention**

```java
// methodName_stateUnderTest_expectedBehavior
// Beispiele:
// add_twoPositiveNumbers_returnsSum
// login_invalidPassword_returnsFalse
// borrowBook_availableBook_returnsTrue
```

---

## **JUnit 5 Grundlagen**

### **Test Class Setup**

```java
import org.junit.jupiter.api.*;

import static org.junit.jupiter.api.Assertions.*;

class CalculatorTests {

    private Calculator calculator;

    // Wird vor JEDEM Test ausgeführt
    @BeforeEach
    void setup() {
        calculator = new Calculator();
    }

    // Wird nach JEDEM Test ausgeführt
    @AfterEach
    void cleanup() {
        calculator = null;
    }

    // Wird 1x vor allen Tests ausgeführt
    @BeforeAll
    static void classSetup() {
        // Einmalige Initialisierung
    }

    // Wird 1x nach allen Tests ausgeführt
    @AfterAll
    static void classCleanup() {
        // Aufräumarbeiten
    }

    @Test
    void myTest() {
        // Test Code
    }
}
```

---

## **Assert Methoden**

> JUnit 5: `org.junit.jupiter.api.Assertions.*`

### **Gleichheit**

```java
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;

class AssertionTests {

    @Test
    void equalityAssertions() {
        int result = 5;
        int expected = 5;

        // Gleichheit prüfen
        assertEquals(5, result);
        assertEquals(expected, result);

        // Doubles mit Toleranz
        double pi = 3.14159;
        assertEquals(3.14, pi, 0.01);

        // Ungleichheit prüfen
        assertNotEquals(0, result);
        assertNotEquals(999, result);
    }
}
```

### **Boolean**

```java
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;

class BooleanAssertionTests {

    @Test
    void booleanAssertions() {
        boolean condition = true;

        assertTrue(condition);
        assertFalse(!condition);
    }
}
```

### **Null Checks**

```java
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;

class NullAssertionTests {

    @Test
    void nullAssertions() {
        Object result = null;
        Object user = new Object();

        // Null prüfen
        assertNull(result);

        // Nicht Null prüfen
        assertNotNull(user);
    }
}
```

### **Type Checks**

```java
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;

class TypeAssertionTests {

    @Test
    void typeAssertions() {
        Object obj = new Customer();

        // Typ prüfen
        assertInstanceOf(Customer.class, obj);

        // Typ verneinen
        assertFalse(obj instanceof Admin);
    }

    static class Customer {}
    static class Admin {}
}
```

### **String Assertions**

```java
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;

class StringAssertionTests {

    @Test
    void stringAssertions() {
        String text = "Hello World";

        // Enthält
        assertTrue(text.contains("World"));

        // Beginnt mit
        assertTrue(text.startsWith("Hello"));

        // Endet mit
        assertTrue(text.endsWith("World"));

        // Regex Match
        assertTrue(text.matches("\\w+ \\w+"));
    }
}
```

### **Collection Assertions**

```java
import org.junit.jupiter.api.Test;

import java.util.List;

import static org.junit.jupiter.api.Assertions.*;

class CollectionAssertionTests {

    @Test
    void collectionAssertions() {
        List<Integer> list = List.of(1, 2, 3, 4, 5);

        // Count prüfen
        assertEquals(5, list.size());

        // Nicht leer
        assertFalse(list.isEmpty());

        // Enthält Element
        assertTrue(list.contains(3));

        // Gleiche Elemente, gleiche Reihenfolge
        List<Integer> expected = List.of(1, 2, 3);
        List<Integer> actual = List.of(1, 2, 3);
        assertIterableEquals(expected, actual);

        // Reihenfolge egal (Vergleich über Set)
        var set1 = new java.util.HashSet<>(List.of(1, 2, 3));
        var set2 = new java.util.HashSet<>(List.of(3, 1, 2));
        assertEquals(set1, set2);
    }
}
```

### **Numerische Assertions**

```java
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;

class NumericAssertionTests {

    @Test
    void numericAssertions() {
        int value = 5;

        assertTrue(value > 3);
        assertTrue(value < 10);
        assertTrue(value >= 5);
        assertTrue(value <= 5);

        // In Range
        assertTrue(value >= 1 && value <= 10);
    }
}
```

---

## **Test Doubles - Eigene Implementierung**

### **Fake Repository**

```java
import java.util.ArrayList;
import java.util.List;

interface UserRepository {
    User getById(int id);
    void add(User user);
    void update(User user);
    void delete(int id);
    List<User> getAll();
}

class FakeUserRepository implements UserRepository {
    private final List<User> users = new ArrayList<>();
    private int nextId = 1;

    // Tracking für Tests
    public final List<User> addedUsers = new ArrayList<>();
    public final List<User> updatedUsers = new ArrayList<>();
    public final List<Integer> deletedIds = new ArrayList<>();

    @Override
    public User getById(int id) {
        return users.stream().filter(u -> u.id == id).findFirst().orElse(null);
    }

    @Override
    public void add(User user) {
        user.id = nextId++;
        users.add(user);
        addedUsers.add(user);
    }

    @Override
    public void update(User user) {
        User existing = getById(user.id);
        if (existing != null) {
            users.remove(existing);
            users.add(user);
            updatedUsers.add(user);
        }
    }

    @Override
    public void delete(int id) {
        User user = getById(id);
        if (user != null) {
            users.remove(user);
            deletedIds.add(id);
        }
    }

    @Override
    public List<User> getAll() {
        return new ArrayList<>(users);
    }

    // Helper für Tests
    public void addTestData(User... seed) {
        for (User u : seed) users.add(u);
    }
}

class User {
    int id;
    String name;
    String email;
    boolean isActive = true;
}
```

### **Fake Service**

```java
import java.util.ArrayList;
import java.util.List;

interface EmailService {
    void sendEmail(String to, String subject, String body);
    void sendEmailWithAttachment(String to, String subject, String body, byte[] attachment);
}

class FakeEmailService implements EmailService {
    public final List<EmailRecord> sentEmails = new ArrayList<>();
    public boolean throwException = false;
    public int callCount = 0;

    @Override
    public void sendEmail(String to, String subject, String body) {
        callCount++;

        if (throwException) {
            throw new IllegalStateException("Email service unavailable");
        }

        sentEmails.add(new EmailRecord(to, subject, body, null));
    }

    @Override
    public void sendEmailWithAttachment(String to, String subject, String body, byte[] attachment) {
        callCount++;
        sentEmails.add(new EmailRecord(to, subject, body, attachment));
    }

    public boolean wasEmailSentTo(String email) {
        return sentEmails.stream().anyMatch(e -> e.to().equals(email));
    }

    public EmailRecord getLastEmail() {
        return sentEmails.isEmpty() ? null : sentEmails.get(sentEmails.size() - 1);
    }
}

record EmailRecord(String to, String subject, String body, byte[] attachment) {}
```

### **Test mit eigenem Fake**

```java
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;

class UserServiceTests {

    @Test
    void registerUser_validData_addsUserAndSendsEmail() {
        // Arrange
        FakeUserRepository fakeRepo = new FakeUserRepository();
        FakeEmailService fakeEmail = new FakeEmailService();
        UserService service = new UserService(fakeRepo, fakeEmail);

        User newUser = new User();
        newUser.name = "John Doe";
        newUser.email = "john@test.com";

        // Act
        service.registerUser(newUser);

        // Assert
        assertEquals(1, fakeRepo.addedUsers.size());
        assertEquals("John Doe", fakeRepo.addedUsers.get(0).name);

        assertEquals(1, fakeEmail.sentEmails.size());
        assertEquals("john@test.com", fakeEmail.sentEmails.get(0).to());
        assertTrue(fakeEmail.sentEmails.get(0).subject().contains("Welcome"));
    }
}

class UserService {
    private final UserRepository repo;
    private final EmailService email;

    UserService(UserRepository repo, EmailService email) {
        this.repo = repo;
        this.email = email;
    }

    void registerUser(User user) {
        repo.add(user);
        email.sendEmail(user.email, "Welcome", "Hello " + user.name);
    }

    void updateUserEmail(int id, String newEmail) {
        User u = new User();
        u.id = id;
        u.email = newEmail;
        repo.update(u);
    }
}
```

---

## **Mockito Framework**

### **Basis Setup**

```java
import org.junit.jupiter.api.Test;

import static org.mockito.Mockito.*;

class UserServiceMockitoTests {

    @Test
    void basicMockExample() {
        // Mock erstellen
        UserRepository mockRepo = mock(UserRepository.class);

        // Mock in Service injizieren
        UserService service = new UserService(mockRepo, (to, subject, body) -> {});

        // service.doSomething();
    }
}
```

### **Setup - Return Values**

```java
import org.junit.jupiter.api.Test;

import static org.mockito.Mockito.*;
import static org.junit.jupiter.api.Assertions.*;

class MockitoReturnTests {

    @Test
    void setupReturnValues() {
        // Arrange
        UserRepository mockRepo = mock(UserRepository.class);

        // Spezifischer Parameter
        User john = new User();
        john.id = 1;
        john.name = "John";
        when(mockRepo.getById(1)).thenReturn(john);

        // Beliebiger Parameter
        User def = new User();
        def.id = 999;
        def.name = "Default";
        when(mockRepo.getById(anyInt())).thenReturn(def);

        // Conditional Setup (ArgumentMatcher)
        when(mockRepo.getById(intThat(id -> id > 0)))
                .thenAnswer(inv -> {
                    int id = inv.getArgument(0);
                    User u = new User();
                    u.id = id;
                    u.name = "User" + id;
                    return u;
                });

        when(mockRepo.getById(intThat(id -> id <= 0))).thenReturn(null);

        // Act
        User u1 = mockRepo.getById(5);
        User u2 = mockRepo.getById(-1);

        // Assert
        assertEquals("User5", u1.name);
        assertNull(u2);
    }
}
```

### **Setup - Collections**

```java
import org.junit.jupiter.api.Test;

import java.util.List;

import static org.mockito.Mockito.*;
import static org.junit.jupiter.api.Assertions.*;

class MockitoCollectionTests {

    @Test
    void setupCollections() {
        UserRepository mockRepo = mock(UserRepository.class);

        List<User> users = List.of(
                user(1, "John"),
                user(2, "Jane"),
                user(3, "Bob")
        );

        when(mockRepo.getAll()).thenReturn(users);

        // Act
        List<User> result = mockRepo.getAll();

        // Assert
        assertFalse(result.isEmpty());
    }

    private static User user(int id, String name) {
        User u = new User();
        u.id = id;
        u.name = name;
        return u;
    }
}
```

### **Setup - Exceptions**

```java
import org.junit.jupiter.api.Test;

import static org.mockito.Mockito.*;
import static org.junit.jupiter.api.Assertions.*;

class MockitoExceptionTests {

    @Test
    void setupExceptions() {
        UserRepository mockRepo = mock(UserRepository.class);

        when(mockRepo.getById(anyInt())).thenThrow(new IllegalStateException());
        when(mockRepo.getById(999)).thenThrow(new IllegalArgumentException("User not found"));

        // Act & Assert
        IllegalArgumentException ex = assertThrows(
                IllegalArgumentException.class,
                () -> mockRepo.getById(999)
        );

        assertTrue(ex.getMessage().contains("not found"));
    }
}
```

### **Verify - Methodenaufrufe prüfen**

```java
import org.junit.jupiter.api.Test;

import static org.mockito.Mockito.*;

class MockitoVerifyTests {

    @Test
    void verifyMethodCalls() {
        // Arrange
        UserRepository mockRepo = mock(UserRepository.class);
        EmailService mockEmail = mock(EmailService.class);

        DeletionService service = new DeletionService(mockRepo, mockEmail);

        // Act
        service.deleteUser(1);

        // Assert - Verify
        verify(mockRepo, times(1)).delete(1);
        verify(mockEmail, never()).sendEmail(anyString(), anyString(), anyString());
        verify(mockRepo, atLeastOnce()).delete(anyInt());
    }

    static class DeletionService {
        private final UserRepository repo;
        private final EmailService email;

        DeletionService(UserRepository repo, EmailService email) {
            this.repo = repo;
            this.email = email;
        }

        void deleteUser(int id) {
            repo.delete(id);
        }
    }
}
```

### **Verify mit ArgumentMatcher (Komplexe Parameter)**

```java
import org.junit.jupiter.api.Test;

import static org.mockito.Mockito.*;

class MockitoComplexVerifyTests {

    @Test
    void verifyWithComplexParameters() {
        UserRepository mockRepo = mock(UserRepository.class);

        UserService service = new UserService(mockRepo, (to, subject, body) -> {});
        service.updateUserEmail(1, "new@email.com");

        verify(mockRepo, times(1)).update(argThat(u ->
                u.id == 1 && "new@email.com".equals(u.email)
        ));
    }
}
```

### **Times Options**

```java
import static org.mockito.Mockito.*;

class TimesOptionsCheat {
    void examples(UserRepository mockRepo) {
        verify(mockRepo, times(1)).getById(anyInt());
        verify(mockRepo, times(3)).getAll();

        verify(mockRepo, never()).delete(anyInt());

        verify(mockRepo, atLeastOnce()).getAll();
        verify(mockRepo, atLeast(2)).getAll();
        verify(mockRepo, atMost(5)).getAll();
    }
}
```

### **any() vs argThat()**

```java
import static org.mockito.Mockito.*;

class AnyVsArgThatCheat {

    void examples(UserRepository mockRepo) {
        // anyInt() - beliebiger Wert dieses Typs
        when(mockRepo.getById(anyInt())).thenReturn(new User());

        // argThat / intThat - mit Bedingung
        when(mockRepo.getById(intThat(id -> id > 0))).thenReturn(new User());
        when(mockRepo.getById(intThat(id -> id <= 0))).thenReturn(null);

        // Verify mit argThat
        verify(mockRepo).update(argThat(u -> u.isActive));
    }
}
```

### **Komplettes Mockito Beispiel**

```java
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.*;

class OrderServiceTests {

    @Test
    void createOrder_validData_savesOrderAndSendsConfirmation() {
        // Arrange
        OrderRepository mockOrderRepo = mock(OrderRepository.class);
        EmailService mockEmailService = mock(EmailService.class);
        PaymentService mockPaymentService = mock(PaymentService.class);

        Order order = new Order();
        order.customerId = 1;
        order.total = 100.00;

        when(mockPaymentService.processPayment(anyDouble())).thenReturn(true);

        OrderService service = new OrderService(mockOrderRepo, mockEmailService, mockPaymentService);

        // Act
        boolean result = service.createOrder(order, "customer@test.com");

        // Assert
        assertTrue(result);

        verify(mockPaymentService, times(1)).processPayment(100.00);
        verify(mockOrderRepo, times(1)).save(argThat(o ->
                o.customerId == 1 && o.total == 100.00
        ));
        verify(mockEmailService, times(1)).sendEmail(
                eq("customer@test.com"),
                argThat(s -> s.contains("Order Confirmation")),
                anyString()
        );
    }

    interface OrderRepository { void save(Order order); }
    interface PaymentService { boolean processPayment(double amount); }

    static class Order {
        int customerId;
        double total;
    }

    static class OrderService {
        private final OrderRepository repo;
        private final EmailService email;
        private final PaymentService payment;

        OrderService(OrderRepository repo, EmailService email, PaymentService payment) {
            this.repo = repo;
            this.email = email;
            this.payment = payment;
        }

        boolean createOrder(Order order, String emailAddress) {
            boolean paid = payment.processPayment(order.total);
            if (!paid) return false;

            repo.save(order);
            email.sendEmail(emailAddress, "Order Confirmation", "Thanks!");
            return true;
        }
    }
}
```

---

## **Äquivalenzklassen**

### **Konzept**

Eingabewerte in Klassen gruppieren, die gleich behandelt werden.

```java
// Beispiel: Altersprüfung (18-65 Jahre)
//
// Äquivalenzklassen:
// 1. Ungültig: < 18 (z.B. 0, 10, 17)
// 2. Gültig: 18-65 (z.B. 18, 30, 65)
// 3. Ungültig: > 65 (z.B. 66, 100)
```

### **Implementierung**

```java
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;

class AgeValidator {
    boolean isValidAge(int age) {
        return age >= 18 && age <= 65;
    }
}

class AgeValidatorTests {
    private AgeValidator validator;

    @BeforeEach
    void setup() {
        validator = new AgeValidator();
    }

    // Äquivalenzklasse 1: Ungültig - Zu jung
    @Test
    void isValidAge_belowMinimum_returnsFalse() {
        boolean result = validator.isValidAge(10);
        assertFalse(result);
    }

    // Äquivalenzklasse 2: Gültig
    @Test
    void isValidAge_withinRange_returnsTrue() {
        boolean result = validator.isValidAge(30);
        assertTrue(result);
    }

    // Äquivalenzklasse 3: Ungültig - Zu alt
    @Test
    void isValidAge_aboveMaximum_returnsFalse() {
        boolean result = validator.isValidAge(70);
        assertFalse(result);
    }
}
```

### **Mehrere Dimensionen**

```java
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;

enum CustomerType { STANDARD, PREMIUM, VIP }

class DiscountCalculator {
    double calculate(CustomerType type, double amount) {
        return switch (type) {
            case STANDARD -> 0.0;
            case PREMIUM -> (amount >= 100 && amount <= 500) ? amount * 0.10 : 0.0;
            case VIP -> (amount > 500) ? amount * 0.20 : amount * 0.10;
        };
    }
}

class DiscountCalculatorTests {

    @Test
    void calculateDiscount_standardCustomer_smallOrder_noDiscount() {
        DiscountCalculator calculator = new DiscountCalculator();
        double discount = calculator.calculate(CustomerType.STANDARD, 50);
        assertEquals(0.0, discount);
    }

    @Test
    void calculateDiscount_premiumCustomer_mediumOrder_gets10Percent() {
        DiscountCalculator calculator = new DiscountCalculator();
        double discount = calculator.calculate(CustomerType.PREMIUM, 200);
        assertEquals(20.0, discount);
    }

    @Test
    void calculateDiscount_vipCustomer_largeOrder_gets20Percent() {
        DiscountCalculator calculator = new DiscountCalculator();
        double discount = calculator.calculate(CustomerType.VIP, 1000);
        assertEquals(200.0, discount);
    }
}
```

---

## **Grenzwertanalyse**

### **Konzept**

Teste die Grenzen zwischen Äquivalenzklassen plus direkte Nachbarn.

```java
// Beispiel: Alter 18-65
// Grenzwerte:
// - 17 (ungültig, direkt vor Untergrenze)
// - 18 (gültig, Untergrenze)
// - 19 (gültig, direkt nach Untergrenze)
// - 64 (gültig, direkt vor Obergrenze)
// - 65 (gültig, Obergrenze)
// - 66 (ungültig, direkt nach Obergrenze)
```

### **Implementierung**

```java
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;

class AgeValidatorBoundaryTests {
    private AgeValidator validator;

    @BeforeEach
    void setup() {
        validator = new AgeValidator();
    }

    // Untergrenze Tests
    @Test
    void isValidAge_oneLessThanMinimum_returnsFalse() {
        assertFalse(validator.isValidAge(17));
    }

    @Test
    void isValidAge_atMinimum_returnsTrue() {
        assertTrue(validator.isValidAge(18));
    }

    @Test
    void isValidAge_oneMoreThanMinimum_returnsTrue() {
        assertTrue(validator.isValidAge(19));
    }

    // Obergrenze Tests
    @Test
    void isValidAge_oneLessThanMaximum_returnsTrue() {
        assertTrue(validator.isValidAge(64));
    }

    @Test
    void isValidAge_atMaximum_returnsTrue() {
        assertTrue(validator.isValidAge(65));
    }

    @Test
    void isValidAge_oneMoreThanMaximum_returnsFalse() {
        assertFalse(validator.isValidAge(66));
    }
}
```

### **Komplexes Beispiel mit Gebühren**

```java
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;

class LateFeeCalculator {
    double calculate(int daysLate) {
        if (daysLate < 0) throw new IllegalArgumentException("Days cannot be negative");

        if (daysLate == 0) return 0.0;
        if (daysLate <= 7) return daysLate * 1.0;
        if (daysLate <= 30) return daysLate * 2.0;
        return daysLate * 5.0;
    }
}

class LateFeeCalculatorTests {
    private LateFeeCalculator calculator;

    @BeforeEach
    void setup() {
        calculator = new LateFeeCalculator();
    }

    // Grenze 1: 0 Tage
    @Test
    void calculate_zeroDays_returnsZero() {
        assertEquals(0.0, calculator.calculate(0));
    }

    @Test
    void calculate_oneDay_returnsOneFranc() {
        assertEquals(1.0, calculator.calculate(1));
    }

    // Grenze 2: 7 Tage (Übergang 1 CHF → 2 CHF)
    @Test
    void calculate_sevenDays_returnsSevenFrancs() {
        assertEquals(7.0, calculator.calculate(7));
    }

    @Test
    void calculate_eightDays_returnsSixteenFrancs() {
        assertEquals(16.0, calculator.calculate(8));
    }

    // Grenze 3: 30 Tage (Übergang 2 CHF → 5 CHF)
    @Test
    void calculate_thirtyDays_returnsSixtyFrancs() {
        assertEquals(60.0, calculator.calculate(30));
    }

    @Test
    void calculate_thirtyOneDays_returns155Francs() {
        assertEquals(155.0, calculator.calculate(31));
    }

    // Negativer Grenzwert
    @Test
    void calculate_negativeDays_throwsException() {
        assertThrows(IllegalArgumentException.class, () -> calculator.calculate(-1));
    }
}
```

---

## **Zustandsbasierte Tests**

### **State Machine Beispiel**

```java
enum OrderState {
    NEW,
    CONFIRMED,
    PAID,
    SHIPPED,
    DELIVERED,
    CANCELLED
}

class Order {
    int id;
    private OrderState state = OrderState.NEW;

    OrderState getState() {
        return state;
    }

    void confirm() {
        if (state != OrderState.NEW) throw new IllegalStateException("Can only confirm new orders");
        state = OrderState.CONFIRMED;
    }

    void pay() {
        if (state != OrderState.CONFIRMED) throw new IllegalStateException("Can only pay confirmed orders");
        state = OrderState.PAID;
    }

    void ship() {
        if (state != OrderState.PAID) throw new IllegalStateException("Can only ship paid orders");
        state = OrderState.SHIPPED;
    }

    void deliver() {
        if (state != OrderState.SHIPPED) throw new IllegalStateException("Can only deliver shipped orders");
        state = OrderState.DELIVERED;
    }

    void cancel() {
        if (state == OrderState.DELIVERED) throw new IllegalStateException("Cannot cancel delivered orders");
        state = OrderState.CANCELLED;
    }
}
```

### **Zustandsübergänge testen**

```java
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;

class OrderStateTests {

    // Gültige Übergänge

    @Test
    void order_newToConfirmed_stateChangesCorrectly() {
        // Arrange
        Order order = new Order();
        assertEquals(OrderState.NEW, order.getState());

        // Act
        order.confirm();

        // Assert
        assertEquals(OrderState.CONFIRMED, order.getState());
    }

    @Test
    void order_confirmedToPaid_stateChangesCorrectly() {
        Order order = new Order();
        order.confirm();

        order.pay();

        assertEquals(OrderState.PAID, order.getState());
    }

    @Test
    void order_paidToShipped_stateChangesCorrectly() {
        Order order = new Order();
        order.confirm();
        order.pay();

        order.ship();

        assertEquals(OrderState.SHIPPED, order.getState());
    }

    @Test
    void order_shippedToDelivered_stateChangesCorrectly() {
        Order order = new Order();
        order.confirm();
        order.pay();
        order.ship();

        order.deliver();

        assertEquals(OrderState.DELIVERED, order.getState());
    }

    // Ungültige Übergänge

    @Test
    void order_newToShipped_throwsException() {
        Order order = new Order();

        IllegalStateException ex = assertThrows(IllegalStateException.class, order::ship);
        assertTrue(ex.getMessage().contains("paid"));
    }

    @Test
    void order_cancelDelivered_throwsException() {
        Order order = new Order();
        order.confirm();
        order.pay();
        order.ship();
        order.deliver();

        IllegalStateException ex = assertThrows(IllegalStateException.class, order::cancel);
        assertTrue(ex.getMessage().contains("delivered"));
    }

    // Kompletter Workflow

    @Test
    void order_completeWorkflow_allStatesCorrect() {
        Order order = new Order();

        assertEquals(OrderState.NEW, order.getState());

        order.confirm();
        assertEquals(OrderState.CONFIRMED, order.getState());

        order.pay();
        assertEquals(OrderState.PAID, order.getState());

        order.ship();
        assertEquals(OrderState.SHIPPED, order.getState());

        order.deliver();
        assertEquals(OrderState.DELIVERED, order.getState());
    }
}
```

### **Zustandsbaum visualisieren**

```
State Transition Diagram:

    NEW ──────> CONFIRMED ──────> PAID ──────> SHIPPED ──────> DELIVERED
     │              │              │              │
     │              │              │              │
     └──> CANCEL <──┴──> CANCEL <──┴──> CANCEL <──┘

Allowed Transitions:
- NEW → CONFIRMED
- CONFIRMED → PAID
- PAID → SHIPPED
- SHIPPED → DELIVERED
- NEW/CONFIRMED/PAID/SHIPPED → CANCELLED

Forbidden Transitions:
- NEW → PAID/SHIPPED/DELIVERED
- DELIVERED → CANCELLED
- etc.
```

---

## **Exception Testing**

### **Methode 1: assertThrows (Empfohlen in JUnit 5)**

```java
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;

class ExceptionTests {

    @Test
    void method_invalidInput_throwsException() {
        Calculator calculator = new Calculator();

        ArithmeticException ex = assertThrows(
                ArithmeticException.class,
                () -> calculator.divide(10, 0)
        );

        // Optional: Message prüfen
        assertNotNull(ex.getMessage());
    }
}
```

### **Methode 2: Try-Catch (wenn du Details prüfen willst)**

```java
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;

class ExceptionTryCatchTests {

    @Test
    void method_invalidInput_tryCatch() {
        Calculator calculator = new Calculator();
        boolean exceptionThrown = false;

        try {
            calculator.divide(10, 0);
            fail("Expected ArithmeticException was not thrown");
        } catch (ArithmeticException ex) {
            exceptionThrown = true;
            assertNotNull(ex.getMessage());
        }

        assertTrue(exceptionThrown);
    }
}
```

### **Exception mit Mockito**

```java
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.*;

class ServiceRepositoryExceptionTest {

    @Test
    void service_repositoryThrowsException_handlesGracefully() {
        UserRepository mockRepo = mock(UserRepository.class);
        when(mockRepo.getById(anyInt())).thenThrow(new RuntimeException("DB down"));

        UserService service = new UserService(mockRepo, (to, subject, body) -> {});

        User result = null;
        try {
            result = mockRepo.getById(1);
        } catch (RuntimeException ignored) {
            // Simulierter Graceful-Handling-Style im Service
        }

        assertNull(result);
    }
}
```

### **Multiple Exception Types**

```java
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;

class UserValidator {
    void validate(User user) {
        if (user == null) throw new NullPointerException("user");
        if (user.name == null || user.name.isBlank()) throw new IllegalArgumentException("Name required");
        if (user.email == null || !user.email.contains("@")) throw new java.util.regex.PatternSyntaxException("Email", user.email, 0);
    }
}

class MultipleExceptionTests {

    @Test
    void validator_differentInvalidInputs_throwsCorrectExceptions() {
        UserValidator validator = new UserValidator();

        // Null Input
        NullPointerException npe = assertThrows(NullPointerException.class, () -> validator.validate(null));
        assertTrue(npe.getMessage().contains("user"));

        // Empty Name
        IllegalArgumentException iae = assertThrows(IllegalArgumentException.class, () -> {
            User u = new User();
            u.name = "";
            u.email = "john@test.com";
            validator.validate(u);
        });
        assertTrue(iae.getMessage().contains("Name"));

        // Invalid Email
        assertThrows(java.util.regex.PatternSyntaxException.class, () -> {
            User u = new User();
            u.name = "John";
            u.email = "invalid";
            validator.validate(u);
        });
    }
}
```

---

## **Best Practices**

### **Test Naming**

```java
// ✅ GUTE Namen (selbsterklärend)
@Test
void add_twoPositiveNumbers_returnsSum()

@Test
void login_invalidPassword_returnsFalse()

@Test
void getUser_nonExistentId_returnsNull()

// ❌ SCHLECHTE Namen
@Test
void test1()

@Test
void addTest()

@Test
void testUserLogin()
```

### **Ein Assert pro Test (wenn möglich)**

```java
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;

class OneAssertTests {

    @Test
    void add_twoPositiveNumbers_returnsCorrectSum() {
        Calculator calculator = new Calculator();
        int result = calculator.add(5, 3);
        assertEquals(8, result);
    }

    @Test
    void add_twoPositiveNumbers_doesNotReturnZero() {
        Calculator calculator = new Calculator();
        int result = calculator.add(5, 3);
        assertNotEquals(0, result);
    }

    // ⚠️ AKZEPTABEL - Logisch zusammenhängend
    @Test
    void createUser_validData_userHasCorrectProperties() {
        User user = new User();
        user.id = 1;
        user.name = "John";
        user.email = "john@test.com";

        assertNotNull(user);
        assertEquals("John", user.name);
        assertEquals("john@test.com", user.email);
        assertTrue(user.id > 0);
    }
}
```

### **Test Isolation**

```java
import org.junit.jupiter.api.*;

import static org.junit.jupiter.api.Assertions.*;

class IsolatedTests {
    private Calculator calculator;

    @BeforeEach
    void setup() {
        // Frische Instanz für jeden Test
        calculator = new Calculator();
    }

    @Test
    void test1() {
        calculator.add(5, 3);
        // beeinflusst NICHT Test2
        assertTrue(true);
    }

    @Test
    void test2() {
        // startet mit frischer Instanz
        assertEquals(4, calculator.add(2, 2));
    }
}
```

### **Arrange-Act-Assert klar trennen**

```java
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;

class AAAStyleTests {

    @Test
    void test_clearStructure() {
        // Arrange
        FakeUserRepository repository = new FakeUserRepository();
        UserService service = new UserService(repository, (to, subject, body) -> {});
        User user = new User();
        user.name = "John";

        // Act
        service.registerUser(user);

        // Assert
        assertEquals(1, repository.addedUsers.size());
    }
}
```

### **Aussagekräftige Assert Messages**

```java
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;

class AssertMessageTests {

    @Test
    void test_withMessage() {
        Calculator calculator = new Calculator();
        int result = calculator.add(5, 3);

        assertEquals(8, result, "Expected 5 + 3 = 8, but got " + result);
    }

    @Test
    void test_complexAssertion() {
        User user = new User();
        user.isActive = true;
        boolean isVerified = true;

        assertTrue(user.isActive && isVerified,
                "User should be active and verified. IsActive: " + user.isActive + ", IsVerified: " + isVerified);
    }
}
```

### **Test Data Builders**

```java
class UserBuilder {
    private String name = "Default User";
    private String email = "default@test.com";
    private boolean isActive = true;

    UserBuilder withName(String name) {
        this.name = name;
        return this;
    }

    UserBuilder withEmail(String email) {
        this.email = email;
        return this;
    }

    UserBuilder inactive() {
        this.isActive = false;
        return this;
    }

    User build() {
        User u = new User();
        u.name = name;
        u.email = email;
        u.isActive = isActive;
        return u;
    }
}

import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;

class BuilderTests {

    @Test
    void test_withBuilder() {
        User user = new UserBuilder()
                .withName("John Doe")
                .withEmail("john@test.com")
                .build();

        assertEquals("John Doe", user.name);
    }
}
```

### **Positive und Negative Tests**

```java
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;

class BalancedTests {

    @Test
    void login_validCredentials_returnsTrue() {
        Auth auth = new Auth();
        assertTrue(auth.login("user", "correct-password"));
    }

    @Test
    void login_invalidPassword_returnsFalse() {
        Auth auth = new Auth();
        assertFalse(auth.login("user", "wrong-password"));
    }

    @Test
    void login_nonExistentUser_returnsFalse() {
        Auth auth = new Auth();
        assertFalse(auth.login("unknown", "password"));
    }

    @Test
    void login_emptyPassword_throwsException() {
        Auth auth = new Auth();
        assertThrows(IllegalArgumentException.class, () -> auth.login("user", ""));
    }

    static class Auth {
        boolean login(String user, String pass) {
            if (pass == null || pass.isEmpty()) throw new IllegalArgumentException("Password empty");
            if ("user".equals(user) && "correct-password".equals(pass)) return true;
            return false;
        }
    }
}
```

### **Test Organisation**

```java
import org.junit.jupiter.api.Test;

class UserServiceTestsOrganized {

    // Gruppiere Tests nach Feature/Methode

    // region Register Tests

    @Test
    void register_validUser_returnsTrue() {
        // ...
    }

    @Test
    void register_duplicateEmail_returnsFalse() {
        // ...
    }

    // endregion

    // region Login Tests

    @Test
    void login_validCredentials_returnsToken() {
        // ...
    }

    @Test
    void login_invalidCredentials_returnsNull() {
        // ...
    }

    // endregion
}
```

---

## **Schnellreferenz - Häufige Patterns**

### **Repository Test Pattern**

```java
import org.junit.jupiter.api.Test;

import static org.mockito.Mockito.*;

class RepositoryPatternTests {

    @Test
    void service_usesRepository_correctInteraction() {
        // Arrange
        UserRepository mockRepo = mock(UserRepository.class);
        User entity = new User();
        entity.id = 1;

        when(mockRepo.getById(1)).thenReturn(entity);

        Service service = new Service(mockRepo);

        // Act
        service.doSomething(1);

        // Assert
        verify(mockRepo, times(1)).getById(1);
        verify(mockRepo, times(1)).update(any(User.class));
    }

    static class Service {
        private final UserRepository repo;

        Service(UserRepository repo) {
            this.repo = repo;
        }

        void doSomething(int id) {
            User u = repo.getById(id);
            if (u != null) repo.update(u);
        }
    }
}
```

### **Email Service Test Pattern**

```java
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;

class EmailPatternTests {

    @Test
    void service_sendsEmail_withCorrectContent() {
        // Arrange
        FakeEmailService fakeEmail = new FakeEmailService();
        EmailOrderService service = new EmailOrderService(fakeEmail);

        // Act
        service.processOrder(123);

        // Assert
        assertEquals(1, fakeEmail.sentEmails.size());
        assertEquals("customer@test.com", fakeEmail.sentEmails.get(0).to());
        assertTrue(fakeEmail.sentEmails.get(0).subject().contains("Order"));
    }

    static class EmailOrderService {
        private final EmailService email;

        EmailOrderService(EmailService email) {
            this.email = email;
        }

        void processOrder(int orderId) {
            email.sendEmail("customer@test.com", "Order " + orderId, "Body");
        }
    }
}
```

### **State Validation Pattern**

```java
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;

class StateValidationPatternTests {

    @Test
    void service_changesState_correctly() {
        // Arrange
        Entity entity = new Entity();
        entity.state = EntityState.NEW;

        Service service = new Service();

        // Act
        service.process(entity);

        // Assert
        assertEquals(EntityState.PROCESSED, entity.state);
    }

    enum EntityState { NEW, PROCESSED }

    static class Entity {
        EntityState state;
    }

    static class Service {
        void process(Entity e) {
            e.state = EntityState.PROCESSED;
        }
    }
}
```

### **Collection Validation Pattern**

```java
import org.junit.jupiter.api.Test;

import java.util.List;

import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.*;

class CollectionValidationPatternTests {

    @Test
    void service_returnsFilteredList() {
        // Arrange
        UserRepository mockRepo = mock(UserRepository.class);
        when(mockRepo.getAll()).thenReturn(List.of(
                item(true, "A"),
                item(false, "B"),
                item(true, "C")
        ));

        ItemService service = new ItemService(mockRepo);

        // Act
        List<Item> result = service.getActiveItems();

        // Assert
        assertEquals(2, result.size());
        assertTrue(result.stream().allMatch(Item::isActive));
    }

    static Item item(boolean active, String name) {
        return new Item(active, name);
    }

    record Item(boolean isActive, String name) {}

    static class ItemService {
        private final UserRepository repo;

        ItemService(UserRepository repo) {
            this.repo = repo;
        }

        List<Item> getActiveItems() {
            return repo.getAll().stream()
                    .map(u -> new Item(u.isActive, u.name))
                    .filter(Item::isActive)
                    .toList();
        }
    }
}
```

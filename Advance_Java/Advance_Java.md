# Advanced Java Practical Exam Questions and Solutions

## Q1. Flight Ticket Booking using JDBC (Transaction Management)
**Problem Statement:** 
Write a Flight Ticket Booking program using JDBC for the `airlinedb` database. Use the `flights` and `bookings` tables to manage seat availability and perform bookings. Utilize JDBC transaction management (`setAutoCommit(false)`, `commit()`, and `rollback()`) to ensure data consistency.

**Solution / Implementation:**

```java
import java.sql.*;
import java.util.Scanner;

public class FlightBooking {
    // Database configuration
    static final String DB_URL = "jdbc:mysql://localhost:3306/airlinedb";
    static final String USER = "root";
    static final String PASS = "password";

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        
        System.out.print("Enter Flight ID: ");
        int flightId = scanner.nextInt();
        scanner.nextLine(); // Consume newline
        
        System.out.print("Enter Passenger Name: ");
        String passName = scanner.nextLine();
        
        System.out.print("Enter Seats Requested: ");
        int seatsReq = scanner.nextInt();

        Connection conn = null;

        try {
            conn = DriverManager.getConnection(DB_URL, USER, PASS);
            
            // 1. Disable AutoCommit to start Transaction
            conn.setAutoCommit(false);
            
            // 2. Check available seats and price
            String checkSql = "SELECT available_seats, price_per_seat FROM flights WHERE flight_id = ?";
            PreparedStatement checkStmt = conn.prepareStatement(checkSql);
            checkStmt.setInt(1, flightId);
            ResultSet rs = checkStmt.executeQuery();
            
            if (rs.next()) {
                int availSeats = rs.getInt("available_seats");
                double price = rs.getDouble("price_per_seat");
                
                // 3. Logic: If seats are available
                if (availSeats >= seatsReq) {
                    double totalAmount = seatsReq * price;
                    
                    // 4. Deduct seats from flights table
                    String updateSql = "UPDATE flights SET available_seats = available_seats - ? WHERE flight_id = ?";
                    PreparedStatement updateStmt = conn.prepareStatement(updateSql);
                    updateStmt.setInt(1, seatsReq);
                    updateStmt.setInt(2, flightId);
                    updateStmt.executeUpdate();
                    
                    // 5. Insert record into bookings table
                    String insertSql = "INSERT INTO bookings (passenger_name, flight_id, seats_booked, total_amount) VALUES (?, ?, ?, ?)";
                    PreparedStatement insertStmt = conn.prepareStatement(insertSql);
                    insertStmt.setString(1, passName);
                    insertStmt.setInt(2, flightId);
                    insertStmt.setInt(3, seatsReq);
                    insertStmt.setDouble(4, totalAmount);
                    insertStmt.executeUpdate();
                    
                    // 6. Commit the transaction
                    conn.commit();
                    System.out.println("Booking Successful! Total Amount: $" + totalAmount);
                } else {
                    // 7. Rollback if seats are insufficient
                    conn.rollback();
                    System.out.println("Booking Failed: Not enough seats available.");
                }
            } else {
                System.out.println("Booking Failed: Flight ID not found.");
            }
        } catch (SQLException e) {
            e.printStackTrace();
            try {
                if (conn != null) {
                    conn.rollback(); // Rollback on any SQL Exception
                    System.out.println("Transaction rolled back due to an error.");
                }
            } catch (SQLException ex) {
                ex.printStackTrace();
            }
        } finally {
            try {
                if (conn != null) conn.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
            scanner.close();
        }
    }
}
```

---

## Q2. Spring Boot @RestController Endpoint
**Problem Statement:** 
Create a Spring Boot project and write a `@RestController` with a single POST endpoint `/register` that accepts a name and email as `@RequestParam` and returns a formatted string.

**Solution / Implementation:**

```java
package com.example.demo.controller;

import org.springframework.web.bind.annotation.*;

@RestController
public class UserController {

    @PostMapping("/register")
    public String registerUser(
            @RequestParam("name") String name, 
            @RequestParam("email") String email) {
        
        return "User Registered: " + name + " | Email: " + email;
    }
}
```

**Testing the Endpoint (Using Postman or cURL):**
Because this is a `POST` request, you cannot test it directly by pasting a URL into a browser's address bar. You must use a tool like Postman or the following cURL command in your terminal:
```bash
curl -X POST "http://localhost:8080/register?name=JohnDoe&email=john@example.com"
```
**Expected Response:** `User Registered: JohnDoe | Email: john@example.com`

---

## Q3. Spring Boot JPA Entity & MySQL Configuration
**Problem Statement:** 
Create a Spring Boot project connected to MySQL (`studentdb`). Write the `Student` entity class with appropriate JPA annotations, constraints, and configure the `application.properties` to auto-generate the table.

**Solution / Implementation:**

**1. Entity Class (`Student.java`):**
```java
package com.example.demo.entity;

import jakarta.persistence.*;

@Entity
@Table(name = "students")
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false, unique = true)
    private String email;

    @Column(nullable = false)
    private String course;

    // Default configuration (nullable = true)
    private String phone;

    // Default constructor (required by Hibernate)
    public Student() {}

    // Getters and Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }

    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }

    public String getCourse() { return course; }
    public void setCourse(String course) { this.course = course; }

    public String getPhone() { return phone; }
    public void setPhone(String phone) { this.phone = phone; }
}
```

**2. Configuration File (`src/main/resources/application.properties`):**
```properties
# MySQL Database Connection configuration
spring.datasource.url=jdbc:mysql://localhost:3306/studentdb
spring.datasource.username=root
spring.datasource.password=your_password_here
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# Hibernate Configuration
# ddl-auto=update automatically creates/updates the 'students' table in 'studentdb'
spring.jpa.hibernate.ddl-auto=update

# Show SQL logs in the console to verify execution
spring.jpa.show-sql=true
```

**Verification:**
After running the Spring Boot Application, you can open your MySQL workbench or terminal and run:
```sql
USE studentdb;
DESC students;
```
You will see the table structure perfectly matching the Entity fields with constraints (e.g., auto-incrementing PK, not null columns, and unique constraints).

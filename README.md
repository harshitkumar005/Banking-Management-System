FinancialLandscapeBanking
FinancialLandscapeBanking is a simple Java console application simulating a basic banking system. It allows users to register, login, and perform essential banking operations such as checking balance, deposits, withdrawals, fund transfers, and viewing transaction history. The application uses MySQL as its backend database.

Features
User registration and login authentication

View current account balance

Deposit funds

Withdraw funds (with balance validation)

Transfer funds to other users (with transaction atomicity)

View detailed transaction history

Technologies Used
Java 8+

MySQL 5.7+

JDBC for database connectivity

Database Setup
Before running the application, set up the MySQL database and tables:

sql
Copy
CREATE DATABASE FinancialLandscape;

USE FinancialLandscape;

CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    password VARCHAR(50) NOT NULL,
    balance DOUBLE DEFAULT 0.0
);

CREATE TABLE transactions (
    id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT,
    type VARCHAR(50),
    amount DOUBLE,
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id)
);
Configuration
Update the DB_URL, USER, and PASS constants in the FinancialLandscapeBanking.java file with your MySQL connection details:

java
Copy
static final String DB_URL = "jdbc:mysql://localhost:3306/FinancialLandscape";
static final String USER = "root";
static final String PASS = "your_password";  // Replace with your actual password
How to Run
Compile the Java source file:

bash
Copy
javac FinancialLandscapeBanking.java
Run the compiled program:

bash
Copy
java FinancialLandscapeBanking
Follow the on-screen prompts to register, login, and use the banking system.

Important Notes
Passwords are stored in plain text for simplicity. For production systems, always use secure password hashing and salting.

The program assumes a local MySQL server with default port 3306.

Transactions in fund transfer are handled with JDBC transaction management (setAutoCommit(false) and commit()).

Input validation is minimal; improvements are recommended for production use.

License
This project is open source and free to use under the MIT License.

Author
Created by Harshit Kumar & Abhinay Pal

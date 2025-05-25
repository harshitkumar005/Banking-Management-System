import java.sql.*;
import java.util.Scanner;

public class FinancialLandscapeBanking {
    static final String DB_URL = "jdbc:mysql://localhost:3306/FinancialLandscape";
    static final String USER = "root";
    static final String PASS = "your_password"; // Replace with your actual MySQL password

    static Scanner scanner = new Scanner(System.in);
    static Connection conn;

    public static void main(String[] args) {
        try {
            conn = DriverManager.getConnection(DB_URL, USER, PASS);
            while (true) {
                System.out.println("\n=== Navigating the Financial Landscape ===");
                System.out.println("1. Register");
                System.out.println("2. Login");
                System.out.println("3. Exit");
                System.out.print("Select an option: ");
                int choice = scanner.nextInt();
                scanner.nextLine();

                switch (choice) {
                    case 1 -> register();
                    case 2 -> login();
                    case 3 -> {
                        System.out.println("Exiting system.");
                        return;
                    }
                    default -> System.out.println("Invalid option.");
                }
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    static void register() throws SQLException {
        System.out.print("Enter new username: ");
        String username = scanner.nextLine();
        System.out.print("Enter new password: ");
        String password = scanner.nextLine();

        String sql = "INSERT INTO users (username, password) VALUES (?, ?)";
        try (PreparedStatement stmt = conn.prepareStatement(sql)) {
            stmt.setString(1, username);
            stmt.setString(2, password);
            stmt.executeUpdate();
            System.out.println("Registration successful.");
        } catch (SQLIntegrityConstraintViolationException e) {
            System.out.println("Username already taken.");
        }
    }

    static void login() throws SQLException {
        System.out.print("Username: ");
        String username = scanner.nextLine();
        System.out.print("Password: ");
        String password = scanner.nextLine();

        String sql = "SELECT * FROM users WHERE username = ? AND password = ?";
        try (PreparedStatement stmt = conn.prepareStatement(sql)) {
            stmt.setString(1, username);
            stmt.setString(2, password);
            ResultSet rs = stmt.executeQuery();

            if (rs.next()) {
                int userId = rs.getInt("id");
                System.out.println("Login successful.");
                userMenu(userId);
            } else {
                System.out.println("Invalid credentials.");
            }
        }
    }

    static void userMenu(int userId) throws SQLException {
        while (true) {
            System.out.println("\n--- User Dashboard ---");
            System.out.println("1. View Balance");
            System.out.println("2. Deposit");
            System.out.println("3. Withdraw");
            System.out.println("4. Transfer Funds");
            System.out.println("5. View Transactions");
            System.out.println("6. Logout");
            System.out.print("Select an option: ");
            int choice = scanner.nextInt();

            switch (choice) {
                case 1 -> viewBalance(userId);
                case 2 -> deposit(userId);
                case 3 -> withdraw(userId);
                case 4 -> transfer(userId);
                case 5 -> viewTransactions(userId);
                case 6 -> {
                    System.out.println("Logged out.");
                    return;
                }
                default -> System.out.println("Invalid choice.");
            }
        }
    }

    static void viewBalance(int userId) throws SQLException {
        String sql = "SELECT balance FROM users WHERE id = ?";
        try (PreparedStatement stmt = conn.prepareStatement(sql)) {
            stmt.setInt(1, userId);
            ResultSet rs = stmt.executeQuery();
            if (rs.next()) {
                System.out.println("Current Balance: $" + rs.getDouble("balance"));
            }
        }
    }

    static void deposit(int userId) throws SQLException {
        System.out.print("Enter deposit amount: ");
        double amount = scanner.nextDouble();
        if (amount <= 0) {
            System.out.println("Invalid amount.");
            return;
        }

        String sql = "UPDATE users SET balance = balance + ? WHERE id = ?";
        try (PreparedStatement stmt = conn.prepareStatement(sql)) {
            stmt.setDouble(1, amount);
            stmt.setInt(2, userId);
            stmt.executeUpdate();
        }

        recordTransaction(userId, "Deposit", amount);
        System.out.println("Deposit successful.");
    }

    static void withdraw(int userId) throws SQLException {
        System.out.print("Enter withdrawal amount: ");
        double amount = scanner.nextDouble();

        String checkSql = "SELECT balance FROM users WHERE id = ?";
        try (PreparedStatement checkStmt = conn.prepareStatement(checkSql)) {
            checkStmt.setInt(1, userId);
            ResultSet rs = checkStmt.executeQuery();

            if (rs.next()) {
                double balance = rs.getDouble("balance");
                if (amount > 0 && amount <= balance) {
                    String sql = "UPDATE users SET balance = balance - ? WHERE id = ?";
                    try (PreparedStatement stmt = conn.prepareStatement(sql)) {
                        stmt.setDouble(1, amount);
                        stmt.setInt(2, userId);
                        stmt.executeUpdate();
                    }
                    recordTransaction(userId, "Withdraw", amount);
                    System.out.println("Withdrawal successful.");
                } else {
                    System.out.println("Insufficient balance.");
                }
            }
        }
    }

    static void transfer(int userId) throws SQLException {
        scanner.nextLine(); // consume newline
        System.out.print("Enter recipient username: ");
        String recipient = scanner.nextLine();
        System.out.print("Enter amount to transfer: ");
        double amount = scanner.nextDouble();

        if (amount <= 0) {
            System.out.println("Invalid amount.");
            return;
        }

        String getRecipient = "SELECT id FROM users WHERE username = ?";
        try (PreparedStatement stmt = conn.prepareStatement(getRecipient)) {
            stmt.setString(1, recipient);
            ResultSet rs = stmt.executeQuery();

            if (rs.next()) {
                int recipientId = rs.getInt("id");

                // Check balance
                String checkSql = "SELECT balance FROM users WHERE id = ?";
                try (PreparedStatement checkStmt = conn.prepareStatement(checkSql)) {
                    checkStmt.setInt(1, userId);
                    ResultSet checkRs = checkStmt.executeQuery();

                    if (checkRs.next()) {
                        double balance = checkRs.getDouble("balance");
                        if (balance >= amount) {
                            conn.setAutoCommit(false); // transaction start

                            // Withdraw from sender
                            String withdrawSql = "UPDATE users SET balance = balance - ? WHERE id = ?";
                            try (PreparedStatement withdrawStmt = conn.prepareStatement(withdrawSql)) {
                                withdrawStmt.setDouble(1, amount);
                                withdrawStmt.setInt(2, userId);
                                withdrawStmt.executeUpdate();
                            }

                            // Deposit to receiver
                            String depositSql = "UPDATE users SET balance = balance + ? WHERE id = ?";
                            try (PreparedStatement depositStmt = conn.prepareStatement(depositSql)) {
                                depositStmt.setDouble(1, amount);
                                depositStmt.setInt(2, recipientId);
                                depositStmt.executeUpdate();
                            }

                            // Record transactions
                            recordTransaction(userId, "Transfer Sent", amount);
                            recordTransaction(recipientId, "Transfer Received", amount);

                            conn.commit(); // transaction commit
                            conn.setAutoCommit(true);

                            System.out.println("Transfer successful.");
                        } else {
                            System.out.println("Insufficient balance.");
                        }
                    }
                } catch (SQLException e) {
                    conn.rollback();
                    System.out.println("Transfer failed. Rolled back.");
                    e.printStackTrace();
                }
            } else {
                System.out.println("Recipient not found.");
            }
        }
    }

    static void recordTransaction(int userId, String type, double amount) throws SQLException {
        String sql = "INSERT INTO transactions (user_id, type, amount) VALUES (?, ?, ?)";
        try (PreparedStatement stmt = conn.prepareStatement(sql)) {
            stmt.setInt(1, userId);
            stmt.setString(2, type);
            stmt.setDouble(3, amount);
            stmt.executeUpdate();
        }
    }

    static void viewTransactions(int userId) throws SQLException {
        String sql = "SELECT * FROM transactions WHERE user_id = ? ORDER BY timestamp DESC";
        try (PreparedStatement stmt = conn.prepareStatement(sql)) {
            stmt.setInt(1, userId);
            ResultSet rs = stmt.executeQuery();

            System.out.println("\n--- Transaction History ---");
            while (rs.next()) {
                System.out.printf("%s | %s | $%.2f%n",
                        rs.getTimestamp("timestamp"),
                        rs.getString("type"),
                        rs.getDouble

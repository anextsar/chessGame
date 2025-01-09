# chessGame

Creating a complete chess game in Java with user profiles, multiplayer, and a database is an ambitious project. I’ll guide you step by step through the process, from setting up your development environment to implementing all the required features. Here's a structured approach to how you can achieve this.

---

### **Step 1: Set Up Your Development Environment**

Before starting the coding process, make sure your development environment is ready.

#### 1.1 **Install Java Development Kit (JDK)**
You need to install the **Java Development Kit (JDK)**. Download the latest version from the [official Java website](https://www.oracle.com/java/technologies/javase-jdk11-downloads.html).

#### 1.2 **Install an Integrated Development Environment (IDE)**
An IDE helps in writing and managing your Java code efficiently. Some popular IDEs are:
- **Eclipse**
- **IntelliJ IDEA**
- **NetBeans**

#### 1.3 **Install MySQL Database**
You need a database to store user data, profiles, and game sessions. Download and install **MySQL** from the [official MySQL website](https://dev.mysql.com/downloads/installer/).

After installation, use MySQL Workbench or the MySQL command line to interact with the database.

---

### **Step 2: Set Up the Database**

You’ll need a MySQL database to store user data (like profiles and game information) and relationships (friends, games, etc.).

#### 2.1 **Create Database and Tables**
Open MySQL Workbench or use the MySQL command-line client, and run the following SQL commands to create the necessary tables:

```sql
CREATE DATABASE chessdb;

USE chessdb;

-- Users table to store user profiles
CREATE TABLE Users (
    user_id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(255) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    country VARCHAR(100),
    profile_picture BLOB,
    is_online BOOLEAN DEFAULT FALSE
);

-- Friends table to store friendships between users
CREATE TABLE Friends (
    user_id INT,
    friend_id INT,
    PRIMARY KEY (user_id, friend_id),
    FOREIGN KEY (user_id) REFERENCES Users(user_id),
    FOREIGN KEY (friend_id) REFERENCES Users(user_id)
);

-- GameSessions table to store ongoing or completed games
CREATE TABLE GameSessions (
    game_id INT PRIMARY KEY AUTO_INCREMENT,
    player1_id INT,
    player2_id INT,
    current_turn INT,
    game_status VARCHAR(50), -- 'ongoing', 'completed', etc.
    winner_id INT, -- NULL if ongoing
    FOREIGN KEY (player1_id) REFERENCES Users(user_id),
    FOREIGN KEY (player2_id) REFERENCES Users(user_id)
);
```

This will set up the tables for storing user profiles, friends, and game sessions.

#### 2.2 **Test Database Connectivity**
Test the connection to the database in your Java program. Use **JDBC** (Java Database Connectivity) to connect to MySQL. You can use the `mysql-connector-java` JDBC driver, which is available via Maven or can be manually added to your project.

```xml
<!-- If using Maven, add this dependency in your pom.xml -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.26</version>
</dependency>
```

### **Step 3: Implement User Registration and Login System**

You’ll need a **login** and **sign-up** system where users can register and log in with a username and password.

#### 3.1 **Create User Registration (Signup.java)**

Create a class to handle user registration and save the data into the database.

```java
import java.sql.*;
import java.util.Scanner;

public class Signup {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        
        // Collect user details
        System.out.println("Enter username:");
        String username = scanner.nextLine();
        System.out.println("Enter password:");
        String password = scanner.nextLine();
        System.out.println("Enter country:");
        String country = scanner.nextLine();
        
        try {
            // Connect to the database
            Connection connection = DriverManager.getConnection("jdbc:mysql://localhost/chessdb", "root", "password");
            
            // Insert user into the Users table
            String query = "INSERT INTO Users (username, password, country) VALUES (?, ?, ?)";
            PreparedStatement preparedStatement = connection.prepareStatement(query);
            preparedStatement.setString(1, username);
            preparedStatement.setString(2, password);
            preparedStatement.setString(3, country);
            preparedStatement.executeUpdate();
            
            System.out.println("User registered successfully!");
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

#### 3.2 **Create User Login (Login.java)**

Now, create a login system where the user can log in with their username and password.

```java
import java.sql.*;
import java.util.Scanner;

public class Login {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        
        // Collect login details
        System.out.println("Enter username:");
        String username = scanner.nextLine();
        System.out.println("Enter password:");
        String password = scanner.nextLine();
        
        try {
            // Connect to the database
            Connection connection = DriverManager.getConnection("jdbc:mysql://localhost/chessdb", "root", "password");
            
            // Validate user login credentials
            String query = "SELECT * FROM Users WHERE username = ? AND password = ?";
            PreparedStatement preparedStatement = connection.prepareStatement(query);
            preparedStatement.setString(1, username);
            preparedStatement.setString(2, password);
            ResultSet resultSet = preparedStatement.executeQuery();
            
            if (resultSet.next()) {
                System.out.println("Login successful!");
            } else {
                System.out.println("Invalid credentials.");
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

---

### **Step 4: Implement the Friendship System**

Users should be able to send and accept friend requests.

#### 4.1 **Create FriendRequest.java**

This class handles sending friend requests.

```java
import java.sql.*;
import java.util.Scanner;

public class FriendRequest {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        
        // Collect user IDs for sending the friend request
        System.out.println("Enter your user ID:");
        int userId = scanner.nextInt();
        System.out.println("Enter friend ID:");
        int friendId = scanner.nextInt();
        
        try {
            // Connect to the database
            Connection connection = DriverManager.getConnection("jdbc:mysql://localhost/chessdb", "root", "password");
            
            // Insert the friendship into the Friends table
            String query = "INSERT INTO Friends (user_id, friend_id) VALUES (?, ?)";
            PreparedStatement preparedStatement = connection.prepareStatement(query);
            preparedStatement.setInt(1, userId);
            preparedStatement.setInt(2, friendId);
            preparedStatement.executeUpdate();
            
            System.out.println("Friend request sent!");
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

#### 4.2 **Handling Friend Requests**

You can add more logic to manage friend request statuses, accepting, or rejecting friend requests.

---

### **Step 5: Implement the Chess Game Logic**

You’ll need to create classes for the **ChessBoard** and **ChessGame** to handle the logic of chess moves and validating the state of the game (like check, checkmate, castling, etc.).

```java
// Chessboard setup with basic chess rules logic
public class ChessGame {
    private String[][] board = new String[8][8];

    public ChessGame() {
        // Initialize the board with pieces
        for (int i = 0; i < 8; i++) {
            for (int j = 0; j < 8; j++) {
                board[i][j] = " "; // Empty square initially
            }
        }
    }

    public boolean movePiece(String from, String to) {
        // Basic validation for move (you can expand this)
        System.out.println("Moving piece from " + from + " to " + to);
        return true;
    }
}
```

### **Step 6: Implement Multiplayer (Using Java Sockets)**

For the multiplayer aspect, you will need a **GameServer** to handle connections between players.

#### 6.1 **Game Server**

```java
import java.io.*;
import java.net.*;
import java.util.*;

public class GameServer {
    private static final int PORT = 12345;
    private static Map<Integer, Socket> userSockets = new HashMap<>();

    public static void main(String[] args) {
        try (ServerSocket serverSocket = new ServerSocket(PORT)) {
            while (true) {
                Socket socket = serverSocket.accept();
                // Handle the connection with the client
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

You will need to send messages to users for actions like moving pieces, ending a game, and chat.

---

### **Step 7: Implement the GUI (JavaFX or Swing)**

The game interface can be built using **JavaFX** or **Swing**. JavaFX is recommended for modern UI elements. Here's a very basic starting point for your main game window using JavaFX:

```java
import javafx.application.Application;
import javafx.scene.Scene;
import javafx.scene.layout.StackPane;
import javafx.scene.text.Text;
import javafx.stage.Stage;

public class MainApp extends Application {
    @Override
    public void start(Stage primaryStage) {
        Text helloText = new Text("Welcome to Chess!");
        StackPane root = new StackPane();
        root.getChildren().add(helloText);
        
        Scene scene = new Scene(root, 300, 250);
        primaryStage.setTitle("Chess Game");
        primaryStage.setScene(scene);
        primaryStage.show();
    }
    
    public static void main(String[] args) {
        launch(args);
    }
}
```

---

### **Step 8: Combine All Parts**

Finally, combine the registration system, the friendship system, the game logic, and the multiplayer communication into a single working system.

1. **Handle database connections** across all components.
2. **Make sure that users can play against their friends** or random players.
3. **Update the user interface** to display chessboard and game statuses.
4. **Test the system** thoroughly to handle edge cases and ensure smooth multiplayer interactions.

---

### **Conclusion**

This guide provides you with the basic steps for creating a multi-functional chess game with user profiles, a friend system, and multiplayer support in Java. There’s a lot of work to do, but following these steps incrementally will help you break the project down into manageable parts. You'll also need to further improve and expand the chess logic, network communication, and UI for a fully functional game. Good luck with your project!

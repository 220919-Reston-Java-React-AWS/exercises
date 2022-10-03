# Implementing Registration Functionality
This is intended to be a guide to help you focus on how you should be building up your project. A project is composed of many features, layers, and parts that work together. However, it is important as a developer to break a large project down into smaller chunks. It is never possible to write an entire application and hope everything works. We must always ensure that each small piece we build is working before moving onto the next

## Requirements
First, think about the requirements of P1. It is an reimbursement ticketing application with two roles: employee and manager. Users must therefore, at a minimum, log in with a username and password and have a role associated with them

The register feature as detailed on the TRELLO board indicates that
1. We must ensure the username is not already registered
2. The employee role should be the role registered by default
3. We should register with a username and password at a MINIMUM

With this information, we can begin to design a database schema to implement this feature

## Design

### Database
The database for supporting registration must have at least 1 table
- users

At a minimum, this table must have username, password, role, and of course an id PK to uniquely identify each user. Therefore we could potentially structure a table as follows:
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(200) NOT NULL UNIQUE,
    password VARCHAR(200) NOT NULL,
    role VARCHAR(20) NOT NULL
);
```

Notice that username is `UNIQUE` and `NOT NULL`

### Backend
The backend itself will be consisting of 3 layers
- Controller
- Service
- Repository

Each of these layers serves a specific purpose:
- The controller layer is meant to handle the logic specific to extracting data from an HTTP request and ensuring authorized access to that endpoint, and invokes a method in the service layer
- The service layer is meant to receive arguments from the controller layer, perform business logic such as input validation and other checks, and then invoke a method in the repository layer
- The repository layer is meant to receive arguments from the service layer, and perform queries on data in the database and return it back to the service layer method

We will have AuthenticationController, AuthenticationService, and UserRepository as our 3 classes

## Implementation
We will work backwards
- Start at the repository layer
    - Write code
    - Make sure it works
- Move to the service layer
    - Write code
    - Make sure it works
- Move to the controller layer
    - Write code
    - Make sure it works

## User Model
```java
public class User {
    private int id;
    private String username;
    private String password;
    private String role;

    // no-args constructor

    // parameterized constructor

    // getters/setters

    // toString()

    // equals()

    // hashCode()
}
```

## Repository Layer
Since the repository layer is in charge of modifying or retrieving data in the database, we first need to think about what SQL statements we need to write.

When registering, we are essentially adding a new user to the users table.

Hence,

```sql
INSERT INTO users (username, password, role)
VALUES (?, ?, ?)
```

Where the ? marks represent placeholders for information we would like to insert

We then create a class called UserRepository

```java
public class UserRepository {}
```

Define a method in the class called `addUser(User user)`, which will return a user object containing an automatically generated ID (possible because of SERIAL on the id primary key). We will also create another method called `getUserByUsername(String user)`.

NOTE: You must create a ConnectionFactory class that contains the `createConnection()` method. Please refer to the [example](https://github.com/220919-Reston-Java-React-AWS/demos/blob/main/week2/assignment-management-demo/assignment-management-app/src/main/java/com/revature/repository/ConnectionFactory.java) in week-2 for reference.

```java
public class UserRepository {

    public User addUser(User user) {
        try (Connection con = ConnectionFactory.createConnection()) {
            String sql = "INSERT INTO users (username, password, role) VALUES (?, ?, ?)";
            PreparedStatement pstmt = con.prepareStatement(sql);

            PreparedStatement pstmt = connectionObject.prepareStatement(sql, Statement.RETURN_GENERATED_KEYS);

            pstmt.setString(1, user.getUsername());
            pstmt.setString(2, user.getPassword());
            pstmt.setString(3, "employee");

            // Execute the INSERT statement
            int numberOfRecordsAdded = pstmt.executeUpdate();

            // Retrieve the id that got automatically generated
            ResultSet rs = pstmt.getGeneratedKeys();
            rs.next();
            int id = rs.getInt(1);

            return new User(id, user.getUsername(), user.getPassword(), user.getRole());
        }
    }

    public User getUserByUsername(String username) {
        try (Connection connectionObj = ConnectionFactory.createConnection()) {
            String sql = "SELECT * FROM users as u WHERE u.username = ?";
            PreparedStatement pstmt = connectionObj.prepareStatement(sql);

            pstmt.setString(1, username);

            ResultSet rs = pstmt.executeQuery(); // ResultSet represents a temporary table that contains all data that we have
            // queried for

            if (rs.next()) { // returns a boolean indicating whether there is a record or not for the "next" row AND iterates to the next row
                int id = rs.getInt("id");
                String un = rs.getString("username");
                String pw = rs.getString("password");
                String role = rs.getInt("role");

                return new User(id, un, pw, role);
            } else {
                return null;
            }

        }
    }

}
```

TESTING IS ALWAYS IMPORTANT. The easiest way to make sure it is working is to simply create a main that that will invoke the method you just made. Always, always write code, make sure it works before moving onto the next thing

```java
public class UserRepository {

    // ALL OTHER CODE ....
    // ....
    // ....

    // Delete this method after making sure addUser works!
    public static void main(String[] args) throws SQLException {
        User user = new User(0, "user123", "password12345", null);

        UserRepository ur = new UserRepository();
        User addedUser = ur.addUser(user); // quick informal test
        System.out.println(addedUser);

        User user123 = ur.getUserByUsername("user123"); // quick informal test
        System.out.println(user123);
    }

}
```

Make sure to the delete the main method once you're sure both methods work

### Service layer
In the AuthenticationService class, we will have a single method called `register(User user)`. We want for the service layer to contain input validation and other business logic

Register, for example, requires that a user does not already exist with a particular username. So, we need to make sure there isn't already a user with a particular username, and throw an exception if they do. Make sure to create a custom exception called `UsernameAlreadyExistsException`.

```java
public class AuthenticationService {

    private UserRepository ur = new UserRepository();

    public User register(User user) throws UsernameAlreadyExistsException {
        if (ur.getUserByUsername(user.getUsername()) != null) {
            throw new UsernameAlreadyExistsException("User with username " + user.getUsername() + " already exists!");
        }

        User addedUser = ur.addUser(user);

        return addedUser;
    }

}
```

TESTING IS ALWAYS IMPORTANT. The easiest way to make sure it is working is to simply create a main that that will invoke the method you just made. Always, always write code, make sure it works before moving onto the next thing.

```java
public class AuthenticationService {
    // ALL OTHER CODE ....
    // ....
    // ....

    // DELETE ONCE YOU KNOW IT'S WORKING
    public static void main(String[] args) {
        AuthenticationService as = new AuthenticationService();

        User user = new User(0, "testing123", "test", null);

        User addedUser = as.register(user);

        System.out.println(addedUser);
    }
}
```
NOTE: Try running the code above twice since we want to make sure register works, as well as whether it throws UsernameAlreadyExistsException twice for the second run (that's exactly what we want to happen).

### Login Endpoint (Controller)
In the controller layer, we will map out an endpoint for supporting registration

Let's revisit the idea of HTTP methods. Each method has a certain convention associated with it. You can of course break conventions, although it's of course not recommended by convention.
- POST: add a resource to the system
- GET: retrieve a resource from the system
- PUT: fully replace a resource
- PATCH: partially replace a resource
- DELETE: delete a resource

If we want to register, then we are essentially adding a user resource to the system. And hence, 
- `POST /users` is an appropriate endpoint to be mapping

In your controller class, the following code serves to map the POST /users endpoint to the webserver
```java
public class AuthenticationController {

    private AuthenticationService authService = new AuthenticationService();

    public void mapEndpoints(Javalin app) {
        app.post("/users", (ctx) -> {
            User userToAdd = ctx.bodyAsClass(User.class);

            try {
                User addedUser = authService.register(userToAdd);

                ctx.json(addedUser);
                ctx.status(201);
            } catch (UsernameAlreadyExistsException e) {
                ctx.result(e.getMessage());
                ctx.status(400);
            }
        });
    }
}
```

TESTING IS ALWAYS IMPORTANT. Open postman and then send a POST request for /users, filling in the JSON data for registering

JSON:
```json
{
    "username": "john_doe",
    "password": "12345"
}
```

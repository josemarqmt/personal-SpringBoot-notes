

# Spring Boot Documentation

## Index
1. [Overview of Spring and Spring Boot](#id1)
2. [Understanding JDK and JVM in Context](#id2)
3. [Key Concepts in Spring and Spring Boot](#id3)
4. [Project Structure and Annotations](#id4)
5. [Testing and Mocking in Spring Boot](#id5)

<a name="id1"> </a>
### 1. Overview of Spring and Spring Boot
Spring is a powerful framework that simplifies Java development by addressing the complexities of building enterprise applications, particularly those based on J2EE (Java 2 Platform, Enterprise Edition). J2EE applications can be complex and heavyweight, requiring extensive configuration.

Spring Boot, as an extension of Spring, was created to simplify the development of standalone applications. It allows developers to quickly bootstrap projects with minimal and default configurations, reducing the overhead associated with traditional J2EE setups. This makes Spring Boot particularly appealing for microservices architectures.

<a name="id2"> </a>
### 2. Understanding JDK and JVM in Context
- **Java Development Kit (JDK)**: The JDK is essential for developing Java applications, including those built with Spring Boot. It provides the necessary tools, such as the Java compiler, which compiles your Java code into bytecode that can be executed on the JVM.
  
  Example commands:
  ```bash
  # Compile Java code
  javac MyApplication.java

  # Run the application
  java MyApplication
  ```

- **Java Virtual Machine (JVM)**: The JVM is the runtime environment that executes Java bytecode, making Java applications platform-independent. When you run a Spring Boot application, the JVM interprets the compiled code, allowing it to run seamlessly across different systems.

<a name="id3"> </a>
### 3. Key Concepts in Spring and Spring Boot
- **Aspect-Oriented Programming (AOP)**: AOP in Spring allows developers to separate cross-cutting concerns (such as logging and security) from business logic. This leads to cleaner and more manageable code, especially beneficial in large applications.

- **Inversion of Control (IoC)** and **Dependency Injection (DI)**: 
  - **Inversion of Control (IoC)**: This is a design principle where the creation and management of objects is delegated to a container or framework. Instead of the application code controlling how dependencies are created and managed, the framework handles it, promoting a weaker coupling between classes.
  
  - **Dependency Injection (DI)**: This is a specific way of implementing IoC. In DI, a class's dependencies are provided externally (usually by an IoC container), rather than being created within the class itself. This allows classes to be easier to test and maintain since their dependencies can be mocked or replaced.

### Explanation of `@Autowired`
The `@Autowired` annotation is used for dependency injection in Spring. It allows the Spring container to automatically inject the appropriate bean into a property, constructor, or method marked with this annotation. This eliminates the need for manually creating instances of dependencies, promoting cleaner and more maintainable code.

**Example of using `@Autowired`**:
```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class UserService {
    private final UserRepository userRepository;

    // Constructor injection
    @Autowired
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public User findUserById(Long id) {
        return userRepository.findById(id).orElse(null);
    }
}
```

In this example:
- The `UserService` class has a dependency on `UserRepository`.
- By using `@Autowired` on the constructor, Spring automatically injects an instance of `UserRepository` when it creates a `UserService` bean.

**Setter Injection**:
You can also use `@Autowired` on setter methods if you prefer that form of injection.

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class UserService {
    private UserRepository userRepository;

    // Setter injection
    @Autowired
    public void setUserRepository(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public User findUserById(Long id) {
        return userRepository.findById(id).orElse(null);
    }
}
```

### Explanation of `@Bean`
The `@Bean` annotation is used to declare a bean in the Spring context. A **bean** in Java is an object that is created, managed, and configured by the Spring container. You can think of a bean as any object that you need to be managed by Spring.

**Example of using `@Bean`**:
```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AppConfig {
    
    @Bean
    public UserService userService(UserRepository userRepository) {
        return new UserService(userRepository);
    }
}
```

In this example:
- The `AppConfig` class is marked as `@Configuration`, indicating that it contains bean definitions.
- The `userService` method is marked with `@Bean`, indicating that it should be managed as a bean. When Spring creates this bean, it injects an instance of `UserRepository` as a constructor argument.

<a name="id4"> </a>
### 4. Project Structure and Annotations

#### Class Interaction Diagram

Below is a simple diagram representing how different classes interact in the user management application workflow:

```
        +------------------+
        |  UserController  |
        +--------+---------+
                 |
                 | calls
                 |
        +--------v---------+
        |    UserService   |
        +--------+---------+
                 |
                 | calls
                 |
        +--------v---------+
        |   UserRepository  |
        +--------+---------+
                 |
                 | returns
                 |
        +--------v---------+
        |       User       |
        +--------+---------+
                 |
                 | converts to
                 |
        +--------v---------+
        |     UserDTO      |
        +------------------+
```

#### Example Workflow

1. **Controller (UserController)**: This is the entry point for HTTP requests. It handles client requests and calls the corresponding service.

   Example class:
   ```java
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.http.ResponseEntity;
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.PathVariable;
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.RestController;

   @RestController
   @RequestMapping("/api/users")
   public class UserController {
       @Autowired
       private UserService userService;

       @GetMapping("/{id}")
       public ResponseEntity<UserDTO> getUserById(@PathVariable Long id) {
           UserDTO userDTO = userService.findById(id);
           return ResponseEntity.ok(userDTO);
       }
   }
   ```

   **Common Annotations**: 
   - `@RestController`: Combines `@Controller` and `@ResponseBody`, making it easier to create RESTful controllers.
   - `@RequestMapping`: Defines the base path for requests.

2. **Service (UserService)**: Contains the business logic for managing users. The controller calls this service to perform operations.

   Example class:
   ```java
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.stereotype.Service;

   @Service
   public class UserService {
       @Autowired
       private UserRepository userRepository;

       public UserDTO findById(Long id) {
           User user = userRepository.findById(id)
               .orElseThrow(() -> new UserNotFoundException(id));
           return convertToDTO(user);
       }

       private UserDTO convertToDTO(User user) {
           UserDTO dto = new UserDTO();
           dto.setName(user.getName());
           dto.setEmail(user.getEmail());
           return dto;
       }
   }
   ```

   **Common Annotations**:
   - `@Service`: Indicates that the class contains business logic.

3. **Repository (UserRepository)**: Interacts with the database to perform data access operations. It is responsible for retrieving user information.

   Example interface:
   ```java
   import org.springframework.data.jpa.repository.JpaRepository;
   import org.springframework.stereotype.Repository;

   @Repository
   public interface UserRepository extends JpaRepository<User, Long> {
       // Additional query methods can be defined here
   }
   ```

   **Common Annotations**:
   - `@Repository`: Indicates that the class is a data access component.

4. **Entity (User)**: Represents a data model that maps to a table in the database.

   Example class:
   ```java
   import javax.persistence.Entity;
   import javax.persistence.GeneratedValue;
   import javax.persistence.GenerationType;
   import javax.persistence.Id;

   @Entity
   public class User {
       @Id
       @GeneratedValue(strategy = GenerationType.IDENTITY)
       private Long id;
       private String name;
       private String email;

       // Getters and setters
   }
   ```

   **Common Annotations**:
   - `@Entity`: Marks the class as an entity that maps to a database table.
   - `@Id`: Indicates the field that is the primary key.
   - `@GeneratedValue`: Specifies that the primary key value will be generated automatically.

5. **DTO (UserDTO)**: Used to transfer data between the controller and the service.

   Example class:
   ```java
   public class UserDTO {
       private String name;
       private String email;

       // Getters and setters
   }
   ```

   **Common Annotations**:
   - Typically does not require specific annotations but can be included in the conversion between entities and DTOs.

6. **Exception (UserNotFoundException)**: Handles custom

 exceptions in case a user is not found.

   Example class:
   ```java
   public class UserNotFoundException extends RuntimeException {
       public UserNotFoundException(Long id) {
           super("User not found: " + id);
       }
   }
   ```

<a name="id5"> </a>
### 5. Testing and Mocking in Spring Boot

Testing is crucial for maintaining application quality. In Spring Boot, there are two primary types of tests: unit tests and integration tests.

#### Unit Testing
Unit tests are used to test individual components or classes in isolation.

**Example of a Unit Test for UserService**:
```java
import static org.mockito.Mockito.*;
import static org.junit.jupiter.api.Assertions.*;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;

public class UserServiceTest {
    @InjectMocks
    private UserService userService;

    @Mock
    private UserRepository userRepository;

    @BeforeEach
    void setUp() {
        MockitoAnnotations.openMocks(this);
    }

    @Test
    public void testFindUserById_Success() {
        User user = new User();
        user.setId(1L);
        user.setName("John Doe");
        user.setEmail("john.doe@example.com");

        when(userRepository.findById(1L)).thenReturn(Optional.of(user));

        UserDTO userDTO = userService.findById(1L);

        assertNotNull(userDTO);
        assertEquals("John Doe", userDTO.getName());
        assertEquals("john.doe@example.com", userDTO.getEmail());
    }

    @Test
    public void testFindUserById_NotFound() {
        when(userRepository.findById(1L)).thenReturn(Optional.empty());

        assertThrows(UserNotFoundException.class, () -> userService.findById(1L));
    }
}
```

In this unit test:
- We use Mockito to create a mock for the `UserRepository`.
- The `@InjectMocks` annotation creates an instance of `UserService` and injects the mocked `UserRepository`.
- We test the success case and the not found case for the `findById` method.

#### Integration Testing
Integration tests are used to test how various components of the application work together.

**Example of an Integration Test**:
```java
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;

@SpringBootTest
@AutoConfigureMockMvc
public class UserControllerIntegrationTest {
    @Autowired
    private MockMvc mockMvc;

    @Test
    public void testGetUserById() throws Exception {
        mockMvc.perform(get("/api/users/1")
                .accept(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.name").value("John Doe"))
                .andExpect(jsonPath("$.email").value("john.doe@example.com"));
    }

    @Test
    public void testGetUserById_NotFound() throws Exception {
        mockMvc.perform(get("/api/users/999")
                .accept(MediaType.APPLICATION_JSON))
                .andExpect(status().isNotFound());
    }
}
```

In this integration test:
- We use Spring's `MockMvc` to perform requests against the controller.
- The first test checks that the endpoint returns a user with the expected details.
- The second test checks the behavior when a user is not found.

### Conclusion
Spring Boot streamlines Java development, offering powerful features like dependency injection and easy setup for testing. Understanding its core concepts, such as IoC and DI, is essential for building robust applications. By leveraging annotations like `@Autowired` and `@Bean`, developers can create clean, maintainable code. Unit and integration tests further ensure the quality and reliability of applications.


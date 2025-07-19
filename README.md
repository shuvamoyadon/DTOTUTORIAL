# Spring Boot E-commerce Project

This is a Spring Boot-based E-commerce application with a clear separation of concerns and a well-structured architecture. The project demonstrates best practices in Spring Boot development, including RESTful APIs, DTOs, exception handling, and more.

## Table of Contents
- [Project Structure](#project-structure)
- [Main Application Class](#main-application-class)
- [Detailed Package Explanation](#detailed-package-explanation-with-code-examples)
- [Interview Questions & Answers](#interview-questions--answers)

## Project Structure

```
src/main/java/com/ecommerce/project/
├── config/         # Configuration classes
├── controller/     # REST API endpoints
├── exceptions/     # Custom exception handling
├── model/          # Entity classes
├── payload/        # DTOs (Data Transfer Objects)
├── repositories/   # Data access layer
└── service/        # Business logic layer
```

## Main Application Class

```java
package com.ecommerce.project;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class main {
    public static void main(String[] args) {
        SpringApplication.run(main.class, args);
    }
}
```
- `@SpringBootApplication`: Enables auto-configuration, component scanning, and property support
- Bootstraps the Spring Boot application
- Starts the embedded Tomcat server on port 8080

## Interview Questions & Answers

### 1. What is the purpose of using DTOs in this application?

**Answer:**
DTOs (Data Transfer Objects) are used to transfer data between different layers of the application. In this project, `CategoryDTO` is used to:
- Decouple the database entities from the API layer
- Control what data is exposed through the API
- Prevent over-fetching of data
- Add validation annotations without polluting the domain model
- Handle data transformation and validation before it reaches the service layer

### 2. Explain the difference between `@Controller` and `@RestController`

**Answer:**
- `@Controller` is a Spring stereotype that marks a class as a web controller, capable of handling HTTP requests. It's typically used with view technologies.
- `@RestController` is a specialized version of `@Controller` that includes `@ResponseBody` by default, meaning it's designed for RESTful web services and returns data in JSON/XML format.

In this project, `@RestController` is used in `CategoryController` to handle REST API endpoints that return JSON responses.

### 3. How does exception handling work in this application?

**Answer:**
The application uses a global exception handling mechanism with `@ControllerAdvice`:
1. Custom exceptions like `ResourceNotFoundException` and `APIException` are defined
2. `MyGlobalExceptionHandler` class handles these exceptions using `@ExceptionHandler`
3. When an exception is thrown, it's caught by the appropriate handler method
4. The handler creates a consistent error response format

For example, when a category is not found, `ResourceNotFoundException` is thrown and converted to a proper HTTP 404 response.

### 4. What is the purpose of ModelMapper in this project?

**Answer:**
ModelMapper is used for object-to-object mapping between DTOs and entities. In this project, it's configured in `AppConfig` and used to:
- Convert `Category` entities to `CategoryDTO` objects
- Convert `CategoryDTO` objects back to `Category` entities
- Reduce boilerplate code for manual mapping between objects
- Handle complex object transformations consistently

### 5. Explain the difference between `@Service` and `@Repository` annotations

**Answer:**
- `@Service` is used to mark classes at the service layer which hold business logic. In this project, `CategoryServiceImpl` is annotated with `@Service`.
- `@Repository` is used for classes that directly interact with the database. It's a specialization of `@Component` and includes automatic exception translation from database exceptions to Spring's `DataAccessException`.

### 6. How would you add pagination to the `getAllCategories` endpoint?

**Answer:**
To add pagination, you would:
1. Update the repository method to accept `Pageable`
2. Modify the service layer to handle pagination
3. Update the DTO to include pagination metadata

Example implementation:
```java
// In CategoryRepository
Page<Category> findAll(Pageable pageable);

// In CategoryService
public CategoryResponse getAllCategories(int pageNo, int pageSize) {
    Pageable pageable = PageRequest.of(pageNo, pageSize);
    Page<Category> categories = categoryRepository.findAll(pageable);
    
    List<CategoryDTO> content = categories.getContent().stream()
        .map(category -> modelMapper.map(category, CategoryDTO.class))
        .collect(Collectors.toList());
    
    CategoryResponse categoryResponse = new CategoryResponse();
    categoryResponse.setContent(content);
    categoryResponse.setPageNo(categories.getNumber());
    categoryResponse.setPageSize(categories.getSize());
    categoryResponse.setTotalElements(categories.getTotalElements());
    categoryResponse.setTotalPages(categories.getTotalPages());
    categoryResponse.setLast(categories.isLast());
    
    return categoryResponse;
}
```

### 7. What is the purpose of `@Valid` annotation in the controller methods?

**Answer:**
The `@Valid` annotation triggers the validation of the request body against the validation constraints defined in the DTO. For example, if `CategoryDTO` has `@NotBlank` on a field, the validation will ensure the field is not null and not empty before the controller method is executed.

### 8. How would you add caching to improve performance?

**Answer:**
To add caching:
1. Add `@EnableCaching` to the main application class
2. Add cache dependencies (e.g., Caffeine or Ehcache)
3. Annotate service methods with `@Cacheable`, `@CachePut`, or `@CacheEvict`

Example:
```java
@Cacheable(value = "categories", key = "#categoryId")
public CategoryDTO getCategoryById(Long categoryId) {
    // implementation
}

@CacheEvict(value = "categories", key = "#categoryId")
public void deleteCategory(Long categoryId) {
    // implementation
}
```

### 9. What is the purpose of the `@Transactional` annotation?

**Answer:**
`@Transactional` ensures that a method executes within a database transaction. It provides:
- Atomicity: All operations complete successfully or none do
- Consistency: The database remains in a consistent state
- Isolation: Concurrent transactions don't interfere with each other
- Durability: Changes persist after transaction completion

In this project, it should be added to service methods that modify data to ensure data integrity.

### 10. How would you secure the admin endpoints?

**Answer:**
To secure admin endpoints:
1. Add Spring Security dependency
2. Configure security rules in a `SecurityConfig` class
3. Use method-level security with `@PreAuthorize`

Example:
```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfig {
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .anyRequest().permitAll()
            )
            .httpBasic();
        return http.build();
    }
}
```

Then secure the admin endpoint:
```java
@DeleteMapping("/admin/categories/{categoryId}")
@PreAuthorize("hasRole('ADMIN')")
public ResponseEntity<String> deleteCategory(@PathVariable Long categoryId) {
    // implementation
}
```

## Project Structure

This is a Spring Boot-based E-commerce application with a clear separation of concerns and a well-structured architecture. Below is a detailed explanation of the project structure with code examples for each component.

### 1. `config/`
Contains Spring configuration classes that define the application's behavior and setup.

**Example: AppConfig.java**
```java
package com.ecommerce.project.config;

import org.modelmapper.ModelMapper;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AppConfig {
    @Bean
    public ModelMapper modelMapper() {
        return new ModelMapper();
    }
}
```
- `@Configuration`: Marks this class as a source of bean definitions
- `@Bean`: Creates a ModelMapper bean for object mapping between DTOs and entities

### 2. `exceptions/`
Handles custom exceptions and global exception handling.

**Example: ResourceNotFoundException.java**
```java
package com.ecommerce.project.exceptions;

import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.ResponseStatus;

@ResponseStatus(value = HttpStatus.NOT_FOUND)
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }
}
```

**Example: MyGlobalExceptionHandler.java**
```java
package com.ecommerce.project.exceptions;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.context.request.WebRequest;

@ControllerAdvice
public class MyGlobalExceptionHandler {
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorDetails> handleResourceNotFoundException(
            ResourceNotFoundException exception, WebRequest webRequest) {
        ErrorDetails errorDetails = new ErrorDetails(
            new Date(),
            exception.getMessage(),
            webRequest.getDescription(false)
        );
        return new ResponseEntity<>(errorDetails, HttpStatus.NOT_FOUND);
    }
}
```

### 3. `service/`
Contains business logic implementations.

**Example: CategoryService.java (Interface)**
```java
package com.ecommerce.project.service;

import com.ecommerce.project.payload.CategoryDTO;
import com.ecommerce.project.payload.CategoryResponse;

public interface CategoryService {
    CategoryDTO createCategory(CategoryDTO categoryDTO);
    CategoryResponse getCategoryById(Long id);
}
```

**Example: CategoryServiceImpl.java**
```java
package com.ecommerce.project.service;

import com.ecommerce.project.exceptions.ResourceNotFoundException;
import com.ecommerce.project.model.Category;
import com.ecommerce.project.payload.CategoryDTO;
import com.ecommerce.project.payload.CategoryResponse;
import com.ecommerce.project.repositories.CategoryRepository;
import org.modelmapper.ModelMapper;
import org.springframework.stereotype.Service;

@Service
public class CategoryServiceImpl implements CategoryService {
    private final CategoryRepository categoryRepository;
    private final ModelMapper modelMapper;

    public CategoryServiceImpl(CategoryRepository categoryRepository, ModelMapper modelMapper) {
        this.categoryRepository = categoryRepository;
        this.modelMapper = modelMapper;
    }

    @Override
    public CategoryDTO createCategory(CategoryDTO categoryDTO) {
        if (categoryRepository.existsByName(categoryDTO.getName())) {
            throw new ResourceNotFoundException("Category already exists");
        }
        Category category = modelMapper.map(categoryDTO, Category.class);
        Category savedCategory = categoryRepository.save(category);
        return modelMapper.map(savedCategory, CategoryDTO.class);
    }

    @Override
    public CategoryResponse getCategoryById(Long id) {
        Category category = categoryRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("Category not found with id: " + id));
        return modelMapper.map(category, CategoryResponse.class);
    }
}
```

### 4. `payload/`
Contains Data Transfer Objects (DTOs) for API requests and responses.

**Example: CategoryDTO.java (Request DTO)**
```java
package com.ecommerce.project.payload;

import lombok.Data;

@Data
public class CategoryDTO {
    private String name;
    private String description;
}
```

**Example: CategoryResponse.java (Response DTO)**
```java
package com.ecommerce.project.payload;

import lombok.Data;

@Data
public class CategoryResponse {
    private Long id;
    private String name;
    private String description;
}
```

## Application Flow with Code Examples

### 1. Request Flow

**1.1 Client Request**
```http
POST /api/categories
Content-Type: application/json

{
    "name": "Electronics",
    "description": "Electronic devices and accessories"
}
```

**1.2 Controller**
```java
@RestController
@RequestMapping("/api/categories")
public class CategoryController {
    private final CategoryService categoryService;

    @PostMapping
    public ResponseEntity<CategoryDTO> createCategory(
            @Valid @RequestBody CategoryDTO categoryDTO) {
        CategoryDTO savedCategory = categoryService.createCategory(categoryDTO);
        return new ResponseEntity<>(savedCategory, HttpStatus.CREATED);
    }
}
```

**1.3 Service**
```java
public CategoryDTO createCategory(CategoryDTO categoryDTO) {
    // Business logic and validation
    Category category = modelMapper.map(categoryDTO, Category.class);
    Category savedCategory = categoryRepository.save(category);
    return modelMapper.map(savedCategory, CategoryDTO.class);
}
```

**1.4 Repository**
```java
@Repository
public interface CategoryRepository extends JpaRepository<Category, Long> {
    boolean existsByName(String name);
}
```

### 2. Response Flow

**2.1 Database to Entity**
```java
// Automatically mapped by JPA
Category {
    id: 1,
    name: "Electronics",
    description: "Electronic devices and accessories"
}
```

**2.2 Entity to DTO**
```java
// In Service Layer
return modelMapper.map(savedCategory, CategoryDTO.class);
```

**2.3 Final Response**
```json
{
    "id": 1,
    "name": "Electronics",
    "description": "Electronic devices and accessories"
}
```

## Getting Started

### Prerequisites
- Java 11 or higher
- Maven 3.6.0 or higher
- MySQL/PostgreSQL (or your preferred database)

### Installation
1. Clone the repository
2. Configure your database in `application.properties`
3. Build the project: `mvn clean install`
4. Run the application: `mvn spring-boot:run`

## API Documentation
API documentation is available at: `http://localhost:8080/swagger-ui.html` (when Swagger is configured)

## Contributing
Please read CONTRIBUTING.md for details on our code of conduct and the process for submitting pull requests.

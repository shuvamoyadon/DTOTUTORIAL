# E-commerce Project

This is a Spring Boot-based E-commerce application with a clear separation of concerns and a well-structured architecture. Below is a detailed explanation of the project structure with code examples for each component.

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

## Detailed Package Explanation with Code Examples

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

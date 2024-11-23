# Simple Validator

A lightweight, tag-based validation library for Go structs. This library provides basic validation functionality using struct tags, making it easy to validate struct fields with minimal setup.

## Features

- Tag-based validation using `validate` tags
- Support for required field validation
- JSON-friendly error messages
- Handles nested structs
- Supports pointer fields
- Uses field names from JSON tags when available

## Installation

```bash
go get github.com/GyroZepelix/simple-validator
```

## Usage

### Basic Usage

Add the `validate` tag to your struct fields that need validation. Currently supported validation rules:
- `required`: Ensures the field is not empty

```go
package main

import (
    "fmt"
    "github.com/GyroZepelix/simple-validator"
)

type User struct {
    Name  string `json:"name" validate:"required"`
    Email string `json:"email" validate:"required"`
    Age   int    // No validation
}

func main() {
    // Valid case
    user := User{
        Name:  "John Doe",
        Email: "john@example.com",
    }
    
    err := validator.Validate(user)
    if err != nil {
        fmt.Printf("Validation failed: %v\n", err)
        return
    }
    fmt.Println("Validation passed!")

    // Invalid case
    invalidUser := User{
        Name: "", // Empty required field
    }
    
    err = validator.Validate(invalidUser)
    if err != nil {
        fmt.Printf("Validation failed: %v\n", err)
        // Will print validation errors for empty Name and Email fields
    }
}
```

### Working with Nested Structs

The validator can handle nested structs and pointers:

```go
type Address struct {
    Street  string `json:"street" validate:"required"`
    City    string `json:"city" validate:"required"`
    Country string `json:"country" validate:"required"`
}

type Customer struct {
    Name    string   `json:"name" validate:"required"`
    Address *Address `json:"address" validate:"required"`
}

func main() {
    customer := Customer{
        Name: "Jane Doe",
        Address: &Address{
            Street:  "123 Main St",
            City:    "", // This will trigger a validation error
            Country: "USA",
        },
    }

    err := validator.Validate(customer)
    if err != nil {
        fmt.Printf("Validation failed: %v\n", err)
    }
}
```

### Error Handling

The validator returns an `ErrValidateError` type that implements the `error` interface. You can type assert to access the validation issues:

```go
if err := simplevalidator.Validate(user); err != nil {
    var validateError validator.ErrValidateError
    if errors.As(err, validateError) {
        for _, issue := range validateError.ValidationIssues {
            fmt.Printf("Field '%s': %s\n", issue.FieldName, issue.Msg)
        }
    }
}
```

The validation error will be returned as JSON in this format:
```json
{
    "validation_issues": [
        {
            "field_name": "name",
            "msg": "Field 'name' is required but empty"
        },
        {
            "field_name": "email",
            "msg": "Field 'email' is required but empty"
        }
    ]
}
```

## Supported Types

The validator currently supports these types for empty checking:
- Strings (empty string check)
- Pointers (nil check)
- Interfaces (nil check)

## Contributing

Contributions are welcome! Feel free to submit issues and pull requests.

## License

This project is licensed under the [MIT License](LICENSE) 

## TODO

Future improvements that could be added:
- Add more validation rules (min/max length, regex patterns, etc.)
- Add custom validation functions
- Add validation for numeric types
- Add validation for slices and maps
- Add support for custom error messages

# RegisteredUser Class and getTotal Method Refactor

This document provides an analysis of issues in the original `getTotal` method in the `RegisteredUser` class and presents a refactored solution for improved flexibility and scalability.

## Problem Analysis

### Issues in the `getTotal` Method

1. **Use of `typeof` Instead of `instanceof`**:

   - The original implementation uses `typeof` to check the class of an instance, which only works for primitive types (like "string", "number", etc.) and not for classes like `StreamingService`, `DownloadService`, or `PremiumContent`.
   - For class instances, `instanceof` should be used instead of `typeof`.

2. **High Dependency on Specific Classes**:

   - The method directly depends on specific classes (`StreamingService`, `DownloadService`, `PremiumContent`), making it rigid and hard to extend.
   - Adding new service or content types would require modifying `getTotal`, which violates the Open-Closed Principle (OCP) of SOLID.

3. **Complex and Non-Scalable Logic**:
   - The logic for calculating `total` involves nested conditionals, making it challenging to scale.
   - Ideally, each service type should handle its own price calculation.

## Refactored Solution

The improved design delegates price calculation to the service and content classes. Each service has a `calculatePrice` method that returns the content cost, while each content type defines its own method for calculating additional charges. This allows `getTotal` to simply iterate over services and sum up the total without handling specific details for each type.

### Refactored Code

```javascript
// RegisteredUser Class
class RegisteredUser {
  constructor(services = []) {
    this.services = services;
  }

  getTotal() {
    let total = 0;

    this.services.forEach((service) => {
      total += service.calculatePrice();
    });

    return total;
  }
}

// Abstract Service Class
class Service {
  constructor(multimediaContent) {
    this.multimediaContent = multimediaContent;
  }

  calculatePrice() {
    return 0; // Default to be overridden by subclasses
  }
}

// Streaming Service
class StreamingService extends Service {
  calculatePrice() {
    let basePrice = this.multimediaContent.streamingPrice;
    return this.multimediaContent.calculateAdditionalFees(basePrice);
  }
}

// Download Service
class DownloadService extends Service {
  calculatePrice() {
    let basePrice = this.multimediaContent.downloadPrice;
    return this.multimediaContent.calculateAdditionalFees(basePrice);
  }
}

// Abstract MultimediaContent Class
class MultimediaContent {
  calculateAdditionalFees(basePrice) {
    return basePrice; // No additional fee by default
  }
}

// Premium Content
class PremiumContent extends MultimediaContent {
  constructor(streamingPrice, downloadPrice, additionalFee) {
    super();
    this.streamingPrice = streamingPrice;
    this.downloadPrice = downloadPrice;
    this.additionalFee = additionalFee;
  }

  calculateAdditionalFees(basePrice) {
    return basePrice + this.additionalFee;
  }
}
```

### Solution Explanation

- **Delegated Price Calculation**: Responsibility for price calculation is moved to `StreamingService`, `DownloadService`, and content classes like `PremiumContent`, allowing `RegisteredUser` to simply accumulate totals.
- **Extensibility**: Adding new service or content types only requires creating a new class with a custom `calculatePrice` or `calculateAdditionalFees` method, without modifying `RegisteredUser`.
- **Implicit Use of `instanceof`**: Instead of manually checking types, each subclass overrides calculation methods, eliminating the need for explicit type checks.

This refactored design adheres to SOLID principles and is more maintainable and scalable for future additions.

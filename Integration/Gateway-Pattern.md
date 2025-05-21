# Gateway Pattern

## Simple Explanation
Think of a hotel concierge who handles all guest requests - whether it's booking a restaurant, calling a taxi, or arranging tours. The Gateway pattern works similarly - it's a single entry point that manages and simplifies access to multiple external services or systems.

## Real-World Scenario
Imagine a mobile app that needs to:
- Fetch weather data from one API
- Get traffic updates from another API
- Send notifications through a third service

Without a Gateway:
- The app needs to know how to talk to each service
- Handle different authentication methods
- Deal with various response formats

With a Gateway:
- The app talks to one Gateway
- The Gateway handles all the complexity
- The app gets data in a consistent format

## Problem Statement
When applications need to interact with multiple external services, they face:
- Different protocols and formats
- Various authentication methods
- Complex error handling
- Inconsistent interfaces
- Multiple dependencies

## Solution
The Gateway pattern provides:
1. Single Entry Point
   - Unified interface for external services
   - Consistent error handling
   - Centralized authentication

2. Service Abstraction
   - Hides external service complexities
   - Translates between protocols
   - Handles format conversions

3. Enhanced Security
   - Centralized authentication
   - Request validation
   - Rate limiting

## Implementation Examples

### TypeScript Implementation
```typescript
interface ExternalService {
    execute(request: any): Promise<any>;
}

class WeatherService implements ExternalService {
    async execute(request: any): Promise<any> {
        // Call weather API
        const response = await fetch('weather-api-url', {
            method: 'GET',
            headers: { 'Authorization': 'weather-api-key' }
        });
        return response.json();
    }
}

class TrafficService implements ExternalService {
    async execute(request: any): Promise<any> {
        // Call traffic API
        const response = await fetch('traffic-api-url', {
            method: 'POST',
            headers: { 'Authorization': 'traffic-api-key' },
            body: JSON.stringify(request)
        });
        return response.json();
    }
}

class ServiceGateway {
    private services: Map<string, ExternalService>;

    constructor() {
        this.services = new Map();
        this.services.set('weather', new WeatherService());
        this.services.set('traffic', new TrafficService());
    }

    async execute(serviceName: string, request: any): Promise<any> {
        const service = this.services.get(serviceName);
        if (!service) {
            throw new Error(`Service ${serviceName} not found`);
        }

        try {
            const result = await service.execute(request);
            return this.formatResponse(result);
        } catch (error) {
            return this.handleError(error);
        }
    }

    private formatResponse(response: any): any {
        // Standardize response format
        return {
            success: true,
            data: response,
            timestamp: new Date()
        };
    }

    private handleError(error: any): any {
        return {
            success: false,
            error: error.message,
            timestamp: new Date()
        };
    }
}
```

### Java Implementation
```java
public interface ExternalService {
    CompletableFuture<Response> execute(Request request);
}

public class WeatherService implements ExternalService {
    @Override
    public CompletableFuture<Response> execute(Request request) {
        return CompletableFuture.supplyAsync(() -> {
            // Call weather API using HttpClient
            HttpClient client = HttpClient.newHttpClient();
            HttpRequest apiRequest = HttpRequest.newBuilder()
                .uri(URI.create("weather-api-url"))
                .header("Authorization", "weather-api-key")
                .build();
            
            return client.sendAsync(apiRequest, BodyHandlers.ofString())
                .thenApply(HttpResponse::body)
                .thenApply(Response::new);
        });
    }
}

public class ServiceGateway {
    private final Map<String, ExternalService> services;
    private final ExecutorService executor;

    public ServiceGateway() {
        this.services = new ConcurrentHashMap<>();
        this.executor = Executors.newCachedThreadPool();
        services.put("weather", new WeatherService());
        services.put("traffic", new TrafficService());
    }

    public CompletableFuture<Response> execute(String serviceName, Request request) {
        ExternalService service = services.get(serviceName);
        if (service == null) {
            return CompletableFuture.failedFuture(
                new ServiceNotFoundException(serviceName));
        }

        return service.execute(request)
            .thenApply(this::formatResponse)
            .exceptionally(this::handleError);
    }

    private Response formatResponse(Response response) {
        // Standardize response format
        return Response.builder()
            .success(true)
            .data(response.getData())
            .timestamp(Instant.now())
            .build();
    }

    private Response handleError(Throwable error) {
        return Response.builder()
            .success(false)
            .error(error.getMessage())
            .timestamp(Instant.now())
            .build();
    }
}
```

### Python Implementation
```python
from abc import ABC, abstractmethod
from typing import Any, Dict
from datetime import datetime
import aiohttp
import asyncio

class ExternalService(ABC):
    @abstractmethod
    async def execute(self, request: Dict[str, Any]) -> Dict[str, Any]:
        pass

class WeatherService(ExternalService):
    async def execute(self, request: Dict[str, Any]) -> Dict[str, Any]:
        async with aiohttp.ClientSession() as session:
            async with session.get(
                'weather-api-url',
                headers={'Authorization': 'weather-api-key'}
            ) as response:
                return await response.json()

class ServiceGateway:
    def __init__(self):
        self.services: Dict[str, ExternalService] = {}
        self.services['weather'] = WeatherService()
        self.services['traffic'] = TrafficService()

    async def execute(self, service_name: str, request: Dict[str, Any]) -> Dict[str, Any]:
        service = self.services.get(service_name)
        if not service:
            raise ValueError(f"Service {service_name} not found")

        try:
            result = await service.execute(request)
            return self._format_response(result)
        except Exception as e:
            return self._handle_error(e)

    def _format_response(self, response: Dict[str, Any]) -> Dict[str, Any]:
        return {
            'success': True,
            'data': response,
            'timestamp': datetime.utcnow().isoformat()
        }

    def _handle_error(self, error: Exception) -> Dict[str, Any]:
        return {
            'success': False,
            'error': str(error),
            'timestamp': datetime.utcnow().isoformat()
        }
```

## Usage Guidelines

1. **When to Use**
   - Multiple external service integrations
   - Need for consistent interface
   - Complex authentication requirements
   - Response format standardization

2. **Configuration Best Practices**
   - Centralize configuration
   - Use dependency injection
   - Implement circuit breakers
   - Add proper logging
   - Include monitoring

3. **Implementation Considerations**
   - Error handling strategy
   - Timeout management
   - Retry policies
   - Rate limiting
   - Caching strategy

## Real-World Examples

1. **API Gateway (AWS/Azure)**
   - Routes requests to microservices
   - Handles authentication
   - Manages rate limiting
   - Provides monitoring

2. **Payment Gateways**
   - Stripe/PayPal integrations
   - Multiple payment provider support
   - Consistent interface
   - Security handling

3. **Mobile Backend Gateway**
   - BFF (Backend for Frontend) pattern
   - Multiple service aggregation
   - Response optimization
   - Device-specific handling

## Key Implementation Differences

1. **TypeScript vs Java**
   - TypeScript uses async/await
   - Java uses CompletableFuture
   - Both support promise-based patterns

2. **Python vs Others**
   - Python uses asyncio
   - Coroutine-based asynchronous code
   - Simple async context managers

3. **Error Handling**
   - TypeScript: Promise rejection
   - Java: CompletableFuture.exceptionally
   - Python: try/except with async

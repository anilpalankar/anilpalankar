In .NET 8.0, you can implement an exception handler by creating a class that implements the `IExceptionHandler` interface. Here's an example:

```csharp
using System;
using Microsoft.Extensions.Logging;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Diagnostics;
using Microsoft.AspNetCore.Http;

public class CustomExceptionHandler : IExceptionHandler
{
    private readonly ILogger<CustomExceptionHandler> _logger;

    public CustomExceptionHandler(ILogger<CustomExceptionHandler> logger)
    {
        _logger = logger;
    }

    public async Task HandleAsync(HttpContext context, ExceptionHandlerDelegate next)
    {
        var exceptionHandlerFeature = context.Features.Get<IExceptionHandlerFeature>();

        if (exceptionHandlerFeature?.Error != null)
        {
            var exception = exceptionHandlerFeature.Error;

            // Log the exception
            _logger.LogError(exception, "An unhandled exception has occurred.");

            // Set the response status code
            context.Response.StatusCode = (int)HttpStatusCode.InternalServerError;

            // Return a custom error message
            context.Response.ContentType = "application/json";
            await context.Response.WriteAsync("An error occurred. Please try again later.");
        }
    }
}
```

To use this exception handler in your .NET 8.0 application, you need to register it in the `ConfigureServices` method of your `Startup` class:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // Register the exception handler
    services.AddSingleton<IExceptionHandler, CustomExceptionHandler>();

    // Other service registrations
    // ...
}
```

Finally, you need to add the exception handler middleware in the `Configure` method of your `Startup` class:

```csharp
public void Configure(IApplicationBuilder app, IExceptionHandler exceptionHandler)
{
    // Other middleware registrations

    // Use the exception handler
    app.UseExceptionHandler(exceptionHandler);

    // Other middleware registrations
    // ...
}
```

Now, whenever an unhandled exception occurs in your application, the `CustomExceptionHandler` will be invoked, logging the exception and returning a custom error message to the client.

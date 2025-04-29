# Securing API in ASP.NET Core

This guide provides a thorough overview of essential techniques and best practices for securing your ASP.NET Core APIs and web applications. Whether you're building a public API or a complex web application, understanding and implementing these security measures is crucial for protecting your users and data.

# **1. Authentication & Authorization**

Authentication and authorization are fundamental security concepts that often get confused. Let's clarify the difference and explore their implementations in ASP.NET Core.

## **Difference between Authentication vs Authorization**

| **Feature** | **Authentication** | **Authorization** |
| --- | --- | --- |
| **Purpose** | Verifying the identity of a user or service. | Determining what an authenticated user can access. |
| **Question** | Who are you? | What are you allowed to do? |
| **Process** | Involves credentials like username/password, tokens. | Involves roles, permissions, and policies. |
| **Outcome** | Establishes the user's identity. | Grants or denies access to specific resources/actions. |

## **Implementing JWT-based authentication for APIs**

JSON Web Tokens (JWTs) are a standard for securely transmitting information between parties as a JSON object. They are commonly used for API authentication.

**Steps:**

1. **Install NuGet Package:**
    
    ```
    Install-Package Microsoft.AspNetCore.Authentication.JwtBearer
    ```
    
2. **Configure JWT Authentication in `Program.cs`:**
    
    ```csharp
    using Microsoft.AspNetCore.Authentication.JwtBearer;
    using Microsoft.IdentityModel.Tokens;
    using System.Text;
    
    var builder = WebApplication.CreateBuilder(args);
    
    // Add authentication
    builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
        .AddJwtBearer(options =>
        {
            options.TokenValidationParameters = new TokenValidationParameters
            {
                ValidateIssuer = true,
                ValidateAudience = true,
                ValidateLifetime = true,
                ValidateIssuerSigningKey = true,
                ValidIssuer = builder.Configuration["Jwt:Issuer"],
                ValidAudience = builder.Configuration["Jwt:Audience"],
                IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(builder.Configuration["Jwt:Key"]))
            };
        });
    
    // Add authorization
    builder.Services.AddAuthorization();
    
    // ... other service configurations ...
    
    var app = builder.Build();
    
    app.UseAuthentication();
    app.UseAuthorization();
    
    // ... other middleware configurations ...
    
    app.Run();
    ```
    
3. **Configure JWT settings in `appsettings.json`:**
    
    ```json
    {
      "Jwt": {
        "Key": "YourSuperSecretKeyHere",
        "Issuer": "yourdomain.com",
        "Audience": "yourclient.com"
      },
      // ... other settings ...
    }
    ```
    
4. **Create an endpoint to issue JWTs (e.g., /api/auth/login):**
    
    This endpoint would typically take user credentials, validate them, and then generate a JWT containing claims about the user's identity and roles.
    
    ```csharp
    using Microsoft.AspNetCore.Authorization;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.IdentityModel.Tokens;
    using System.IdentityModel.Tokens.Jwt;
    using System.Security.Claims;
    using System.Text;
    
    [ApiController]
    [Route("api/auth")]
    public class AuthController : ControllerBase
    {
        private readonly IConfiguration _config;
    
        public AuthController(IConfiguration config)
        {
            _config = config;
        }
    
        [AllowAnonymous]
        [HttpPost("login")]
        public IActionResult Login([FromBody] UserLoginDto userLogin)
        {
            // Authenticate the user (replace with your actual logic)
            if (!AuthenticateUser(userLogin.Username, userLogin.Password))
            {
                return Unauthorized();
            }
    
            var token = GenerateJwtToken(userLogin.Username);
            return Ok(new { Token = token });
        }
    
        private bool AuthenticateUser(string username, string password)
        {
            // Replace with your actual user authentication logic (e.g., database lookup)
            return username == "testuser" && password == "password";
        }
    
        private string GenerateJwtToken(string username)
        {
            var claims = new List<Claim>
            {
                new Claim(ClaimTypes.Name, username),
                new Claim(ClaimTypes.Role, "User") // Example role
            };
    
            var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(_config["Jwt:Key"]));
            var creds = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);
    
            var token = new JwtSecurityToken(
                issuer: _config["Jwt:Issuer"],
                audience: _config["Jwt:Audience"],
                claims: claims,
                expires: DateTime.Now.AddHours(1),
                signingCredentials: creds
            );
    
            return new JwtSecurityTokenHandler().WriteToken(token);
        }
    }
    
    public class UserLoginDto
    {
        public string Username { get; set; }
        public string Password { get; set; }
    }
    ```
    

## **Using ASP.NET Core Identity for web apps**

ASP.NET Core Identity is a robust and flexible system for managing users, passwords, profiles, role claims, tokens, email confirmation, and more in web applications.

**Steps:**

1. **Install NuGet Packages:**
    
    ```
    Install-Package Microsoft.AspNetCore.Identity.EntityFrameworkCore
    Install-Package Microsoft.EntityFrameworkCore.SqlServer (or your database provider)
    ```
    
2. **Configure Identity in `Program.cs`:**
    
    ```csharp
    using Microsoft.AspNetCore.Identity;
    using Microsoft.EntityFrameworkCore;
    using YourWebApp.Data; // Replace with your DbContext namespace
    
    var builder = WebApplication.CreateBuilder(args);
    
    // Add DbContext (replace with your configuration)
    builder.Services.AddDbContext<ApplicationDbContext>(options =>
        options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));
    
    // Add Identity
    builder.Services.AddDefaultIdentity<IdentityUser>(options => options.SignIn.RequireConfirmedAccount = true)
        .AddRoles<IdentityRole>() // Enable roles
        .AddEntityFrameworkStores<ApplicationDbContext>();
    
    // ... other service configurations ...
    
    var app = builder.Build();
    
    // ... other middleware configurations ...
    
    app.Run();
    
    ```
    
3. **Create `ApplicationDbContext` (inherits from `IdentityDbContext`):**
    
    ```
    using Microsoft.AspNetCore.Identity.EntityFrameworkCore;
    using Microsoft.EntityFrameworkCore;
    
    namespace YourWebApp.Data
    {
        public class ApplicationDbContext : IdentityDbContext
        {
            public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
                : base(options)
            {
            }
    
            // Add your custom entities here, if any
        }
    }
    
    ```
    
4. **Create database migrations and apply them:**
    
    ```
    dotnet ef migrations add InitialIdentity
    dotnet ef database update
    ```
    
5. **Use Identity features in your controllers and views:**
    
    You can use UserManager<IdentityUser>, SignInManager<IdentityUser>, and RoleManager<IdentityRole> (injected via dependency injection) to manage users, sign-in, and roles.
    

## **External providers (Google, Facebook, Microsoft, etc.)**

ASP.NET Core allows you to easily integrate with external authentication providers using OAuth 2.0 and OpenID Connect.

**Steps (example for Google):**

1. **Install NuGet Package:**
    
    ```
    Install-Package Microsoft.AspNetCore.Authentication.Google
    ```
    
2. **Register your application with the provider (e.g., Google Cloud Console) to get Client ID and Client Secret.**
3. **Configure the provider in `Program.cs`:**
    
    ```
    builder.Services.AddAuthentication()
        .AddGoogle(options =>
        {
            options.ClientId = builder.Configuration["Authentication:Google:ClientId"];
            options.ClientSecret = builder.Configuration["Authentication:Google:ClientSecret"];
            options.CallbackPath = "/signin-google"; // Default callback path
        });
    ```
    
4. **Configure the `Authentication:Google` section in `appsettings.json`:**
    
    ```csharp
    {
      "Authentication": {
        "Google": {
          "ClientId": "YOUR_GOOGLE_CLIENT_ID",
          "ClientSecret": "YOUR_GOOGLE_CLIENT_SECRET"
        }
      },
      // ... other settings ...
    }
    ```
    
5. **Implement login and callback logic in your controllers/pages.** ASP.NET Core Identity handles much of the complexity behind the scenes.

## **Role-based and Policy-based authorization**

ASP.NET Core offers two main approaches for authorization:

- **Role-based authorization:** Controls access based on the roles assigned to a user.
- **Policy-based authorization:** A more flexible approach that allows you to define custom authorization rules (policies) based on claims, roles, or any other criteria.

## **Using `[Authorize]`, `[AllowAnonymous]`, and custom policies**

- `[Authorize]`: Applied to controllers or actions to require the user to be authenticated.
- `[AllowAnonymous]`: Applied to controllers or actions to bypass authentication requirements, even if the controller has the `[Authorize]` attribute.
- **Role-based authorization:** Use the `Roles` property of the `[Authorize]` attribute:
    
    ```csharp
    [Authorize(Roles = "Admin,Editor")]
    public IActionResult AdminPanel()
    {
        // ...
        return View();
    }
    ```
    
- **Policy-based authorization:**
    1. **Define policies in `Program.cs`:**
        
        ```csharp
        builder.Services.AddAuthorization(options =>
        {
            options.AddPolicy("RequireAdminRole", policy =>
                policy.RequireRole("Admin"));
        
            options.AddPolicy("RequireClaim", policy =>
                policy.RequireClaim("Permission", "View"));
        
            options.AddPolicy("MinimumAge", policy =>
                policy.Requirements.Add(new MinimumAgeRequirement(18)));
        });
        
        // Custom requirement class
        public class MinimumAgeRequirement : IAuthorizationRequirement
        {
            public MinimumAgeRequirement(int minimumAge) => MinimumAge = minimumAge;
            public int MinimumAge { get; }
        }
        
        // Custom handler class
        public class MinimumAgeHandler : AuthorizationHandler<MinimumAgeRequirement>
        {
            protected override Task HandleRequirementAsync(AuthorizationHandlerContext context, MinimumAgeRequirement requirement)
            {
                var dateOfBirthClaim = context.User.FindFirst(claim => claim.Type == ClaimTypes.DateOfBirth);
        
                if (dateOfBirthClaim != null && DateTime.TryParse(dateOfBirthClaim.Value, out var dateOfBirth))
                {
                    var age = DateTime.Today.Year - dateOfBirth.Year;
                    if (dateOfBirth > DateTime.Today.AddYears(-age))
                    {
                        age--;
                    }
        
                    if (age >= requirement.MinimumAge)
                    {
                        context.Succeed(requirement);
                    }
                }
        
                return Task.CompletedTask;
            }
        }
        
        builder.Services.AddSingleton<IAuthorizationHandler, MinimumAgeHandler>();
        ```
        
    2. **Apply policies to controllers or actions:**
        
        ```csharp
        [Authorize(Policy = "RequireAdminRole")]
        public IActionResult AdminAction()
        {
            // ...
            return View();
        }
        
        [Authorize(Policy = "MinimumAge")]
        public IActionResult RestrictedContent()
        {
            // ...
            return View();
        }
        ```
        

## **Securing endpoints and controllers effectively**

- **Apply `[Authorize]` attribute by default at the controller level and use `[AllowAnonymous]` for specific public actions.** This follows the principle of least privilege.
- **Be specific with your authorization rules.** Avoid overly broad roles or policies.
- **Test your authorization logic thoroughly** to ensure that only authorized users can access protected resources.

# **2. API Security Best Practices**

Beyond authentication and authorization, several other best practices are crucial for securing your APIs.

## **Validating input (Model validation + custom filters)**

- **Use Data Annotations (`[Required]`, `[MaxLength]`, `[EmailAddress]`, etc.) in your models to enforce basic validation rules.** ASP.NET Core automatically handles this during model binding.
- **Implement custom validation logic using `IValidatableObject` interface or custom validation attributes** for more complex scenarios.
- **Use FluentValidation library for a more powerful and expressive validation experience.**
- **Implement global exception handling and return informative error responses** for validation failures (e.g., HTTP 400 Bad Request with details of validation errors).
- **Consider using custom action filters to perform additional validation or sanitization of input.**
    
    ```csharp
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.AspNetCore.Mvc.Filters;
    using System.Linq;
    
    public class ValidateModelAttribute : ActionFilterAttribute
    {
        public override void OnActionExecuting(ActionExecutingContext context)
        {
            if (!context.ModelState.IsValid)
            {
                context.Result = new BadRequestObjectResult(context.ModelState);
            }
        }
    }
    
    [ApiController]
    [Route("api/[controller]")]
    [ValidateModel] // Apply the filter to the controller
    public class ExampleController : ControllerBase
    {
        [HttpPost]
        public IActionResult Post([FromBody] MyModel model)
        {
            // Model validation is handled by the filter
            // ... your logic ...
            return Ok();
        }
    }
    
    public class MyModel
    {
        [Required]
        [MaxLength(100)]
        public string Name { get; set; }
    
        [EmailAddress]
        public string Email { get; set; }
    }
    ```
    

## **Enforcing HTTPS and HSTS**

- **HTTPS (HTTP Secure) encrypts communication between the client and the server, protecting data in transit.** Configure your server with an SSL/TLS certificate and ensure your application redirects HTTP requests to HTTPS.
- **HSTS (HTTP Strict Transport Security) is a security enhancement that instructs browsers to only interact with your site over HTTPS.**
    
    **Configuration in `Program.cs`:**
    
    ```csharp
    var app = builder.Build();
    
    // Configure HTTPS redirection
    app.UseHttpsRedirection();
    
    // Configure HSTS (recommended for production)
    app.UseHsts();
    
    // ... other middleware ...
    
    app.Run();
    ```
    

## **API Key authentication (for external access)**

API keys are often used for simpler authentication scenarios, especially for third-party access to public APIs.

**Implementation:**

1. **Generate unique API keys for clients.**
2. **Decide how to transmit the API key (e.g., query parameter, request header).** Request headers are generally preferred.
3. **Create custom middleware to check for the API key:**
    
    ```csharp
    using Microsoft.AspNetCore.Builder;
    using Microsoft.AspNetCore.Http;
    using System.Threading.Tasks;
    
    public class ApiKeyAuthenticationMiddleware
    {
        private readonly RequestDelegate _next;
        private readonly IConfiguration _configuration;
    
        public ApiKeyAuthenticationMiddleware(RequestDelegate next, IConfiguration configuration)
        {
            _next = next;
            _configuration = configuration;
        }
    
        public async Task InvokeAsync(HttpContext context)
        {
            if (!context.Request.Path.StartsWithSegments("/api"))
            {
                await _next(context);
                return;
            }
    
            if (!context.Request.Headers.TryGetValue("X-API-Key", out var apiKeyHeaderValues))
            {
                context.Response.StatusCode = 401; // Unauthorized
                await context.Response.WriteAsync("API Key is missing.");
                return;
            }
    
            var apiKey = apiKeyHeaderValues.FirstOrDefault();
            if (string.IsNullOrEmpty(apiKey) || !ValidateApiKey(apiKey))
            {
                context.Response.StatusCode = 401; // Unauthorized
                await context.Response.WriteAsync("Invalid API Key.");
                return;
            }
    
            // API key is valid, proceed to the next middleware
            await _next(context);
        }
    
        private bool ValidateApiKey(string apiKey)
        {
            var validApiKey = _configuration["ApiSettings:ApiKey"]; // Store in configuration
            return apiKey == validApiKey;
        }
    }
    
    public static class ApiKeyAuthenticationMiddlewareExtensions
    {
        public static IApplicationBuilder UseApiKeyAuthentication(this IApplicationBuilder builder)
        {
            return builder.UseMiddleware<ApiKeyAuthenticationMiddleware>();
        }
    }
    ```
    
4. **Configure the middleware in `Program.cs`:**
    
    ```csharp
    var app = builder.Build();
    
    app.UseApiKeyAuthentication(); // Add before UseAuthentication and UseAuthorization
    
    // ... other middleware ...
    
    app.Run();
    ```
    

## **Rate limiting and throttling (using AspNetCoreRateLimit or custom middleware)**

Rate limiting and throttling help protect your API from abuse (e.g., DDoS attacks) by restricting the number of requests a client can make within a specific time window.

- **AspNetCoreRateLimit:** A popular NuGet package for implementing rate limiting.
    1. **Install NuGet Packages:**
        
        ```
        Install-Package AspNetCoreRateLimit
        ```
        
    2. **Configure rate limiting services in `Program.cs`:**
        
        ```csharp
        using AspNetCoreRateLimit;
        
        // ... inside ConfigureServices method ...
        builder.Services.AddMemoryCache(); // Required for AspNetCoreRateLimit
        builder.Services.Configure<IpRateLimitOptions>(options =>
        {
            options.EnableEndpointRateLimiting = true;
            options.StackBlockedRequests = false;
            options.HttpStatusCode = 429; // Too Many Requests
            options.RealIpHeaderKey = "X-Real-IP"; //  Header for proxy scenarios
            options.GeneralRules = new List<RateLimitRule>
            {
                new RateLimitRule
                {
                    Endpoint = "*", // Apply to all endpoints
                    Limit = 100,     // Max requests
                    Period = "1m"    // Per minute
                }
            };
        });
        builder.Services.AddSingleton<IIpPolicyStore, MemoryCacheIpPolicyStore>();
        builder.Services.AddSingleton<IRateLimitCounterStore, MemoryCacheRateLimitCounterStore>();
        builder.Services.AddSingleton<IRateLimitConfiguration, RateLimitConfiguration>();
        builder.Services.AddSingleton<IProcessingEngine, AsyncKeyLockProcessingEngine>();
        
        // ...
        
        ```
        
    3. **Configure the middleware in `Program.cs`:**
        
        ```csharp
        var app = builder.Build();
        app.UseIpRateLimiting();
        // ...
        
        ```
        
- **Custom middleware:** You can also create custom middleware for more fine-grained control.

## **Avoiding over-posting and under-posting (DTOs + AutoMapper)**

- **Over-posting:** Occurs when a client sends more data than the server expects, potentially allowing them to modify properties they shouldn't.
- **Under-posting:** Occurs when a client doesn't send all the required data, leading to incomplete or invalid data on the server.
- **DTOs (Data Transfer Objects):** Define specific classes to handle the data being sent to and from your API, containing only the properties that should be exposed or modified.
- **AutoMapper:** A library that simplifies the process of mapping between your domain models and DTOs.
    
    **Example:**
    
    1. **Install NuGet Package:**
        
        ```
        Install-Package AutoMapper
        Install-Package AutoMapper.Extensions.Microsoft.DependencyInjection
        
        ```
        
    2. **Create DTOs:**
        
        ```csharp
        public class ProductDto
        {
            public int Id { get; set; }
            [Required]
            public string Name { get; set; }
            public string Description { get; set; }
            public decimal Price { get; set; }
        }
        
        public class CreateProductDto
        {
            [Required]
            public string Name { get; set; }
            public string Description { get; set; }
            public decimal Price { get; set; }
        }
        
        public class UpdateProductDto
        {
            [Required]
            public int Id { get; set; }
            public string Name { get; set; }
            public string Description { get; set; }
            public decimal Price { get; set; }
        }
        ```
        
    3. **Configure AutoMapper in `Program.cs`:**
        
        ```csharp
        using AutoMapper;
        
        // ...
        builder.Services.AddAutoMapper(typeof(Startup)); //  Add AutoMapper
        // ...
        
        //Create a profile
        public class AutoMapperProfile : Profile
        {
            public AutoMapperProfile()
            {
               CreateMap<Product, ProductDto>();
               CreateMap<CreateProductDto, Product>();
               CreateMap<UpdateProductDto, Product>();
            }
        }
        ```
        
    4. **Use DTOs in your controllers:**
        
        ```csharp
        using AutoMapper;
        using Microsoft.AspNetCore.Mvc;
        using System.Collections.Generic;
        using System.Threading.Tasks;
        
        [ApiController]
        [Route("api/products")]
        public class ProductsController : ControllerBase
        {
            private readonly IProductRepository _productRepository; //  replace with your repos
            private readonly IMapper _mapper;
        
            public ProductsController(IProductRepository productRepository, IMapper mapper)
            {
                _productRepository = productRepository;
                _mapper = mapper;
            }
        
            [HttpGet]
            public async Task<ActionResult<IEnumerable<ProductDto>>> GetProducts()
            {
                var products = await _productRepository.GetAllAsync();
                var productDtos = _mapper.Map<IEnumerable<ProductDto>>(products);
                return Ok(productDtos);
            }
        
            [HttpGet("{id}")]
            public async Task<ActionResult<ProductDto>> GetProduct(int id)
            {
                var product = await _productRepository.GetByIdAsync(id);
                if (product == null)
                {
                    return NotFound();
                }
                var productDto = _mapper.Map<ProductDto>(product);
                return Ok(productDto);
            }
        
            [HttpPost]
            public async Task<ActionResult<ProductDto>> CreateProduct(CreateProductDto createProductDto)
            {
                var product = _mapper.Map<Product>(createProductDto);
                await _productRepository.AddAsync(product);
                var productDto = _mapper.Map<ProductDto>(product);
                return CreatedAtAction(nameof(GetProduct), new { id = product.Id }, productDto);
            }
        
            [HttpPut("{id}")]
            public async Task<IActionResult> UpdateProduct(int id, UpdateProductDto updateProductDto)
            {
                if (id != updateProductDto.Id)
                {
                    return BadRequest();
                }
        
                var existingProduct = await _productRepository.GetByIdAsync(id);
                if (existingProduct == null)
                {
                    return NotFound();
                }
        
                _mapper.Map(updateProductDto, existingProduct); // Update the existing entity
                await _productRepository.UpdateAsync(existingProduct);
                return NoContent();
            }
        
            [HttpDelete("{id}")]
            public async Task<IActionResult> DeleteProduct(int id)
            {
                var product = await _productRepository.GetByIdAsync(id);
                if (product == null)
                {
                    return NotFound();
                }
        
                await _productRepository.DeleteAsync(product);
                return NoContent();
            }
        }
        
        ```
        

## **Hiding sensitive data from responses**

- **Never include sensitive data (e.g., passwords, API keys, credit card numbers) in your API responses,** even if they are stored securely on the server.
- **Use DTOs** to control exactly what data is included in the response.
- **Remove sensitive properties** from your domain models before serializing them.
- **Use the `[JsonIgnore]` attribute** from the `System.Text.Json.Serialization` namespace to exclude properties from serialization.

## **Using CORS to control cross-origin access**

- **CORS (Cross-Origin Resource Sharing) is a mechanism that allows or restricts requests from different domains.** By default, web browsers block cross-origin requests for security reasons.
- **Configure CORS in your ASP.NET Core application to specify which origins are allowed to access your API.**
    
    **Configuration in `Program.cs`:**
    
    ```csharp
    var builder = WebApplication.CreateBuilder(args);
    
    builder.Services.AddCors(options =>
    {
        options.AddPolicy("AllowSpecificOrigin", policy =>
        {
            policy.WithOrigins("https://example.com", "https://anotherdomain.com")  //  allowed origins
                   .AllowAnyHeader()
                   .AllowAnyMethod();
        });
    
        options.AddPolicy("AllowAll", policy =>
        {
            policy.AllowAnyOrigin()
                  .AllowAnyHeader()
                  .AllowAnyMethod();
        });
    });
    
    // ...
    
    var app = builder.Build();
    
    app.UseCors("AllowSpecificOrigin"); //  Use the policy
    //or
    app.UseCors("AllowAll");
    
    // ...
    app.Run();
    
    ```
    

## **Handling authentication errors securely (custom error responses)**

- **Avoid revealing sensitive information in error messages.** For example, don't tell an attacker whether a username exists or not.
- **Return standard HTTP status codes** (e.g., 401 Unauthorized, 403 Forbidden) to indicate the type of error.
- **Create custom error response objects** to provide consistent and user-friendly error messages.
- **Use exception handling middleware** to catch exceptions and convert them into appropriate error responses.
    
    ```csharp
    using Microsoft.AspNetCore.Builder;
    using Microsoft.AspNetCore.Diagnostics;
    using Microsoft.AspNetCore.Http;
    using System.Text.Json;
    using System.Threading.Tasks;
    
    public static class ExceptionHandlerMiddlewareExtensions
    {
        public static IApplicationBuilder UseCustomExceptionHandler(this IApplicationBuilderbuilder)
        {
            return builder.UseExceptionHandler(app =>
            {
                app.Run(async context =>
                {
                    context.Response.StatusCode = 500; // Internal Server Error
                    context.Response.ContentType = "application/json";
    
                    var errorFeature = context.Features.Get<IExceptionHandlerFeature>();
                    if (errorFeature != null)
                    {
                        var error = errorFeature.Error;
    
                        var errorResponse = new
                        {
                            Message = "An unexpected error occurred.",
                            Details = error.Message //  Include for development only.
                        };
    
                        //  Serialize the error
                        var json = JsonSerializer.Serialize(errorResponse);
                        await context.Response.WriteAsync(json);
                    }
                });
            });
        }
    }
    
    //configure
    var app = builder.Build();
    app.UseCustomExceptionHandler();
    //...
    ```
    

# **3. Tools & Middleware**

ASP.NET Core provides built-in middleware and tools to help you secure your applications.

## **Using built-in ASP.NET Core Middleware for security**

ASP.NET Core middleware is a series of components that process requests and responses. Several built-in middleware components enhance security:

- **`UseHttpsRedirection`:** Redirects HTTP requests to HTTPS.
- **`UseHsts`:** Sends the Strict-Transport-Security header to clients.
- **`UseAuthentication`:** Enables authentication.
- **`UseAuthorization`:** Enables authorization.
- **`UseCors`**: Enables Cross-Origin Requests.
- **`UseExceptionHandler`**: Handles unhandled exceptions.

## **Adding custom middleware to inspect or log security threats**

You can create custom middleware to implement specific security logic, such as:

- **Logging suspicious activity:** Log requests with unusual patterns or potential attacks.
- **Blocking specific IPs:** Block requests from known malicious IP addresses.
- **Inspecting request headers:** Check for malicious or unexpected headers.
- **Implementing custom authentication/authorization schemes.**

## **Logging and alerting with Application Insights or Serilog**

- **Logging:** Record security-related events (e.g., failed login attempts, authorization failures) to help with monitoring and auditing. Serilog is a good option.
- **Alerting:** Configure alerts to notify you of critical security events in real-time. Application Insights is a good option for Azure.

# **Summary / Best Practices Checklist**

- ✅ Implement authentication to verify user identity.
- ✅ Implement authorization to control access to resources.
- ✅ Use JWT for API authentication.
- ✅ Use ASP.NET Core Identity for web application user management.
- ✅ Enforce HTTPS and HSTS.
- ✅ Validate all user input.
- ✅ Use DTOs to prevent over-posting and under-posting.
- ✅ Hide sensitive data in API responses.
- ✅ Use CORS to control cross-origin access.
- ✅ Handle authentication errors securely.
- ✅ Use rate limiting to protect against abuse.
- ✅ Log security-related events.
- ✅ Consider using API keys for external access.
- ✅ Keep your ASP.NET Core framework and libraries up to date.
- ✅ Regularly review and test your application's security.

---

# **References**

- **Microsoft Docs - ASP.NET Core Security:** [https://learn.microsoft.com/en-us/aspnet/core/security/](https://learn.microsoft.com/en-us/aspnet/core/security/)
- **OWASP:** [https://owasp.org/](https://owasp.org/)
### USER :
great actually we had discussion with client , they expect some differnt kind of login mechanism and we need to store much login details , so according to our ER diagram lets implement the logic , 
i dont want to use Idendity DB context , because if we have controll over our code then it is easy to adapt & in future we are going for DynamoDB as well

### RESPONSE :
That is a perfectly valid and strong architectural decision.

Choosing to implement your own identity logic instead of using the built-in `IdentityDbContext` gives you **maximum control and flexibility**. It makes the system simpler to understand, avoids the "black box" nature of a large framework, and crucially, makes the future migration to DynamoDB **significantly easier**.

You are absolutely right: if DynamoDB is the end goal, avoiding a deep dependency on a relational-specific framework like EF Core Identity is the smart move.

Let's proceed with this plan. I will provide the complete code for building out the user registration and login flow based on our finalized ER diagram.

---

### **Building a Custom Identity Flow (Users, Auth, JWT)**

Here is the step-by-step implementation plan and code.

#### **Step 1: Update the Core Project (`TamilApp.Core`)**

We need models for our user and DTOs for handling login/register requests and responses.

**1. Create the `User` Model**
This will map directly to the `Users` table we designed.

```csharp
// TamilApp.Core/Models/User.cs
namespace TamilApp.Core.Models
{
    public class User
    {
        public int UserId { get; set; }
        public string Username { get; set; } = string.Empty;
        public string Email { get; set; } = string.Empty;
        public string PasswordHash { get; set; } = string.Empty;
        public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
    }
}
```

**2. Create Request/Response DTOs**
These define the "shape" of the data for our API endpoints.

```csharp
// TamilApp.Core/Dtos/RegisterDto.cs
using System.ComponentModel.DataAnnotations;
namespace TamilApp.Core.Dtos
{
    public class RegisterDto
    {
        [Required]
        [StringLength(50, MinimumLength = 3)]
        public string Username { get; set; } = string.Empty;

        [Required]
        [EmailAddress]
        public string Email { get; set; } = string.Empty;

        [Required]
        [StringLength(100, MinimumLength = 6)]
        public string Password { get; set; } = string.Empty;
    }
}

// TamilApp.Core/Dtos/LoginDto.cs
using System.ComponentModel.DataAnnotations;
namespace TamilApp.Core.Dtos
{
    public class LoginDto
    {
        [Required]
        public string Username { get; set; } = string.Empty;

        [Required]
        public string Password { get; set; } = string.Empty;
    }
}

// TamilApp.Core/Dtos/AuthResponseDto.cs
namespace TamilApp.Core.Dtos
{
    public class AuthResponseDto
    {
        public bool IsSuccess { get; set; }
        public string Message { get; set; } = string.Empty;
        public string? Token { get; set; }
    }
}
```

#### **Step 2: Update the Infrastructure Project (`TamilApp.Infrastructure`)**

We need a repository and service for handling users and authentication.

**1. Update `AppDbContext.cs`**
Add the `Users` table to your DbContext.

```csharp
// TamilApp.Infrastructure/Data/AppDbContext.cs
// ... (add this inside the class)
public DbSet<User> Users { get; set; }

// ... (add this inside OnModelCreating)
modelBuilder.Entity<User>().HasIndex(u => u.Username).IsUnique();
modelBuilder.Entity<User>().HasIndex(u => u.Email).IsUnique();
```

**2. Create `IAuthRepository` and `AuthRepository`**
This repository will handle all direct database interactions related to users.

```csharp
// TamilApp.Infrastructure/Repositories/IAuthRepository.cs
using TamilApp.Core.Models;
namespace TamilApp.Infrastructure.Repositories
{
    public interface IAuthRepository
    {
        Task<User?> GetUserByUsernameAsync(string username);
        Task<bool> UserExistsAsync(string username, string email);
        Task<User> RegisterUserAsync(User user);
    }
}

// TamilApp.Infrastructure/Repositories/AuthRepository.cs
using Microsoft.EntityFrameworkCore;
using TamilApp.Core.Models;
namespace TamilApp.Infrastructure.Repositories
{
    public class AuthRepository : IAuthRepository
    {
        private readonly AppDbContext _context;
        public AuthRepository(AppDbContext context) { _context = context; }

        public async Task<User?> GetUserByUsernameAsync(string username)
        {
            return await _context.Users.FirstOrDefaultAsync(u => u.Username.ToLower() == username.ToLower());
        }

        public async Task<bool> UserExistsAsync(string username, string email)
        {
            return await _context.Users.AnyAsync(u => u.Username.ToLower() == username.ToLower() || u.Email.ToLower() == email.ToLower());
        }

        public async Task<User> RegisterUserAsync(User user)
        {
            await _context.Users.AddAsync(user);
            await _context.SaveChangesAsync();
            return user;
        }
    }
}
```

**3. Create `IAuthService` and `AuthService`**
This service will contain the core business logic: hashing passwords, verifying them, and creating JWTs.

*   **Install necessary packages** in the `TamilApp.Api` project:
    *   `Microsoft.AspNetCore.Authentication.JwtBearer`
    *   `System.IdentityModel.Tokens.Jwt`

```csharp
// TamilApp.Infrastructure/Services/IAuthService.cs
using TamilApp.Core.Dtos;
namespace TamilApp.Infrastructure.Services
{
    public interface IAuthService
    {
        Task<AuthResponseDto> RegisterAsync(RegisterDto registerDto);
        Task<AuthResponseDto> LoginAsync(LoginDto loginDto);
    }
}


// TamilApp.Infrastructure/Services/AuthService.cs
// NOTE: This file has dependencies on ASP.NET Core libraries for JWT creation.
// It's a common pattern to place it in the Infrastructure layer.
using Microsoft.Extensions.Configuration;
using Microsoft.IdentityModel.Tokens;
using System.IdentityModel.Tokens.Jwt;
using System.Security.Claims;
using System.Text;
using TamilApp.Core.Dtos;
using TamilApp.Core.Models;
using TamilApp.Infrastructure.Repositories;

namespace TamilApp.Infrastructure.Services
{
    public class AuthService : IAuthService
    {
        private readonly IAuthRepository _authRepository;
        private readonly IConfiguration _configuration;

        public AuthService(IAuthRepository authRepository, IConfiguration configuration)
        {
            _authRepository = authRepository;
            _configuration = configuration;
        }

        public async Task<AuthResponseDto> RegisterAsync(RegisterDto registerDto)
        {
            if (await _authRepository.UserExistsAsync(registerDto.Username, registerDto.Email))
            {
                return new AuthResponseDto { IsSuccess = false, Message = "Username or Email already exists." };
            }

            // Hash the password
            string passwordHash = BCrypt.Net.BCrypt.HashPassword(registerDto.Password);

            var user = new User
            {
                Username = registerDto.Username,
                Email = registerDto.Email,
                PasswordHash = passwordHash
            };

            await _authRepository.RegisterUserAsync(user);

            return new AuthResponseDto { IsSuccess = true, Message = "User registered successfully." };
        }

        public async Task<AuthResponseDto> LoginAsync(LoginDto loginDto)
        {
            var user = await _authRepository.GetUserByUsernameAsync(loginDto.Username);

            if (user == null || !BCrypt.Net.BCrypt.Verify(loginDto.Password, user.PasswordHash))
            {
                return new AuthResponseDto { IsSuccess = false, Message = "Invalid username or password." };
            }

            // Create JWT Token
            var token = CreateToken(user);

            return new AuthResponseDto { IsSuccess = true, Message = "Login successful.", Token = token };
        }

        private string CreateToken(User user)
        {
            var claims = new List<Claim>
            {
                new Claim(ClaimTypes.NameIdentifier, user.UserId.ToString()),
                new Claim(ClaimTypes.Name, user.Username)
            };

            var appSettingsToken = _configuration.GetSection("AppSettings:Token").Value;
            if (appSettingsToken is null)
                throw new Exception("AppSettings Token is null!");
            
            SymmetricSecurityKey key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(appSettingsToken));
            SigningCredentials creds = new SigningCredentials(key, SecurityAlgorithms.HmacSha512Signature);

            var tokenDescriptor = new SecurityTokenDescriptor
            {
                Subject = new ClaimsIdentity(claims),
                Expires = DateTime.Now.AddDays(1),
                SigningCredentials = creds
            };

            JwtSecurityTokenHandler tokenHandler = new JwtSecurityTokenHandler();
            SecurityToken token = tokenHandler.CreateToken(tokenDescriptor);

            return tokenHandler.WriteToken(token);
        }
    }
}
```
*Note: I've used the `BCrypt.Net-Next` library for password hashing. You'll need to install it: `Install-Package BCrypt.Net-Next`*

#### **Step 3: Update the API Project (`TamilApp.Api`)**

**1. Create `AuthController`**

```csharp
// TamilApp.Api/Controllers/AuthController.cs
using Microsoft.AspNetCore.Mvc;
using TamilApp.Core.Dtos;
using TamilApp.Infrastructure.Services;

[ApiController]
[Route("api/[controller]")]
public class AuthController : ControllerBase
{
    private readonly IAuthService _authService;
    public AuthController(IAuthService authService)
    {
        _authService = authService;
    }

    [HttpPost("register")]
    public async Task<IActionResult> Register(RegisterDto registerDto)
    {
        var response = await _authService.RegisterAsync(registerDto);
        if (!response.IsSuccess)
        {
            return BadRequest(response);
        }
        return Ok(response);
    }

    [HttpPost("login")]
    public async Task<IActionResult> Login(LoginDto loginDto)
    {
        var response = await _authService.LoginAsync(loginDto);
        if (!response.IsSuccess)
        {
            return Unauthorized(response);
        }
        return Ok(response);
    }
}
```

**2. Configure `Program.cs`**
Update `Program.cs` to register the new services and configure JWT authentication.

```csharp
// TamilApp.Api/Program.cs
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.IdentityModel.Tokens;
using System.Text;
// ... other usings

var builder = WebApplication.CreateBuilder(args);

// ... Add DbContext and other services as before

// Add Auth services for Dependency Injection
builder.Services.AddScoped<IAuthRepository, AuthRepository>();
builder.Services.AddScoped<IAuthService, AuthService>();

// Configure JWT Authentication
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuerSigningKey = true,
            IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(
                builder.Configuration.GetSection("AppSettings:Token").Value!)),
            ValidateIssuer = false,
            ValidateAudience = false
        };
    });

// ... AddControllers, Swagger, etc.

var app = builder.Build();

// ...
app.UseHttpsRedirection();

// IMPORTANT: Add these two lines between UseHttpsRedirection and UseAuthorization
app.UseAuthentication(); 
app.UseAuthorization();

app.MapControllers();
app.Run();
```

**3. Update `appsettings.json`**
Add the secret key for signing your JWTs. **Make this key long and random. Do not commit a real production key to GitHub.**

```json
{
  "AppSettings": {
    "Token": "your-super-secret-key-that-is-long-and-random-12345"
  },
  "ConnectionStrings": { ... },
  "Logging": { ... }
}
```

You now have a complete, custom-built authentication system. You have full control over the database schema and logic, which perfectly sets you up for future adaptations, including the migration to DynamoDB. You can now use the `[Authorize]` attribute on any controller or endpoint that should only be accessible to logged-in users.


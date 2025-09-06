This is a critical feature for any application with public user registration. You've correctly identified the two main components: **Email Verification** and **Forgot Password**.

Let's design a secure, robust, and user-friendly workflow for this. Using a redirect link with a token is the standard and best practice for this, and it works perfectly with mobile apps.

---

### **Architectural Overview**

The core principle for both features is the same:
1.  **User Action:** A user registers or requests a password reset.
2.  **API Generates a Token:** The backend generates a secure, single-use, time-limited token and saves it to the database, associated with the user.
3.  **API Sends an Email:** The backend sends an email to the user containing a special link. This link includes the token.
4.  **User Clicks the Link:** The user opens their email and clicks the link.
5.  **Mobile App Handles the Link:** The mobile app is configured to handle this specific link format (this is called **Deep Linking**). It opens the app and passes the token to it.
6.  **App Verifies the Token:** The mobile app sends the token back to a special API endpoint for validation.
7.  **API Completes the Action:** The backend validates the token, marks the email as verified or allows the user to set a new password, and then deletes the token so it cannot be used again.

---

### **Part 1: Backend Implementation (ASP.NET Core)**

We'll need a way to send emails. I'll use a simple interface so you can plug in any email provider (like SendGrid, Mailgun, or AWS SES).

#### **Step 1.1: Domain and Infrastructure Changes**

1.  **Update `User` Entity:** Add fields for email verification.
    **File: `Domain/Entities/User.cs`**
    ```csharp
    public class User
    {
        // ...
        public bool IsEmailVerified { get; set; } = false;
        public string? VerificationToken { get; set; }
        public DateTime? VerificationTokenExpiry { get; set; }
        public string? PasswordResetToken { get; set; }
        public DateTime? PasswordResetTokenExpiry { get; set; }
    }
    ```
2.  **Create and Apply a Migration** to add these new columns.

3.  **Create an `IEmailService` Interface:**
    **File: `Application/Services/Interfaces/IEmailService.cs`**
    ```csharp
    public interface IEmailService
    {
        Task SendEmailAsync(string toEmail, string subject, string body);
    }
    ```
4.  **Create a simple implementation (for now):**
    You can create a dummy implementation in the `Infrastructure` layer that just logs the email to the console during development. Later, you'll replace this with a real email provider's SDK.
    **File: `Infrastructure/Services/LogEmailService.cs`**
    ```csharp
    public class LogEmailService : IEmailService
    {
        private readonly ILogger<LogEmailService> _logger;
        public LogEmailService(ILogger<LogEmailService> logger) => _logger = logger;

        public Task SendEmailAsync(string toEmail, string subject, string body)
        {
            _logger.LogWarning("---- SENDING EMAIL (LOG ONLY) ----");
            _logger.LogInformation("To: {ToEmail}", toEmail);
            _logger.LogInformation("Subject: {Subject}", subject);
            _logger.LogInformation("Body:\n{Body}", body);
            _logger.LogWarning("---------------------------------");
            return Task.CompletedTask;
        }
    }
    ```
5.  **Register `IEmailService` in DI:**
    In `InfrastructureServiceExtensions.cs`, add `services.AddSingleton<IEmailService, LogEmailService>();`.

#### **Step 1.2: Update the `AuthService`**

Modify the `RegisterAsync` method and add new methods for verification and password resets.

**File: `Application/Services/Implementation/AuthService.cs` (Expanded)**
```csharp
public class AuthService : IAuthService
{
    private readonly IUnitOfWork _unitOfWork;
    private readonly IEmailService _emailService; // Inject the email service
    private readonly IConfiguration _configuration;

    public AuthService(IUnitOfWork unitOfWork, IEmailService emailService, IConfiguration configuration)
    {
        _unitOfWork = unitOfWork;
        _emailService = emailService;
        _configuration = configuration;
    }

    public async Task<AuthResponseDto> RegisterAsync(RegisterRequestDto registerDto)
    {
        // ... (check if user exists)

        var user = new User
        {
            // ...,
            Role = "Student",
            VerificationToken = GenerateSecureToken() // Generate a token
        };
        await _unitOfWork.Users.CreateAsync(user);
        await _unitOfWork.CompleteAsync();

        // Send the verification email
        var verificationLink = $"{_configuration["AppSettings:ClientAppUrl"]}/verify-email?token={user.VerificationToken}";
        var emailBody = $"<p>Please verify your email by clicking <a href='{verificationLink}'>here</a>.</p>";
        await _emailService.SendEmailAsync(user.Email, "Verify Your Email", emailBody);

        // Don't return tokens until email is verified
        return new AuthResponseDto { IsSuccess = true, Message = "Registration successful. Please check your email to verify your account." };
    }
    
    // --- NEW METHODS ---

    public async Task<bool> VerifyEmailAsync(string token)
    {
        var user = await _unitOfWork.Users.GetUserByVerificationTokenAsync(token); // New repo method

        if (user == null) return false;

        user.IsEmailVerified = true;
        user.VerificationToken = null; // Invalidate the token
        await _unitOfWork.CompleteAsync();
        return true;
    }

    public async Task ForgotPasswordAsync(string email)
    {
        var user = await _unitOfWork.Users.GetUserByEmailAsync(email); // New repo method
        if (user == null) return; // Don't reveal if the email exists or not for security

        user.PasswordResetToken = GenerateSecureToken();
        user.PasswordResetTokenExpiry = DateTime.UtcNow.AddHours(1); // Token is valid for 1 hour
        await _unitOfWork.CompleteAsync();
        
        var resetLink = $"{_configuration["AppSettings:ClientAppUrl"]}/reset-password?token={user.PasswordResetToken}";
        var emailBody = $"<p>To reset your password, click <a href='{resetLink}'>here</a>. This link is valid for one hour.</p>";
        await _emailService.SendEmailAsync(user.Email, "Password Reset Request", emailBody);
    }

    public async Task<bool> ResetPasswordAsync(ResetPasswordRequestDto dto)
    {
        var user = await _unitOfWork.Users.GetUserByPasswordResetTokenAsync(dto.Token); // New repo method
        if (user == null || user.PasswordResetTokenExpiry < DateTime.UtcNow)
        {
            return false; // Token is invalid or expired
        }

        user.PasswordHash = BCrypt.Net.BCrypt.HashPassword(dto.NewPassword);
        user.PasswordResetToken = null; // Invalidate the token
        user.PasswordResetTokenExpiry = null;
        await _unitOfWork.CompleteAsync();
        return true;
    }

    private string GenerateSecureToken() => Convert.ToBase64String(RandomNumberGenerator.GetBytes(64));
}
```
*Note: You'll need to add `ClientAppUrl` to `appsettings.json`, e.g., `"ClientAppUrl": "myapp://"`*

#### **Step 1.3: Update `AuthController` with New Endpoints**

**File: `Web/Controllers/AuthController.cs` (Add these methods)**
```csharp
[HttpPost("verify-email")]
public async Task<IActionResult> VerifyEmail([FromBody] TokenRequestDto dto)
{
    var success = await _authService.VerifyEmailAsync(dto.Token);
    if (!success) return BadRequest(new { message = "Invalid or expired token." });
    return Ok(new { message = "Email verified successfully. You can now log in." });
}

[HttpPost("forgot-password")]
public async Task<IActionResult> ForgotPassword([FromBody] EmailRequestDto dto)
{
    await _authService.ForgotPasswordAsync(dto.Email);
    // Always return a success message to prevent email enumeration attacks
    return Ok(new { message = "If an account with that email exists, a password reset link has been sent." });
}

[HttpPost("reset-password")]
public async Task<IActionResult> ResetPassword([FromBody] ResetPasswordRequestDto dto)
{
    var success = await _authService.ResetPasswordAsync(dto);
    if (!success) return BadRequest(new { message = "Invalid or expired token." });
    return Ok(new { message = "Password has been reset successfully." });
}
```
*(You will need to create the DTOs: `TokenRequestDto`, `EmailRequestDto`, `ResetPasswordRequestDto`)*

---

### **Part 2: Mobile App Integration (React Native with Expo)**

This is how your mobile app will handle the links.

#### **Step 2.1: Configure Deep Linking in Expo**

You need to tell your Expo app what kind of URLs it should listen for.

In your `app.json` (or `app.config.js`), add a `scheme`:
```json
{
  "expo": {
    "scheme": "myapp", // This is the myapp:// part of the link
    // ... other config
  }
}
```
When you build your app, this will configure the native projects to open the app when a URL like `myapp://verify-email?token=...` is clicked.

#### **Step 2.2: Handle the Incoming Link in Your App**

Expo provides hooks to listen for incoming URLs. You can set this up in your main app component or navigation container.

```javascript
import * as Linking from 'expo-linking';
import { useEffect } from 'react';
import { useNavigation } from '@react-navigation/native';

const AppNavigator = () => {
    const navigation = useNavigation();

    // This effect runs once when the app loads
    useEffect(() => {
        // Handle a URL that opened the app from a killed state
        const handleInitialUrl = async () => {
            const initialUrl = await Linking.getInitialURL();
            if (initialUrl) {
                handleDeepLink(initialUrl);
            }
        };
        handleInitialUrl();

        // Listen for a URL that opens the app while it's already running
        const subscription = Linking.addEventListener('url', (event) => {
            handleDeepLink(event.url);
        });

        return () => subscription.remove();
    }, []);

    const handleDeepLink = (url: string) => {
        const { path, queryParams } = Linking.parse(url);
        
        console.log("Deep Link Path:", path);
        console.log("Query Params:", queryParams);

        if (path === 'verify-email' && queryParams?.token) {
            // Navigate to a special, hidden screen to process the token
            navigation.navigate('EmailVerificationScreen', { token: queryParams.token });
        }
        if (path === 'reset-password' && queryParams?.token) {
            // Navigate to the password reset screen
            navigation.navigate('PasswordResetScreen', { token: queryParams.token });
        }
    };
    
    return (
        // ... Your Stack Navigator ...
    );
};
```

*   **`EmailVerificationScreen`:** This screen would be a simple loading screen. When it mounts, it takes the `token` from its route params, calls your `POST /api/auth/verify-email` endpoint, and on success, shows a "Success! You can now log in" message and navigates the user to the login screen.
*   **`PasswordResetScreen`:** This screen would show a form with "New Password" and "Confirm Password" fields. When the user submits the form, it calls your `POST /api/auth/reset-password` endpoint, sending the new password and the token.

This is a complete, secure, and user-friendly flow for handling email verification and password resets in a mobile application.

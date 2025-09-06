That is a fantastic and very logical next step. Building a user management section into the admin panel is essential for a real-world application.

Your idea to introduce a **"Teacher" role** is brilliant. It expands the scope of the application in a very valuable way and sets the stage for a much richer feature set.

Let's design the complete User Management feature, including the necessary backend changes and the frontend UI.

---

### **Architectural Plan: Multi-Role User Management**

#### **1. Updated Role Hierarchy**

We will now have three distinct roles with a clear hierarchy of permissions:
1.  **`Student`:** (Default role) Can only access the mobile app, log in, and update their own progress. They have **no access** to the admin panel.
2.  **`Teacher`:** Can access a limited version of the admin panel. They can **view** all users (especially students) and **track their progress**, but they **cannot** edit content (Levels, Lessons, etc.).
3.  **`Admin`:** (Superuser) Has full access to everything. They can manage content, manage users (create, edit, delete, change roles), and view all student progress.

#### **2. Backend Changes Required**

*   **User Entity:** The `Role` property on our `User` entity is already perfect for this.
*   **New Endpoints in `UsersController`:** We need to build out a full CRUD API for managing users, protected by role-based authorization.
*   **New Endpoints for Progress Tracking:** We need endpoints for Admins/Teachers to view the progress of other users.

#### **3. Frontend (React Admin Panel) Changes Required**

*   **New `UsersPage`:** A new page in the admin panel to list, create, edit, and delete users. This will use our `InlineCrudTable`.
*   **New `UserProgressPage`:** A page that shows the detailed progress of a single student.
*   **Role-Based UI:** The admin panel UI will need to show or hide certain features based on the logged-in user's role (e.g., a Teacher won't see the "Levels" or "Lessons" links in the navigation).

---

### **Implementation Plan (Step-by-Step)**

#### **Part 1: Backend API Enhancements**

**1. Create New DTOs for User Management**

```csharp
// Application/DTOs/Response/UserDto.cs
public class UserDto
{
    public int UserId { get; set; }
    public string Username { get; set; }
    public string Email { get; set; }
    public string Role { get; set; }
    public DateTime CreatedAt { get; set; }
}

// Application/DTOs/Request/CreateUserRequestDto.cs
public class CreateUserRequestDto
{
    [Required] public string Username { get; set; }
    [Required] [EmailAddress] public string Email { get; set; }
    [Required] [MinLength(6)] public string Password { get; set; }
    [Required] public string Role { get; set; } // "Admin" or "Teacher" or "Student"
}

// Application/DTOs/Request/UpdateUserRequestDto.cs
public class UpdateUserRequestDto
{
    [Required] public string Username { get; set; }
    [Required] [EmailAddress] public string Email { get; set; }
    [Required] public string Role { get; set; }
    // Password is not updated here, it should be a separate "reset password" action.
}
```

**2. Build out the `UsersController`**
This controller will be protected so that only Admins can access it.

**File: `Web/Controllers/UsersController.cs` (Expanded)**
```csharp
[ApiController]
[Route("api/[controller]")]
[Authorize(Roles = "Admin")] // Only Admins can manage users
public class UsersController : ControllerBase
{
    private readonly IUserService _userService;
    public UsersController(IUserService userService) => _userService = userService;

    [HttpGet]
    public async Task<IActionResult> GetAllUsers()
    {
        var users = await _userService.GetAllUsersAsync();
        return Ok(users);
    }

    [HttpPost]
    public async Task<IActionResult> CreateUser([FromBody] CreateUserRequestDto dto)
    {
        var newUser = await _userService.CreateUserAsync(dto);
        return CreatedAtAction(nameof(GetUserById), new { id = newUser.UserId }, newUser);
    }

    [HttpPut("{id}")]
    public async Task<IActionResult> UpdateUser(int id, [FromBody] UpdateUserRequestDto dto)
    {
        var updatedUser = await _userService.UpdateUserAsync(id, dto);
        return Ok(updatedUser);
    }

    [HttpDelete("{id}")]
    public async Task<IActionResult> DeleteUser(int id)
    {
        await _userService.DeleteUserAsync(id);
        return NoContent();
    }
    
    // Endpoint for getting a single user
    [HttpGet("{id}")]
    public async Task<IActionResult> GetUserById(int id) 
    {
        var user = await _userService.GetUserByIdAsync(id);
        return Ok(user);
    }
}
```
*(You will need to build out the `IUserService` and `IUserRepository` with these corresponding CRUD methods).*

**3. Create New Endpoints in `ProgressController` for Teachers/Admins**
These endpoints will allow authorized users to view the progress of others.

**File: `Web/Controllers/ProgressController.cs` (Add these new methods)**
```csharp
[ApiController]
[Route("api/progress")] // Change base route for clarity
[Authorize] // All endpoints require login
public class ProgressController : ControllerBase
{
    // ... existing "current-lesson" and "complete-activity" endpoints for students ...
    // These should be marked with [Authorize(Roles = "Student")]

    // --- NEW ENDPOINTS FOR TEACHERS/ADMINS ---
    
    [HttpGet("user/{userId}/summary")]
    [Authorize(Roles = "Admin, Teacher")] // Allow both roles
    public async Task<IActionResult> GetUserProgressSummary(int userId)
    {
        // This service method would return high-level progress:
        // e.g., current lesson, total activities completed, etc.
        var summary = await _progressService.GetUserProgressSummaryAsync(userId);
        return Ok(summary);
    }

    [HttpGet("user/{userId}/detailed")]
    [Authorize(Roles = "Admin, Teacher")]
    public async Task<IActionResult> GetUserDetailedProgress(int userId)
    {
        // This service method would return the full logbook of every
        // completed activity from the UserProgress table.
        var detailedProgress = await _progressService.GetDetailedProgressForUserAsync(userId);
        return Ok(detailedProgress);
    }
}
```
*(This requires new DTOs and new methods in your `IProgressService` and `IProgressRepository` to fetch progress data for a specific `userId`.)*

---

### **Part 2: Frontend React Admin Panel Implementation**

#### **1. Role-Based Navigation in `App.tsx`**

We need to know the logged-in user's role.

*   **Update `AuthContext.tsx`:**
    *   First, decode the JWT to get the user's role and name. Install `jwt-decode`: `npm install jwt-decode`.
    *   Store the user info in the context state.

    ```typescript
    // In AuthContext.tsx
    import { jwtDecode } from 'jwt-decode';

    interface UserInfo {
        name: string;
        role: string;
    }
    
    // Add to AuthContextType
    interface AuthContextType {
        user: UserInfo | null;
        // ...
    }

    // Inside AuthProvider
    const [user, setUser] = useState<UserInfo | null>(null);

    // When logging in or on initial load:
    const token = authService.getAccessToken();
    if (token) {
        const decodedToken: { unique_name: string, role: string } = jwtDecode(token);
        setUser({ name: decodedToken.unique_name, role: decodedToken.role });
    }

    // In logout function:
    const logout = () => {
        // ...
        setUser(null);
    }
    ```

*   **Update `App.tsx` to show links based on role:**
    ```typescript
    // In App.tsx's AppContent component
    const { isAuthenticated, user, logout } = useAuth();
    
    // In the AppBar
    {user?.role === 'Admin' && (
        <>
            <Button component={RouterLink} to="/levels" color="inherit">Levels</Button>
            <Button component={RouterLink} to="/lessons" color="inherit">Lessons</Button>
            {/* ... other admin-only links */}
        </>
    )}
    {(user?.role === 'Admin' || user?.role === 'Teacher') && (
         <Button component={RouterLink} to="/users" color="inherit">User Management</Button>
    )}
    <Button onClick={logout} color="inherit">Logout</Button>
    ```

#### **2. Create the `UsersPage.tsx`**

This page will use our `InlineCrudTable` to manage users.

```typescript
// In pages/UsersPage.tsx

// Define columns
const columns = [
    { field: 'username', headerName: 'Username', type: 'string' },
    { field: 'email', headerName: 'Email', type: 'string' },
    { field: 'role', headerName: 'Role', type: 'string' } 
    // For editing 'role', the renderEditCell would render a dropdown with ['Admin', 'Teacher', 'Student']
];

// Define custom actions
const renderCustomUserActions = (user: User) => (
    <Button 
        component={RouterLink} 
        to={`/user-progress/${user.userId}`}
        variant="outlined" size="small"
    >
        View Progress
    </Button>
);

// Render the InlineCrudTable with this configuration...
```
This is a high-level plan, but it shows how your existing components can be reused to quickly build this powerful new feature. You now have a complete roadmap for adding multi-tenancy and detailed progress tracking to your application.

Of course. Let's build this entire User Management feature step-by-step with the complete code for both the backend and frontend. This will be a comprehensive implementation.

---

### **Part 1: Backend Implementation (ASP.NET Core)**

#### **Step 1.1: Create the DTOs**

**File: `src/ICEDT_TamilApp.Application/DTOs/Response/UserDto.cs`**
```csharp
namespace ICEDT_TamilApp.Application.DTOs.Response
{
    public class UserDto
    {
        public int UserId { get; set; }
        public string Username { get; set; } = string.Empty;
        public string Email { get; set; } = string.Empty;
        public string Role { get; set; } = string.Empty;
        public DateTime CreatedAt { get; set; }
    }
}
```

**File: `src/ICEDT_TamilApp.Application/DTOs/Request/CreateUserRequestDto.cs`**
```csharp
using System.ComponentModel.DataAnnotations;

namespace ICEDT_TamilApp.Application.DTOs.Request
{
    public class CreateUserRequestDto
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

        [Required]
        public string Role { get; set; } = string.Empty; // e.g., "Admin", "Teacher", "Student"
    }
}
```

**File: `src/ICEDT_TamilApp.Application/DTOs/Request/UpdateUserRequestDto.cs`**
```csharp
using System.ComponentModel.DataAnnotations;

namespace ICEDT_TamilApp.Application.DTOs.Request
{
    public class UpdateUserRequestDto
    {
        [Required]
        [StringLength(50, MinimumLength = 3)]
        public string Username { get; set; } = string.Empty;

        [Required]
        [EmailAddress]
        public string Email { get; set; } = string.Empty;

        [Required]
        public string Role { get; set; } = string.Empty;
    }
}
```

#### **Step 1.2: Create the `UserService` and `UserRepository`**

**File: `src/ICEDT_TamilApp.Domain/Interfaces/IUserRepository.cs`**
```csharp
using ICEDT_TamilApp.Domain.Entities;
using System.Collections.Generic;
using System.Threading.Tasks;

namespace ICEDT_TamilApp.Domain.Interfaces
{
    public interface IUserRepository
    {
        Task<User?> GetByIdAsync(int id);
        Task<List<User>> GetAllAsync();
        Task<User> CreateAsync(User user);
        Task UpdateAsync(User user);
        Task DeleteAsync(int id);
    }
}
```

**File: `src/ICEDT_TamilApp.Infrastructure/Repositories/UserRepository.cs`**
```csharp
// This file will be simple because UnitOfWork handles the saving.
namespace ICEDT_TamilApp.Infrastructure.Repositories
{
    public class UserRepository : IUserRepository
    {
        private readonly ApplicationDbContext _context;
        public UserRepository(ApplicationDbContext context) => _context = context;

        public async Task<User?> GetByIdAsync(int id) => await _context.Users.FindAsync(id);
        public async Task<List<User>> GetAllAsync() => await _context.Users.OrderBy(u => u.Username).ToListAsync();
        public async Task<User> CreateAsync(User user) { await _context.Users.AddAsync(user); return user; }
        public async Task UpdateAsync(User user) { _context.Users.Update(user); await Task.CompletedTask; }
        public async Task DeleteAsync(int id)
        {
            var user = await GetByIdAsync(id);
            if (user != null) _context.Users.Remove(user);
        }
    }
}
```

**File: `src/ICEDT_TamilApp.Application/Services/Interfaces/IUserService.cs`**
```csharp
namespace ICEDT_TamilApp.Application.Services.Interfaces
{
    public interface IUserService
    {
        Task<List<UserDto>> GetAllUsersAsync();
        Task<UserDto> GetUserByIdAsync(int id);
        Task<UserDto> CreateUserAsync(CreateUserRequestDto dto);
        Task<UserDto> UpdateUserAsync(int id, UpdateUserRequestDto dto);
        Task DeleteUserAsync(int id);
    }
}
```

**File: `src/ICEDT_TamilApp.Application/Services/Implementation/UserService.cs`**
```csharp
namespace ICEDT_TamilApp.Application.Services.Implementation
{
    public class UserService : IUserService
    {
        private readonly IUnitOfWork _unitOfWork;
        public UserService(IUnitOfWork unitOfWork) => _unitOfWork = unitOfWork;

        public async Task<List<UserDto>> GetAllUsersAsync()
        {
            var users = await _unitOfWork.Users.GetAllAsync();
            return users.Select(MapToUserDto).ToList();
        }
        
        public async Task<UserDto> GetUserByIdAsync(int id)
        {
            var user = await _unitOfWork.Users.GetByIdAsync(id);
            if (user == null) throw new NotFoundException(nameof(User), id);
            return MapToUserDto(user);
        }

        public async Task<UserDto> CreateUserAsync(CreateUserRequestDto dto)
        {
            if (await _unitOfWork.Auth.UserExistsAsync(dto.Username, dto.Email))
                throw new ConflictException("Username or Email already exists.");

            var user = new User
            {
                Username = dto.Username,
                Email = dto.Email,
                PasswordHash = BCrypt.Net.BCrypt.HashPassword(dto.Password),
                Role = dto.Role,
                CreatedAt = DateTime.UtcNow
            };
            await _unitOfWork.Users.CreateAsync(user);
            await _unitOfWork.CompleteAsync();
            return MapToUserDto(user);
        }

        public async Task<UserDto> UpdateUserAsync(int id, UpdateUserRequestDto dto)
        {
            var user = await _unitOfWork.Users.GetByIdAsync(id);
            if (user == null) throw new NotFoundException(nameof(User), id);

            user.Username = dto.Username;
            user.Email = dto.Email;
            user.Role = dto.Role;
            
            await _unitOfWork.Users.UpdateAsync(user);
            await _unitOfWork.CompleteAsync();
            return MapToUserDto(user);
        }

        public async Task DeleteUserAsync(int id)
        {
            await _unitOfWork.Users.DeleteAsync(id);
            await _unitOfWork.CompleteAsync();
        }

        private UserDto MapToUserDto(User user) => new UserDto
        {
            UserId = user.UserId,
            Username = user.Username,
            Email = user.Email,
            Role = user.Role,
            CreatedAt = user.CreatedAt
        };
    }
}
```

**Step 1.3: Update `IUnitOfWork` and `UnitOfWork`**
*   **`IUnitOfWork.cs`:** Add `IUserRepository Users { get; }`
*   **`UnitOfWork.cs`:** Add `public IUserRepository Users { get; private set; }` and initialize it in the constructor: `Users = new UserRepository(_context);`

**Step 1.4: Create the `UsersController`**
This is the same controller we designed in the last step, now fully functional.

**Step 1.5: Register Dependencies**
*   **`ApplicationServiceExtensions.cs`:** `services.AddScoped<IUserService, UserService>();`
*   **`InfrastructureServiceExtensions.cs`:** The `IUnitOfWork` registration is enough.

---

### **Part 2: Frontend Implementation (React)**

#### **Step 2.1: Update `AuthContext.tsx` to Decode and Store User Info**

**1. Install `jwt-decode`:**
   ```bash
   npm install jwt-decode
   ```

**2. Modify `AuthContext.tsx`:**
```typescript
// At top
import { jwtDecode } from 'jwt-decode';

// New interface for the decoded user info
export interface UserInfo {
    userId: string;
    username: string;
    role: string;
}

// Update the context type
interface AuthContextType {
    isAuthenticated: boolean;
    user: UserInfo | null; // Changed from boolean to object
    // ...
}

// Inside AuthProvider component
const [user, setUser] = useState<UserInfo | null>(null);

// In the login function, after getting a token
const handleLoginSuccess = (token: string) => {
    try {
        const decoded: { nameid: string, unique_name: string, role: string } = jwtDecode(token);
        const userInfo: UserInfo = {
            userId: decoded.nameid,
            username: decoded.unique_name,
            role: decoded.role
        };
        setUser(userInfo);
        setAccessToken(token);
    } catch (error) {
        console.error("Failed to decode token", error);
        logout();
    }
};

// Use this in your login and useEffect
useEffect(() => {
    const token = authService.getAccessToken();
    if (token) {
        handleLoginSuccess(token);
    }
    setIsLoading(false);
}, []);

const login = async (credentials: LoginRequest): Promise<AuthResponse> => {
    const response = await authApi.login(credentials);
    if (response.isSuccess && response.token) {
        authService.saveTokens(response);
        handleLoginSuccess(response.token);
    }
    return response;
};

// Update logout function
const logout = () => {
    authService.clearTokens();
    setAccessToken(null);
    setUser(null);
};

// Update the value provided by the context
const value = { isAuthenticated: !!user, user, accessToken, ... };
```

#### **Step 2.2: Create `UsersPage.tsx`**

This will be a new page for admins to manage all users.

*   **Create `src/types/user.ts`:**
    ```typescript
    export interface User {
        userId: number;
        username: string;
        email: string;
        role: string;
        createdAt: string;
    }
    ```
*   **Create `src/api/userApi.ts`:**
    This will have `getAll`, `create`, `update`, `deleteItem` functions calling `/api/users`.

*   **Create `src/pages/UsersPage.tsx`:**
```typescript
import React from 'react';
import { Button } from '@mui/material';
import { Link as RouterLink } from 'react-router-dom';
import InlineCrudTable, { ColumnDef } from '../components/common/InlineCrudTable';
import { User } from '../types/user';
import * as userApi from '../api/userApi';

// This is just a placeholder type for the create/update DTO
type UserCreateDto = Omit<User, 'userId' | 'createdAt'> & { password?: string };

const UsersPage: React.FC = () => {
    const columns: ColumnDef<User, UserCreateDto>[] = [
        { field: 'username', headerName: 'Username', type: 'string' },
        { field: 'email', headerName: 'Email', type: 'string' },
        {
            field: 'role',
            headerName: 'Role',
            // Custom editor for the role field: a dropdown
            renderEditCell: (value, onChange) => (
                <select 
                    value={value} 
                    onChange={(e) => onChange('role', e.target.value)}
                    className="form-control" // simple bootstrap-like class
                >
                    <option value="Student">Student</option>
                    <option value="Teacher">Teacher</option>
                    <option value="Admin">Admin</option>
                </select>
            )
        },
    ];

    const apiService = {
        getAll: userApi.getAll,
        create: userApi.create,
        update: userApi.update,
        delete: userApi.deleteItem
    };

    // We can add a "View Progress" button here for Teachers and Admins
    const renderCustomUserActions = (user: User) => (
        <Button 
            component={RouterLink} 
            to={`/progress/${user.userId}`}
            variant="outlined" 
            size="small"
        >
            View Progress
        </Button>
    );

    return (
        <InlineCrudTable<User, UserCreateDto>
            entityName="User"
            apiService={apiService}
            columns={columns}
            idField="userId"
            renderCustomActions={renderCustomUserActions}
        />
    );
};

export default UsersPage;
```

#### **Step 2.3: Update `App.tsx` with Role-Based Navigation**

This is the final step to tie everything together.

```typescript
// In App.tsx's AppContent component
const { isAuthenticated, user, logout } = useAuth();

// ...
<AppBar position="static">
    <Toolbar>
        <Typography variant="h6" sx={{ flexGrow: 1 }}>Admin Panel</Typography>
        
        {/* Links for Admins and Teachers */}
        {(user?.role === 'Admin' || user?.role === 'Teacher') && (
            <Button component={RouterLink} to="/users" color="inherit">User Management</Button>
        )}

        {/* Links for Admins ONLY */}
        {user?.role === 'Admin' && (
            <>
                <Button component={RouterLink} to="/levels" color="inherit">Levels</Button>
                <Button component={RouterLink} to="/lessons" color="inherit">Lessons</Button>
                {/* ... other content management links */}
            </>
        )}

        {isAuthenticated && <Button onClick={logout} color="inherit">Logout</Button>}
    </Toolbar>
</AppBar>

<Container>
    <Routes>
        {/* Public Route */}
        <Route path="/login" element={<LoginPage />} />
        
        {/* Admin & Teacher Routes */}
        <Route path="/users" element={<PrivateRoute><UsersPage /></PrivateRoute>} />
        {/* Add progress page route here later */}

        {/* Admin ONLY Routes */}
        <Route path="/levels" element={<PrivateRoute><LevelsPage /></PrivateRoute>} />
        {/* ... other admin-only routes ... */}

        {/* ... default redirect ... */}
    </Routes>
</Container>
```
You now have a complete, end-to-end implementation of a multi-role user management system. Admins can create and manage users, and the UI will adapt based on whether a Teacher or an Admin is logged in.

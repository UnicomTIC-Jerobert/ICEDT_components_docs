### USER :
shall we proceed to the remaining things for user progress and related stuffs

### RESPONSE :
Of course. Now that we have a solid authentication system in place, building the user progress and lesson access logic is the next critical step. This will bring your "sequential learning path" requirement to life.

Let's implement the logic for:
1.  **Creating progress** for a new user.
2.  **Fetching the current lesson** for a logged-in user.
3.  **Updating progress** when an activity is completed.

---

### **Step 1: Update the Core Project (`TamilApp.Core`)**

We need to create the models for our progress tables.

**1. Create `UserCurrentProgress` and `UserProgress` Models**

```csharp
// TamilApp.Core/Models/UserCurrentProgress.cs
namespace TamilApp.Core.Models
{
    public class UserCurrentProgress
    {
        // This forms a one-to-one relationship with the User table
        public int UserId { get; set; }
        public int CurrentLessonId { get; set; }
        public DateTime LastActivityAt { get; set; } = DateTime.UtcNow;

        public virtual User? User { get; set; }
        public virtual Lesson? CurrentLesson { get; set; }
    }
}

// TamilApp.Core/Models/UserProgress.cs
namespace TamilApp.Core.Models
{
    public class UserProgress
    {
        public long ProgressId { get; set; }
        public int UserId { get; set; }
        public int ActivityId { get; set; }
        public bool IsCompleted { get; set; }
        public int? Score { get; set; }
        public DateTime CompletedAt { get; set; }

        public virtual User? User { get; set; }
        public virtual Activity? Activity { get; set; }
    }
}
```
*Note: We also need to add the `Activity` model, which was designed earlier.*

```csharp
// TamilApp.Core/Models/Activity.cs
namespace TamilApp.Core.Models
{
    public class Activity
    {
        public int ActivityId { get; set; }
        public int LessonId { get; set; }
        public int ActivityTypeId { get; set; }
        public string? Title { get; set; }
        public int SequenceOrder { get; set; }
        public string ContentJson { get; set; } = string.Empty;

        public virtual Lesson? Lesson { get; set; }
        public virtual ActivityType? ActivityType { get; set; }
    }
}
```

---

### **Step 2: Update the Infrastructure Project (`TamilApp.Infrastructure`)**

**1. Update `AppDbContext.cs`**
Add the new `DbSet`s and configure the one-to-one relationship.

```csharp
// TamilApp.Infrastructure/Data/AppDbContext.cs
// ...
public DbSet<UserCurrentProgress> UserCurrentProgress { get; set; }
public DbSet<UserProgress> UserProgress { get; set; }
public DbSet<Activity> Activities { get; set; } // Add this if not already present
public DbSet<ActivityType> ActivityTypes { get; set; } // Add this if not already present

protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    base.OnModelCreating(modelBuilder);
    // ... existing configurations
    
    // Configure one-to-one relationship for UserCurrentProgress
    modelBuilder.Entity<UserCurrentProgress>()
        .HasKey(ucp => ucp.UserId); // UserId is both PK and FK

    modelBuilder.Entity<User>()
        .HasOne(u => u.UserCurrentProgress) // Add navigation property to User model
        .WithOne(ucp => ucp.User)
        .HasForeignKey<UserCurrentProgress>(ucp => ucp.UserId);
}
```
*You'll need to add a navigation property to your `User.cs` model:*
```csharp
// TamilApp.Core/Models/User.cs
// ...
public virtual UserCurrentProgress? UserCurrentProgress { get; set; }
```

**2. Create a `ProgressService` (and Repository)**
This service will handle all logic related to fetching and updating user progress.

```csharp
// TamilApp.Infrastructure/Repositories/IProgressRepository.cs
using TamilApp.Core.Models;
public interface IProgressRepository
{
    Task CreateInitialProgressAsync(int userId, int firstLessonId);
    Task<UserCurrentProgress?> GetCurrentProgressAsync(int userId);
    Task<int> GetTotalActivitiesForLessonAsync(int lessonId);
    Task<int> GetCompletedActivitiesForLessonAsync(int userId, int lessonId);
    Task MarkActivityAsCompleteAsync(int userId, int activityId, int? score);
    Task UpdateCurrentLessonAsync(int userId, int newLessonId);
    Task<Lesson?> GetNextLessonAsync(int currentLessonId);
}

// TamilApp.Infrastructure/Repositories/ProgressRepository.cs
// (Abridged for clarity - this is where you'd write the EF Core queries)
public class ProgressRepository : IProgressRepository
{
    private readonly AppDbContext _context;
    public ProgressRepository(AppDbContext context) { _context = context; }
    
    // Implementation of all the interface methods...
    public async Task CreateInitialProgressAsync(int userId, int firstLessonId)
    {
        var initialProgress = new UserCurrentProgress
        {
            UserId = userId,
            CurrentLessonId = firstLessonId
        };
        await _context.UserCurrentProgress.AddAsync(initialProgress);
        await _context.SaveChangesAsync();
    }
    
    public async Task MarkActivityAsCompleteAsync(int userId, int activityId, int? score)
    {
        var existingProgress = await _context.UserProgress
            .FirstOrDefaultAsync(p => p.UserId == userId && p.ActivityId == activityId);

        if (existingProgress == null)
        {
            var newProgress = new UserProgress
            {
                UserId = userId,
                ActivityId = activityId,
                IsCompleted = true,
                Score = score,
                CompletedAt = DateTime.UtcNow
            };
            await _context.UserProgress.AddAsync(newProgress);
        }
        // else, you could update the score if you allow re-tries
        
        await _context.SaveChangesAsync();
    }

    // ... other method implementations
}

// TamilApp.Infrastructure/Services/IProgressService.cs
using TamilApp.Core.Dtos; // You will need DTOs for this
public interface IProgressService
{
    Task<LessonDto> GetCurrentLessonForUserAsync(int userId);
    Task<bool> CompleteActivityAsync(int userId, int activityId, int? score);
}

// TamilApp.Infrastructure/Services/ProgressService.cs
public class ProgressService : IProgressService
{
    private readonly IProgressRepository _progressRepository;
    private readonly ILessonRepository _lessonRepository; // To fetch lesson data
    public ProgressService(IProgressRepository progressRepository, ILessonRepository lessonRepository) 
    { 
        _progressRepository = progressRepository;
        _lessonRepository = lessonRepository;
    }

    public async Task<LessonDto> GetCurrentLessonForUserAsync(int userId)
    {
        var currentProgress = await _progressRepository.GetCurrentProgressAsync(userId);
        if (currentProgress == null)
        {
            // This case handles a brand new user. Find the very first lesson.
            var firstLesson = await _progressRepository.GetFirstLessonEver(); // You'd need to implement this
            if(firstLesson == null) throw new Exception("No lessons found in the system.");

            await _progressRepository.CreateInitialProgressAsync(userId, firstLesson.LessonId);
            return await _lessonRepository.GetLessonByIdAsync(firstLesson.LessonId); // Fetch and return the DTO
        }

        return await _lessonRepository.GetLessonByIdAsync(currentProgress.CurrentLessonId); // Fetch and return DTO
    }
    
    public async Task<bool> CompleteActivityAsync(int userId, int activityId, int? score)
    {
        // 1. Mark the activity as complete
        await _progressRepository.MarkActivityAsCompleteAsync(userId, activityId, score);

        // 2. Check if the lesson is now fully complete
        var activity = await _progressRepository.GetActivityByIdAsync(activityId); // Implement this
        int totalActivities = await _progressRepository.GetTotalActivitiesForLessonAsync(activity.LessonId);
        int completedActivities = await _progressRepository.GetCompletedActivitiesForLessonAsync(userId, activity.LessonId);

        if (completedActivities >= totalActivities)
        {
            // 3. If yes, unlock the next lesson
            var nextLesson = await _progressRepository.GetNextLessonAsync(activity.LessonId);
            if (nextLesson != null)
            {
                await _progressRepository.UpdateCurrentLessonAsync(userId, nextLesson.LessonId);
                return true; // Return true to indicate a new lesson was unlocked
            }
        }
        return false; // No new lesson unlocked
    }
}
```

**3. Update `AuthService`**
When a user registers, we need to create their initial progress entry.

```csharp
// TamilApp.Infrastructure/Services/AuthService.cs
// ...
private readonly IProgressRepository _progressRepository;

public AuthService(IAuthRepository authRepository, IConfiguration configuration, IProgressRepository progressRepository)
{
    //...
    _progressRepository = progressRepository;
}

public async Task<AuthResponseDto> RegisterAsync(RegisterDto registerDto)
{
    // ... (existing registration logic)
    
    var createdUser = await _authRepository.RegisterUserAsync(user);

    // **NEW LOGIC**: Create initial progress for the new user
    // Find the very first lesson (Level 1, Lesson 1)
    var firstLesson = await _progressRepository.GetFirstLessonEver();
    if(firstLesson != null)
    {
        await _progressRepository.CreateInitialProgressAsync(createdUser.UserId, firstLesson.LessonId);
    }
    
    return new AuthResponseDto { IsSuccess = true, Message = "User registered successfully." };
}
```

---

### **Step 3: Update the API Project (`TamilApp.Api`)**

Create a new controller to handle progress and current lesson fetching.

```csharp
// TamilApp.Api/Controllers/ProgressController.cs
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;
using System.Security.Claims;
using TamilApp.Core.Dtos; // For ActivityCompletionDto
using TamilApp.Infrastructure.Services;

[ApiController]
[Route("api/[controller]")]
[Authorize] // IMPORTANT: All endpoints here require a logged-in user
public class ProgressController : ControllerBase
{
    private readonly IProgressService _progressService;
    public ProgressController(IProgressService progressService)
    {
        _progressService = progressService;
    }

    private int GetUserId() => int.Parse(User.FindFirstValue(ClaimTypes.NameIdentifier)!);

    [HttpGet("current-lesson")]
    public async Task<IActionResult> GetCurrentLesson()
    {
        var userId = GetUserId();
        var lesson = await _progressService.GetCurrentLessonForUserAsync(userId);
        if (lesson == null)
        {
            return NotFound("Could not determine current lesson for the user.");
        }
        // Here you would fetch all activities for this lesson and return them
        // var activities = await _activityService.GetActivitiesByLessonIdAsync(lesson.LessonId);
        // return Ok(new { lesson, activities });
        return Ok(lesson); 
    }

    [HttpPost("complete-activity")]
    public async Task<IActionResult> CompleteActivity([FromBody] ActivityCompletionDto dto)
    {
        var userId = GetUserId();
        var newLessonUnlocked = await _progressService.CompleteActivityAsync(userId, dto.ActivityId, dto.Score);
        
        return Ok(new { NewLessonUnlocked = newLessonUnlocked });
    }
}

// You'd need to create this DTO:
// TamilApp.Core/Dtos/ActivityCompletionDto.cs
public class ActivityCompletionDto
{
    public int ActivityId { get; set; }
    public int? Score { get; set; }
}
```

**Final `Program.cs` registrations:**
```csharp
// TamilApp.Api/Program.cs
// ...
builder.Services.AddScoped<IProgressRepository, ProgressRepository>();
builder.Services.AddScoped<IProgressService, ProgressService>();
```

You now have a complete, logical flow for user progress. The API enforces the sequential learning path by only ever exposing the user's *current* lesson, and automatically "unlocks" the next one upon full completion of the previous one. This is a robust and scalable implementation of the core feature.

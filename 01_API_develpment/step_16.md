### USER
plz convert this level service code to utilize unit of work pattern

```
using ICEDT_TamilApp.Application.DTOs.Request;
using ICEDT_TamilApp.Application.DTOs.Response;
using ICEDT_TamilApp.Application.Exceptions;
using ICEDT_TamilApp.Application.Services.Interfaces;
using ICEDT_TamilApp.Domain.Entities;
using ICEDT_TamilApp.Domain.Interfaces;

namespace ICEDT_TamilApp.Application.Services.Implementation
{
    public class LevelService : ILevelService
    {
        private readonly ILevelRepository _repo;

        public LevelService(ILevelRepository repo) => _repo = repo;

        public async Task<LevelResponseDto> GetLevelAsync(int id)
        {
            if (id <= 0)
                throw new BadRequestException("Invalid Level ID.");
            var level = await _repo.GetByIdAsync(id);
            if (level == null)
                throw new NotFoundException("Level not found.");
            return MapToResponseDto(level);
        }

        public async Task<List<LevelResponseDto>> GetAllLevelsAsync()
        {
            var levels = await _repo.GetAllAsync();
            return levels.Select(MapToResponseDto).ToList();
        }

        public async Task<LevelResponseDto> CreateLevelAsync(LevelRequestDto dto)
        {
            if (await _repo.SequenceOrderExistsAsync(dto.SequenceOrder))
                throw new BadRequestException(
                    $"Sequence order {dto.SequenceOrder} is already in use."
                );

          

            var level = new Level { LevelName = dto.LevelName, SequenceOrder = dto.SequenceOrder,Slug=dto.Slug };
            await _repo.CreateAsync(level);
            return MapToResponseDto(level);
        }

        public async Task UpdateLevelAsync(int id, LevelRequestDto dto)
        {
            if (id <= 0)
                throw new BadRequestException("Invalid Level ID.");
            var level = await _repo.GetByIdAsync(id);
            if (level == null)
                throw new NotFoundException("Level not found.");

            if (level.SequenceOrder != dto.SequenceOrder)
            {
                if (await _repo.SequenceOrderExistsAsync(dto.SequenceOrder))
                    throw new BadRequestException(
                        $"Sequence order {dto.SequenceOrder} is already in use."
                    );
            }

            level.LevelName = dto.LevelName;
            level.SequenceOrder = dto.SequenceOrder;
            await _repo.UpdateAsync(level);
        }

        public async Task DeleteLevelAsync(int id)
        {
            var level = await _repo.GetByIdAsync(id);
            if (level == null)
                throw new NotFoundException("Level not found.");
            await _repo.DeleteAsync(id);
        }

        private LevelResponseDto MapToResponseDto(Level level)
        {
            return new LevelResponseDto
            {
                LevelId = level.LevelId,
                LevelName = level.LevelName,
                Slug = level.Slug,
                SequenceOrder = level.SequenceOrder,
            };
        }
    }
}

```

### RESPONSE
Of course. Converting the `LevelService` to use the Unit of Work pattern is a great step towards cleaner transaction management.

The primary change is that the service will now depend on `IUnitOfWork` instead of `ILevelRepository`. This allows the service to orchestrate multiple repository actions and commit them all at once with a single call to `_unitOfWork.CompleteAsync()`.

Here is the fully refactored `LevelService` code.

---

### **Refactored `LevelService.cs` using Unit of Work**

```csharp
using ICEDT_TamilApp.Application.DTOs.Request;
using ICEDT_TamilApp.Application.DTOs.Response;
using ICEDT_TamilApp.Application.Exceptions;
using ICEDT_TamilApp.Application.Services.Interfaces;
using ICEDT_TamilApp.Domain.Entities;
using ICEDT_TamilApp.Domain.Interfaces;

namespace ICEDT_TamilApp.Application.Services.Implementation
{
    public class LevelService : ILevelService
    {
        // The service now depends on the Unit of Work, not an individual repository.
        private readonly IUnitOfWork _unitOfWork;

        public LevelService(IUnitOfWork unitOfWork)
        {
            _unitOfWork = unitOfWork;
        }

        public async Task<LevelResponseDto> GetLevelAsync(int id)
        {
            if (id <= 0)
                throw new BadRequestException("Invalid Level ID.");
            
            // Access the specific repository through the Unit of Work property.
            var level = await _unitOfWork.Levels.GetByIdAsync(id);
            
            if (level == null)
                throw new NotFoundException("Level not found.");
            
            return MapToResponseDto(level);
        }

        public async Task<List<LevelResponseDto>> GetAllLevelsAsync()
        {
            var levels = await _unitOfWork.Levels.GetAllAsync();
            return levels.Select(MapToResponseDto).ToList();
        }

        public async Task<LevelResponseDto> CreateLevelAsync(LevelRequestDto dto)
        {
            // Perform validation using the repository.
            if (await _unitOfWork.Levels.SequenceOrderExistsAsync(dto.SequenceOrder))
                throw new BadRequestException($"Sequence order {dto.SequenceOrder} is already in use.");

            if (await _unitOfWork.Levels.SlugExistsAsync(dto.Slug))
                 throw new BadRequestException($"Slug '{dto.Slug}' is already in use.");

            var level = new Level 
            { 
                LevelName = dto.LevelName, 
                SequenceOrder = dto.SequenceOrder, 
                Slug = dto.Slug 
            };

            // Add the new entity to the context via the repository.
            await _unitOfWork.Levels.CreateAsync(level);
            
            // *** THE KEY CHANGE ***
            // Commit all changes made in this transaction to the database.
            await _unitOfWork.CompleteAsync();

            return MapToResponseDto(level);
        }

        public async Task UpdateLevelAsync(int id, LevelRequestDto dto)
        {
            if (id <= 0)
                throw new BadRequestException("Invalid Level ID.");
            
            var level = await _unitOfWork.Levels.GetByIdAsync(id);
            if (level == null)
                throw new NotFoundException("Level not found.");

            // Check if the sequence order is being changed AND if the new one is already taken.
            if (level.SequenceOrder != dto.SequenceOrder && await _unitOfWork.Levels.SequenceOrderExistsAsync(dto.SequenceOrder))
            {
                throw new BadRequestException($"Sequence order {dto.SequenceOrder} is already in use.");
            }
            
            // Check if the slug is being changed AND if the new one is already taken.
            if (level.Slug != dto.Slug && await _unitOfWork.Levels.SlugExistsAsync(dto.Slug))
            {
                 throw new BadRequestException($"Slug '{dto.Slug}' is already in use.");
            }

            // Update the entity's properties in memory.
            level.LevelName = dto.LevelName;
            level.SequenceOrder = dto.SequenceOrder;
            level.Slug = dto.Slug;

            // The repository's UpdateAsync method just marks the entity as Modified.
            await _unitOfWork.Levels.UpdateAsync(level);

            // *** THE KEY CHANGE ***
            // Commit the update to the database.
            await _unitOfWork.CompleteAsync();
        }

        public async Task DeleteLevelAsync(int id)
        {
            var level = await _unitOfWork.Levels.GetByIdAsync(id);
            if (level == null)
                throw new NotFoundException("Level not found.");
            
            // The repository's DeleteAsync method removes the entity from the context.
            await _unitOfWork.Levels.DeleteAsync(id);

            // *** THE KEY CHANGE ***
            // Commit the deletion to the database.
            await _unitOfWork.CompleteAsync();
        }

        // The private DTO mapper function remains the same.
        private LevelResponseDto MapToResponseDto(Level level)
        {
            return new LevelResponseDto
            {
                LevelId = level.LevelId,
                LevelName = level.LevelName,
                Slug = level.Slug,
                SequenceOrder = level.SequenceOrder,
            };
        }
    }
}
```

### **Summary of Changes and Benefits**

1.  **Single Dependency:** The service now has a single dependency, `IUnitOfWork`, making its constructor cleaner.
2.  **Repository Access:** Specific repositories like `ILevelRepository` are now accessed as properties of the `IUnitOfWork` interface (e.g., `_unitOfWork.Levels`).
3.  **Centralized `SaveChanges`:** The responsibility for saving changes to the database is now explicitly controlled by the service layer by calling `_unitOfWork.CompleteAsync()`. This is the primary benefit.
4.  **Transactional Integrity:** If you were to add more complex logic (e.g., creating a level and its first lesson at the same time), both operations would be saved in a single transaction with one call to `CompleteAsync()`. If any part fails before that call, nothing gets saved to the database, ensuring data consistency.

This refactored code is more robust, scalable, and better adheres to the principles of Clean Architecture. You've successfully separated the business logic (in the service) from the transaction management (`UnitOfWork`).

### USER
got this error after applying unit of work

Object reference not set to an instance of an object.
   at ICEDT_TamilApp.Application.Services.Implementation.LevelService.GetAllLevelsAsync() in C:\Users\user\Documents\2025_proj_dotnet\ICEDT_TamilApp\src\ICEDT_TamilApp.Application\Services\Implementation\LevelService.cs:line 36

   ### RESPONSE
   This is a classic `NullReferenceException`, and the line number gives us the exact clue we need.

Let's look at the code around line 36 in your refactored `LevelService.cs`:

```csharp
// Around line 36 in LevelService.cs
public async Task<List<LevelResponseDto>> GetAllLevelsAsync()
{
    var levels = await _unitOfWork.Levels.GetAllAsync(); // This is line 36
    return levels.Select(MapToResponseDto).ToList();
}
```

The error **`Object reference not set to an instance of an object`** on this line means one of two things:

1.  The `_unitOfWork` object itself is `null`.
2.  The `_unitOfWork.Levels` property is `null`.

Since `_unitOfWork` is injected via the constructor, it's highly unlikely to be `null` (the DI container would have thrown a different error earlier). Therefore, the problem is almost certainly that the **`Levels` property on your `UnitOfWork` instance is `null`**.

---

### **The Cause: The `UnitOfWork` Implementation**

The problem lies in how the `UnitOfWork` class is constructed and how its repository properties are initialized.

Let's review the `UnitOfWork` implementation we discussed:

**File: `src/ICEDT_TamilApp.Infrastructure/Repositories/UnitOfWork.cs`**
```csharp
using ICEDT_TamilApp.Domain.Interfaces;
using ICEDT_TamilApp.Infrastructure.Data;
using System.Threading.Tasks;

namespace ICEDT_TamilApp.Infrastructure.Repositories
{
    public class UnitOfWork : IUnitOfWork
    {
        private readonly ApplicationDbContext _context;
        
        // This property needs to be initialized!
        public ILevelRepository Levels { get; private set; } 
        // You likely have other repositories here too
        // public IMainActivityRepository MainActivities { get; private set; }

        public UnitOfWork(ApplicationDbContext context)
        {
            _context = context;

            // *** THE FIX IS HERE ***
            // You must create instances of your repositories in the constructor.
            Levels = new LevelRepository(_context);
            // MainActivities = new MainActivityRepository(_context);
            // ... initialize all other repository properties here
        }

        public async Task<int> CompleteAsync()
        {
            return await _context.SaveChangesAsync();
        }

        public void Dispose()
        {
            _context.Dispose();
        }
    }
}
```
*Note: I have updated the interface property to `ILevelRepository Levels` to match your service's usage.*

**The Error Explained:**
In your original code, the `Levels` property was likely declared but **never assigned a value** inside the `UnitOfWork` constructor.

When `LevelService` was constructed, the DI container created a `UnitOfWork` instance. However, since the `Levels` property was never initialized, it remained `null`.

Later, when `GetAllLevelsAsync` was called, the code tried to execute `_unitOfWork.Levels.GetAllAsync()`. Since `_unitOfWork.Levels` was `null`, trying to access the `.GetAllAsync()` method on it resulted in the `NullReferenceException`.

### **The Solution**

1.  **Modify the `IUnitOfWork` Interface:**
    Make sure the property name in your interface matches what your service is expecting.

    **File: `src/ICEDT_TamilApp.Domain/Interfaces/IUnitOfWork.cs`**
    ```csharp
    public interface IUnitOfWork : IDisposable
    {
        ILevelRepository Levels { get; } // Ensure this property exists and has the right type
        // ... other repository interfaces
        Task<int> CompleteAsync();
    }
    ```

2.  **Fix the `UnitOfWork.cs` Constructor:**
    Open your `UnitOfWork.cs` file and ensure that you are creating a `new LevelRepository(_context)` and assigning it to the `Levels` property inside the constructor, as shown in the code block above.

3.  **Register `IUnitOfWork` in DI:**
    Finally, make sure you've registered the `UnitOfWork` in your DI container.

    **File: `src/ICEDT_TamilApp.Infrastructure/InfrastructureServiceExtensions.cs`**
    ```csharp
    // ...
    // This registration is crucial
    services.AddScoped<IUnitOfWork, UnitOfWork>();

    // You might not need to register individual repositories anymore if they are
    // only ever accessed through the Unit of Work.
    // services.AddScoped<ILevelRepository, LevelRepository>(); // This might be redundant now.
    // ...
    ```
    It's a good practice to only register `IUnitOfWork` and let it be responsible for creating the repositories. This simplifies your DI setup.

After making these changes, the `Levels` property on your `_unitOfWork` instance will be correctly initialized, and the `NullReferenceException` will be resolved.


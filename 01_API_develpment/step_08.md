### USER
great i had made changes as you mentioned , and i had pushed the changes to the repository as well , but i didnt flow the proper CLEAN architecture , so plz guide me to do this refactor , mean time i need API as well as Razor pages for Admin panel , i need to run both on same URL in aws in ec2 as we progress now.

i have doubts in CLEAN architecture folder sturucture?
How to link one project with other ?
plz guide me with VS code ?
mean time for JWT packages how can i add those?
how to do migrations?

### RESPONSE
This is a superb set of questions. Moving to a proper Clean Architecture setup is the single best thing you can do for the long-term health and maintainability of your project. It might seem like a bit of work now, but it will pay off immensely.

Let's address all your questions with a step-by-step guide tailored for VS Code.

---

### **1. Understanding Clean Architecture: The "Why" and Folder Structure**

Clean Architecture is all about **separation of concerns** and **direction of dependencies**.

**The Core Rule:** Dependencies can only point **inwards**.
*   **API/UI (Outer Layer):** Knows about Application services.
*   **Application (Middle Layer):** Knows about Domain models.
*   **Domain (Center):** Knows about **nothing** else. It's the pure core of your app.

**Your Refactored Project Structure in VS Code:**

You'll have a main folder (e.g., `TamilApp`) with a solution file and sub-folders for each project.

```
TamilApp/
├── TamilApp.sln
|
├── src/
│   ├── TamilApp.Domain/
│   │   ├── Entities/         (User.cs, Level.cs, Lesson.cs...)
│   │   └── Interfaces/       (IRepositories, e.g., ILessonRepository.cs)
│   │
│   ├── TamilApp.Application/
│   │   ├── Dtos/             (LessonDto.cs, RegisterDto.cs...)
│   │   └── Services/         (IAuthService.cs, AuthService.cs...)
│   │
│   ├── TamilApp.Infrastructure/
│   │   ├── Data/             (AppDbContext.cs, DbInitializer.cs, Migrations/)
│   │   └── Repositories/     (LessonRepository.cs)
│   │
│   └── TamilApp.Web/          (This is your main ASP.NET Core Project)
│       ├── Controllers/      (LessonsController.cs)
│       ├── Pages/Admin/      (Your Razor Pages for the Admin Panel)
│       ├── wwwroot/js/admin/ (Your AJAX scripts)
│       └── Program.cs
```

**Why this structure is better than your current one:**
*   **Domain:** Your `Core` project was a mix of things. Now, `Domain` is just your business objects (`Entities`) and the contracts they depend on (`Interfaces`). It has zero external dependencies (no EF Core, no ASP.NET).
*   **Application:** This layer orchestrates the domain. It defines the data shapes for the outside world (`Dtos`) and contains the core business logic (`Services`). It depends on `Domain`.
*   **Infrastructure:** This is where the outside world is implemented (databases, file systems, etc.). `Repositories` here *implement* the interfaces from the `Domain`. It depends on `Application` and `Domain`.
*   **Web:** This is the entry point. It can be an API, a Razor Pages app, or both. It depends on `Infrastructure`, `Application`, and `Domain`.

---

### **2. Step-by-Step Refactoring Guide (Using VS Code Terminal)**

Let's do this from scratch.

#### **Step 2.1: Set up the Solution and Projects**

1.  Open a new terminal in VS Code (`Ctrl` + `~`).
2.  Navigate to your desired workspace folder.
3.  Run these commands one by one:

    ```bash
    # Create a solution file to group all projects
    dotnet new sln -n TamilApp

    # Create the project folders
    mkdir src
    cd src

    # Create the projects inside the 'src' folder
    dotnet new classlib -n TamilApp.Domain
    dotnet new classlib -n TamilApp.Application
    dotnet new classlib -n TamilApp.Infrastructure
    dotnet new webapp -n TamilApp.Web --razor-pages # Creates a project with API and Razor Pages support

    # Go back to the root and add projects to the solution
    cd ..
    dotnet sln add src/TamilApp.Domain/TamilApp.Domain.csproj
    dotnet sln add src/TamilApp.Application/TamilApp.Application.csproj
    dotnet sln add src/TamilApp.Infrastructure/TamilApp.Infrastructure.csproj
    dotnet sln add src/TamilApp.Web/TamilApp.Web.csproj
    ```
    You now have the correct folder and project structure. Open the root `TamilApp` folder in VS Code.

#### **Step 2.2: Link the Projects (Project References)**

This is how you make one project "know about" another.

1.  In the VS Code terminal, navigate to the project folder that needs the reference.
2.  Run the `dotnet add reference` command.

    ```bash
    # Application needs to know about Domain
    cd src/TamilApp.Application
    dotnet add reference ../TamilApp.Domain/TamilApp.Domain.csproj

    # Infrastructure needs Application and Domain
    cd ../TamilApp.Infrastructure
    dotnet add reference ../TamilApp.Domain/TamilApp.Domain.csproj
    dotnet add reference ../TamilApp.Application/TamilApp.Application.csproj

    # Web needs everything
    cd ../TamilApp.Web
    dotnet add reference ../TamilApp.Domain/TamilApp.Domain.csproj
    dotnet add reference ../TamilApp.Application/TamilApp.Application.csproj
    dotnet add reference ../TamilApp.Infrastructure/TamilApp.Infrastructure.csproj

    cd ../.. # Go back to root
    ```

#### **Step 2.3: Add NuGet Packages**

Add the necessary packages to the correct projects.

```bash
# EF Core for Infrastructure
dotnet add src/TamilApp.Infrastructure package Microsoft.EntityFrameworkCore.Sqlite
dotnet add src/TamilApp.Infrastructure package Microsoft.EntityFrameworkCore.Design

# JWT Packages and EF Core for Web
dotnet add src/TamilApp.Web package Microsoft.AspNetCore.Authentication.JwtBearer
dotnet add src/TamilApp.Web package Microsoft.EntityFrameworkCore.Tools

# BCrypt for Application (where password hashing logic lives)
dotnet add src/TamilApp.Application package BCrypt.Net-Next
```

#### **Step 2.4: Move Your Files**

Now, drag and drop your existing files from your old project into the new structure:
*   `User.cs`, `Level.cs`, etc. -> `src/TamilApp.Domain/Entities/`
*   All Repository Interfaces (`ILessonRepository.cs`) -> `src/TamilApp.Domain/Interfaces/`
*   All DTOs (`LessonDto.cs`) -> `src/TamilApp.Application/Dtos/`
*   All Service Interfaces and Implementations (`IAuthService.cs`, `AuthService.cs`) -> `src/TamilApp.Application/Services/`
*   `AppDbContext.cs` and `DbInitializer.cs` -> `src/TamilApp.Infrastructure/Data/`
*   All Repository Implementations (`LessonRepository.cs`) -> `src/TamilApp.Infrastructure/Repositories/`
*   All Controllers, `Pages` folder, `wwwroot` folder, `Program.cs`, `appsettings.json` -> `src/TamilApp.Web/`

You will need to fix `using` statements and namespaces in all files to match the new project names (e.g., `namespace TamilApp.Domain.Entities`).

---

### **3. Configuring `Program.cs` for API and Razor Pages**

Your `TamilApp.Web/Program.cs` needs to be configured to serve both. Since you created it with `dotnet new webapp`, it's already mostly set up.

```csharp
// In TamilApp.Web/Program.cs

// ... builder setup

// This is for Razor Pages
builder.Services.AddRazorPages();

// This is for API Controllers
builder.Services.AddControllers();

// ... DI registration, DbContext, JWT Auth, etc.

var app = builder.Build();

// ...
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles(); // Serves your wwwroot folder (JS, CSS)

app.UseRouting();

// Auth must come after Routing but before Authorization and Endpoints
app.UseAuthentication();
app.UseAuthorization();

// This maps your API controllers (e.g., /api/levels)
app.MapControllers();

// This maps your Razor Pages (e.g., /Admin/Levels)
app.MapRazorPages();

app.Run();
```

With this setup, a request to `/Admin/Levels` will be handled by the Razor Page engine, and a request to `/api/levels` will be routed to your API controller. **They can run on the same URL and port.**

---

### **4. How to Do Migrations**

EF Core Migrations allow you to manage your database schema as code.

1.  **Ensure EF Core Tools are installed:**
    ```bash
    dotnet tool install --global dotnet-ef
    ```

2.  **Add a Migration:** When you change your models (e.g., add a new table or column), you create a migration. **Run this command from the root folder (`TamilApp/`).**

    ```bash
    dotnet ef migrations add InitialCreate --project src/TamilApp.Infrastructure --startup-project src/TamilApp.Web
    ```
    *   `--project`: Tells EF Core where your `DbContext` is.
    *   `--startup-project`: Tells EF Core where to find the configuration (`appsettings.json`).
    *   This will create a `Migrations` folder in your `Infrastructure` project with code that represents your schema.

3.  **Apply the Migration:** To create or update your database with the changes.

    ```bash
    dotnet ef database update --startup-project src/TamilApp.Web
    ```

You now have a complete, step-by-step guide to refactoring your project into a proper Clean Architecture, running both API and Razor Pages, and managing your database schema with migrations, all from within VS Code. This structure will serve you very well as you scale the project.

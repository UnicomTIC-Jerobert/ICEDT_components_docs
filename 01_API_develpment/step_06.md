### USER :
here im going to build Admin page with Razor using Javascript AJAX functionalities without introducing the code behind file , and we have to connect our existing controllers that provide REST API , so with the help of AJAX we can create musch flexible UI rather than stick with traditional razor pages style , so according to my requirement lets first crreate Levels & Lessons based UI

requirements
1. Levels table can be inline editable
2. in lesson UI let introduce Levels in drop down , and we can add lesson through inline table

### RESPONSE :
Excellent. This is a fantastic architectural choice for an admin panel. Using Razor Pages as the static shell and then letting JavaScript and AJAX take over for all dynamic functionality creates a fast, responsive, SPA-like experience for the administrator. It perfectly leverages the strengths of both technologies.

Let's build this out, starting with the necessary API enhancements and then creating the Razor Pages and their corresponding JavaScript files.

---

### **Part 1: Backend API Enhancements**

First, we need to add the `PUT` (update) and `POST` (create) endpoints to our existing controllers so the AJAX calls have something to connect to.

#### **1. Add DTOs to `TamilApp.Core`**

Create these DTOs to handle incoming data for creating and updating.

**File: `TamilApp.Core/Dtos/CreateLessonDto.cs`**
```csharp
using System.ComponentModel.DataAnnotations;

namespace TamilApp.Core.Dtos
{
    public class CreateLessonDto
    {
        [Required]
        public int LevelId { get; set; }

        [Required]
        [StringLength(200)]
        public string LessonName { get; set; } = string.Empty;

        [StringLength(500)]
        public string? Description { get; set; }

        [Required]
        public int SequenceOrder { get; set; }
    }
}
```

**File: `TamilApp.Core/Dtos/UpdateLevelDto.cs`**
```csharp
using System.ComponentModel.DataAnnotations;

namespace TamilApp.Core.Dtos
{
    public class UpdateLevelDto
    {
        [Required]
        [StringLength(100)]
        public string LevelName { get; set; } = string.Empty;

        [Required]
        public int SequenceOrder { get; set; }
    }
}
```

#### **2. Update Services and Repositories in `TamilApp.Infrastructure`**

We need to add the methods to handle the actual database operations.

**In `ILevelService.cs` & `LevelService.cs` (and their Repositories):**
```csharp
// Add to ILevelService.cs
Task<bool> UpdateLevelAsync(int levelId, UpdateLevelDto levelDto);

// Add to LevelService.cs
public async Task<bool> UpdateLevelAsync(int levelId, UpdateLevelDto levelDto)
{
    // Implementation in LevelRepository would find the level by ID,
    // update its properties, and call SaveChangesAsync().
    // Return true on success.
    return await _levelRepository.UpdateLevelAsync(levelId, levelDto);
}
```

**In `ILessonService.cs` & `LessonService.cs` (and their Repositories):**
```csharp
// Add to ILessonService.cs
Task<LessonDto> CreateLessonAsync(CreateLessonDto lessonDto);

// Add to LessonService.cs
public async Task<LessonDto> CreateLessonAsync(CreateLessonDto lessonDto)
{
    // Implementation in LessonRepository would create a new Lesson entity,
    // add it to the context, call SaveChangesAsync(), and return the created lesson as a DTO.
    return await _lessonRepository.CreateLessonAsync(lessonDto);
}
```

#### **3. Update Controllers in `TamilApp.Api`**

Now, expose these new service methods via API endpoints.

**In `LevelsController.cs`:**
```csharp
// Add this method
[HttpPut("{id}")]
public async Task<IActionResult> UpdateLevel(int id, [FromBody] UpdateLevelDto updateLevelDto)
{
    var success = await _levelService.UpdateLevelAsync(id, updateLevelDto);
    if (!success)
    {
        return NotFound($"Level with ID {id} not found.");
    }
    return NoContent(); // Standard response for a successful PUT
}
```

**In `LessonsController.cs`:**
```csharp
// Add this method
[HttpPost]
public async Task<IActionResult> CreateLesson([FromBody] CreateLessonDto createLessonDto)
{
    if (!ModelState.IsValid)
    {
        return BadRequest(ModelState);
    }
    var newLesson = await _lessonService.CreateLessonAsync(createLessonDto);
    // Return a 201 Created response with the location of the new resource
    return CreatedAtAction(nameof(GetLessonsByLevel), new { levelId = newLesson.LevelId }, newLesson);
}
```

With the backend ready, we can now build the frontend.

---

### **Part 2: Admin UI for Levels (Inline Editable Table)**

#### **1. Create the Razor Page**

In your `TamilApp.Api` project, create a `Pages/Admin` folder. Inside it, create a new Razor Page named `Levels.cshtml`.

**File: `Pages/Admin/Levels.cshtml`**
```html
@page
@{
    ViewData["Title"] = "Manage Levels";
}

<h1>Manage Levels</h1>
<p>Click "Edit" to make a row editable. Click "Save" to commit changes.</p>

<table class="table table-bordered table-striped" id="levelsTable">
    <thead>
        <tr>
            <th>ID</th>
            <th>Level Name</th>
            <th>Sequence Order</th>
            <th>Actions</th>
        </tr>
    </thead>
    <tbody id="levelsTableBody">
        <!-- Rows will be dynamically inserted here by JavaScript -->
    </tbody>
</table>

@section Scripts {
    <script src="~/js/admin/levels.js"></script>
}
```
*Note: The `@section Scripts` part assumes you have a `@RenderSection("Scripts", required: false)` in your `_Layout.cshtml` file, which is standard.*

#### **2. Create the JavaScript File**

Create a `wwwroot/js/admin` folder. Inside it, create a new file named `levels.js`.

**File: `wwwroot/js/admin/levels.js`**
```javascript
document.addEventListener('DOMContentLoaded', function () {
    const tableBody = document.getElementById('levelsTableBody');

    // 1. Function to load levels from the API
    async function loadLevels() {
        try {
            const response = await fetch('/api/levels');
            if (!response.ok) throw new Error('Failed to fetch levels');
            const levels = await response.json();

            tableBody.innerHTML = ''; // Clear existing rows
            levels.forEach(level => {
                const row = document.createElement('tr');
                row.setAttribute('data-level-id', level.levelId);
                row.innerHTML = `
                    <td>${level.levelId}</td>
                    <td data-field="levelName">${level.levelName}</td>
                    <td data-field="sequenceOrder">${level.sequenceOrder}</td>
                    <td>
                        <button class="btn btn-sm btn-primary btn-edit">Edit</button>
                        <button class="btn btn-sm btn-success btn-save" style="display:none;">Save</button>
                        <button class="btn btn-sm btn-secondary btn-cancel" style="display:none;">Cancel</button>
                    </td>
                `;
                tableBody.appendChild(row);
            });
        } catch (error) {
            console.error('Error loading levels:', error);
            alert('Could not load levels.');
        }
    }

    // 2. Event listener for the entire table body (Event Delegation)
    tableBody.addEventListener('click', function (e) {
        const target = e.target;
        const row = target.closest('tr');
        if (!row) return;

        if (target.classList.contains('btn-edit')) {
            handleEdit(row);
        } else if (target.classList.contains('btn-save')) {
            handleSave(row);
        } else if (target.classList.contains('btn-cancel')) {
            handleCancel(row);
        }
    });

    // 3. Functions to handle button clicks
    function handleEdit(row) {
        // Store original values in case of cancel
        row.setAttribute('data-original-name', row.querySelector('[data-field="levelName"]').textContent);
        row.setAttribute('data-original-order', row.querySelector('[data-field="sequenceOrder"]').textContent);
        
        // Convert cells to input fields
        row.querySelectorAll('[data-field]').forEach(cell => {
            const value = cell.textContent;
            cell.innerHTML = `<input type="text" class="form-control" value="${value}">`;
        });
        
        // Toggle buttons
        toggleButtons(row, true);
    }

    async function handleSave(row) {
        const levelId = row.dataset.levelId;
        const levelNameInput = row.querySelector('[data-field="levelName"] input');
        const sequenceOrderInput = row.querySelector('[data-field="sequenceOrder"] input');

        const payload = {
            levelName: levelNameInput.value,
            sequenceOrder: parseInt(sequenceOrderInput.value, 10)
        };
        
        try {
            const response = await fetch(`/api/levels/${levelId}`, {
                method: 'PUT',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(payload)
            });

            if (!response.ok) throw new Error('Failed to save changes');
            
            // Update UI with new values
            levelNameInput.parentElement.innerHTML = payload.levelName;
            sequenceOrderInput.parentElement.innerHTML = payload.sequenceOrder;
            
            toggleButtons(row, false);
            alert('Level updated successfully!');
        } catch (error) {
            console.error('Error saving level:', error);
            alert('Failed to save. Please check console.');
        }
    }

    function handleCancel(row) {
        // Revert to original values
        row.querySelector('[data-field="levelName"]').innerHTML = row.dataset.originalName;
        row.querySelector('[data-field="sequenceOrder"]').innerHTML = row.dataset.originalOrder;
        
        toggleButtons(row, false);
    }

    function toggleButtons(row, isEditing) {
        row.querySelector('.btn-edit').style.display = isEditing ? 'none' : 'inline-block';
        row.querySelector('.btn-save').style.display = isEditing ? 'inline-block' : 'none';
        row.querySelector('.btn-cancel').style.display = isEditing ? 'inline-block' : 'none';
    }

    // Initial load
    loadLevels();
});
```

---

### **Part 3: Admin UI for Lessons (Dropdown & Inline Add)**

#### **1. Create the Razor Page**

**File: `Pages/Admin/Lessons.cshtml`**
```html
@page
@{
    ViewData["Title"] = "Manage Lessons";
}

<h1>Manage Lessons</h1>

<div class="form-group mb-3">
    <label for="levelSelector">Select a Level:</label>
    <select class="form-control" id="levelSelector">
        <option value="">-- Loading Levels --</option>
    </select>
</div>

<table class="table table-bordered" id="lessonsTable">
    <thead>
        <tr>
            <th>ID</th>
            <th>Lesson Name</th>
            <th>Description</th>
            <th>Order</th>
        </tr>
    </thead>
    <tbody id="lessonsTableBody">
        <!-- Lesson rows will be inserted here -->
    </tbody>
    <tfoot>
        <!-- Row for adding a new lesson -->
        <tr id="addLessonRow">
             <td></td>
             <td><input type="text" id="newLessonName" class="form-control" placeholder="New Lesson Name"></td>
             <td><input type="text" id="newLessonDesc" class="form-control" placeholder="Description"></td>
             <td><input type="number" id="newLessonOrder" class="form-control" placeholder="Order"></td>
             <td><button class="btn btn-success" id="btnAddLesson">Add New Lesson</button></td>
        </tr>
    </tfoot>
</table>

@section Scripts {
    <script src="~/js/admin/lessons.js"></script>
}
```

#### **2. Create the JavaScript File**

**File: `wwwroot/js/admin/lessons.js`**
```javascript
document.addEventListener('DOMContentLoaded', function() {
    const levelSelector = document.getElementById('levelSelector');
    const lessonsTableBody = document.getElementById('lessonsTableBody');
    const btnAddLesson = document.getElementById('btnAddLesson');

    // 1. Load levels into the dropdown
    async function loadLevelsDropdown() {
        try {
            const response = await fetch('/api/levels');
            const levels = await response.json();
            levelSelector.innerHTML = '<option value="">-- Select a Level --</option>';
            levels.forEach(level => {
                const option = document.createElement('option');
                option.value = level.levelId;
                option.textContent = `${level.sequenceOrder}. ${level.levelName}`;
                levelSelector.appendChild(option);
            });
        } catch (error) {
            console.error('Error loading levels dropdown:', error);
        }
    }

    // 2. Load lessons for the selected level
    async function loadLessonsForLevel(levelId) {
        if (!levelId) {
            lessonsTableBody.innerHTML = '<tr><td colspan="4">Please select a level to view lessons.</td></tr>';
            return;
        }

        try {
            const response = await fetch(`/api/lessons/level/${levelId}`);
            if (response.status === 404) {
                 lessonsTableBody.innerHTML = '<tr><td colspan="4">No lessons found for this level.</td></tr>';
                 return;
            }
            const lessons = await response.json();

            lessonsTableBody.innerHTML = '';
            lessons.forEach(lesson => {
                const row = document.createElement('tr');
                row.innerHTML = `
                    <td>${lesson.lessonId}</td>
                    <td>${lesson.lessonName}</td>
                    <td>${lesson.description || ''}</td>
                    <td>${lesson.sequenceOrder}</td>
                `;
                lessonsTableBody.appendChild(row);
            });
        } catch (error) {
            console.error('Error loading lessons:', error);
        }
    }

    // 3. Add a new lesson
    async function addNewLesson() {
        const levelId = levelSelector.value;
        const newName = document.getElementById('newLessonName').value;
        const newDesc = document.getElementById('newLessonDesc').value;
        const newOrder = document.getElementById('newLessonOrder').value;

        if (!levelId || !newName || !newOrder) {
            alert('Please select a level and fill in at least the Name and Order fields.');
            return;
        }

        const payload = {
            levelId: parseInt(levelId, 10),
            lessonName: newName,
            description: newDesc,
            sequenceOrder: parseInt(newOrder, 10)
        };

        try {
            const response = await fetch('/api/lessons', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(payload)
            });

            if (!response.ok) throw new Error('Failed to add lesson');

            // Clear input fields and reload lessons for the current level
            document.getElementById('newLessonName').value = '';
            document.getElementById('newLessonDesc').value = '';
            document.getElementById('newLessonOrder').value = '';
            
            await loadLessonsForLevel(levelId);
            alert('Lesson added successfully!');
        } catch (error) {
            console.error('Error adding lesson:', error);
            alert('Failed to add lesson. Check console.');
        }
    }

    // 4. Event Listeners
    levelSelector.addEventListener('change', () => loadLessonsForLevel(levelSelector.value));
    btnAddLesson.addEventListener('click', addNewLesson);

    // Initial Load
    loadLevelsDropdown();
    loadLessonsForLevel(null);
});
```
This setup fully meets your requirements. The Razor Pages are just simple HTML containers, and all the dynamic data loading, editing, and creating is handled client-side with JavaScript, providing a much more fluid and modern user experience for the admin.

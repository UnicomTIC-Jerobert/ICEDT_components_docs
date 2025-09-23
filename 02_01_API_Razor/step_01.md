### USER
ok , lets get started with levels page , i think you had remembered about the menu and the page structure i had given as illustration , 
so give styles for that also , i need the level CRUD should be handled in inline fashion

### RESPONSE
Excellent, let's get started. I remember the plan perfectly: a clean, modern admin panel using Razor Pages for the structure and JavaScript/AJAX for all dynamic, inline CRUD operations.

We will build the complete "Manage Levels" page, including the HTML structure, the CSS for a clean look, and the JavaScript for the inline editing functionality.

---

### **Step 1: The Razor Page (`Levels.cshtml`)**

This file provides the basic HTML structure for the page, including the table where our data will be displayed.

**File: `src/ICEDT_TamilApp.Web/Pages/Admin/Levels.cshtml`**
```html
@page "/Admin/Levels"
@model ICEDT_TamilApp.Web.Pages.Admin.LevelsModel
@{
    ViewData["Title"] = "Manage Levels";
    // Assuming you have a layout page, otherwise, add full HTML structure.
}

@* Add a reference to our custom stylesheet *@
<link rel="stylesheet" href="~/css/admin.css" />

<div class="admin-container">
    <h1 class="admin-header">Manage Levels</h1>
    <p class="admin-description">Click "Edit" to modify a row inline. Click "Save" to commit your changes or "Cancel" to revert.</p>

    <div class="table-responsive">
        <table class="table table-bordered table-striped" id="levelsTable">
            <thead>
                <tr>
                    <th>ID</th>
                    <th>Level Name</th>
                    <th>Sequence Order</th>
                    <th class="actions-column">Actions</th>
                </tr>
            </thead>
            <tbody id="levelsTableBody">
                <!-- A loading state for better UX -->
                <tr>
                    <td colspan="4" class="text-center">
                        <div class="spinner-border" role="status">
                            <span class="visually-hidden">Loading...</span>
                        </div>
                    </td>
                </tr>
            </tbody>
        </table>
    </div>
</div>

@* This section will inject our script at the bottom of the page *@
@section Scripts {
    <script src="~/js/admin/levels.js"></script>
}
```
**File: `src/ICEDT_TamilApp.Web/Pages/Admin/Levels.cshtml.cs`** (The code-behind can be minimal)
```csharp
using Microsoft.AspNetCore.Mvc.RazorPages;

namespace ICEDT_TamilApp.Web.Pages.Admin
{
    // You can add [Authorize] attribute here later to protect the page
    public class LevelsModel : PageModel
    {
        public void OnGet()
        {
        }
    }
}
```

---

### **Step 2: The Stylesheet (`admin.css`)**

This provides a clean and modern look for our admin tables. This file can be shared across all your admin pages.

**File: `src/ICEDT_TamilApp.Web/wwwroot/css/admin.css`**
```css
/* General Admin Page Styles */
.admin-container {
    padding: 20px;
    background-color: #f8f9fa;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.admin-header {
    color: #343a40;
    margin-bottom: 10px;
    border-bottom: 2px solid #dee2e6;
    padding-bottom: 10px;
}

.admin-description {
    color: #6c757d;
    font-size: 0.9em;
    margin-bottom: 20px;
}

/* Table Styles */
.table {
    margin-top: 20px;
}

.table thead th {
    background-color: #e9ecef;
    color: #495057;
    text-align: center;
    vertical-align: middle;
}

.table tbody td {
    vertical-align: middle;
}

.actions-column {
    width: 200px; /* Fixed width for the actions column */
    text-align: center;
}

.table .btn {
    margin: 0 2px;
}

/* Styles for inline editing input fields */
.table input[type="text"], .table input[type="number"] {
    width: 100%;
    box-sizing: border-box;
}

/* Helper class to show validation errors */
.input-error {
    border: 1px solid red !important;
}

```

---

### **Step 3: The JavaScript Logic (`levels.js`)**

This is the core of the inline editing functionality. It uses AJAX to communicate with your API without page reloads.

**File: `src/ICEDT_TamilApp.Web/wwwroot/js/admin/levels.js`**
```javascript
document.addEventListener('DOMContentLoaded', function () {
    const tableBody = document.getElementById('levelsTableBody');

    // --- RENDER FUNCTION ---
    // Renders the table rows from the data fetched from the API
    function renderTable(levels) {
        tableBody.innerHTML = ''; // Clear existing rows
        if (!levels || levels.length === 0) {
            tableBody.innerHTML = '<tr><td colspan="4" class="text-center">No levels found.</td></tr>';
            return;
        }

        levels.forEach(level => {
            const row = document.createElement('tr');
            row.setAttribute('data-level-id', level.levelId);
            row.innerHTML = `
                <td>${level.levelId}</td>
                <td data-field="levelName">${level.levelName}</td>
                <td data-field="sequenceOrder">${level.sequenceOrder}</td>
                <td class="actions-column">
                    <button class="btn btn-sm btn-primary btn-edit">Edit</button>
                    <button class="btn btn-sm btn-success btn-save" style="display:none;">Save</button>
                    <button class="btn btn-sm btn-secondary btn-cancel" style="display:none;">Cancel</button>
                </td>
            `;
            tableBody.appendChild(row);
        });
    }

    // --- API CALLS ---
    // Fetches all levels from the API
    async function fetchLevels() {
        try {
            const response = await fetch('/api/levels');
            if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`);
            const levels = await response.json();
            renderTable(levels);
        } catch (error) {
            console.error('Error fetching levels:', error);
            tableBody.innerHTML = '<tr><td colspan="4" class="text-center text-danger">Could not load levels.</td></tr>';
        }
    }

    // Updates a level via a PUT request
    async function updateLevel(levelId, payload) {
        try {
            const response = await fetch(`/api/levels/${levelId}`, {
                method: 'PUT',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(payload)
            });

            if (!response.ok) {
                const errorData = await response.json();
                throw new Error(errorData.title || 'Failed to save changes');
            }
            return true;
        } catch (error) {
            console.error('Error updating level:', error);
            alert(`Error: ${error.message}`);
            return false;
        }
    }
    
    // --- EVENT HANDLING ---
    // Uses event delegation for edit, save, and cancel buttons
    tableBody.addEventListener('click', async function (e) {
        const target = e.target;
        const row = target.closest('tr');
        if (!row) return;

        if (target.classList.contains('btn-edit')) {
            enterEditMode(row);
        } else if (target.classList.contains('btn-save')) {
            await exitEditMode(row, true); // true = save changes
        } else if (target.classList.contains('btn-cancel')) {
            exitEditMode(row, false); // false = discard changes
        }
    });

    // --- EDIT MODE LOGIC ---
    // Converts a row's cells into editable input fields
    function enterEditMode(row) {
        // Store original values on the row element for cancellation
        const nameCell = row.querySelector('[data-field="levelName"]');
        const orderCell = row.querySelector('[data-field="sequenceOrder"]');
        row.setAttribute('data-original-name', nameCell.textContent);
        row.setAttribute('data-original-order', orderCell.textContent);
        
        nameCell.innerHTML = `<input type="text" class="form-control" value="${nameCell.textContent}">`;
        orderCell.innerHTML = `<input type="number" class="form-control" value="${orderCell.textContent}">`;
        
        toggleButtons(row, true);
    }

    // Handles saving or canceling the edit
    async function exitEditMode(row, shouldSave) {
        const levelId = row.dataset.levelId;
        const nameCell = row.querySelector('[data-field="levelName"]');
        const orderCell = row.querySelector('[data-field="sequenceOrder"]');

        if (shouldSave) {
            const nameInput = nameCell.querySelector('input');
            const orderInput = orderCell.querySelector('input');
            
            const payload = {
                levelName: nameInput.value.trim(),
                sequenceOrder: parseInt(orderInput.value, 10)
            };

            // Basic validation
            if (!payload.levelName || isNaN(payload.sequenceOrder)) {
                alert('Level Name cannot be empty and Sequence Order must be a number.');
                return;
            }

            const success = await updateLevel(levelId, payload);
            if (success) {
                nameCell.textContent = payload.levelName;
                orderCell.textContent = payload.sequenceOrder;
                toggleButtons(row, false);
            }
        } else {
            // Revert to original values on cancel
            nameCell.textContent = row.dataset.originalName;
            orderCell.textContent = row.dataset.originalOrder;
            toggleButtons(row, false);
        }
    }

    // Toggles the visibility of Edit/Save/Cancel buttons
    function toggleButtons(row, isEditing) {
        row.querySelector('.btn-edit').style.display = isEditing ? 'none' : 'inline-block';
        row.querySelector('.btn-save').style.display = isEditing ? 'inline-block' : 'none';
        row.querySelector('.btn-cancel').style.display = isEditing ? 'inline-block' : 'none';
    }

    // --- INITIALIZATION ---
    // Fetch and render the levels when the page loads
    fetchLevels();
});
```

### **How to Run It**

1.  Place the files in their correct locations.
2.  Make sure your `_Layout.cshtml` in `Pages/Shared` includes a Bootstrap CSS link in the `<head>` and `@RenderSection("Scripts", required: false)` before the closing `</body>` tag.
3.  Run your `Web` project.
4.  Navigate to `/Admin/Levels` in your browser.

You should now have a fully functional, inline-editable table for managing your `Levels`.

---

### USER
Once again great , but im expecting ADD button to add new entries , so do the further modification to incoporate this issue

### RESPONSE
Of course. Adding an "Add New" feature is a crucial part of any CRUD page. We can integrate this seamlessly into your existing inline editing structure.

The standard and most user-friendly way to do this is to have a dedicated "Add New Level" button that appends a new, editable row to the bottom of the table.

Here are the required modifications for all three files.

---

### **Step 1: Update the Razor Page (`Levels.cshtml`)**

We will add a button above the table to trigger the "add new" action and a `<tfoot>` section to act as a template for the new entry row.

**File: `src/ICEDT_TamilApp.Web/Pages/Admin/Levels.cshtml` (Modified)**
```html
@page "/Admin/Levels"
@model ICEDT_TamilApp.Web.Pages.Admin.LevelsModel
@{
    ViewData["Title"] = "Manage Levels";
}

<link rel="stylesheet" href="~/css/admin.css" />

<div class="admin-container">
    <div class="d-flex justify-content-between align-items-center">
        <h1 class="admin-header mb-0">Manage Levels</h1>
        
        @* --- NEW: ADD BUTTON --- *@
        <button class="btn btn-success" id="addNewLevelBtn">
            <i class="fas fa-plus"></i> Add New Level
        </button>
    </div>

    <p class="admin-description">Click "Add New Level" to add an entry. Click "Edit" to modify an existing row.</p>

    <div class="table-responsive">
        <table class="table table-bordered table-striped" id="levelsTable">
            <thead>
                <tr>
                    <th>ID</th>
                    <th>Level Name</th>
                    <th>Sequence Order</th>
                    <th class="actions-column">Actions</th>
                </tr>
            </thead>
            <tbody id="levelsTableBody">
                <!-- Rows will be dynamically inserted here by JavaScript -->
            </tbody>
            
            @* --- NEW: TFOOT FOR NEW ENTRY TEMPLATE --- *@
            <tfoot id="newLevelTfoot" style="display: none;">
                <tr class="new-entry-row">
                    <td>(New)</td>
                    <td><input type="text" id="newLevelName" class="form-control" placeholder="Enter level name"></td>
                    <td><input type="number" id="newLevelOrder" class="form-control" placeholder="Enter sequence"></td>
                    <td class="actions-column">
                        <button class="btn btn-sm btn-success btn-save-new">Save</button>
                        <button class="btn btn-sm btn-secondary btn-cancel-new">Cancel</button>
                    </td>
                </tr>
            </tfoot>
        </table>
    </div>
</div>

@* To use the plus icon, you might need a Font Awesome CDN link in your _Layout.cshtml *@
@* <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css" /> *@

@section Scripts {
    <script src="~/js/admin/levels.js"></script>
}
```

---

### **Step 2: Update the Stylesheet (`admin.css`)**

Let's add a subtle style for the new entry row to make it stand out.

**File: `src/ICEDT_TamilApp.Web/wwwroot/css/admin.css` (Modified)**
```css
/* ... (all existing styles) ... */

/* Style for the new entry row in the tfoot */
.new-entry-row {
    background-color: #fcf8e3; /* A soft yellow to indicate it's a new, unsaved entry */
}
```

---

### **Step 3: Update the API and Services**

We need a `POST` endpoint to handle the creation of a new level.

**1. Create a DTO in `Application/DTOs`**
```csharp
// src/ICEDT_TamilApp.Application/DTOs/Request/CreateLevelRequestDto.cs
public class CreateLevelRequestDto
{
    [Required]
    [StringLength(100)]
    public string LevelName { get; set; }

    [Required]
    public int SequenceOrder { get; set; }
}
```

**2. Update `ILevelService` and `LevelService`**
```csharp
// Add to ILevelService.cs
Task<Level> CreateLevelAsync(CreateLevelRequestDto createLevelDto);

// Add to LevelService.cs (and its corresponding repository)
public async Task<Level> CreateLevelAsync(CreateLevelRequestDto createLevelDto)
{
    // ... logic in the repository to create and save a new Level ...
    // ... return the newly created Level object with its ID ...
}
```

**3. Update `LevelsController.cs`**
```csharp
// In src/ICEDT_TamilApp.Web/Controllers/LevelsController.cs
[HttpPost]
public async Task<IActionResult> CreateLevel([FromBody] CreateLevelRequestDto createLevelDto)
{
    if (!ModelState.IsValid)
    {
        return BadRequest(ModelState);
    }

    var newLevel = await _levelService.CreateLevelAsync(createLevelDto);

    // Return a 201 Created response with the new object
    return CreatedAtAction(nameof(GetLevelById), new { id = newLevel.LevelId }, newLevel);
}
```

---

### **Step 4: Update the JavaScript Logic (`levels.js`)**

Now we'll add the logic to handle showing the new entry row and saving it.

**File: `src/ICEDT_TamilApp.Web/wwwroot/js/admin/levels.js` (Modified & Final)**
```javascript
document.addEventListener('DOMContentLoaded', function () {
    const tableBody = document.getElementById('levelsTableBody');
    const addNewLevelBtn = document.getElementById('addNewLevelBtn');
    const newLevelTfoot = document.getElementById('newLevelTfoot');

    // --- RENDER FUNCTION (Unchanged) ---
    function renderTable(levels) {
        // ... (same as before) ...
    }

    // --- API CALLS ---
    async function fetchLevels() {
        // ... (same as before) ...
    }
    async function updateLevel(levelId, payload) {
        // ... (same as before) ...
    }
    
    // *** NEW: API call to create a new level ***
    async function createLevel(payload) {
        try {
            const response = await fetch('/api/levels', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(payload)
            });

            if (!response.ok) {
                const errorData = await response.json();
                throw new Error(errorData.title || 'Failed to create level');
            }
            return true;
        } catch (error) {
            console.error('Error creating level:', error);
            alert(`Error: ${error.message}`);
            return false;
        }
    }

    // --- EVENT HANDLING ---
    // *** MODIFIED: Handle new buttons ***
    document.body.addEventListener('click', async function (e) {
        const target = e.target;
        const row = target.closest('tr');

        // Handle buttons within the main table body
        if (tableBody.contains(target) && row) {
            if (target.classList.contains('btn-edit')) {
                enterEditMode(row);
            } else if (target.classList.contains('btn-save')) {
                await exitEditMode(row, true);
            } else if (target.classList.contains('btn-cancel')) {
                exitEditMode(row, false);
            }
        }
        
        // Handle global buttons
        if (target.id === 'addNewLevelBtn') {
            showAddNewRow();
        } 
        // Handle buttons within the tfoot
        else if (newLevelTfoot.contains(target) && row) {
            if (target.classList.contains('btn-save-new')) {
                await handleSaveNew();
            } else if (target.classList.contains('btn-cancel-new')) {
                hideAddNewRow();
            }
        }
    });

    // --- ADD NEW LOGIC ---
    function showAddNewRow() {
        newLevelTfoot.style.display = 'table-footer-group';
        addNewLevelBtn.style.display = 'none'; // Hide the "Add" button while the form is open
        document.getElementById('newLevelName').focus();
    }

    function hideAddNewRow() {
        document.getElementById('newLevelName').value = '';
        document.getElementById('newLevelOrder').value = '';
        newLevelTfoot.style.display = 'none';
        addNewLevelBtn.style.display = 'block';
    }

    async function handleSaveNew() {
        const newNameInput = document.getElementById('newLevelName');
        const newOrderInput = document.getElementById('newLevelOrder');

        const payload = {
            levelName: newNameInput.value.trim(),
            sequenceOrder: parseInt(newOrderInput.value, 10)
        };

        if (!payload.levelName || isNaN(payload.sequenceOrder)) {
            alert('Level Name cannot be empty and Sequence Order must be a number.');
            return;
        }

        const success = await createLevel(payload);
        if (success) {
            hideAddNewRow();
            await fetchLevels(); // Refresh the entire table to show the new entry
            alert('Level created successfully!');
        }
    }

    // --- EDIT MODE LOGIC (Unchanged) ---
    function enterEditMode(row) {
        // ... (same as before) ...
    }
    async function exitEditMode(row, shouldSave) {
        // ... (same as before) ...
    }
    function toggleButtons(row, isEditing) {
        // ... (same as before) ...
    }

    // --- INITIALIZATION ---
    fetchLevels();
});
```

### **Summary of Changes**

1.  **UI:** Added an "Add New Level" button and a hidden `<tfoot>` to act as our inline form.
2.  **API:** Added the standard `POST /api/levels` endpoint to handle the creation logic.
3.  **JavaScript:**
    *   Added a `createLevel` function to call the new `POST` endpoint.
    *   Added event listeners for the new buttons (`#addNewLevelBtn`, `.btn-save-new`, `.btn-cancel-new`).
    *   Implemented logic to show/hide the new entry row.
    *   When a new level is successfully saved, the entire table is refreshed from the API to ensure the data is consistent and the new ID is displayed correctly.

Now your "Manage Levels" page has full, inline CRUD functionality in a clean and user-friendly way.

### USER
great , another thing missing out here is delete , so shall we implement that also

### RESPONSE
You are absolutely right. A full CRUD implementation isn't complete without the "D" for Delete. Adding a delete button with a confirmation step is a crucial part of a robust admin panel.

Let's incorporate the delete functionality into our existing structure.

---

### **Step 1: Update the API and Services**

First, we need to create the `DELETE` endpoint in our `LevelsController` and the corresponding logic in the service and repository layers.

**1. Update `ILevelService` and `LevelService` (and Repository)**
*   **Add to `ILevelService.cs`:**
    ```csharp
    Task<bool> DeleteLevelAsync(int levelId);
    ```
*   **Add to `LevelService.cs`:**
    ```csharp
    public async Task<bool> DeleteLevelAsync(int levelId)
    {
        // Add logic to check if the level has dependent lessons.
        // If so, you might want to prevent deletion.
        // For now, we'll assume direct deletion is allowed.
        return await _levelRepository.DeleteLevelAsync(levelId);
    }
    ```
*   **Add to `ILevelRepository.cs` and implement in `LevelRepository.cs`:**
    The implementation will find the level by ID, remove it from the context, and save the changes.

**2. Update `LevelsController.cs`**
Add the `DELETE` endpoint.

**File: `src/ICEDT_TamilApp.Web/Controllers/LevelsController.cs` (Add this method)**
```csharp
[HttpDelete("{id}")]
public async Task<IActionResult> DeleteLevel(int id)
{
    var success = await _levelService.DeleteLevelAsync(id);
    if (!success)
    {
        // This could mean the level was already deleted or never existed.
        return NotFound($"Level with ID {id} not found.");
    }
    // Standard response for a successful DELETE is 204 No Content.
    return NoContent();
}
```

The backend is now ready to handle delete requests.

---

### **Step 2: Update the Razor Page (`Levels.cshtml`)**

We just need to add a placeholder for the delete button in our table rows.

**File: `src/ICEDT_TamilApp.Web/Pages/Admin/Levels.cshtml` (No major changes needed)**
The existing `Actions` column is already sufficient. Our JavaScript will dynamically add the delete button to this column.

---

### **Step 3: Update the JavaScript Logic (`levels.js`)**

This is where the main changes will happen. We'll add a delete button, a confirmation prompt, and the AJAX call to the `DELETE` endpoint.

**File: `src/ICEDT_TamilApp.Web/wwwroot/js/admin/levels.js` (Modified & Final)**
```javascript
document.addEventListener('DOMContentLoaded', function () {
    const tableBody = document.getElementById('levelsTableBody');
    const addNewLevelBtn = document.getElementById('addNewLevelBtn');
    const newLevelTfoot = document.getElementById('newLevelTfoot');

    // --- RENDER FUNCTION ---
    // *** MODIFIED: Add a Delete button to each row ***
    function renderTable(levels) {
        tableBody.innerHTML = ''; 
        if (!levels || levels.length === 0) {
            tableBody.innerHTML = '<tr><td colspan="4" class="text-center">No levels found.</td></tr>';
            return;
        }

        levels.forEach(level => {
            const row = document.createElement('tr');
            row.setAttribute('data-level-id', level.levelId);
            // Added btn-danger for delete
            row.innerHTML = `
                <td>${level.levelId}</td>
                <td data-field="levelName">${level.levelName}</td>
                <td data-field="sequenceOrder">${level.sequenceOrder}</td>
                <td class="actions-column">
                    <button class="btn btn-sm btn-primary btn-edit">Edit</button>
                    <button class="btn btn-sm btn-danger btn-delete">Delete</button>
                    <button class="btn btn-sm btn-success btn-save" style="display:none;">Save</button>
                    <button class="btn btn-sm btn-secondary btn-cancel" style="display:none;">Cancel</button>
                </td>
            `;
            tableBody.appendChild(row);
        });
    }

    // --- API CALLS ---
    async function fetchLevels() { /* ... (same as before) ... */ }
    async function updateLevel(levelId, payload) { /* ... (same as before) ... */ }
    async function createLevel(payload) { /* ... (same as before) ... */ }

    // *** NEW: API call to delete a level ***
    async function deleteLevel(levelId) {
        try {
            const response = await fetch(`/api/levels/${levelId}`, {
                method: 'DELETE'
            });

            if (!response.ok) {
                // Handle cases where the server returns an error (e.g., 404 Not Found)
                const errorText = response.status === 404 ? "Level not found." : "Failed to delete.";
                throw new Error(errorText);
            }
            return true;
        } catch (error) {
            console.error('Error deleting level:', error);
            alert(`Error: ${error.message}`);
            return false;
        }
    }

    // --- EVENT HANDLING ---
    // *** MODIFIED: Add a handler for the delete button ***
    document.body.addEventListener('click', async function (e) {
        const target = e.target;
        const row = target.closest('tr');

        // Handle buttons within the main table body
        if (tableBody.contains(target) && row) {
            if (target.classList.contains('btn-edit')) {
                enterEditMode(row);
            } else if (target.classList.contains('btn-save')) {
                await exitEditMode(row, true);
            } else if (target.classList.contains('btn-cancel')) {
                exitEditMode(row, false);
            } else if (target.classList.contains('btn-delete')) {
                await handleDelete(row); // New handler
            }
        }
        
        // ... (rest of the event handler is the same) ...
    });

    // --- DELETE LOGIC ---
    // *** NEW: Function to handle the delete process ***
    async function handleDelete(row) {
        const levelId = row.dataset.levelId;
        const levelName = row.querySelector('[data-field="levelName"]').textContent;

        // Use a confirmation dialog for safety
        if (!confirm(`Are you sure you want to delete the level "${levelName}"? This action cannot be undone.`)) {
            return;
        }

        const success = await deleteLevel(levelId);

        if (success) {
            row.remove(); // Remove the row from the UI immediately for a responsive feel
            alert('Level deleted successfully!');
            // Optional: You could call fetchLevels() again, but removing the row directly is faster UX.
        }
    }

    // --- ADD NEW LOGIC (Unchanged) ---
    function showAddNewRow() { /* ... */ }
    function hideAddNewRow() { /* ... */ }
    async function handleSaveNew() { /* ... */ }

    // --- EDIT MODE LOGIC (Unchanged) ---
    // *** MODIFIED: Toggle delete button visibility during edit ***
    function enterEditMode(row) {
        // ... (same as before)
        toggleButtons(row, true);
    }
    async function exitEditMode(row, shouldSave) {
        // ... (same as before)
    }
    
    // Toggles the visibility of all buttons based on edit state
    function toggleButtons(row, isEditing) {
        row.querySelector('.btn-edit').style.display = isEditing ? 'none' : 'inline-block';
        row.querySelector('.btn-delete').style.display = isEditing ? 'none' : 'inline-block'; // Hide delete during edit
        row.querySelector('.btn-save').style.display = isEditing ? 'inline-block' : 'none';
        row.querySelector('.btn-cancel').style.display = isEditing ? 'inline-block' : 'none';
    }

    // --- INITIALIZATION ---
    fetchLevels();
});
```

### **Summary of Changes**

1.  **API:** A `DELETE /api/levels/{id}` endpoint was added to handle the deletion request.
2.  **JavaScript (`renderTable`):** A red "Delete" button is now dynamically added to each row when the table is rendered.
3.  **JavaScript (`handleDelete`):** A new function was created.
    *   It retrieves the `levelId` and `levelName` from the row.
    *   It shows a `confirm()` dialog to the user to prevent accidental deletions. This is a critical UX feature.
    *   If confirmed, it calls the `deleteLevel` AJAX function.
    *   On success, it removes the row directly from the DOM for an immediate visual update.
4.  **JavaScript (`toggleButtons`):** The logic was updated to also hide the "Delete" button when a row enters edit mode. This prevents confusion, as the user should either be editing or deleting, not both at the same time.

Your "Manage Levels" admin page now has complete, inline **C**reate, **R**ead, **U**pdate, and **D**elete functionality. It's a robust and reusable pattern for the rest of your admin panel.


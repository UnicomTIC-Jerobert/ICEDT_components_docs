### USER
Great progress , now lets implement for Levels table , make sure that we need to intoduce customization , bcz levels table have link for manage lessons

### RESPONSE
You're absolutely right. Now we'll use our generic `InlineCrudTable` component for the `Levels` page and leverage its `renderCustomActions` prop to add the "Manage Lessons" link.

This will showcase the full flexibility of the reusable component we built. The core CRUD logic will be handled automatically, and we will just "inject" the special link we need.

---

### **Step 1: Define the TypeScript Type**

This file should already exist from our first React implementation.

**File: `src/types/level.ts`**
```typescript
export interface Level {
  levelId: number;
  levelName: string;
  sequenceOrder: number;
}
```

### **Step 2: Create (or Verify) the API Service**

This file should also exist. We'll ensure it exports the functions with the names our generic component expects.

**File: `src/api/levelApi.ts`**
```typescript
import { Level } from '../types/level';

const API_BASE_URL = '/api/levels';

export type LevelCreateDto = Omit<Level, 'levelId'>;

// GET all levels
export const getAll = async (): Promise<Level[]> => {
    const response = await fetch(API_BASE_URL);
    if (!response.ok) throw new Error('Failed to fetch levels');
    return response.json();
};

// POST a new level
export const create = async (newItem: LevelCreateDto): Promise<Level> => {
    const response = await fetch(API_BASE_URL, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(newItem)
    });
    if (!response.ok) throw new Error('Failed to create level');
    return response.json();
};

// PUT (update) an existing level
export const update = async (id: number | string, itemToUpdate: Partial<LevelCreateDto>): Promise<void> => {
    const response = await fetch(`${API_BASE_URL}/${id}`, {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(itemToUpdate)
    });
    if (!response.ok) throw new Error('Failed to update level');
};

// DELETE a level
export const deleteItem = async (id: number | string): Promise<void> => {
    const response = await fetch(`${API_BASE_URL}/${id}`, {
        method: 'DELETE'
    });
    if (!response.ok) throw new Error('Failed to delete level');
};
```

### **Step 3: Implement the `LevelsPage` Component**

This is where the customization happens. We'll define the columns and the special action button, then pass them as props to our `InlineCrudTable`.

**File: `src/pages/LevelsPage.tsx` (Complete & Final)**
```typescript
import React from 'react';
import { Link as RouterLink } from 'react-router-dom';
import { Button } from '@mui/material';
import InlineCrudTable from '../components/common/InlineCrudTable';
import { Level } from '../types/level';
import * as levelApi from '../api/levelApi';
import { LevelCreateDto } from '../api/levelApi';

const LevelsPage: React.FC = () => {
    
    // 1. Define the columns for the table.
    const columns = [
        { field: 'levelName' as keyof Level, headerName: 'Level Name', type: 'string' as const },
        { field: 'sequenceOrder' as keyof Level, headerName: 'Sequence Order', type: 'number' as const }
    ];

    // 2. Define the API service object.
    const apiService = {
        getAll: levelApi.getAll,
        create: levelApi.create,
        update: levelApi.update,
        delete: levelApi.deleteItem
    };

    // 3. *** THE CUSTOMIZATION PART ***
    //    Define a function that returns the custom "Manage Lessons" button.
    //    This function will be passed as a prop to the generic table.
    const renderCustomLevelActions = (level: Level) => (
        <Button 
            component={RouterLink} 
            to={`/lessons?levelId=${level.levelId}`} // Link to the lessons page with the levelId
            variant="outlined" 
            size="small"
            sx={{ mr: 1 }} // Add some margin to the right
        >
            Manage Lessons
        </Button>
    );

    return (
        // 4. Render the generic table, passing in the specific configuration and the custom action renderer.
        <InlineCrudTable<Level, LevelCreateDto>
            entityName="Level"
            apiService={apiService}
            columns={columns}
            idField="levelId" // Tell the component the unique ID is 'levelId'
            renderCustomActions={renderCustomLevelActions} // Pass the custom function here
        />
    );
};

export default LevelsPage;
```

### **How It Works**

1.  **`renderCustomLevelActions`:** We create a small function that takes a `level` object as an argument. It uses this object to construct a dynamic link (e.g., `/lessons?levelId=3`). This link is an MUI `Button` styled to look good.
2.  **Passing the Function as a Prop:** We pass this function to our `InlineCrudTable` through the `renderCustomActions` prop.
3.  **Inside `InlineCrudTable.tsx`:** The generic component has a section in its `renderRow` function that checks if this prop exists.
    ```typescript
    // Inside InlineCrudTable's renderRow function
    <TableCell>
        {isEditing ? (
            // ... save/cancel buttons
        ) : (
            <>
                {/* THIS IS WHERE THE MAGIC HAPPENS */}
                {renderCustomActions && renderCustomActions(item)}
                
                {/* Standard edit/delete buttons */}
                <IconButton onClick={() => handleEdit(item)} color="primary"><EditIcon /></IconButton>
                <IconButton onClick={() => handleDelete(id)} color="error"><DeleteIcon /></IconButton>
            </>
        )}
    </TableCell>
    ```
    If `renderCustomActions` was passed in, it calls the function with the current row's `item` data, and the button is rendered. If not (like in the `MainActivityPage`), it simply does nothing.

This is a beautiful example of the **Composition** pattern in React. Instead of making the table component more complex, we keep it generic and "inject" the specific functionality it needs from the outside.

Your `LevelsPage` is now fully implemented, and you have a clear, scalable pattern for adding custom links or buttons to any other admin page in the future.


### USER
im good to go ahead this one , we can start your idea , so before going to to build this Activity Editor , 
lets rebuild our levels , lessons , MainActivity , ActivityTypes , Activities Pages as we have done with Razor pages , so will you help me to rebuild that scenarion without loosing any thing that we have done , 
once we finished this admin panel with react , then we shall remove the coded there , for razor pages in our API app

### RESPONSE
Excellent decision. Rebuilding the admin panel in React from the ground up while the Razor Pages version is still functional is the perfect, low-risk strategy. It allows for a smooth transition.

I will guide you through this process step-by-step. We'll rebuild the `Levels` page first, as it's a great starting point and covers all the essential concepts: fetching data, displaying it in a table, and handling full inline CRUD.

Here is a complete guide to rebuilding the "Manage Levels" page in your new React + TypeScript project using MUI.

---

### **Prerequisites**

1.  **React App is Created:** You have run `npx create-react-app admin-app --template typescript`.
2.  **Proxy is Set:** You've added `"proxy": "https://localhost:7123"` (or your correct port) to your `admin-app/package.json`.
3.  **MUI is Installed:** You have run `npm install @mui/material @emotion/react @emotion/styled @mui/icons-material`.
4.  **Backend is Running:** Your ASP.NET Core application is running, so the React app can call its APIs.

---

### **Step 1: Project Structure and Setup in React**

Let's create a clean structure inside your `admin-app/src/` folder.

```
admin-app/
└── src/
    ├── api/                // To centralize API calls
    │   └── levelApi.ts
    ├── components/         // Reusable UI components
    │   └── layout/         // For sidebar, header, etc.
    ├── pages/              // Top-level pages
    │   └── LevelsPage.tsx
    ├── types/              // TypeScript interfaces
    │   └── level.ts
    ├── App.tsx             // Main app component
    └── index.tsx           // Entry point
```

### **Step 2: Define the TypeScript Type**

**File: `src/types/level.ts`**
```typescript
export interface Level {
  levelId: number;
  levelName: string;
  sequenceOrder: number;
}
```

### **Step 3: Create the API Service**

Centralizing API calls keeps your components clean.

**File: `src/api/levelApi.ts`**
```typescript
import { Level } from '../types/level';

const API_BASE_URL = '/api/levels'; // The proxy will handle the full URL

export const getLevels = async (): Promise<Level[]> => {
    const response = await fetch(API_BASE_URL);
    if (!response.ok) {
        throw new Error('Failed to fetch levels');
    }
    return response.json();
};

export const createLevel = async (level: Omit<Level, 'levelId'>): Promise<Level> => {
    const response = await fetch(API_BASE_URL, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(level)
    });
    if (!response.ok) {
        throw new Error('Failed to create level');
    }
    return response.json();
};

export const updateLevel = async (levelId: number, level: Omit<Level, 'levelId'>): Promise<void> => {
    const response = await fetch(`${API_BASE_URL}/${levelId}`, {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(level)
    });
    if (!response.ok) {
        throw new Error('Failed to update level');
    }
};

export const deleteLevel = async (levelId: number): Promise<void> => {
    const response = await fetch(`${API_BASE_URL}/${levelId}`, {
        method: 'DELETE'
    });
    if (!response.ok) {
        throw new Error('Failed to delete level');
    }
};
```

### **Step 4: Build the React `LevelsPage` Component**

This is the main component that will manage the state and render the MUI table.

**File: `src/pages/LevelsPage.tsx`**
```typescript
import React, { useState, useEffect } from 'react';
import {
    Table, TableBody, TableCell, TableContainer, TableHead, TableRow, Paper,
    Button, TextField, IconButton, Box, Typography, CircularProgress
} from '@mui/material';
import EditIcon from '@mui/icons-material/Edit';
import DeleteIcon from '@mui/icons-material/Delete';
import SaveIcon from '@mui/icons-material/Save';
import CancelIcon from '@mui/icons-material/Cancel';
import AddIcon from '@mui/icons-material/Add';

import { Level } from '../types/level';
import * as levelApi from '../api/levelApi';

const LevelsPage: React.FC = () => {
    const [levels, setLevels] = useState<Level[]>([]);
    const [isLoading, setIsLoading] = useState<boolean>(true);
    const [editRowId, setEditRowId] = useState<number | null>(null);
    const [editedRowData, setEditedRowData] = useState<Omit<Level, 'levelId'> | null>(null);
    const [isAdding, setIsAdding] = useState<boolean>(false);

    useEffect(() => {
        fetchData();
    }, []);

    const fetchData = async () => {
        setIsLoading(true);
        try {
            const data = await levelApi.getLevels();
            setLevels(data);
        } catch (error) {
            console.error(error);
            // Here you would set an error state to show in the UI
        } finally {
            setIsLoading(false);
        }
    };

    const handleEdit = (level: Level) => {
        setEditRowId(level.levelId);
        setEditedRowData({ levelName: level.levelName, sequenceOrder: level.sequenceOrder });
    };

    const handleCancel = () => {
        setEditRowId(null);
        setEditedRowData(null);
        setIsAdding(false);
    };

    const handleSave = async () => {
        if (!editRowId || !editedRowData) return;

        try {
            await levelApi.updateLevel(editRowId, editedRowData);
            setEditRowId(null);
            setEditedRowData(null);
            fetchData(); // Refresh data
        } catch (error) {
            console.error(error);
        }
    };
    
    const handleAddNew = async () => {
        if (!editedRowData || !editedRowData.levelName) {
            alert("Level Name cannot be empty.");
            return;
        }
        try {
            await levelApi.createLevel(editedRowData);
            setIsAdding(false);
            setEditedRowData(null);
            fetchData();
        } catch (error) {
            console.error(error);
        }
    };
    
    const handleDelete = async (levelId: number) => {
        if (window.confirm("Are you sure you want to delete this level?")) {
            try {
                await levelApi.deleteLevel(levelId);
                fetchData();
            } catch (error) {
                console.error(error);
            }
        }
    };

    const handleInputChange = (e: React.ChangeEvent<HTMLInputElement>) => {
        const { name, value } = e.target;
        setEditedRowData(prev => ({ ...prev!, [name]: value }));
    };

    const renderRow = (level: Level) => {
        const isEditing = editRowId === level.levelId;
        return (
            <TableRow key={level.levelId}>
                <TableCell>{level.levelId}</TableCell>
                <TableCell>
                    {isEditing ? (
                        <TextField
                            name="levelName"
                            value={editedRowData?.levelName}
                            onChange={handleInputChange}
                            size="small"
                        />
                    ) : (
                        level.levelName
                    )}
                </TableCell>
                <TableCell>
                    {isEditing ? (
                        <TextField
                            name="sequenceOrder"
                            type="number"
                            value={editedRowData?.sequenceOrder}
                            onChange={handleInputChange}
                            size="small"
                        />
                    ) : (
                        level.sequenceOrder
                    )}
                </TableCell>
                <TableCell>
                    {isEditing ? (
                        <>
                            <IconButton onClick={handleSave} color="primary"><SaveIcon /></IconButton>
                            <IconButton onClick={handleCancel}><CancelIcon /></IconButton>
                        </>
                    ) : (
                        <>
                            <IconButton onClick={() => handleEdit(level)} color="primary"><EditIcon /></IconButton>
                            <IconButton onClick={() => handleDelete(level.levelId)} color="error"><DeleteIcon /></IconButton>
                        </>
                    )}
                </TableCell>
            </TableRow>
        );
    };
    
    const renderAddRow = () => (
        <TableRow>
            <TableCell>(New)</TableCell>
            <TableCell>
                <TextField name="levelName" placeholder="New Level Name" onChange={handleInputChange} size="small" autoFocus />
            </TableCell>
            <TableCell>
                <TextField name="sequenceOrder" type="number" placeholder="Order" onChange={handleInputChange} size="small" />
            </TableCell>
            <TableCell>
                <IconButton onClick={handleAddNew} color="primary"><SaveIcon /></IconButton>
                <IconButton onClick={handleCancel}><CancelIcon /></IconButton>
            </TableCell>
        </TableRow>
    );

    return (
        <Box p={3}>
            <Box display="flex" justifyContent="space-between" alignItems="center" mb={2}>
                <Typography variant="h4" component="h1">Manage Levels</Typography>
                <Button variant="contained" startIcon={<AddIcon />} onClick={() => { setIsAdding(true); setEditedRowData({ levelName: '', sequenceOrder: 0 }); }}>
                    Add New Level
                </Button>
            </Box>
            
            <TableContainer component={Paper}>
                <Table>
                    <TableHead>
                        <TableRow>
                            <TableCell>ID</TableCell>
                            <TableCell>Level Name</TableCell>
                            <TableCell>Sequence Order</TableCell>
                            <TableCell>Actions</TableCell>
                        </TableRow>
                    </TableHead>
                    <TableBody>
                        {isLoading ? (
                            <TableRow><TableCell colSpan={4} align="center"><CircularProgress /></TableCell></TableRow>
                        ) : (
                           levels.map(renderRow)
                        )}
                        {isAdding && renderAddRow()}
                    </TableBody>
                </Table>
            </TableContainer>
        </Box>
    );
};

export default LevelsPage;
```

### **Step 5: Integrate into `App.tsx`**

Finally, replace the default content of `App.tsx` with your new `LevelsPage`.

**File: `src/App.tsx`**
```typescript
import React from 'react';
import { CssBaseline, ThemeProvider, createTheme } from '@mui/material';
import LevelsPage from './pages/LevelsPage';

// You can define a theme for your admin panel
const theme = createTheme({
    palette: {
        primary: {
            main: '#1976d2',
        },
    },
});

function App() {
    return (
        <ThemeProvider theme={theme}>
            <CssBaseline />
            {/* Later, you will add routing here to switch between pages */}
            <main>
                <LevelsPage />
            </main>
        </ThemeProvider>
    );
}

export default App;
```

### **How to Run It**

1.  **Start your ASP.NET Core backend** as usual (`dotnet run...`).
2.  **Navigate to your React app folder** in a new terminal (`cd admin-app`).
3.  **Run the React development server:** `npm start`.

Your browser should open to `http://localhost:3000`, and you will see your new, fully functional "Manage Levels" page built with React and MUI. It will fetch data from your API and allow full inline CRUD operations, just like your Razor Pages version, but now within a powerful and scalable React component.

This provides a perfect template for you to now rebuild the `Lessons`, `MainActivities`, and `ActivityTypes` pages.

### USER
great you can notice the UI logic of levels , MainActivity and ActivityType is same as inline table , so why dont we use a generalized inline table to handle this scenario , 
so we can reduce the code duplicates , and only thing we need to do is we can pass the titles which ever we need , 
but for the Levels and Lessons we have to include dynamic links as well , bcz levels have lesson and lessons have activities

### RESPONSE
You are absolutely right. That is a brilliant observation and the perfect next step in building a truly professional and maintainable React application.

Creating a single, reusable, generic `InlineCrudTable` component is the ideal solution. It will dramatically reduce code duplication, enforce a consistent UI/UX across your admin pages, and make adding new CRUD pages in the future incredibly fast.

Let's design and build this generic component.

---

### **Concept: The Generic `InlineCrudTable` Component**

This component will know nothing about "Levels" or "MainActivities." It will be a "dumb" but powerful component that receives its entire configuration via props.

**What it will need via `props`:**
*   `entityName`: A string for display purposes (e.g., "Level", "Lesson").
*   `apiService`: An object containing the CRUD functions (`getAll`, `create`, `update`, `delete`).
*   `columns`: An array of objects defining the table columns, their names, and field types.
*   `renderActions`: An optional function to render custom action buttons (like your "Manage Activities" link).

---

### **Step 1: Create the Generic API Service Type**

First, let's define a generic type for our API services in TypeScript.

**File: `src/api/apiService.types.ts`**
```typescript
export interface CrudApiService<T, TCreateDto> {
    getAll: () => Promise<T[]>;
    create: (newItem: TCreateDto) => Promise<T>;
    update: (id: number | string, itemToUpdate: Partial<TCreateDto>) => Promise<void>;
    delete: (id: number | string) => Promise<void>;
}
```

### **Step 2: Create the Reusable `InlineCrudTable` Component**

This will be our new, powerful, generic component.

**File: `src/components/common/InlineCrudTable.tsx`**
```typescript
import React, { useState, useEffect, useCallback } from 'react';
import {
    Table, TableBody, TableCell, TableContainer, TableHead, TableRow, Paper,
    Button, TextField, IconButton, Box, Typography, CircularProgress
} from '@mui/material';
import EditIcon from '@mui/icons-material/Edit';
import DeleteIcon from '@mui/icons-material/Delete';
import SaveIcon from '@mui/icons-material/Save';
import CancelIcon from '@mui/icons-material/Cancel';
import AddIcon from '@mui/icons-material/Add';
import { CrudApiService } from '../../api/apiService.types';

// --- Generic Props Definition ---
interface ColumnDef<T> {
    field: keyof T;
    headerName: string;
    type?: 'string' | 'number';
}

interface InlineCrudTableProps<T, TCreateDto> {
    entityName: string;
    apiService: CrudApiService<T, TCreateDto>;
    columns: ColumnDef<T>[];
    idField: keyof T;
    renderCustomActions?: (item: T) => React.ReactNode;
}

// --- The Generic Component ---
const InlineCrudTable = <T extends Record<string, any>, TCreateDto extends object>({
    entityName,
    apiService,
    columns,
    idField,
    renderCustomActions
}: InlineCrudTableProps<T, TCreateDto>) => {
    const [items, setItems] = useState<T[]>([]);
    const [isLoading, setIsLoading] = useState<boolean>(true);
    const [editRowId, setEditRowId] = useState<number | string | null>(null);
    const [editedRowData, setEditedRowData] = useState<Partial<TCreateDto> | null>(null);
    const [isAdding, setIsAdding] = useState<boolean>(false);

    const fetchData = useCallback(async () => {
        setIsLoading(true);
        try {
            const data = await apiService.getAll();
            setItems(data);
        } catch (error) {
            console.error(error);
        } finally {
            setIsLoading(false);
        }
    }, [apiService]);

    useEffect(() => {
        fetchData();
    }, [fetchData]);

    const handleEdit = (item: T) => {
        setEditRowId(item[idField]);
        // Dynamically create the initial edit object based on columns
        const initialEditData: Partial<TCreateDto> = {};
        columns.forEach(col => {
            (initialEditData as any)[col.field] = item[col.field];
        });
        setEditedRowData(initialEditData);
    };

    const handleCancel = () => {
        setEditRowId(null);
        setEditedRowData(null);
        setIsAdding(false);
    };

    const handleSave = async () => {
        if (!editRowId || !editedRowData) return;
        try {
            await apiService.update(editRowId, editedRowData);
            handleCancel();
            await fetchData();
        } catch (error) {
            console.error(error);
        }
    };

    const handleAddNew = async () => {
        if (!editedRowData) return;
        try {
            await apiService.create(editedRowData as TCreateDto);
            handleCancel();
            await fetchData();
        } catch (error) {
            console.error(error);
        }
    };
    
    const handleDelete = async (id: number | string) => {
        if (window.confirm(`Are you sure you want to delete this ${entityName}?`)) {
            try {
                await apiService.delete(id);
                await fetchData();
            } catch (error) {
                console.error(error);
            }
        }
    };

    const handleInputChange = (e: React.ChangeEvent<HTMLInputElement>) => {
        const { name, value } = e.target;
        const column = columns.find(c => c.field === name);
        const parsedValue = column?.type === 'number' ? parseInt(value, 10) || 0 : value;
        setEditedRowData(prev => ({ ...prev!, [name]: parsedValue }));
    };

    const renderRow = (item: T) => {
        const id = item[idField];
        const isEditing = editRowId === id;
        return (
            <TableRow key={id}>
                <TableCell>{id}</TableCell>
                {columns.map(col => (
                    <TableCell key={String(col.field)}>
                        {isEditing ? (
                            <TextField
                                name={String(col.field)}
                                type={col.type === 'number' ? 'number' : 'text'}
                                value={(editedRowData as any)?.[col.field] ?? ''}
                                onChange={handleInputChange}
                                size="small"
                                fullWidth
                            />
                        ) : (
                            item[col.field]
                        )}
                    </TableCell>
                ))}
                <TableCell>
                    {isEditing ? (
                        <>
                            <IconButton onClick={handleSave} color="primary"><SaveIcon /></IconButton>
                            <IconButton onClick={handleCancel}><CancelIcon /></IconButton>
                        </>
                    ) : (
                        <>
                            {renderCustomActions && renderCustomActions(item)}
                            <IconButton onClick={() => handleEdit(item)} color="primary"><EditIcon /></IconButton>
                            <IconButton onClick={() => handleDelete(id)} color="error"><DeleteIcon /></IconButton>
                        </>
                    )}
                </TableCell>
            </TableRow>
        );
    };
    
    const renderAddRow = () => {
        // Create an initial empty object for the new row
        const initialAddData: Partial<TCreateDto> = {};
        columns.forEach(col => {
            (initialAddData as any)[col.field] = col.type === 'number' ? 0 : '';
        });

        return (
            <TableRow>
                <TableCell>(New)</TableCell>
                {columns.map(col => (
                    <TableCell key={String(col.field)}>
                        <TextField
                            name={String(col.field)}
                            placeholder={col.headerName}
                            type={col.type === 'number' ? 'number' : 'text'}
                            value={(editedRowData as any)?.[col.field] ?? ''}
                            onChange={handleInputChange}
                            size="small"
                            fullWidth
                            autoFocus={columns.indexOf(col) === 0}
                        />
                    </TableCell>
                ))}
                <TableCell>
                    <IconButton onClick={handleAddNew} color="primary"><SaveIcon /></IconButton>
                    <IconButton onClick={handleCancel}><CancelIcon /></IconButton>
                </TableCell>
            </TableRow>
        );
    };

    return (
        <Box p={3}>
            <Box display="flex" justifyContent="space-between" alignItems="center" mb={2}>
                <Typography variant="h4" component="h1">Manage {entityName}s</Typography>
                <Button variant="contained" startIcon={<AddIcon />} onClick={() => setIsAdding(true)}>
                    Add New {entityName}
                </Button>
            </Box>
            
            <TableContainer component={Paper}>
                <Table>
                    <TableHead>
                        <TableRow>
                            <TableCell>ID</TableCell>
                            {columns.map(col => <TableCell key={String(col.field)}>{col.headerName}</TableCell>)}
                            <TableCell>Actions</TableCell>
                        </TableRow>
                    </TableHead>
                    <TableBody>
                        {isLoading ? (
                            <TableRow><TableCell colSpan={columns.length + 2} align="center"><CircularProgress /></TableCell></TableRow>
                        ) : (
                           items.map(renderRow)
                        )}
                        {isAdding && renderAddRow()}
                    </TableBody>
                </Table>
            </TableContainer>
        </Box>
    );
};

export default InlineCrudTable;
```

### **Step 3: How to Use the Generic Component**

Now, your page components become incredibly simple. They just need to provide the specific configuration for the generic table.

#### **Refactored `LevelsPage.tsx`**
```typescript
import React from 'react';
import { Link as RouterLink } from 'react-router-dom'; // Assuming you use react-router
import { Button } from '@mui/material';
import InlineCrudTable from '../components/common/InlineCrudTable';
import { Level } from '../types/level';
import * as levelApi from '../api/levelApi';

const LevelsPage: React.FC = () => {
    // 1. Define the columns
    const columns = [
        { field: 'levelName', headerName: 'Level Name', type: 'string' },
        { field: 'sequenceOrder', headerName: 'Sequence Order', type: 'number' }
    ];

    // 2. Define the DTO for creating a new level
    type LevelCreateDto = Omit<Level, 'levelId'>;

    // 3. Define custom actions
    const renderCustomLevelActions = (level: Level) => (
        <Button 
            component={RouterLink} 
            to={`/lessons?levelId=${level.levelId}`}
            variant="outlined" 
            size="small"
            sx={{ mr: 1 }}
        >
            Manage Lessons
        </Button>
    );

    return (
        <InlineCrudTable<Level, LevelCreateDto>
            entityName="Level"
            apiService={levelApi}
            columns={columns}
            idField="levelId"
            renderCustomActions={renderCustomLevelActions}
        />
    );
};

export default LevelsPage;
```

#### **New `MainActivityPage.tsx`**
```typescript
import React from 'react';
import InlineCrudTable from '../components/common/InlineCrudTable';
import { MainActivity } from '../types/mainActivity'; // Create this type file
import * as mainActivityApi from '../api/mainActivityApi'; // Create this api file

const MainActivityPage: React.FC = () => {
    const columns = [
        { field: 'name', headerName: 'Activity Name', type: 'string' }
    ];

    type MainActivityCreateDto = Omit<MainActivity, 'id'>;

    return (
        // No custom actions needed here, so we don't pass the prop
        <InlineCrudTable<MainActivity, MainActivityCreateDto>
            entityName="Main Activity"
            apiService={mainActivityApi}
            columns={columns}
            idField="id"
        />
    );
};

export default MainActivityPage;
```

### **Summary of this Approach**

1.  **DRY (Don't Repeat Yourself):** You've eliminated nearly 90% of the duplicated code. The entire state management and rendering logic for an inline CRUD table is now in one place.
2.  **Highly Reusable:** You can now create a new admin page for `ActivityTypes` in about 5 minutes by creating the type, the API service, and a simple page component like the one for `MainActivityPage`.
3.  **Flexible and Extensible:** The `renderCustomActions` prop is the key to handling special cases. It allows the `LevelsPage` (and `LessonsPage` in the future) to inject their unique "Manage..." links without cluttering the generic component with specific logic.

This is a professional, scalable, and highly efficient way to build your admin panel. You've now created a powerful internal tool (the `InlineCrudTable` component) that will accelerate all future development.


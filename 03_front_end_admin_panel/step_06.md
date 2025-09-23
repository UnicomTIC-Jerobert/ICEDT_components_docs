### USER
Great now lets build Lesson Page As well , you know which columns should Appear thier , and we have to intoduce manage Activities links as well 

### RESPONSE
Yes, absolutely. Let's build the `LessonsPage`. This is the most complex page yet because it combines several key features:

1.  **Dependency on another page:** It needs to know which `Level` it's displaying lessons for. We'll get this from the URL query string (`?levelId=...`).
2.  **Filtered Data:** It won't fetch *all* lessons, only the ones for the selected level.
3.  **Inline CRUD:** It will use our generic `InlineCrudTable` for its own CRUD operations.
4.  **Custom Action Link:** Like the `Levels` page, it will have a custom "Manage Activities" link for each lesson.

This page will truly showcase the power of the component-based, reusable architecture we've built.

---

### **Step 1: Update the API**

Our `levelApi.ts` is fine, but our `lessonApi.ts` needs to be able to fetch lessons by `levelId`.

**File: `src/api/lessonApi.ts` (Create or update this file)**
```typescript
import { Lesson } from '../types/lesson'; // Create lesson.ts type file next

const API_BASE_URL = '/api';

export type LessonGetDto = Omit<Lesson, 'level'>; // DTO for getting lessons
export type LessonCreateDto = Omit<Lesson, 'lessonId' | 'level'>; // DTO for creating/updating

// GET lessons for a specific level
export const getLessonsByLevelId = async (levelId: number | string): Promise<LessonGetDto[]> => {
    const response = await fetch(`${API_BASE_URL}/levels/${levelId}/lessons`);
    if (!response.ok) {
        throw new Error('Failed to fetch lessons for this level');
    }
    return response.json();
};

// POST a new lesson
export const create = async (newItem: LessonCreateDto): Promise<Lesson> => {
    const response = await fetch(`${API_BASE_URL}/lessons`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(newItem)
    });
    if (!response.ok) {
        throw new Error('Failed to create lesson');
    }
    return response.json();
};

// PUT (update) an existing lesson
export const update = async (id: number | string, itemToUpdate: Partial<LessonCreateDto>): Promise<void> => {
    const response = await fetch(`${API_BASE_URL}/lessons/${id}`, {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(itemToUpdate)
    });
    if (!response.ok) {
        throw new Error('Failed to update lesson');
    }
};

// DELETE a lesson
export const deleteItem = async (id: number | string): Promise<void> => {
    const response = await fetch(`${API_BASE_URL}/lessons/${id}`, {
        method: 'DELETE'
    });
    if (!response.ok) {
        throw new Error('Failed to delete lesson');
    }
};
```

### **Step 2: Define the TypeScript Type**

**File: `src/types/lesson.ts` (Create this new file)**
```typescript
import { Level } from './level';

export interface Lesson {
    lessonId: number;
    levelId: number;
    lessonName: string;
    description: string | null;
    sequenceOrder: number;
    level?: Level; // Optional navigation property
}
```

### **Step 3: Build the `LessonsPage` Component**

This component will be a "wrapper" that reads the `levelId` from the URL and then passes the correctly configured props to our generic `InlineCrudTable`.

**File: `src/pages/LessonsPage.tsx` (Create this new file)**
```typescript
import React, { useMemo } from 'react';
import { Link as RouterLink, useLocation, useNavigate } from 'react-router-dom';
import { Button, IconButton, Typography } from '@mui/material';
import ArrowBackIcon from '@mui/icons-material/ArrowBack';
import InlineCrudTable from '../components/common/InlineCrudTable';
import { Lesson } from '../types/lesson';
import * as lessonApi from '../api/lessonApi';
import { LessonCreateDto } from '../api/lessonApi';

// A custom hook to easily get URL query parameters
function useQuery() {
    return new URLSearchParams(useLocation().search);
}

const LessonsPage: React.FC = () => {
    const query = useQuery();
    const navigate = useNavigate();
    const levelId = query.get('levelId');

    // Define the API service with the levelId baked in.
    // useMemo ensures this object is not recreated on every render.
    const apiService = useMemo(() => {
        if (!levelId) return null;
        
        const numericLevelId = parseInt(levelId, 10);

        return {
            getAll: () => lessonApi.getLessonsByLevelId(numericLevelId),
            create: (newItem: LessonCreateDto) => lessonApi.create({ ...newItem, levelId: numericLevelId }),
            update: (id: number | string, itemToUpdate: Partial<LessonCreateDto>) => lessonApi.update(id, { ...itemToUpdate, levelId: numericLevelId }),
            delete: lessonApi.deleteItem
        };
    }, [levelId]);

    // Define the columns for the table.
    const columns = [
        { field: 'lessonName' as keyof Lesson, headerName: 'Lesson Name', type: 'string' as const },
        { field: 'description' as keyof Lesson, headerName: 'Description', type: 'string' as const },
        { field: 'sequenceOrder' as keyof Lesson, headerName: 'Sequence Order', type: 'number' as const }
    ];

    // Define the custom "Manage Activities" button.
    const renderCustomLessonActions = (lesson: Lesson) => (
        <Button 
            component={RouterLink} 
            to={`/activities?lessonId=${lesson.lessonId}`}
            variant="outlined" 
            size="small"
            sx={{ mr: 1 }}
        >
            Manage Activities
        </Button>
    );

    // If levelId is missing from the URL, show an error and a way back.
    if (!apiService || !levelId) {
        return (
            <div style={{ padding: '20px' }}>
                <Typography variant="h5" color="error">Error: No Level ID provided.</Typography>
                <Button 
                    startIcon={<ArrowBackIcon />} 
                    sx={{ mt: 2 }}
                    variant="contained"
                    onClick={() => navigate('/levels')}
                >
                    Back to Levels
                </Button>
            </div>
        );
    }
    
    return (
        <div>
            <IconButton onClick={() => navigate('/levels')} sx={{ mb: 2 }}>
                <ArrowBackIcon />
                <Typography variant="button" sx={{ ml: 1 }}>Back to Levels</Typography>
            </IconButton>

            <InlineCrudTable<Lesson, LessonCreateDto>
                entityName={`Lesson for Level ${levelId}`}
                apiService={apiService}
                columns={columns}
                idField="lessonId"
                renderCustomActions={renderCustomLessonActions}
            />
        </div>
    );
};

export default LessonsPage;
```

### **Step 4: Add the Route in `App.tsx`**

Finally, add the new route so the page can be accessed.

**File: `src/App.tsx` (Modified)**
```typescript
// ... (imports)
import LessonsPage from './pages/LessonsPage'; // <-- IMPORT NEW PAGE

// ...

function App() {
    return (
        // ... (ThemeProvider, Router, AppBar)
                
                <Container component="main" sx={{ mt: 4 }}>
                    <Routes>
                        <Route path="/" element={<Typography variant="h5">Welcome!</Typography>} />
                        <Route path="/levels" element={<LevelsPage />} />
                        <Route path="/lessons" element={<LessonsPage />} /> {/* <-- ADD NEW ROUTE */}
                        <Route path="/main-activities" element={<MainActivityPage />} />
                        <Route path="/activity-types" element={<ActivityTypesPage />} />
                    </Routes>
                </Container>
            </Router>
        </ThemeProvider>
    );
}
```

### **Summary of the Implementation**

1.  **URL-Driven State:** The page is completely driven by the `levelId` in the URL query string. This makes the page linkable and refreshable.
2.  **Custom Hook (`useQuery`):** A small helper makes reading from the URL clean and reusable.
3.  **Dynamic API Service (`useMemo`):** We construct the `apiService` object on the fly, "baking in" the required `levelId` for the `getAll` and `create` calls. `useMemo` is a performance optimization that prevents this object from being recreated unnecessarily.
4.  **Graceful Error Handling:** If a user navigates to `/lessons` without a `levelId`, the page displays a helpful error message and a button to go back.
5.  **Reusability:** The core of the page is still our generic `InlineCrudTable`. All we had to do was provide a new configuration (columns, API service, custom actions).

You now have a fully functional page for managing lessons that are correctly filtered by their parent level, and it seamlessly integrates with the rest of your admin panel.

### USER

### USER
ok , im good to go ahead of this so , give complete implementation of MainActivity page , bcz it has only CRUD opertions

### RESPONSE
Excellent choice. Starting with `MainActivityPage` is perfect because it's the simplest use case for our new generic `InlineCrudTable` component, with no custom actions. This will serve as a clean and clear example of how powerful this reusable component is.

Here is the complete, end-to-end implementation for the "Manage Main Activities" page.

---

### **Step 1: Define the TypeScript Type**

First, we need to define the shape of our `MainActivity` data.

**File: `src/types/mainActivity.ts` (Create this new file)**
```typescript
export interface MainActivity {
    id: number;
    name: string;
}
```

### **Step 2: Create the API Service**

This file will contain all the functions to communicate with your `/api/mainactivities` endpoint.

**File: `src/api/mainActivityApi.ts` (Create this new file)**
```typescript
import { MainActivity } from '../types/mainActivity';

const API_BASE_URL = '/api/mainactivities';

// Define the type for the data needed to create/update a MainActivity
export type MainActivityCreateDto = Omit<MainActivity, 'id'>;

// GET all main activities
export const getAll = async (): Promise<MainActivity[]> => {
    const response = await fetch(API_BASE_URL);
    if (!response.ok) {
        throw new Error('Failed to fetch main activities');
    }
    return response.json();
};

// POST a new main activity
export const create = async (newItem: MainActivityCreateDto): Promise<MainActivity> => {
    const response = await fetch(API_BASE_URL, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(newItem)
    });
    if (!response.ok) {
        throw new Error('Failed to create main activity');
    }
    return response.json();
};

// PUT (update) an existing main activity
export const update = async (id: number | string, itemToUpdate: Partial<MainActivityCreateDto>): Promise<void> => {
    const response = await fetch(`${API_BASE_URL}/${id}`, {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(itemToUpdate)
    });
    if (!response.ok) {
        throw new Error('Failed to update main activity');
    }
};

// DELETE a main activity
export const deleteItem = async (id: number | string): Promise<void> => {
    const response = await fetch(`${API_BASE_URL}/${id}`, {
        method: 'DELETE'
    });
    if (!response.ok) {
        throw new Error('Failed to delete main activity');
    }
};
```

### **Step 3: Create the `MainActivityPage` Component**

This is the main page component. Notice how incredibly simple and declarative it is. It just configures and renders our generic `InlineCrudTable`.

**File: `src/pages/MainActivityPage.tsx` (Create this new file)**
```typescript
import React from 'react';
import InlineCrudTable from '../components/common/InlineCrudTable';
import { MainActivity } from '../types/mainActivity';
import * as mainActivityApi from '../api/mainActivityApi';
import { MainActivityCreateDto } from '../api/mainActivityApi'; // Import the DTO type

const MainActivityPage: React.FC = () => {
    
    // 1. Define the columns for the table.
    // 'field' must match a property in the MainActivity interface.
    // 'headerName' is what the user sees.
    const columns = [
        { field: 'name' as keyof MainActivity, headerName: 'Activity Name', type: 'string' as const }
    ];

    // 2. Define the API service object that the table will use.
    // The keys (getAll, create, etc.) must match the CrudApiService interface.
    const apiService = {
        getAll: mainActivityApi.getAll,
        create: mainActivityApi.create,
        update: mainActivityApi.update,
        delete: mainActivityApi.deleteItem // Use the specific name from the import
    };

    return (
        // 3. Render the generic table with the specific configuration.
        <InlineCrudTable<MainActivity, MainActivityCreateDto>
            entityName="Main Activity"
            apiService={apiService}
            columns={columns}
            idField="id" // Tell the component which property is the unique ID.
        />
    );
};

export default MainActivityPage;
```
*Note the `as keyof MainActivity` and `as const` additions. These are TypeScript best practices to ensure strong typing and prevent errors if you ever rename properties.*

### **Step 4: Add Routing in `App.tsx`**

To see this new page, you need to set up routing. We'll use `react-router-dom`, the standard for routing in React.

**1. Install React Router:**
In your `admin-app` terminal, run:
```bash
npm install react-router-dom
```

**2. Update `App.tsx` to handle routing:**
This will set up a basic navigation structure.

**File: `src/App.tsx` (Modified)**
```typescript
import React from 'react';
import { BrowserRouter as Router, Routes, Route, Link as RouterLink } from 'react-router-dom';
import { CssBaseline, ThemeProvider, createTheme, AppBar, Toolbar, Typography, Container, Box, Button } from '@mui/material';

// Import your page components
import LevelsPage from './pages/LevelsPage';
import MainActivityPage from './pages/MainActivityPage';
// Import other pages as you create them
// import LessonsPage from './pages/LessonsPage';
// import ActivityTypesPage from './pages/ActivityTypesPage';

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
            <Router>
                {/* Simple Navigation Header */}
                <AppBar position="static">
                    <Toolbar>
                        <Typography variant="h6" component="div" sx={{ flexGrow: 1 }}>
                            Tamil App Admin
                        </Typography>
                        <Button component={RouterLink} to="/levels" color="inherit">Levels</Button>
                        <Button component={RouterLink} to="/main-activities" color="inherit">Main Activities</Button>
                        {/* <Button component={RouterLink} to="/activity-types" color="inherit">Activity Types</Button> */}
                    </Toolbar>
                </AppBar>

                {/* Main Content Area */}
                <Container component="main" sx={{ mt: 4 }}>
                    <Routes>
                        {/* Define the route for each page */}
                        <Route path="/" element={<Typography variant="h5">Welcome to the Admin Panel!</Typography>} />
                        <Route path="/levels" element={<LevelsPage />} />
                        <Route path="/main-activities" element={<MainActivityPage />} />
                        {/* <Route path="/lessons" element={<LessonsPage />} /> */}
                        {/* <Route path="/activity-types" element={<ActivityTypesPage />} /> */}
                    </Routes>
                </Container>
            </Router>
        </ThemeProvider>
    );
}

export default App;
```

### **Final Check**

You now have a complete implementation.
1.  Run your ASP.NET Core backend.
2.  Run your React frontend (`npm start`).
3.  Navigate to `http://localhost:3000`. You'll see the welcome message and the new navigation bar.
4.  Click on the "Main Activities" link in the nav bar.
5.  You should see your fully functional, generic inline CRUD table for managing Main Activities.

This process is now your template. You can repeat it for `ActivityTypesPage` in a matter of minutes.

### USER
ok , now the app is work fine and shall we develop activity editor page

### RESPONSE
Excellent news! Getting the full CRUD workflow running for all the main entities is a huge accomplishment.

Yes, let's dive into the most interesting and crucial part of the admin panel: the **`ActivityEditor` page**.

We'll build this based on the architecture we designed: a two-column layout with the **Form on the left** and a **live Device Preview on the right**. The preview will dynamically render the correct activity component based on the JSON content.

---

### **Step 1: The Parent `ActivityEditor` Component**

This component will be the "brain" of the page. It fetches the data and manages the central state that both columns will share. This is an evolution of the `ActivityEditPage.tsx` we previously designed.

**File: `src/pages/ActivityEditorPage.tsx`**
*(You can rename `ActivityEditPage.tsx` to this or create a new file)*
```typescript
import React, { useState, useEffect, useCallback } from 'react';
import { useLocation, useNavigate } from 'react-router-dom';
import { Box, Typography, CircularProgress, Paper, Grid } from '@mui/material';
import ArrowBackIcon from '@mui/icons-material/ArrowBack';

import { Activity } from '../types/activity';
import * as activityApi from '../api/activityApi';

import ActivityForm from '../components/activities/ActivityForm'; // Left Column
import DevicePreview from '../components/activities/DevicePreview'; // Right Column

function useQuery() {
    return new URLSearchParams(useLocation().search);
}

const ActivityEditorPage: React.FC = () => {
    const query = useQuery();
    const navigate = useNavigate();
    const activityId = query.get('activityId');
    const lessonId = query.get('lessonId');
    const isEditMode = !!activityId;

    // The central state for the entire page
    const [activity, setActivity] = useState<Partial<Activity> | null>(null);
    const [isLoading, setIsLoading] = useState<boolean>(true);

    const backUrl = `/activities?lessonId=${activity?.lessonId || lessonId}`;

    const loadData = useCallback(async () => {
        setIsLoading(true);
        try {
            if (isEditMode && activityId) {
                const actData = await activityApi.getActivityById(activityId);
                // Pretty-print JSON for the editor
                try {
                   actData.contentJson = JSON.stringify(JSON.parse(actData.contentJson), null, 2);
                } catch { /* ignore if not valid json */ }
                setActivity(actData);
            } else {
                // Default state for a new activity
                setActivity({
                    title: '',
                    sequenceOrder: 1, // Default to 1
                    mainActivityId: 0,
                    activityTypeId: 0,
                    contentJson: '{}',
                    lessonId: parseInt(lessonId || '0', 10)
                });
            }
        } catch (error) {
            console.error("Failed to load data", error);
        } finally {
            setIsLoading(false);
        }
    }, [activityId, isEditMode, lessonId]);

    useEffect(() => {
        loadData();
    }, [loadData]);
    
    const handleFormChange = (updatedActivityData: Partial<Activity>) => {
        setActivity(updatedActivityData);
    };

    const handleSave = async () => {
        if (!activity) return;

        try {
            const payload = { ...activity };
            // Ensure numbers are numbers
            payload.sequenceOrder = Number(payload.sequenceOrder);
            payload.mainActivityId = Number(payload.mainActivityId);
            payload.activityTypeId = Number(payload.activityTypeId);
            payload.lessonId = Number(payload.lessonId);

            if (isEditMode && activityId) {
                await activityApi.updateActivity(activityId, payload as any);
            } else {
                await activityApi.createActivity(payload as any);
            }
            alert('Activity saved successfully!');
            navigate(backUrl);
        } catch (error) {
            console.error("Failed to save activity", error);
            alert("An error occurred while saving.");
        }
    };
    
    if (isLoading || !activity) {
        return <CircularProgress />;
    }

    return (
        <Box p={3}>
            <Box display="flex" justifyContent="space-between" alignItems="center" mb={2}>
                 <Typography variant="h4" component="h1">
                    {isEditMode ? `Edit Activity #${activityId}` : `Add New Activity`}
                </Typography>
                 <Button onClick={() => navigate(backUrl)} startIcon={<ArrowBackIcon />}>Back to List</Button>
            </Box>
           
            <Paper elevation={3} sx={{ p: 3 }}>
                <Grid container spacing={4}>
                    {/* Left Column: The Form */}
                    <Grid item xs={12} md={6}>
                        <ActivityForm 
                            activityData={activity}
                            onDataChange={handleFormChange}
                            onSave={handleSave}
                        />
                    </Grid>
                    
                    {/* Right Column: The Preview */}
                    <Grid item xs={12} md={6}>
                        <DevicePreview 
                            activityData={activity}
                        />
                    </Grid>
                </Grid>
            </Paper>
        </Box>
    );
};

export default ActivityEditorPage;
```

---

### **Step 2: The Form Component (`ActivityForm`)**

This component is responsible for all the input fields. It receives the current activity data and calls a function to update it whenever something changes.

**File: `src/components/activities/ActivityForm.tsx` (New component)**
```typescript
import React, { useState, useEffect } from 'react';
import { TextField, Grid, FormControl, InputLabel, Select, MenuItem, Button, SelectChangeEvent } from '@mui/material';
import { Activity } from '../../types/activity';
import { MainActivity } from '../../types/mainActivity';
import { ActivityType } from '../../types/activityType';
import * as mainActivityApi from '../../api/mainActivityApi';
import * as activityTypeApi from '../../api/activityTypeApi';

interface ActivityFormProps {
    activityData: Partial<Activity>;
    onDataChange: (updatedData: Partial<Activity>) => void;
    onSave: () => void;
}

const ActivityForm: React.FC<ActivityFormProps> = ({ activityData, onDataChange, onSave }) => {
    const [mainActivities, setMainActivities] = useState<MainActivity[]>([]);
    const [activityTypes, setActivityTypes] = useState<ActivityType[]>([]);
    const [jsonError, setJsonError] = useState<string>('');

    useEffect(() => {
        // Load dropdown data
        const loadDropdowns = async () => {
            const [mainActs, actTypes] = await Promise.all([
                mainActivityApi.getAll(),
                activityTypeApi.getAll()
            ]);
            setMainActivities(mainActs);
            setActivityTypes(actTypes);
        };
        loadDropdowns();
    }, []);

    const handleChange = (e: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement> | SelectChangeEvent<number>) => {
        const { name, value } = e.target;
        
        if (name === 'contentJson') {
            try {
                JSON.parse(value as string);
                setJsonError('');
            } catch {
                setJsonError('Invalid JSON format');
            }
        }
        onDataChange({ ...activityData, [name]: value });
    };

    const handleSubmit = (e: React.FormEvent) => {
        e.preventDefault();
        if (jsonError) {
            alert("Please fix the JSON content before saving.");
            return;
        }
        onSave();
    };

    return (
        <form onSubmit={handleSubmit}>
            <Grid container spacing={3}>
                <Grid item xs={12} md={8}>
                    <TextField fullWidth label="Activity Title" name="title" value={activityData.title || ''} onChange={handleChange} />
                </Grid>
                <Grid item xs={12} md={4}>
                    <TextField fullWidth type="number" label="Sequence Order" name="sequenceOrder" value={activityData.sequenceOrder || ''} onChange={handleChange} required />
                </Grid>
                <Grid item xs={12} md={6}>
                    <FormControl fullWidth>
                        <InputLabel>Main Activity Category</InputLabel>
                        <Select name="mainActivityId" value={activityData.mainActivityId || ''} onChange={handleChange} label="Main Activity Category">
                            {mainActivities.map(ma => <MenuItem key={ma.id} value={ma.id}>{ma.name}</MenuItem>)}
                        </Select>
                    </FormControl>
                </Grid>
                <Grid item xs={12} md={6}>
                     <FormControl fullWidth>
                        <InputLabel>Activity Type</InputLabel>
                        <Select name="activityTypeId" value={activityData.activityTypeId || ''} onChange={handleChange} label="Activity Type">
                            {activityTypes.map(at => <MenuItem key={at.id} value={at.id}>{at.name}</MenuItem>)}
                        </Select>
                    </FormControl>
                </Grid>
                <Grid item xs={12}>
                    <TextField
                        fullWidth
                        multiline
                        rows={15}
                        label="Content JSON"
                        name="contentJson"
                        value={activityData.contentJson || ''}
                        onChange={handleChange}
                        required
                        error={!!jsonError}
                        helperText={jsonError}
                        variant="outlined"
                        sx={{ fontFamily: 'monospace', '& .MuiOutlinedInput-root': { height: '100%' } }}
                    />
                </Grid>
                <Grid item xs={12}>
                    <Button type="submit" variant="contained" color="primary" size="large">Save Activity</Button>
                </Grid>
            </Grid>
        </form>
    );
};

export default ActivityForm;
```

---

### **Step 3: The Preview Component (`DevicePreview`)**

This component is the "player." It will render the activity inside a simulated phone frame. For now, we will create the frame and the logic to switch between components. **We will implement the actual activity components (`MCQActivity`, `MatchingActivity`, etc.) next.**

**File: `src/components/activities/DevicePreview.tsx` (New component)**
```typescript
import React, { useState } from 'react';
import { Box, Paper, Typography, ToggleButtonGroup, ToggleButton } from '@mui/material';
import PhoneIphoneIcon from '@mui/icons-material/PhoneIphone';
import TabletMacIcon from '@mui/icons-material/TabletMac';
import { Activity } from '../../types/activity';

// --- Placeholder Activity Components (We will build these out properly next) ---
const PlaceholderActivity = ({ type, content }: { type: string, content: any }) => (
    <Box p={2} sx={{ textAlign: 'center' }}>
        <Typography variant="h6">{type} Component</Typography>
        <Typography variant="body2" sx={{ color: 'text.secondary', wordBreak: 'break-all' }}>
            Content: {JSON.stringify(content)}
        </Typography>
    </Box>
);
const MatchingActivity = ({ content }: { content: any }) => <PlaceholderActivity type="Matching" content={content} />;
const MCQActivity = ({ content }: { content: any }) => <PlaceholderActivity type="MCQ" content={content} />;
// --- End of Placeholders ---

interface DevicePreviewProps {
    activityData: Partial<Activity>;
}

const DevicePreview: React.FC<DevicePreviewProps> = ({ activityData }) => {
    const [device, setDevice] = useState<'phone' | 'tablet'>('phone');

    const handleDeviceChange = (event: React.MouseEvent<HTMLElement>, newDevice: 'phone' | 'tablet' | null) => {
        if (newDevice !== null) {
            setDevice(newDevice);
        }
    };
    
    const renderActivityComponent = () => {
        if (!activityData.activityTypeId || !activityData.contentJson) {
            return <Typography p={2} color="text.secondary">Please select an activity type and provide JSON content.</Typography>;
        }
        
        let content;
        try {
            content = JSON.parse(activityData.contentJson);
        } catch (e) {
            return <Typography p={2} color="error">Invalid JSON format. Preview is paused.</Typography>;
        }

        switch (activityData.activityTypeId) {
            case 4: // Matching (Assuming ID from your DB)
                 return <MatchingActivity content={content} />;
            case 13: // MultipleChoiceQuestion
                 return <MCQActivity content={content} />;
            // Add cases for all 18 activity types here
            default:
                return <Typography p={2} color="text.secondary">Preview for this activity type is not yet implemented.</Typography>;
        }
    };

    const deviceStyles = {
        phone: { width: '375px', height: '667px' },
        tablet: { width: '768px', height: '1024px' }
    };

    return (
        <Box>
            <Box display="flex" justifyContent="center" mb={2}>
                 <ToggleButtonGroup
                    value={device}
                    exclusive
                    onChange={handleDeviceChange}
                    aria-label="device orientation"
                >
                    <ToggleButton value="phone" aria-label="phone">
                        <PhoneIphoneIcon />
                    </ToggleButton>
                    <ToggleButton value="tablet" aria-label="tablet">
                        <TabletMacIcon />
                    </ToggleButton>
                </ToggleButtonGroup>
            </Box>
            
            <Box display="flex" justifyContent="center">
                <Paper
                    elevation={6}
                    sx={{
                        width: deviceStyles[device].width,
                        height: deviceStyles[device].height,
                        borderRadius: '36px',
                        border: '10px solid black',
                        boxSizing: 'border-box',
                        overflow: 'hidden',
                        transition: 'width 0.3s, height 0.3s',
                        position: 'relative',
                        backgroundColor: '#fff'
                    }}
                >
                    <Box 
                        sx={{ 
                            position: 'absolute', top: 0, left: 0, right: 0, 
                            height: '25px', background: 'black', 
                            borderTopLeftRadius: '25px', borderTopRightRadius: '25px'
                        }}
                    >
                        <Box sx={{
                            width: '40%', height: '5px', background: '#333',
                            borderRadius: '5px', position: 'absolute',
                            top: '10px', left: '50%', transform: 'translateX(-50%)'
                        }}/>
                    </Box>
                    <Box sx={{ paddingTop: '25px', height: '100%', overflowY: 'auto' }}>
                        {renderActivityComponent()}
                    </Box>
                </Paper>
            </Box>
        </Box>
    );
};

export default DevicePreview;
```

### **Final Step: Update `App.tsx` Route**

Make sure the route points to your new editor page.
```typescript
// in App.tsx
import ActivityEditorPage from './pages/ActivityEditorPage';

// ... inside <Routes>
<Route path="/activity-edit" element={<ActivityEditorPage />} />
```

You now have a fully functional `ActivityEditor`. The admin can create and edit activities, and as they type valid JSON into the textarea, they will see a placeholder for the live preview on the right. Our next task will be to replace those placeholders with the actual, interactive React components for each activity type.

### USER
getting this errror
Property 'lessonName' does not exist on type 'LessonGetDto[]'.

### RESPONSE
You've hit a very common and important TypeScript error. This is exactly the kind of bug that TypeScript is designed to catch for you before your code even runs!

Let's break down the error, why it's happening, and how to fix it correctly.

### **Analyzing the Error**

> Property '`lessonName`' does not exist on type '`LessonGetDto[]`'.

**Translation:**
1.  **`LessonGetDto[]`**: The `[]` at the end means "an array of `LessonGetDto` objects".
2.  The error is telling you that you are trying to access a property called `lessonName` on an **entire array**, not on a single object *within* that array.
3.  Arrays don't have a `lessonName` property; only the individual lesson objects inside the array do.

**Where is the error happening?**

The error is almost certainly in your **`ActivitiesListPage.tsx`** component. Let's look at the `useEffect` hook where you fetch the data:

```typescript
// Inside ActivitiesListPage.tsx

useEffect(() => {
    // ...
    const fetchData = async () => {
        // ...
        try {
            // Here you are fetching the lesson details
            const lessonPromise = lessonApi.getLessonsByLevelId(lessonId); // <-- PROBLEM IS HERE

            // ...
            const [lesson, activitiesData] = await Promise.all([lessonPromise, activitiesPromise]);

            // And here you are trying to use it
            setLessonName(lesson.lessonName); // <-- ERROR OCCURS HERE
            // ...
        } // ...
    };

    fetchData();
}, [lessonId]);
```

You are calling `lessonApi.getLessonsByLevelId(lessonId)`. As the name implies (and as it's defined in your `lessonApi.ts`), this function returns a **list (an array)** of all lessons for a given level.

You need a function that fetches a **single lesson** by its specific `lessonId`.

---

### **The Solution: Create a `getLessonById` API Function**

The fix involves two simple steps: adding the correct API function and then calling it from your component.

#### **Step 1: Add `getLessonById` to your API Service**

We need a way to fetch the details of just one lesson.

**File: `src/api/lessonApi.ts` (Add this new function)**
```typescript
// ... (existing functions)

// GET a single lesson by its ID
export const getLessonById = async (lessonId: number | string): Promise<Lesson> => {
    const response = await fetch(`${API_BASE_URL}/lessons/${lessonId}`);
    if (!response.ok) {
        throw new Error('Failed to fetch lesson details');
    }
    return response.json();
};
```
*Note: This assumes you have the corresponding endpoint `GET /api/lessons/{id}` in your `LessonsController.cs`, which we designed earlier. Make sure it's implemented.*

#### **Step 2: Update `ActivitiesListPage.tsx` to Call the Correct Function**

Now, in your component, simply call this new function instead of the old one.

**File: `src/pages/ActivitiesListPage.tsx` (Corrected `useEffect`)**
```typescript
// ... (imports)

const ActivitiesListPage: React.FC = () => {
    // ... (state declarations)

    useEffect(() => {
        if (!lessonId) return;

        const fetchData = async () => {
            setIsLoading(true);
            try {
                // *** THE FIX IS HERE ***
                // Call the new function to get a SINGLE lesson object
                const lessonPromise = lessonApi.getLessonById(lessonId); 
                
                const activitiesPromise = activityApi.getActivitiesByLessonId(lessonId);

                // Now 'lesson' will be a single object, not an array
                const [lesson, activitiesData] = await Promise.all([lessonPromise, activitiesPromise]);

                // This will now work correctly!
                setLessonName(lesson.lessonName); 
                setActivities(activitiesData);

            } catch (error) {
                console.error(error);
            } finally {
                setIsLoading(false);
            }
        };

        fetchData();
    }, [lessonId]);

    // ... (rest of the component is the same)
};

export default ActivitiesListPage;
```

### **Why This Fixes the Problem**

*   **Before:** `lessonPromise` was resolving to an array (`LessonGetDto[]`). When you tried `lesson.lessonName`, you were essentially asking for a property on an array, which doesn't exist.
*   **After:** `lessonPromise` now resolves to a single object (`Lesson`). This object correctly has the `lessonName` property, so the line `setLessonName(lesson.lessonName)` is now valid, and the TypeScript error will disappear.

This is a perfect example of how TypeScript's strict typing helps you catch logical errors during development, saving you from frustrating runtime bugs later.


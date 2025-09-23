### USER
ok , then we will move to Activities page , you can remember we have Two pages for this one is for Entry&Update(Upsert) and other for Listing , 
this is not come under inline based operation , so can you plz implent this TWO pages with required other things as well 

### RESPONSE
Of course. This is the final and most complex piece of the admin panel. As we planned, we will move away from the inline-editing pattern for Activities because of the complexity of the `ContentJson`.

We will build this feature in two distinct parts:
1.  **`ActivitiesListPage.tsx`**: A read-only table that lists all activities for a specific lesson, with links to "Edit" and a button for "Add New".
2.  **`ActivityEditPage.tsx`**: A dedicated form for both creating and editing an activity, featuring a large textarea for the `ContentJson` and dropdowns for relationships.

This is the standard and best approach for managing complex entities.

---

### **Part 1: The Activities List Page**

This page is the destination when a user clicks "Manage Activities" from the `LessonsPage`.

#### **Step 1: Define TypeScript Types**

We need types for `Activity` and its related entities.

**File: `src/types/activity.ts` (Create this new file)**
```typescript
import { ActivityType } from './activityType';
import { MainActivity } from './mainActivity';

export interface Activity {
    activityId: number;
    lessonId: number;
    title: string | null;
    sequenceOrder: number;
    activityTypeId: number;
    mainActivityId: number;
    contentJson: string;
    
    // Optional navigation properties for display purposes
    activityType?: ActivityType; 
    mainActivity?: MainActivity;
}
```

#### **Step 2: Create the API Service**

**File: `src/api/activityApi.ts` (Create this new file)**
```typescript
import { Activity } from '../types/activity';

const API_BASE_URL = '/api';

export type ActivityCreateDto = Omit<Activity, 'activityId' | 'activityType' | 'mainActivity'>;
export type ActivityUpdateDto = Omit<Activity, 'activityId' | 'activityType' | 'mainActivity'>;

// GET activities for a specific lesson
export const getActivitiesByLessonId = async (lessonId: number | string): Promise<Activity[]> => {
    // NOTE: This assumes your backend API can populate the ActivityType and MainActivity names.
    // If not, we would need to fetch them separately and map them on the client.
    const response = await fetch(`${API_BASE_URL}/lessons/${lessonId}/activities`);
    if (!response.ok) throw new Error('Failed to fetch activities');
    return response.json();
};

// GET a single activity by its ID
export const getActivityById = async (activityId: number | string): Promise<Activity> => {
    const response = await fetch(`${API_BASE_URL}/activities/${activityId}`);
    if (!response.ok) throw new Error('Failed to fetch activity');
    return response.json();
};

// POST a new activity
export const createActivity = async (newItem: ActivityCreateDto): Promise<Activity> => {
    const response = await fetch(`${API_BASE_URL}/activities`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(newItem)
    });
    if (!response.ok) throw new Error('Failed to create activity');
    return response.json();
};

// PUT (update) an existing activity
export const updateActivity = async (id: number | string, itemToUpdate: ActivityUpdateDto): Promise<void> => {
    const response = await fetch(`${API_BASE_URL}/activities/${id}`, {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(itemToUpdate)
    });
    if (!response.ok) throw new Error('Failed to update activity');
};

// DELETE an activity
export const deleteActivity = async (id: number | string): Promise<void> => {
    const response = await fetch(`${API_BASE_URL}/activities/${id}`, {
        method: 'DELETE'
    });
    if (!response.ok) throw new Error('Failed to delete activity');
};
```

#### **Step 3: Build the `ActivitiesListPage.tsx` Component**

**File: `src/pages/ActivitiesListPage.tsx` (Create this new file)**
```typescript
import React, { useState, useEffect, useMemo } from 'react';
import { useLocation, useNavigate, Link as RouterLink } from 'react-router-dom';
import {
    Table, TableBody, TableCell, TableContainer, TableHead, TableRow, Paper,
    Button, IconButton, Box, Typography, CircularProgress
} from '@mui/material';
import EditIcon from '@mui/icons-material/Edit';
import DeleteIcon from '@mui/icons-material/Delete';
import AddIcon from '@mui/icons-material/Add';
import ArrowBackIcon from '@mui/icons-material/ArrowBack';

import { Activity } from '../types/activity';
import * as activityApi from '../api/activityApi';
import * as lessonApi from '../api/lessonApi'; // To get the lesson name

function useQuery() {
    return new URLSearchParams(useLocation().search);
}

const ActivitiesListPage: React.FC = () => {
    const query = useQuery();
    const navigate = useNavigate();
    const lessonId = query.get('lessonId');
    
    const [activities, setActivities] = useState<Activity[]>([]);
    const [lessonName, setLessonName] = useState<string>('');
    const [isLoading, setIsLoading] = useState<boolean>(true);

    useEffect(() => {
        if (!lessonId) return;

        const fetchData = async () => {
            setIsLoading(true);
            try {
                // Fetch both lesson name and activities in parallel
                const lessonPromise = lessonApi.getLessonById(lessonId); // You'll need to create this simple API function
                const activitiesPromise = activityApi.getActivitiesByLessonId(lessonId);

                const [lesson, activitiesData] = await Promise.all([lessonPromise, activitiesPromise]);

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

    const handleDelete = async (activityId: number) => {
        if (window.confirm("Are you sure you want to delete this activity?")) {
            try {
                await activityApi.deleteActivity(activityId);
                // Refresh the list after deleting
                setActivities(prev => prev.filter(act => act.activityId !== activityId));
            } catch (error) {
                console.error(error);
                alert("Failed to delete activity.");
            }
        }
    };

    if (!lessonId) {
        return <Typography color="error">Error: No Lesson ID provided.</Typography>;
    }

    return (
        <Box p={3}>
            <IconButton onClick={() => navigate('/lessons')} sx={{ mb: 2 }}>
                <ArrowBackIcon />
                <Typography variant="button" sx={{ ml: 1 }}>Back to Lessons</Typography>
            </IconButton>

            <Box display="flex" justifyContent="space-between" alignItems="center" mb={2}>
                <Typography variant="h4" component="h1">
                    Activities for: "{lessonName}"
                </Typography>
                <Button 
                    component={RouterLink} 
                    to={`/activity-edit?lessonId=${lessonId}`}
                    variant="contained" 
                    startIcon={<AddIcon />}
                >
                    Add New Activity
                </Button>
            </Box>
            
            <TableContainer component={Paper}>
                <Table>
                    <TableHead>
                        <TableRow>
                            <TableCell>ID</TableCell>
                            <TableCell>Title</TableCell>
                            <TableCell>Order</TableCell>
                            <TableCell>Actions</TableCell>
                        </TableRow>
                    </TableHead>
                    <TableBody>
                        {isLoading ? (
                            <TableRow><TableCell colSpan={4} align="center"><CircularProgress /></TableCell></TableRow>
                        ) : (
                           activities.map(activity => (
                               <TableRow key={activity.activityId}>
                                   <TableCell>{activity.activityId}</TableCell>
                                   <TableCell>{activity.title}</TableCell>
                                   <TableCell>{activity.sequenceOrder}</TableCell>
                                   <TableCell>
                                       <IconButton 
                                           component={RouterLink} 
                                           to={`/activity-edit?activityId=${activity.activityId}`}
                                           color="primary"
                                        >
                                           <EditIcon />
                                       </IconButton>
                                       <IconButton onClick={() => handleDelete(activity.activityId)} color="error">
                                           <DeleteIcon />
                                       </IconButton>
                                   </TableCell>
                               </TableRow>
                           ))
                        )}
                    </TableBody>
                </Table>
            </TableContainer>
        </Box>
    );
};

export default ActivitiesListPage;
```

---

### **Part 2: The Activity Add/Edit Page**

This is the dedicated form page we designed earlier.

**File: `src/pages/ActivityEditPage.tsx` (Create this new file)**
```typescript
import React, { useState, useEffect } from 'react';
import { useLocation, useNavigate, Link as RouterLink } from 'react-router-dom';
import {
    Box, Typography, TextField, Button, CircularProgress, Paper,
    Grid, FormControl, InputLabel, Select, MenuItem, SelectChangeEvent
} from '@mui/material';
import ArrowBackIcon from '@mui/icons-material/ArrowBack';

import { Activity } from '../types/activity';
import { MainActivity } from '../types/mainActivity';
import { ActivityType } from '../types/activityType';

import * as activityApi from '../api/activityApi';
import * as mainActivityApi from '../api/mainActivityApi';
import * as activityTypeApi from '../api/activityTypeApi';

function useQuery() {
    return new URLSearchParams(useLocation().search);
}

const ActivityEditPage: React.FC = () => {
    const query = useQuery();
    const navigate = useNavigate();
    const activityId = query.get('activityId');
    const lessonId = query.get('lessonId');
    const isEditMode = !!activityId;

    const [activity, setActivity] = useState<Partial<Activity>>({
        title: '',
        sequenceOrder: 0,
        mainActivityId: 0,
        activityTypeId: 0,
        contentJson: '{}',
        lessonId: parseInt(lessonId || '0', 10)
    });
    const [mainActivities, setMainActivities] = useState<MainActivity[]>([]);
    const [activityTypes, setActivityTypes] = useState<ActivityType[]>([]);
    const [isLoading, setIsLoading] = useState<boolean>(true);
    const [jsonError, setJsonError] = useState<string>('');

    useEffect(() => {
        const loadInitialData = async () => {
            setIsLoading(true);
            try {
                const mainActivitiesPromise = mainActivityApi.getAll();
                const activityTypesPromise = activityTypeApi.getAll();
                let activityPromise = Promise.resolve(null);

                if (isEditMode && activityId) {
                    activityPromise = activityApi.getActivityById(activityId);
                }

                const [mainActs, actTypes, actData] = await Promise.all([
                    mainActivitiesPromise, activityTypesPromise, activityPromise
                ]);

                setMainActivities(mainActs);
                setActivityTypes(actTypes);

                if (isEditMode && actData) {
                    // Pretty-print JSON for editing
                    try {
                       actData.contentJson = JSON.stringify(JSON.parse(actData.contentJson), null, 2);
                    } catch { /* ignore if not valid json */ }
                    setActivity(actData);
                }
            } catch (error) {
                console.error("Failed to load data", error);
            } finally {
                setIsLoading(false);
            }
        };

        loadInitialData();
    }, [activityId, isEditMode]);
    
    const handleInputChange = (e: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement>) => {
        const { name, value } = e.target;
        
        if (name === 'contentJson') {
            try {
                JSON.parse(value);
                setJsonError('');
            } catch {
                setJsonError('Invalid JSON format');
            }
        }

        setActivity(prev => ({ ...prev, [name]: value }));
    };

    const handleSelectChange = (e: SelectChangeEvent<number>) => {
        const { name, value } = e.target;
        setActivity(prev => ({ ...prev, [name]: Number(value) }));
    };
    
    const handleSubmit = async (e: React.FormEvent) => {
        e.preventDefault();
        if (jsonError) {
            alert("Please fix the JSON content before saving.");
            return;
        }

        const payload = {
            ...activity,
            sequenceOrder: Number(activity.sequenceOrder),
            mainActivityId: Number(activity.mainActivityId),
            activityTypeId: Number(activity.activityTypeId),
            lessonId: Number(activity.lessonId || lessonId)
        };

        try {
            if (isEditMode && activityId) {
                await activityApi.updateActivity(activityId, payload as activityApi.ActivityUpdateDto);
            } else {
                await activityApi.createActivity(payload as activityApi.ActivityCreateDto);
            }
            navigate(`/activities?lessonId=${payload.lessonId}`);
        } catch (error) {
            console.error("Failed to save activity", error);
            alert("An error occurred while saving.");
        }
    };

    if (isLoading) {
        return <CircularProgress />;
    }

    return (
        <Box p={3} component={Paper}>
            <IconButton onClick={() => navigate(-1)} sx={{ mb: 2 }}>
                <ArrowBackIcon />
                <Typography variant="button" sx={{ ml: 1 }}>Back</Typography>
            </IconButton>
            <Typography variant="h4" component="h1" mb={3}>
                {isEditMode ? `Edit Activity #${activityId}` : `Add New Activity for Lesson #${lessonId}`}
            </Typography>
            
            <form onSubmit={handleSubmit}>
                <Grid container spacing={3}>
                    <Grid item xs={12} md={8}>
                        <TextField fullWidth label="Activity Title" name="title" value={activity.title} onChange={handleInputChange} />
                    </Grid>
                    <Grid item xs={12} md={4}>
                        <TextField fullWidth type="number" label="Sequence Order" name="sequenceOrder" value={activity.sequenceOrder} onChange={handleInputChange} required />
                    </Grid>
                    <Grid item xs={12} md={6}>
                        <FormControl fullWidth>
                            <InputLabel>Main Activity Category</InputLabel>
                            <Select name="mainActivityId" value={activity.mainActivityId || ''} onChange={handleSelectChange} label="Main Activity Category">
                                {mainActivities.map(ma => <MenuItem key={ma.id} value={ma.id}>{ma.name}</MenuItem>)}
                            </Select>
                        </FormControl>
                    </Grid>
                    <Grid item xs={12} md={6}>
                         <FormControl fullWidth>
                            <InputLabel>Activity Type</InputLabel>
                            <Select name="activityTypeId" value={activity.activityTypeId || ''} onChange={handleSelectChange} label="Activity Type">
                                {activityTypes.map(at => <MenuItem key={at.id} value={at.id}>{at.name}</MenuItem>)}
                            </Select>
                        </FormControl>
                    </Grid>
                    <Grid item xs={12}>
                        <TextField
                            fullWidth
                            multiline
                            rows={12}
                            label="Content JSON"
                            name="contentJson"
                            value={activity.contentJson}
                            onChange={handleInputChange}
                            required
                            error={!!jsonError}
                            helperText={jsonError}
                            variant="outlined"
                            sx={{ fontFamily: 'monospace' }}
                        />
                    </Grid>
                    <Grid item xs={12}>
                        <Button type="submit" variant="contained" color="primary">Save Activity</Button>
                    </Grid>
                </Grid>
            </form>
        </Box>
    );
};

export default ActivityEditPage;
```

### **Step 5: Add Routes in `App.tsx`**

```typescript
// in App.tsx
import ActivitiesListPage from './pages/ActivitiesListPage';
import ActivityEditPage from './pages/ActivityEditPage';

// ... inside <Routes>
<Route path="/activities" element={<ActivitiesListPage />} />
<Route path="/activity-edit" element={<ActivityEditPage />} />
```

You now have a complete, robust, and user-friendly workflow for managing the most complex entity in your system. This two-page approach is highly scalable and provides a much better experience for the administrator than trying to cram everything into a single inline table.

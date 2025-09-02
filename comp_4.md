Excellent. I have analyzed the new set of activities from `Lesson 03` and `Lesson 07`.

You have identified another core activity pattern here. This is a slight but important variation of the `<RecognitionGrid />` component we designed earlier.

---

### **Activity Analysis**

*   **Examples from PDF:**
    *   `Lesson 03, செயல் 01`
    *   `Lesson 07, செயல் 01`
*   **Name:** "Find in Scene" or "Interactive Scene Recognition"
*   **Description:** A large, complex background image (a scene, like a farm or a forest) is displayed. An audio prompt plays, naming an object within that scene (e.g., "ஆடு" - goat). The user must find and tap on the correct object *within the larger image*. The instructions "படங்கள் மாறி, மாறித் தோன்ற வேண்டும்" (the pictures should change) suggest that different items will be highlighted or become interactive in sequence.
*   **Analysis:** This is a more advanced recognition task than the simple grid. It tests vocabulary and visual search skills. The core mechanic is still "Listen and Find," but the layout is a single, rich scene instead of a grid of isolated images.
*   **Classification:**
    *   **Main Activity Type ID:** `2` (`AudioImageRecognition`) - It's still a perfect fit for this category.
    *   **Component to Use:** We need a new, dedicated component for this. Let's call it **`<SceneFinder />`**.

---

### **Step 1: Define the `ContentJson` Type**

The JSON needs to define the background scene, the audio for the whole scene (optional), and the coordinates and details for each "tappable" object within the scene.

**File: `src/types/activityContentTypes.ts` (Add to this file)**
```typescript
// --- NEW: Type for Interactive Scene Finder Activity ---
export interface Hotspot {
    id: number;           // Unique ID for this object in the scene
    name: string;         // The name of the object, e.g., "ஆடு"
    audioUrl: string;     // The audio prompt that asks the user to find this object
    // Coordinates are percentages (0-100) for responsive design
    x: number;            // X-coordinate of the top-left corner
    y: number;            // Y-coordinate of the top-left corner
    width: number;        // Width of the tappable area
    height: number;       // Height of the tappable area
}

export interface SceneFinderContent {
    title: string;
    sceneImageUrl: string; // The main background image
    sceneAudioUrl?: string; // Optional ambient sound for the scene
    hotspots: Hotspot[];    // The list of all interactive objects in the scene
}
```

**Example JSON for `Lesson 07, செயல் 01` (The Farm Scene):**
```json
{
  "title": "Find the items on the farm",
  "sceneImageUrl": "https://your-bucket.../farm_scene.jpg",
  "hotspots": [
    {
      "id": 1, "name": "ஆடு", "audioUrl": ".../aadu_find.mp3",
      "x": 60, "y": 70, "width": 15, "height": 15 
    },
    {
      "id": 2, "name": "குதிரை", "audioUrl": ".../kuthirai_find.mp3",
      "x": 20, "y": 55, "width": 25, "height": 30
    },
    {
      "id": 3, "name": "சேவல்", "audioUrl": ".../seval_find.mp3",
      "x": 80, "y": 40, "width": 10, "height": 10
    },
    {
      "id": 4, "name": "வைக்கோல்", "audioUrl": ".../vaikkol_find.mp3",
      "x": 5, "y": 75, "width": 20, "height": 15
    }
  ]
}
```

---

### **Step 2: Create the `<SceneFinder />` Component**

This component will be responsible for displaying the scene, layering the invisible "hotspots" on top, and managing the game logic.

**File: `src/components/activities/activity-types/SceneFinder.tsx` (Create this new file)**
```typescript
import React, { useState, useEffect, useRef } from 'react';
import { Box, Typography, Paper, Fab, IconButton } from '@mui/material';
import CheckCircleIcon from '@mui/icons-material/CheckCircle';
import ReplayIcon from '@mui/icons-material/Replay';
import VolumeUpIcon from '@mui/icons-material/VolumeUp';
import { SceneFinderContent } from '../../../types/activityContentTypes';

interface SceneFinderProps {
    content: SceneFinderContent;
}

const SceneFinder: React.FC<SceneFinderProps> = ({ content }) => {
    const [foundItems, setFoundItems] = useState<number[]>([]);
    const [currentItemIndex, setCurrentItemIndex] = useState(0);
    const audioRef = useRef<HTMLAudioElement | null>(null);

    const itemsToFind = content.hotspots;
    const currentItem = itemsToFind[currentItemIndex];

    useEffect(() => {
        // Reset game when content changes
        setFoundItems([]);
        setCurrentItemIndex(0);
    }, [content]);

    useEffect(() => {
        // Autoplay the prompt for the current item to find
        if (currentItem?.audioUrl) {
            const timer = setTimeout(() => playAudio(currentItem.audioUrl), 500);
            return () => clearTimeout(timer);
        }
    }, [currentItem]);

    const playAudio = (audioUrl: string) => {
        if (audioRef.current) {
            audioRef.current.src = audioUrl;
            audioRef.current.play().catch(e => console.error(e));
        }
    };

    const handleHotspotClick = (hotspotId: number) => {
        if (foundItems.includes(hotspotId) || !currentItem) return;

        if (hotspotId === currentItem.id) {
            // Correct guess
            setFoundItems(prev => [...prev, hotspotId]);

            // Move to the next item
            if (currentItemIndex < itemsToFind.length - 1) {
                setCurrentItemIndex(prev => prev + 1);
            } else {
                // All items found
                setCurrentItemIndex(-1); // Sentinel value for completion
            }
        }
    };

    const handleReset = () => {
        setFoundItems([]);
        setCurrentItemIndex(0);
    };

    const isComplete = foundItems.length === itemsToFind.length;

    return (
        <Box p={2} sx={{ fontFamily: 'sans-serif', textAlign: 'center', height: '100%', display: 'flex', flexDirection: 'column' }}>
            <Typography variant="h5" component="h1" gutterBottom>{content.title}</Typography>
            
            <Paper elevation={2} sx={{ p: 1, mb: 2, display: 'flex', alignItems: 'center', justifyContent: 'center', gap: 2 }}>
                <Typography variant="h6">Find:</Typography>
                <Typography variant="h6" color="primary.main" fontWeight="bold">
                    {isComplete ? "Well Done!" : currentItem?.name || "..."}
                </Typography>
                <IconButton onClick={() => currentItem && playAudio(currentItem.audioUrl)} disabled={isComplete}>
                    <VolumeUpIcon fontSize="large" />
                </IconButton>
            </Paper>

            {/* The Scene Container */}
            <Box sx={{ flexGrow: 1, position: 'relative', width: '100%', maxWidth: '800px', margin: 'auto' }}>
                <img src={content.sceneImageUrl} alt={content.title} style={{ width: '100%', height: '100%', objectFit: 'contain' }} />
                
                {/* Overlay for Hotspots */}
                {itemsToFind.map(hotspot => {
                    const isFound = foundItems.includes(hotspot.id);
                    return (
                        <Box
                            key={hotspot.id}
                            onClick={() => handleHotspotClick(hotspot.id)}
                            sx={{
                                position: 'absolute',
                                top: `${hotspot.y}%`,
                                left: `${hotspot.x}%`,
                                width: `${hotspot.width}%`,
                                height: `${hotspot.height}%`,
                                cursor: isFound ? 'default' : 'pointer',
                                // For debugging, add a border:
                                // border: '1px dashed red', 
                            }}
                        >
                            {isFound && (
                                <Box sx={{ width: '100%', height: '100%', backgroundColor: 'rgba(46, 204, 113, 0.5)', borderRadius: '50%', display: 'flex', alignItems: 'center', justifyContent: 'center' }}>
                                    <CheckCircleIcon sx={{ fontSize: 40, color: 'white' }} />
                                </Box>
                            )}
                        </Box>
                    );
                })}
            </Box>
            
            <Box sx={{ height: '60px', mt: 2, display: 'flex', alignItems: 'center', justifyContent: 'center' }}>
                 {isComplete && (
                     <Button variant="contained" startIcon={<ReplayIcon />} onClick={handleReset}>
                        Play Again
                    </Button>
                )}
            </Box>
            <audio ref={audioRef} style={{ display: 'none' }} />
        </Box>
    );
};

export default SceneFinder;
```

---

### **Step 3: Integrate into `ActivityRenderer.tsx`**

We'll use another type guard under `activityTypeId = 2`.

**File: `src/components/activities/previews/ActivityRenderer.tsx` (Modified)**
```typescript
// ... (imports)
import SceneFinder from '../activity-types/SceneFinder';
import { SceneFinderContent } from '../../../types/activityContentTypes';

// ...

const ActivityRenderer: React.FC<ActivityRendererProps> = ({ activityTypeId, content }) => {
    // ...
    switch (activityTypeId) {
        case 2:
            // This check for hotspots should be specific
            if ('sceneImageUrl' in content && 'hotspots' in content) {
                return <SceneFinder content={content as SceneFinderContent} />;
            }
            // ... (rest of case 2 logic for RecognitionGrid, Flashcard, etc.)
            break;
        // ... (other cases)
    }
};
```

This completes the design for the `<SceneFinder />` component. It's a powerful and engaging way to handle the vocabulary recognition tasks that take place within a larger visual context.

You've provided another excellent set of activities. I have analyzed all three pages from the PDF.

This document contains **two distinct types** of activities that require two different components. One is a brand new pattern, and the other is a variation of something we've already built.

Let's break them down.

---

### **Activity Type 1: "Listen and Find the Letter/Character"**

*   **Examples from PDF:**
    *   Grade 01, Lesson 01, `செயல் 04`
    *   Grade 01, Lesson 02, `செயல் 02`
    *   Grade 01, Lesson 03, `செயல் 01`
    *   All activities on the second page (Lesson 06 to 09).
*   **Description:** An audio clip of a single Tamil letter/character (e.g., "க", "கா", "கி") is played. The user is presented with a grid of many different letters. The user must find and tap the correct letter that matches the sound.
*   **Analysis:** This is a pure recognition task. It's very similar to the `<RecognitionGrid />` component we designed for images, but instead of images, the grid contains text (the letters).
*   **Classification:**
    *   **Main Activity Type ID:** `2` (`AudioImageRecognition`) - We can reuse this as it's a recognition task.
    *   **Component to Use:** We need a new component. Let's call it **`<CharacterGrid />`**. It will be a variation of `<RecognitionGrid />` but optimized for displaying large, clear text characters instead of images.

#### **`ContentJson` for `<CharacterGrid />` Component:**
The structure can be almost identical to the `RecognitionGridContent`.

```typescript
// --- NEW: Type for Character Grid Activity ---
export interface CharacterGridItem {
    id: number;
    character: string; // The letter to display, e.g., "க"
    audioUrl: string;  // The audio of that letter's sound
}

export interface CharacterGridPage {
    gridItems: CharacterGridItem[];
    correctItemIds: number[];
}

export interface CharacterGridContent {
    title: string;
    pages: CharacterGridPage[];
}
```
**Example `ContentJson` for Grade 01, Lesson 01, `செயல் 04`:**
```json
{
  "title": "Find the letter that matches the sound",
  "pages": [
    {
      "gridItems": [
        { "id": 1, "character": "க", "audioUrl": ".../ka.mp3" },
        { "id": 2, "character": "ங", "audioUrl": ".../nga.mp3" },
        { "id": 3, "character": "ச", "audioUrl": ".../sa.mp3" },
        // ... and so on for all 18 letters
      ],
      "correctItemIds": [1, 2, 3, ...] // The user must find all of them
    }
  ]
}
```
This is a very efficient structure. The component can randomly pick a `correctItemId`, play its `audioUrl`, and wait for the user to tap the correct character in the grid.

---

### **Activity Type 2: "Auditory Word Pair Discrimination"**

*   **Example from PDF:** Lesson 01 - mcq words, `செயல் 04`.
*   **Description:** The app presents two visually similar words (e.g., "பல்" and "பால்"). An audio clip of **one** of the words is played (e.g., "பால்"). The user must tap the correct word that matches the sound.
*   **Analysis:** This is a crucial phonics exercise, testing the user's ability to distinguish between short and long vowel sounds (`குறில்/நெடில்`). It's a variation of a Multiple Choice Question where the "choices" are the words themselves.
*   **Classification:**
    *   **Main Activity Type ID:** `13` (`MultipleChoiceQuestion`) - It fits perfectly.
    *   **Component to Use:** We need a new component designed for this specific interaction. Let's call it **`<WordPairMCQ />`**.

#### **`ContentJson` for `<WordPairMCQ />` Component:**
The JSON will be an array of "questions," where each question consists of the word pair and the audio prompt.

```typescript
// --- NEW: Type for Word Pair MCQ Activity ---
export interface WordPairQuestion {
    id: number;
    // The audio prompt to play for this question
    promptAudioUrl: string; 
    // The two words to display as choices
    choices: [string, string];
    // The correct word
    correctAnswer: string;
}

export interface WordPairMCQContent {
    title: string;
    questions: WordPairQuestion[];
}
```
**Example `ContentJson` for Lesson 01 - mcq words, `செயல் 04`:**
```json
{
  "title": "Listen to the sound and choose the correct word",
  "questions": [
    {
      "id": 1,
      "promptAudioUrl": ".../paal_sound.mp3",
      "choices": ["பல்", "பால்"],
      "correctAnswer": "பால்"
    },
    {
      "id": 2,
      "promptAudioUrl": ".../aram_sound.mp3",
      "choices": ["அரம்", "ஆரம்"],
      "correctAnswer": "அரம்"
    },
    {
      "id": 3,
      "promptAudioUrl": ".../kaal_sound.mp3",
      "choices": ["கல்", "கால்"],
      "correctAnswer": "கால்"
    }
    // ... and so on for all word pairs
  ]
}
```
This structure provides everything the `<WordPairMCQ />` component needs to render the choices, play the correct sound, and validate the user's answer. The `ActivityPlayerModal` will handle the navigation between the different questions.

### **Summary and Plan**

Your analysis is correct. This document requires us to build **two new, reusable components**:

1.  **`<CharacterGrid />`:** For the "Listen and Find the Letter" activities. This will be visually similar to `<RecognitionGrid />` but will display text instead of images.
2.  **`<WordPairMCQ />`:** For the "Auditory Word Pair Discrimination" activity. This will be a specialized form of MCQ.

By building these two components, we will be able to cover every single activity presented in this new document.

Excellent. Let's build both of these new components. We'll start with the `<CharacterGrid />` as it's a variation of a pattern we've already built, and then move to the `<WordPairMCQ />`.

---

### **Component 1: `<CharacterGrid />`**

This component is for the "Listen and Find the Letter" activities. It's very similar to the `<RecognitionGrid />` but displays characters instead of images.

**File: `src/components/activities/activity-types/CharacterGrid.tsx` (Create this new file)**
```typescript
import React, { useState, useEffect, useRef } from 'react';
import { Box, Typography, Paper, Grid, Fab } from '@mui/material';
import CheckCircleIcon from '@mui/icons-material/CheckCircle';
import ReplayIcon from '@mui/icons-material/Replay';
import VolumeUpIcon from '@mui/icons-material/VolumeUp';
import ArrowForwardIcon from '@mui/icons-material/ArrowForward';
import { CharacterGridContent } from '../../../types/activityContentTypes';

interface CharacterGridProps {
    content: CharacterGridContent;
}

const CharacterGrid: React.FC<CharacterGridProps> = ({ content }) => {
    const [currentPageIndex, setCurrentPageIndex] = useState(0);
    const [foundItems, setFoundItems] = useState<number[]>([]);
    const [currentItemToFindIndex, setCurrentItemToFindIndex] = useState(0);
    const audioRef = useRef<HTMLAudioElement | null>(null);

    const currentPage = content.pages[currentPageIndex];
    const currentCorrectIds = currentPage.correctItemIds;
    const currentItemToFind = currentPage.gridItems.find(item => item.id === currentCorrectIds[currentItemToFindIndex]);

    useEffect(() => {
        setFoundItems([]);
        setCurrentItemToFindIndex(0);
    }, [currentPageIndex, content]);

    useEffect(() => {
        if (currentItemToFind?.audioUrl) {
            const timer = setTimeout(() => playAudio(currentItemToFind.audioUrl), 500);
            return () => clearTimeout(timer);
        }
    }, [currentItemToFind]);

    const playAudio = (audioUrl: string) => {
        if (audioRef.current) {
            audioRef.current.src = audioUrl;
            audioRef.current.play().catch(e => console.error("Audio playback failed:", e));
        }
    };

    const handleCharacterClick = (clickedItemId: number) => {
        if (foundItems.includes(clickedItemId) || !currentItemToFind) return;

        if (clickedItemId === currentItemToFind.id) {
            const newFoundItems = [...foundItems, clickedItemId];
            setFoundItems(newFoundItems);
            if (currentItemToFindIndex < currentCorrectIds.length - 1) {
                setCurrentItemToFindIndex(prev => prev + 1);
            } else {
                setCurrentItemToFindIndex(prev => prev + 1);
            }
        }
    };

    const goToNextPage = () => {
        if (currentPageIndex < content.pages.length - 1) {
            setCurrentPageIndex(prev => prev + 1);
        }
    };

    const isPageComplete = foundItems.length === currentCorrectIds.length;
    const isActivityComplete = isPageComplete && currentPageIndex === content.pages.length - 1;

    return (
        <Box p={2} sx={{ fontFamily: 'sans-serif', textAlign: 'center', height: '100%', display: 'flex', flexDirection: 'column' }}>
            <Typography variant="h5" component="h1" gutterBottom>{content.title}</Typography>
            
            <Paper elevation={2} sx={{ p: 1, mb: 2, display: 'flex', alignItems: 'center', justifyContent: 'center', gap: 2 }}>
                <Typography variant="h6">Listen:</Typography>
                <IconButton onClick={() => currentItemToFind && playAudio(currentItemToFind.audioUrl)} disabled={isPageComplete}>
                    <VolumeUpIcon fontSize="large" color="primary" />
                </IconButton>
            </Paper>

            <Box sx={{ flexGrow: 1 }}>
                <Grid container spacing={1} justifyContent="center" alignItems="center">
                    {currentPage.gridItems.map(item => {
                         const isFound = foundItems.includes(item.id);
                         return (
                            <Grid item key={item.id} xs={3} sm={2}>
                                <Paper
                                    onClick={() => handleCharacterClick(item.id)}
                                    sx={{
                                        aspectRatio: '1 / 1', // Make it a square
                                        display: 'flex',
                                        alignItems: 'center',
                                        justifyContent: 'center',
                                        cursor: 'pointer',
                                        borderRadius: '8px',
                                        border: '2px solid',
                                        borderColor: isFound ? 'success.main' : 'grey.300',
                                        backgroundColor: isFound ? 'success.light' : 'white',
                                        transition: 'transform 0.2s, background-color 0.2s',
                                        '&:hover': { transform: 'scale(1.1)' }
                                    }}
                                >
                                    <Typography variant="h3" fontWeight="bold">
                                        {item.character}
                                    </Typography>
                                </Paper>
                            </Grid>
                        );
                    })}
                </Grid>
            </Box>
            
            <Box sx={{ height: '80px', mt: 2, display: 'flex', alignItems: 'center', justifyContent: 'center' }}>
                {isPageComplete && !isActivityComplete && (
                    <Fab color="primary" variant="extended" onClick={goToNextPage}>
                        Next Page <ArrowForwardIcon sx={{ ml: 1 }} />
                    </Fab>
                )}
                {isActivityComplete && (
                     <Box textAlign="center">
                        <Typography variant="h5" color="success.main">Well Done!</Typography>
                        <Button startIcon={<ReplayIcon />} onClick={() => setCurrentPageIndex(0)}>Play Again</Button>
                    </Box>
                )}
            </Box>

            <audio ref={audioRef} style={{ display: 'none' }} />
        </Box>
    );
};

export default CharacterGrid;
```
**Integration into `ActivityRenderer.tsx` (assuming `activityTypeId = 2`):**
```typescript
// ...
case 2:
    if ('pages' in content && content.pages[0]?.gridItems[0]?.character) {
        return <CharacterGrid content={content as CharacterGridContent} />;
    }
    // ... (rest of case 2 logic)
// ...
```

---

### **Component 2: `<WordPairMCQ />`**

This component is for the "Auditory Word Pair Discrimination" activity.

**File: `src/components/activities/activity-types/WordPairMCQ.tsx` (Create this new file)**
```typescript
import React, { useState, useEffect, useRef } from 'react';
import { Box, Typography, Paper, Button, IconButton } from '@mui/material';
import VolumeUpIcon from '@mui/icons-material/VolumeUp';
import CheckCircleIcon from '@mui/icons-material/CheckCircle';
import CancelIcon from '@mui/icons-material/Cancel';
import ReplayIcon from '@mui/icons-material/Replay';
import { WordPairMCQContent, WordPairQuestion } from '../../../types/activityContentTypes';

interface WordPairProps {
    content: WordPairMCQContent;
}

// This is a "mini-player" that will be managed by the parent ActivityPlayerModal
const WordPairMCQ: React.FC<{ question: WordPairQuestion }> = ({ question }) => {
    const [userAnswer, setUserAnswer] = useState<string | null>(null);
    const audioRef = useRef<HTMLAudioElement | null>(null);

    useEffect(() => {
        // Reset state when the question changes
        setUserAnswer(null);
        // Autoplay the prompt sound
        const timer = setTimeout(() => playAudio(), 500);
        return () => clearTimeout(timer);
    }, [question]);

    const playAudio = () => {
        if (audioRef.current) {
            audioRef.current.play().catch(e => console.error(e));
        }
    };

    const handleAnswer = (choice: string) => {
        if (userAnswer) return; // Already answered
        setUserAnswer(choice);
    };

    return (
        <Box p={2} textAlign="center">
            <audio ref={audioRef} src={question.promptAudioUrl} />
            <Paper elevation={2} sx={{ p: 2, mb: 3 }}>
                <Typography variant="h6">Listen to the sound and choose the correct word:</Typography>
                <IconButton onClick={playAudio}><VolumeUpIcon fontSize="large" color="primary" /></IconButton>
            </Paper>

            <Box display="flex" justifyContent="space-around" alignItems="center">
                {question.choices.map(choice => {
                    const isSelected = userAnswer === choice;
                    const isCorrect = choice === question.correctAnswer;
                    let color: "success" | "error" | "primary" = "primary";
                    if (isSelected && isCorrect) color = "success";
                    if (isSelected && !isCorrect) color = "error";

                    return (
                        <Button
                            key={choice}
                            variant={userAnswer ? 'contained' : 'outlined'}
                            color={color}
                            onClick={() => handleAnswer(choice)}
                            disabled={!!userAnswer && !isSelected}
                            sx={{ fontSize: '2rem', padding: '20px 40px', textTransform: 'none' }}
                        >
                            {choice}
                            {isSelected && isCorrect && <CheckCircleIcon sx={{ ml: 1 }} />}
                            {isSelected && !isCorrect && <CancelIcon sx={{ ml: 1 }} />}
                        </Button>
                    );
                })}
            </Box>

             {userAnswer && ! (userAnswer === question.correctAnswer) && (
                <Typography variant="h6" color="success.main" mt={2}>
                    Correct Answer: {question.correctAnswer}
                </Typography>
            )}
        </Box>
    );
};

// You can then use this inside the ActivityPlayerModal's flow if your ContentJson
// is an array of questions.

export default WordPairMCQ;
```
**Integration into `ActivityRenderer.tsx` (assuming `activityTypeId = 13`):**
```typescript
// ...
case 13:
    if ('questions' in content && content.questions[0]?.choices?.length === 2) {
        // This is a "mini-player" handled by the parent ActivityPlayerModal
        const singleQuestion = content.questions[currentQuestionIndex] || {};
        return <WordPairMCQ question={singleQuestion as WordPairQuestion} />;
    }
    // Fallback to the original MCQ component
    return <MCQActivity content={content as MCQContent | MCQContent[]} />;
// ...
```
This demonstrates how the renderer can host different "flavors" of MCQ based on the JSON structure, while the `ActivityPlayerModal` provides the overall navigation between the individual questions in the `questions` array.


### ** Fixed Issues **

You are absolutely right. The components described in that document (`CharacterGrid` and `WordPairMCQ`) are following the old, incorrect pattern where the component itself tries to manage the array of exercises.

Let's refactor both of them to adhere to our final, correct architecture:
1.  The component will be **"dumb"** and only know how to render a **single** exercise/question.
2.  The props interface will accept a **single content object**, not an array.
3.  All navigation logic (Next/Previous) is removed. This is the responsibility of the `ActivityPlayerModal`.
4.  The type definitions will be **colocated** in the same file as the component.

Here are the complete, refactored versions of both components.

---

### **Component 1: `<CharacterGrid />` (Refactored)**

This component will now render a single grid for a single "find all these letters" challenge. The `ActivityPlayerModal` will handle switching between different pages/challenges.

**File: `src/components/activities/activity-types/CharacterGrid.tsx` (Refactored)**
```typescript
import React, { useState, useEffect, useRef } from 'react';
import { Box, Typography, Paper, Grid, IconButton, Button } from '@mui/material';
import CheckCircleIcon from '@mui/icons-material/CheckCircle';
import ReplayIcon from '@mui/icons-material/Replay';
import VolumeUpIcon from '@mui/icons-material/VolumeUp';

// --- COLOCATED TYPES ---
export interface CharacterGridItem {
    id: number;
    character: string;
    audioUrl: string;
}

// This is the content for a SINGLE page/grid
export interface CharacterGridContent {
    title: string;
    gridItems: CharacterGridItem[];
    correctItemIds: number[];
}

// --- PROPS INTERFACE ---
interface CharacterGridProps {
    content: CharacterGridContent;
}

const CharacterGrid: React.FC<CharacterGridProps> = ({ content }) => {
    // State is now only for this single grid
    const [foundItems, setFoundItems] = useState<number[]>([]);
    const [currentItemToFindIndex, setCurrentItemToFindIndex] = useState(0);
    const audioRef = useRef<HTMLAudioElement | null>(null);

    const currentCorrectIds = content.correctItemIds;
    const currentItemToFind = content.gridItems.find(item => item.id === currentCorrectIds[currentItemToFindIndex]);

    // Reset game when the content prop (a new grid) changes
    useEffect(() => {
        setFoundItems([]);
        setCurrentItemToFindIndex(0);
    }, [content]);

    // Autoplay the sound for the next item to find
    useEffect(() => {
        if (currentItemToFind?.audioUrl) {
            const timer = setTimeout(() => playAudio(currentItemToFind.audioUrl), 500);
            return () => clearTimeout(timer);
        }
    }, [currentItemToFind]);

    const playAudio = (audioUrl: string) => {
        if (audioRef.current) {
            audioRef.current.src = audioUrl;
            audioRef.current.play().catch(e => console.error("Audio playback failed:", e));
        }
    };

    const handleCharacterClick = (clickedItemId: number) => {
        if (foundItems.includes(clickedItemId) || !currentItemToFind) return;

        if (clickedItemId === currentItemToFind.id) {
            setFoundItems(prev => [...prev, clickedItemId]);
            setCurrentItemToFindIndex(prev => prev + 1);
        }
    };

    const isComplete = foundItems.length === currentCorrectIds.length;

    return (
        <Box p={2} sx={{ fontFamily: 'sans-serif', textAlign: 'center', height: '100%', display: 'flex', flexDirection: 'column' }}>
            <Typography variant="h5" component="h1" gutterBottom>{content.title}</Typography>
            
            <Paper elevation={2} sx={{ p: 1, mb: 2, display: 'flex', alignItems: 'center', justifyContent: 'center', gap: 2 }}>
                <Typography variant="h6">Listen:</Typography>
                <IconButton onClick={() => currentItemToFind && playAudio(currentItemToFind.audioUrl)} disabled={isComplete}>
                    <VolumeUpIcon fontSize="large" color="primary" />
                </IconButton>
            </Paper>

            <Box sx={{ flexGrow: 1 }}>
                <Grid container spacing={1} justifyContent="center" alignItems="center">
                    {content.gridItems.map(item => {
                        const isFound = foundItems.includes(item.id);
                        return (
                            <Grid item key={item.id} xs={3} sm={2}>
                                <Paper
                                    onClick={() => handleCharacterClick(item.id)}
                                    sx={{
                                        aspectRatio: '1 / 1', display: 'flex', alignItems: 'center',
                                        justifyContent: 'center', cursor: 'pointer', borderRadius: '8px',
                                        border: '2px solid', borderColor: isFound ? 'success.main' : 'grey.300',
                                        backgroundColor: isFound ? 'success.light' : 'white',
                                        transition: 'transform 0.2s', '&:hover': { transform: 'scale(1.1)' }
                                    }}
                                >
                                    <Typography variant="h3" fontWeight="bold">{item.character}</Typography>
                                </Paper>
                            </Grid>
                        );
                    })}
                </Grid>
            </Box>
            
            <Box sx={{ height: '80px', mt: 2, display: 'flex', alignItems: 'center', justifyContent: 'center' }}>
                {isComplete && (
                    <Box textAlign="center">
                        <Typography variant="h5" color="success.main">Well Done!</Typography>
                    </Box>
                )}
            </Box>

            <audio ref={audioRef} style={{ display: 'none' }} />
        </Box>
    );
};

export default CharacterGrid;
```

---

### **Component 2: `<WordPairMCQ />` (Refactored)**

This component will now render a single word-pair question. The `ActivityPlayerModal` will be responsible for passing the next question from the `questions` array in the `ContentJson`.

**File: `src/components/activities/activity-types/WordPairMCQ.tsx` (Refactored)**
```typescript
import React, { useState, useEffect, useRef } from 'react';
import { Box, Typography, Paper, Button, IconButton } from '@mui/material';
import VolumeUpIcon from '@mui/icons-material/VolumeUp';
import CheckCircleIcon from '@mui/icons-material/CheckCircle';
import CancelIcon from '@mui/icons-material/Cancel';
import ReplayIcon from '@mui/icons-material/Replay';

// --- COLOCATED TYPES ---
export interface WordPairQuestion {
    id: number;
    promptAudioUrl: string; 
    choices: [string, string];
    correctAnswer: string;
}

// This is the top-level structure.
// The ActivityPlayerModal will loop through the 'questions' array.
export interface WordPairMCQContent {
    title: string;
    questions: WordPairQuestion[];
}

// --- PROPS INTERFACE ---
// The component receives a SINGLE question object.
interface WordPairProps {
    content: WordPairQuestion;
}

const WordPairMCQ: React.FC<WordPairProps> = ({ content: question }) => {
    const [userAnswer, setUserAnswer] = useState<string | null>(null);
    const audioRef = useRef<HTMLAudioElement | null>(null);

    // Reset state and autoplay sound when the question prop changes
    useEffect(() => {
        setUserAnswer(null);
        const timer = setTimeout(() => playAudio(), 300);
        return () => clearTimeout(timer);
    }, [question]);

    const playAudio = () => {
        if (audioRef.current) {
            audioRef.current.play().catch(e => console.error(e));
        }
    };

    const handleAnswer = (choice: string) => {
        if (userAnswer) return; // Already answered
        setUserAnswer(choice);
    };

    return (
        <Box p={2} textAlign="center">
            <audio ref={audioRef} src={question.promptAudioUrl} />
            <Paper elevation={2} sx={{ p: 2, mb: 4 }}>
                <Typography variant="h6">Listen and choose the correct word:</Typography>
                <IconButton onClick={playAudio}><VolumeUpIcon fontSize="large" color="primary" /></IconButton>
            </Paper>

            <Box display="flex" justifyContent="space-around" alignItems="center">
                {question.choices.map(choice => {
                    const isSelected = userAnswer === choice;
                    const isCorrect = choice === question.correctAnswer;
                    let color: "success" | "error" | "primary" = "primary";
                    if (isSelected) {
                        color = isCorrect ? "success" : "error";
                    }

                    return (
                        <Button
                            key={choice}
                            variant={userAnswer ? 'contained' : 'outlined'}
                            color={color}
                            onClick={() => handleAnswer(choice)}
                            disabled={!!userAnswer && !isSelected}
                            sx={{ fontSize: '2rem', padding: '20px 40px', textTransform: 'none', minWidth: '180px' }}
                        >
                            {choice}
                            {isSelected && isCorrect && <CheckCircleIcon sx={{ ml: 2 }} />}
                            {isSelected && !isCorrect && <CancelIcon sx={{ ml: 2 }} />}
                        </Button>
                    );
                })}
            </Box>

            {userAnswer && !(userAnswer === question.correctAnswer) && (
                <Typography variant="h6" color="success.main" mt={3}>
                    Correct Answer: {question.correctAnswer}
                </Typography>
            )}
        </Box>
    );
};

export default WordPairMCQ;
```

### **Integration into `ActivityRenderer.tsx`**

Your renderer now has a very clear job for these components.

```typescript
// In ActivityRenderer.tsx

// ... (imports for the new components and their types)
import CharacterGrid, { CharacterGridContent } from '../activity-types/CharacterGrid';
import WordPairMCQ, { WordPairQuestion, WordPairMCQContent } from '../activity-types/WordPairMCQ';

// ...

const ActivityRenderer: React.FC<ActivityRendererProps> = ({ activityTypeId, content }) => {
    switch (activityTypeId) {
        case 2:
            // ...
            // The ActivityPlayerModal will pass a SINGLE grid object to 'content'
            if ('gridItems' in content && 'correctItemIds' in content) {
                return <CharacterGrid content={content as CharacterGridContent} />;
            }
            // ...
            break;

        case 13:
            // ...
            // The ActivityPlayerModal will pass a SINGLE question object to 'content'
            if ('promptAudioUrl' in content && 'choices' in content) {
                return <WordPairMCQ content={content as WordPairQuestion} />;
            }
            // Fallback to the original MCQ component
            return <MCQActivity content={content as MCQContent} />;
            
        // ...
    }
}
```

This completes the refactoring. Both components are now simple, stateless (in terms of sequence), and correctly follow the "dumb component, smart player" architecture.

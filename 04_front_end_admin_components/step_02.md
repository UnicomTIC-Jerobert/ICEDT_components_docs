### USER
ok this is great , Lets move to ActivityEditor Page , so we had build that page with Device Preview , but we did not implement the Logic for Eveery Activities , 
Currently we have placeholder Activity inorder to just display the JSON that we enter, 
so i think we have to create separate components for each activities and should place in ActivityEditor Page , so shall we do this by MCQ activity

### RESPONSE
Yes, absolutely. This is the final and most exciting step. We will now replace the placeholder component with a real, interactive `MCQActivity` component.

This will bring your `ActivityEditor` to life, providing instant, tangible feedback to the admin as they create the content.

We'll follow these steps:
1.  **Define the TypeScript Type** for the `MCQ` `ContentJson`. This is crucial for type safety.
2.  **Create the `<MCQActivity />` Component:** This will be the actual UI component that a student would see in the Flutter app.
3.  **Integrate it into the `DevicePreview`:** We'll replace the placeholder by adding a new `case` in the `switch` statement.

---

### **Step 1: Define the `ContentJson` Type for MCQ**

First, let's establish a clear and strict structure for what the JSON of an MCQ activity should look like. This contract is essential for both the backend and frontend.

**File: `src/types/activityContentTypes.ts` (Create this new file)**
```typescript
// This file will hold the interfaces for all 18 activity content types.

export interface MCQChoice {
    id: string | number; // A unique identifier for the choice
    text: string;
    isCorrect: boolean;
}

export interface MCQContent {
    question: string;
    choices: MCQChoice[];
    // You could add optional properties later, like an image URL for the question
    // imageUrl?: string;
}
```
This interface provides strong typing, so you'll get autocomplete and compile-time errors if you try to access a property that doesn't exist (e.g., `choice.is_correct` instead of `choice.isCorrect`).

---

### **Step 2: Create the `MCQActivity` Component**

This is the UI component that renders the MCQ. It will be a self-contained, interactive component.

**File: `src/components/activities/activity-types/MCQActivity.tsx` (Create this new folder and file)**
```typescript
import React, { useState } from 'react';
import { Box, Typography, Button, Paper } from '@mui/material';
import { MCQContent, MCQChoice } from '../../../types/activityContentTypes';
import CheckCircleIcon from '@mui/icons-material/CheckCircle';
import CancelIcon from '@mui/icons-material/Cancel';

interface MCQActivityProps {
    content: MCQContent;
}

const MCQActivity: React.FC<MCQActivityProps> = ({ content }) => {
    const [selectedChoiceId, setSelectedChoiceId] = useState<string | number | null>(null);
    const [isAnswered, setIsAnswered] = useState<boolean>(false);

    const handleChoiceClick = (choice: MCQChoice) => {
        if (isAnswered) return; // Prevent changing answer
        setSelectedChoiceId(choice.id);
        setIsAnswered(true);
    };
    
    // Helper to determine the button color after an answer is given
    const getButtonVariant = (choice: MCQChoice): "contained" | "outlined" => {
        if (!isAnswered) return 'outlined';
        if (choice.isCorrect) return 'contained'; // Always highlight the correct answer
        if (selectedChoiceId === choice.id && !choice.isCorrect) return 'contained'; // Highlight the user's wrong choice
        return 'outlined';
    };
    
    const getButtonColor = (choice: MCQChoice): "success" | "error" | "primary" => {
        if (!isAnswered) return 'primary';
        if (choice.isCorrect) return 'success';
        if (selectedChoiceId === choice.id && !choice.isCorrect) return 'error';
        return 'primary';
    };

    return (
        <Box p={3} sx={{ fontFamily: 'sans-serif' }}>
            <Paper elevation={2} sx={{ p: 2, mb: 3 }}>
                <Typography variant="h6" component="h2" textAlign="center">
                    {content.question}
                </Typography>
            </Paper>

            <Box display="flex" flexDirection="column" gap={2}>
                {content.choices.map((choice) => (
                    <Button
                        key={choice.id}
                        variant={getButtonVariant(choice)}
                        color={getButtonColor(choice)}
                        onClick={() => handleChoiceClick(choice)}
                        sx={{ 
                            justifyContent: 'space-between', 
                            textAlign: 'left',
                            padding: '12px 16px',
                            textTransform: 'none' // Keep text normal case
                        }}
                        fullWidth
                    >
                        {choice.text}
                        {isAnswered && choice.isCorrect && <CheckCircleIcon sx={{ ml: 1 }} />}
                        {isAnswered && selectedChoiceId === choice.id && !choice.isCorrect && <CancelIcon sx={{ ml: 1 }} />}
                    </Button>
                ))}
            </Box>
            
            {isAnswered && (
                <Box mt={3} display="flex" justifyContent="center">
                    <Button variant="contained" onClick={() => { setIsAnswered(false); setSelectedChoiceId(null); }}>
                        Try Again
                    </Button>
                </Box>
            )}
        </Box>
    );
};

export default MCQActivity;
```

### **Step 3: Integrate into `DevicePreview.tsx`**

Now, we replace the placeholder with our new, real component.

**File: `src/components/activities/DevicePreview.tsx` (Modified)**
```typescript
// ... (imports)
import { Activity } from '../../types/activity';
// --- IMPORT THE NEW COMPONENT ---
import MCQActivity from './activity-types/MCQActivity'; 
import { MCQContent } from '../../types/activityContentTypes';

// --- Remove or keep the placeholder for other types ---
const PlaceholderActivity = ({ type, content }: { type: string, content: any }) => (
    // ...
);
const MatchingActivity = ({ content }: { content: any }) => <PlaceholderActivity type="Matching" content={content} />;
// ...

interface DevicePreviewProps {
    activityData: Partial<Activity>;
}

const DevicePreview: React.FC<DevicePreviewProps> = ({ activityData }) => {
    // ... (useState for device, handleDeviceChange function are the same)

    const renderActivityComponent = () => {
        if (!activityData.activityTypeId || !activityData.contentJson) {
            return <Typography p={2}>Please select an activity type and provide JSON content.</Typography>;
        }
        
        let content: any;
        try {
            content = JSON.parse(activityData.contentJson);
        } catch (e) {
            return <Typography p={2} color="error">Invalid JSON format. Preview is paused.</Typography>;
        }

        // *** THE INTEGRATION HAPPENS HERE ***
        switch (activityData.activityTypeId) {
            case 4: // Matching (Still a placeholder)
                 return <MatchingActivity content={content} />;
                 
            case 13: // MultipleChoiceQuestion (ID from your DB seeder)
                 // Type assertion tells TypeScript to trust us that the content matches the MCQContent interface
                 return <MCQActivity content={content as MCQContent} />;

            default:
                return <Typography p={2}>Preview for this activity type is not yet implemented.</Typography>;
        }
    };
    
    // ... (The rest of the component with the device frame JSX is the same)
};

export default DevicePreview;
```

---

### **How to Use It**

1.  **Run your application** (backend and frontend).
2.  Navigate to the `ActivityEditorPage`.
3.  Select **"MultipleChoiceQuestion"** from the "Activity Type" dropdown (assuming its ID is 13).
4.  Paste the following valid JSON into the `ContentJson` textarea:

    ```json
    {
      "question": "Which of these is a fruit?",
      "choices": [
        { "id": 1, "text": "Carrot", "isCorrect": false },
        { "id": 2, "text": "Apple", "isCorrect": true },
        { "id": 3, "text": "Broccoli", "isCorrect": false }
      ]
    }
    ```
### USER
its work fine for one question , will it work for more than one , so 

### RESPONSE
That is a superb and critical question. You've correctly identified that real lessons aren't just one activity; they are a **sequence** of activities.

The architecture we have built is **perfectly designed to handle this**, but it requires a clear understanding of the roles of the `ActivityEditor` vs. the `ActivityPlayer` in the final Flutter app.

Let's clarify the workflow.

---

### **Clarifying the Roles: Editor vs. Player**

1.  **`ActivityEditorPage` (What you are building now in the Admin Panel):**
    *   **Purpose:** To create or edit **ONE SINGLE** activity at a time.
    *   **Scope:** It focuses on the details of a single row in your `Activities` database table. It manages the `title`, `sequenceOrder`, `activityTypeId`, and the `contentJson` for *that one activity*.
    *   **"Next/Previous" buttons:** It does **not** have "Next/Previous" buttons for navigating between activities. Its job is to perfect one activity and then save it. The navigation back to the list page (`ActivitiesListPage`) is its "exit point."

2.  **`ActivityPlayerScreen` (What you will build in the Flutter App):**
    *   **Purpose:** To present a **SEQUENCE** of activities to the student for a given lesson.
    *   **Scope:** It will fetch the *entire list* of activities for a lesson (e.g., the 5 activities for "பாடம் 01").
    *   **"Next/Previous" buttons:** This is where the navigation logic will live. It will manage a `currentActivityIndex` and will have "Next" and "Previous" buttons to move through the array of activities.

**In short: The Admin Panel builds the individual Lego bricks. The Flutter App assembles them in order for the student to play with.**

---

### **How the Flutter App Will Handle Multiple Questions**

Let's walk through the exact logic your Flutter app will use. This will reassure you that the current backend and admin panel design is correct.

**1. Data Fetching in Flutter:**
When a student starts a lesson, the Flutter app will make a single API call to the endpoint we designed:
`GET /api/progress/current-lesson`

It will receive a JSON response containing the lesson details and an **array of all its activities**:
```json
{
  "lessonId": 1,
  "lessonName": "...",
  "activities": [
    { "activityId": 101, "activityTypeId": 13, "contentJson": "{...}" }, // Question 1
    { "activityId": 102, "activityTypeId": 4,  "contentJson": "{...}" }, // Question 2
    { "activityId": 103, "activityTypeId": 7,  "contentJson": "{...}" }  // Question 3
  ]
}
```

**2. State Management in the Flutter `ActivityPlayerScreen`:**

The Flutter screen will hold this state:
```dart
class ActivityPlayerScreen extends StatefulWidget {
  // ...
}

class _ActivityPlayerScreenState extends State<ActivityPlayerScreen> {
  List<Activity> activities = []; // The full list from the API
  int currentActivityIndex = 0;   // The index of the activity to show
  
  // ... initState() to fetch data from the API ...
}
```

**3. The `build` Method (Rendering Logic):**

The `build` method will be responsible for displaying the UI.

```dart
@override
Widget build(BuildContext context) {
  // Get the current activity based on the index
  final currentActivity = activities[currentActivityIndex];
  
  return Scaffold(
    appBar: AppBar(title: Text("Lesson Name")),
    body: Column(
      children: [
        // This is where the dynamic component rendering happens
        Expanded(
          child: buildActivityWidget(currentActivity),
        ),
        
        // Navigation controls
        Row(
          mainAxisAlignment: MainAxisAlignment.spaceAround,
          children: [
            ElevatedButton(
              onPressed: currentActivityIndex > 0 
                ? () => setState(() => currentActivityIndex--) 
                : null, // Disable "Prev" on the first question
              child: Text("Previous"),
            ),
            ElevatedButton(
              onPressed: currentActivityIndex < activities.length - 1
                ? () => setState(() => currentActivityIndex++)
                : null, // Disable "Next" on the last question
              child: Text("Next"),
            ),
          ],
        ),
      ],
    ),
  );
}
```

**4. The `buildActivityWidget` Helper:**

This is the Dart equivalent of your React `renderActivityComponent` function.

```dart
Widget buildActivityWidget(Activity activity) {
  // Parse the contentJson string
  final content = jsonDecode(activity.contentJson);
  
  switch (activity.activityTypeId) {
    case 13: // MCQ
      return MCQActivityWidget(content: content);
    case 4: // Matching
      return MatchingActivityWidget(content: content);
    // ... cases for all 18 types
    default:
      return Text("Unsupported activity type");
  }
}
```

### **Conclusion: Yes, It Works Perfectly for Multiple Questions**

The system you are building is designed from the ground up to handle a sequence of multiple, different activities.

*   Your **Admin Panel** is correctly focused on the **atomic unit:** creating and editing one activity perfectly.
*   Your **API** is correctly focused on serving the **entire sequence** of activities for a lesson.
*   Your future **Flutter App** will be responsible for consuming that sequence and providing the **navigation** (Next/Previous) for the student.

You are on exactly the right track. Continue building out the individual activity components for the `DevicePreview` in the admin panel. Each one you complete is a component that you can then easily recreate in Flutter, knowing that the data structure and logic are already proven to work.

### USER
so , then why cant we add this feature in Admin Panel As well , bcz we can test this here as well and mean time the content JSON For MCQ will look some thing like this for more questions
```
[
{
  "question": "Which of these is a fruit?",
  "choices": [
    { "id": 1, "text": "Carrot", "isCorrect": false },
    { "id": 2, "text": "Apple", "isCorrect": true },
    { "id": 3, "text": "Broccoli", "isCorrect": false }
  ]
},
{
  "question": "Which of these is a vegetable?",
  "choices": [
    { "id": 1, "text": "Carrot", "isCorrect": false },
    { "id": 2, "text": "Apple", "isCorrect": true },
    { "id": 3, "text": "Broccoli", "isCorrect": false }
  ]
}

]
```

### RESPONSE
That is a very sharp observation, and it brings us to a fundamental architectural decision about the **granularity of an "Activity."**

You've proposed a different model, and it's important to analyze its pros and cons.

Let's compare the two approaches.

---

### **Approach A: "One Activity, One Interaction" (Our Current Model)**

*   **Definition:** An `Activity` in the database represents a single, atomic interaction. One MCQ question is one `Activity`. One matching grid is one `Activity`. A sequence of 5 questions would be 5 separate rows in the `Activities` table.
*   **`ContentJson` for MCQ:** Contains a single question object.
    ```json
    {
      "question": "Which of these is a fruit?",
      "choices": [ ... ]
    }
    ```
*   **How the Player Works:** The Flutter `ActivityPlayer` fetches a *list of Activities* and navigates between them using its `currentActivityIndex`. The "Next" button increments the index to show the next `Activity` row from the database.

### **Approach B: "One Activity, Multiple Interactions" (Your Proposed Model)**

*   **Definition:** An `Activity` in the database represents a *collection* or a *quiz* of interactions. A single "MCQ Quiz" `Activity` could contain 5 questions.
*   **`ContentJson` for MCQ:** Contains an **array** of question objects.
    ```json
    [
      { "question": "Which of these is a fruit?", "choices": [...] },
      { "question": "Which is the largest planet?", "choices": [...] }
    ]
    ```
*   **How the Player Works:** The Flutter `ActivityPlayer` would fetch a list of activities. When it gets to the "MCQ Quiz" activity, the `MCQActivity` component *itself* would need to have its own internal state (`currentQuestionIndex`) and its own "Next/Previous" buttons to navigate through the questions *inside its own JSON*.

---

### **Comparison of the Two Approaches**

| Feature | Approach A (One Activity, One Interaction) | Approach B (One Activity, Multiple Interactions) | **Recommendation & Analysis** |
| :--- | :--- | :--- | :--- |
| **Simplicity** | **Simpler.** The data model is flat. Each activity component (like `<MCQActivity />`) is very "dumb"—it just renders what it's given and doesn't need internal navigation. | **More Complex.** Activity components now need their own internal state management for sequencing (e.g., `currentQuestionIndex`). The `ContentJson` structure becomes more complex (sometimes an object, sometimes an array). | 🏆 **Approach A is simpler to implement and maintain.** |
| **Flexibility** | **Highly Flexible.** You can easily mix and match activity types. A lesson can be: MCQ -> Matching -> FillInTheBlanks -> MCQ. This is very powerful for creating varied learning experiences. | **Less Flexible.** A lesson would be: "MCQ Quiz" -> "Matching Grid". You can't easily insert a Matching activity *in between* two MCQ questions without creating a new "MCQ Quiz" activity. | 🏆 **Approach A provides far greater flexibility for the course creator.** |
| **Data Tracking & Analytics**| **Granular.** You can track a student's progress on *every single question*. The `UserProgress` table will have a row for each interaction. You can easily find out which specific questions are the hardest for students. | **Coarse.** You would only know that the user completed the "MCQ Quiz" activity. Tracking performance on individual questions within the JSON is much harder and would require a more complex data model. | 🏆 **Approach A is vastly superior for analytics.** |
| **Admin Experience** | **Clear and Atomic.** The admin creates one interaction at a time. The form is always for a single question. This is simple and less error-prone. | **Potentially Complex.** The admin has to manage a potentially large and complex JSON array in a single textarea. A single syntax error could invalidate the entire quiz. | 🏆 **Approach A provides a better and safer admin experience.** |
| **Adding a Preview Feature in Admin** | **Difficult.** As you noted, previewing the *flow* between activities is hard because the editor only knows about one activity at a time. | **Easier.** The `DevicePreview` for an "MCQ Quiz" could naturally include "Next/Prev" buttons to cycle through the questions within its own JSON. | 🏆 **Approach B makes the admin *preview* feature more powerful.** |

---

### **Conclusion and The Recommended Path Forward**

While your proposed **Approach B** makes the *admin preview* feature more intuitive, **Approach A (our current model) is architecturally superior for the final application in almost every other way**: flexibility, data tracking, and simplicity.

**So, what do we do? We can get the best of both worlds.**

**Recommendation:**
1.  **Stick with Architecture A (One Activity, One Interaction).** The benefits of flexibility and granular analytics are too significant to give up. This is the correct long-term design for the application's data model.

2.  **Enhance the Admin Preview to Simulate the Flow.** We can absolutely add a "preview flow" feature to the admin panel, but we'll do it smartly.

**How to Implement a "Flow Preview" in the Admin Panel:**

On your **`ActivitiesListPage`** (the one that shows the table of all activities for a lesson), add a new button:

```html
<!-- In ActivitiesListPage.tsx -->
<Button variant="contained" color="secondary" startIcon={<PlayArrowIcon />}>
    Preview Lesson Flow
</Button>
```

When an admin clicks this button, it would open a modal window or a new page that acts as a **mini "Activity Player,"** just like the one we designed for Flutter.

*   It would fetch the **full list of activities** for that lesson.
*   It would display the first activity.
*   It would have **"Next" and "Previous" buttons** that cycle through the activities in the list.

**Why this is the best solution:**
*   You keep your robust and flexible "One Activity, One Interaction" data model.
*   You still give the content creator the ability to test the entire sequence of a lesson exactly as a student would experience it.
*   Your `ActivityEditor` page remains simple and focused on its job: editing a single, atomic activity.

So, to answer your original question:
> "why cant we add this feature in Admin Panel As well"

We absolutely can, and we should! But we should implement it as a **"Lesson Preview"** on the list page, not by changing the fundamental data model to make a single `Activity` responsible for holding multiple questions. This keeps our architecture clean and our application powerful.

### USER
lets keep this aside , and lets build matching activity componnet , after words we will create word scrambled based  component , lets test this analyse and came back to our conclusion

### RESPONSE
Excellent idea. Let's focus on building the next interactive component. Deferring the final decision on the "flow preview" is smart; building out more of the individual components will give us a better feel for the system's needs.

The `MatchingActivity` is a fantastic next step because it's highly interactive and visually different from the MCQ component.

We'll follow the same successful pattern:
1.  **Define the TypeScript Type** for the `Matching` `ContentJson`.
2.  **Create the `<MatchingActivity />` Component** with drag-and-drop or line-drawing logic.
3.  **Integrate it into the `DevicePreview`**.

---

### **Step 1: Define the `ContentJson` Type for Matching**

The JSON needs to define the pairs of items that need to be matched. A good structure is an array of pairs.

**File: `src/types/activityContentTypes.ts` (Add to this file)**
```typescript
// ... (existing MCQ types)

// --- NEW: Types for Matching Activity ---
export interface MatchItem {
    id: string; // Unique ID for this item (e.g., 'A', 'B', '1', '2')
    content: string; // The text to display
    matchId: string; // The ID of the item it should be paired with
}

export interface MatchingContent {
    columnA: MatchItem[]; // Items for the left column
    columnB: MatchItem[]; // Items for the right column
    // Optional title for the activity
    title?: string;
}
```
**Example JSON based on this type:**
```json
{
  "title": "Match the Animal to its Sound",
  "columnA": [
    { "id": "A1", "content": "Dog", "matchId": "B1" },
    { "id": "A2", "content": "Cat", "matchId": "B2" },
    { "id": "A3", "content": "Lion", "matchId": "B3" }
  ],
  "columnB": [
    { "id": "B1", "content": "Barks", "matchId": "A1" },
    { "id": "B2", "content": "Meows", "matchId": "A2" },
    { "id": "B3", "content": "Roars", "matchId": "A3" }
  ]
}
```
This structure is very flexible and clear.

---

### **Step 2: Create the `MatchingActivity` Component**

This is the most complex part. We need to render the two columns and handle the user interaction for creating connections. A simple and effective way to do this on the web is by clicking an item from each column.

**File: `src/components/activities/activity-types/MatchingActivity.tsx` (Create this new file)**
```typescript
import React, { useState, useEffect } from 'react';
import { Box, Typography, Paper, Button, Grid } from '@mui/material';
import CheckCircleIcon from '@mui/icons-material/CheckCircle';
import ReplayIcon from '@mui/icons-material/Replay';
import { MatchingContent, MatchItem } from '../../../types/activityContentTypes';

interface MatchingActivityProps {
    content: MatchingContent;
}

const MatchingActivity: React.FC<MatchingActivityProps> = ({ content }) => {
    const [selectedA, setSelectedA] = useState<MatchItem | null>(null);
    const [selectedB, setSelectedB] = useState<MatchItem | null>(null);
    const [correctPairs, setCorrectPairs] = useState<string[]>([]); // Stores IDs of correctly matched items

    // This effect checks for a match whenever the user selects one item from each column
    useEffect(() => {
        if (selectedA && selectedB) {
            if (selectedA.matchId === selectedB.id) {
                // Correct match!
                setCorrectPairs(prev => [...prev, selectedA.id, selectedB.id]);
            }
            // Reset selections after a short delay to give user feedback
            setTimeout(() => {
                setSelectedA(null);
                setSelectedB(null);
            }, 300);
        }
    }, [selectedA, selectedB]);

    const handleItemClick = (item: MatchItem, column: 'A' | 'B') => {
        // Don't allow clicking an already matched item
        if (correctPairs.includes(item.id)) return;

        if (column === 'A') {
            setSelectedA(item);
        } else {
            setSelectedB(item);
        }
    };
    
    const isComplete = correctPairs.length === content.columnA.length + content.columnB.length;

    const handleReset = () => {
        setSelectedA(null);
        setSelectedB(null);
        setCorrectPairs([]);
    };

    // Helper function to render a single item button
    const renderItem = (item: MatchItem, column: 'A' | 'B', isSelected: boolean) => {
        const isCorrect = correctPairs.includes(item.id);
        
        let variant: "contained" | "outlined" = "outlined";
        if (isCorrect) variant = "contained";
        else if (isSelected) variant = "contained";
        
        let color: "success" | "primary" = "primary";
        if(isCorrect) color = "success";

        return (
            <Button
                key={item.id}
                variant={variant}
                color={color}
                onClick={() => handleItemClick(item, column)}
                disabled={isCorrect}
                fullWidth
                sx={{ 
                    my: 1, 
                    height: '60px', 
                    textTransform: 'none',
                    justifyContent: 'center',
                    fontSize: '1rem',
                }}
            >
                {item.content}
            </Button>
        );
    }

    return (
        <Box p={2} sx={{ fontFamily: 'sans-serif' }}>
            {content.title && (
                <Typography variant="h6" textAlign="center" mb={3}>{content.title}</Typography>
            )}

            <Grid container spacing={4} justifyContent="center" alignItems="center">
                {/* Column A */}
                <Grid item xs={5}>
                    {content.columnA.map(item => renderItem(item, 'A', selectedA?.id === item.id))}
                </Grid>
                {/* Column B */}
                <Grid item xs={5}>
                    {content.columnB.map(item => renderItem(item, 'B', selectedB?.id === item.id))}
                </Grid>
            </Grid>

            {isComplete && (
                <Box textAlign="center" mt={4}>
                    <Typography variant="h5" color="success.main" sx={{ display: 'flex', alignItems: 'center', justifyContent: 'center', gap: 1 }}>
                        <CheckCircleIcon /> All Matched!
                    </Typography>
                </Box>
            )}
            
             <Box textAlign="center" mt={4}>
                <Button variant="outlined" startIcon={<ReplayIcon />} onClick={handleReset}>
                    Reset
                </Button>
            </Box>
        </Box>
    );
};

export default MatchingActivity;
```

### **Step 3: Integrate into `DevicePreview.tsx`**

Finally, we'll add this new component to our previewer's `switch` statement.

**File: `src/components/activities/DevicePreview.tsx` (Modified)**
```typescript
// ... (imports)
// --- IMPORT THE NEW COMPONENT AND TYPE ---
import MatchingActivity from './activity-types/MatchingActivity';
import { MatchingContent } from '../../types/activityContentTypes';
// ...

const DevicePreview: React.FC<DevicePreviewProps> = ({ activityData }) => {
    // ... (logic is the same)

    const renderActivityComponent = () => {
        // ... (JSON parsing and error handling is the same)
        
        // *** THE INTEGRATION HAPPENS HERE ***
        switch (activityData.activityTypeId) {
            case 4: // Matching (ID from your DB seeder)
                 return <MatchingActivity content={content as MatchingContent} />;
                 
            case 13: // MultipleChoiceQuestion
                 return <MCQActivity content={content as MCQContent} />;

            default:
                return <Typography p={2}>Preview for this activity type is not yet implemented.</Typography>;
        }
    };
    
    // ... (rest of the component is the same)
};

export default DevicePreview;
```

### **How to Use It**

1.  **Run your application**.
2.  Navigate to the `ActivityEditorPage`.
3.  Select **"Matching"** from the "Activity Type" dropdown (assuming its ID is 4).
4.  Paste the following valid JSON into the `ContentJson` textarea:

    ```json
    {
      "title": "Match the Country to its Capital",
      "columnA": [
        { "id": "A1", "content": "France", "matchId": "B1" },
        { "id": "A2", "content": "Japan", "matchId": "B2" },
        { "id": "A3", "content": "Canada", "matchId": "B3" }
      ],
      "columnB": [
        { "id": "B1", "content": "Paris", "matchId": "A1" },
        { "id": "B2", "content": "Tokyo", "matchId": "A2" },
        { "id": "B3", "content": "Ottawa", "matchId": "A3" }
      ]
    }
    ```

**Result:** The preview pane on the right will instantly update to show two columns of buttons. You can click one from the left and one from the right. If they are a correct pair, they will both turn green and become disabled. If they are incorrect, they will flash and reset. This provides immediate, interactive feedback for the content creator.

We've now successfully implemented a second, more complex activity type. The pattern is proving to be very effective. Ready for the Word Scramble next?



**Result:** As soon as you finish typing, the `DevicePreview` on the right will **instantly** update. It will no longer show a placeholder; it will show your fully interactive MCQ component. You can click the buttons and see the "Correct/Incorrect" feedback, demonstrating the real-time preview in action.

You have now successfully built the first live component for your activity editor. You can repeat this pattern (Define Type -> Create Component -> Integrate into Preview) for all the other 17 activity types.

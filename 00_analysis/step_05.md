### USER :
here we had found out 18 kinds of activities , so can you list all of these in both english and tamil , after that we  will then generate Insert query for activities table as well

### RESPONSE :
Of course. This is the perfect next step. Having a definitive list of activity types is essential before populating the database.

Here is the complete list of the 18 activity types in both English and Tamil, followed by the ready-to-use SQL `INSERT` script for your `ActivityTypes` table.

---

### **List of 18 Activity Types (English & Tamil)**

This table provides a clear reference for both developers (English Name, for enums/code) and content creators (Tamil Name, for display).

| ActivityTypeId | English Name (for Code) | Tamil Name (for Display) | Description |
| :--- | :--- | :--- | :--- |
| **Foundational Skills** |
| 1 | `PronunciationPractice` | ஒலித்துப்பழகுவோம் | Listen and repeat sounds, letters, and words. |
| 2 | `AudioImageRecognition` | ஒலி-பட அடையாளம் | Match a spoken word or sound to the correct image. |
| 3 | `Dictation` | சொல்லக் கேட்டு எழுதுதல் | Type a word or sentence after hearing it. |
| **Vocabulary & Association** |
| 4 | `Matching` | பொருத்துதல் | Connect related items (e.g., word-synonym, image-word). |
| 5 | `SortingAndClassification`| வகைப்படுத்துதல் | Drag items into their correct grammatical or thematic groups. |
| 6 | `OddOneOut` | பொருந்தாததை நீக்கு | Identify the item that doesn't belong in a set. |
| **Word & Sentence Structure** |
| 7 | `FillInTheBlanks` | இடைவெளியை நிரப்புதல் | Complete a word or sentence by filling in a missing part. |
| 8 | `WordScramble` | எழுத்துக்களை ஒழுங்குபடுத்துதல்| Unscramble letters to form a correct word. |
| 9 | `SentenceScramble` | சொற்களை ஒழுங்குபடுத்துதல் | Unscramble words to form a correct sentence. |
| 10 | `WordFormation` | சொல் உருவாக்கம் | Join or split words based on grammar rules. |
| 11 | `SentenceBuilding` | வாக்கியம் அமைத்தல் | Construct logical sentences from a bank of words. |
| 12 | `GrammarPuzzle` | இலக்கணப் புதிர் | Solve riddles based on grammar or vocabulary knowledge. |
| **Comprehension & Application** |
| 13 | `MultipleChoiceQuestion` | பலவுள் தெரிவு | Select the single correct answer from multiple options. |
| 14 | `TrueOrFalse` | சரி / பிழை | Validate whether a statement is true or false. |
| 15 | `ReadingComprehension` | வாசித்துப் புரிதல் | Read a passage and answer questions about it. |
| 16 | `StorySequencing` | கதை வரிசைப்படுத்தல் | Arrange story panels or sentences in chronological order. |
| **Gamification & Engagement** |
| 17 | `TimedChallenge` | நேர சவால் | A feature to add a timer to any other activity. |
| 18 | `InteractiveDialogue` | ஊடாடும் உரையாடல் | A reading passage presented as an interactive chat. |

---

### **SQL `INSERT` Script for `ActivityTypes` Table**

Here is the T-SQL script to populate your `ActivityTypes` table. This script is **idempotent**, meaning it's safe to run multiple times; it will only insert the data if the table is empty, preventing duplicates.

You can run this directly in SQL Server Management Studio (SSMS).

```sql
-- ==============================================================
--      SCRIPT TO SEED THE 'ActivityTypes' TABLE
-- ==============================================================
-- This script will populate the ActivityTypes table with the 18
-- defined activity types for the Tamil Learning App.

-- Check if the table already has data to prevent duplicates.
IF NOT EXISTS (SELECT 1 FROM ActivityTypes)
BEGIN
    PRINT 'Seeding ActivityTypes table...';

    -- Using IDENTITY_INSERT to ensure the IDs match the list above,
    -- which makes development and debugging easier.
    SET IDENTITY_INSERT ActivityTypes ON;

    INSERT INTO ActivityTypes (ActivityTypeId, ActivityName)
    VALUES
        -- Foundational Skills
        (1,  'PronunciationPractice'),
        (2,  'AudioImageRecognition'),
        (3,  'Dictation'),
        -- Vocabulary & Association
        (4,  'Matching'),
        (5,  'SortingAndClassification'),
        (6,  'OddOneOut'),
        -- Word & Sentence Structure
        (7,  'FillInTheBlanks'),
        (8,  'WordScramble'),
        (9,  'SentenceScramble'),
        (10, 'WordFormation'),
        (11, 'SentenceBuilding'),
        (12, 'GrammarPuzzle'),
        -- Comprehension & Application
        (13, 'MultipleChoiceQuestion'),
        (14, 'TrueOrFalse'),
        (15, 'ReadingComprehension'),
        (16, 'StorySequencing'),
        -- Gamification & Engagement
        (17, 'TimedChallenge'),
        (18, 'InteractiveDialogue');

    SET IDENTITY_INSERT ActivityTypes OFF;

    PRINT 'Seeding of ActivityTypes table completed successfully.';
END
ELSE
BEGIN
    PRINT 'ActivityTypes table already contains data. Seeding skipped.';
END
GO
```

With this list and the corresponding SQL script, your database is now perfectly set up to categorize and manage all the different learning activities your app will offer.

### USER :
great now we had generated the activity types , then shall we proceed to activities , currently at this moment we are i dont need the content json , 
but we fill fill other fields , is it ok to do one by one one so lets start from malalaiyar nilai

### RESPONSE :
Yes, that is the perfect strategy.

Populating the `Activities` table is the most content-intensive part of this project. Tackling it one level at a time, starting with `மழலையர் நிலை`, is the most organized and manageable way to proceed.

We will focus on getting the structure right—linking each activity to the correct lesson and activity type—and use placeholder JSON for now. This allows you to build and test the entire API and database structure without getting bogged down in creating the detailed JSON content for every single activity at once.

### **Plan of Action**

1.  **Map `மழலையர் நிலை` Activities:** We will go through each lesson in the `மழலையர் நிலை` document and map its exercises to our 18 activity types.
2.  **Generate SQL Script:** I will provide the `INSERT` script for the `Activities` table for this entire level.
3.  **Placeholder `ContentJson`:** For now, the `ContentJson` will be a simple placeholder like `'{"status": "pending"}'`. This field is `NOT NULL`, so it requires a value. This placeholder fulfills that requirement.

Let's begin.

---

### **Mapping Activities for `மழலையர் நிலை` (Preschool Level)**

First, we need the `LevelId` and `LessonId`s for this level. Let's assume after seeding, `மழலையர் நிலை` has `LevelId = 1`, and its lessons will have IDs starting from 1.

| Lesson Name | Assumed LessonId | Mapped Activities |
| :--- | :--- | :--- |
| **பாடம் 01: உடல் உறுப்புகள்** | 1 | 1. **Audio-Image Recog.** (`ActivityTypeId=2`): Match body part names to images. |
| **பாடம் 02: எனது குடும்பம்** | 2 | 1. **Audio-Image Recog.** (`ActivityTypeId=2`): Match family member names to images. |
| **பாடம் 03: எனது வீடு** | 3 | 1. **Audio-Image Recog.** (`ActivityTypeId=2`): Match house/room names to images. |
| **பாடம் 04: உணவுகள்** | 4 | 1. **Audio-Image Recog.** (`ActivityTypeId=2`): Food items.<br>2. **Sorting/Classification** (`ActivityTypeId=5`): Sort items by taste (இனிப்பு, புளிப்பு). |
| **பாடம் 05: வண்ணங்கள்** | 5 | 1. **Matching** (`ActivityTypeId=4`): Match color names to color swatches. |
| **பாடம் 06: பூக்கள்** | 6 | 1. **Audio-Image Recog.** (`ActivityTypeId=2`): Flower names.<br>2. **Audio-Image Recog.** (`ActivityTypeId=2`): Related items (தேன், வண்டு). |
| **பாடம் 07: பறவைகள்** | 7 | 1. **Audio-Image Recog.** (`ActivityTypeId=2`): Bird names.<br>2. **Matching** (`ActivityTypeId=4`): Match bird to its feature (e.g., மயில் -> தோகை). |
| **பாடம் 08: விலங்குகள்** | 8 | 1. **Audio-Image Recog.** (`ActivityTypeId=2`): Animal names.<br>2. **Matching** (`ActivityTypeId=4`): Match animal to its product (e.g., மாடு -> பால்). |
| **பாடம் 09: விளையாட்டுகள்**| 9 | 1. **Audio-Image Recog.** (`ActivityTypeId=2`): Different games/activities. |
| **பாடம் 10: கொண்டாட்டம்** | 10 | 1. **Audio-Image Recog.** (`ActivityTypeId=2`): Festival-related items. |
| **மீட்டற் பயிற்சிகள்** | 10 | 2. **Pronunciation Practice** (`ActivityTypeId=1`): Recite all vowels. |

*Note: The Preschool level is very visual and recognition-based, so most activities fall into the `AudioImageRecognition` and `Matching` types.*

---

### **SQL `INSERT` Script for `மழலையர் நிலை` Activities**

This script will populate the `Activities` table for the entire Preschool level. It dynamically fetches the `LevelId` and `LessonId`s, making it robust and safe to run.

```sql
-- ===================================================================
-- SCRIPT TO SEED 'Activities' TABLE FOR 'மழலையர் நிலை' (Preschool)
-- ===================================================================

BEGIN TRANSACTION;

BEGIN TRY
    -- Step 1: Declare variables to hold the IDs we need.
    DECLARE @LevelId INT;
    DECLARE @Lesson1Id INT, @Lesson2Id INT, @Lesson3Id INT, @Lesson4Id INT, @Lesson5Id INT;
    DECLARE @Lesson6Id INT, @Lesson7Id INT, @Lesson8Id INT, @Lesson9Id INT, @Lesson10Id INT;

    -- Step 2: Get the LevelId for 'மழலையர் நிலை'.
    SELECT @LevelId = LevelId FROM Levels WHERE LevelName = N'மழலையர் நிலை';

    IF @LevelId IS NULL
    BEGIN
        -- If the level doesn't exist, we can't proceed.
        PRINT 'Error: Level "மழலையர் நிலை" not found. Seeding cannot continue.';
        ROLLBACK TRANSACTION;
        RETURN;
    END

    -- Step 3: Get the LessonIds for this level.
    -- This makes the script independent of the exact auto-incremented ID values.
    SELECT @Lesson1Id = LessonId FROM Lessons WHERE LevelId = @LevelId AND SequenceOrder = 1;
    SELECT @Lesson2Id = LessonId FROM Lessons WHERE LevelId = @LevelId AND SequenceOrder = 2;
    SELECT @Lesson3Id = LessonId FROM Lessons WHERE LevelId = @LevelId AND SequenceOrder = 3;
    SELECT @Lesson4Id = LessonId FROM Lessons WHERE LevelId = @LevelId AND SequenceOrder = 4;
    SELECT @Lesson5Id = LessonId FROM Lessons WHERE LevelId = @LevelId AND SequenceOrder = 5;
    SELECT @Lesson6Id = LessonId FROM Lessons WHERE LevelId = @LevelId AND SequenceOrder = 6;
    SELECT @Lesson7Id = LessonId FROM Lessons WHERE LevelId = @LevelId AND SequenceOrder = 7;
    SELECT @Lesson8Id = LessonId FROM Lessons WHERE LevelId = @LevelId AND SequenceOrder = 8;
    SELECT @Lesson9Id = LessonId FROM Lessons WHERE LevelId = @LevelId AND SequenceOrder = 9;
    SELECT @Lesson10Id = LessonId FROM Lessons WHERE LevelId = @LevelId AND SequenceOrder = 10;
    
    PRINT 'Seeding activities for "மழலையர் நிலை"...';

    -- Use a temporary table to hold the data before inserting. This is cleaner.
    DECLARE @ActivitiesToSeed TABLE (
        LessonId INT,
        ActivityTypeId INT,
        Title NVARCHAR(250),
        SequenceOrder INT,
        ContentJson NVARCHAR(MAX)
    );

    INSERT INTO @ActivitiesToSeed (LessonId, ActivityTypeId, Title, SequenceOrder, ContentJson)
    VALUES
        -- பாடம் 01: உடல் உறுப்புகள்
        (@Lesson1Id, 2, N'உடல் உறுப்புகளை அடையாளம் காண்க', 1, N'{"status": "pending"}'),
        -- பாடம் 02: எனது குடும்பம்
        (@Lesson2Id, 2, N'குடும்ப உறவுகளை அடையாளம் காண்க', 1, N'{"status": "pending"}'),
        -- பாடம் 03: எனது வீடு
        (@Lesson3Id, 2, N'வீட்டின் பகுதிகளை அடையாளம் காண்க', 1, N'{"status": "pending"}'),
        -- பாடம் 04: உணவுகள்
        (@Lesson4Id, 2, N'உணவுப் பொருட்களை அடையாளம் காண்க', 1, N'{"status": "pending"}'),
        (@Lesson4Id, 5, N'சுவைகளை வகைப்படுத்துக', 2, N'{"status": "pending"}'),
        -- பாடம் 05: வண்ணங்கள்
        (@Lesson5Id, 4, N'வண்ணங்களை பெயர்களுடன் பொருத்துக', 1, N'{"status": "pending"}'),
        -- பாடம் 06: பூக்கள்
        (@Lesson6Id, 2, N'பூக்களை அடையாளம் காண்க', 1, N'{"status": "pending"}'),
        (@Lesson6Id, 2, N'தொடர்புடைய பொருட்களைக் காண்க', 2, N'{"status": "pending"}'),
        -- பாடம் 07: பறவைகள்
        (@Lesson7Id, 2, N'பறவைகளை அடையாளம் காண்க', 1, N'{"status": "pending"}'),
        (@Lesson7Id, 4, N'பறவையை அதன் சிறப்போடு பொருத்துக', 2, N'{"status": "pending"}'),
        -- பாடம் 08: விலங்குகள்
        (@Lesson8Id, 2, N'விலங்குகளை அடையாளம் காண்க', 1, N'{"status": "pending"}'),
        (@Lesson8Id, 4, N'விலங்குகளை அதன் பயனுடன் பொருத்துக', 2, N'{"status": "pending"}'),
        -- பாடம் 09: விளையாட்டுகள்
        (@Lesson9Id, 2, N'விளையாட்டுகளை அடையாளம் காண்க', 1, N'{"status": "pending"}'),
        -- பாடம் 10: கொண்டாட்டம்
        (@Lesson10Id, 2, N'பண்டிகை பொருட்களை அடையாளம் காண்க', 1, N'{"status": "pending"}'),
        (@Lesson10Id, 1, N'உயிர் எழுத்துகள் மீட்டல்', 2, N'{"status": "pending"}');


    -- Insert only if the target activities do not already exist.
    INSERT INTO Activities (LessonId, ActivityTypeId, Title, SequenceOrder, ContentJson)
    SELECT 
        s.LessonId, 
        s.ActivityTypeId, 
        s.Title, 
        s.SequenceOrder, 
        s.ContentJson
    FROM @ActivitiesToSeed s
    WHERE NOT EXISTS (
        SELECT 1 
        FROM Activities a 
        WHERE a.LessonId = s.LessonId AND a.SequenceOrder = s.SequenceOrder
    );

    PRINT 'Seeding for "மழலையர் நிலை" completed successfully.';

    COMMIT TRANSACTION;

END TRY
BEGIN CATCH
    -- If any error occurs, rollback the entire transaction.
    ROLLBACK TRANSACTION;
    PRINT 'An error occurred during seeding. Transaction rolled back.';
    -- Re-throw the error to see the details.
    THROW;
END CATCH
GO
```

This script is now production-quality. It's wrapped in a transaction to ensure that either all activities are added or none are, preventing partial data insertion if an error occurs. You can now run this safely.

Let me know when you're ready to proceed with the next level, `சிறுவர் நிலை`.

### USER :
now lets proceed to siruvar nilai

### RESPONSE :
Excellent. Let's proceed with the `சிறுவர் நிலை` (Kindergarten) level.

This level introduces more complexity, including the alphabet, basic sentence structure, and more specific vocabulary sets. The activities will therefore be more varied.

### **Mapping Activities for `சிறுவர் நிலை` (Kindergarten Level)**

Let's assume `சிறுவர் நிலை` has `LevelId = 2`, and its lessons will have IDs starting after the preschool lessons.

| Lesson Name | Mapped Activities |
| :--- | :--- |
| **பாடம் 01: நான்** | 1. **Audio-Image Recog.** (`ActivityTypeId=2`): Body Parts.<br>2. **Matching** (`ActivityTypeId=4`): Match body part to its function (கண் -> காண்கிறோம்).<br>3. **Pronunciation Practice** (`ActivityTypeId=1`): Vowels அ, ஆ, இ, ஈ.<br>4. **Audio-Image Recog.** (`ActivityTypeId=2`): Words starting with அ, ஆ, இ, ஈ. |
| **பாடம் 02: என் குடும்பம்**| 1. **Audio-Image Recog.** (`ActivityTypeId=2`): Family members.<br>2. **Pronunciation Practice** (`ActivityTypeId=1`): Vowels உ, ஊ, எ, ஏ.<br>3. **Audio-Image Recog.** (`ActivityTypeId=2`): Words starting with உ, ஊ, எ, ஏ. |
| **பாடம் 03: எனது வீடு** | 1. **Audio-Image Recog.** (`ActivityTypeId=2`): Rooms and items in a house.<br>2. **Pronunciation Practice** (`ActivityTypeId=1`): Vowels ஐ, ஒ, ஓ.<br>3. **Audio-Image Recog.** (`ActivityTypeId=2`): Words starting with ஐ, ஒ, ஓ. |
| **பாடம் 04: உணவுகள்** | 1. **Audio-Image Recog.** (`ActivityTypeId=2`): Food types.<br>2. **Sorting/Classification** (`ActivityTypeId=5`): Sort food into categories (மரக்கறி, கடல் உணவு).<br>3. **Pronunciation Practice** (`ActivityTypeId=1`): Ayutha Ezhuthu `ஃ`. |
| **பாடம் 05: உடைகள்** | 1. **Audio-Image Recog.** (`ActivityTypeId=2`): Types of clothes.<br>2. **FillInTheBlanks** (`ActivityTypeId=7`): Complete words with `க், ங், ச், ஞ்`. (e.g., த__காளி). |
| **பாடம் 06: விலங்குகள்** | 1. **Audio-Image Recog.** (`ActivityTypeId=2`): Animal names.<br>2. **FillInTheBlanks** (`ActivityTypeId=7`): Complete words with `ட், ண், த், ந்`. |
| **பாடம் 07: பிறந்தநாள்** | 1. **Audio-Image Recog.** (`ActivityTypeId=2`): Birthday items.<br>2. **FillInTheBlanks** (`ActivityTypeId=7`): Complete words with `ப், ம், ய், ர்`. |
| **பாடம் 08: வகுப்பறை** | 1. **Audio-Image Recog.** (`ActivityTypeId=2`): Classroom objects.<br>2. **FillInTheBlanks** (`ActivityTypeId=7`): Complete words with `ல், வ், ழ், ள்`. |
| **பாடம் 09: உந்துகள்** | 1. **Audio-Image Recog.** (`ActivityTypeId=2`): Vehicle types.<br>2. **FillInTheBlanks** (`ActivityTypeId=7`): Complete words with `ற், ன்`. |
| **பாடம் 10: விளையாட்டு** | 1. **Matching** (`ActivityTypeId=4`): Match action words (ஓடுதல்) to images.<br>2. **Review/Pronunciation** (`ActivityTypeId=1`): Review all vowels and consonants. |

---

### **SQL `INSERT` Script for `சிறுவர் நிலை` Activities**

This script follows the same safe, transactional pattern as the previous one. It will populate the `Activities` table for the entire Kindergarten level.

```sql
-- ===================================================================
-- SCRIPT TO SEED 'Activities' TABLE FOR 'சிறுவர் நிலை' (Kindergarten)
-- ===================================================================

BEGIN TRANSACTION;

BEGIN TRY
    -- Step 1: Declare variables to hold the IDs we need.
    DECLARE @LevelId INT;
    DECLARE @Lesson1Id INT, @Lesson2Id INT, @Lesson3Id INT, @Lesson4Id INT, @Lesson5Id INT;
    DECLARE @Lesson6Id INT, @Lesson7Id INT, @Lesson8Id INT, @Lesson9Id INT, @Lesson10Id INT;

    -- Step 2: Get the LevelId for 'சிறுவர் நிலை'.
    SELECT @LevelId = LevelId FROM Levels WHERE LevelName = N'சிறுவர் நிலை';

    IF @LevelId IS NULL
    BEGIN
        PRINT 'Error: Level "சிறுவர் நிலை" not found. Seeding cannot continue.';
        ROLLBACK TRANSACTION;
        RETURN;
    END

    -- Step 3: Get the LessonIds for this level.
    SELECT @Lesson1Id = LessonId FROM Lessons WHERE LevelId = @LevelId AND SequenceOrder = 1;
    SELECT @Lesson2Id = LessonId FROM Lessons WHERE LevelId = @LevelId AND SequenceOrder = 2;
    SELECT @Lesson3Id = LessonId FROM Lessons WHERE LevelId = @LevelId AND SequenceOrder = 3;
    SELECT @Lesson4Id = LessonId FROM Lessons WHERE LevelId = @LevelId AND SequenceOrder = 4;
    SELECT @Lesson5Id = LessonId FROM Lessons WHERE LevelId = @LevelId AND SequenceOrder = 5;
    SELECT @Lesson6Id = LessonId FROM Lessons WHERE LevelId = @LevelId AND SequenceOrder = 6;
    SELECT @Lesson7Id = LessonId FROM Lessons WHERE LevelId = @LevelId AND SequenceOrder = 7;
    SELECT @Lesson8Id = LessonId FROM Lessons WHERE LevelId = @LevelId AND SequenceOrder = 8;
    SELECT @Lesson9Id = LessonId FROM Lessons WHERE LevelId = @LevelId AND SequenceOrder = 9;
    SELECT @Lesson10Id = LessonId FROM Lessons WHERE LevelId = @LevelId AND SequenceOrder = 10;
    
    PRINT 'Seeding activities for "சிறுவர் நிலை"...';

    DECLARE @ActivitiesToSeed TABLE (
        LessonId INT, ActivityTypeId INT, Title NVARCHAR(250), SequenceOrder INT, ContentJson NVARCHAR(MAX)
    );

    INSERT INTO @ActivitiesToSeed (LessonId, ActivityTypeId, Title, SequenceOrder, ContentJson)
    VALUES
        -- பாடம் 01: நான்
        (@Lesson1Id, 2, N'உடல் உறுப்புகள்', 1, N'{"status": "pending"}'),
        (@Lesson1Id, 4, N'உறுப்பும் தொழிலும்', 2, N'{"status": "pending"}'),
        (@Lesson1Id, 1, N'உயிர் எழுத்துகள் (அ-ஈ)', 3, N'{"status": "pending"}'),
        (@Lesson1Id, 2, N'எழுத்து-சொல் அடையாளம்', 4, N'{"status": "pending"}'),

        -- பாடம் 02: என் குடும்பம்
        (@Lesson2Id, 2, N'என் உறவுகள்', 1, N'{"status": "pending"}'),
        (@Lesson2Id, 1, N'உயிர் எழுத்துகள் (உ-ஏ)', 2, N'{"status": "pending"}'),
        (@Lesson2Id, 2, N'எழுத்து-சொல் அடையாளம்', 3, N'{"status": "pending"}'),

        -- பாடம் 03: எனது வீடு
        (@Lesson3Id, 2, N'வீட்டின் அறைகள்', 1, N'{"status": "pending"}'),
        (@Lesson3Id, 1, N'உயிர் எழுத்துகள் (ஐ-ஓ)', 2, N'{"status": "pending"}'),
        (@Lesson3Id, 2, N'எழுத்து-சொல் அடையாளம்', 3, N'{"status": "pending"}'),

        -- பாடம் 04: உணவுகள்
        (@Lesson4Id, 2, N'உணவு வகைகள்', 1, N'{"status": "pending"}'),
        (@Lesson4Id, 5, N'உணவை வகைப்படுத்து', 2, N'{"status": "pending"}'),
        (@Lesson4Id, 1, N'ஆய்த எழுத்து (ஃ)', 3, N'{"status": "pending"}'),

        -- பாடம் 05: உடைகள்
        (@Lesson5Id, 2, N'உடைகளின் பெயர்கள்', 1, N'{"status": "pending"}'),
        (@Lesson5Id, 7, N'விடுபட்ட மெய்யெழுத்து (க்,ங்,ச்,ஞ்)', 2, N'{"status": "pending"}'),

        -- பாடம் 06: விலங்குகள்
        (@Lesson6Id, 2, N'விலங்குகளின் பெயர்கள்', 1, N'{"status": "pending"}'),
        (@Lesson6Id, 7, N'விடுபட்ட மெய்யெழுத்து (ட்,ண்,த்,ந்)', 2, N'{"status": "pending"}'),

        -- பாடம் 07: பிறந்தநாள்
        (@Lesson7Id, 2, N'பிறந்தநாள் கொண்டாட்டம்', 1, N'{"status": "pending"}'),
        (@Lesson7Id, 7, N'விடுபட்ட மெய்யெழுத்து (ப்,ம்,ய்,ர்)', 2, N'{"status": "pending"}'),

        -- பாடம் 08: வகுப்பறை
        (@Lesson8Id, 2, N'வகுப்பறைப் பொருட்கள்', 1, N'{"status": "pending"}'),
        (@Lesson8Id, 7, N'விடுபட்ட மெய்யெழுத்து (ல்,வ்,ழ்,ள்)', 2, N'{"status": "pending"}'),

        -- பாடம் 09: உந்துகள்
        (@Lesson9Id, 2, N'உந்துகளின் பெயர்கள்', 1, N'{"status": "pending"}'),
        (@Lesson9Id, 7, N'விடுபட்ட மெய்யெழுத்து (ற்,ன்)', 2, N'{"status": "pending"}'),

        -- பாடம் 10: விளையாட்டு
        (@Lesson10Id, 4, N'செயல்களைப் பொருத்துக', 1, N'{"status": "pending"}'),
        (@Lesson10Id, 1, N'எழுத்துகள் மீட்டல்', 2, N'{"status": "pending"}');

    -- Insert only if the target activities do not already exist.
    INSERT INTO Activities (LessonId, ActivityTypeId, Title, SequenceOrder, ContentJson)
    SELECT 
        s.LessonId, s.ActivityTypeId, s.Title, s.SequenceOrder, s.ContentJson
    FROM @ActivitiesToSeed s
    WHERE NOT EXISTS (
        SELECT 1 FROM Activities a WHERE a.LessonId = s.LessonId AND a.SequenceOrder = s.SequenceOrder
    ) AND s.LessonId IS NOT NULL; -- Ensure we don't insert if a lesson was not found

    PRINT 'Seeding for "சிறுவர் நிலை" completed successfully.';

    COMMIT TRANSACTION;

END TRY
BEGIN CATCH
    ROLLBACK TRANSACTION;
    PRINT 'An error occurred during seeding for "சிறுவர் நிலை". Transaction rolled back.';
    THROW;
END CATCH
GO
```

We have now successfully mapped and created the seeding script for the Kindergarten level. The activities are more diverse, introducing `FillInTheBlanks` for the consonant exercises, which is a perfect progression from the simple recognition in Preschool.

Let me know when you are ready to move on to `ஆண்டு 01` (Grade 1).

### USER :
ok , now lets  proceed to grade 1

### RESPONSE :
Excellent. Let's move on to `ஆண்டு 01` (Grade 1).

This level is highly structured and introduces the Tamil alphabet systematically. Almost every lesson introduces a new vowel-consonant (`உயிர்மெய்`) series, making the mapping very consistent. The activities focus on recognizing these new letters, using them in words, and simple sentence-level understanding.

### **Mapping Activities for `ஆண்டு 01` (Grade 1)**

We'll assume `ஆண்டு 01` has `LevelId = 3`.

| Lesson Name | Mapped Activities |
| :--- | :--- |
| **பாடம் 01: தமிழ்ப்பள்ளி** | 1. **Audio-Image Recog.** (`ActivityTypeId=2`): Click on classroom items.<br>2. **Pronunciation Practice** (`ActivityTypeId=1`): Vowels & Consonants.<br>3. **Pronunciation Practice** (`ActivityTypeId=1`): `அ` series (`க, ங, ச...`).<br>4. **FillInTheBlanks** (`ActivityTypeId=7`): Complete `அ` series words. |
| **பாடம் 02: விடுமுறை** | 1. **Audio-Image Recog.** (`ActivityTypeId=2`): Vacation items.<br>2. **Pronunciation Practice** (`ActivityTypeId=1`): `ஆ` series (`கா, ஙா, சா...`).<br>3. **Sorting/Classification** (`ActivityTypeId=5`): Differentiate short/long vowel words (`பல்` vs `பால்`). |
| **பாடம் 03: நண்பர்கள்** | 1. **Audio-Image Recog.** (`ActivityTypeId=2`): Words related to friendship.<br>2. **Pronunciation Practice** (`ActivityTypeId=1`): `இ` series (`கி, ஙி, சி...`).<br>3. **FillInTheBlanks** (`ActivityTypeId=7`): Complete `இ` series words. |
| **பாடம் 04: அன்னையர் நாள்**| 1. **Audio-Image Recog.** (`ActivityTypeId=2`): Mother's Day items.<br>2. **Pronunciation Practice** (`ActivityTypeId=1`): `ஈ` series (`கீ, ஙீ, சீ...`).<br>3. **FillInTheBlanks** (`ActivityTypeId=7`): Complete `ஈ` series words. |
| **பாடம் 05: தைப்பொங்கல்** | 1. **Audio-Image Recog.** (`ActivityTypeId=2`): Pongal items.<br>2. **Pronunciation Practice** (`ActivityTypeId=1`): `உ` series (`கு, ஙு, சு...`).<br>3. **MultipleChoiceQuestion** (`ActivityTypeId=13`): Simple comprehension based on images. |
| **பாடம் 06: அங்காடி** | 1. **Audio-Image Recog.** (`ActivityTypeId=2`): Market items.<br>2. **Pronunciation Practice** (`ActivityTypeId=1`): `ஊ` series (`கூ, ஙூ, சூ...`).<br>3. **Sorting/Classification** (`ActivityTypeId=5`): Sorting by taste (`சுவைகள்`). |
| **பாடம் 07: வீட்டு விலங்குகள்**| 1. **Audio-Image Recog.** (`ActivityTypeId=2`): Domestic animals and their homes.<br>2. **Pronunciation Practice** (`ActivityTypeId=1`): `எ` & `ஏ` series (`கெ/கே...`).<br>3. **FillInTheBlanks** (`ActivityTypeId=7`): Complete `எ/ஏ` series words. |
| **பாடம் 08: பனி காலம்** | 1. **Audio-Image Recog.** (`ActivityTypeId=2`): Winter items.<br>2. **Pronunciation Practice** (`ActivityTypeId=1`): `ஐ` series (`கை, ஙை, சை...`).<br>3. **FillInTheBlanks** (`ActivityTypeId=7`): Complete `ஐ` series words. |
| **பாடம் 09: நிறையுணவு** | 1. **Audio-Image Recog.** (`ActivityTypeId=2`): Healthy food items.<br>2. **Pronunciation Practice** (`ActivityTypeId=1`): `ஒ` & `ஓ` series (`கொ/கோ...`).<br>3. **Matching** (`ActivityTypeId=4`): Match food to its category. |
| **பாடம் 10: நாள்கள், மாதங்கள்**| 1. **Audio-Image Recog.** (`ActivityTypeId=2`): Days and months.<br>2. **Pronunciation Practice** (`ActivityTypeId=1`): `ஔ` series & `ஃ`.<br>3. **StorySequencing** (`ActivityTypeId=16`): Arrange days of the week in order. |

---

### **SQL `INSERT` Script for `ஆண்டு 01` Activities**

This script populates the `Activities` table for the entire Grade 1 level.

```sql
-- ===================================================================
-- SCRIPT TO SEED 'Activities' TABLE FOR 'ஆண்டு 01' (Grade 1)
-- ===================================================================

BEGIN TRANSACTION;

BEGIN TRY
    -- Step 1: Declare variables
    DECLARE @LevelId INT;
    DECLARE @Lesson1Id INT, @Lesson2Id INT, @Lesson3Id INT, @Lesson4Id INT, @Lesson5Id INT;
    DECLARE @Lesson6Id INT, @Lesson7Id INT, @Lesson8Id INT, @Lesson9Id INT, @Lesson10Id INT;

    -- Step 2: Get the LevelId for 'ஆண்டு 01'.
    SELECT @LevelId = LevelId FROM Levels WHERE LevelName = N'ஆண்டு 01';

    IF @LevelId IS NULL
    BEGIN
        PRINT 'Error: Level "ஆண்டு 01" not found. Seeding cannot continue.';
        ROLLBACK TRANSACTION;
        RETURN;
    END

    -- Step 3: Get the LessonIds for this level.
    SELECT @Lesson1Id = LessonId FROM Lessons WHERE LevelId = @LevelId AND SequenceOrder = 1;
    SELECT @Lesson2Id = LessonId FROM Lessons WHERE LevelId = @LevelId AND SequenceOrder = 2;
    SELECT @Lesson3Id = LessonId FROM Lessons WHERE LevelId = @LevelId AND SequenceOrder = 3;
    SELECT @Lesson4Id = LessonId FROM Lessons WHERE LevelId = @LevelId AND SequenceOrder = 4;
    SELECT @Lesson5Id = LessonId FROM Lessons WHERE LevelId = @LevelId AND SequenceOrder = 5;
    SELECT @Lesson6Id = LessonId FROM Lessons WHERE LevelId = @LevelId AND SequenceOrder = 6;
    SELECT @Lesson7Id = LessonId FROM Lessons WHERE LevelId = @LevelId AND SequenceOrder = 7;
    SELECT @Lesson8Id = LessonId FROM Lessons WHERE LevelId = @LevelId AND SequenceOrder = 8;
    SELECT @Lesson9Id = LessonId FROM Lessons WHERE LevelId = @LevelId AND SequenceOrder = 9;
    SELECT @Lesson10Id = LessonId FROM Lessons WHERE LevelId = @LevelId AND SequenceOrder = 10;
    
    PRINT 'Seeding activities for "ஆண்டு 01"...';

    DECLARE @ActivitiesToSeed TABLE (
        LessonId INT, ActivityTypeId INT, Title NVARCHAR(250), SequenceOrder INT, ContentJson NVARCHAR(MAX)
    );

    INSERT INTO @ActivitiesToSeed (LessonId, ActivityTypeId, Title, SequenceOrder, ContentJson)
    VALUES
        -- பாடம் 01: தமிழ்ப்பள்ளி
        (@Lesson1Id, 2,  N'வகுப்பறைப் பொருட்களை அறிவோம்', 1, N'{"status": "pending"}'),
        (@Lesson1Id, 1,  N'எழுத்துகளை ஒலித்துப் பழகுவோம்', 2, N'{"status": "pending"}'),
        (@Lesson1Id, 1,  N'அ-ஓசை உயிர்மெய்', 3, N'{"status": "pending"}'),
        (@Lesson1Id, 7,  N'சொற்களை நிரப்புக', 4, N'{"status": "pending"}'),

        -- பாடம் 02: விடுமுறை
        (@Lesson2Id, 2,  N'விடுமுறை சொற்கள்', 1, N'{"status": "pending"}'),
        (@Lesson2Id, 1,  N'ஆ-ஓசை உயிர்மெய்', 2, N'{"status": "pending"}'),
        (@Lesson2Id, 5,  N'குறில்-நெடில் வேறுபாடு', 3, N'{"status": "pending"}'),

        -- பாடம் 03: நண்பர்கள்
        (@Lesson3Id, 2,  N'நட்பு சொற்கள்', 1, N'{"status": "pending"}'),
        (@Lesson3Id, 1,  N'இ-ஓசை உயிர்மெய்', 2, N'{"status": "pending"}'),
        (@Lesson3Id, 7,  N'இ-ஓசை சொற்களை நிரப்புக', 3, N'{"status": "pending"}'),

        -- பாடம் 04: அன்னையர் நாள்
        (@Lesson4Id, 2,  N'அன்னையர் நாள் சொற்கள்', 1, N'{"status": "pending"}'),
        (@Lesson4Id, 1,  N'ஈ-ஓசை உயிர்மெய்', 2, N'{"status": "pending"}'),
        (@Lesson4Id, 7,  N'ஈ-ஓசை சொற்களை நிரப்புக', 3, N'{"status": "pending"}'),

        -- பாடம் 05: தைப்பொங்கல்
        (@Lesson5Id, 2,  N'தைப்பொங்கல் சொற்கள்', 1, N'{"status": "pending"}'),
        (@Lesson5Id, 1,  N'உ-ஓசை உயிர்மெய்', 2, N'{"status": "pending"}'),
        (@Lesson5Id, 13, N'படங்களைப் பார்த்து விடையளி', 3, N'{"status": "pending"}'),

        -- பாடம் 06: அங்காடி
        (@Lesson6Id, 2,  N'அங்காடிப் பொருட்கள்', 1, N'{"status": "pending"}'),
        (@Lesson6Id, 1,  N'ஊ-ஓசை உயிர்மெய்', 2, N'{"status": "pending"}'),
        (@Lesson6Id, 5,  N'சுவைகளை வகைப்படுத்து', 3, N'{"status": "pending"}'),

        -- பாடம் 07: வீட்டு விலங்குகள்
        (@Lesson7Id, 2,  N'விலங்குகளும் இருப்பிடமும்', 1, N'{"status": "pending"}'),
        (@Lesson7Id, 1,  N'எ, ஏ-ஓசை உயிர்மெய்', 2, N'{"status": "pending"}'),
        (@Lesson7Id, 7,  N'எ, ஏ-ஓசை சொற்களை நிரப்புக', 3, N'{"status": "pending"}'),

        -- பாடம் 08: பனி காலம்
        (@Lesson8Id, 2,  N'பனிகாலப் பொருட்கள்', 1, N'{"status": "pending"}'),
        (@Lesson8Id, 1,  N'ஐ-ஓசை உயிர்மெய்', 2, N'{"status": "pending"}'),
        (@Lesson8Id, 7,  N'ஐ-ஓசை சொற்களை நிரப்புக', 3, N'{"status": "pending"}'),

        -- பாடம் 09: நிறையுணவு
        (@Lesson9Id, 2,  N'சத்தான உணவுகள்', 1, N'{"status": "pending"}'),
        (@Lesson9Id, 1,  N'ஒ, ஓ-ஓசை உயிர்மெய்', 2, N'{"status": "pending"}'),
        (@Lesson9Id, 4,  N'உணவுப் பொருட்களைப் பொருத்துக', 3, N'{"status": "pending"}'),

        -- பாடம் 10: நாள்கள், மாதங்கள்
        (@Lesson10Id, 2, N'நாள்களும் மாதங்களும்', 1, N'{"status": "pending"}'),
        (@Lesson10Id, 1, N'ஔ-ஓசை உயிர்மெய் மற்றும் ஃ', 2, N'{"status": "pending"}'),
        (@Lesson10Id, 16, N'கிழமைகளை வரிசைப்படுத்துக', 3, N'{"status": "pending"}');

    -- Insert only if the target activities do not already exist.
    INSERT INTO Activities (LessonId, ActivityTypeId, Title, SequenceOrder, ContentJson)
    SELECT 
        s.LessonId, s.ActivityTypeId, s.Title, s.SequenceOrder, s.ContentJson
    FROM @ActivitiesToSeed s
    WHERE NOT EXISTS (
        SELECT 1 FROM Activities a WHERE a.LessonId = s.LessonId AND a.SequenceOrder = s.SequenceOrder
    ) AND s.LessonId IS NOT NULL;

    PRINT 'Seeding for "ஆண்டு 01" completed successfully.';

    COMMIT TRANSACTION;

END TRY
BEGIN CATCH
    ROLLBACK TRANSACTION;
    PRINT 'An error occurred during seeding for "ஆண்டு 01". Transaction rolled back.';
    THROW;
END CATCH
GO
```

We have now seeded the activities for Grade 1. This level systematically builds the user's knowledge of the Tamil script, and the chosen activities reflect that structured approach.

When you're ready, we can proceed to `ஆண்டு 02` (Grade 2), where the focus will shift more towards vocabulary building, sentence structure, and basic grammar concepts.

### USER
lets proceed to grade 2

### RESPONSE
Excellent. Let's move on to `ஆண்டு 02` (Grade 2).

This level marks a significant step up. The focus shifts from basic alphabet recognition to vocabulary building, understanding simple grammatical concepts like gender/number agreement, and forming sentences. The activities will reflect this, incorporating more logic-based tasks.

### **Mapping Activities for `ஆண்டு 02` (Grade 2)**

We'll assume `ஆண்டு 02` has `LevelId = 4`.

| Lesson Name | Mapped Activities |
| :--- | :--- |
| **பாடம் 01: தமிழ்மொழி** | 1. **Reading Comprehension** (`ActivityTypeId=15`): Simple sentences about Tamil.<br>2. **Sorting/Classification** (`ActivityTypeId=5`): `ஒரு` vs `ஓர்` usage.<br>3. **Word Scramble** (`ActivityTypeId=8`): Unscramble basic words. |
| **பாடம் 02: எங்கள் தாயகம்**| 1. **Reading Comprehension** (`ActivityTypeId=15`): About Sri Lanka and its cities.<br>2. **Sentence Building** (`ActivityTypeId=11`): Match subject to correct verb form (அவன் செல்கிறான், அவை வளர்கின்றன).<br>3. **Word Scramble** (`ActivityTypeId=8`): Unscramble names of cities and features. |
| **பாடம் 03: இன்கலைகள்** | 1. **Audio-Image Recog.** (`ActivityTypeId=2`): Art forms and instruments.<br>2. **Sentence Building** (`ActivityTypeId=11`): Fill in the correct verb based on gender/number.<br>3. **Sorting/Classification** (`ActivityTypeId=5`): Sort verbs by gender (அவன், அவள், அது, அவை). |
| **பாடம் 04: கை கொடுப்போம்**| 1. **Audio-Image Recog.** (`ActivityTypeId=2`): Community helpers.<br>2. **Matching** (`ActivityTypeId=4`): Match helper to their tool/vehicle.<br>3. **Sentence Building** (`ActivityTypeId=11`): Match subject (நாங்கள், நீங்கள்) to correct verb form. |
| **பாடம் 05: சங்கிலியன்** | 1. **Reading Comprehension** (`ActivityTypeId=15`): Story of King Cankiliyan.<br>2. **Sentence Scramble** (`ActivityTypeId=9`): Reorder sentences from the story.<br>3. **Sorting/Classification** (`ActivityTypeId=5`): Sort items into Fruits (`பழங்கள்`) vs Vegetables (`காய்கறிகள்`). |
| **பாடம் 06: பருவகாலங்கள்**| 1. **StorySequencing** (`ActivityTypeId=16`): Arrange the months of the year in order.<br>2. **Matching** (`ActivityTypeId=4`): Match the days of the week (`கிழமை`).<br>3. **FillInTheBlanks** (`ActivityTypeId=7`): Complete sentences about seasons. |
| **பாடம் 07: நாம் வாழும் சூழல்**| 1. **Audio-Image Recog.** (`ActivityTypeId=2`): Small creatures (`சிற்றுயிரிகள்`).<br>2. **Sorting/Classification** (`ActivityTypeId=5`): Sort words into singular/plural (`ஒருமை/பன்மை`).<br>3. **Sorting/Classification** (`ActivityTypeId=5`): Sort sentences based on `அது` vs `அவை`. |
| **பாடம் 08: சோமசுந்தரப் புலவர்**| 1. **Reading Comprehension** (`ActivityTypeId=15`): About the poet.<br>2. **Matching** (`ActivityTypeId=4`): Match animal to its sound (`சேவல் -> கூவும்`).<br>3. **FillInTheBlanks** (`ActivityTypeId=7`): Complete sentences with the correct animal sound. |
| **பாடம் 09: பனைமரம்** | 1. **Reading Comprehension** (`ActivityTypeId=15`): About the Palmyra tree.<br>2. **Sentence Scramble** (`ActivityTypeId=9`): Reorder sentences about the tree.<br>3. **Matching** (`ActivityTypeId=4`): Match parts of the tree to its products (`பனம்பழம், நுங்கு`). |
| **பாடம் 10: ஔவையார்** | 1. **Reading Comprehension** (`ActivityTypeId=15`): Introduction to Aathichoodi.<br>2. **Matching** (`ActivityTypeId=4`): Match Aathichoodi lines to their simple meaning (`அறம் செய விரும்பு -> நல்லதையே செய்வோம்`).<br>3. **FillInTheBlanks** (`ActivityTypeId=7`): Complete Aathichoodi verses. |
| **பாடம் 11: மூத்தோரை மதிப்போம்**| 1. **Matching** (`ActivityTypeId=4`): Match relationship terms (e.g., `அம்மாவின் அண்ணா -> மாமா`).<br>2. **Audio-Image Recog.** (`ActivityTypeId=2`): Identify family members from a chart. |
| **பாடம் 12: விண்வெளி** | 1. **Audio-Image Recog.** (`ActivityTypeId=2`): Planets and sea creatures.<br>2. **Sorting/Classification** (`ActivityTypeId=5`): Sort into Planets vs Sea Creatures.<br>3. **Matching** (`ActivityTypeId=4`): Match times on a clock face (analog to digital). |

---

### **SQL `INSERT` Script for `ஆண்டு 02` Activities**

This script populates the `Activities` table for the entire Grade 2 level.

```sql
-- ===================================================================
-- SCRIPT TO SEED 'Activities' TABLE FOR 'ஆண்டு 02' (Grade 2)
-- ===================================================================

BEGIN TRANSACTION;

BEGIN TRY
    DECLARE @LevelId INT;
    SELECT @LevelId = LevelId FROM Levels WHERE LevelName = N'ஆண்டு 02';

    IF @LevelId IS NULL
    BEGIN
        PRINT 'Error: Level "ஆண்டு 02" not found. Seeding cannot continue.';
        ROLLBACK TRANSACTION;
        RETURN;
    END
    
    -- Dynamically get all LessonIds for this level to avoid hardcoding
    DECLARE @LessonsInLevel TABLE (LessonId INT, SequenceOrder INT);
    INSERT INTO @LessonsInLevel SELECT LessonId, SequenceOrder FROM Lessons WHERE LevelId = @LevelId;

    PRINT 'Seeding activities for "ஆண்டு 02"...';

    DECLARE @ActivitiesToSeed TABLE (
        LessonSequence INT, ActivityTypeId INT, Title NVARCHAR(250), ActivitySequence INT, ContentJson NVARCHAR(MAX)
    );

    INSERT INTO @ActivitiesToSeed (LessonSequence, ActivityTypeId, Title, ActivitySequence, ContentJson)
    VALUES
        (1, 15, N'தமிழ்மொழி வாசிப்பு', 1, N'{"status": "pending"}'),
        (1, 5,  N'ஒரு/ஓர் வேறுபாடு', 2, N'{"status": "pending"}'),
        (1, 8,  N'சொற்களை ஒழுங்குபடுத்துக', 3, N'{"status": "pending"}'),

        (2, 15, N'எங்கள் தாயகம் வாசிப்பு', 1, N'{"status": "pending"}'),
        (2, 11, N'வினைச்சொல் பொருத்தம்', 2, N'{"status": "pending"}'),
        (2, 8,  N'ஊர்ப் பெயர்களை உருவாக்குக', 3, N'{"status": "pending"}'),

        (3, 2,  N'இன்கலைகளை அறிவோம்', 1, N'{"status": "pending"}'),
        (3, 11, N'சரியான வினையை நிரப்புக', 2, N'{"status": "pending"}'),
        (3, 5,  N'பால்வேறுபாடு அறிதல்', 3, N'{"status": "pending"}'),

        (4, 2,  N'சமூக உதவியாளர்கள்', 1, N'{"status": "pending"}'),
        (4, 4,  N'உதவியாளரும் கருவியும்', 2, N'{"status": "pending"}'),
        (4, 11, N' தன்மை, முன்னிலை வினைமுற்று', 3, N'{"status": "pending"}'),

        (5, 15, N'சங்கிலியன் கதை', 1, N'{"status": "pending"}'),
        (5, 9,  N'கதையை வரிசைப்படுத்துக', 2, N'{"status": "pending"}'),
        (5, 5,  N'பழங்கள்-காய்கறிகள் வகைப்படுத்து', 3, N'{"status": "pending"}'),

        (6, 16, N'மாதங்களை வரிசைப்படுத்துக', 1, N'{"status": "pending"}'),
        (6, 4,  N'கிழமைகளைப் பொருத்துக', 2, N'{"status": "pending"}'),
        (6, 7,  N'பருவங்கள் வாக்கியம் நிரப்பல்', 3, N'{"status": "pending"}'),

        (7, 2,  N'சிற்றுயிரிகளை அறிவோம்', 1, N'{"status": "pending"}'),
        (7, 5,  N'ஒருமை / பன்மை', 2, N'{"status": "pending"}'),
        (7, 5,  N'அது / அவை', 3, N'{"status": "pending"}'),

        (8, 15, N'சோமசுந்தரப் புலவர் வாசிப்பு', 1, N'{"status": "pending"}'),
        (8, 4,  N'விலங்கும் ஒலியும்', 2, N'{"status": "pending"}'),
        (8, 7,  N'ஒலிகளை நிரப்புக', 3, N'{"status": "pending"}'),

        (9, 15, N'பனைமரம் வாசிப்பு', 1, N'{"status": "pending"}'),
        (9, 9,  N'வாக்கியங்களை வரிசைப்படுத்துக', 2, N'{"status": "pending"}'),
        (9, 4,  N'பனைமரம் பயன்கள்', 3, N'{"status": "pending"}'),

        (10, 15, N'ஔவையார் ஆத்திசூடி', 1, N'{"status": "pending"}'),
        (10, 4, N'ஆத்திசூடியும் பொருளும்', 2, N'{"status": "pending"}'),
        (10, 7, N'ஆத்திசூடியை நிரப்புக', 3, N'{"status": "pending"}'),

        (11, 4,  N'உறவுமுறைகளைப் பொருத்துக', 1, N'{"status": "pending"}'),
        (11, 2,  N'குடும்ப வரைபடம்', 2, N'{"status": "pending"}'),
        
        (12, 2,  N'விண்வெளி ও கடல்', 1, N'{"status": "pending"}'),
        (12, 5,  N'கோள்கள்-கடல்வாழ் உயிரினங்கள்', 2, N'{"status": "pending"}'),
        (12, 4,  N'கடிகாரம் பார்த்தல்', 3, N'{"status": "pending"}');

    -- Insert using the dynamically fetched LessonIds
    INSERT INTO Activities (LessonId, ActivityTypeId, Title, ActivitySequence, ContentJson)
    SELECT 
        l.LessonId, 
        s.ActivityTypeId, 
        s.Title, 
        s.ActivitySequence, 
        s.ContentJson
    FROM @ActivitiesToSeed s
    JOIN @LessonsInLevel l ON s.LessonSequence = l.SequenceOrder
    WHERE NOT EXISTS (
        SELECT 1 FROM Activities a WHERE a.LessonId = l.LessonId AND a.SequenceOrder = s.ActivitySequence
    );

    PRINT 'Seeding for "ஆண்டு 02" completed successfully.';
    COMMIT TRANSACTION;

END TRY
BEGIN CATCH
    ROLLBACK TRANSACTION;
    PRINT 'An error occurred during seeding for "ஆண்டு 02". Transaction rolled back.';
    THROW;
END CATCH
GO

```

We're making great progress. Grade 2 is now mapped and seeded. The activities are becoming more diverse, setting the stage for the more complex grammatical and comprehension tasks in Grade 3.

Let me know when you're ready for `ஆண்டு 03`.

### USER
lets proceed for grade 3

### RESPONSE
Excellent. Let's proceed with `ஆண்டு 03` (Grade 3).

This level introduces more explicit grammar rules (`வேற்றுமை உருபுகள்` - case markers), synonyms/antonyms, and more complex reading comprehension. The activities will now start to test not just recognition, but also application of these new rules.

### **Mapping Activities for `ஆண்டு 03` (Grade 3)**

We'll assume `ஆண்டு 03` has `LevelId = 5`.

| Lesson Name | Mapped Activities |
| :--- | :--- |
| **பாடம் 01: எமது மொழி** | 1. **Matching** (`ActivityTypeId=4`): Match authors to their works.<br>2. **Sorting/Classification** (`ActivityTypeId=5`): Sort into `இயல், இசை, நாடகம்`.<br>3. **SentenceBuilding** (`ActivityTypeId=11`): Construct sentences using the `ஐ` case marker.<br>4. **PronunciationPractice** (`ActivityTypeId=1`): Practice `லகர/ளகர/ழகர` words. |
| **பாடம் 02: முக்கனிகள்** | 1. **Matching** (`ActivityTypeId=4`): Identify the three great fruits (`மா, பலா, வாழை`).<br>2. **SentenceBuilding** (`ActivityTypeId=11`): Construct sentences using the `ஆல்` case marker.<br>3. **ReadingComprehension** (`ActivityTypeId=15`): Read a passage about farmers and answer questions. |
| **பாடம் 03: கொடை** | 1. **SentenceBuilding** (`ActivityTypeId=11`): Construct sentences using the `கு` case marker.<br>2. **Matching** (`ActivityTypeId=4`): Match synonyms (`வீடு -> இல்லம்`).<br>3. **Matching** (`ActivityTypeId=4`): Match singular subjects to plural verbs to show agreement (e.g., `அவள் -> பாடுகிறாள்`, `அவர்கள் -> பாடுகிறார்கள்`). |
| **பாடம் 04: ஒற்றுமை** | 1. **ReadingComprehension** (`ActivityTypeId=15`): Read a story about unity and re-sequence it.<br>2. **Matching** (`ActivityTypeId=4`): Match antonyms with pictures (`சூடு -> குளிர்`).<br>3. **GrammarPuzzle** (`ActivityTypeId=12`): Solve riddles (`நொடி`). |
| **பாடம் 05: மட்டக்களப்பு** | 1. **Matching** (`ActivityTypeId=4`): Match locations on a map (`படம் பொருத்துதல்`).<br>2. **SentenceScramble** (`ActivityTypeId=9`): Reorder sentences about Matticaloa.<br>3. **OddOneOut** (`ActivityTypeId=6`): Find the item that doesn't belong in a set. |
| **பாடம் 06: பண்டாரவன்னியன்**| 1. **StorySequencing** (`ActivityTypeId=16`): Arrange the panels of the play in the correct order.<br>2. **Sorting/Classification** (`ActivityTypeId=5`): Sort items by position (`மேலே, கீழே, அருகில்`).<br>3. **GrammarPuzzle** (`ActivityTypeId=12`): Solve riddles (`நொடி`). |
| **பாடம் 07: நாட்டார் பாடல்கள்**| 1. **Audio-Image Recog.** (`ActivityTypeId=2`): Identify different types of folk songs from audio clips.<br>2. **Matching** (`ActivityTypeId=4`): Match professions to their tools (`நெசவு -> நூல்`).<br>3. **SentenceBuilding** (`ActivityTypeId=11`): Construct sentences using the `இல்` case marker. |
| **பாடம் 08: திருவள்ளுவர்**| 1. **Interactive Activity (Coloring):** A non-standard activity where a user can color a picture of Thiruvalluvar. (We can classify this as a simple `Audio-Image Recog.` where tapping a color fills a region).<br>2. **PronunciationPractice** (`ActivityTypeId=1`): Recite two specific Kurals.<br>3. **WordScramble** (`ActivityTypeId=8`): Unscramble words from the Kurals. |
| **பாடம் 09: எங்கள் பட்டம்** | 1. **Matching** (`ActivityTypeId=4`): Match the correct head to the animal's body.<br>2. **FillInTheBlanks** (`ActivityTypeId=7`): Complete words with missing letters (`எழுத்துக்குவியல்`).<br>3. **GrammarPuzzle** (`ActivityTypeId=12`): Solve riddles. |
| **பாடம் 10: கல்லணை** | 1. **Matching** (`ActivityTypeId=4`): Match world wonders to their countries.<br>2. **WordFormation** (`ActivityTypeId=10`): Join words (`சேர்த்து எழுதுவோம்`).<br>3. **ReadingComprehension** (`ActivityTypeId=15`): Read a story and answer questions based on the sequence of events. |
| **பாடம் 11: தண்ணீர்** | 1. **Matching** (`ActivityTypeId=4`): Match images with opposite meanings (`நீளமான പലகை vs அகலமான പലகை`).<br>2. **Sorting/Classification** (`ActivityTypeId=5`): Sort by gender and number (`ஒருமை, பன்மை, ஆண்பால், பெண்பால்`).<br>3. **ReadingComprehension** (`ActivityTypeId=15`): Read a story about a park. |
| **பாடம் 12: கிளித்தட்டு** | 1. **Matching** (`ActivityTypeId=4`): Match sports to their equipment.<br>2. **FillInTheBlanks** (`ActivityTypeId=7`): Construct questions using `என்ன, எங்கே, யார்`.<br>3. **Sorting/Classification** (`ActivityTypeId=5`): Sort sentences by tense (`நேற்று, இன்று, நாளைக்கு`). |

---

### **SQL `INSERT` Script for `ஆண்டு 03` Activities**

This script populates the `Activities` table for the entire Grade 3 level.

```sql
-- ===================================================================
-- SCRIPT TO SEED 'Activities' TABLE FOR 'ஆண்டு 03' (Grade 3)
-- ===================================================================

BEGIN TRANSACTION;

BEGIN TRY
    DECLARE @LevelId INT;
    SELECT @LevelId = LevelId FROM Levels WHERE LevelName = N'ஆண்டு 03';

    IF @LevelId IS NULL
    BEGIN
        PRINT 'Error: Level "ஆண்டு 03" not found. Seeding cannot continue.';
        ROLLBACK TRANSACTION;
        RETURN;
    END
    
    DECLARE @LessonsInLevel TABLE (LessonId INT, SequenceOrder INT);
    INSERT INTO @LessonsInLevel SELECT LessonId, SequenceOrder FROM Lessons WHERE LevelId = @LevelId;

    PRINT 'Seeding activities for "ஆண்டு 03"...';

    DECLARE @ActivitiesToSeed TABLE (
        LessonSequence INT, ActivityTypeId INT, Title NVARCHAR(250), ActivitySequence INT, ContentJson NVARCHAR(MAX)
    );

    INSERT INTO @ActivitiesToSeed (LessonSequence, ActivityTypeId, Title, ActivitySequence, ContentJson)
    VALUES
        (1, 4, N'நூல்களும் ஆசிரியர்களும்', 1, N'{"status": "pending"}'),
        (1, 5, N'முத்தமிழ் பிரிவுகள்', 2, N'{"status": "pending"}'),
        (1, 11, N'ஐ-உருபு வாக்கியங்கள்', 3, N'{"status": "pending"}'),
        (1, 1, N'ல,ள,ழ ஒலிப்புப் பயிற்சி', 4, N'{"status": "pending"}'),

        (2, 4, N'முக்கனிகளை அறிவோம்', 1, N'{"status": "pending"}'),
        (2, 11, N'ஆல்-உருபு வாக்கியங்கள்', 2, N'{"status": "pending"}'),
        (2, 15, N'உழவர் - வாசித்துப் புரிதல்', 3, N'{"status": "pending"}'),

        (3, 11, N'கு-உருபு வாக்கியங்கள்', 1, N'{"status": "pending"}'),
        (3, 4, N'ஒத்தபொருட் சொற்கள்', 2, N'{"status": "pending"}'),
        (3, 4, N'வினைமுற்று பொருத்தம்', 3, N'{"status": "pending"}'),

        (4, 15, N'ஒற்றுமை கதை', 1, N'{"status": "pending"}'),
        (4, 4, N'எதிர்ப்பொருட் சொற்கள்', 2, N'{"status": "pending"}'),
        (4, 12, N'நொடி-விடுகதை', 3, N'{"status": "pending"}'),

        (5, 4, N'வரைபடம் பொருத்துதல்', 1, N'{"status": "pending"}'),
        (5, 9, N'வாக்கியங்களை வரிசைப்படுத்து', 2, N'{"status": "pending"}'),
        (5, 6, N'பொருந்தாததை நீக்கு', 3, N'{"status": "pending"}'),

        (6, 16, N'நாடகத்தை வரிசைப்படுத்து', 1, N'{"status": "pending"}'),
        (6, 5, N'இடம் அறிதல் (மேலே, கீழே)', 2, N'{"status": "pending"}'),
        (6, 12, N'நொடி-விடுகதை', 3, N'{"status": "pending"}'),

        (7, 2, N'நாட்டார் பாடல்களை அறிவோம்', 1, N'{"status": "pending"}'),
        (7, 4, N'தொழிலும் கருவியும்', 2, N'{"status": "pending"}'),
        (7, 11, N'இல்-உருபு வாக்கியங்கள்', 3, N'{"status": "pending"}'),

        (8, 2, N'திருவள்ளுவர் வண்ணம் தீட்டு', 1, N'{"status": "pending"}'),
        (8, 1, N'குறள் ஒலிப்புப் பயிற்சி', 2, N'{"status": "pending"}'),
        (8, 8, N'குறள் சொற்களை உருவாக்குக', 3, N'{"status": "pending"}'),

        (9, 4, N'சரியான தலையைப் பொருத்து', 1, N'{"status": "pending"}'),
        (9, 7, N'விடுபட்ட எழுத்துக்கள்', 2, N'{"status": "pending"}'),
        (9, 12, N'நொடி-விடுகதை', 3, N'{"status": "pending"}'),

        (10, 4, N'உலக merikalai poruttuka', 1, N'{"status": "pending"}'),
        (10, 10, N'சேர்த்து எழுதுவோம்', 2, N'{"status": "pending"}'),
        (10, 15, N'கதை நிகழ்வு வரிசை', 3, N'{"status": "pending"}'),
        
        (11, 4, N'எதிர்மறைப் படங்களைப் பொருத்துக', 1, N'{"status": "pending"}'),
        (11, 5, N'இலக்கண வகைப்படுத்தல்', 2, N'{"status": "pending"}'),
        (11, 15, N'பூங்கா - வாசித்துப் புரிதல்', 3, N'{"status": "pending"}'),

        (12, 4, N'விளையாட்டும் கருவியும்', 1, N'{"status": "pending"}'),
        (12, 7, N'வினா வாக்கியம் அமைத்தல்', 2, N'{"status": "pending"}'),
        (12, 5, N'காலம் வகைப்படுத்தல்', 3, N'{"status": "pending"}');

    -- Insert using the dynamically fetched LessonIds
    INSERT INTO Activities (LessonId, ActivityTypeId, Title, ActivitySequence, ContentJson)
    SELECT 
        l.LessonId, s.ActivityTypeId, s.Title, s.ActivitySequence, s.ContentJson
    FROM @ActivitiesToSeed s
    JOIN @LessonsInLevel l ON s.LessonSequence = l.SequenceOrder
    WHERE NOT EXISTS (
        SELECT 1 FROM Activities a WHERE a.LessonId = l.LessonId AND a.SequenceOrder = s.ActivitySequence
    );

    PRINT 'Seeding for "ஆண்டு 03" completed successfully.';
    COMMIT TRANSACTION;

END TRY
BEGIN CATCH
    ROLLBACK TRANSACTION;
    PRINT 'An error occurred during seeding for "ஆண்டு 03". Transaction rolled back.';
    THROW;
END CATCH
GO
```
Grade 3 is now mapped and seeded. The complexity has clearly increased, with a strong focus on applying grammatical rules within sentences.

Let me know when you are ready to proceed with `ஆண்டு 04` (Grade 4).

### USER :
lets proceed grade 4

### RESPONSE :
Of course. Let's proceed with `ஆண்டு 04` (Grade 4).

This level solidifies all the major grammatical concepts in Tamil: `திணை` (class), `பால்` (gender), `எண்` (number), `இடம்` (person), and `காலம்` (tense). The activities are designed to test the student's ability to correctly classify words and sentences according to these rules.

### **Mapping Activities for `ஆண்டு 04` (Grade 4)**

We'll assume `ஆண்டு 04` has `LevelId = 6`.

| Lesson Name | Mapped Activities |
| :--- | :--- |
| **பாடம் 01 & 02: மொழிப்பயிற்சி**| 1. **TrueOrFalse** (`ActivityTypeId=14`): Statements about Tamil language.<br>2. **Matching** (`ActivityTypeId=4`): Match `முப்பால், முத்தமிழ்` to their components.<br>3. **FillInTheBlanks** (`ActivityTypeId=7`): Complete the Thirukkural verse.<br>4. **Sorting/Classification** (`ActivityTypeId=5`): Sort vowels into `குறில்/நெடில்`.<br>5. **Sorting/Classification** (`ActivityTypeId=5`): Sort consonants into `வல்லினம்/மெல்லினம்/இடையினம்`. |
| **பாடம் 03: கல்வியின் சிறப்பு**| 1. **FillInTheBlanks** (`ActivityTypeId=7`): Complete sentences about education.<br>2. **FillInTheBlanks** (`ActivityTypeId=7`): Complete the Thirukkural verse.<br>3. **Sorting/Classification** (`ActivityTypeId=5`): Sort words into `உயர்திணை/அஃறிணை`. |
| **பாடம் 04: பால் (Gender)**| 1. **Sorting/Classification** (`ActivityTypeId=5`): Sort words into the 5 `பால்` categories. |
| **பாடம் 05: தொடர்பாடல்** | 1. **Matching** (`ActivityTypeId=4`): Match inventors to their inventions.<br>2. **FillInTheBlanks** (`ActivityTypeId=7`): Complete the Thirukkural on speech.<br>3. **Audio-Image Recog.** (`ActivityTypeId=2`): Identify different modes of communication. |
| **பாடம் 06: ஆடிப்பிறப்பு** | 1. **SentenceScramble** (`ActivityTypeId=9`): Reorder sentences about Aadi Pirappu.<br>2. **Sorting/Classification** (`ActivityTypeId=5`): Sort words into `பெயர்ச்சொல்` vs `வினைச்சொல்`. |
| **பாடம் 07: நூலகம்** | 1. **TrueOrFalse** (`ActivityTypeId=14`): Statements about the library.<br>2. **FillInTheBlanks** (`ActivityTypeId=7`): Complete the Thirukkural verse.<br>3. **Matching** (`ActivityTypeId=4`): Match synonyms and antonyms. |
| **பாடம் 08: இசைக்கருவிகள்**| 1. **FillInTheBlanks** (`ActivityTypeId=7`): Complete sentences about musical instruments.<br>2. **Sorting/Classification** (`ActivityTypeId=5`): Classify instruments into the 4 types. |
| **பாடம் 09: திருகோணமலை**| 1. **FillInTheBlanks** (`ActivityTypeId=7`): Complete a passage about Trincomalee.<br>2. **Audio-Image Recog.** (`ActivityTypeId=2`): Identify landmarks of Trincomalee.<br>3. **TrueOrFalse** (`ActivityTypeId=14`): Statements about the location. |
| **பாடம் 10: குளிர்காலம்** | 1. **FillInTheBlanks** (`ActivityTypeId=7`): Complete sentences about winter.<br>2. **Matching** (`ActivityTypeId=4`): Match cause and effect phrases related to winter.<br>3. **WordFormation** (`ActivityTypeId=10`): Practice `சேர்த்து/பிரித்து எழுதுக`. |
| **பாடம் 11: உண்மையின் உயர்வு**| 1. **FillInTheBlanks** (`ActivityTypeId=7`): Complete sentences based on the story.<br>2. **WordScramble** (`ActivityTypeId=8`): Unscramble vocabulary from the story.<br>3. **WordScramble** (`ActivityTypeId=8`): Unscramble words with complex letters (`ழ/ள/ல`, `ர/ற`, `ன/ண/ந`). |
| **பாடம் 12: நடுகல்** | 1. **FillInTheBlanks** (`ActivityTypeId=7`): Complete sentences about hero stones.<br>2. **MultipleChoiceQuestion** (`ActivityTypeId=13`): Choose the correct `ஒருமை/பன்மை` form.<br>3. **WordScramble** (`ActivityTypeId=8`): Unscramble key vocabulary. |

---

### **SQL `INSERT` Script for `ஆண்டு 04` Activities**

This script populates the `Activities` table for the entire Grade 4 level.

```sql
-- ===================================================================
-- SCRIPT TO SEED 'Activities' TABLE FOR 'ஆண்டு 04' (Grade 4)
-- ===================================================================

BEGIN TRANSACTION;

BEGIN TRY
    DECLARE @LevelId INT;
    SELECT @LevelId = LevelId FROM Levels WHERE LevelName = N'ஆண்டு 04';

    IF @LevelId IS NULL
    BEGIN
        PRINT 'Error: Level "ஆண்டு 04" not found. Seeding cannot continue.';
        ROLLBACK TRANSACTION;
        RETURN;
    END
    
    DECLARE @LessonsInLevel TABLE (LessonId INT, SequenceOrder INT);
    INSERT INTO @LessonsInLevel SELECT LessonId, SequenceOrder FROM Lessons WHERE LevelId = @LevelId;

    PRINT 'Seeding activities for "ஆண்டு 04"...';

    DECLARE @ActivitiesToSeed TABLE (
        LessonSequence INT, ActivityTypeId INT, Title NVARCHAR(250), ActivitySequence INT, ContentJson NVARCHAR(MAX)
    );

    INSERT INTO @ActivitiesToSeed (LessonSequence, ActivityTypeId, Title, ActivitySequence, ContentJson)
    VALUES
        (1, 14, N'தமிழ்மொழி - சரி/பிழை', 1, N'{"status": "pending"}'),
        (1, 4, N'மொழிப் பாகங்களைப் பொருத்துக', 2, N'{"status": "pending"}'),
        (1, 7, N'குறளை நிரப்புக', 3, N'{"status": "pending"}'),
        (1, 5, N'குறில்/நெடில் வகைப்படுத்து', 4, N'{"status": "pending"}'),
        (1, 5, N'மெய்யெழுத்து வகைப்படுத்து', 5, N'{"status": "pending"}'),

        (2, 7, N'கல்வியின் சிறப்பு - வாக்கியம் நிரப்பல்', 1, N'{"status": "pending"}'),
        (2, 7, N'குறளை நிரப்புக (மணற்கேணி)', 2, N'{"status": "pending"}'),
        (2, 5, N'திணை வகைப்படுத்துக', 3, N'{"status": "pending"}'),

        (3, 5, N'பால் வகைப்படுத்துக', 1, N'{"status": "pending"}'),

        (4, 4, N'கண்டுபிடிப்பும் கண்டுபிடிப்பாளரும்', 1, N'{"status": "pending"}'),
        (4, 7, N'குறளை நிரப்புக (இன்சொல்)', 2, N'{"status": "pending"}'),
        (4, 2, N'தொடர்பாடல் முறைகள்', 3, N'{"status": "pending"}'),

        (5, 9, N'வாக்கியங்களை ஒழுங்குபடுத்துக', 1, N'{"status": "pending"}'),
        (5, 5, N'பெயர்ச்சொல் / வினைச்சொல்', 2, N'{"status": "pending"}'),

        (6, 14, N'நூலகம் - சரி/பிழை', 1, N'{"status": "pending"}'),
        (6, 7, N'குறளை நிரப்புக (கற்க கசடற)', 2, N'{"status": "pending"}'),
        (6, 4, N'ஒத்தபொருள் / எதிர்பொருள்', 3, N'{"status": "pending"}'),

        (7, 7, N'இசைக்கருவிகள் - வாக்கியம் நிரப்பல்', 1, N'{"status": "pending"}'),
        (7, 5, N'இசைக்கருவிகளை வகைப்படுத்துக', 2, N'{"status": "pending"}'),

        (8, 7, N'திருகோணமலை - பந்தி நிரப்பல்', 1, N'{"status": "pending"}'),
        (8, 2, N'திருகோணமலை இடங்கள்', 2, N'{"status": "pending"}'),
        (8, 14, N'திருகோணமலை - சரி/பிழை', 3, N'{"status": "pending"}'),

        (9, 7, N'குளிர்காலம் - வாக்கியம் நிரப்பல்', 1, N'{"status": "pending"}'),
        (9, 4, N'காரண காரியம் பொருத்துக', 2, N'{"status": "pending"}'),
        (9, 10, N'சேர்த்து / பிரித்து எழுதுக', 3, N'{"status": "pending"}'),

        (10, 7, N'உண்மையின் உயர்வு - வாக்கியம் நிரப்பல்', 1, N'{"status": "pending"}'),
        (10, 8, N'கதைச் சொற்களை உருவாக்குக', 2, N'{"status": "pending"}'),
        (10, 8, N'மயங்கொலிச் சொற்கள்', 3, N'{"status": "pending"}'),

        (11, 7, N'நடுகல் - வாக்கியம் நிரப்பல்', 1, N'{"status": "pending"}'),
        (11, 13, N'ஒருமை / பன்மை அறிதல்', 2, N'{"status": "pending"}'),
        (11, 8, N'முக்கிய சொற்களை உருவாக்குக', 3, N'{"status": "pending"}');

    -- Insert using the dynamically fetched LessonIds
    INSERT INTO Activities (LessonId, ActivityTypeId, Title, ActivitySequence, ContentJson)
    SELECT 
        l.LessonId, s.ActivityTypeId, s.Title, s.ActivitySequence, s.ContentJson
    FROM @ActivitiesToSeed s
    JOIN @LessonsInLevel l ON s.LessonSequence = l.SequenceOrder
    WHERE NOT EXISTS (
        SELECT 1 FROM Activities a WHERE a.LessonId = l.LessonId AND a.SequenceOrder = s.ActivitySequence
    );

    PRINT 'Seeding for "ஆண்டு 04" completed successfully.';
    COMMIT TRANSACTION;

END TRY
BEGIN CATCH
    ROLLBACK TRANSACTION;
    PRINT 'An error occurred during seeding for "ஆண்டு 04". Transaction rolled back.';
    THROW;
END CATCH
GO

```

Grade 4 is now fully mapped and seeded. The curriculum is clearly reinforcing core grammatical structures through classification and rule-based exercises.

We have one final level to go. Let me know when you're ready for `ஆண்டு 05`.

### USER :
lets proceed to grade 5

### RESPONSE : 
Excellent. Let's complete the final level, `ஆண்டு 05` (Grade 5).

This level serves as a culmination of all previous learning. The activities are less about introducing new concepts and more about applying existing knowledge to complex scenarios, reading comprehension, and understanding nuanced vocabulary like idioms and compound words.

### **Mapping Activities for `ஆண்டு 05` (Grade 5)**

We'll assume `ஆண்டு 05` has `LevelId = 7`.

| Lesson Name | Mapped Activities |
| :--- | :--- |
| **பாடம் 01: தமிழர் கலைகள்**| 1. **Matching** (`ActivityTypeId=4`): Match art forms to their names/images.<br>2. **Sorting/Classification** (`ActivityTypeId=5`): Sort words into `பெயர்ச்சொல்` vs `வினைச்சொல்`. |
| **பாடம் 02: ஒழுக்கம்** | 1. **MultipleChoiceQuestion** (`ActivityTypeId=13`): Choose the correct word to complete a sentence about discipline.<br>2. **FillInTheBlanks** (`ActivityTypeId=7`): Use the correct conjunctions (`ஆனால், ஆகவே, ...`). |
| **பாடம் 03: நட்பு** | 1. **FillInTheBlanks** (`ActivityTypeId=7`): Complete Thirukkural verses on friendship.<br>2. **WordScramble (Grid)** (`ActivityTypeId=8`): Find `உயர்திணை/அஃறிணை` words in a word search grid. |
| **பாடம் 04: பாரதியார்** | 1. **Sorting/Classification** (`ActivityTypeId=5`): Sort items into "ஆளுமைகள்" (Personalities) vs "ஆக்கங்கள்" (Creations).<br>2. **Sorting/Classification** (`ActivityTypeId=5`): Differentiate between `பலர்பால்` and `பலவின்பால்`. |
| **பாடம் 05: இளங்கோ அடிகள்** | 1. **MultipleChoiceQuestion** (`ActivityTypeId=13`): Comprehension questions about Ilango Adigal.<br>2. **StorySequencing** (`ActivityTypeId=16`): Reorder sentences to tell the story correctly. |
| **பாடம் 06: செய்தித்தாள்** | 1. **Audio-Image Recog.** (`ActivityTypeId=2`): Identify different types of media.<br>2. **Sorting/Classification** (`ActivityTypeId=5`): Identify and color-code `எழுவாய், செயப்படுபொருள், பயனிலை` (Subject, Object, Verb). |
| **பாடம் 07: தைத்திருநாள்** | 1. **Matching** (`ActivityTypeId=4`): Match items related to Pongal.<br>2. **Matching** (`ActivityTypeId=4`): Match proverbs to their meanings. |
| **பாடம் 08: பவளக்கொடி** | 1. **ReadingComprehension** (`ActivityTypeId=15`): Find compound words (`இணைமொழிகள்`) within a passage.<br>2. **Matching** (`ActivityTypeId=4`): Match collective nouns (`மரபுப்பெயர்கள்`) like `ஆட்டு மந்தை`. |
| **பாடம் 09: மன்னார்** | 1. **FillInTheBlanks** (`ActivityTypeId=7`): Complete a passage about Mannar.<br>2. **SentenceScramble** (`ActivityTypeId=9`): Reorder sentences about the location. |
| **பாடம் 10: நோயற்ற வாழ்வு** | 1. **Matching** (`ActivityTypeId=4`): Match grains to their names.<br>2. **Matching** (`ActivityTypeId=4`): Match idioms (`மரபுத்தொடர்கள்`) to their meanings. |
| **பாடம் 11: மாவீரம்** | 1. **Matching** (`ActivityTypeId=4`): Match historical figures to their descriptions.<br>2. **Sorting/Classification** (`ActivityTypeId=5`): Sort sentences into positive (`உடன்பாடு`) vs negative (`எதிர்மறை`). |
| **பாடம் 12: ஒலிம்பிக்** | 1. **Matching** (`ActivityTypeId=4`): Match Olympic terms to their meanings.<br>2. **MultipleChoiceQuestion** (`ActivityTypeId=13`): Choose the correct case-marked noun to fill in a sentence (`மரத்துக்கு, மரத்தை...`). |

---

### **SQL `INSERT` Script for `ஆண்டு 05` Activities**

This script populates the `Activities` table for the final Grade 5 level.

```sql
-- ===================================================================
-- SCRIPT TO SEED 'Activities' TABLE FOR 'ஆண்டு 05' (Grade 5)
-- ===================================================================

BEGIN TRANSACTION;

BEGIN TRY
    DECLARE @LevelId INT;
    SELECT @LevelId = LevelId FROM Levels WHERE LevelName = N'ஆண்டு 05';

    IF @LevelId IS NULL
    BEGIN
        PRINT 'Error: Level "ஆண்டு 05" not found. Seeding cannot continue.';
        ROLLBACK TRANSACTION;
        RETURN;
    END
    
    DECLARE @LessonsInLevel TABLE (LessonId INT, SequenceOrder INT);
    INSERT INTO @LessonsInLevel SELECT LessonId, SequenceOrder FROM Lessons WHERE LevelId = @LevelId;

    PRINT 'Seeding activities for "ஆண்டு 05"...';

    DECLARE @ActivitiesToSeed TABLE (
        LessonSequence INT, ActivityTypeId INT, Title NVARCHAR(250), ActivitySequence INT, ContentJson NVARCHAR(MAX)
    );

    INSERT INTO @ActivitiesToSeed (LessonSequence, ActivityTypeId, Title, ActivitySequence, ContentJson)
    VALUES
        (1, 4, N'தமிழர் கலைகளைப் பொருத்துக', 1, N'{"status": "pending"}'),
        (1, 5, N'பெயர்ச்சொல், வினைச்சொல் வகைப்படுத்து', 2, N'{"status": "pending"}'),

        (2, 13, N'ஒழுக்கம் - சரியான சொல்லைத் தெரிவுசெய்', 1, N'{"status": "pending"}'),
        (2, 7, N'இணைப்புச் சொற்களை நிரப்புக', 2, N'{"status": "pending"}'),

        (3, 7, N'நட்பு - குறளை நிரப்புக', 1, N'{"status": "pending"}'),
        (3, 8, N'சொற்கட்டத்திலிருந்து சொல் தேடல்', 2, N'{"status": "pending"}'),

        (4, 5, N'பாரதியார் - ஆளுமையும் ஆக்கங்களும்', 1, N'{"status": "pending"}'),
        (4, 5, N'பலர்பால், பலவின்பால் வேறுபாடு', 2, N'{"status": "pending"}'),

        (5, 13, N'இளங்கோ அடிகள் - பலவுள் தெரிவு', 1, N'{"status": "pending"}'),
        (5, 16, N'கதையை வரிசைப்படுத்துக', 2, N'{"status": "pending"}'),

        (6, 2, N'ஊடகங்களை அடையாளம் காண்க', 1, N'{"status": "pending"}'),
        (6, 5, N'எழுவாய், செயப்படுபொருள், பயனிலை', 2, N'{"status": "pending"}'),

        (7, 4, N'தைத்திருநாள் - பொருத்துக', 1, N'{"status": "pending"}'),
        (7, 4, N'பழமொழியும் பொருளும்', 2, N'{"status": "pending"}'),

        (8, 15, N'இணைமொழிகளைக் கண்டறிக', 1, N'{"status": "pending"}'),
        (8, 4, N'மரபுப்பெயர்களைப் பொருத்துக', 2, N'{"status": "pending"}'),

        (9, 7, N'மன்னார் - பந்தி நிரப்பல்', 1, N'{"status": "pending"}'),
        (9, 9, N'வாக்கியங்களை ஒழுங்குபடுத்துக', 2, N'{"status": "pending"}'),

        (10, 4, N'சிறு தானியங்களைப் பொருத்துக', 1, N'{"status": "pending"}'),
        (10, 4, N'மரபுத்தொடரும் பொருளும்', 2, N'{"status": "pending"}'),
        
        (11, 4, N'மாவீரர்களைப் பொருத்துக', 1, N'{"status": "pending"}'),
        (11, 5, N'உடன்பாடு / எதிர்மறை வாக்கியம்', 2, N'{"status": "pending"}'),

        (12, 4, N'ஒலிம்பிக் - பொருத்துக', 1, N'{"status": "pending"}'),
        (12, 13, N'சரியான வேற்றுமை உருபு', 2, N'{"status": "pending"}');

    -- Insert using the dynamically fetched LessonIds
    INSERT INTO Activities (LessonId, ActivityTypeId, Title, ActivitySequence, ContentJson)
    SELECT 
        l.LessonId, s.ActivityTypeId, s.Title, s.ActivitySequence, s.ContentJson
    FROM @ActivitiesToSeed s
    JOIN @LessonsInLevel l ON s.LessonSequence = l.SequenceOrder
    WHERE NOT EXISTS (
        SELECT 1 FROM Activities a WHERE a.LessonId = l.LessonId AND a.SequenceOrder = s.ActivitySequence
    );

    PRINT 'Seeding for "ஆண்டு 05" completed successfully.';
    COMMIT TRANSACTION;

END TRY
BEGIN CATCH
    ROLLBACK TRANSACTION;
    PRINT 'An error occurred during seeding for "ஆண்டு 05". Transaction rolled back.';
    THROW;
END CATCH
GO
```

**Congratulations!**

We have now successfully mapped and created the seeding scripts for all levels, from Preschool to Grade 5. Your database is now fully structured to hold the entire curriculum's activities. The next major step is to start populating the `ContentJson` field for each activity, which will drive the UI in your Flutter app. This provides a clear and organized task list for your content creation team.

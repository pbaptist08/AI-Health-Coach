

## How Lovable/Bolt + Supabase + LLM fit together

### 3.1 Tech roles

- **Supabase**
  - Auth (users sign in).
  - Postgres DB (tables for everything above).
  - Optional: Row-level security.

- **Lovable/Bolt (frontend + basic backend)**
  - UI screens:
    - AI Coach onboarding.
    - Program overview (week view).
    - Today’s Workout.
    - Program settings.
  - API routes / serverless functions:
    - `/api/generate-training-profile` (calls LLM).
    - `/api/generate-program` (rule-based engine on top of TrainingProfile).
    - `/api/log-set` (inserts WorkoutSetLog).

- **LLM (via OpenAI API or similar)**
  - Only used for:
    - Turning onboarding answers → TrainingProfile.
    - Later: other AI features (weekly review, RAG, etc.).

### Flow of data in MVP1

1. **User signs up / logs in**
   - Supabase creates a `User`.
   - Optionally create an empty `UserProfile`.

2. **AI Coach onboarding**
   - Frontend (Lovable UI) shows chat-like flow / stepped form.
   - When user clicks “Build my plan”:
     - Call `/api/generate-training-profile` with all answers.
     - That route:
       - Calls LLM with a prompt + the user answers.
       - Receives TrainingProfile JSON.
       - Validates + saves it to `training_profiles` table.
       - Returns TrainingProfile to frontend.

3. **Program generation**
   - Frontend calls `/api/generate-program` with TrainingProfile ID (or inline JSON).
   - That route:
     - Runs your **deterministic Program Engine**:
       - Chooses split.
       - Creates Program row.
       - Creates ProgramWeekTemplate + ProgramDayTemplate + ProgramDayExerciseTemplate rows.
     - Returns the generated week template to frontend.

4. **Program overview UI**
   - Frontend reads `program_day_templates` & `program_day_exercise_templates` from Supabase (via Supabase client).
   - Shows a weekly calendar or simple list (Mon: Push, Tue: Pull, etc.).

5. **Today’s Workout**
   - Frontend:
     - Determines “today” (e.g., Monday).
     - Fetches the matching `ProgramDayTemplate` + its exercises.
     - Renders each exercise + sets.
   - As user logs sets:
     - Calls `/api/log-set` or writes via Supabase client to `workout_sessions` & `workout_set_logs`.

6. **Change Today’s Focus**
   - Option A (switch to another planned day):
     - Just fetch another `ProgramDayTemplate` for this `Program` & `weekday`.
   - Option B (choose muscles):
     - For MVP: a small local generator that:
       - Picks 4–6 exercises from `ExerciseLibrary` matching chosen muscles + equipment.
       - Creates a `WorkoutSession` without tying it to a `ProgramDayTemplate` (ad-hoc).
     - You don’t need LLM for this part initially; it can be rule-based.

7. **Program Settings & Replan**
   - User changes days per week / goals.
   - Frontend sends updates to `/api/generate-training-profile` again (shorter prompt).
   - That returns a new TrainingProfile.
   - Call `/api/generate-program` again to create a new Program that starts next week.

---


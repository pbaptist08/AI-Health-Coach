# FitMind Strength – AI Coach MVP (Part 1)

> An AI strength coach that builds a realistic lifting program around your life and makes logging sets/reps/weights fast and tap-first, so serious-but-busy lifters can actually follow a plan.

---

## Overview

FitMind Strength is a **mobile-first strength training app** for serious-but-busy knowledge workers.

MVP1 focuses on:

- An **AI coach** that turns your goals, schedule, and constraints into a structured training profile.
- A deterministic **program engine** that generates a weekly lifting split (1–7 days/week) and specific workouts.
- A **“Today’s Workout”** screen optimized for tap-first logging (minimal typing).
- Ability to **change today’s focus** (e.g., hit back instead of push) without breaking the program.
- A **curated exercise library** with search & swap (no custom exercises in MVP1).
- Support for **kg/lb** units.

Planned for later (not in MVP1):

- Email reminders.
- Weekly summary / muscle-group coverage UI.
- Exercise/programming Q&A with RAG.
- Custom user-defined exercises.

---

## Target Persona

**Persona:** *Serious-but-busy lifter* (Prashanth proxy)

- Age: 27–45  
- Role: Knowledge worker (PM / engineer / consultant / data, etc.)
- Training:
  - Wants to lift **1–7 days/week** (typically 3–5).
  - Prefers structured strength training (Push/Pull/Legs, Upper/Lower, full body).
  - Not a “time-pass gym-goer” – cares about progression and coverage.
- Goals:
  - Lose fat, build strength and muscle, look and feel athletic again.
- Constraints:
  - Busy schedule, meetings, family, travel.
  - Hates typing exercise names / sets / reps into a phone mid-workout.
  - Doesn’t want to overthink “what should I train today?” every session.

---

## Job To Be Done

**Primary JTBD**

> When I’m at the gym and trying to follow a structured lifting routine, I want an AI coach to design a plan around my schedule and preferences, tell me what to train today, and let me log sets/reps/weights quickly, so I can stay consistent without wasting time on planning or typing.

**Supporting Jobs**

- Express my goals and constraints in normal language and have the app turn that into a training plan.
- Set a realistic training frequency (1–7 days/week) and get a sensible split.
- Know exactly what today’s workout is when I open the app.
- Override today’s planned split/muscle if my mood or energy is different.
- Log sets/reps/weights with minimal friction.
- Use kg or lb depending on where I train.

---

## Key Pain Points & How We Solve Them

### 1. “Forms don’t feel like a coach”

**Pain:** Plain forms feel dry. User wants an AI *coach*, not just a wizard of dropdowns.

**Solution: AI Coach Onboarding**

- Chat-like onboarding where user can:
  - Tap chips **or** type free-text:
    - Goals (fat loss, strength, hypertrophy, general fitness)
    - Days per week they can train (1–7)
    - Time per session (30–75 minutes)
    - Experience level (beginner, intermediate, returning)
    - Available equipment
    - Problem areas / focus muscles
    - Constraints (injuries, movement dislikes)
- LLM converts this into a structured **TrainingProfile** JSON + a short **“why this plan”** explanation.
- Deterministic program engine uses TrainingProfile to build the actual weekly template.

---

### 2. “Typing sets/reps/exercise names is painful”

**Pain:** Tracking sets/reps is tedious; typing exercise names and muscle groups on a phone sucks.

**Solution: Pre-defined workouts + tap-first logging**

- Program engine outputs:
  - Named exercises  
  - Muscle tags  
  - Planned sets & rep ranges
- **Today’s Workout**:
  - Exercises are already listed.
  - For each set:
    - Weight pre-filled with last used (or default)
    - Reps pre-filled with target
    - User adjusts with +/- and taps “Done”.
- User never types:
  - Muscle groups  
  - Exercise names  
  - Sets/reps text strings

---

### 3. “I don’t want to think what to train or how many days”

**Pain:** Decision fatigue picking splits and muscle groups (“Is 4 or 5 days better?”, “Push or legs today?”).

**Solution: AI + Program Engine**

- AI Coach creates `TrainingProfile` with:
  - `days_per_week` (1–7)
  - `split_type`
  - `session_length_minutes`
  - `goal_focus`
  - `experience_level`
  - `available_equipment`
  - `constraints`
  - `priority_muscles`
- Program Engine:
  - Maps `days_per_week` → sensible splits:
    - 1: Full body  
    - 2: Upper / Lower  
    - 3: Full body or Push/Pull/Legs  
    - 4: ULUL or PPL + Upper  
    - 5–7: PPL + specialization / repeated cycles
  - Fills each day with 4–7 exercises from the curated library.
- **Override Today**:
  - In “Today’s Workout”, button: **“Change today’s focus”**:
    - Use another planned day this week (e.g., switch to Legs).
    - Or choose muscle groups directly (e.g., Back + Biceps) → generate a new workout for today.

---

### 4. “My life changes; my program should too”

**Pain:** Program is static; if my schedule changes, I need a new plan.

**Solution: Program Settings + AI Re-planning**

- **Program Settings** shows:
  - Current training days/week
  - Current split description (“5-day Push/Pull/Legs + Upper/Lower”).
- User can:
  - Change `days_per_week` (1–7)
  - Optionally tweak goals/constraints
- Clicking **“Rebuild my plan with coach”**:
  - Runs a shorter AI Coach flow
  - LLM outputs a new TrainingProfile
  - Program Engine generates a new weekly template
- MVP1 behavior:
  - Current week stays unchanged
  - New plan starts from **next week**

---

### 5. “Exercise choices are overwhelming”

**Pain:** User wants some control (swap exercises, pick preferred movements) without defining everything themselves.

**Solution: Curated exercise library + search & swap**

- Predefined exercise library with metadata:
  - `id`, `name`, `primary_muscle`, optional `secondary_muscles`, `equipment`
- In “Today’s Workout”, each exercise has **“Swap exercise”**.
- Swap modal:
  - Search by name (“row”, “curl”, “incline press”)
  - Filter by:
    - Muscle group (Chest, Back, Quads, etc.)
    - Equipment (Dumbbell, Barbell, Machine, Cable, Bodyweight)
- User selects a replacement; sets/reps pattern is preserved or lightly adapted.
- No custom user-defined exercises in MVP1.

---

### 6. “Kg vs lb”

**Pain:** Units differ by gym and country; mixing them is confusing.

**Solution: Profile-level units**

- Onboarding / Settings:
  - “Weight units: kg / lb”
- All weight fields:
  - Respect this preference in UI
- Storage:
  - Store `weight_value` + `weight_unit`
  - (Conversion on unit change can be refined later.)

---

## High-Level Architecture (Conceptual)

MVP1 separates **AI understanding** from **program generation**:

1. **AI Layer – TrainingProfile Generator**
   - Input: User Q&A (chips + text)
   - Output: `TrainingProfile` JSON + optional plan explanation

2. **Program Engine (Deterministic)**
   - Input: TrainingProfile + exercise library
   - Output: Weekly training template with days, muscle focus, and exercises

3. **UI & Logging**
   - Today’s Workout (shows a day from the template)
   - Override focus for today
   - Log sets/reps/weights, respecting unit preference

---

## Functional Specs (MVP1)

### 1. TrainingProfile (AI Output)

**Example schema:**

```json
{
  "days_per_week": 5,
  "split_type": "PPL+UL",
  "session_length_minutes": 60,
  "goal_focus": ["fat_loss", "strength"],
  "experience_level": "intermediate",
  "style_preference": "hypertrophy_bias",
  "available_equipment": ["commercial_gym"],
  "constraints": [
    "no_high_impact_cardio",
    "avoid_heavy_overhead_pressing"
  ],
  "priority_muscles": ["back", "legs"]
}

### LLM Contract (Conceptual)

- **Input:**
  - System prompt (e.g., “You are a strength coach…”)
  - User’s structured answers (chips) + free-text responses

- **Output:**
  - Valid JSON matching `TrainingProfile` schema
  - Optional `plan_explanation` string (plain text)

- **Guardrails:**
  - `days_per_week` in `[1, 7]`
  - Respect available equipment and stated constraints
  - Avoid obviously unsafe volume/intensity

---

### 2. Program Engine & Week Template

**Responsibility:** Turn `TrainingProfile` into a concrete weekly program.

**Steps:**

1. Choose split template based on:
   - `days_per_week`
   - `experience_level`
   - `goal_focus`
   - `split_type` (if provided)

```json
{
  "week_template": [
    {
      "weekday": "Mon",
      "label": "Push",
      "primary_muscles": ["chest", "shoulders", "triceps"],
      "exercises": [
        { "exercise_id": "bench_press", "sets": 4, "reps": "6-8" },
        { "exercise_id": "incline_db_press", "sets": 3, "reps": "8-10" }
      ]
    }
  ]
}

3. **Exercise selection**

- Use `primary_muscles`
- Respect `available_equipment`
- Use `priority_muscles` to bias extra volume
- Avoid exercises that conflict with `constraints`

4. **Sets & reps**

- Strength focus → more lower-rep compounds
- Hypertrophy focus → more mid-rep ranges (8–12)

---

### 3. Program Settings & Re-planning

**Features:**

- **View:**
  - Current days/week
  - Current split description

- **Change:**
  - `days_per_week` (1–7)
  - Optional goals/constraints tweaks via a short mini-flow

- **Rebuild:**
  - Trigger AI Coach again → new `TrainingProfile` → new `week_template`

- **Behavior:**
  - Old plan continues for the current week
  - New plan is applied from the next week

---

### 4. Today’s Workout & Logging

**UI:**

- **Header:**
  - `Today: Push Day`
  - `Focus: Chest · Shoulders · Triceps`

- **Exercises:**
  - Name, muscle tags, planned sets & reps

**Per set logging:**

- **Weight (kg/lb):**
  - Pre-filled (last used or default)
  - Adjust via +/- or small picker

- **Reps:**
  - Pre-filled with target
  - Adjust via +/- or small picker

- **Done:**
  - Checkbox / tap area to mark set as completed

**Data stored per set:**

- `exercise_id`
- `set_index`
- `weight_value`
- `weight_unit`
- `reps`
- `done` (boolean)
- `timestamp`

---

### 5. Change Today’s Focus

**Behavior:**

- Button on Today’s Workout: **“Change today’s focus”**

**Options:**

1. **Use another planned day** from this week’s template  
   Example: switch today’s session to this week’s Legs workout.

2. **Choose muscle groups directly**
   - User picks muscle tags (e.g., Back + Biceps)
   - Program engine (or helper) generates a reasonable workout for today:
     - Uses chosen muscles
     - Uses `available_equipment`
     - Keeps volume within a safe range

**Scope:**

- Override affects **today only**
- Underlying weekly template stays unchanged in MVP1

### 6. Exercise Library & Swap

**Data Model Example**

{
  "id": "bench_press",
  "name": "Barbell Bench Press",
  "primary_muscle": "chest",
  "secondary_muscles": ["triceps", "shoulders"],
  "equipment": "barbell"
}

**Swap Flow**

- In **Today’s Workout**:
  - Tap **“Swap exercise”** on an exercise card

- **Swap modal:**
  - Search by name
  - Filter by muscle group and equipment
  - Select replacement

- **The replacement:**
  - Uses the same sets/reps pattern or a close variant

**Constraints:**

- No user-defined exercises in MVP1
- Library should cover common gym movements across all major muscle groups

---

### 7. Units (kg/lb)

**Behavior:**

- **Onboarding / Settings:**
  - User chooses kg or lb

- **UI:**
  - All weights display in that unit

- **Storage:**
  - Store weight with an explicit unit (`weight_value`, `weight_unit`)
  - Conversion on unit change can be added later

---

## Out of Scope (MVP1)

These will be considered in later iterations (MVP Part 2+):

- Email reminders and notifications
- Weekly “coach summary” or muscle-volume analytics UI
- Exercise / programming Q&A with RAG
- Social features, sharing, or leaderboards
- Nutrition tracking (calories, macros, meals)
- Wearable integrations
- Custom exercises
- Advanced progression logic (periodization, deloads, auto-progression rules)

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


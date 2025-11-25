# AI-Health-Coach MVP (Part 1) PRD

#Product Summary

Working name: FitMind Strength
Tagline:

An AI strength coach that builds a realistic lifting program around your life and makes logging sets/reps/weights fast and tap-first, so serious-but-busy lifters can actually follow a plan.

MVP1 Scope (AI-based, locked):
	•	AI Coach onboarding & re-planning (conversational).
	•	LLM → structured TrainingProfile (JSON).
	•	Deterministic Program Engine → weekly template (split + exercises).
	•	Program Settings to change days/week and regenerate plan via AI.
	•	Today’s Workout screen with tap-first logging.
	•	Change today’s focus (switch planned day or choose muscles).
	•	Curated exercise library + search & swap (no custom exercises).
	•	Kg/lb units support.

Explicitly out of scope for MVP1 (planned for later):
	•	Email reminders on training days.
	•	Weekly summary / muscle-group coverage UI.
	•	Exercise / programming Q&A (RAG).
	•	Custom user-defined exercises.

#Target User Persona

Persona: “Serious-but-busy lifter” (Prashanth-proxy)
	•	Age: 27–45.
	•	Role: Knowledge worker (PM / engineer / consulting / data, etc.).
	•	Training pattern:
	•	Wants to lift 1–7 days/week (typically 3–5).
	•	Prefers structured strength training (splits like Push/Pull/Legs, Upper/Lower, full-body).
	•	Not a “time-pass gym-goer” – cares about progression and coverage.
	•	Goals:
	•	Lose fat, build strength and muscle, look and feel athletic again.
	•	Constraints:
	•	Busy schedule, meetings, family, travel.
	•	Hates typing exercise names / sets / reps into a phone mid-workout.
	•	Doesn’t want to overthink “what should I train today?” every time.

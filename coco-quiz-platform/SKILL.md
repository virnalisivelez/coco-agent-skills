---
name: coco-quiz-platform
description: Build production-style quiz platforms with persistent storage, scoring logic, and admin capabilities. Use when creating educational quizzes, knowledge assessments, trivia games, certification tests, or any interactive Q&A system with user authentication and score tracking.
license: Apache-2.0
metadata:
  author: good-stories-llc
  version: "1.0"
  brand: COCO
  website: https://goodstories.world
---

# Quiz Platform Builder

Build a complete quiz system with user authentication, persistent storage, weighted scoring, admin tools, and a polished player experience.

## When to Use

- User wants to build a quiz or trivia game
- User needs a knowledge assessment or certification tool
- User asks for an educational platform with scoring
- User wants a gamified learning experience
- User needs admin tools to manage question banks

## Architecture Overview

```
Authentication Layer → Quiz Engine → Scoring System → Results Dashboard
        |                  |              |                    |
   User Profiles    Question Bank    Score History        Leaderboard
        |                  |              |                    |
   Session Mgmt     Randomization    Weighted Logic      Analytics
```

## Module 1: User Authentication

### Registration & Login

```python
# User model
user = {
    "id": "unique-id",
    "username": "player1",
    "email": "user@example.com",
    "password_hash": "bcrypt-hash",
    "role": "player",           # "player" or "admin"
    "created_at": "2026-03-27",
    "stats": {
        "quizzes_taken": 0,
        "total_score": 0,
        "best_score": 0,
        "average_score": 0
    }
}
```

### Session Management

- Generate session tokens on login (JWT or UUID-based)
- Set expiry (e.g., 24 hours)
- Validate session on each request
- Handle expired sessions gracefully (redirect to login, preserve quiz state if mid-quiz)

### Security Checklist

- Hash passwords with bcrypt (never store plaintext)
- Rate-limit login attempts (5 per minute)
- Validate email format on registration
- Prevent duplicate registrations
- Sanitize all inputs

## Module 2: Question Bank

### Question Structure

```python
question = {
    "id": "q-001",
    "text": "What is the primary purpose of a CPU?",
    "options": [
        {"id": "a", "text": "Process instructions and manage data flow"},
        {"id": "b", "text": "Store permanent data"},
        {"id": "c", "text": "Render graphics"},
        {"id": "d", "text": "Connect to the internet"}
    ],
    "correct": "a",
    "difficulty": "medium",     # easy, medium, hard
    "category": "Technology",
    "points": 10,               # base points (modified by difficulty)
    "time_limit": 30,           # seconds
    "explanation": "CPUs execute program instructions and coordinate data between components."
}
```

### Category Management

Organize questions by topic and difficulty:

```python
categories = {
    "Technology": {"easy": 15, "medium": 20, "hard": 10},
    "Science": {"easy": 12, "medium": 18, "hard": 8},
    "Business": {"easy": 10, "medium": 15, "hard": 5}
}
```

## Module 3: Quiz Engine

### Session Configuration

```python
quiz_config = {
    "total_questions": 10,
    "difficulty_mix": {
        "easy": 3,      # 30%
        "medium": 5,     # 50%
        "hard": 2        # 20%
    },
    "time_per_question": 30,    # seconds
    "categories": ["Technology", "Science"],  # filter or "all"
    "shuffle_options": True,
    "prevent_repeats": True     # no duplicate questions in a session
}
```

### Randomization Logic

```python
import random

def select_questions(bank: list, config: dict, user_history: list) -> list:
    """Select randomized question set avoiding repeats."""
    available = [q for q in bank if q["id"] not in user_history]
    selected = []

    for difficulty, count in config["difficulty_mix"].items():
        pool = [q for q in available
                if q["difficulty"] == difficulty
                and q["category"] in config["categories"]]
        if len(pool) < count:
            # Fall back to any available at this difficulty
            pool = [q for q in available if q["difficulty"] == difficulty]
        chosen = random.sample(pool, min(count, len(pool)))
        selected.extend(chosen)
        for q in chosen:
            available.remove(q)

    random.shuffle(selected)
    return selected
```

### Timing Logic

```python
def score_question(question: dict, answer: str, time_taken: float) -> dict:
    """Score a single question with time bonus."""
    base_points = question["points"]
    difficulty_multiplier = {"easy": 1.0, "medium": 1.5, "hard": 2.0}
    multiplier = difficulty_multiplier.get(question["difficulty"], 1.0)

    correct = answer == question["correct"]

    if not correct:
        return {"correct": False, "points": 0, "time_taken": time_taken}

    # Time bonus: faster answers get up to 50% bonus
    time_limit = question.get("time_limit", 30)
    time_ratio = max(0, 1 - (time_taken / time_limit))
    time_bonus = base_points * 0.5 * time_ratio

    total = round((base_points * multiplier) + time_bonus)

    return {
        "correct": True,
        "points": total,
        "base_points": base_points,
        "time_bonus": round(time_bonus),
        "time_taken": round(time_taken, 1)
    }
```

## Module 4: Admin Capabilities

### Question CRUD

```python
# Admin endpoints / functions
def add_question(question: dict) -> bool:
    """Validate and add a question to the bank."""
    # Validation
    assert question.get("text"), "Question text required"
    assert len(question.get("options", [])) >= 2, "Minimum 2 options"
    assert question.get("correct") in [o["id"] for o in question["options"]], "Correct answer must match an option"
    assert question.get("difficulty") in ("easy", "medium", "hard"), "Invalid difficulty"
    # Save to storage
    return save_question(question)

def edit_question(question_id: str, updates: dict) -> bool:
    """Update existing question with validation."""
    existing = get_question(question_id)
    if not existing:
        return False
    merged = {**existing, **updates}
    return save_question(merged)

def delete_question(question_id: str) -> bool:
    """Remove question from bank."""
    return remove_question(question_id)
```

### Analytics Dashboard (Admin View)

```python
admin_stats = {
    "total_questions": 150,
    "by_difficulty": {"easy": 50, "medium": 70, "hard": 30},
    "by_category": {"Technology": 45, "Science": 40, "Business": 35, "General": 30},
    "most_missed": [  # questions with lowest correct rate
        {"id": "q-042", "text": "...", "correct_rate": 0.23},
    ],
    "average_completion_time": 245,  # seconds per quiz
    "total_attempts": 1250,
    "unique_players": 340
}
```

## Module 5: Results & Leaderboard

### Score Breakdown

```python
quiz_result = {
    "user_id": "user-123",
    "quiz_id": "quiz-456",
    "date": "2026-03-27",
    "score": 145,
    "max_possible": 200,
    "percentage": 72.5,
    "questions": 10,
    "correct": 7,
    "incorrect": 3,
    "average_time": 18.3,     # seconds per question
    "fastest_answer": 4.2,
    "by_difficulty": {
        "easy": {"correct": 3, "total": 3},
        "medium": {"correct": 3, "total": 5},
        "hard": {"correct": 1, "total": 2}
    }
}
```

### Leaderboard

```python
def get_leaderboard(period: str = "all", limit: int = 10) -> list:
    """Get top scores for a time period."""
    # period: "daily", "weekly", "monthly", "all"
    scores = fetch_scores(period=period)
    ranked = sorted(scores, key=lambda x: x["score"], reverse=True)
    return ranked[:limit]
```

## Module 6: Edge Cases & Reliability

Handle these scenarios:

- **Empty submission:** Require answer selection before advancing
- **Invalid question format:** Validate on admin input, reject malformed data
- **Expired session mid-quiz:** Save progress, restore on re-login
- **Duplicate registration:** Check email/username uniqueness, clear error message
- **Network errors:** Queue answers locally, sync when connection restores
- **Tie-breaking:** Use average time as tiebreaker on leaderboard
- **Question pool exhaustion:** Warn admin when category has fewer questions than quiz config requires

## Module 7: UI Requirements

- Clean layout with clear question numbering (3/10)
- Progress bar showing completion
- Timer countdown per question (visual, not just text)
- Immediate feedback: green for correct, red for incorrect, show explanation
- Final results: score breakdown, time analysis, comparison to average
- Leaderboard: top 10 with current user highlighted

## Storage Options

- **SQLite:** Good for single-server, embedded quizzes
- **Supabase:** Free tier with Postgres, auth, and real-time — recommended
- **localStorage:** Okay for prototypes, not for multi-user
- **JSON files:** Development only

## Scaling Considerations

- Separate question bank into categories for faster queries
- Cache leaderboard (recompute every 5 minutes, not on every request)
- Index scores by user_id and date for fast lookups
- Consider read replicas for high-traffic leaderboards

---
name: coco-beauty-consultant
description: Build modular beauty consultation systems that analyze skin profiles and generate personalized skincare and makeup routines. Use when creating beauty advisor tools, skincare recommendation engines, product matching systems, or personalized wellness platforms. Handles intake, ingredient reasoning, safety guardrails, and structured output.
license: Apache-2.0
metadata:
  author: good-stories-llc
  version: "1.0"
  brand: COCO
  website: https://goodstories.world
---

# Beauty Consultation System

Build a production-level digital beauty advisor that collects user profiles, analyzes skin characteristics, and generates personalized skincare and makeup routines with ingredient reasoning and safety guardrails.

## When to Use

- User wants to build a beauty or skincare recommendation tool
- User needs a personalized consultation system for wellness products
- User asks for ingredient analysis or product matching logic
- User wants to build a beauty brand's digital advisor
- User needs conditional logic for skincare routines (climate, sensitivity, skin type)

## Architecture Overview

```
Intake Layer → Personalization Engine → Ingredient Reasoning → Safety Guardrails → Output
     |                |                       |                      |              |
  Validation    Profile Analysis        Conflict Detection     Disclaimers    Routine Card
     |                |                       |                      |              |
  Normalization  Conditional Logic      Explanations           Allergy Flags   Export/Save
```

## Module 1: Structured Intake

### User Profile Schema

```python
profile = {
    "skin_type": "combination",        # dry, oily, combination, normal, sensitive
    "skin_tone": "medium",             # fair, light, medium, tan, deep
    "undertone": "warm",               # warm, cool, neutral, olive
    "primary_concerns": [
        "acne",                         # acne, aging, hyperpigmentation, dryness,
        "uneven_texture"                # redness, dark_circles, large_pores, dullness
    ],
    "climate": "humid_tropical",       # dry_cold, temperate, humid_tropical, arid_hot
    "sensitivities": ["fragrance"],    # fragrance, retinol, AHA, BHA, essential_oils, none
    "allergies": [],                    # specific ingredient allergies
    "budget": "mid",                   # budget, mid, premium, luxury
    "routine_length": "moderate",      # minimal (3 steps), moderate (5), comprehensive (7+)
    "age_range": "25-34"               # 18-24, 25-34, 35-44, 45-54, 55+
}
```

### Input Validation

```python
VALID_SKIN_TYPES = {"dry", "oily", "combination", "normal", "sensitive"}
VALID_CONCERNS = {"acne", "aging", "hyperpigmentation", "dryness", "redness",
                  "dark_circles", "large_pores", "dullness", "uneven_texture"}
VALID_CLIMATES = {"dry_cold", "temperate", "humid_tropical", "arid_hot"}

def validate_profile(profile: dict) -> list[str]:
    """Return list of validation errors, empty if valid."""
    errors = []
    if profile.get("skin_type") not in VALID_SKIN_TYPES:
        errors.append(f"Invalid skin type: {profile.get('skin_type')}")
    for concern in profile.get("primary_concerns", []):
        if concern not in VALID_CONCERNS:
            errors.append(f"Unrecognized concern: {concern}")
    if not profile.get("primary_concerns"):
        errors.append("At least one concern is required")
    return errors
```

### Handling Missing Data

- Missing `climate`: default to "temperate"
- Missing `undertone`: skip color-matching recommendations
- Missing `budget`: default to "mid"
- Missing `routine_length`: default to "moderate"
- Missing `age_range`: omit age-specific ingredients (retinol dosing, etc.)

## Module 2: Personalization Engine

### Concern-to-Ingredient Mapping

```python
INGREDIENT_MAP = {
    "acne": {
        "actives": ["salicylic acid (BHA)", "benzoyl peroxide", "niacinamide", "tea tree oil"],
        "avoid": ["heavy oils", "coconut oil", "isopropyl myristate"],
        "notes": "Start with lower concentrations. BHA works inside pores."
    },
    "aging": {
        "actives": ["retinol", "vitamin C", "peptides", "hyaluronic acid"],
        "avoid": ["harsh scrubs", "drying alcohols"],
        "notes": "Retinol at night only. Always pair with SPF in morning."
    },
    "hyperpigmentation": {
        "actives": ["vitamin C", "alpha arbutin", "niacinamide", "azelaic acid"],
        "avoid": ["fragranced products on affected areas"],
        "notes": "Consistent SPF is the most important step."
    },
    "dryness": {
        "actives": ["hyaluronic acid", "ceramides", "squalane", "glycerin"],
        "avoid": ["alcohol denat", "astringent toners", "foaming cleansers"],
        "notes": "Apply hyaluronic acid to damp skin for best absorption."
    },
    "redness": {
        "actives": ["centella asiatica", "green tea extract", "azelaic acid", "aloe vera"],
        "avoid": ["fragrance", "essential oils", "harsh exfoliants"],
        "notes": "Minimize routine steps to reduce irritation triggers."
    },
    "large_pores": {
        "actives": ["niacinamide", "BHA", "clay masks (weekly)", "retinol"],
        "avoid": ["pore-clogging oils", "heavy silicones"],
        "notes": "Niacinamide at 5-10% is effective for pore appearance."
    },
    "dullness": {
        "actives": ["vitamin C", "AHA (glycolic/lactic)", "exfoliating toner"],
        "avoid": ["over-exfoliation (max 2-3x/week)"],
        "notes": "Chemical exfoliation reveals brighter skin beneath."
    }
}
```

### Climate Adjustments

```python
CLIMATE_ADJUSTMENTS = {
    "humid_tropical": {
        "prefer": ["lightweight gels", "oil-free moisturizers", "mattifying SPF"],
        "avoid": ["heavy creams", "occlusive layers"],
        "spf_note": "Reapply every 2 hours due to sweat."
    },
    "dry_cold": {
        "prefer": ["rich creams", "facial oils", "occlusive balms"],
        "avoid": ["foaming cleansers", "alcohol-based toners"],
        "spf_note": "UV reflects off snow — SPF still essential."
    },
    "arid_hot": {
        "prefer": ["hydrating mists", "hyaluronic layers", "lightweight SPF"],
        "avoid": ["heavy occlusives during day"],
        "spf_note": "SPF 50+ required. Reapply frequently."
    },
    "temperate": {
        "prefer": ["balanced moisturizers", "seasonal adjustments"],
        "avoid": [],
        "spf_note": "SPF 30+ daily, even on cloudy days."
    }
}
```

### Routine Generation

```python
def generate_routine(profile: dict) -> dict:
    """Generate morning and evening skincare routines."""
    morning = []
    evening = []

    # Step 1: Cleanser (both AM and PM)
    if profile["skin_type"] in ("oily", "combination"):
        morning.append({"step": "Cleanser", "type": "Gel or foam cleanser", "why": "Controls oil without stripping"})
    else:
        morning.append({"step": "Cleanser", "type": "Cream or milk cleanser", "why": "Gentle, maintains moisture barrier"})

    # Step 2: Toner (optional based on routine length)
    if profile["routine_length"] != "minimal":
        morning.append({"step": "Toner", "type": "Hydrating toner (no alcohol)", "why": "Preps skin for actives"})

    # Step 3: Active serums (based on concerns)
    for concern in profile["primary_concerns"]:
        ingredients = INGREDIENT_MAP.get(concern, {})
        if ingredients.get("actives"):
            am_active = ingredients["actives"][0]  # first active for AM
            pm_active = ingredients["actives"][1] if len(ingredients["actives"]) > 1 else am_active
            morning.append({"step": "Serum", "type": am_active, "why": ingredients.get("notes", "")})
            evening.append({"step": "Treatment", "type": pm_active, "why": ingredients.get("notes", "")})

    # Step 4: Moisturizer
    climate = CLIMATE_ADJUSTMENTS.get(profile.get("climate", "temperate"), {})
    moisturizer_type = climate.get("prefer", ["balanced moisturizer"])[0]
    morning.append({"step": "Moisturizer", "type": moisturizer_type, "why": "Locks in hydration and actives"})
    evening.append({"step": "Moisturizer", "type": "Night cream or sleeping mask", "why": "Supports overnight repair"})

    # Step 5: SPF (morning only)
    morning.append({"step": "SPF", "type": "Broad spectrum SPF 30+", "why": climate.get("spf_note", "Protects from UV damage")})

    return {"morning": morning, "evening": evening}
```

## Module 3: Ingredient Reasoning

Every recommendation must include a **why**. Never just list products — explain the reasoning:

```python
EXPLANATION_TEMPLATE = """
**{ingredient}** is recommended because:
- **What it does:** {mechanism}
- **Why for your profile:** {personalized_reason}
- **How to use:** {usage_instructions}
- **Expected timeline:** {timeline}
"""
```

### Ingredient Conflict Detection

```python
CONFLICTS = [
    {"a": "retinol", "b": "AHA", "risk": "Over-exfoliation, irritation", "rule": "Never use in same routine. Alternate nights."},
    {"a": "retinol", "b": "BHA", "risk": "Increased sensitivity", "rule": "Start slow. Use retinol 3x/week, BHA on alternate days."},
    {"a": "retinol", "b": "vitamin C", "risk": "Reduced efficacy", "rule": "Vitamin C in AM, retinol in PM."},
    {"a": "benzoyl peroxide", "b": "retinol", "risk": "Deactivation", "rule": "Do not layer. Use on alternate nights."},
    {"a": "AHA", "b": "BHA", "risk": "Over-exfoliation", "rule": "Use one per routine, not both at once."},
    {"a": "vitamin C", "b": "niacinamide", "risk": "Previously debated, now considered safe", "rule": "Can use together. Apply vitamin C first."},
]

def check_conflicts(recommended_ingredients: list[str]) -> list[dict]:
    """Return any conflicts between recommended ingredients."""
    found = []
    normalized = [i.lower() for i in recommended_ingredients]
    for conflict in CONFLICTS:
        if conflict["a"].lower() in normalized and conflict["b"].lower() in normalized:
            found.append(conflict)
    return found
```

## Module 4: Safety Guardrails

### Sensitivity Handling

```python
SENSITIVITY_RULES = {
    "fragrance": {"avoid": ["parfum", "fragrance", "essential oils", "linalool"], "suggest": "fragrance-free alternatives"},
    "retinol": {"avoid": ["retinol", "retinaldehyde", "tretinoin"], "suggest": "bakuchiol (plant-based alternative)"},
    "AHA": {"avoid": ["glycolic acid", "lactic acid", "mandelic acid"], "suggest": "PHA (polyhydroxy acid) for gentle exfoliation"},
    "BHA": {"avoid": ["salicylic acid"], "suggest": "willow bark extract (milder BHA source)"},
}
```

### Required Disclaimers

Always include:
- "This is for informational purposes. Consult a dermatologist for persistent skin conditions."
- "Patch test new products on a small area before full application."
- "If you experience irritation, discontinue use and consult a professional."
- "Pregnant or nursing individuals should consult their doctor before using retinoids or certain actives."

### Unrealistic Expectations

Flag when users expect results faster than possible:
- Acne clearing: 6-12 weeks minimum
- Hyperpigmentation fading: 8-16 weeks
- Anti-aging visible results: 12+ weeks
- "Overnight" results: set realistic expectations in output

## Module 5: Budget-Aware Recommendations

```python
BUDGET_TIERS = {
    "budget": {"range": "$5-$15/product", "brands": "The Ordinary, CeraVe, Neutrogena"},
    "mid": {"range": "$15-$40/product", "brands": "Paula's Choice, La Roche-Posay, Drunk Elephant (select)"},
    "premium": {"range": "$40-$80/product", "brands": "SkinCeuticals, Dr. Dennis Gross, Sunday Riley"},
    "luxury": {"range": "$80+/product", "brands": "La Mer, SK-II, Augustinus Bader"}
}
```

For each budget tier, suggest comparable alternatives:
- If budget but needs retinol: "The Ordinary Retinol 0.5%" ($7)
- If premium: "SkinCeuticals Retinol 0.5" ($65)
- Always note that expensive does not always mean more effective

## Module 6: Structured Output

### Routine Card Format

```markdown
# Your Personalized Routine

**Profile:** Combination skin | Warm undertone | Humid climate
**Concerns:** Acne, uneven texture
**Budget:** Mid-range

## Morning Routine (5 steps)

| Step | Product Type | Key Ingredient | Why |
|------|-------------|----------------|-----|
| 1 | Gel cleanser | - | Controls oil without stripping |
| 2 | Hydrating toner | Hyaluronic acid | Preps skin for actives |
| 3 | Serum | Niacinamide 10% | Reduces pore appearance, controls sebum |
| 4 | Moisturizer | Lightweight gel | Hydrates without heaviness in humidity |
| 5 | SPF 50+ | Mattifying formula | UV protection, reapply every 2 hours |

## Evening Routine (4 steps)
...

## Timeline
- Week 1-2: Skin adjusting, possible mild purging
- Week 4-6: Reduced breakouts, smoother texture
- Week 8-12: Visible improvement in tone and clarity

## Warnings
- Do NOT use retinol and AHA in the same routine
- Patch test salicylic acid if you have sensitive skin
```

## Module 7: Feedback & Adaptation

### After Initial Consultation

```python
feedback = {
    "routine_rating": 4,            # 1-5
    "products_tried": ["cleanser", "serum"],
    "issues": ["serum caused tingling"],
    "improvements_noticed": ["less oily by afternoon"],
    "wants_to_adjust": True
}
```

Use feedback to refine:
- Tingling → suggest lower concentration or alternative active
- No improvement after 8 weeks → escalate actives or add treatment
- Positive feedback → reinforce current routine, add next-level treatments

## Scaling Considerations

- Product database with ingredient lists for specific recommendations
- Image-based skin analysis (computer vision integration)
- Regional product availability filters
- Multi-language support for global brands
- Seasonal routine adjustments (auto-detect based on date + location)

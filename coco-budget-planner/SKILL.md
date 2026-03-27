---
name: coco-budget-planner
description: Build personal or business budgeting tools with savings projections. Use when creating budget calculators, expense trackers, savings goal planners, or financial dashboards. Includes allocation strategies and visual reporting.
license: Apache-2.0
metadata:
  author: good-stories-llc
  version: "1.0"
  brand: COCO
---

# Budget Planner

Build budgeting tools that calculate allocations, project savings, and produce visual dashboards. Handles personal budgets, business budgets, irregular income, and goal-based savings planning.

## When to Use

- User wants to build a budget calculator or expense tracker
- User needs savings projections over 3, 6, or 12 months
- User asks for budget allocation using 50/30/20 or custom rules
- User wants a financial dashboard with charts and category breakdowns

## Input Requirements

### Required Inputs

```python
{
    "monthly_income": 5000.00,       # After-tax income
    "fixed_expenses": {              # Bills that stay the same each month
        "rent": 1500.00,
        "utilities": 150.00,
        "insurance": 200.00,
        "subscriptions": 50.00,
        "loan_payments": 300.00,
    },
    "variable_expenses": {           # Expenses that change month to month
        "groceries": 400.00,
        "dining_out": 200.00,
        "transportation": 150.00,
        "entertainment": 100.00,
        "personal_care": 75.00,
    },
}
```

### Optional Inputs

```python
{
    "savings_goal": 10000.00,        # Target savings amount
    "savings_deadline_months": 12,   # Months to reach the goal
    "allocation_strategy": "50/30/20",  # Or "custom"
    "irregular_income": [            # For freelance/variable income
        {"month": "January", "amount": 4200.00},
        {"month": "February", "amount": 6800.00},
    ],
    "currency": "USD",
}
```

## Allocation Strategies

### 50/30/20 Rule

The standard allocation framework:

| Category | % of Income | Description |
|---|---|---|
| Needs | 50% | Housing, utilities, insurance, groceries, minimum debt payments |
| Wants | 30% | Dining, entertainment, subscriptions, shopping |
| Savings | 20% | Emergency fund, investments, extra debt payments |

### 70/20/10 Rule (Conservative)

| Category | % of Income | Description |
|---|---|---|
| Living expenses | 70% | All needs and wants combined |
| Savings | 20% | Emergency fund, investments |
| Giving/debt | 10% | Charity, extra debt payments |

### Zero-Based Budgeting

Every dollar is assigned a job. Income minus all allocations equals zero:

```python
def zero_based_budget(income: float, categories: dict) -> dict:
    total_allocated = sum(categories.values())
    remaining = income - total_allocated
    return {
        "categories": categories,
        "total_allocated": total_allocated,
        "remaining_to_assign": remaining,
        "is_balanced": abs(remaining) < 0.01,
    }
```

### Custom Rules

Let the user define their own percentage splits:

```python
def custom_allocation(income: float, rules: dict) -> dict:
    """
    rules: {"needs": 55, "wants": 25, "savings": 15, "investments": 5}
    Percentages must sum to 100.
    """
    if abs(sum(rules.values()) - 100) > 0.01:
        raise ValueError("Allocation percentages must sum to 100")
    return {
        category: round(income * pct / 100, 2)
        for category, pct in rules.items()
    }
```

## Core Calculations

### Budget Summary

```python
def calculate_budget(income: float, fixed: dict, variable: dict) -> dict:
    total_fixed = sum(fixed.values())
    total_variable = sum(variable.values())
    total_expenses = total_fixed + total_variable
    savings = income - total_expenses
    savings_rate = (savings / income * 100) if income > 0 else 0

    return {
        "income": income,
        "total_fixed": round(total_fixed, 2),
        "total_variable": round(total_variable, 2),
        "total_expenses": round(total_expenses, 2),
        "savings": round(savings, 2),
        "savings_rate": round(savings_rate, 1),
        "status": "surplus" if savings > 0 else "deficit" if savings < 0 else "break-even",
    }
```

### Savings Projection

Project savings growth over time with optional interest:

```python
def project_savings(
    monthly_savings: float,
    months: int,
    starting_balance: float = 0,
    annual_interest_rate: float = 0,
) -> list:
    monthly_rate = annual_interest_rate / 12 / 100
    projections = []
    balance = starting_balance

    for month in range(1, months + 1):
        interest = balance * monthly_rate
        balance += monthly_savings + interest
        projections.append({
            "month": month,
            "deposit": round(monthly_savings, 2),
            "interest": round(interest, 2),
            "balance": round(balance, 2),
        })

    return projections
```

### Goal Feasibility

Determine if a savings goal is reachable:

```python
def check_goal_feasibility(
    goal: float,
    monthly_savings: float,
    deadline_months: int,
    starting_balance: float = 0,
) -> dict:
    total_possible = starting_balance + (monthly_savings * deadline_months)
    gap = goal - total_possible
    required_monthly = (goal - starting_balance) / deadline_months if deadline_months > 0 else float("inf")

    return {
        "goal": goal,
        "deadline_months": deadline_months,
        "monthly_savings_available": round(monthly_savings, 2),
        "monthly_savings_required": round(required_monthly, 2),
        "total_projected": round(total_possible, 2),
        "feasible": total_possible >= goal,
        "gap": round(max(gap, 0), 2),
        "extra_monthly_needed": round(max(gap / deadline_months, 0), 2) if deadline_months > 0 else 0,
    }
```

### Irregular Income Handling

For users with variable income (freelance, contract, commission):

```python
def analyze_irregular_income(income_history: list) -> dict:
    amounts = [entry["amount"] for entry in income_history]
    if not amounts:
        return {"error": "No income data provided"}

    avg = sum(amounts) / len(amounts)
    minimum = min(amounts)
    maximum = max(amounts)

    return {
        "average_monthly": round(avg, 2),
        "minimum_month": round(minimum, 2),
        "maximum_month": round(maximum, 2),
        "variance": round(maximum - minimum, 2),
        "recommended_budget_base": round(minimum * 0.9, 2),
        "note": "Budget on 90% of your lowest month. Save windfalls from high months.",
    }
```

## Category Breakdown

### Expense Categorization

```python
CATEGORY_MAP = {
    "needs": [
        "rent", "mortgage", "utilities", "insurance", "groceries",
        "healthcare", "childcare", "minimum_debt_payments", "transportation",
    ],
    "wants": [
        "dining_out", "entertainment", "subscriptions", "shopping",
        "hobbies", "personal_care", "travel", "gifts",
    ],
    "savings": [
        "emergency_fund", "investments", "retirement", "extra_debt_payments",
        "education_fund", "vacation_fund",
    ],
}

def categorize_expenses(expenses: dict) -> dict:
    categorized = {"needs": {}, "wants": {}, "savings": {}, "uncategorized": {}}
    for expense, amount in expenses.items():
        placed = False
        for category, keywords in CATEGORY_MAP.items():
            if expense.lower() in keywords:
                categorized[category][expense] = amount
                placed = True
                break
        if not placed:
            categorized["uncategorized"][expense] = amount
    return categorized
```

## Dashboard Layout

### HTML Dashboard Structure

When building a visual budget dashboard, include these panels:

```
+------------------------------------------+
|          Monthly Budget Overview          |
|  Income: $5,000  |  Expenses: $3,125     |
|  Savings: $1,875 |  Rate: 37.5%          |
+------------------------------------------+
|                    |                      |
|   Expense Donut   |   Category Bars      |
|   (needs/wants/   |   (ranked by $)      |
|    savings)        |                      |
+--------------------+----------------------+
|          Savings Projection               |
|  [Line chart: balance over 12 months]     |
+------------------------------------------+
|          Goal Progress                    |
|  [Progress bar: $3,750 / $10,000 = 37%]  |
+------------------------------------------+
```

### Chart Specifications

**Expense Donut Chart:**
- 3 segments: Needs, Wants, Savings
- Colors: Needs = slate (#64748B), Wants = amber (#D97706), Savings = green (#059669)
- Center text: total monthly expenses

**Category Bar Chart:**
- Horizontal bars, sorted by amount descending
- Show dollar amount at the end of each bar
- Use consistent color per category (needs/wants/savings)

**Savings Projection Line Chart:**
- X-axis: months (1-12)
- Y-axis: balance in dollars
- Mark the goal line as a dashed horizontal line
- Shade the area under the projection curve

**Goal Progress Bar:**
- Single horizontal bar with percentage fill
- Show current/target amounts as text
- Green if on track, amber if behind schedule, red if infeasible

## Edge Cases

| Scenario | Handling |
|---|---|
| Zero income | Show warning: "Income must be greater than zero for meaningful budgeting." |
| Expenses exceed income | Show deficit amount, flag the largest variable expenses as reduction candidates |
| Negative savings | Highlight the gap, suggest expense categories to cut |
| Irregular income | Use 90% of lowest month as budget base; display variance analysis |
| Single expense category | Run pipeline, note limited analysis |
| Goal deadline = 0 months | Return error: "Deadline must be at least 1 month" |
| Very large numbers | Format with thousand separators, use abbreviations for >$1M |

## Output Expectations

When building a budget tool, deliver:

1. **Budget summary** with income, expenses, savings, and savings rate
2. **Category breakdown** with needs/wants/savings classification
3. **Allocation comparison** showing actual vs. recommended (50/30/20)
4. **Savings projection** over 3, 6, and 12 months
5. **Goal feasibility assessment** if a savings goal is provided
6. **Visual dashboard** with charts (donut, bars, line, progress bar)
7. **Actionable recommendations** for improving the budget

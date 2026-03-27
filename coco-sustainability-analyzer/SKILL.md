---
name: coco-sustainability-analyzer
description: Evaluate and optimize sustainability impact for engineering and business projects. Use when calculating carbon footprint, assessing energy efficiency, scoring material sustainability, or generating green recommendations for projects.
license: Apache-2.0
metadata:
  author: good-stories-llc
  version: "1.0"
  brand: COCO
---

# Sustainability Analyzer

Evaluate the environmental impact of engineering and business projects. Calculate carbon footprints, score energy efficiency and material sustainability, flag high-impact risks, and generate actionable recommendations with estimated impact reduction.

## When to Use

- User wants to assess the environmental impact of a project
- User needs carbon footprint calculations for materials, energy, or transport
- User asks for sustainability scoring or green recommendations
- User wants to compare design alternatives by environmental impact

## Input Requirements

### Project Profile

```python
{
    "project_name": "Office Renovation - Building A",
    "project_type": "construction",  # construction, manufacturing, software, operations
    "location": "New York, NY",
    "duration_months": 6,
    "materials": [
        {"name": "concrete", "quantity": 500, "unit": "metric_tons"},
        {"name": "steel", "quantity": 120, "unit": "metric_tons"},
        {"name": "glass", "quantity": 200, "unit": "sq_meters"},
        {"name": "timber", "quantity": 80, "unit": "cubic_meters"},
    ],
    "energy_systems": {
        "electricity_kwh_per_month": 25000,
        "natural_gas_therms_per_month": 500,
        "renewable_percentage": 15,
    },
    "transport": {
        "material_transport_km": 350,
        "vehicle_type": "diesel_truck",
        "trips": 40,
    },
    "lifecycle_years": 30,
}
```

### Project Types

| Type | Key Factors |
|---|---|
| **Construction** | Materials, transport, embodied carbon, operational energy |
| **Manufacturing** | Raw materials, energy per unit, waste, supply chain transport |
| **Software/IT** | Server energy, data center PUE, device lifecycle, cloud provider carbon |
| **Operations** | Office energy, commute, travel, procurement, waste |

## Carbon Footprint Calculation

### Emission Factors Database

Standard emission factors (kg CO2e per unit). These are reference values -- always note the source and year:

```python
EMISSION_FACTORS = {
    # Materials (kg CO2e per metric ton)
    "concrete": 410,
    "steel_virgin": 1850,
    "steel_recycled": 650,
    "aluminum_virgin": 11000,
    "aluminum_recycled": 600,
    "timber_softwood": -1600,    # Net negative (carbon sequestration)
    "timber_hardwood": -900,
    "glass": 1200,
    "plastic_hdpe": 1900,
    "copper": 3500,
    "brick": 230,
    "insulation_fiberglass": 1350,
    "insulation_cellulose": 70,

    # Energy (kg CO2e per unit)
    "electricity_kwh_us_avg": 0.42,
    "electricity_kwh_eu_avg": 0.28,
    "electricity_kwh_renewable": 0.02,
    "natural_gas_therm": 5.3,
    "diesel_liter": 2.68,
    "gasoline_liter": 2.31,

    # Transport (kg CO2e per ton-km)
    "diesel_truck": 0.062,
    "electric_truck": 0.025,
    "rail": 0.022,
    "cargo_ship": 0.008,
    "air_freight": 0.602,
}
```

### Calculation Engine

```python
def calculate_carbon_footprint(project: dict) -> dict:
    results = {
        "materials": {},
        "energy": {},
        "transport": {},
        "total_kg_co2e": 0,
    }

    # Materials emissions
    for material in project.get("materials", []):
        name = material["name"]
        qty = material["quantity"]
        factor = EMISSION_FACTORS.get(name, 0)
        emissions = qty * factor
        results["materials"][name] = {
            "quantity": qty,
            "unit": material["unit"],
            "factor": factor,
            "emissions_kg_co2e": round(emissions, 1),
        }
        results["total_kg_co2e"] += emissions

    # Energy emissions
    energy = project.get("energy_systems", {})
    duration = project.get("duration_months", 12)
    renewable_pct = energy.get("renewable_percentage", 0) / 100

    elec_kwh = energy.get("electricity_kwh_per_month", 0) * duration
    elec_renewable = elec_kwh * renewable_pct
    elec_grid = elec_kwh - elec_renewable
    elec_emissions = (
        elec_grid * EMISSION_FACTORS["electricity_kwh_us_avg"]
        + elec_renewable * EMISSION_FACTORS["electricity_kwh_renewable"]
    )
    results["energy"]["electricity"] = {
        "total_kwh": round(elec_kwh, 1),
        "renewable_kwh": round(elec_renewable, 1),
        "grid_kwh": round(elec_grid, 1),
        "emissions_kg_co2e": round(elec_emissions, 1),
    }
    results["total_kg_co2e"] += elec_emissions

    gas_therms = energy.get("natural_gas_therms_per_month", 0) * duration
    gas_emissions = gas_therms * EMISSION_FACTORS["natural_gas_therm"]
    results["energy"]["natural_gas"] = {
        "total_therms": round(gas_therms, 1),
        "emissions_kg_co2e": round(gas_emissions, 1),
    }
    results["total_kg_co2e"] += gas_emissions

    # Transport emissions
    transport = project.get("transport", {})
    distance_km = transport.get("material_transport_km", 0)
    trips = transport.get("trips", 0)
    vehicle = transport.get("vehicle_type", "diesel_truck")
    # Estimate average load in tons
    total_material_tons = sum(m["quantity"] for m in project.get("materials", []))
    tons_per_trip = total_material_tons / max(trips, 1)
    transport_emissions = trips * distance_km * tons_per_trip * EMISSION_FACTORS.get(vehicle, 0.062)
    results["transport"] = {
        "total_km": distance_km * trips,
        "vehicle_type": vehicle,
        "emissions_kg_co2e": round(transport_emissions, 1),
    }
    results["total_kg_co2e"] += transport_emissions

    results["total_kg_co2e"] = round(results["total_kg_co2e"], 1)
    results["total_metric_tons_co2e"] = round(results["total_kg_co2e"] / 1000, 2)

    return results
```

## Energy Efficiency Score

Rate the project's energy efficiency on a 0-100 scale:

```python
def score_energy_efficiency(energy: dict) -> dict:
    renewable_pct = energy.get("renewable_percentage", 0)
    total_kwh = energy.get("electricity_kwh_per_month", 0)

    # Base score from renewable percentage
    base_score = min(renewable_pct, 100)

    # Penalty for high absolute consumption (normalized)
    consumption_penalty = 0
    if total_kwh > 50000:
        consumption_penalty = min((total_kwh - 50000) / 5000, 20)

    # Bonus for natural gas reduction
    gas_therms = energy.get("natural_gas_therms_per_month", 0)
    gas_penalty = min(gas_therms / 100, 15) if gas_therms > 0 else 0

    score = max(0, min(100, base_score - consumption_penalty - gas_penalty))

    rating = "Excellent" if score >= 80 else "Good" if score >= 60 else "Fair" if score >= 40 else "Poor"

    return {
        "score": round(score, 1),
        "rating": rating,
        "renewable_percentage": renewable_pct,
        "factors": {
            "renewable_base": round(base_score, 1),
            "consumption_penalty": round(consumption_penalty, 1),
            "gas_penalty": round(gas_penalty, 1),
        },
    }
```

## Material Sustainability Score

Score each material on sustainability:

```python
MATERIAL_SUSTAINABILITY = {
    "timber_softwood": {"score": 90, "renewable": True, "recyclable": True, "note": "Carbon-negative if sustainably sourced"},
    "timber_hardwood": {"score": 75, "renewable": True, "recyclable": True, "note": "Verify FSC certification"},
    "steel_recycled": {"score": 70, "renewable": False, "recyclable": True, "note": "65% lower emissions than virgin steel"},
    "brick": {"score": 60, "renewable": False, "recyclable": True, "note": "Durable, low maintenance"},
    "insulation_cellulose": {"score": 85, "renewable": True, "recyclable": True, "note": "Made from recycled paper"},
    "glass": {"score": 55, "renewable": False, "recyclable": True, "note": "Infinitely recyclable"},
    "concrete": {"score": 30, "renewable": False, "recyclable": False, "note": "High embodied carbon, consider alternatives"},
    "steel_virgin": {"score": 25, "renewable": False, "recyclable": True, "note": "Use recycled steel where possible"},
    "aluminum_virgin": {"score": 15, "renewable": False, "recyclable": True, "note": "Extremely energy-intensive; use recycled"},
    "plastic_hdpe": {"score": 20, "renewable": False, "recyclable": True, "note": "Petroleum-based, limited recycling rates"},
}

def score_materials(materials: list) -> dict:
    scores = []
    flags = []
    for material in materials:
        name = material["name"]
        info = MATERIAL_SUSTAINABILITY.get(name, {"score": 50, "note": "No data available"})
        scores.append(info["score"])
        if info["score"] < 35:
            flags.append({
                "material": name,
                "score": info["score"],
                "issue": "High embodied carbon",
                "note": info["note"],
            })

    avg_score = sum(scores) / len(scores) if scores else 0
    return {
        "average_score": round(avg_score, 1),
        "rating": "Excellent" if avg_score >= 75 else "Good" if avg_score >= 55 else "Fair" if avg_score >= 35 else "Poor",
        "risk_flags": flags,
    }
```

## Overall Sustainability Index

Combine the three scores into a weighted index:

```python
def overall_sustainability_index(carbon: dict, energy: dict, materials: dict) -> dict:
    # Normalize carbon to a 0-100 score (lower is better)
    total_tons = carbon["total_metric_tons_co2e"]
    # Rough benchmark: <100 tons = good, >1000 tons = poor
    carbon_score = max(0, min(100, 100 - (total_tons / 10)))

    energy_score = energy["score"]
    material_score = materials["average_score"]

    # Weighted average
    weights = {"carbon": 0.40, "energy": 0.30, "materials": 0.30}
    index = (
        carbon_score * weights["carbon"]
        + energy_score * weights["energy"]
        + material_score * weights["materials"]
    )

    return {
        "index": round(index, 1),
        "rating": "Excellent" if index >= 75 else "Good" if index >= 55 else "Fair" if index >= 35 else "Poor",
        "breakdown": {
            "carbon_score": round(carbon_score, 1),
            "energy_score": round(energy_score, 1),
            "material_score": round(material_score, 1),
        },
        "weights": weights,
    }
```

## Risk Flagging

Automatically flag high-impact issues:

```python
def flag_risks(carbon: dict, energy: dict, materials: dict, transport: dict) -> list:
    flags = []

    # High embodied carbon materials
    for name, data in carbon.get("materials", {}).items():
        if data["emissions_kg_co2e"] > 100000:
            flags.append({
                "severity": "high",
                "category": "materials",
                "issue": f"{name} contributes {data['emissions_kg_co2e']:,.0f} kg CO2e",
                "recommendation": f"Consider alternatives to {name} or use recycled variants",
            })

    # Low renewable energy
    if energy.get("score", 0) < 40:
        flags.append({
            "severity": "high",
            "category": "energy",
            "issue": f"Energy efficiency score is {energy['score']}/100",
            "recommendation": "Increase renewable energy percentage or reduce consumption",
        })

    # High transport impact
    if transport.get("emissions_kg_co2e", 0) > 50000:
        flags.append({
            "severity": "medium",
            "category": "transport",
            "issue": f"Transport emissions: {transport['emissions_kg_co2e']:,.0f} kg CO2e",
            "recommendation": "Source materials locally or switch to rail/electric transport",
        })

    return flags
```

## Recommendations Engine

Generate actionable recommendations with estimated impact:

```python
def generate_recommendations(carbon: dict, energy: dict, materials_data: list) -> list:
    recommendations = []

    # Material substitutions
    for material in materials_data:
        name = material["name"]
        if name == "concrete":
            recommendations.append({
                "action": "Replace 30% of concrete with timber or engineered wood",
                "category": "materials",
                "estimated_reduction_pct": 25,
                "estimated_reduction_kg": round(
                    carbon["materials"].get("concrete", {}).get("emissions_kg_co2e", 0) * 0.30 * 1.5, 0
                ),
                "difficulty": "medium",
                "notes": "Requires structural engineering review",
            })
        if name == "steel_virgin":
            recommendations.append({
                "action": "Switch to recycled steel (EAF process)",
                "category": "materials",
                "estimated_reduction_pct": 65,
                "estimated_reduction_kg": round(
                    carbon["materials"].get("steel_virgin", {}).get("emissions_kg_co2e", 0) * 0.65, 0
                ),
                "difficulty": "low",
                "notes": "Recycled steel meets the same structural standards",
            })

    # Energy improvements
    renewable_pct = energy.get("renewable_percentage", 0)
    if renewable_pct < 50:
        recommendations.append({
            "action": f"Increase renewable energy from {renewable_pct}% to 50%",
            "category": "energy",
            "estimated_reduction_pct": 20,
            "difficulty": "medium",
            "notes": "Purchase renewable energy certificates or install on-site solar",
        })

    if renewable_pct < 100:
        recommendations.append({
            "action": "Transition to 100% renewable electricity",
            "category": "energy",
            "estimated_reduction_pct": 35,
            "difficulty": "high",
            "notes": "May require PPA (power purchase agreement) or on-site generation",
        })

    return recommendations
```

## Dashboard Output

### Report Structure

```markdown
# Sustainability Assessment: [Project Name]

## Summary
- Overall Index: [score]/100 ([rating])
- Total Carbon Footprint: [X] metric tons CO2e
- Energy Efficiency: [score]/100
- Material Sustainability: [score]/100

## Carbon Breakdown
| Source | Emissions (kg CO2e) | % of Total |
|---|---|---|
| Materials | X | X% |
| Energy | X | X% |
| Transport | X | X% |

## Risk Flags
- [severity] [issue] -- [recommendation]

## Recommendations (by impact)
| # | Action | Reduction | Difficulty |
|---|---|---|---|
| 1 | [action] | [X kg CO2e] | [low/med/high] |

## Material Detail
| Material | Quantity | Emissions | Sustainability Score |
|---|---|---|---|
| [name] | [qty] | [kg CO2e] | [score]/100 |
```

## Edge Cases

| Scenario | Handling |
|---|---|
| No materials listed | Skip materials section, note in report |
| Zero energy consumption | Score energy at 100, note in report |
| Unknown material type | Use average emission factor, flag as estimated |
| Negative emissions (timber) | Show as carbon sequestration, include in total |
| Software project (no physical materials) | Focus on server energy, cloud provider carbon data |
| Missing transport data | Exclude transport section, note in report |

## Output Expectations

1. Complete sustainability report with all sections
2. Numerical scores with clear rating scales
3. At least 3 actionable recommendations ranked by impact
4. Risk flags for any high-severity issues
5. Comparison to benchmarks where available
6. All calculations shown with factors and sources noted

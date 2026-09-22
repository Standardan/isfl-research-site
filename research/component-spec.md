# Build selector contract

The component accepts league, game role, archetype and available TPE. Its visual design is deferred. `recommendation_engine.py` implements the calculation layer without a web framework or external dependencies.

## Inputs

- `league`: DSFL or ISFL. DSFL accepts 0–250 TPE and no purchased traits.
- `role`: explicit game role. OL expands to C/G/T, LB to MLB/OLB, and S to FS/SS. Preserve KR and the combined K/P test role.
- `archetype`: filter using `engine-data.json → best_screening_by_role_archetype`; each key is `role|archetype`.
- `budget`: nonnegative integer TPE, measured above free archetype starting ratings. Trait fees share this budget.
- `mode`: `research` (default), `mechanic`, or `maxed`.
- `objective`: required for mechanic mode; one of the seven named formula objectives.
- `selected_traits`: optional list of portal trait names for ISFL. The engine reserves their costs and prerequisite ratings, includes prerequisite unlock traits, and checks legality.
- `profile`: optional frozen candidate ID for research mode. Omit to select the highest screening-margin profile within the chosen archetype and role.

Start with numeric TPE entry. A slider can use the same integer input later. Changing role resets an incompatible archetype. Retain the user's budget when changing archetype; show an inline error when selected traits cannot be afforded.

## Recommendation modes

| Mode | Calculation | Display basis |
|---|---|---|
| research, DSFL 250 | Retrieves the exact frozen tested allocation | Tested DSFL sample |
| research, other budgets or ISFL | Replays ordered research goals, then cyclic remainder priorities within recipe ceilings | Research-guided allocation |
| mechanic | Exact integer-budget optimization of one local formula, respecting caps and selected-trait prerequisites | Exact formula optimum |
| maxed, ISFL | All effective attribute caps and every purchasable portal trait, including kicker cap unlocks | Fully maxed template |

Research mode does not automatically purchase ISFL traits. Use selected traits or the explicit fully maxed view. Recipe ceilings can leave TPE unused; report the remainder. A changed budget, trait set or league does not inherit the observed score of a DSFL sample. Formula mode maximizes its stated subtotal; the simulator's subsequent rolls and contextual modifiers remain separate.

## Output and presentation

Return all 14 ratings, per-attribute TPE, paid and automatic traits, total spent, remainder, dimensions, basis, and mode-specific explanation. `predicted_game_margin` is null. Display current rating beside starting rating and effective cap. Show selected-trait prerequisites and unlock order in an expandable cost breakdown.

Link explanation sections to stable mechanics fact IDs: movement → `speed.base`; contact → `contact.runner` / `contact.defender`; pass/run blocking → `blocking.pass` / `blocking.run`; kicking → the kicking facts in the mechanics index. Preserve source references with the explanations.

The comparison view is role-specific. Show each archetype's highest screening candidate and the separately confirmed finalist means. Label margins as treated-team points for minus points against; these are sample outcomes, not bonuses caused solely by the player. Show the paired comparison interval with its A/B candidate labels. Use `simultaneous_supported_winner_candidate_id` for a statistically supported winner badge; it is currently null for all 17 roles.

## Callable interface

```python
from recommendation_engine import recommend
result = recommend('DSFL', 'WR', 'Speed Receiver', 250)
result = recommend('DSFL', 'K/P', 'Accurate', 250,
                   mode='mechanic', objective='field_goal_rating_sum')
result = recommend('ISFL', 'K/P', 'Accurate', 1240, mode='maxed')
```

Serve this function from a future backend or port it with the frozen test fixtures. Convert `ValueError` into a field-level validation message. `component-examples.json` supplies complete request/response fixtures, including an invalid request. Ship `recommendation_engine.py` beside `engine-data.json` for standalone execution.

## Evidence needed for game-performance optimization

For new budgets and fully maxed ISFL builds, generate legal candidates with the engine, compare equal-budget alternatives in original-engine games, and reserve fresh seeds and matchups for confirmation. Record host roster, gameplans, skills, experience, automatic-trait export, dimensions, simulator hash and lookup-table state. Choose a declared role objective before selection; report paired uncertainty and multiple-comparison adjustment. Store each tested allocation as a new evidence record rather than replacing the existing measurements.

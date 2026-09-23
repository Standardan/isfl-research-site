> **Superseded.** This describes the earlier joint mechanics-proxy builder, which the website no longer uses. The current builder optimizes the build-WAR win models fitted to paired native-engine simulations; see the `build_war` section of research-pack.json and the Build WAR section of LLM-RESEARCH.md.

# Joint attribute and trait recommendations

Version: joint-mechanics-v1. Goal: help spend TPE toward team contribution. **This is a mechanics-informed model, not a calibrated prediction of wins or a demonstrated best game build.** No new ISFL games were run for this release. The DSFL 250-TPE tested examples remain unchanged.

## Search and spending

For ISFL, enumerate all legal purchasable trait subsets (including no traits), expand dependencies, exclude duplicates of automatic traits, charge each trait and all required attribute upgrades, and apply cap unlocks. For each affordable subset, solve a multiple-choice knapsack over integer attribute ratings, within the remaining budget. Compare scores across subsets. Ties prefer lower expenditure; remaining TPE is not spent on a modeled zero-gain purchase merely to exhaust the budget. Numeric tie tolerance is 1e-10. There are at most 32 raw subsets in the present portal inventory; predecessor-expanded duplicates are evaluated once.

This is an exact search for the stated separable objective, not an exact simulation optimizer. The model is additive and cannot reproduce all interactions. A higher score is not a percentage increase in wins. The attributes-only comparator is optimized under the same objective, not the old ordered recipe. Alternatives shown per trait re-optimize the entire build with that trait included/excluded, so multiple ratings and other traits may change together.

Existing saved/shared all-max or formula requests open in the unified research-guided builder. The audited legacy engine remains available to research code/tests; its existing Python parity checks do not validate the new objective. New joint-search tests separately cover legality, model comparisons and exhaustive small problems.

## Explicit modeling assumptions

Role weights are chosen research heuristics, **not fitted coefficients from game outcomes**:

| Role family | Components and weights |
|---|---|
| WR | Movement40%, receiving35%, endurance15%, discipline10% |
| TE | 75% of WR blend + blocking25% |
| RB | Movement35%, raw contact30%, receiving10%, endurance15%, discipline10% |
| FB | 75% of RB blend + blocking25% |
| OL | Blocking65%, block release15%, endurance10%, discipline10% |
| QB | Passing subtotal65%, movement15%, interception range10%, discipline10% |
| DE/DT/LB | Contact30%, block resistance20%, movement20%, coverage10%, endurance10%, discipline10%; multiply all by.95, add recognition5% |
| CB/S | Coverage35%, movement30%, contact15%, endurance10%, discipline10%; multiply all by.95, add recognition5% |
| KR | Baseline movement60%, endurance20%, discipline20%; no pass-carrier or blitz bonus |
| K/P | Kick accuracy + kick power subtotal100% |

Movement mixes: WR/TE half baseline and half pass carrier; RB/FB80% run carrier/20% pass carrier; QB80% baseline/20% run; defensive front half baseline/half blitz; secondary90% baseline/10% blitz. These are declared scenarios, not observed snap proportions. Specialized playbooks, target share and return workloads can change which build is preferable.

## Mechanics used and normalization

- Movement uses (1.25+.015*Speed)/2.75 with applicable multiplicative traits only in their declared contexts. Source Player.cs2072 and2141–2469. Traits do not grant unconditional route speed.
- Receiving uses the expected own-rating contributions from GetPassChance: the three independent Hands draws, Hands70/80 penalties, Agility and Intelligence failure draws, and Competitiveness>70 bonus; divide by3.5 as a model scale. Source GameFunctions.cs12388–12422. This is not completion probability.
- Blocking uses a50/50 blend of pass/run attribute subtotals divided by500; applicable+10 engagement traits become+10/50 in this scale. Resistance uses STR+2*TACK+AGI divided by400, with each rusher trait+10/40. Source Player.cs258–289. Opposing ratings, clamps, engagement opportunity and integer truncation omitted.
- Contact uses raw runner coefficients4.7AGI+2SPD+5STR+4INT divided by1570, or defender4STR+SPD+AGI+7.5TACK+4INT divided by1750. Competitor fatigue resilience is an additive approximation to the conditional>=5 tackle branch, not the actual multiplied/truncated score. Actual contact clamps800–1200/1300 can eliminate marginal gains; this surrogate does not reproduce them. Source GameFunctions.cs9696,9779,9814,9949–9965.
- Coverage uses AGI+HND+1.35INT+SPD/2.3 divided by100*(3.35+1/2.3), with the selected interception-trait modifier. It is a subtotal, not interception probability. Source GameFunctions.cs9519–9527,9652.
- QB passing uses4INT+4ACC+1.1ARM divided by910. The separate interception-range term preserves stock INT/ACC/ARM tiers and FilmGeek/GameManager/Legend precedence on a shared137 scale. Full joint-score truncation, opponent comparison and RNG probability are omitted. Source GameFunctions.cs9549–9607,9651.
- Recognition is one Cover-action run threshold scenario with Perceptive5, else BoxSafety2, else PressCorner1, divided by100. This does not value every recognition branch. Source GameFunctions.cs5531–5545.
- Discipline uses min(100,INT+trait threshold credit)/100 as a neutral-context proxy. RoleModel+10 respects preceding BadInfluence/Diva/MediaDarling. Penalty occurrence, opponent, home/away, experience and sportsmanship are omitted. Source GameFunctions.cs10055–10128.
- Endurance blends70% normalized baseline recovery-minus-loss (before clamps) and30% workload reserve. WR/TE workload uses floor(END/10)/10 from catch loss; others use END/100 as a heuristic reserve. This does not simulate energy or establish a full-game stamina guarantee. Source Player.cs5091–5134; GameFunctions.cs6858.
- K/P uses (KA+KP)/200 with no fictional direct effect from portal cap-unlock traits. This favors a fixed-distance field-goal subtotal; it does not optimize punts, kickoffs or coaching decisions.

## Trait coverage and limits

All26 purchasable portal names have source-linked explanations in trait-effects.json. Each record distinguishes scored and unscored effects. DeepThreat target selection, SlotReceiver pathing, BlockingTE/FB reduced receiving targets, Dualthreat scramble tendency/resistance, and several other interactions are omitted. Gunslinger preference and ShutDownCorner post-failure credit receive no positive model score; this does not prove they are inert or undesirable in all contexts. Automatic/exported game trait mapping remains an unverified roster-export assumption.

Team wins require opponents, teammates, schemes and game-level validation. The current input has none of that context. This release improves the budget comparison and exposes assumptions; it does not settle the empirical attribute-versus-trait question. Next validation should compare chosen builds against attributes-only, alternative trait subsets and perturbed scenario weights at matched budgets using paired game seeds and holdout confirmations.

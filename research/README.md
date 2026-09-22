# DDSPF21 website research handoff

Start with **LLM-RESEARCH.md**: one factual reference with mechanics, all archetype builds, comparisons and the component contract. **research-pack.json** contains the same core material as structured records with stable IDs and source references.

- 153 mechanics records: every core attribute, personality field, position skill and all 59 trait enum entries.
- 36 DSFL archetypes, expanded to 52 archetype/role examples; each is the best observed screening candidate within its group and costs at most 250 TPE.
- 36 ISFL templates: every effective attribute cap and every purchasable portal trait, including specialist cap unlocks.
- 17 role comparison records, retaining paired confirmation intervals.
- A working budget calculation layer and six request/response fixtures for the future selector.

## Run the selector

Requires Python 3.9+ and its standard library. Keep `recommendation_engine.py` and `engine-data.json` together.

```powershell
python recommendation_engine.py --league DSFL --role WR --archetype "Speed Receiver" --tpe 250
python recommendation_engine.py --league DSFL --role "K/P" --archetype Accurate --tpe 250 --mode mechanic --objective field_goal_rating_sum
python recommendation_engine.py --league ISFL --role "K/P" --archetype Accurate --tpe 1240 --mode maxed
```

`component-spec.md` defines input validation, selection modes, output fields and evidence labels. `component-examples.json` provides complete fixtures. Visual design and hosting are deferred.

## Evidence meaning

DSFL examples were selected from the accepted 12,120-game campaign. All 17 simultaneous confirmation intervals include zero. Arbitrary-budget research allocations replay the tested recipe's priorities; exact formula mode optimizes its named local objective. Fully maxed ISFL templates are cost- and legality-checked; maxed archetype performance comparisons remain to be run.

## Files and reproducibility

`mechanics.json` and `build-examples.json` retain detailed source records and complete measured metrics. The normalized pack removes repeated text and irrelevant metric columns. `engine-data.json` includes all frozen research recipes and portal inputs.

Run `python verify_knowledge_pack.py` from the extracted bundle to verify transfer, tables, fixtures and the isolated two-file runtime. The original independent engine and build checks are included as scripts and `*-qa.json` results; they access archived research inputs when rerun and therefore require the original workspace. `prepare_engine_data.py` and `generate_build_examples.py` also require that workspace. `build_knowledge_pack.py` can regenerate the consolidated handoff from the included intermediate inputs.

Source aliases refer to paths in the original game workspace and require the original decompilation. The prior evidence archive contains reviewed chapters and game evidence, and excludes complete decompiled source. The original PDF was delivered separately. Portal rules are the recorded September 21, 2026 snapshot.

`manifest.json` lists every packaged file and SHA-256 hash except the manifest itself.

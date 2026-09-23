# DDSPF21 build research — LLM handoff

Use this file for the factual narrative and complete build examples. Use `research-pack.json` for structured facts, caps, trait prerequisites, metrics and source references. The current build evidence is the **Build WAR** section (paired native-engine simulations); its fitted models drive the website builder. `recommendation_engine.py` with `engine-data.json` is the legacy recipe engine (DSFL budgets below 250 only). Stable fact and example IDs connect the sections.

All current research (the native attribute matrix and the build-WAR study) uses the native C# port of the game engine (DDSPFHeadless.Native). The stock DDSPF21 engine is permanently retired for research; older studies that used it (the DSFL 40-game screening with finalist confirmation, and the ISFL attribute-cohort games) are kept as labeled history.

## Scope and evidence

Externally developed and imported ISFL/DSFL rosters. DSFL budget 250; no purchased traits. ISFL maxed templates include all attributes and all purchasable portal traits; the build-WAR rankings use maxed ratings with only the traits whose fitted effect was positive.

Game version: 5.0.11.0. Portal snapshot: 2026-09-21T04:24:15.948833+00:00.

Earlier study (stock engine, superseded by Build WAR): DSFL examples are the highest observed screening-margin profile for each archetype and role: 8,040 screening games and 4,080 confirmation games, four fixed matchups, both treated sides. Screening uses 40 games per arm; finalists use 120 fresh-seed games each in the same matchups. Skills, experience and host teams are held as recorded; injuries are disabled. All 17 simultaneous 95% finalist intervals include zero.

ISFL templates fund every attribute cap and every purchasable portal trait. Their costs and legality are verified; these all-trait templates were not raced head to head (Build WAR measures maxed builds with only the positive-effect traits). Formula leaders below identify specific mathematical strengths.

Source facts describe code; inventory facts summarize reviewed uses. Arithmetic examples illustrate those facts. A named trait with no identified direct effect means no effect was found in the reviewed searchable scope. Automatic TE traits are assumed from the portal; private export mapping remains unverified. Accepted game evidence uses the corrected single-initialization lookup tables.

Notation: R(a,b) is an inclusive integer draw; if a >= b it returns a. E means Endurance in endurance facts; Energy is current state. trunc means toward zero; roundEven means nearest integer with ties to even.

## TPE and portal fields

Starting ratings are free. Each purchased rating point costs according to its destination rating: 1–50: 1 TPE; 51–70: 2; 71–80: 5; 81–90: 10; 91–95: 15; 96–100: 25. Trait fees and their prerequisite rating increases use the same budget.

**Every rating vector uses this order:** STR / AGI / INT / TKL / SPD / HND / PB / RB / END / COMP / KP / KA / ARM / ACC.

These map to strength, agility, intelligence, tackling, speed, hands, passBlocking, runBlocking, endurance, competitiveness, kickPower, kickAccuracy, arm, throwingAccuracy. KP maps to stock KickDistance, ACC to Accuracy, and COMP to Personality.Competitiveness. The other fields map to their capitalized Attributes names.

## Mechanics

- **rng.integer** — R(a,b) inclusive; if a>=b returns a Context: Integer overload only. Example: R(1,100)<80 has probability 79/100. [source_proven; S2:1473-1480]

- **rating.getters** — Getters return Attributes values without generic Energy scaling Context: Energy enters individual formulas explicitly. Example: Energy70 does not itself multiply every rating by 0.7. [source_proven; S1:4136-4272]

- **personality.enabled** — Getter always true Context: Installed 5.0.11.0; disabled fallback branches unreachable. Example: Do not expose a working personality-off simulation toggle. [source_proven; S3:8888-8893]

- **speed.base** — 1.25f+0.015f*Speed Context: Before position, ballcarrier, workload, traits and league modifiers. Example: Speed65=2.225;70=2.300;71=2.315 movement units/iteration. [source_proven; S1:2072-2074]

- **speed.heavy-runner** — Movement x1.05 if Strength>90 AND Weight>235 AND Speed<80 Context: RB/FB carrying on run; before other multipliers. Example: Strength91,Weight236: base+bonus Speed79=2.55675 versus 80=2.45. [source_proven; S1:2317-2320]

- **speed.receiving-rb** — Speed>=80: x.96/.94/.92/.90 at iteration>30/>40/>50/>60; Speed>85 adds x.97 Context: RB carrying on pass; not routes or ordinary rushes. Example: Speed86 crosses an extra negative movement threshold. [source_proven; S1:2185-2206]

- **speed.scramble** — Base 5; Speed<60 sets 0; >70 adds 5; >80 adds 10 Context: QB scramble tendency accumulator before other terms/overrides. Example: Speed70 to71 changes this accumulator by 5. [source_proven; S4:6940-6951]

- **agility.recovery** — P(R(1,A)>20)=0 for A<=20 else(A-20)/A Context: Delay-path recovery branch. Example: A50=60%;A80=75% per qualifying check. [source_proven; S4:6094-6097,7415-7418]

- **agility.receiver** — R(1,A)<20 subtracts 1; probability19/A for A>=20 Context: Receiver GetPassChance term. Example: A80 gives 23.75% chance of this -1 term. [source_proven; S4:12411-12414]

- **strength.release** — R(1,110)<Strength; probability(S-1)/110 Context: Engaged movement release check. Example: Strength90:89/110;91:90/110. [source_proven; S1:158,182]

- **contact.runner** — trunc((4.7*A+2*Speed+5*Strength+4*(Intelligence+min(Experience,8)))*Energy/100) Context: Initial runner composite; before subsequent modifiers and clamp. Example: All relevant ratings80,EXP0,Energy100 gives1256. [source_proven; S4:9696]

- **contact.defender** — trunc((4*Strength+Speed+A+7.5*Tackling+4*(Intelligence+min(Experience,8)))*Energy/100) Context: Initial defender composite; before subsequent modifiers and clamp. Example: All relevant ratings80,EXP0,Energy100 gives1400. [source_proven; S4:9779]

- **contact.final** — Both final scores clamp800..1200; upper1300 if ballcarrier Dualthreat OR RunningQB. Threshold=formationBase+trunc((defender-runner)/10); success iff threshold>R(1,100) Context: After all intermediate modifiers; threshold itself not clamped. Example: Threshold88 gives87% per check. [source_proven; S4:9949-9965]

- **blocking.pass** — 55+(15 if ThrowDeep else 0)+trunc((blockerSTR+2*PB+2*AGI+HANDS+EXP-defenderEXP-defenderSTR-2*defenderTACK-defenderAGI)*.1) Context: Initial pass engagement threshold; later traits, weight, role, skill and other modifiers still apply. Example: +5 PB adds 10 to pre-.1 sum. [source_proven; S1:258]

- **blocking.run** — 40+trunc((blockerSTR+2*RB+EXP+INT-defenderEXP-defenderSTR-2*defenderTACK-defenderAGI)*.1) Context: Initial non-pass engagement threshold; later traits, weight, role, skill and other modifiers still apply. Example: +5 RunBlocking adds 10 to pre-.1 sum. [source_proven; S1:263]

- **blocking.team-pass** — B=sum PB offense slots6..10; R(1,500)>B =>-2; otherwise new R(1,500)<B/2 =>+2 Context: Integer B/2; GetPassChance intermediate term. Example: B500 expected term2*249/500=.996. [source_proven; S4:12423-12431]

- **hands.catch** — Independent R(1,10100)<H adds2; fresh roll<5H adds1; fresh roll<H^2 adds1 Context: Receiver GetPassChance intermediate terms. Example: Quadratic H^2 term still changes within70..79. [source_proven; S4:12387-12410]

- **hands.penalty-cliffs** — One shared R(1,10100): H<80 and roll>9000 subtract1; H<70 and roll>6000 subtract another1 Context: Same shared penalty draw; completion calculation has other terms. Example: 69->70 removes4100/10100 expected penalty points;79->80 removes1100/10100. [source_proven; S4:12387-12410]

- **hands.targeting** — Generic H<80 weight-5 floor1; RB H>80:+15,>70:+10,>60:+5,else-15/floor1; FB>70:+15,>60:+10,else-15/floor1; TE>85:+5,>70:+2,else-10/floor1 Context: Receiver selection weights; role branches. Example: RB80 removes generic penalty;81 changes role bonus. [source_proven; S4:4030-4092]

- **hands.fumble** — Intermediate fumble risk multiplied by(100-H)/100 Context: Other additions precede/follow multiplication; final risk not solely this factor. Example: H100 removes this component; Thumper/COMP additions can remain. [source_proven; S4:9402-9437]

- **drop.attribution** — Drop modifiers classify an already failed pass Context: PassPlay first fails completion comparison; no new catch-success roll. Example: Fewer credited drops need not imply more completions. [source_proven; S4:6647-6734]

- **intelligence.interception** — Base interception denominator: INT1..69=90;70..74=100;75..80=108;81..85=115;86..90=120;91..95=125;96..100=130 Context: QB intelligence ladder before WorkEthic, trait, Accuracy, Arm and skill adjustments. Example: INT 90 uses denominator 120; INT 91 uses 125 before other adjustments. [source_proven; S4:9548-9572]

- **intelligence.decisions** — Subtract R(1,100); positive difference inspects1 target plus extras at>30,>60,>90 Context: Target filtering before final passing decision. Example: INT affects target consideration as well as final roll. [source_proven; S4:4164-4200]

- **accuracy.pass** — Accuracy<65:-3;65..79:0(+1 if INT>=95);80..89:+1;90+:+2 Context: GetPassChance else-if branch. Example: Accuracy80 and90 are separate positive boundaries. [source_proven; S4:12540-12555]

- **accuracy.interception** — Denominator bonus0 through80;81..90:+2;91..95:+4;96+:+5 Context: Interception random denominator, fixed numerator. Example: Accuracy80->81 changes interception range despite no new pass-chance step. [source_proven; S4:9589-9599]

- **arm.interception** — Denominator+1 at>80,+2 at>90; raw QB weighted coefficient1.1 Context: Interception; before final conditional probability. Example: Arm81 and91 cross range steps. [source_proven; S4:9601-9608,9651]

- **arm.deep** — Deep-throw accumulator+5 at>80 and another+5 at>90 Context: Pass decision; preceding context gates remain. Example: Arm91 differs in preference from90. [source_proven; S4:6506-6513]

- **kicking.fg** — Threshold=roundEven((80+KA+KD)*skillModifier); modifier1.02 ifskill>80,.95 if<60,else 1; successR(1,upper)<threshold Context: Modern distance D: upper300+(D-10)+adjustment; adjustment-50 D<20,-40 D<30,-30 D<40,+15 D<50,+100 D>=50 plus50 D>=60; D>=70 forcedfailure. Example: KA90,KD90,skill70,D40 =>259/345. [source_proven; S4:9106-9143,9188]

- **kicking.xp** — Threshold=clamp(115+KA+4*EXP,190,200); successR(1,202)<threshold Context: Isolated extra-point success equation. Example: EXP0 KA85 reaches200 threshold,199/202 chance. [source_proven; S4:9269-9274]

- **kicking.selection** — Field threshold190 KD<80;178 KD80..90;169 KD91..95;160 KD96..98;154 KD99+ Context: Uses maximum KD among ALL roster current-K players; downstream game-state gates. Example: Reserve kicker can change attempt decision. [source_proven; S4:4528-4547]

- **punt.control** — 5+roundEven(KA/20)+roundEven(INT/20), plus5 skill>80 or-10 skill<60; plus5 KA>80 and5 KA>90 Context: Avoid-touchback branch when punt would reach endzone. Example: Rounded twentieth transitions11,30,51,70,91. [source_proven; S4:8244-8266]

- **endurance.loss** — Subtract R(2,6)+away R(0,3)+convertedWRtoRB R(1,4)+ifE<40 R(0,3)+ifE<60 R(0,3); clamp40..100 Context: Current QB exempt; each listed draw independent. Example: Home non-QB E60 removes low-endurance draw. [source_proven; S1:5091-5119]

- **endurance.recovery** — Add R(1,5)+ifE>40 R(0,3)+ifE>70 R(0,3); clamp40..100 Context: Recovery called for all players, including active. Example: E70->71 adds recovery draw mean1.5. [source_proven; S1:5123-5134]

- **endurance.order** — Subtract active offense/defense, then recover EVERY player on both teams Context: CheckFatigue returns early preseason; before new lineup selection. Example: Active players recover too; not bench-only. [source_proven; S4:10600-10620,11437]

- **endurance.carry** — loss18-R(0,floor(E/4)); negative loss replaced1,zero retained; workloadextra7 ifcarries>30,5 if>20,3 if>15 Context: Run ballcarrier; no immediate clamp at assignment. Example: E71 baseline meanloss9.5;E72 mean9. [source_proven; S4:5475-5491]

- **endurance.catch** — loss18-floor(E/10) Context: Completed catch; E is Endurance rating. Example: E69 loss12;70 loss11;80 loss10. [source_proven; S4:6858]

- **endurance.action** — Lose1 ifR(1,100)>Endurance Context: Specific defensive pass-action visits, not every snap. Example: Endurance80 gives20% per visited check. [source_proven; S4:7344-7346,7405-7407]

- **endurance.workload** — q=.90 E<=70,.92 E71..80,.94 E81..90,.96 E91..95,.99 E96+; multiply .99q >10rushes,.95q >15,.92q >20,.90q >30 Context: Movement workload branch; on pass plays >10 attempts q first x1.1; E is rating not current energy. Example: Over30 rushes run factor E70=.8100,E71=.8280. [source_proven; S1:2093-2125]

- **competitiveness.receiving** — R(1,C)>70 adds1 pass-chance point Context: Receiver; personalities enabled (always true in stock). Example: C71 triggers1/71;C80 triggers10/80. [source_proven; S4:12419]

- **competitiveness.fatigue** — At>=5 tackles and noCompetitor, R(1,100)>C multiplies tackle score.9 Context: Separate>=8 tackles .9 remains. Example: C80 triggers20% conditional penalty. [source_proven; S4:9810-9815]

- **competitiveness.fumble** — Runner C>tackler C subtract.5 risk; else add1; final risk x.9 later Context: No personality gate; ties favor defender. Example: C91 versusdefender90 crosses relative boundary. [source_proven; S4:9423-9437]

- **height.receiving** — <72:-1;72:0;73..76:+1;77..80:+2;81+:+3; extra+1 iftaller than closestdefender Context: Inches; GetPassChance receiver modifier. Example: Height73 and76 share base bonus. [source_proven; S4:12367-12385]

- **weight.blocking** — Blocker-opponent >30:+25,>20:+20,>10:+5; opponent-blocker>25:-10,>15:-5;else 0 Context: Relative pounds; blocking intermediate score. Example: Weightdifference20->21 adds15 intermediate points. [source_proven; S1:293-312]

- **experience.contact** — Contact/decision terms commonly min(EXP,8); blocking/penalties/teamplay uncapped Context: Consumer-specific; XP uses4*EXP within own cap. Example: EXP9 addsno extra min(EXP,8) contact term. [source_proven; S4:9696,9779,10055-10056,12227-12264]

- **energy.initial** — min(100,100-InjurySeverity) Context: Game energy initialization. Example: InjurySeverity20 starts Energy80. [source_proven; S4:543,581]

- **skills.common-penalty** — 1..10:15;11..20:10;21..30:8;31..40:6;41..50:4;51..60:2;61..70:1;71+:0 Context: GetPositionPenalty; lined-up/current skill depends on caller. Example: Skill70->71 clears this penalty only. [source_proven; S4:12791]

- **skills.contact** — Current-position skill>80 x1.05; <60 x.95; else 1 Context: Separate from shared position penalty; both contact participants. Example: Skill80->81 changes contact multiplier. [source_proven; S4:9714,9781]

- **skills.movement** — Current skill<30 x.90;30..59 x.95;60+ x1 Context: Final movement skill penalty. Example: Skill59->60 removes.95 multiplier. [source_proven; S1:2490]

- **skills.qb-pass** — QBskill<30:-20;30..39:-15;40..49:-10;50..59:-7;60..69:-5;70+:0 Context: GetPassChance; other skill consumers retain other steps. Example: QBskill69->70 removes5-point penalty. [source_proven; S4:12459-12480]

- **trait.None** — Sentinel, no benefit Example: Lookup None by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:139; S6:1342]

- **trait.Dualthreat** — QB run/scramble movement x1.05, tackle resistance x1.05 in QB-run branch; scramble tendency+30. RunningQB flag overlaps some branches but adds separately in scramble tendency (`Player:2397,2449`;GameFunctions:6952,9742,9952) Example: Lookup Dualthreat by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:140; S6:1345]

- **trait.Gunslinger** — Deep-throw preference accumulator+5 when deep consideration not disabled by fourth down/field position (`6502`) Example: Lookup Gunslinger by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:141; S6:1348]

- **trait.GameManager** — Interception range+4 unless FilmGeek takes precedence (`9581`); throwaway logic bonus min(5,Int/15) (`7516`) Example: Lookup GameManager by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:142; S6:1351]

- **trait.ClutchQB** — Auto-generation found; no direct named game-action use found in full-source symbol search Example: Lookup ClutchQB by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:143; S6:1354]

- **trait.PowerRunner** — Run RB/FB movement x1.02; QB movement x1.05; resistance x1.08 if stronger and separate x1.02 near starting ball position; bypasses ballcarrier path branch (`Player:2325,2401,2453`;GameFunctions:5666,7022,9698,9702) Example: Lookup PowerRunner by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:144; S6:1357]

- **trait.ScatBack** — Pass ballcarrier movement x1.01; RB/FB rushing x1.02; automatic third-down-back preference x1.1 (`Full:Libraries/GamePlan.cs:396`) Example: Lookup ScatBack by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:145; S6:1360]

- **trait.AllPurposeRB** — Drop threshold-3 within nonstacking receiving-trait chain; automatic third-down-back preference x1.1 (`6727`;Full:Libraries/GamePlan:388) Context: The drop-threshold effect is post-failure attribution only, not a second completion check; the receiving drop-trait chain does not stack.. Example: Lookup AllPurposeRB by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:146; S6:1363]

- **trait.BlockingFB** — Blocking score+10 when Position FB; target weight reduced10, floor1 when FB (`Player:273`;GameFunctions:4010) Example: Lookup BlockingFB by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:147; S6:1366]

- **trait.ReceiveFB** — Pass ballcarrier movement x1.05; drop threshold-3 in chain; auto third-down preference x1.05 (`Player:2249`;6723;Full:Libraries/GamePlan:384) Context: The drop-threshold effect is post-failure attribution only; it does not independently change completion success. The receiving drop-trait chain does not stack.. Example: Lookup ReceiveFB by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:148; S6:1369]

- **trait.PossessionWR** — Third-down target weight+5; drop threshold-3; pass ballcarrier movement x.95, a real tradeoff (`3998,6715`;Player:2245) Context: The drop-threshold effect is post-failure attribution only; it does not independently change completion success. The receiving drop-trait chain does not stack.. Example: Lookup PossessionWR by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:149; S6:1372]

- **trait.DeepThreat** — Deep-target weight+10; pass ballcarrier movement x1.01; auto third-down preference x1.1 (`4018`;Player:2241;Full:Libraries/GamePlan:392) Example: Lookup DeepThreat by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:150; S6:1375]

- **trait.SlotReceiver** — Pass ballcarrier movement x1.01; pass-path decision threshold25→15 (`Player:2261`;7025) Example: Lookup SlotReceiver by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:151; S6:1378]

- **trait.ReceivingTE** — TE target weight+2; pass ballcarrier movement x1.05; drop threshold-3 in chain (`4002,6719`;Player:2253) Context: The drop-threshold effect is post-failure attribution only; it does not independently change completion success. The receiving drop-trait chain does not stack.. Example: Lookup ReceivingTE by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:152; S6:1381]

- **trait.BlockingTE** — TE blocking+10; TE target weight reduced10, floor1 (`Player:269`;4006) Example: Lookup BlockingTE by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:153; S6:1384]

- **trait.BookEndTackle** — Auto-generation found; no direct named action use found Example: Lookup BookEndTackle by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:154; S6:1387]

- **trait.TenaciousBlocker** — Blocking score+10, stacks with AthBlocker (`Player:281`) Example: Lookup TenaciousBlocker by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:155; S6:1390]

- **trait.AthBlocker** — Blocking score+10 (`Player:277`) Example: Lookup AthBlocker by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:156; S6:1393]

- **trait.NoseTackle** — Auto-generation found; no direct named action use found Example: Lookup NoseTackle by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:157; S6:1396]

- **trait.BullRusher** — Opposing blocker score-10; pass blitz movement x1.05 (`Player:289,2149`) Example: Lookup BullRusher by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:158; S6:1399]

- **trait.SpeedRusher** — Opposing blocker score-10; pass blitz movement x1.05 (`Player:285,2145`) Example: Lookup SpeedRusher by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:159; S6:1402]

- **trait.CoverageLB** — Auto-generation found; no direct named action use found; game-plan LBRole.Coverage is a separate mechanic Example: Lookup CoverageLB by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:160; S6:1405]

- **trait.Thumper** — Blitz movement x1.05; run recognition branches+2; tackle score x1.03; fumble risk accumulator+2 before final x.9 (`Player:2157`;5579,5777,9419,9818) Example: Lookup Thumper by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:161; S6:1408]

- **trait.HybridLB** — Run tackle score x1.15 against play names containing Option, subject to preceding SS/BoxSafety else-if (`9832`) Example: Lookup HybridLB by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:162; S6:1411]

- **trait.ShutDownCorner** — Passes-defensed credit accumulator+15 in else-if chain (`6802`); this occurs after failed catch, so not proof it prevents the completion Context: The passes-defensed credit effect occurs after a failed catch; that credit effect is not an independent completion-prevention bonus.. Example: Lookup ShutDownCorner by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:163; S6:1414]

- **trait.PressCorner** — Run recognition+1 in chain; passes-defensed credit+2 in chain (`5539,5575,6798`) Context: The passes-defensed credit effect occurs after a failed catch; that credit effect is not an independent completion-prevention bonus.. Example: Lookup PressCorner by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:164; S6:1417]

- **trait.ZoneCorner** — Auto-generation found; no direct named action use found Example: Lookup ZoneCorner by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:165; S6:1420]

- **trait.SlotCorner** — Blitz movement x1.05; RUN ballcarrier path threshold25→15 (`Player:2153`;5669). Source actually says SlotCorner here, not SlotReceiver Example: Lookup SlotCorner by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:166; S6:1423]

- **trait.BoxSafety** — Run recognition+2 or+1 in different chains; SS near-line run tackle x1.03 (`5535,5571,5781,9825`) Example: Lookup BoxSafety by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:167; S6:1426]

- **trait.Centerfielder** — Passes-defensed credit+5 in chain; interception comparison modifier-1 unless Perceptive earlier (`6790,9523`) Context: The passes-defensed credit effect occurs after a failed catch; that credit effect is not an independent completion-prevention bonus.. Example: Lookup Centerfielder by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:168; S6:1429]

- **trait.ClutchKicker** — Auto-generation found; no direct named action use found Example: Lookup ClutchKicker by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:169; S6:1432]

- **trait.PowerKicker** — Auto-generation found; no direct named action use found Example: Lookup PowerKicker by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:170; S6:1435]

- **trait.LockerLeader** — Mentor eligibility and training/social mechanisms; no direct named action use found Example: Lookup LockerLeader by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:171; S6:1438]

- **trait.RawTalent** — RB/FB run movement x1.02 (`Player:2329`) Example: Lookup RawTalent by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:172; S6:1441]

- **trait.WorkoutFanatic** — Training/combine/social uses; no direct named action use found Example: Lookup WorkoutFanatic by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:173; S6:1444]

- **trait.FilmGeek** — Defender interception modifier additional-5 on repeated play; QB interception range+2 taking precedence over GameManager/Legend (`9514,9577`) Example: Lookup FilmGeek by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:174; S6:1447]

- **trait.Competitor** — Bypasses competitiveness tackle fatigue check at>=5 tackles; pass-defense credit+2; training/combine/social uses (`9814,6794`) Context: The passes-defensed credit effect occurs after a failed catch; that credit effect is not an independent completion-prevention bonus.. Example: Lookup Competitor by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:175; S6:1450]

- **trait.ConsummatePro** — Mentoring/training/combine/social; no direct named action use found Example: Lookup ConsummatePro by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:176; S6:1453]

- **trait.RoughDiamond** — No named consumption found in searchable full source; enum/description existence is not an effect Example: Lookup RoughDiamond by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:177; S6:1456]

- **trait.HiddenGem** — Combine effect (`Full:PlayerCombine.cs:99`); no direct named action use found Example: Lookup HiddenGem by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:178; S6:1459]

- **trait.DevelopingStar** — No named consumption found in searchable full source Example: Lookup DevelopingStar by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:179; S6:1462]

- **trait.ProBloodline** — Training/combine/social; no direct named action use found Example: Lookup ProBloodline by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:180; S6:1465]

- **trait.Diva** — Penalty score+2 unless BadInfluence takes precedence; training/social/coach valuation (`10096,10116`) Example: Lookup Diva by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:181; S6:1468]

- **trait.Distraction** — Training/social/coach valuation; no direct named action use found Example: Lookup Distraction by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:182; S6:1471]

- **trait.FanFavorite** — Team fan/bandwagon effects (`Team.cs:1239,1254`), social; no direct named action use found Example: Lookup FanFavorite by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:183; S6:1474]

- **trait.Legend** — Defender interception modifier-3 if no Perceptive/Centerfielder; QB range+4 if no FilmGeek/GameManager; fan/coach/social (`9527,9585`) Example: Lookup Legend by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:184; S6:1477]

- **trait.RoleModel** — Penalty score-10 if earlier BadInfluence/Diva/MediaDarling absent; mentor/training/social (`10104,10124`) Example: Lookup RoleModel by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:185; S6:1480]

- **trait.BadInfluence** — Penalty score+5 and takes precedence over positive traits; training/social/fans (`10092,10112`) Example: Lookup BadInfluence by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:186; S6:1483]

- **trait.OFRedFlags** — Social/coach/fan effects; no direct named action use found Example: Lookup OFRedFlags by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:187; S6:1486]

- **trait.InjuryProne** — Injury threshold-3; training/coach/social effects (`10282,10425,10537`) Example: Lookup InjuryProne by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:188; S6:1489]

- **trait.TeamPlayer** — Named trait penalty score-7, last in chain; distinct from numeric personality TeamPlayer (`10108,10128`) Example: Lookup TeamPlayer by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:189; S6:1492]

- **trait.Athlete** — Movement: blitz x1.05; non-QB pass ballcarrier x1.01; RB/FB run x1.02; QB run/scramble x1.05 (`Player:2141,2237,2321,2393,2445`) Example: Lookup Athlete by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:190; S6:1495]

- **trait.CommunityBenefactor** — Fan/social effects; no direct named action use found Example: Lookup CommunityBenefactor by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:191; S6:1498]

- **trait.MediaDarling** — Penalty score-10 if BadInfluence/Diva absent; social/coach (`10100,10120`) Example: Lookup MediaDarling by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:192; S6:1501]

- **trait.Pick1Overall** — Social/identity use; no direct named game-action benefit found Example: Lookup Pick1Overall by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:193; S6:1504]

- **trait.MrIrellevant** — Social/identity use; no direct named game-action benefit found Example: Lookup MrIrellevant by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:194; S6:1507]

- **trait.Journeyman** — Social use; no direct named game-action benefit found Example: Lookup Journeyman by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:195; S6:1510]

- **trait.Perceptive** — Run recognition+5 or+4 in different branches; defender interception modifier-3, precedence over Centerfielder/Legend (`5531,5567,5773,9519`) Example: Lookup Perceptive by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:196; S6:1513]

- **trait.UnderInvestigation** — Social/coach effects; no direct named action use found Example: Lookup UnderInvestigation by exact enum name; apply only the listed role/branch.. [reviewed_source_audit; S5:197; S6:1516]

- **personality.Competitiveness** — See detailed list below: receiving>70, tackling after 5 tackles, relative fumble and OT checks, QB leadership sum, return and recovery events Context: Postgame morale response buckets at 20/40/60/80 (`3068+`); trait generation and social/development valuation. Example: Use numeric Personality.Competitiveness; distinct from any same-named trait.. [reviewed_source_audit; S5:53]

- **personality.Leadership** — QB decision R(1,250)>Leadership+Intelligence+min(Exp,8), defaults80 if disabled (`6931`); sum with Comp for blitz/tackle decisions; radio-defender teamplay; relative OT x1.03 Context: Mentor/coach fit, social effects and generation; no universal 70 cap. Example: Use numeric Personality.Leadership; distinct from any same-named trait.. [reviewed_source_audit; S5:54]

- **personality.WorkEthic** — QB>90 adds4 to interception random-range parameter (`9573`), **not gated by PersonalitiesEnabled at that use**; radio-defender teamplay also ungated Context: Training, combine, mentor/social/coach effects; website development bypass does not remove these direct uses. Example: Use numeric Personality.WorkEthic; distinct from any same-named trait.. [reviewed_source_audit; S5:55]

- **personality.TeamPlayer** — Sum of all 11 players plus experience for teamplay; radio defender weighted bonus Context: See inversion warning below; cannot assume higher is always better in every formula. Example: Use numeric Personality.TeamPlayer; distinct from any same-named trait.. [reviewed_source_audit; S5:56]

- **personality.Sportsmanship** — Penalty score: <20 +8;20–39 +3;40–60 0;61–80 -3;81+ -8, enabled only with personalities (`10059–10089`) Context: Strict 20/40/61/81 boundaries; contribution can be hidden by outcome clamp/other score terms. Example: Use numeric Personality.Sportsmanship; distinct from any same-named trait.. [reviewed_source_audit; S5:57]

- **personality.SocialDisposition** — No direct GameFunctions on-field expression found Context: Personality/social type, interactions, morale, mentoring, contracts/coach fit; do not label globally inert. Example: Use numeric Personality.SocialDisposition; distinct from any same-named trait.. [reviewed_source_audit; S5:58]

- **personality.Money** — No direct GameFunctions on-field expression found Context: Contract valuation/preferences, social behavior; no benefit to fixed-roster on-field rating identified. Example: Use numeric Personality.Money; distinct from any same-named trait.. [reviewed_source_audit; S5:59]

- **personality.Security** — No direct GameFunctions on-field expression found Context: Contracts and social behavior. Example: Use numeric Personality.Security; distinct from any same-named trait.. [reviewed_source_audit; S5:60]

- **personality.Loyalty** — No direct GameFunctions on-field expression found Context: Contracts/social/coach effects. Example: Use numeric Personality.Loyalty; distinct from any same-named trait.. [reviewed_source_audit; S5:61]

- **personality.Winning** — OT relative check: runner higher x1.07, otherwise tackler x1.07 (`9890`), personality gate Context: Contracts, morale, trait generation. Example: Use numeric Personality.Winning; distinct from any same-named trait.. [reviewed_source_audit; S5:62]

- **personality.PlayingTime** — No direct GameFunctions on-field expression found Context: Contract and attitude/playing-time concerns can change morale. Example: Use numeric Personality.PlayingTime; distinct from any same-named trait.. [reviewed_source_audit; S5:63]

- **personality.CloseToHome** — No direct GameFunctions on-field expression found Context: Contract/social geography preferences. Example: Use numeric Personality.CloseToHome; distinct from any same-named trait.. [reviewed_source_audit; S5:64]

- **personality.MarketSize** — No direct GameFunctions on-field expression found Context: Contract/social preferences. Example: Use numeric Personality.MarketSize; distinct from any same-named trait.. [reviewed_source_audit; S5:65]

- **personality.Morale** — QB pass-read gate uses floor(Morale/10), or90 if disabled (`6630–6631`) Context: Injury and game results modify it; UI display buckets are not gameplay breakpoints. Example: Use numeric Personality.Morale; distinct from any same-named trait.. [reviewed_source_audit; S5:66]

- **skills.extra** — All blockers | Current skill>80 adds R(1,5) to engagement threshold; <60 subtracts R(1,5);60–80 neutral (`Player.cs:357–365`). Thus skill81 has a blocking effect as well as tackle effects Context: Scope and condition clauses within the recorded source summary apply.. [reviewed_source_audit; S5:95]

- **skills.run** — RB,FB rushing movement | Skill<50 x.98;50 neutral;51–80 x1.02;81–90 x1.04;91+ x1.05 (`Player.cs:2287+`) Context: Scope and condition clauses within the recorded source summary apply.. [reviewed_source_audit; S5:100]

- **skills.pass-carry** — TE/RB/WR with ball on pass | Skill<50 x.98;50 neutral;51–70 x1.01;71–85 x1.02;86+ x1.03. TE uses TE,RB uses RB, all other non-QB players use WR here, even FB (`2210–2234`) Context: Scope and condition clauses within the recorded source summary apply.. [reviewed_source_audit; S5:101]

- **skills.drop** — Failed-pass drop attribution | Current skill<=60 +5;61–70 +3;71–80 +2;81–90 +1;91–95 0;96+ -2 to drop threshold (`6691–6713`), whole threshold later clamped2–30 Context: Scope and condition clauses within the recorded source summary apply.. [reviewed_source_audit; S5:102]

- **skills.lb** — LB assignment | Lined-up LB skill<60 can trigger early Delay using R(1,Intelligence)<15 (`5786,7132`) Context: Scope and condition clauses within the recorded source summary apply.. [reviewed_source_audit; S5:103]

- **skills.kick** — K,P | Skill>80 reduces block threshold1; <60 raises4 (`8096,8512,9107`); P accuracy/disposition computations use >80/<60 too (`8226,8249`) Context: Scope and condition clauses within the recorded source summary apply.. [reviewed_source_audit; S5:104]

- **injury.gates** — Injury checks skip exhibition/Pro Bowl and DisableInjuries (`10248–10256,10392–10404`). Random-player injury gate is R(1,100)<8; targeted-player gate <10, with TeamID=0 excluded. They draw an injury severity and a second roll. Random-player threshold n=96, -1 at E<50, +5 at E>75, -3 InjuryProne (`10273–10285`). Targeted n=96, -3 at E<50, +3 at E>75, -3 InjuryProne (`10416–10428`). Severity>70 succeeds when second roll>n; failing that, severity>30 succeeds when roll>n-5; severity<=30 when roll>n-35. **A severe injury draw can still succeed via the second branch**: E76 random-player n101 eliminates first comparison >101, but second comparison >96 still permits a severe injury. Do not claim immunity. With InjuryProne n98 instead. Training/off-field overload uses the latter threshold family (`10529–10540`) and a league InjurySetting gate. Context: Scope and condition clauses within the recorded source summary apply.. [reviewed_source_audit; S5:41]

- **substitution.energy** — `GameFunctions.cs:11509–11555`: if Energy<90, a candidate can be skipped on `R(1,E)<threshold`. RB and WR threshold 15 (14/E probability); the subsequent WR-only threshold 10 branch is unreachable because WR matched already. DT 8; DE 15; G/T/C 5; other positions default 0. This is within offensive lineup selection, so listing DT/DE here is not proof these are the ordinary defensive rotation rules. Main offense selection also requires Energy>80, injury severity<10, workload below dynamically selected limits, and no duplicate player. E>80 adds R(1,4) to carry limit; E<60 subtracts R(1,4). Main defensive selection similarly checks Energy>80 (`11727`), with fallback paths relaxing energy requirements. A skip is not guaranteed to bench the player for the whole snap: fallback logic and available depth matter. Context: Scope and condition clauses within the recorded source summary apply.. [reviewed_source_audit; S5:39]

- **teamplay.sign** — `GetPassChance:12432–12439` SUBTRACTS positive teamplaymod and adds absolute negative mod: more offensive teamplay lowers this intermediate pass score. `CheckTackle:9930–9938` adjusts a tackle threshold in the same sign pattern but a different comparison. Treat the pass result as a code anomaly requiring isolated stock verification, not blanket advice to minimize TeamPlayer. The morale and off-field costs of low ratings still exist. Even the theoretical disabled branch would not make every personality read inert (fumble, QB WorkEthic and radio defender remain). In the actual stock assembly the getter is always true. Context: Scope and condition clauses within the recorded source summary apply.. [reviewed_source_audit; S5:84]

- **flags.running-qb** — The entire `PlayerFlags` enum consists of None=0 and RunningQB=1. RunningQB is not the same stored field as Dualthreat, although some branches accept either: Context: Scope and condition clauses within the recorded source summary apply.. [reviewed_source_audit; S5:125]

- **age.scope** — Tackle resistance gives ballcarrier x1.05 if strictly heavier (`9706`); rushing movement RB/FB receives x1.05 for Strength>90 AND Weight>235 AND Speed<80 (`Player.cs:2317`). Weight also affects overall/selection scores and auto-generated traits, so fixed lineup testing should separate physical mechanics from changing who gets selected. No direct age term found in snap resolution; age appears in postgame skill progression (`2779`), generation, valuation, contracts and development. Do not pay for age reduction expecting a direct speed boost if attributes/state are identical. Context: Scope and condition clauses within the recorded source summary apply.. [reviewed_source_audit; S5:113]

- **potential.scope** — Potential: no direct snap formula found in GameFunctions. In `Player.GetValue:6774–6784`, Potential>90 +.5,>80 +.3,<30 -.5 affects player evaluation; it also participates in creation and training. This can affect selection/development, not an automatic game-action multiplier. Parent report covers full selection paths; do not equate a display/value boost with better performance while roster and deployment are held fixed. Context: Scope and condition clauses within the recorded source summary apply.. [reviewed_source_audit; S5:117]

- **flags.running-qb.use-127** — `Player.CurrentSpeed:2397,2449`: QB run and scramble movement x1.05 for Dualthreat OR RunningQB. Having both does not double this multiplier. Context: RunningQB is a separate flag from Dualthreat; OR tests do not double-stack.. [reviewed_source_audit; S5:127]

- **flags.running-qb.use-128** — `GameFunctions:6952–6960`: scramble tendency adds 30 for Dualthreat and separately 15 for RunningQB; here they do stack. Game-plan QBScrambling can override the accumulator. Context: RunningQB is a separate flag from Dualthreat; OR tests do not double-stack.. [reviewed_source_audit; S5:128]

- **flags.running-qb.use-129** — `9742`: QB-run tackle resistance x1.05 for either, not twice. `9952`: either expands the final tackle-score ceiling from 1200 to 1300 **for both runner and defender**, not just the QB. Context: RunningQB is a separate flag from Dualthreat; OR tests do not double-stack.. [reviewed_source_audit; S5:129]

- **flags.running-qb.use-130** — `9758–9776`: RunningQB changes the QB half-resistance test threshold from 50 to 10; at Leadership+Comp 160 the event changes from 49/160 to 9/160 where that branch is reached. With the flag, an Option-named play behind the line instead gives x1.2; another run behind the line gives x1.1 and bypasses the later half-resistance branch. Context: RunningQB is a separate flag from Dualthreat; OR tests do not double-stack.. [reviewed_source_audit; S5:130]

- **flags.running-qb.use-131** — `4142`: run ballcarrier selection weight gets+15 for RunningQB only after the preceding QB workload>15/>10/>5 branches fail; thus the bonus is reached at<=5 recorded attempts in that chain. This affects opportunity rather than per-contact power. Context: RunningQB is a separate flag from Dualthreat; OR tests do not double-stack.. [reviewed_source_audit; S5:131]

- **flags.running-qb.use-132** — `Full:Coach.cs:2602–2633`: a flagged package QB permits QB-designed runs in automatically generated offensive playbooks. `Full:Functions.cs:8514` influences automatic scheme selection along with Speed>65. Context: RunningQB is a separate flag from Dualthreat; OR tests do not double-stack.. [reviewed_source_audit; S5:132]

- **flags.running-qb.use-133** — Generation uses different default-rating arrays (`Full:Functions.cs:1707`), draft rating adjustments (`Full:Draft.cs:2960`), and a random 70–80 minimum speed floor (`Player.cs:1350`). `AssignFlags:9902` may add Dualthreat when RunningQB and Speed>70 and Agility>65. None of these generation functions implies that simply toggling the flag immediately mutates an already-imported player's ratings; inspect the specific import/generation caller. Context: RunningQB is a separate flag from Dualthreat; OR tests do not double-stack.. [reviewed_source_audit; S5:133]

- **interception.composite** — QBscore=trunc((4I+4ACC+1.1ARM+Energy+min(EXP,8)+context)/10); DEFscore=trunc((AGI+HANDS+trunc(1.35I)+trunc(SPEED/2.3)+Energy+min(EXP,8)-context)/10); numerator=clamp(6+DEFscore-QBscore,6,60) Context: Recognized-play adjustments+15/-15 occur after numerator clamp; context itself has multiple inputs. Example: Numerator6 can stay unchanged as QB ratings rise. [source_proven; S4:9651-9664]

- **punt.distance** — trunc((45+R(15,KD))*.01*skillModifier*150) field units; KD>95 gives independent R(1,100)<5 gate then R(1,60) extra; KA>95 gives same4% gate then R(1,30) Context: Normal punt distance before endzone handling and clamps. Example: KD95->96 unlocks an extra4% branch. [source_proven; S4:8225-8242]

- **kickoff.distance** — R(65,max(KD,75))*.01 times skill modifier; distance=trunc(factor*(90+floor(KD/20))) Context: Mishit branch and downstream field clipping also apply; not complete play outcome. Example: KD below75 keeps this random upper bound fixed but floor(KD/20) still changes. [source_proven; S4:8511-8524]

- **return.kickoff** — R(minimum,Speed+Agility-60); 1% long-return gate addsR(0,2*(Speed+Agility)) Context: Before field clipping and ceiling-to-yards; inverted-range draw returns minimum. Example: Combined Speed+Agility affects range rather than a universal individual rating bucket. [source_proven; S4:8612-8618]

- **blocking.success** — Engagement succeeds iffR(1,100)<threshold; boolean true is reserved for pancake Context: Normal engagement also returnsfalse; inspect EngagedInBlock or pancake state. Example: Threshold70 implies69% initial comparison, before other branches. [source_proven; S1:366-386]

- **competitiveness.long-gain** — Runner resistance x.97 ifR(1,200)>Endurance+Competitiveness Context: Ballcarrier Y-starting ballY>240 field units (80yards). Example: Sum200 removes only this penalty draw. [source_proven; S4:9747-9753]

- **competitiveness.overtime** — RunnerC>defenderC =>runnerx1.05;else defenderx1.05 Context: Overtime contact; personality enabled. Example: Ties favor defender. [source_proven; S4:9874-9882]

- **competitiveness.recovery** — Redraw selected fumble recoverer ifR(1,150)>Agility+Competitiveness Context: Redraw may choose same player; not possession guarantee. Example: Combined150 removes this redraw. [source_proven; S4:7814]

- **teamplay.sum** — Sum TeamPlayer+EXP per offense/defense; radio defender addsfloor((2INT+4EXP+TeamPlayer+WorkEthic+Leadership)/10). Offense-defensedifference0=>0;positive=>+1;negative=>-1;absdifference>200 adds another same-sign1 Context: Numeric personality TeamPlayer, not PlayerTraits.TeamPlayer. Example: Difference200=>+1;201=>+2. [source_proven; S4:12227-12264]

- **traits.drop-chain** — PossessionWR elseReceivingTE elseReceiveFB elseAllPurposeRB: drop threshold-3 once Context: Only after pass already failed completion comparison. Example: All four together still apply only-3. [source_proven; S4:6647,6715-6729]

- **traits.qb-int-chain** — FilmGeek+2 elseGameManager+4 elseLegend+4 interception denominator Context: QB branch; other contexts separate. Example: FilmGeek+GameManager uses+2 here. [source_proven; S4:9577-9586]

- **traits.def-int-chain** — Perceptive-3 elseCenterfielder-1 elseLegend-3 comparison modifier Context: Defender interception branch. Example: Centerfielder+Legend uses-1 here. [source_proven; S4:9519-9529]

## Field lookup

### Core Fields

- Strength: rating.getters, speed.heavy-runner, strength.release, contact.runner, contact.defender, contact.final, blocking.pass, blocking.run
- Agility: rating.getters, agility.recovery, agility.receiver, contact.runner, contact.defender, blocking.pass, blocking.run, interception.composite, return.kickoff, competitiveness.recovery
- Speed: rating.getters, speed.base, speed.heavy-runner, speed.receiving-rb, speed.scramble, contact.runner, contact.defender, interception.composite, return.kickoff
- Hands: rating.getters, blocking.pass, hands.catch, hands.penalty-cliffs, hands.targeting, hands.fumble, drop.attribution, skills.drop, interception.composite
- Intelligence: rating.getters, contact.runner, contact.defender, blocking.run, intelligence.interception, intelligence.decisions, accuracy.pass, punt.control, skills.lb, interception.composite, teamplay.sum
- Tackling: rating.getters, contact.defender, contact.final, blocking.pass, blocking.run
- PassBlocking: rating.getters, blocking.pass, blocking.team-pass, blocking.success
- RunBlocking: rating.getters, blocking.run, blocking.success
- Endurance: rating.getters, endurance.loss, endurance.recovery, endurance.order, endurance.carry, endurance.catch, endurance.action, endurance.workload, injury.gates, substitution.energy, competitiveness.long-gain
- Arm: rating.getters, arm.interception, arm.deep, interception.composite
- Accuracy: rating.getters, accuracy.pass, accuracy.interception, interception.composite
- KickAccuracy: rating.getters, kicking.fg, kicking.xp, punt.control, skills.kick, punt.distance
- KickDistance: rating.getters, kicking.fg, kicking.selection, punt.distance, kickoff.distance
- Competitiveness: competitiveness.receiving, competitiveness.fatigue, competitiveness.fumble, personality.Competitiveness, competitiveness.long-gain, competitiveness.overtime, competitiveness.recovery

### Personality Fields

- Competitiveness: competitiveness.receiving, competitiveness.fatigue, competitiveness.fumble, personality.Competitiveness, competitiveness.long-gain, competitiveness.overtime, competitiveness.recovery
- Leadership: personality.Leadership, teamplay.sum
- WorkEthic: personality.WorkEthic, teamplay.sum
- TeamPlayer: personality.TeamPlayer, teamplay.sign, teamplay.sum
- Sportsmanship: personality.Sportsmanship
- SocialDisposition: personality.SocialDisposition
- Money: personality.Money
- Security: personality.Security
- Loyalty: personality.Loyalty
- Winning: personality.Winning
- PlayingTime: personality.PlayingTime
- CloseToHome: personality.CloseToHome
- MarketSize: personality.MarketSize
- Morale: personality.Morale

### Position Skills

- QB: skills.common-penalty, skills.contact, skills.movement, skills.qb-pass, skills.extra
- RB: skills.common-penalty, skills.contact, skills.movement, skills.run, skills.pass-carry, skills.drop, skills.extra
- FB: skills.common-penalty, skills.contact, skills.movement, skills.run, skills.extra
- WR: skills.common-penalty, skills.contact, skills.movement, skills.pass-carry, skills.drop, skills.extra
- TE: skills.common-penalty, skills.contact, skills.movement, skills.pass-carry, skills.drop, skills.extra
- G: skills.common-penalty, skills.contact, skills.movement, skills.extra
- T: skills.common-penalty, skills.contact, skills.movement, skills.extra
- C: skills.common-penalty, skills.contact, skills.movement, skills.extra
- P: skills.common-penalty, skills.contact, skills.movement, skills.kick, skills.extra
- K: skills.common-penalty, skills.contact, skills.movement, skills.kick, skills.extra
- DT: skills.common-penalty, skills.contact, skills.movement, skills.extra
- DE: skills.common-penalty, skills.contact, skills.movement, skills.extra
- LB: skills.common-penalty, skills.contact, skills.movement, skills.lb, skills.extra
- CB: skills.common-penalty, skills.contact, skills.movement, skills.extra
- SS: skills.common-penalty, skills.contact, skills.movement, skills.extra
- FS: skills.common-penalty, skills.contact, skills.movement, skills.extra

### Traits

- None: trait.None
- Dualthreat: trait.Dualthreat
- Gunslinger: trait.Gunslinger
- GameManager: trait.GameManager
- ClutchQB: trait.ClutchQB
- PowerRunner: trait.PowerRunner
- ScatBack: trait.ScatBack
- AllPurposeRB: trait.AllPurposeRB
- BlockingFB: trait.BlockingFB
- ReceiveFB: trait.ReceiveFB
- PossessionWR: trait.PossessionWR
- DeepThreat: trait.DeepThreat
- SlotReceiver: trait.SlotReceiver
- ReceivingTE: trait.ReceivingTE
- BlockingTE: trait.BlockingTE
- BookEndTackle: trait.BookEndTackle
- TenaciousBlocker: trait.TenaciousBlocker
- AthBlocker: trait.AthBlocker
- NoseTackle: trait.NoseTackle
- BullRusher: trait.BullRusher
- SpeedRusher: trait.SpeedRusher
- CoverageLB: trait.CoverageLB
- Thumper: trait.Thumper
- HybridLB: trait.HybridLB
- ShutDownCorner: trait.ShutDownCorner
- PressCorner: trait.PressCorner
- ZoneCorner: trait.ZoneCorner
- SlotCorner: trait.SlotCorner
- BoxSafety: trait.BoxSafety
- Centerfielder: trait.Centerfielder
- ClutchKicker: trait.ClutchKicker
- PowerKicker: trait.PowerKicker
- LockerLeader: trait.LockerLeader
- RawTalent: trait.RawTalent
- WorkoutFanatic: trait.WorkoutFanatic
- FilmGeek: trait.FilmGeek
- Competitor: trait.Competitor
- ConsummatePro: trait.ConsummatePro
- RoughDiamond: trait.RoughDiamond
- HiddenGem: trait.HiddenGem
- DevelopingStar: trait.DevelopingStar
- ProBloodline: trait.ProBloodline
- Diva: trait.Diva
- Distraction: trait.Distraction
- FanFavorite: trait.FanFavorite
- Legend: trait.Legend
- RoleModel: trait.RoleModel
- BadInfluence: trait.BadInfluence
- OFRedFlags: trait.OFRedFlags
- InjuryProne: trait.InjuryProne
- TeamPlayer: trait.TeamPlayer
- Athlete: trait.Athlete
- CommunityBenefactor: trait.CommunityBenefactor
- MediaDarling: trait.MediaDarling
- Pick1Overall: trait.Pick1Overall
- MrIrellevant: trait.MrIrellevant
- Journeyman: trait.Journeyman
- Perceptive: trait.Perceptive
- UnderInvestigation: trait.UnderInvestigation

## DSFL: every archetype and tested role

Each row is the selected allocation within that archetype and role. Ratings are in the 14-field order above. Margin is mean treated-team points for minus points against. Paid traits: none. Automatic portal assumptions: Vertical Threat → DeepThreat; Possession TE → Athlete. Dimensions and starting ratings appear in the catalog below.

| Archetype | Role | Profile | Ratings | TPE | Screen margin | Confirm margin |
|---|---|---|---|---:|---:|---:|
| Accurate | K/P | punt_control | 1/1/72/1/1/1/1/1/1/1/80/90/1/1 | 250 | +2.525 | +2.000 |
| All-Purpose DT | DT | interior_contact | 82/51/61/77/62/20/1/1/72/42/1/1/1/1 | 250 | +3.225 | -1.975 |
| Athletic Lineman | C | low_endurance_reallocation | 75/64/64/1/55/54/75/75/60/1/1/1/1/1 | 249 | +1.225 | — |
| Athletic Lineman | G | pass_protection | 72/62/62/1/55/52/82/56/73/1/1/1/1/1 | 249 | +7.050 | +1.283 |
| Athletic Lineman | T | run_blocking | 76/51/66/1/55/41/70/81/71/1/1/1/1/1 | 250 | +3.000 | -2.017 |
| Balanced Lineman | C | low_endurance_reallocation | 74/64/63/1/45/53/74/74/60/1/1/1/1/1 | 250 | +2.300 | — |
| Balanced Lineman | G | low_endurance_reallocation | 74/64/63/1/45/53/74/74/60/1/1/1/1/1 | 250 | +6.400 | -1.917 |
| Balanced Lineman | T | balanced_workload | 72/63/62/1/46/52/72/72/73/1/1/1/1/1 | 250 | +1.325 | +2.158 |
| Ball Hawk | FS | deep_coverage | 40/70/70/35/85/60/1/1/55/40/1/1/1/1 | 250 | +2.550 | -0.392 |
| Ball Hawk | SS | box_contact | 70/55/65/76/70/40/1/1/71/40/1/1/1/1 | 250 | +1.125 | — |
| Blocking TE | TE | blocking_assignment | 75/60/60/1/45/45/71/70/71/40/1/1/1/1 | 250 | +0.900 | -0.692 |
| Center Fielder | FS | box_contact | 70/51/66/75/71/35/1/1/71/36/1/1/1/1 | 250 | +1.950 | -1.042 |
| Center Fielder | SS | deep_coverage | 35/70/70/36/85/60/1/1/52/35/1/1/1/1 | 250 | +1.150 | -0.750 |
| Cover Corner | CB | low_endurance_reallocation | 63/74/69/69/79/48/1/1/60/35/1/1/1/1 | 250 | +1.275 | — |
| Coverage LB | MLB | sustained_balanced | 65/50/65/70/70/35/1/1/75/65/1/1/1/1 | 250 | +2.725 | — |
| Coverage LB | OLB | coverage_reactions | 40/70/82/35/76/50/1/1/71/30/1/1/1/1 | 250 | +0.650 | — |
| Enforcer | FS | deep_coverage | 45/70/70/50/85/60/1/1/40/45/1/1/1/1 | 250 | +1.125 | — |
| Enforcer | SS | deep_coverage | 45/70/70/50/85/60/1/1/40/45/1/1/1/1 | 250 | +1.325 | +2.775 |
| Field General | QB | passing_security | 40/60/81/1/45/1/1/1/45/60/1/1/72/81 | 250 | +0.775 | — |
| Fullback | FB | receiving_workload | 60/70/52/1/75/70/40/40/71/41/1/1/1/1 | 250 | +0.075 | -2.142 |
| Gunslinger | QB | passing_security | 40/60/82/1/45/1/1/1/45/60/1/1/71/81 | 250 | +4.950 | -0.508 |
| Interior Rusher | DT | low_endurance_reallocation | 76/66/67/76/66/20/1/1/60/66/1/1/1/1 | 250 | +3.950 | -1.483 |
| Mauler | C | balanced_workload | 71/62/62/1/35/51/72/71/72/1/1/1/1/1 | 250 | +5.900 | -1.650 |
| Mauler | G | pass_protection | 71/62/61/1/35/51/81/47/72/1/1/1/1/1 | 250 | +1.775 | — |
| Mauler | T | balanced_workload | 71/62/62/1/35/51/72/71/72/1/1/1/1/1 | 250 | +0.050 | — |
| Mobile | QB | vertical_passing | 65/41/67/1/65/35/1/1/50/50/1/1/91/65 | 250 | +0.300 | — |
| Nose Tackle | DT | interior_contact | 80/40/60/75/60/20/1/1/71/40/1/1/1/1 | 250 | +0.775 | — |
| Pass Rusher | MLB | low_endurance_reallocation | 68/58/68/73/73/21/1/1/60/68/1/1/1/1 | 250 | +4.100 | -0.683 |
| Pass Rusher | OLB | sustained_balanced | 65/55/65/70/70/20/1/1/77/65/1/1/1/1 | 250 | +2.775 | -1.767 |
| Physical Corner | CB | low_endurance_reallocation | 62/72/67/67/78/43/1/1/60/40/1/1/1/1 | 250 | +2.100 | -0.167 |
| Pocket Passer | QB | passing_security | 40/60/82/1/45/1/1/1/45/60/1/1/71/81 | 250 | +2.450 | +0.033 |
| Possession Receiver | KR | route_movement | 46/75/52/1/85/70/20/20/71/40/1/1/1/1 | 250 | -0.650 | — |
| Possession Receiver | WR | route_movement | 46/75/52/1/85/70/20/20/71/40/1/1/1/1 | 250 | +0.750 | -1.525 |
| Possession TE | TE | low_endurance_reallocation | 67/45/62/1/72/73/63/63/60/40/1/1/1/1 | 250 | +0.450 | -0.067 |
| Power | K/P | punt_control | 1/1/72/1/1/1/1/1/1/1/80/90/1/1 | 250 | +2.525 | — |
| Power Back | RB | movement | 60/70/40/1/85/70/10/10/71/40/1/1/1/1 | 250 | +1.425 | -0.992 |
| Power Rusher | DE | low_endurance_reallocation | 74/54/65/74/69/21/1/1/60/69/1/1/1/1 | 250 | -0.125 | +0.242 |
| Receiving Back | RB | movement | 46/70/57/1/85/70/10/10/71/40/1/1/1/1 | 250 | -3.150 | — |
| Return Specialist | KR | return_movement | 40/85/45/1/85/50/20/20/40/30/1/1/1/1 | 250 | -1.250 | — |
| Return Specialist | WR | low_endurance_reallocation | 69/73/54/1/78/64/20/20/60/53/1/1/1/1 | 250 | +0.050 | -1.433 |
| Run Stuffer | DE | low_endurance_reallocation | 74/48/64/74/69/20/1/1/60/68/1/1/1/1 | 250 | -0.450 | — |
| Scrambler | QB | passing_security | 45/60/81/1/70/36/1/1/50/57/1/1/71/80 | 250 | -0.975 | — |
| Slot Corner | CB | movement_coverage | 50/75/65/40/85/36/1/1/67/45/1/1/1/1 | 250 | -0.750 | — |
| Slot Receiver | KR | low_endurance_reallocation | 40/72/63/1/78/83/20/20/60/44/1/1/1/1 | 250 | +1.750 | -3.233 |
| Slot Receiver | WR | low_endurance_reallocation | 40/72/63/1/78/83/20/20/60/44/1/1/1/1 | 250 | -0.150 | — |
| Speed Back | RB | movement | 30/70/35/1/85/70/10/10/65/35/1/1/1/1 | 250 | -0.400 | -1.975 |
| Speed Receiver | KR | catch_targeting | 30/71/60/1/76/81/5/5/71/30/1/1/1/1 | 250 | -0.900 | — |
| Speed Receiver | WR | catch_targeting | 30/71/60/1/76/81/5/5/71/30/1/1/1/1 | 250 | -0.475 | — |
| Speed Rusher | DE | low_endurance_reallocation | 74/59/65/74/69/21/1/1/60/69/1/1/1/1 | 250 | +2.850 | +1.208 |
| Versatile LB | MLB | coverage_reactions | 60/70/79/65/75/50/1/1/71/40/1/1/1/1 | 250 | +2.850 | +1.500 |
| Versatile LB | OLB | contact_support | 77/52/62/82/71/25/1/1/72/42/1/1/1/1 | 250 | +5.150 | -0.108 |
| Vertical Threat | TE | sustained_hybrid | 65/45/55/1/70/70/60/60/71/35/1/1/1/1 | 250 | +0.100 | — |

Default display roles: OL → C, LB → MLB, S → FS, Return Specialist → KR, specialists → K/P. The JSON maps all 36 defaults to the 52 rows.

### Archetype comparisons by role

The screening column ranks all tested candidates. Confirmation compares two selected finalists; its leader can differ from the screening leader. The paired interval is A minus B, with candidate names given explicitly in JSON. No simultaneous interval resolves a winner.

| Role | Screening leader | Confirmation mean leader | Confirm mean margin | A / B candidate IDs | A−B mean | Simultaneous 95% interval |
|---|---|---|---:|---|---:|---|
| QB | Gunslinger | Pocket Passer | +0.033 | gunslinger__passing_security / pocket-passer__passing_security | -0.542 | [-3.996, +2.913] |
| RB | Power Back | Power Back | -0.992 | power-back__movement / speed-back__movement | +0.983 | [-3.818, +5.785] |
| FB | Fullback | Fullback | -1.658 | fullback__receiving_workload / fullback__contact_workload | -0.483 | [-6.080, +5.113] |
| WR | Possession Receiver | Return Specialist | -1.433 | possession-receiver__route_movement / return-specialist__low_endurance_reallocation | -0.092 | [-4.971, +4.788] |
| KR | Slot Receiver | Slot Receiver | -1.158 | slot-receiver__low_endurance_reallocation / slot-receiver__catch_targeting | -2.075 | [-8.267, +4.117] |
| TE | Blocking TE | Possession TE | -0.067 | blocking-te__blocking_assignment / possession-te__low_endurance_reallocation | -0.625 | [-5.429, +4.179] |
| C | Mauler | Mauler | -1.650 | mauler__balanced_workload / mauler__run_blocking | +0.367 | [-4.802, +5.536] |
| G | Athletic Lineman | Athletic Lineman | +1.283 | athletic-lineman__pass_protection / balanced-lineman__low_endurance_reallocation | +3.200 | [-1.823, +8.223] |
| T | Athletic Lineman | Balanced Lineman | +2.158 | athletic-lineman__run_blocking / balanced-lineman__balanced_workload | -4.175 | [-9.532, +1.182] |
| DE | Speed Rusher | Speed Rusher | +1.208 | speed-rusher__low_endurance_reallocation / power-rusher__low_endurance_reallocation | +0.967 | [-4.051, +5.985] |
| DT | Interior Rusher | Interior Rusher | -1.483 | interior-rusher__low_endurance_reallocation / all-purpose-dt__interior_contact | +0.492 | [-4.772, +5.756] |
| MLB | Pass Rusher | Versatile LB | +1.500 | pass-rusher__low_endurance_reallocation / versatile-lb__coverage_reactions | -2.183 | [-7.346, +2.980] |
| OLB | Versatile LB | Versatile LB | -0.108 | versatile-lb__contact_support / pass-rusher__sustained_balanced | +1.658 | [-3.590, +6.907] |
| CB | Physical Corner | Physical Corner | +0.200 | physical-corner__low_endurance_reallocation / physical-corner__movement_coverage | -0.367 | [-5.610, +4.877] |
| FS | Ball Hawk | Ball Hawk | -0.392 | ball-hawk__deep_coverage / center-fielder__box_contact | +0.650 | [-4.486, +5.786] |
| SS | Enforcer | Enforcer | +2.775 | enforcer__deep_coverage / center-fielder__deep_coverage | +3.525 | [-1.817, +8.867] |
| K/P | Accurate | Accurate | +2.000 | accurate__punt_control / accurate__power_first | +1.300 | [-3.729, +6.329] |

## ISFL: all attributes and all purchasable traits

Each row gives the free starting allocation, fully maxed allocation, dimensions, and cost split. Base caps equal the maxed vector except Accurate KP starts capped at 90 and Power KA at 90. Their two sequential unlocks raise the respective cap to 95, then 100. Both full specialist builds cost 1,240 TPE, including INT 90 and 350 TPE in unlock traits.

| Archetype | Position | Height in / weight lb | Free starting ratings | Fully maxed ratings | Attributes + traits = total TPE | Paid traits | Automatic traits |
|---|---|---|---|---|---|---|---|
| Speed Receiver | WR | 70/195 | 30/55/40/1/60/55/5/5/50/30/1/1/1/1 | 65/85/80/1/100/85/20/20/90/70/1/1/1/1 | 1060 + 150 = 1210 | DeepThreat, Athlete, RoleModel | — |
| Return Specialist | WR | 69/175 | 40/70/30/1/60/35/20/20/40/30/1/1/1/1 | 80/100/80/1/100/70/35/35/70/65/1/1/1/1 | 1115 + 50 = 1165 | RoleModel | — |
| Possession Receiver | WR | 77/230 | 45/60/45/1/50/65/20/20/50/40/1/1/1/1 | 75/90/85/1/90/95/35/35/90/80/1/1/1/1 | 1130 + 150 = 1280 | SlotReceiver, Athlete, RoleModel | — |
| Slot Receiver | WR | 73/205 | 40/65/40/1/55/60/20/20/50/40/1/1/1/1 | 70/95/80/1/95/90/35/35/90/80/1/1/1/1 | 1130 + 150 = 1280 | SlotReceiver, Athlete, RoleModel | — |
| Blocking TE | TE | 76/265 | 50/40/45/1/45/45/50/50/50/40/1/1/1/1 | 90/75/80/1/85/80/85/85/80/80/1/1/1/1 | 1070 + 150 = 1220 | BlockingTE, RoleModel | — |
| Possession TE | TE | 78/255 | 50/45/45/1/50/55/40/40/50/40/1/1/1/1 | 85/80/80/1/85/90/65/65/90/80/1/1/1/1 | 1020 + 200 = 1220 | ReceivingTE, RoleModel | Athlete |
| Vertical Threat | TE | 76/250 | 40/45/45/1/55/50/30/30/50/35/1/1/1/1 | 80/85/80/1/90/85/60/60/90/75/1/1/1/1 | 1010 + 200 = 1210 | ReceivingTE, RoleModel | DeepThreat |
| Mauler | OL | 79/325 | 65/45/45/1/35/30/45/45/50/1/1/1/1/1 | 100/65/70/1/60/60/90/95/75/1/1/1/1/1 | 1045 + 200 = 1245 | TenaciousBlocker, RoleModel | — |
| Balanced Lineman | OL | 77/315 | 60/40/45/1/45/35/55/55/50/1/1/1/1/1 | 95/70/75/1/70/65/95/95/75/1/1/1/1/1 | 1030 + 200 = 1230 | TenaciousBlocker, RoleModel | — |
| Athletic Lineman | OL | 75/310 | 55/50/50/1/55/40/60/55/50/1/1/1/1/1 | 90/80/80/1/75/70/100/90/75/1/1/1/1/1 | 1080 + 200 = 1280 | TenaciousBlocker, RoleModel | — |
| Speed Rusher | DE | 75/300 | 55/55/45/55/50/20/1/1/40/50/1/1/1/1 | 85/95/75/85/80/60/1/1/80/85/1/1/1/1 | 965 + 250 = 1215 | SpeedRusher, Competitor, Perceptive, RoleModel | — |
| Power Rusher | DE | 77/305 | 60/50/40/55/45/20/1/1/40/50/1/1/1/1 | 90/85/75/90/75/60/1/1/80/90/1/1/1/1 | 975 + 250 = 1225 | BullRusher, Competitor, Perceptive, RoleModel | — |
| Run Stuffer | DE | 76/310 | 60/45/40/55/35/20/1/1/40/50/1/1/1/1 | 95/75/75/95/70/60/1/1/80/85/1/1/1/1 | 990 + 250 = 1240 | BullRusher, Competitor, Perceptive, RoleModel | — |
| Interior Rusher | DT | 75/315 | 55/60/40/60/50/20/1/1/55/40/1/1/1/1 | 90/90/80/95/75/50/1/1/80/80/1/1/1/1 | 970 + 300 = 1270 | BullRusher, SpeedRusher, Competitor, Perceptive, RoleModel | — |
| All-Purpose DT | DT | 76/330 | 60/50/40/60/45/20/1/1/55/40/1/1/1/1 | 95/80/80/95/70/50/1/1/80/80/1/1/1/1 | 935 + 300 = 1235 | BullRusher, SpeedRusher, Competitor, Perceptive, RoleModel | — |
| Nose Tackle | DT | 77/340 | 55/40/40/55/35/20/1/1/45/40/1/1/1/1 | 95/70/80/95/65/50/1/1/80/80/1/1/1/1 | 930 + 300 = 1230 | BullRusher, SpeedRusher, Competitor, Perceptive, RoleModel | — |
| Coverage LB | LB | 75/230 | 40/50/55/35/60/35/1/1/50/30/1/1/1/1 | 80/90/95/75/90/75/1/1/80/75/1/1/1/1 | 1050 + 200 = 1250 | Perceptive, Competitor, RoleModel | — |
| Versatile LB | LB | 73/250 | 60/50/50/65/55/25/1/1/50/40/1/1/1/1 | 90/80/85/95/85/50/1/1/80/85/1/1/1/1 | 1030 + 200 = 1230 | BullRusher, Competitor, RoleModel | — |
| Pass Rusher | LB | 72/235 | 55/55/40/45/55/20/1/1/50/40/1/1/1/1 | 85/95/75/85/85/50/1/1/80/90/1/1/1/1 | 1055 + 200 = 1255 | SpeedRusher, Athlete, RoleModel | — |
| Slot Corner | CB | 74/210 | 50/55/55/40/55/35/1/1/40/45/1/1/1/1 | 80/90/90/80/95/75/1/1/80/80/1/1/1/1 | 1080 + 250 = 1330 | SlotCorner, Perceptive, RoleModel, Competitor, Athlete | — |
| Physical Corner | CB | 76/215 | 50/55/50/35/55/40/1/1/40/40/1/1/1/1 | 80/85/90/75/95/80/1/1/80/80/1/1/1/1 | 1045 + 250 = 1295 | PressCorner, Perceptive, RoleModel, Competitor, Athlete | — |
| Cover Corner | CB | 72/200 | 35/55/55/35/70/45/1/1/40/35/1/1/1/1 | 75/85/85/70/100/85/1/1/80/75/1/1/1/1 | 1070 + 250 = 1320 | ShutDownCorner, Perceptive, RoleModel, Competitor, Athlete | — |
| Enforcer | S | 74/220 | 45/50/50/50/55/30/1/1/40/45/1/1/1/1 | 80/85/85/85/90/70/1/1/90/85/1/1/1/1 | 1100 + 200 = 1300 | BoxSafety, RoleModel, Competitor, Athlete | — |
| Ball Hawk | S | 71/210 | 40/55/40/35/60/40/1/1/40/40/1/1/1/1 | 75/90/80/80/95/80/1/1/90/80/1/1/1/1 | 1105 + 200 = 1305 | Perceptive, RoleModel, Competitor, Athlete | — |
| Center Fielder | S | 72/195 | 35/50/40/35/65/35/1/1/40/35/1/1/1/1 | 70/85/80/75/100/75/1/1/90/75/1/1/1/1 | 1095 + 200 = 1295 | CenterFielder, RoleModel, Competitor, Athlete | — |
| Power | K/P | 72/200 | 1/1/60/1/1/1/1/1/1/1/70/60/1/1 | 1/1/90/1/1/1/1/1/1/1/100/100/1/1 | 890 + 350 = 1240 | StraightShooter, LaserSights | — |
| Accurate | K/P | 72/200 | 1/1/60/1/1/1/1/1/1/1/60/70/1/1 | 1/1/90/1/1/1/1/1/1/1/100/100/1/1 | 890 + 350 = 1240 | SteelToeBoots, BionicLeg | — |
| Gunslinger | QB | 75/220 | 40/40/60/1/45/1/1/1/45/45/1/1/65/55 | 70/65/90/1/65/1/1/1/75/80/1/1/100/95 | 1075 + 150 = 1225 | Gunslinger, RoleModel, Athlete | — |
| Pocket Passer | QB | 77/240 | 40/40/60/1/45/1/1/1/45/45/1/1/55/65 | 65/70/95/1/65/1/1/1/75/80/1/1/95/95 | 1025 + 200 = 1225 | GameManager, RoleModel, Athlete | — |
| Field General | QB | 73/210 | 40/35/65/1/45/1/1/1/45/45/1/1/60/55 | 65/60/100/1/65/1/1/1/75/80/1/1/95/90 | 1060 + 200 = 1260 | GameManager, RoleModel, Athlete | — |
| Scrambler | QB | 72/205 | 45/50/50/1/70/35/1/1/50/50/1/1/50/60 | 65/75/90/1/90/60/1/1/80/80/1/1/90/90 | 1015 + 200 = 1215 | RoleModel, Dualthreat, Athlete | — |
| Mobile | QB | 77/245 | 65/40/50/1/65/35/1/1/50/50/1/1/60/50 | 80/65/90/1/85/60/1/1/80/80/1/1/95/85 | 1000 + 200 = 1200 | RoleModel, Dualthreat, Athlete | — |
| Fullback | RB | 75/255 | 60/35/40/1/40/30/40/40/50/40/1/1/1/1 | 90/70/75/1/90/70/80/80/90/85/1/1/1/1 | 1100 + 200 = 1300 | BlockingFB, Athlete, RoleModel | — |
| Power Back | RB | 71/240 | 60/50/35/1/55/40/10/10/60/40/1/1/1/1 | 90/85/80/1/95/70/25/25/95/80/1/1/1/1 | 1095 + 250 = 1345 | ScatBack, RawTalent, Athlete, RoleModel | — |
| Receiving Back | RB | 72/205 | 45/60/55/1/50/50/10/10/50/40/1/1/1/1 | 80/95/80/1/95/80/25/25/90/80/1/1/1/1 | 1095 + 250 = 1345 | ScatBack, RawTalent, Athlete, RoleModel | — |
| Speed Back | RB | 69/190 | 30/50/35/1/60/30/10/10/50/35/1/1/1/1 | 70/90/80/1/100/75/25/25/90/75/1/1/1/1 | 1110 + 225 = 1335 | RawTalent, Athlete, RoleModel | — |

### Portal trait costs and prerequisites

Names below are exact portal identifiers. Trait effects use the mechanics index; portal cap unlocks follow the explicit specialist sequence.

- **Speed Receiver**: DeepThreat (50 TPE; speed>=75, agility>=70, hands>=65); Athlete (50 TPE; speed>=90, agility>=70, endurance>=85); RoleModel (50 TPE; intelligence>=70).
- **Return Specialist**: RoleModel (50 TPE; intelligence>=70).
- **Possession Receiver**: SlotReceiver (50 TPE; speed>=70, agility>=80, hands>=70); Athlete (50 TPE; speed>=90, agility>=70, endurance>=85); RoleModel (50 TPE; intelligence>=70).
- **Slot Receiver**: SlotReceiver (50 TPE; speed>=70, agility>=80, hands>=70); Athlete (50 TPE; speed>=90, agility>=70, endurance>=85); RoleModel (50 TPE; intelligence>=70).
- **Blocking TE**: BlockingTE (100 TPE; strength>=80, passBlocking>=70, runBlocking>=70); RoleModel (50 TPE; intelligence>=70).
- **Possession TE**: ReceivingTE (150 TPE; agility>=80, intelligence>=70, hands>=80); Athlete (automatic; 0 TPE charged; nominal portal menu price 50); RoleModel (50 TPE; intelligence>=70).
- **Vertical Threat**: ReceivingTE (150 TPE; agility>=80, intelligence>=70, hands>=80); DeepThreat (automatic; 0 TPE charged; nominal portal menu price 50); RoleModel (50 TPE; intelligence>=70).
- **Mauler**: TenaciousBlocker (150 TPE; strength>=90, passBlocking>=90, runBlocking>=90); RoleModel (50 TPE; intelligence>=70).
- **Balanced Lineman**: TenaciousBlocker (150 TPE; strength>=90, passBlocking>=90, runBlocking>=90); RoleModel (50 TPE; intelligence>=70).
- **Athletic Lineman**: TenaciousBlocker (150 TPE; strength>=90, passBlocking>=90, runBlocking>=90); RoleModel (50 TPE; intelligence>=70).
- **Speed Rusher**: SpeedRusher (100 TPE; agility>=90, hands>=60, endurance>=80); Competitor (50 TPE; endurance>=75); Perceptive (50 TPE; agility>=60, intelligence>=70); RoleModel (50 TPE; intelligence>=70).
- **Power Rusher**: BullRusher (100 TPE; strength>=90, hands>=60, competitiveness>=80); Competitor (50 TPE; endurance>=75); Perceptive (50 TPE; agility>=60, intelligence>=70); RoleModel (50 TPE; intelligence>=70).
- **Run Stuffer**: BullRusher (100 TPE; strength>=90, hands>=60, competitiveness>=80); Competitor (50 TPE; endurance>=75); Perceptive (50 TPE; agility>=60, intelligence>=70); RoleModel (50 TPE; intelligence>=70).
- **Interior Rusher**: BullRusher (75 TPE; strength>=90, hands>=50, competitiveness>=80); SpeedRusher (75 TPE; speed>=65, agility>=70, endurance>=80); Competitor (50 TPE; endurance>=75); Perceptive (50 TPE; agility>=60, intelligence>=70); RoleModel (50 TPE; intelligence>=70).
- **All-Purpose DT**: BullRusher (75 TPE; strength>=90, hands>=50, competitiveness>=80); SpeedRusher (75 TPE; speed>=65, agility>=70, endurance>=80); Competitor (50 TPE; endurance>=75); Perceptive (50 TPE; agility>=60, intelligence>=70); RoleModel (50 TPE; intelligence>=70).
- **Nose Tackle**: BullRusher (75 TPE; strength>=90, hands>=50, competitiveness>=80); SpeedRusher (75 TPE; speed>=65, agility>=70, endurance>=80); Competitor (50 TPE; endurance>=75); Perceptive (50 TPE; agility>=60, intelligence>=70); RoleModel (50 TPE; intelligence>=70).
- **Coverage LB**: Perceptive (100 TPE; agility>=60, intelligence>=70); Competitor (50 TPE; endurance>=75); RoleModel (50 TPE; intelligence>=70).
- **Versatile LB**: BullRusher (100 TPE; strength>=90, hands>=50, competitiveness>=80); Competitor (50 TPE; endurance>=75); RoleModel (50 TPE; intelligence>=70).
- **Pass Rusher**: SpeedRusher (100 TPE; agility>=90, hands>=50, competitiveness>=80); Athlete (50 TPE; speed>=80, agility>=85, endurance>=80); RoleModel (50 TPE; intelligence>=70).
- **Slot Corner**: SlotCorner (50 TPE; speed>=75, intelligence>=75, tackling>=75); Perceptive (50 TPE; agility>=85, intelligence>=80, hands>=75); RoleModel (50 TPE; intelligence>=70); Competitor (50 TPE; endurance>=75); Athlete (50 TPE; speed>=90, agility>=80, endurance>=75).
- **Physical Corner**: PressCorner (50 TPE; strength>=75, intelligence>=75); Perceptive (50 TPE; agility>=85, intelligence>=80, hands>=75); RoleModel (50 TPE; intelligence>=70); Competitor (50 TPE; endurance>=75); Athlete (50 TPE; speed>=90, agility>=80, endurance>=75).
- **Cover Corner**: ShutDownCorner (50 TPE; agility>=75, intelligence>=75, hands>=75); Perceptive (50 TPE; agility>=85, intelligence>=80, hands>=75); RoleModel (50 TPE; intelligence>=70); Competitor (50 TPE; endurance>=75); Athlete (50 TPE; speed>=90, agility>=80, endurance>=75).
- **Enforcer**: BoxSafety (50 TPE; strength>=75, intelligence>=75, tackling>=75); RoleModel (50 TPE; intelligence>=70); Competitor (50 TPE; endurance>=75); Athlete (50 TPE; speed>=90, agility>=80, endurance>=75).
- **Ball Hawk**: Perceptive (50 TPE; agility>=85, intelligence>=80, hands>=75); RoleModel (50 TPE; intelligence>=70); Competitor (50 TPE; endurance>=75); Athlete (50 TPE; speed>=90, agility>=80, endurance>=75).
- **Center Fielder**: CenterFielder (50 TPE; agility>=85, intelligence>=75, hands>=75); RoleModel (50 TPE; intelligence>=70); Competitor (50 TPE; endurance>=75); Athlete (50 TPE; speed>=90, agility>=80, endurance>=75).
- **Power**: StraightShooter (150 TPE; kickPower>=100, kickAccuracy>=90); LaserSights (200 TPE; kickPower>=100, kickAccuracy>=95).
- **Accurate**: SteelToeBoots (150 TPE; kickPower>=90, kickAccuracy>=100); BionicLeg (200 TPE; kickPower>=95, kickAccuracy>=100).
- **Gunslinger**: Gunslinger (50 TPE; arm>=90); RoleModel (50 TPE; intelligence>=70); Athlete (50 TPE; speed>=65, agility>=60).
- **Pocket Passer**: GameManager (100 TPE; intelligence>=90, throwingAccuracy>=90); RoleModel (50 TPE; intelligence>=70); Athlete (50 TPE; speed>=65, agility>=60).
- **Field General**: GameManager (100 TPE; intelligence>=90, throwingAccuracy>=90); RoleModel (50 TPE; intelligence>=70); Athlete (50 TPE; speed>=65, agility>=60).
- **Scrambler**: RoleModel (50 TPE; intelligence>=70); Dualthreat (100 TPE; intelligence>=85, throwingAccuracy>=85, arm>=85); Athlete (50 TPE; speed>=80, competitiveness>=70, agility>=60).
- **Mobile**: RoleModel (50 TPE; intelligence>=70); Dualthreat (100 TPE; intelligence>=85, throwingAccuracy>=85, arm>=85); Athlete (50 TPE; speed>=80, competitiveness>=70, agility>=60).
- **Fullback**: BlockingFB (50 TPE; strength>=80, passBlocking>=60, runBlocking>=60); Athlete (100 TPE; speed>=90, agility>=70, endurance>=85); RoleModel (50 TPE; intelligence>=70).
- **Power Back**: ScatBack (50 TPE; agility>=80, hands>=70); RawTalent (50 TPE; strength>=70, competitiveness>=75); Athlete (100 TPE; speed>=90, agility>=70, endurance>=85); RoleModel (50 TPE; intelligence>=70).
- **Receiving Back**: ScatBack (50 TPE; agility>=80, hands>=70); RawTalent (50 TPE; strength>=70, competitiveness>=75); Athlete (100 TPE; speed>=90, agility>=70, endurance>=85); RoleModel (50 TPE; intelligence>=70).
- **Speed Back**: RawTalent (75 TPE; strength>=60, competitiveness>=75); Athlete (100 TPE; speed>=90, agility>=70, endurance>=85); RoleModel (50 TPE; intelligence>=70).

### ISFL analytical component leaders

Each comparison holds the named formula context fixed and uses fully maxed ratings. These quantities explain specific strengths; game outcomes combine multiple mechanics.

| Role | Component | Highest value | Archetypes | Context |
|---|---|---:|---|---|
| QB | Arm rating | 100 | Gunslinger | arm |
| QB | Throwing accuracy rating | 95 | Gunslinger, Pocket Passer | throwingAccuracy |
| QB | Intelligence rating | 100 | Field General | intelligence |
| RB | Raw runner contact score | 1359 | Power Back | G:9696; EXP0 Energy100 before context |
| FB | Raw runner contact score | 1259 | Fullback | G:9696; EXP0 Energy100 before context |
| WR | Base movement component | 2.75 | Return Specialist, Speed Receiver | P: base CurrentSpeed; before role/trait/workload modifiers |
| KR | Return speed-plus-agility component | 200 | Return Specialist | G:8612-8618 |
| TE | Base movement component | 2.5999999999999996 | Vertical Threat | P: base CurrentSpeed; before role/trait/workload modifiers |
| C | Pass-block own-rating sum | 520 | Athletic Lineman | P:258; before opponent/experience/scaling/modifiers |
| C | Run-block own-rating sum | 360 | Balanced Lineman, Mauler | P:263; before opponent/experience/scaling/modifiers |
| G | Pass-block own-rating sum | 520 | Athletic Lineman | P:258; before opponent/experience/scaling/modifiers |
| G | Run-block own-rating sum | 360 | Balanced Lineman, Mauler | P:263; before opponent/experience/scaling/modifiers |
| T | Pass-block own-rating sum | 520 | Athletic Lineman | P:258; before opponent/experience/scaling/modifiers |
| T | Run-block own-rating sum | 360 | Balanced Lineman, Mauler | P:263; before opponent/experience/scaling/modifiers |
| DE | Raw defender contact score | 1537 | Run Stuffer | G:9779; EXP0 Energy100 before context |
| DT | Raw defender contact score | 1562 | All-Purpose DT | G:9779; EXP0 Energy100 before context |
| MLB | Raw defender contact score | 1577 | Versatile LB | G:9779; EXP0 Energy100 before context |
| OLB | Raw defender contact score | 1577 | Versatile LB | G:9779; EXP0 Energy100 before context |
| CB | Raw defender contact score | 1465 | Slot Corner | G:9779; EXP0 Energy100 before context |
| FS | Raw defender contact score | 1472 | Enforcer | G:9779; EXP0 Energy100 before context |
| SS | Raw defender contact score | 1472 | Enforcer | G:9779; EXP0 Energy100 before context |
| K/P | Field-goal kick-power-plus-accuracy component | 200 | Accurate, Power | G:9056+; not complete success probability |

## Build WAR: archetype rankings from paired simulations

WAR here = Wins per regular season above a league-average starter at the same depth-chart slot (tie = half a win), from paired simulations: the build and the baseline play the identical matchups and random seeds. A regular season is 16 games (ISFL), 14 games (DSFL).

Simulated native-engine games across the research program: 10,635,240 (attribute matrix: base: 186,840; attribute matrix: expansion v1: 1,868,400; build WAR: phase1-DSFL: 2,400,000; build WAR: phase1-ISFL: 2,400,000; build WAR: phase2-DSFL: 720,000; build WAR: phase2-ISFL: 720,000; build WAR: phase3-ISFL: 720,000; build WAR: phase3b-ISFL: 864,000; build WAR: phase3c-ISFL: 756,000).

Tested depth-chart slots: Tested as the starting quarterback. Tested as the #1 running back. Fullback tested at fullback, against a fullback baseline (not the running-back baseline). Tested as the #1 wide receiver. Return Specialist is tested as the #1 receiver too, not as a returner. Tested as the starting tight end. Offensive linemen tested at center. Defensive ends tested at left end. Tested at the #1 defensive tackle slot. Linebackers tested at middle linebacker. Tested as the #1 cornerback. Safeties tested at free safety. The kicker also punts: these leagues roster no punter.

Tied with #1 means the paired 95% interval for the gap to the group leader includes zero (the leader row shows —). Traits column: purchased portal traits; kicker cap unlocks follow a slash.

| League | Group | Rank | Archetype | WAR | 95% CI | Tied with #1 | TPE | Traits / unlocks |
|---|---|---:|---|---:|---|---|---:|---|
| ISFL | Quarterback | 1 | Gunslinger | +0.11 | [-0.05, +0.27] | — | 1175 | Gunslinger, Athlete |
| ISFL | Quarterback | 2 | Field General | +0.09 | [-0.07, +0.24] | yes | 1210 | GameManager, Athlete |
| ISFL | Quarterback | 3 | Pocket Passer | -0.16 | [-0.31, +0.00] | no | 1175 | GameManager, Athlete |
| ISFL | Quarterback | 4 | Mobile | -0.84 | [-1.00, -0.68] | no | 1150 | Dualthreat, Athlete |
| ISFL | Quarterback | 5 | Scrambler | -1.06 | [-1.22, -0.91] | no | 1165 | Dualthreat, Athlete |
| ISFL | Running Back | 1 | Power Back | +1.20 | [+1.04, +1.36] | — | 1295 | ScatBack, RawTalent, Athlete |
| ISFL | Running Back | 2 | Speed Back | +1.04 | [+0.89, +1.20] | yes | 1285 | RawTalent, Athlete |
| ISFL | Running Back | 3 | Receiving Back | +0.99 | [+0.83, +1.14] | no | 1295 | ScatBack, RawTalent, Athlete |
| ISFL | Running Back | 4 | Fullback | +0.30 | [+0.14, +0.46] | no | 1300 | BlockingFB, Athlete, RoleModel |
| ISFL | Wide Receiver | 1 | Speed Receiver | +0.20 | [+0.04, +0.35] | — | 1160 | Athlete, RoleModel |
| ISFL | Wide Receiver | 2 | Possession Receiver | +0.08 | [-0.08, +0.24] | yes | 1230 | Athlete, RoleModel |
| ISFL | Wide Receiver | 3 | Slot Receiver | +0.08 | [-0.08, +0.24] | yes | 1230 | Athlete, RoleModel |
| ISFL | Wide Receiver | 4 | Return Specialist | +0.05 | [-0.10, +0.21] | yes | 1165 | RoleModel |
| ISFL | Tight End | 1 | Vertical Threat | +0.33 | [+0.18, +0.49] | — | 1210 | ReceivingTE, RoleModel |
| ISFL | Tight End | 2 | Blocking TE | +0.28 | [+0.12, +0.43] | yes | 1220 | BlockingTE, RoleModel |
| ISFL | Tight End | 3 | Possession TE | +0.18 | [+0.02, +0.33] | no | 1220 | ReceivingTE, RoleModel |
| ISFL | Offensive Lineman | 1 | Athletic Lineman | +0.36 | [+0.21, +0.52] | — | 1280 | TenaciousBlocker, RoleModel |
| ISFL | Offensive Lineman | 2 | Balanced Lineman | +0.18 | [+0.02, +0.34] | no | 1230 | TenaciousBlocker, RoleModel |
| ISFL | Offensive Lineman | 3 | Mauler | +0.16 | [-0.00, +0.32] | no | 1245 | TenaciousBlocker, RoleModel |
| ISFL | Defensive End | 1 | Power Rusher | +0.37 | [+0.21, +0.53] | — | 1225 | BullRusher, Competitor, Perceptive, RoleModel |
| ISFL | Defensive End | 2 | Speed Rusher | +0.37 | [+0.21, +0.53] | yes | 1215 | SpeedRusher, Competitor, Perceptive, RoleModel |
| ISFL | Defensive End | 3 | Run Stuffer | +0.35 | [+0.19, +0.51] | yes | 1240 | BullRusher, Competitor, Perceptive, RoleModel |
| ISFL | Defensive Tackle | 1 | All-Purpose DT | +0.39 | [+0.23, +0.54] | — | 1160 | SpeedRusher, Competitor, Perceptive, RoleModel |
| ISFL | Defensive Tackle | 2 | Nose Tackle | +0.38 | [+0.22, +0.54] | yes | 1155 | SpeedRusher, Competitor, Perceptive, RoleModel |
| ISFL | Defensive Tackle | 3 | Interior Rusher | +0.30 | [+0.15, +0.46] | yes | 1195 | SpeedRusher, Competitor, Perceptive, RoleModel |
| ISFL | Linebacker | 1 | Versatile LB | +0.24 | [+0.08, +0.39] | — | 1230 | BullRusher, Competitor, RoleModel |
| ISFL | Linebacker | 2 | Coverage LB | +0.23 | [+0.07, +0.38] | yes | 1150 | Competitor, RoleModel |
| ISFL | Linebacker | 3 | Pass Rusher | +0.01 | [-0.14, +0.17] | no | 1155 | Athlete, RoleModel |
| ISFL | Cornerback | 1 | Cover Corner | +0.30 | [+0.15, +0.46] | — | 1220 | ShutDownCorner, Perceptive, RoleModel |
| ISFL | Cornerback | 2 | Slot Corner | +0.05 | [-0.10, +0.21] | no | 1230 | SlotCorner, Perceptive, RoleModel |
| ISFL | Cornerback | 3 | Physical Corner | +0.04 | [-0.12, +0.19] | no | 1195 | PressCorner, Perceptive, RoleModel |
| ISFL | Safety | 1 | Ball Hawk | +0.09 | [-0.06, +0.24] | — | 1255 | RoleModel, Competitor, Athlete |
| ISFL | Safety | 2 | Center Fielder | +0.06 | [-0.09, +0.22] | yes | 1295 | CenterFielder, RoleModel, Competitor, Athlete |
| ISFL | Safety | 3 | Enforcer | -0.04 | [-0.20, +0.11] | yes | 1300 | BoxSafety, RoleModel, Competitor, Athlete |
| ISFL | Kicker | 1 | Power | +0.36 | [+0.21, +0.52] | — | 1240 | — / StraightShooter, LaserSights |
| ISFL | Kicker | 2 | Accurate | +0.36 | [+0.21, +0.52] | yes | 1240 | — / SteelToeBoots, BionicLeg |
| DSFL | Quarterback | 1 | Field General | +0.32 | [+0.18, +0.47] | — | 249 | — |
| DSFL | Quarterback | 2 | Pocket Passer | +0.30 | [+0.16, +0.45] | yes | 249 | — |
| DSFL | Quarterback | 3 | Gunslinger | +0.30 | [+0.15, +0.45] | yes | 249 | — |
| DSFL | Quarterback | 4 | Scrambler | +0.06 | [-0.09, +0.21] | no | 249 | — |
| DSFL | Quarterback | 5 | Mobile | +0.04 | [-0.11, +0.18] | no | 249 | — |
| DSFL | Running Back | 1 | Power Back | +0.41 | [+0.26, +0.55] | — | 250 | — |
| DSFL | Running Back | 2 | Fullback | +0.26 | [+0.11, +0.40] | yes | 250 | — |
| DSFL | Running Back | 3 | Receiving Back | +0.25 | [+0.11, +0.40] | no | 250 | — |
| DSFL | Running Back | 4 | Speed Back | -0.04 | [-0.18, +0.11] | no | 250 | — |
| DSFL | Wide Receiver | 1 | Possession Receiver | +0.37 | [+0.22, +0.51] | — | 249 | — |
| DSFL | Wide Receiver | 2 | Slot Receiver | +0.35 | [+0.20, +0.49] | yes | 249 | — |
| DSFL | Wide Receiver | 3 | Speed Receiver | -0.05 | [-0.20, +0.09] | no | 249 | — |
| DSFL | Wide Receiver | 4 | Return Specialist | -0.07 | [-0.21, +0.08] | no | 250 | — |
| DSFL | Tight End | 1 | Possession TE | +0.51 | [+0.37, +0.66] | — | 250 | — |
| DSFL | Tight End | 2 | Vertical Threat | +0.45 | [+0.31, +0.60] | yes | 250 | — |
| DSFL | Tight End | 3 | Blocking TE | +0.43 | [+0.28, +0.57] | yes | 250 | — |
| DSFL | Offensive Lineman | 1 | Athletic Lineman | +0.21 | [+0.07, +0.36] | — | 249 | — |
| DSFL | Offensive Lineman | 2 | Balanced Lineman | +0.20 | [+0.06, +0.35] | yes | 249 | — |
| DSFL | Offensive Lineman | 3 | Mauler | +0.12 | [-0.03, +0.26] | yes | 249 | — |
| DSFL | Defensive End | 1 | Speed Rusher | +0.66 | [+0.52, +0.81] | — | 249 | — |
| DSFL | Defensive End | 2 | Run Stuffer | +0.62 | [+0.47, +0.76] | yes | 249 | — |
| DSFL | Defensive End | 3 | Power Rusher | +0.58 | [+0.44, +0.73] | yes | 249 | — |
| DSFL | Defensive Tackle | 1 | All-Purpose DT | +0.35 | [+0.21, +0.49] | — | 250 | — |
| DSFL | Defensive Tackle | 2 | Nose Tackle | +0.29 | [+0.15, +0.43] | yes | 250 | — |
| DSFL | Defensive Tackle | 3 | Interior Rusher | +0.20 | [+0.07, +0.34] | no | 250 | — |
| DSFL | Linebacker | 1 | Coverage LB | +0.55 | [+0.40, +0.69] | — | 250 | — |
| DSFL | Linebacker | 2 | Versatile LB | +0.54 | [+0.39, +0.68] | yes | 250 | — |
| DSFL | Linebacker | 3 | Pass Rusher | +0.45 | [+0.30, +0.59] | yes | 250 | — |
| DSFL | Cornerback | 1 | Cover Corner | +1.21 | [+1.06, +1.35] | — | 250 | — |
| DSFL | Cornerback | 2 | Slot Corner | +1.19 | [+1.04, +1.33] | yes | 250 | — |
| DSFL | Cornerback | 3 | Physical Corner | +1.14 | [+0.99, +1.28] | yes | 250 | — |
| DSFL | Safety | 1 | Ball Hawk | +0.34 | [+0.19, +0.48] | — | 250 | — |
| DSFL | Safety | 2 | Enforcer | +0.33 | [+0.19, +0.48] | yes | 250 | — |
| DSFL | Safety | 3 | Center Fielder | +0.21 | [+0.06, +0.36] | yes | 250 | — |
| DSFL | Kicker | 1 | Accurate | +0.20 | [+0.14, +0.26] | — | 250 | — |
| DSFL | Kicker | 2 | Power | +0.14 | [+0.00, +0.29] | yes | 250 | — |

ISFL rankings use fully maxed builds (every rating at its effective cap, every kicker unlock, and each purchasable trait with a positive fitted effect). DSFL rankings use the model-optimal legal 250-TPE build with no purchased traits. `research-pack.json` → `build_war` holds every build, the per-slot fitted win models (exact terms and covariance) and the optimizer definition used by the website builder.

Model calibration (ISFL): every headline build was also measured directly in paired games. Raw model predictions agreed within their 95% range for 34 of 36 builds (chi2 51.05/36 df, p = 0.050). After anchoring each depth-chart slot's level to its direct measurements, the remaining disagreement is chi2 15.44/24 df (p = 0.907). Builder WAR = model WAR + slot anchor: C1 +0.20 ± 0.11, CB1 -0.22 ± 0.11, DT1 +0.19 ± 0.11, FB1 +0.31 ± 0.19, FS1 -0.22 ± 0.11, K1 -0.20 ± 0.12, LE1 +0.03 ± 0.11, MLB1 +0.20 ± 0.11, QB1 -0.09 ± 0.08, RB1 +0.34 ± 0.10, TE1 -0.02 ± 0.11, WR1 +0.05 ± 0.10.

Model calibration (DSFL): every headline build was also measured directly in paired games. Raw model predictions agreed within their 95% range for 31 of 36 builds (chi2 79.05/36 df, p < 0.001). After anchoring each depth-chart slot's level to its direct measurements, the remaining disagreement is chi2 8.45/24 df (p = 0.999). Builder WAR = model WAR + slot anchor: C1 -0.17 ± 0.08, CB1 +0.02 ± 0.08, DT1 -0.05 ± 0.08, FB1 -0.24 ± 0.14, FS1 -0.16 ± 0.08, K1 +0.07 ± 0.07, LE1 -0.17 ± 0.08, MLB1 -0.11 ± 0.08, QB1 -0.46 ± 0.07, RB1 +0.02 ± 0.08, TE1 -0.16 ± 0.09, WR1 -0.06 ± 0.07.

Builder validation (ISFL, builder_validation): 108 builds at 500, 1000, 1700 TPE, 7,000 paired games per slot: predictions within their 95% range for 108 of 108 (chi2 59.0/108, p = 1.000); mean predicted minus measured +0.064 WAR; at 1700 TPE the Builder build beat the fully maxed build for 4 of 36 archetypes and was worse for 0.

Builder validation (ISFL, builder_validation_v1): 72 builds at 800, 1700 TPE, 10,000 paired games per slot: predictions within their 95% range for 43 of 72 (chi2 301.81/72, p < 0.001); mean predicted minus measured +0.365 WAR; at 1700 TPE the Builder build beat the fully maxed build for 3 of 36 archetypes and was worse for 8. Status: rejected: the unconstrained Builder model's cheaper-than-maxed recommendations did not hold up in direct paired tests; replaced by the monotone-constrained Builder model.

Builder validation (ISFL, builder_validation_v2): 108 builds at 500, 1000, 1700 TPE, 8,000 paired games per slot: predictions within their 95% range for 96 of 108 (chi2 191.07/108, p < 0.001); mean predicted minus measured +0.061 WAR; at 1700 TPE the Builder build beat the fully maxed build for 2 of 36 archetypes and was worse for 4. Status: intermediate: monotone model without overrides; its direct results showed QB speed harm is real and the model's RB hands/endurance harm is not, which set the final model's two overrides (the final model is then validated on fresh, unused pairs).

## Build selector

Legacy recipe engine. The website builder now uses the Build WAR models above for ISFL, and the measured Build WAR builds for DSFL at 250 TPE; this engine remains only for DSFL budgets below 250.


The component accepts league, game role, archetype and available TPE. Its visual design is deferred. `recommendation_engine.py` implements the calculation layer without a web framework or external dependencies.

### Inputs

- `league`: DSFL or ISFL. DSFL accepts 0–250 TPE and no purchased traits.
- `role`: explicit game role. OL expands to C/G/T, LB to MLB/OLB, and S to FS/SS. Preserve KR and the combined K/P test role.
- `archetype`: filter using `engine-data.json → best_screening_by_role_archetype`; each key is `role|archetype`.
- `budget`: nonnegative integer TPE, measured above free archetype starting ratings. Trait fees share this budget.
- `mode`: `research` (default), `mechanic`, or `maxed`.
- `objective`: required for mechanic mode; one of the seven named formula objectives.
- `selected_traits`: optional list of portal trait names for ISFL. The engine reserves their costs and prerequisite ratings, includes prerequisite unlock traits, and checks legality.
- `profile`: optional frozen candidate ID for research mode. Omit to select the highest screening-margin profile within the chosen archetype and role.

Start with numeric TPE entry. A slider can use the same integer input later. Changing role resets an incompatible archetype. Retain the user's budget when changing archetype; show an inline error when selected traits cannot be afforded.

### Recommendation modes

| Mode | Calculation | Display basis |
|---|---|---|
| research, DSFL 250 | Retrieves the exact frozen tested allocation | Tested DSFL sample |
| research, other budgets or ISFL | Replays ordered research goals, then cyclic remainder priorities within recipe ceilings | Research-guided allocation |
| mechanic | Exact integer-budget optimization of one local formula, respecting caps and selected-trait prerequisites | Exact formula optimum |
| maxed, ISFL | All effective attribute caps and every purchasable portal trait, including kicker cap unlocks | Fully maxed template |

Research mode does not automatically purchase ISFL traits. Use selected traits or the explicit fully maxed view. Recipe ceilings can leave TPE unused; report the remainder. A changed budget, trait set or league does not inherit the observed score of a DSFL sample. Formula mode maximizes its stated subtotal; the simulator's subsequent rolls and contextual modifiers remain separate.

### Output and presentation

Return all 14 ratings, per-attribute TPE, paid and automatic traits, total spent, remainder, dimensions, basis, and mode-specific explanation. `predicted_game_margin` is null. Display current rating beside starting rating and effective cap. Show selected-trait prerequisites and unlock order in an expandable cost breakdown.

Link explanation sections to stable mechanics fact IDs: movement → `speed.base`; contact → `contact.runner` / `contact.defender`; pass/run blocking → `blocking.pass` / `blocking.run`; kicking → the kicking facts in the mechanics index. Preserve source references with the explanations.

The comparison view is role-specific. Show each archetype's highest screening candidate and the separately confirmed finalist means. Label margins as treated-team points for minus points against; these are sample outcomes, not bonuses caused solely by the player. Show the paired comparison interval with its A/B candidate labels. Use `simultaneous_supported_winner_candidate_id` for a statistically supported winner badge; it is currently null for all 17 roles.

### Callable interface

```python
from recommendation_engine import recommend
result = recommend('DSFL', 'WR', 'Speed Receiver', 250)
result = recommend('DSFL', 'K/P', 'Accurate', 250,
                   mode='mechanic', objective='field_goal_rating_sum')
result = recommend('ISFL', 'K/P', 'Accurate', 1240, mode='maxed')
```

Serve this function from a future backend or port it with the frozen test fixtures. Convert `ValueError` into a field-level validation message. `component-examples.json` supplies complete request/response fixtures, including an invalid request. Ship `recommendation_engine.py` beside `engine-data.json` for standalone execution.

### Evidence needed for game-performance optimization

For new budgets and fully maxed ISFL builds, generate legal candidates with the engine, compare equal-budget alternatives in original-engine games, and reserve fresh seeds and matchups for confirmation. Record host roster, gameplans, skills, experience, automatic-trait export, dimensions, simulator hash and lookup-table state. Choose a declared role objective before selection; report paired uncertainty and multiple-comparison adjustment. Store each tested allocation as a new evidence record rather than replacing the existing measurements.

### Worked requests

- Request: `{"league": "DSFL", "role": "WR", "archetype": "Speed Receiver", "budget": 250}`. Result: 30/71/60/1/76/81/5/5/71/30/1/1/1/1; 250 spent, 0 remaining; basis `tested_dsfl_250`.
- Request: `{"league": "DSFL", "role": "WR", "archetype": "Speed Receiver", "budget": 100}`. Result: 30/55/40/1/70/80/5/5/50/30/1/1/1/1; 100 spent, 0 remaining; basis `research_guided_budget_projection`.
- Request: `{"league": "DSFL", "role": "K/P", "archetype": "Accurate", "budget": 250, "mode": "mechanic", "objective": "field_goal_rating_sum"}`. Result: 1/1/60/1/1/1/1/1/1/1/83/90/1/1; 250 spent, 0 remaining; basis `exact_local_objective`.
- Request: `{"league": "ISFL", "role": "K/P", "archetype": "Accurate", "budget": 1240, "mode": "maxed"}`. Result: 1/1/90/1/1/1/1/1/1/1/100/100/1/1; 1240 spent, 0 remaining; basis `legal_maxed_template`.
- Request: `{"league": "ISFL", "role": "WR", "archetype": "Speed Receiver", "budget": 500, "selected_traits": ["DeepThreat"]}`. Result: 30/78/68/1/84/85/5/5/79/39/1/1/1/1; 500 spent, 0 remaining; basis `research_guided_budget_projection`.
- Request: `{"league": "DSFL", "role": "WR", "archetype": "Speed Receiver", "budget": 251}`. Validation: DSFL maximum is 250 TPE.

## Source paths

Paths are relative to the original game workspace and require the original decompilation. The prior evidence bundle contains reviewed chapters and game evidence; it excludes complete decompiled source.

- S1: `Headless/_reference/Player.cs`
- S2: `Headless/research/attribute-audit/source-full/Draft Day Sports - Pro Football 2021/DDSPF/Functions.cs`
- S3: `Headless/research/attribute-audit/source-full/Draft Day Sports - Pro Football 2021/DDSPF/League.cs`
- S4: `Headless/_reference/GameFunctions.cs`
- S5: `Headless/research/attribute-audit/traits-fatigue.md`
- S6: `Headless/_reference/Enums.cs`

## File map

- `research-pack.json`: normalized catalog, mechanics, examples, comparisons and provenance.
- `engine-data.json`: portable catalog, all 137 recipe inputs and selection lookup.
- `recommendation_engine.py`: runnable budget allocator and exact local-formula optimizer.
- `component-spec.md`, `component-examples.json`: UI integration contract and request/response fixtures.
- `mechanics.json`, `build-examples.json`: detailed source records and full measured metrics.
- `build-war-results.json`: snapshot of the build-WAR export (paired confirmations, fitted per-slot win models, simulation counts).
- `*-qa.json`, `test_recommendation_engine.py`, `verify_build_examples.py`: independent checks and evidence.

# EVAtAR-IDEA — Detailed Presentation Speech Scripts
**Jahidul Arafat · Auburn University CSSE · PhD Research Proposal**
**Supervisor: Prof. Gerry Dozier · Domain: IDEC**

> These scripts are written for verbal delivery to a PhD committee.
> Each slide script is detailed, example-rich, and covers every component visible on the slide.
> Estimated total delivery time: **8 minutes 15 seconds** (excluding Q&A pauses).

---

## SLIDE 1 — Project Title
**Estimated time: 45 seconds**

> *[Stand. Pause 2 seconds. Make eye contact before speaking.]*

Good morning, members of the committee. My name is Jahidul Arafat — I am a PhD candidate in the Department of Computer Science and Software Engineering here at Auburn University, and I am advised by Professor Gerry Dozier.

The research I am presenting today is called **EVAtAR-IDEA**: the Interactive Distributed Evolutionary Algorithm for Optimising Expression Tag Sequences in Scripted Avatar Video Generation.

This work sits at the intersection of three fields: **evolutionary computation**, **affective computing**, and **AI-driven media production**. The central question is this — when an AI avatar like Aubria is delivering a keynote speech — to an Auburn BoT meeting, a DCSITE briefing, or an AI Ethics Iron Bowl audience — can a systematic algorithm automatically decide which expression tags to place at which words, for how long, in order to maximise the perceptual quality of the avatar's facial expression as measured by DeepFace and the AffectNet taxonomy?

The research is structured as three sequential peer-reviewed papers, and it tests the algorithm across three avatar platforms: **HeyGen**, **D-ID**, and **Akool**. The ground truth for all scoring comes from **AffectNet** — the largest publicly available facial expression database in the wild, published by Mollahosseini, Hasani, and Mahoor in the IEEE Transactions on Affective Computing in 2019, with over one million images and 450,000 manually annotated facial expressions across eight emotion classes.

---

## SLIDE 2 — The Problem
**Estimated time: 90 seconds**

> *[Speak slowly here. The committee needs to feel the problem before they hear the solution. Use first person throughout. Pause after each statistic.]*

Let me ground this research in a concrete, first-hand experience.

During February to May 2026, I was producing scripted speeches using **Aubria** — HeyGen's AI avatar — as part of the **AI@AU/Team Since Series initiative** at Auburn University. Aubria was the face of this initiative, delivering keynote speeches at **Auburn Board of Trustees meetings**, programme briefings for the **DCSITE** detection canine science initiative, and the **AI Ethics Iron Bowl** event, where Aubria addressed an audience of students, faculty, and industry professionals live. A typical speech script is 500 to 800 words long — not a single sentence, but a full, structured address. Each word is a potential tag placement point. Tags like `[serious]`, `[excited]`, `[pause: 2s]` — placed manually, word by word, clip by clip.

The problem I encountered was that every attempt broke something different. In one iteration, the avatar's face would **freeze mid-sentence** — a classic F1 "no emotion tag" failure — the "no emotion tag" failure, where no expression tag was present at all, where no emotion tag was present for the first half of the clip. In another, the expression would **peak too late** — I placed `[excited]` at word 14 of a 16-word sentence, but the avatar's rendering engine needs to fire the expression *before* the corresponding audio peak, not simultaneously with it. That is the F3 — the late-onset failure mode, where the expression fires after the audio peak.

In the worst cases, I placed two emotion tags within three words of each other — for example `[serious]` at word 4 and `[excited]` at word 6. The avatar engine cannot transition between two emotion states that close together; both degrade into a neutral blank expression. That is **F2 — tag stacking, where two emotion tags are placed too close together** — one of the eight failure modes I later formalised.

The result, in aggregate: **more than 200 hours** of manual iteration, across **more than 100 re-renders**, to produce a single polished four-to-ten-minute video. The token cost — 12 HeyGen credits per clip, at $0.004 per credit — works out to approximately **$0.048 per render**. For a 500-word script, that is 35 clips. At 100 re-renders per clip, the token cost exceeds **$166 per video**, purely in API calls.

Now, HeyGen does offer a built-in solution: **Auto-Enhance**, a button that places expression tags automatically. I want to be precise about why that is insufficient as a research baseline. Auto-Enhance typically places tags at approximately 30%, 65%, and 80% of sentence length. The 65% and 80% positions almost always trigger the F3 — the late-onset failure, where the expression fires after the audio peak — the expression fires after the audio has already peaked. There is no perceptual fitness function behind Auto-Enhance. There is no AffectNet validation. There is no failure taxonomy. There is no published methodology — no pre-registered hypothesis, no statistical guarantee.

For example: given the sentence *"In twelve months, three papers, one hundred and sixty-six dollars, two hundred hours — and it is finally done,"* Auto-Enhance places `[enthusiastic]` at word 3 and again at word 13 — which is position 68% of 19 words. F3 late onset — meaning the expression fires after the audio peak — meaning the expression fires after the audio peak is triggered because the emotional peak of that sentence arrives at word 16: *"finally done."* The avatar expresses enthusiasm before it arrives at the emotional content.

**EVAtAR-IDEA addresses this systematically.** It is not a rule-based enhancement — it is a rigorous evolutionary search with a pre-registered fitness function, validated against AffectNet, with documented failure modes and statistical reproducibility guarantees.

---

## SLIDE 3 — Research Objective
**Estimated time: 75 seconds**

> *[Technical but accessible. Write the formula on the board if available. Emphasise "pre-registered" — this is a methodological strength.]*

The core insight behind EVAtAR-IDEA is that manual expression tag placement is, at its heart, a **combinatorial optimisation problem**.

Given a sentence, a vocabulary of 21 confirmed expression tokens — divided into three tiers: six AffectNet-confirmed emotion tags, six excluded tokens with documented reasons, and nine gesture and delivery tokens — and given the possible word positions and the set of hold durations from 0.5 to 3 seconds, the search space exceeds **one thousand distinct combinations per sentence**. No human can systematically explore that space. The question is whether an evolutionary algorithm can do it better.

I formalise the optimisation target as:

**F(p) = 0.5 · f1 + 0.3 · f2 + 0.2 · f3**

Where:
- **f1** is AffectNet class accuracy — the mean probability across all video frames that DeepFace detects the intended emotion class. For a sentence tagged as `[excited]`, f1 is the mean P(Happy) returned by DeepFace, because `[excited]` maps to AffectNet's Happy class.
- **f2** is timing alignment — does the peak of the detected emotion coincide with the peak position in the audio? A tag placed at word 2 of a 12-word sentence produces a strong f1 but may have weak f2 if the sentence's emotional peak is at word 9.
- **f3** is expression consistency — is the detected emotion stable across frames, or does it flicker below the detection threshold mid-sentence?

The weights — 50%, 30%, 20% — reflect a priority ordering that I pre-registered at OSF before any data collection. Getting the correct emotion class is the primary failure mode. Timing is secondary. Consistency is tertiary. These weights are themselves a research decision, and Paper 2 includes a sensitivity analysis.

Three baselines are compared: **p_auto** from HeyGen's Auto-Enhance — no fitness function, no validation; **p_manual** from expert human placement — my own best effort after years of working with Aubria; and **p-star** from EVAtAR-IDEA. The pre-registered success threshold is F(p*) greater than or equal to **0.75**, confirmed by Wilcoxon signed-rank test with p less than 0.05 and Cohen's d of at least 0.5.

---

## SLIDE 4 — Three Prototypes · Three Papers
**Estimated time: 105 seconds**

> *[Pause between prototypes. Point to the P1, P2, P3 cards on screen. Emphasise "sequential" — P2 cannot begin without P1. P3 cannot begin without P2.]*

EVAtAR-IDEA is built across three sequential prototypes, each targeting a peer-reviewed paper.

**Prototype 1 — Baseline Measurement.**
I collect 50 sentences across the 8 AffectNet emotion classes: Neutral, Happy, Sad, Surprise, Fear, Anger, Disgust, and Contempt. Each sentence is assigned a difficulty level — Easy, Medium, or Hard — using a three-factor rubric: Flesch-Kincaid grade for syntactic complexity, Russell and Mehrabian's Valence-Arousal-Dominance model for emotional ambiguity, and the F3 late-onset — meaning the expression fires after the audio peak — meaning the expression fires after the audio has already peaked risk computed from the position of the primary emotion cue word. For example, the sentence *"Today we will explore calculus"* is Easy — 9 words, Flesch-Kincaid grade 8, emotion cue at word 4 which is 44% of sentence length, low late-onset risk. The sentence *"In twelve months, three papers, two hundred hours — and it is finally done"* is Hard — 19 words, grade 11, emotion cue at word 16 which is 84% of sentence length, very high F3 late-onset late-onset late-onset late-onset late-onset — meaning the expression fires after the audio peak risk — how likely the expression is to fire after the audio peak.

I must be transparent about one vocabulary decision: the HeyGen tag `[curious]` has no direct AffectNet equivalent. AffectNet's eight classes are exactly Neutral, Happy, Sad, Surprise, Fear, Anger, Disgust, and Contempt — as confirmed in Table 3 of Mollahosseini et al. (2019). I map `[curious]` to the Fear class as the closest proxy in the Russell circumplex — curiosity shares high arousal and low-to-negative dominance with Fear. This is a **researcher inference**, not an AffectNet definition, and it is pre-registered as a threat to validity in Paper 1. The GA compensates by pairing `[curious]` with `[head:tilt]` and `[eyebrow:raise]` gesture tokens.

The output of P1 is the **D1 dataset** — 50 sentences with F(p_manual), F(p_auto), and documented F1 through F8, our eight documented avatar expression failure modes — the sub-perceptual failure where a tag produces too weak a signal for DeepFace to detect reliably — our eight documented avatar expression failure modes failure modes for each. This targets Paper 1: the Vocabulary and Corpus Study.

**Prototype 2 — EVAtAR-IDEA Engine.**
For each of the 50 sentences, EVAtAR-IDEA initialises 8 candidate tag sequences — randomly drawn from the 21-token vocabulary — and evolves them across 8 generations. Each generation involves rendering 8 clips via HeyGen and scoring each via DeepFace. That is 64 API calls per sentence per run — in the real system, approximately 64 minutes of render time and approximately $3 in token costs per sentence. Tournament selection with k equals 3 preserves diversity while applying selection pressure. Mutation operators — shift position, change tag, add tag, remove tag, change duration — explore the search space. Constraint repair fixes any F2 — tag stacking, where two emotion tags are placed too close together or F3 late-onset — meaning the expression fires after the audio peak — meaning the expression fires after the audio has already peaked violations post-mutation. Sequences that achieve F(p*) above 0.70 are stored in the **Meme Space** and reused as seeds for subsequent sentences of the same emotion class, reducing render cost from ~64 to ~20 per sentence. This targets Paper 2: the Optimisation Study.

**Prototype 3 — Cross-Platform Portability.**
The optimised p-star sequences from P2 are tested on three platforms using the identical DeepFace scoring pipeline. HeyGen with inline tag support scores highest. D-ID, which translates tags into natural-language motion hints, is expected to degrade by 15 to 25 percent. Akool, which is audio-driven with no expression control, is the lower bound. This answers whether EVAtAR-IDEA is a HeyGen-specific tool or a generalisable optimisation framework. This targets Paper 3: the Portability Study.

---

## SLIDE 5 — What You Are About to Experience
**Estimated time: 90 seconds**

> *[Conversational tone. You are a guide now. Invite the committee to follow along. Make it feel like an experience, not a demo.]*

Before you interact with the playground, let me map what each stage demonstrates and which paper it corresponds to.

**Stage 1 — Select Script:** This is P1 corpus collection in miniature. You choose from 32 sentences, 4 per emotion category, across all 8 AffectNet classes. When you select a category, three things appear: a rationale panel explaining why there are exactly 8 categories and how difficulty was assigned, a paragraph-to-clips explainer showing how a real 500-word Auburn Canine Science script breaks into individual render units, and a live Auto-Enhance preview showing which tags HeyGen would place and which failure modes they would trigger — for example, `[thoughtful]` at word 4 and again at word 9 on a 13-word sentence, flagged as F2 tag-stacking near-stacking, where two tags are placed dangerously close together — two tags placed dangerously close together.

**Stage 2 — Build Tags Manually:** This is P1 baseline measurement. You click any word to add an expression tag. The F1 through F8, our eight documented avatar expression failure modes — the sub-perceptual failure where a tag produces too weak a signal for DeepFace to detect reliably — our eight documented avatar expression failure modes failure taxonomy panel on the right updates in real time. If you place a tag at position 9 of a 12-word sentence, F3 late onset — meaning the expression fires after the audio peak — meaning the expression fires after the audio peak fires immediately — you see it happen. Your final F(p_manual) is computed and stored as the baseline the GA must beat.

**Stage 3 — Run GA:** This is P2. You choose 1 to 5 independent runs. Each run evolves 8 generations. You watch the convergence chart with your F(p_manual) as the red dashed baseline. The Meme Space fills in real time. The evolution trail shows every mutation applied — shift position, change tag, add tag — and the best candidate sequence of each generation.

**Stage 4 — Results:** This is P2 and P3 combined. You see the three-way comparison table: Auto-Enhance in amber, your manual sequence in red, GA optimised in green. The pre-registered threshold check tells you whether F(p*) reached 0.75. The cross-platform projection shows expected degradation on D-ID and Akool. You can export the full session as a D1-format CSV — the exact data structure used in real P1 data collection.

---

## SLIDE 6 — Full Pipeline, Pseudocode, and Algorithm
**Estimated time: 90 seconds**

> *[Use the three tab buttons. Pipeline SVG first — 30 seconds. Pseudocode second — 30 seconds. Plain English third — 30 seconds. Never read pseudocode line by line; summarise the structure.]*

This slide presents the complete EVAtAR-IDEA algorithm from three angles.

**Pipeline View — left to right:**
P1 is on the left in orange. It takes the corpus, runs both Auto-Enhance and human labelling, renders both via HeyGen, scores both via DeepFace, and produces the D1 dataset with F1 through F8, our eight documented avatar expression failure modes — the sub-perceptual failure where a tag produces too weak a signal for DeepFace to detect reliably — our eight documented avatar expression failure modes flags. The baseline arrow carries F(p_manual) and F(p_auto) rightward into P2. P2, in the centre green box, contains the inner GA loop — the dashed box labelled "64 renders per sentence." Each of 8 candidates per generation is rendered, scored, selected, mutated, and repaired. Sequences above 0.70 enter the Meme Space at the right of the loop. After 8 generations, p-star moves right into P3, where the same sequence is tested on HeyGen, D-ID, and Akool. The pre-registered threshold dashed line at the bottom — F(p*) ≥ 0.75, Wilcoxon p < 0.05, Cohen d ≥ 0.5 — is the primary success criterion for Paper 2.

**Pseudocode View:**
The algorithm opens with `PRE-REGISTER at OSF` — this happens before any data collection. P1 runs `AutoEnhance` and `Human.LabelTags` in parallel on each sentence, rendering and scoring both. P2's inner loop is: `for each candidate p in pop → render → score → store if ≥ 0.70 → tournament → mutate → constraint repair`. The final line is `COMPARE F(p*) vs F(p_auto) vs F(p_manual)` — a three-way Wilcoxon signed-rank test. P3 applies the same p-star to three platforms and reports degradation.

**Plain English View:**
P1 answers: where does human expertise break down, and where does Auto-Enhance fail systematically? P2 answers: can an evolutionary algorithm beat both, and by a statistically meaningful margin? P3 answers: does that improvement transfer across platforms, or is it locked to HeyGen's architecture?

This is the first application of Dozier's 2005 IDEA framework to the domain of affective avatar expression optimisation — and the first work to formally compare evolutionary tag optimisation against a commercially deployed heuristic baseline.

---

## TIMING SUMMARY

| Slide | Title | Time |
|---|---|---|
| 1 | Project Title | 45 sec |
| 2 | The Problem | 90 sec |
| 3 | Research Objective | 75 sec |
| 4 | Three Prototypes | 105 sec |
| 5 | What You Will Experience | 90 sec |
| 6 | Full Pipeline + Algorithm | 90 sec |
| **Total** | | **8 min 15 sec** |

---

## ANTICIPATED COMMITTEE QUESTIONS — COMPLETE

> These 26 questions cover every angle a PhD committee is likely to probe.
> Answers are written for verbal delivery — direct, confident, technically precise.

---

### SECTION A — Research Design & Methodology

**Q1: Why exactly 8 categories — why not 10 or 12?**
Because AffectNet has exactly 8 classes in its categorical model — Neutral, Happy, Sad, Surprise, Fear, Anger, Disgust, Contempt (Table 3, Mollahosseini et al. 2019). DeepFace uses these exact 8 classes as its classification backbone. If I added a 9th category with no corresponding AffectNet class, f1 would be undefined for that category — the scoring oracle would break. The 8 categories are not a design choice; they are determined by the measurement infrastructure.

**Q2: Why are the F(p) weights 0.5, 0.3, 0.2 and not equal thirds?**
These weights were pre-registered before data collection. f1 at 50%: a wrong emotion class is a categorical failure regardless of timing. f2 at 30%: timing determines whether expression and audio feel synchronised — the most perceptually salient issue after class accuracy. f3 at 20%: flickering is visible but less disruptive. Paper 2 includes a sensitivity analysis varying each weight by ±0.1 to confirm the threshold finding is robust.

**Q3: Why not use a data-driven learned reward model instead of the hand-designed F(p) formula?**
Two reasons. First, a data-driven reward model would require a large labelled dataset of human perceptual ratings of avatar expressions — that dataset does not exist yet; building it is a future research direction. Second, the hand-designed formula with pre-registered weights is methodologically stronger for a PhD proposal: it makes the success criterion falsifiable before data collection, prevents post-hoc weight tuning, and produces an interpretable decomposition of the score. A learned reward model would be a black box that the committee could not interrogate. The formula also allows direct comparison across emotion classes, which a learned model would require normalisation to achieve.

**Q4: You map [curious] to Fear — how do you defend that?**
`[curious]` is not in AffectNet. I chose Fear as the closest proxy using Russell's Valence-Arousal-Dominance circumplex: both curiosity and fear share high arousal and low-to-negative dominance, placing them in the same quadrant. This is a researcher inference, pre-registered as a threat to validity in Paper 1. Empirically, the GA compensates by pairing `[curious]` with `[head:tilt]` and `[eyebrow:raise]` gesture tokens. The pre-registration means I cannot retroactively redefine the mapping if it underperforms — the threat to validity is documented and the committee can evaluate the finding accordingly.

**Q5: Is there circularity — you defined the F1–F8 failure modes from the same observation that motivates the research?**
This is a fair methodological concern, and I address it directly. The F1–F8 taxonomy was derived from exploratory production work (February–May 2026) and then treated as a fixed, pre-registered framework before P1 data collection. It is used as a constraint specification for the GA's constraint repair module and as a diagnostic tool in P1 — it is not used to compute F(p). The fitness function F(p) = 0.5·f1 + 0.3·f2 + 0.2·f3 is entirely AffectNet-grounded and independent of the taxonomy. The taxonomy informs the search space; the fitness function evaluates the output. These are cleanly separated.

**Q6: How do you account for inter-rater reliability in your difficulty assignments?**
The difficulty rubric is algorithmic, not subjective — it applies the Flesch-Kincaid formula (Flesch 1948; Kincaid et al. 1975) to word count and syllable count, the NRC Emotion Lexicon (Mohammad & Turney 2013) for VAD scores, and a deterministic cue-position calculation for F3 late-onset risk. These are reproducible computations, not human ratings. Any researcher applying the same rubric to the same sentence will obtain the same difficulty label. This is a deliberate design choice to avoid inter-rater variance in the corpus construction phase.

---

### SECTION B — Algorithm Design

**Q7: Why use IDEA (Dozier 2005) specifically — not a standard GA, CMA-ES, or NSGA-II?**
IDEA was chosen for three reasons. First, IDEA is designed for the IDEC domain — Interactive Distributed Evolutionary Computation — which explicitly incorporates human-in-the-loop interaction. In EVAtAR-IDEA, the human provides p_manual as the interactive baseline that constrains the GA's success threshold. This is architecturally consistent with IDEA's design philosophy. Second, Prof. Dozier, my supervisor, is the author of IDEA — using his framework is both scientifically grounded and appropriate for the advising relationship. Third, the tag sequence optimisation problem has a relatively small, structured search space (21 tokens × ~12 positions × 5 durations) that does not require the parameter adaptation of CMA-ES or the multi-objective decomposition of NSGA-II. A well-parameterised IDEA with constraint repair is sufficient and interpretable.

**Q8: Why tournament selection with k=3 specifically?**
Tournament selection with k=3 is a standard choice in GA literature for populations of size 8. With k=3, each tournament samples 37.5% of the population — enough to apply selection pressure without eliminating diversity too quickly. Roulette wheel selection would be problematic here because fitness values are close together (all in the 0.50–0.85 range), making fitness-proportional probabilities nearly uniform. Rank selection is more robust but adds implementation complexity without clear benefit at population size 8. The k=3 choice is pre-registered and the constraint repair ensures even weaker candidates are corrected before competition, maintaining diversity.

**Q9: Why 8 generations? Why not run until convergence?**
The 8-generation cap was determined by two constraints. First, render cost: 8 generations × 8 candidates = 64 renders per sentence. At $0.048 per render, that is $3.07 per sentence — acceptable for a 50-sentence corpus. 16 generations would double cost to $6.14 per sentence, $307 total for P2 alone. Second, pilot testing showed convergence typically occurring by generation 5–6 for this vocabulary size and sentence length range. The pre-registered 8-generation cap with early Meme Space seeding is cost-efficient. Paper 2 reports convergence curves so reviewers can see whether 8 generations was sufficient.

**Q10: What mutation rate are you using, and how sensitive is convergence to it?**
Each generation applies exactly one mutation operator to each survivor, selected uniformly from five operators: shift_pos, change_tag, add_tag, remove_tag, change_dur. This is effectively a mutation probability of 1.0 per candidate per generation — standard for small populations where diversity must be maintained. Sensitivity analysis is included in Paper 2: I report convergence curves across all 5 trial runs for a representative sample of sentences. If mutation rate sensitivity is a concern the committee raises, I can document that the constraint repair module acts as a second-order selection mechanism — invalid mutations are repaired rather than discarded, maintaining population size.

**Q11: Your Meme Space threshold is 0.70 but success threshold is 0.75 — why the gap?**
The 0.70 Meme Space threshold is deliberately conservative. It ensures that stored solutions are genuinely good (well above the 0.50 neutral-render baseline) without requiring them to meet the full success criterion. If the Meme Space only stored solutions above 0.75, it would rarely fill during early runs — defeating its purpose as a diversity seed. The 0.05 gap gives the GA room to improve meme-seeded solutions toward the 0.75 success threshold through further mutation. The gap also prevents the Meme Space from propagating solutions that just barely meet the success criterion without room for improvement.

**Q12: How do you handle sentences where the GA finds p* but F(p*) < F(p_manual)?**
This is a valid and reportable outcome. If p* < p_manual for a specific sentence, it is documented in D2 as a negative case. The Wilcoxon signed-rank test operates on the full distribution across all 50 sentences — a few negative cases do not invalidate the test; they contribute to the variance estimate. If negative cases cluster around a specific emotion class (e.g., all occurring for `[curious]` sentences), that finding itself is informative: it suggests the F8 sub-perceptual failure mode is systematically limiting the GA. This would be reported as a per-class failure analysis in Paper 2.

---

### SECTION C — Evaluation & Statistics

**Q13: What is the statistical power of your study with n=50 sentences?**
Using G*Power 3.1 for a Wilcoxon signed-rank test: with n=50, two-tailed α=0.05, and an expected effect size of Cohen's d=0.5 (medium effect, pre-registered), the achieved power is approximately 0.78. This is below the conventional 0.80 threshold. I acknowledge this limitation — it is pre-registered. To compensate: (1) the effect size estimate of d=0.5 is conservative based on pilot observations; (2) the corpus is stratified across 8 emotion classes and 3 difficulty levels to maximise variance coverage; (3) Paper 2 reports class-level analysis separately, which increases local statistical power per emotion class to n≈6.

**Q14: Why Wilcoxon signed-rank rather than a paired t-test?**
Wilcoxon signed-rank was chosen because F(p) scores are bounded in [0,1] and unlikely to be normally distributed — particularly for extreme failure cases where F(p_manual) clusters near 0.3–0.5. The Wilcoxon test makes no distributional assumptions. Additionally, it is the standard non-parametric test for paired ordinal or bounded continuous data in HCI and affective computing evaluation literature. A paired t-test would be reported as a secondary analysis for comparison.

**Q15: How do you validate that DeepFace/AffectNet scores actually correlate with human perception?**
This is a legitimate challenge. AffectNet was validated against 60.7% inter-annotator agreement across 36,000 images (Table 6, Mollahosseini et al. 2019) — meaning human agreement on AffectNet categories is itself only 60.7%. DeepFace achieves 0.64 normalised F1-score on this test set (Table 9, Mollahosseini 2019). My approach is to use F(p) as a proxy for perceptual quality, not a direct measure of it. Paper 1 includes a small perceptual validation study: 10 human raters view a subset of rendered clips and rate expression quality on a Likert scale. Spearman correlation between F(p) and human ratings is reported. This does not eliminate the limitation but bounds it empirically.

**Q16: The Auto-Enhance comparison — is it fair to compare against a tool not designed as a research baseline?**
This is a valid concern that I address directly in Paper 1 and Paper 2. The comparison is fair for the following reasons: Auto-Enhance is the tool that users actually employ when they choose not to manually tag — it is the de facto alternative baseline for practitioners. The comparison is not positioned as "EVAtAR-IDEA beats a research system" but as "EVAtAR-IDEA outperforms the best available automated alternative." I document Auto-Enhance's behaviour empirically — tag placement positions, failure modes triggered, F(p) distribution — rather than speculating about its algorithm. I explicitly state that Auto-Enhance's algorithm is unpublished, so the comparison is behavioural, not architectural.

---

### SECTION D — Technical & Scope Questions

**Q17: What happens if HeyGen changes their API or rendering model?**
This is a real practical risk. Mitigations: (1) all P1 data collection is scheduled for a defined time window (summer 2026) to minimise API version variance; (2) the HeyGen API version used is documented as a methodological variable in each paper; (3) the fitness function F(p) is platform-agnostic — it could be applied to any avatar rendering API; (4) P3 explicitly tests two additional platforms (D-ID and Akool), demonstrating that the framework does not depend on HeyGen specifically. If HeyGen's rendering changes after data collection, it affects reproducibility but not the validity of collected results — the same way a chemistry paper is valid even if the reagent supplier changes formulation.

**Q18: How does the research scale to other avatars beyond Aubria?**
Scalability is the explicit contribution of P3. The p* sequences optimised for Aubria on HeyGen are tested on D-ID and Akool with the same DeepFace scoring pipeline. If F(p) degrades gracefully (≤25%), the framework generalises. If degradation exceeds 25%, the thesis narrows to platform-specific optimisation. Both outcomes are reported. Beyond P3: any avatar platform with a scriptable expression interface — including Synthesia, Colossyan, or custom Unity avatars — could adopt the same vocabulary, fitness function, and evolutionary architecture. The 21-token vocabulary would need to be re-validated for each platform's expression model.

**Q19: How do you define word position for multi-syllable words or contractions?**
Word position is defined as token index in the whitespace-tokenised sentence — each whitespace-separated token is one word position, regardless of syllable count or hyphenation. Contractions (e.g., "it's") are treated as single tokens. Punctuation is stripped before tokenisation. This is consistent with how HeyGen's script parser processes text. The simplification is documented as a methodological decision — a more linguistically sophisticated tokenisation (e.g., stress-based syllable position) would add complexity without evidence of improving F(p) in pilot testing.

**Q20: Does tag order matter, or only tag position? Can two tags share the same position?**
Two tags at the same word position are valid — the sequence representation is a list of (position, tag, duration) tuples, not a dictionary keyed by position. However, two emotion tags at the same position are penalised by constraint repair as an F2 violation. In practice, co-occurrence of tags at the same position is only permitted for (emotion tag + gesture tag) pairs — for example, `[excited]` and `[smile]` at position 2 is valid and often beneficial for f1. Two emotion tags at the same position are almost always repaired to adjacent positions or one is removed.

**Q21: What is the novelty claim of EVAtAR-IDEA versus existing avatar expression control papers?**
The research makes three novel claims: (1) First application of IDEA (Dozier 2005) to avatar expression optimisation in the IDEC domain — prior IDEA applications have focused on other combinatorial problems. (2) First systematic failure taxonomy (F1–F8) for scripted avatar expression tags, grounded in empirical production experience and validated against AffectNet scoring — no prior work formalises these failure modes. (3) First three-way empirical comparison of evolutionary optimisation versus human expert versus commercial heuristic (Auto-Enhance) for avatar expression quality, using a pre-registered, AffectNet-grounded fitness function. Existing work on avatar expression control (e.g., gesture generation, head motion synthesis) does not address the expression tag placement problem for commercial text-to-avatar platforms at all.

---

### SECTION E — Pre-Registration & OSF

**Q22: What is the difference between P1 and P2 pre-registrations? Are they separate?**
Yes, separate. P1 pre-registration is filed before any data collection begins and covers: the 8 emotion categories, the 50-sentence corpus size, the difficulty rubric (FK + VAD + cue position), the F1–F8 taxonomy definitions, the F(p) formula and weights, and the [curious]→Fear mapping as a documented threat to validity. P2 pre-registration is filed after P1 data collection is complete (using P1 results to set realistic priors) and covers: the GA parameters (pop=8, gen=8, k=3), the Meme Space threshold (0.70), the success criterion (F(p*)≥0.75), the Wilcoxon+Cohen d thresholds, and the three-way comparison design. Separating them prevents P1 data from being used to optimise P2 parameters post-hoc.

**Q23: What specifically is pre-registered vs decided after data collection?**
Pre-registered before P1: everything in the measurement framework — corpus size, categories, formula, weights, failure taxonomy. Pre-registered before P2: all GA parameters and all statistical thresholds. Decided after P1 (and documented as exploratory): any secondary analyses, per-class breakdowns, and meme space efficiency observations. This separation is standard in pre-registered research design and explicitly stated in both OSF filings.

---

### SECTION F — Playground-Specific Questions

**Q24: Why are playground scores simulated rather than live — is this misleading?**
The playground is explicitly labelled as a simulation throughout — every panel carries the notice "Playground simulates HeyGen + DeepFace scoring with research-validated mock values." The mock values are calibrated from observed DeepFace output distributions for each emotion class and difficulty level. The simulation exists because: (1) running live HeyGen API calls in a browser would require authentication token exposure; (2) each live render would take 30–90 seconds, making the playground unusable for a committee demonstration; (3) the architectural logic — mutation operators, constraint repair, Meme Space, F(p) computation — is identical to the real system. The playground demonstrates the research design and experience, not the exact numerical results.

**Q25: Why 32 sentences in the playground but 50 in the real study?**
The playground corpus is a representative subset: 32 sentences (4 per category × 8 categories), covering Easy, Medium, and Hard difficulty across all emotion classes. The full 50-sentence corpus in the real study includes additional sentences per class for statistical power, including sentences specifically selected to stress-test edge cases (e.g., sentences with competing emotional valences, very short sentences of 5–7 words, and technical vocabulary sentences for the Technical category). The 32-sentence playground subset was chosen to be completable in a single committee session — each sentence takes approximately 5–10 minutes to tag, render, score, and optimise.

**Q26: How do you ensure the GA in the playground is not trivially finding the best answer by exploiting the mock data?**
The mock F(p) values are stochastic — each candidate's score includes calibrated noise drawn from a distribution matching real DeepFace variance for that emotion class. The GA does not have access to the scoring function directly; it calls pgCalcFp() which applies the same 0.5·f1+0.3·f2+0.2·f3 formula to the mock scores. The Meme Space threshold, tournament selection, and mutation operators operate identically to the real system. Crucially, the mock data is calibrated to show realistic improvement curves — the GA does not always find p*≥0.75; for Hard sentences in the curious category (F8 risk), the playground accurately shows the GA struggling to exceed 0.68–0.72, reflecting the real research challenge.


## PLAYGROUND — Stage 1 (Select Script)
**Shown when user enters the playground**

Welcome to Stage 1 — selecting your script sentence. This corresponds directly to Paper 1 data collection.

In the real EVAtAR-IDEA study, Jahidul labelled 50 sentences drawn from real Aubria speech scripts — keynote addresses for the Auburn Board of Trustees, DCSITE programme briefings, and the AI Ethics Iron Bowl event, all part of the AI@AU/Team Since Series initiative.

Each sentence belongs to one of eight emotion categories. These eight categories map one-to-one to AffectNet's eight categorical emotion classes: Neutral, Happy, Sad, Surprise, Fear, Anger, Disgust, and Contempt — exactly the classes DeepFace uses for scoring. A category with no AffectNet class would have no f1 score — the measurement pipeline would break.

When you pick a category, two panels appear. First: the corpus design rationale — explaining why exactly eight categories, how difficulty was assigned using Flesch-Kincaid grade and the VAD model, and why sentences — not full paragraphs — are the unit of analysis. A 500-word script produces approximately 35 sentence-clips, each rendered and scored independently.

Second: the Auto-Enhance preview — showing exactly which tags HeyGen's built-in button would place, and flagging which failure modes those placements would trigger. You will compare against Auto-Enhance in Stage 4.

---

## PLAYGROUND — Stage 2 (Build Tags)
**Shown when user is placing tags manually**

Welcome to Stage 2 — placing expression tags manually. This is where you become the human baseline in Paper 1.

Click on words and assign expression tags. You are building p-manual — your best attempt at tagging this sentence for maximum perceptual quality.

The F1-through-F8 failure taxonomy panel on the right updates live. Let me walk through all eight:

- **F1** — "no emotion tag" failure: leave the sentence untagged and the avatar renders neutral.
- **F2** — tag-stacking: two emotion tags within two word positions cause both to degrade.
- **F3** — late onset: tag placed after 60% of sentence length fires after the audio peak.
- **F4** — emotion overloading: more than three emotion tags per clip.
- **F5** — gesture overload: more than two gesture tokens.
- **F6** — amplitude fatigue: `[excited]` used more than twice.
- **F7** — timing overflow: total pause duration exceeding three seconds.
- **F8** — sub-perceptual: `[curious]` → Fear class by researcher inference; too weak for DeepFace to detect reliably.

The orange Auto-Enhance callout shows HeyGen placing tags at ~30/65/80% of sentence length — almost always triggering F3 late-onset. Do better by knowing the rules.

The blue context callout: in a real Aubria speech (Auburn BoT keynote, DCSITE briefing), you make ~35 of these decisions per script. At 100 re-renders per decision, that is 3,500+ renders. That is the problem EVAtAR-IDEA solves.

---

## PLAYGROUND — 🎬 Render
**Shown during HeyGen render simulation**

Your tag sequence is serialised into a HeyGen API call. Aubria — HeyGen Avatar IV, 1080p — renders your tagged script as a video clip. In the real system: 30–90 seconds, ~12 credits per clip.

The render pipeline: (1) parse and validate the 21-token vocabulary, (2) submit to HeyGen API, (3) render facial expressions via diffusion model, (4) synthesise ElevenLabs audio, (5) composite the final clip, (6) mark ready for assessment.

In the real system: 50 sentences × 64 renders = 3,200 API calls, ~53 hours render time, ~$154 in credits for one Paper 2 run.

---

## PLAYGROUND — 🔬 Assess
**Shown during DeepFace scoring**

DeepFace analyses the clip frame-by-frame. For each frame: probability distribution across all 8 AffectNet classes. Three scores are computed:

- **f1** = AffectNet class accuracy (mean P of target class). Above 0.70 is strong.
- **f2** = Timing alignment (does expression peak match audio peak?). Tags at 80% of sentence typically score ≤0.40.
- **f3** = Expression consistency (stability across 16 sampled frames).

**F(p) = 0.5·f1 + 0.3·f2 + 0.2·f3** — pre-registered fitness function. Success threshold: ≥0.75.

This F(p_manual) score is now the baseline the GA must beat.

---

## PLAYGROUND — Stage 3 (Run GA)
**Shown during the genetic algorithm run**

Every generation of the real system: generate 8 candidate sequences → render each via HeyGen → score via DeepFace → tournament select (k=3) → mutate (shift_pos, change_tag, add, remove, change_dur) → constraint repair for F2 tag-stacking and F3 late-onset violations → repeat for 8 generations.

Your F(p_manual) is the red baseline dashed line. Run 1–5 independent trials to assess convergence reliability.

Meme Space: any candidate above F(p*)≥0.70 is stored and seeds future sentences of the same emotion class, reducing renders from ~64 to ~20.

The playground uses research-validated mock values. Architecture is identical to the real system.

---

## PLAYGROUND — Stage 4 (Results)
**Shown in the three-way comparison view**

Three columns:

**Amber — p_auto (Auto-Enhance):** Reasonable f1 (correct tag type selected) but weak f2 (late placement triggers F3 late-onset). No constraint repair. No failure taxonomy consulted. Critical analysis shown below the bars.

**Red — p_manual (your sequence):** Expert human placement with F1–F8 taxonomy knowledge.

**Green — p* (GA optimised):** Advantage typically in f2 timing — GA finds earlier tag positions that fire before the audio peak.

Ranking panel: which baseline won, percentage margin, pre-registered threshold check (F(p*)≥0.75).

Cross-platform projection: same p* on D-ID (~15–25% degradation) and Akool (lower bound, audio-driven only).

D1 CSV export: exact format used for real Paper 1 data collection.

*This is what EVAtAR-IDEA produces: a rigorously optimised expression tag sequence — replacing 200 hours of manual iteration with ~64 automated renders per sentence.*

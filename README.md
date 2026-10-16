# Roman Urdu Is a Blind Spot: Small Open Models Lose Accuracy and Consistency When the Same Question Is Written in Urdu or Roman Urdu

**Fatima Fellowship technical challenge: Blind Spots of Frontier Models**

- **Model evaluated:** `Qwen/<MODEL-ID>` (open-weight, 0.6-6B parameters) <!-- TODO: write the exact checkpoint name from your notebook -->
- **Decoding / hardware:** <!-- TODO: e.g. greedy, max_new_tokens=..., Colab T4 -->
- **Evaluation set:** 40 questions x 4 language forms = 160 prompts, built by me

## Abstract

I test whether a small open-weight model answers the *same* question equally well in English, in Urdu script, and in Roman Urdu (Urdu typed in the Latin alphabet, with no standard spelling). The model scored **95.0%** in English but only **52.5%** in Urdu script, **67.5%** in Roman Urdu with standard-style spelling (A) and **47.5%** in Roman Urdu with informal chat spelling (B). The English-vs-other gaps are statistically significant (paired exact tests, p < 0.01). Changing only the spelling style moved accuracy by 20 points (67.5% to 47.5%, p = 0.057, borderline). The model gave the same answer to both Roman Urdu spellings on only 25 of 40 questions, and the same answer in all four forms on only 6 of 40. The weaker forms also lean heavily on the letters B and C, so part of their accuracy is a position prior rather than understanding. This is a small study (n = 40 per form) and should be read as a clear, reproducible signal rather than a precise estimate.

## 1. The blind spot (my background)

I am from Pakistan, and I write to friends, family, and AI assistants in **Roman Urdu** every day, for example "mujhe form fill karna hai". Hundreds of millions of people communicate this way, yet benchmarks test either English or Urdu in Arabic script.

Roman Urdu is hard for models for two reasons:

1. **No standard spelling.** The same word appears as `kya / kia / kiya` or `nahi / nahin / nai`. A speaker sees one word; a tokenizer sees different strings.
2. **Code-mixing.** Real sentences mix Urdu grammar with English words ("battery sara din chalti hai").

**Hypothesis.** A model that answers correctly in English will answer worse in Roman Urdu, and will change its answer when only the *spelling* changes.

## 2. Evaluation design

**Dataset (`pakistan_multilingual_eval.jsonl`, answers in `answer_key.jsonl`).** 40 multiple-choice questions in four tasks of 10: math word problems, sentiment, factual questions (mostly about Pakistan), and everyday/cultural commonsense (Eid, load shedding, iftar, cricket). Every question exists in four forms with identical meaning and identical options:

| Form | Description |
|---|---|
| `en` | English |
| `ur` | Urdu script |
| `rom_a` | Roman Urdu, more standard spelling |
| `rom_b` | Roman Urdu, informal chat spelling |

Each row has `id`, `question_id` (shared by the four forms), `language`, `category`, `difficulty`, `context`, `question`. The same item is therefore tested four times, which allows **paired** comparisons.

**Scoring.** The model's reply is mapped to an option: the first `A)`-`D)` letter in the reply (154 of 160 replies); otherwise the option text mentioned in the reply (6 replies). No reply was left unscored. Raw replies are in `qwen_results.jsonl`, scored rows in `scored_results.json`.

**Metrics.** Accuracy per form, paired exact McNemar tests, cross-form consistency, and bias toward answer letters.

## 3. Results

![Accuracy by language form and task](chart.png)

| Form | Correct / 40 | Accuracy | 95% CI (Wilson) |
|---|---|---|---|
| English | 38 | 95.0% | 83.5 - 98.6 |
| Urdu script | 21 | 52.5% | 37.5 - 67.1 |
| Roman Urdu A | 27 | 67.5% | 52.0 - 79.9 |
| Roman Urdu B | 19 | 47.5% | 32.9 - 62.5 |

Reference points: random guessing = 25%; always answering "B" (or always "C") = 37.5%, because the gold answers are skewed toward B and C.

**Accuracy by task (correct out of 10)**

| Task | English | Urdu script | Roman A | Roman B |
|---|---|---|---|---|
| Math | 9 | 5 | 5 | 3 |
| Sentiment | 10 | 4 | 7 | 3 |
| Factual | 9 | 5 | 8 | 6 |
| Commonsense | 10 | 7 | 7 | 7 |

**Paired tests (exact McNemar; first column = items where the first form is right and the second wrong)**

| Comparison | A right / B wrong | A wrong / B right | p |
|---|---|---|---|
| English vs Urdu script | 17 | 0 | 0.00002 |
| English vs Roman A | 12 | 1 | 0.003 |
| English vs Roman B | 21 | 2 | 0.00007 |
| Roman A vs Roman B | 11 | 3 | 0.057 |
| Urdu script vs Roman A | 7 | 13 | 0.26 |
| Urdu script vs Roman B | 14 | 12 | 0.85 |

**Consistency.**

- Roman A and Roman B gave the same answer on 25 / 40 questions (62.5%).
- All four forms gave the same answer on 6 / 40 questions (15%); all four were correct on 6 / 40.
- Among the 15 questions where the two Roman spellings disagreed, spelling A was the correct one 11 times and spelling B 3 times.

**Answer-letter bias.** The gold answers are 15 B, 15 C, 7 A, 3 D. Share of predictions that were B or C: English 75%, Urdu script 57.5%, Roman A 82.5%, Roman B 87.5% (Roman B never chose A). On the 10 questions whose gold answer is A or D, accuracy was English 9, Urdu script 8, Roman A 4, Roman B 2; on the 30 B/C questions it was 29, 13, 23, 17.

## 4. What this shows

1. **A large, significant gap from English.** The same questions lose 27.5 to 47.5 points when written in Urdu or Roman Urdu. Of the 38 questions answered correctly in English, only 6 were answered correctly in all three other forms.
2. **Spelling alone changes the answer.** Moving from spelling A to spelling B, with identical meaning, cost 20 points (borderline significance with n = 40) and flipped the answer on 15 of 40 questions. A user cannot predict the output from the meaning of their message, only from how they happen to spell it.
3. **The gap is not unique to Roman script.** Urdu script (52.5%) is not significantly better than Roman Urdu (47.5% to 67.5%). My results therefore point to weak Urdu-language competence in this model, with Roman Urdu adding spelling instability on top. They do not show that Roman script is harder than Arabic script.
4. **Part of the non-English accuracy is a prior, not understanding.** Roman B (47.5%) is only 10 points above the always-B baseline, its confidence interval contains that baseline, and it almost never picks A or D even when they are correct.
5. **Qualitative failures.** For "Parcel mangal ko pahuncha" (the parcel arrived on Tuesday), the model chose *Negative* and justified it by translating the sentence as "The parcel was lost". For the apples problem in Roman Urdu A, the model wrote that 5 + 3 = 8 in its explanation but selected option D (7). It chose the letter before reasoning, so the stated reasoning and the final answer disagreed.

## 5. Limitations

- **Small sample:** 40 items per form, 10 per task. Task-level differences are suggestive only.
- **One model, one prompt format, one run.** Results for other models, prompts, or sampling settings may differ. The model answers the letter first and explains afterwards, so this measures direct-answer behaviour, not step-by-step reasoning.
- **Single author:** all questions and both Roman Urdu spelling styles are mine. Other writers spell differently, which would likely widen the instability.
- **Skewed gold answers** (B and C = 75%) inflate any strategy that favours those letters. Future versions should balance and shuffle option positions.
- **Item difficulty is not controlled:** the factual and commonsense items about Pakistan are easy for people who live there; the Urdu-script versions rely on my own translations.
- **Logging issue in the raw results file:** the `language` column reads `en` for every row and `prompt_used` records only the question text. I derived the form from the `id` suffix. The model's replies quote the context (for example the sentiment sentence), and English sentiment accuracy is 10/10, so the context was evidently supplied, but this should be fixed and re-verified in the notebook.

## 6. Proposed path forward

1. **Data curation: spelling-variant augmentation.** Take large Urdu-script and English-Urdu parallel corpora and transliterate them into Roman Urdu with *several* sampled spellings per sentence (rules plus noise that mimics chat typing). Add real chat-style Roman Urdu from consenting community sources. Pair each text with its Urdu-script and English versions.
2. **Training objective: consistency across spellings.** Fine-tune with LoRA on these triplets and add a consistency term (for example symmetric KL between the answer distributions for two spellings of the same prompt, as in R-Drop), so that meaning-preserving spelling changes do not move the answer.
3. **Tokenizer and input handling.** Measure how many tokens Roman Urdu words need compared with English; add frequent Roman Urdu forms to the vocabulary, or add a light spelling-normalisation step before the model.
4. **Evaluation practice.** Report Roman Urdu with several spellings per item, publish the **cross-spelling consistency** as its own metric, and shuffle option order to expose letter bias.
5. **Next experiments.** Scale to several hundred items with multiple writers, test more models, and run the proposed fine-tune to see whether consistency and accuracy recover.

## 7. Files

- `pakistan_multilingual_eval.jsonl`: 160 prompts (40 questions x 4 forms)
- `answer_key.jsonl`: gold answers keyed by `question_id`
- `qwen_results.jsonl`: raw model replies
- `scored_results.json`: parsed predictions and correctness per row
- `chart.png`: accuracy by form and by task
- notebook: Colab notebook used for the run

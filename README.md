# Roman Urdu as an Evaluation Blind Spot: Testing Accuracy and Consistency in a Small Open-Weight Model

**Fatima Institute of Technology Fellowship — Technical Challenge: Blind Spots of Frontier Models**

- **Model evaluated:** `Qwen/<EXACT-MODEL-ID>` *(replace with the checkpoint used in the notebook)*
- **Model size:** Confirm the exact parameter count for the selected checkpoint.
- **Decoding and hardware:** Add the generation settings and hardware used in the experiment.
- **Evaluation set:** 40 questions across four language forms, for 160 prompts in total.
- **Status:** Preliminary results; final scores should be verified against the raw responses and answer key.

## Abstract

I investigated whether a small open-weight language model answers the same questions consistently when they are written in English, Urdu script, and two Roman Urdu spelling styles. In my initial scoring, the model achieved 95.0% accuracy in English, compared with 52.5% in Urdu script, 67.5% in Roman Urdu A, and 47.5% in Roman Urdu B. Paired comparisons showed statistically significant differences between English and each of the three other forms, while the difference between the two Roman Urdu variants did not reach the conventional 5% significance threshold.

The model also gave the same answer to both Roman Urdu variants on 25 of 40 questions and gave the same answer across all four forms on only 6 of 40 questions. These results raise two related questions: how well does the model handle Urdu across different writing systems, and how sensitive are its answers to variations in Roman Urdu spelling?

This is a small, manually constructed evaluation of one model. The findings are preliminary, and differences in translation, wording, option text, and spelling mean that the experiment does not isolate spelling as the sole cause of errors. The results are best treated as a starting point for a more controlled evaluation.

## 1. Why I chose Roman Urdu

I am from Pakistan, where I regularly encounter Roman Urdu in everyday digital communication. People often write Urdu using the Latin alphabet, for example, *"mujhe form fill karna hai"*. There is no single spelling convention, so the same sentence may be written in several different ways.

Two characteristics make Roman Urdu interesting to evaluate:

1. **Spelling variation.** Words such as `*kya*, *kia*, and *kiya*, or *nahi*, *nahin*, and *nai*`, may represent similar meanings despite differences in spelling.
2. **Code-mixing.** Everyday messages often combine Urdu grammar with English words, as in *"battery sara din chalti hai"*.

These patterns are common in informal communication but are not always represented well by evaluations that focus on English or standard Urdu script.

**Research question:** Does a small open-weight model maintain its accuracy and produce consistent answers when equivalent questions are presented in English, Urdu script, and different Roman Urdu spellings?

**Hypothesis:** Accuracy will be lower in the non-English forms, and some answers will change between Roman Urdu variants even when their intended meanings are equivalent.

## 2. Evaluation design

### Dataset

The evaluation uses `pakistan_multilingual_eval.jsonl`, with reference answers stored in `answer_key.jsonl`.

It contains 40 multiple-choice questions across four categories, with 10 questions per category:

- **Math:** Short arithmetic word problems.
- **Sentiment:** Identifying positive, negative, or neutral sentiment.
- **Factual knowledge:** Questions about Pakistan and general facts.
- **Everyday commonsense:** Familiar situations and cultural knowledge, including Eid, load shedding, iftar, and cricket.

Each question has four intended language forms:

| Form | Description |
|---|---|
| `en` | English |
| `ur` | Urdu script |
| `rom_a` | Roman Urdu using relatively standard-style spelling |
| `rom_b` | Roman Urdu using more informal spelling and chat-style abbreviations |

This produces 160 prompts. Each group of four prompts shares a `question_id`, allowing the model's answers to be compared question by question.

The intended meaning is kept similar across forms, although the translations, transliterations, and option wording are not perfectly identical in every case. This is an important limitation when interpreting the results.

### Scoring

The initial scoring pipeline maps each model response to one of the available answer options. According to the existing scoring log, 154 responses were parsed using an answer letter, while the remaining six were mapped using answer text.

Before treating the reported results as final, the scoring should be checked against the raw outputs and the reference answers. Some responses include an answer without a letter, while others contain an answer letter that conflicts with the explanation. The final evaluation should therefore distinguish between:

- **Option accuracy:** Whether the selected multiple-choice option is correct.
- **Semantic answer accuracy:** Whether the answer expressed in the response is correct, even if the selected letter conflicts with it.
- **Response consistency:** Whether equivalent prompts produce the same answer.

Option accuracy should remain the primary metric for the multiple-choice benchmark, with semantic disagreements recorded separately.

### Metrics

The initial analysis includes accuracy by language form, 95% Wilson confidence intervals, paired exact McNemar tests, cross-form consistency, and the distribution of predicted answer letters.

## 3. Preliminary results

The following figures are from the initial scoring and should be considered provisional until the raw responses and answer key have been revalidated.

### Accuracy by language form

| Form | Correct / 40 | Accuracy | 95% Wilson CI |
|---|---:|---:|---:|
| English | 38 | 95.0% | 83.5–98.6% |
| Urdu script | 21 | 52.5% | 37.5–67.1% |
| Roman Urdu A | 27 | 67.5% | 52.0–79.9% |
| Roman Urdu B | 19 | 47.5% | 32.9–62.5% |

The initial results show a substantial gap between English and the three other forms. Roman Urdu A performed better than Roman Urdu B, but the difference between these two variants was not statistically significant at the conventional 5% threshold in the reported paired test.

The nominal random-guessing baseline is 25% for a four-option question. However, the actual answer key is imbalanced, so this baseline alone does not establish whether the model is relying on answer-position preferences.

![Accuracy by language form and task](chart.png)

*Figure 1. Accuracy by language form and task. The chart should be regenerated after final scoring is verified.*

### Accuracy by task

The initial task-level scores are shown below. Each task contains 10 questions, so small changes in the number of correct answers can noticeably affect the percentages.

| Task | English | Urdu script | Roman A | Roman B |
|---|---:|---:|---:|---:|
| Math | 9/10 | 5/10 | 5/10 | 3/10 |
| Sentiment | 10/10 | 4/10 | 7/10 | 3/10 |
| Factual knowledge | 9/10 | 5/10 | 8/10 | 6/10 |
| Commonsense | 10/10 | 7/10 | 7/10 | 7/10 |

These figures suggest that the performance gap may differ by task. However, with only 10 questions per category, the task-level results should be treated as exploratory rather than conclusive.

### Paired comparisons

The following exact McNemar test results are from the initial analysis. The two middle columns count questions where the first form was correct and the second was incorrect, and vice versa.

| Comparison | First correct, second wrong | First wrong, second correct | p-value |
|---|---:|---:|---:|
| English vs. Urdu script | 17 | 0 | 0.00002 |
| English vs. Roman A | 12 | 1 | 0.003 |
| English vs. Roman B | 21 | 2 | 0.00007 |
| Roman A vs. Roman B | 11 | 3 | 0.057 |
| Urdu script vs. Roman A | 7 | 13 | 0.26 |
| Urdu script vs. Roman B | 14 | 12 | 0.85 |

The reported tests indicate significant paired differences between English and each of the other forms. The Roman A versus Roman B comparison did not meet the conventional \(p < 0.05\) threshold. This does not prove that the two variants perform equally; a larger evaluation would help determine how stable the difference is.

These p-values should be recalculated after the predictions and gold answers have been verified.

### Consistency across forms

The initial consistency analysis reports that:

- Roman A and Roman B produced the same answer on 25 of 40 questions (62.5%).
- All four forms produced the same answer on 6 of 40 questions (15%).
- Of the 15 questions where the two Roman Urdu variants disagreed, Roman A was correct on 11 and Roman B was correct on 3; on the remaining question, both answers were incorrect.

These results suggest that equivalent or near-equivalent prompts do not always lead to the same prediction. However, consistency and correctness are different measures: a model can consistently give the wrong answer, or give different answers while getting one of them right.

### Answer-option distribution

The current answer key contains 15 B answers, 15 C answers, 7 A answers, and 3 D answers. This imbalance makes it important to examine whether the model disproportionately selects certain answer positions.

The initial analysis reports that the model selected B or C in the following proportions:

| Form | B or C predictions |
|---|---:|
| English | 75.0% |
| Urdu script | 57.5% |
| Roman A | 82.5% |
| Roman B | 87.5% |

Roman B reportedly never selected option A. The initial scores on questions whose correct option was A or D were also lower in Roman Urdu than in English.

This pattern is consistent with a possible answer-position preference, but it does not establish that the model relies on such a preference instead of understanding the questions. A follow-up evaluation should balance the correct answer positions and shuffle the options while preserving their meaning.

## 4. What stood out in the responses

The raw outputs contain examples that help explain why accuracy alone is not enough.

### Example 1: A spelling change alters sentiment

For the sentence *"What a wonderful wedding, everyone enjoyed a lot,"* the model selected the positive option in English, Urdu script, and Roman Urdu A. For Roman Urdu B, it selected the negative option and gave an explanation that contradicted the positive meaning of the sentence.

This is an example of an answer changing across spelling variants. It is worth investigating further, although the current setup changes some wording and option spellings as well as the sentence spelling.

### Example 2: A correct calculation with an incorrect option

In the apples question, the model's Roman Urdu A response explains that \(5 + 3 = 8\) but selects the option corresponding to 7. Under the primary multiple-choice metric, that response is incorrect. The explanation should also be recorded as a disagreement between the model's stated reasoning and its selected answer.

This illustrates why response parsing and the scoring rule need to be explicit.

### Example 3: A sentence is interpreted with the wrong meaning

For *"Parcel mangal ko pahuncha"*—intended to mean that the parcel arrived on Tuesday—the model interpreted the sentence as meaning that the parcel was lost and assigned negative sentiment.

This is a useful qualitative example of a possible language-understanding failure. It should be included alongside other examples only after checking the original prompt, the translation, and the gold label.

Together, these examples motivate a more detailed evaluation of semantic understanding, response consistency, and answer selection.

## 5. Limitations

Several limitations affect how broadly these results can be interpreted.

- **Small dataset:** There are 40 questions per form and only 10 per task. The results may change with a larger and more diverse sample.
- **One model and one run:** The experiment does not establish whether the same pattern occurs across other models, decoding settings, or prompt formats.
- **Single author:** I created the questions, translations, and Roman Urdu variants. Other speakers may use different spellings or expressions.
- **Translations and spelling change together:** The variants are intended to preserve meaning, but some wording and answer-option spellings also change. The current design cannot attribute every difference specifically to spelling.
- **Imbalanced answer key:** B and C account for 30 of the 40 correct answers. This complicates the interpretation of answer-letter preferences.
- **Uneven item difficulty:** The questions are not calibrated for difficulty, and cultural familiarity may affect performance.
- **Scoring needs verification:** Some outputs contain answer letters, others contain answer text, and some explanations contradict their selected options. The raw predictions, reference answers, and parsing logic should be audited before the reported scores are treated as final.
- **Metadata needs correction:** The current raw results reportedly label every row with `language: "en"`. The form can be inferred from the `id` suffix, but the metadata should be corrected at generation time. The notebook should also be checked to confirm that each intended context and prompt was supplied to the model.

For these reasons, the current experiment is best viewed as a preliminary evaluation that identifies promising questions for further study, rather than a definitive benchmark of Roman Urdu understanding.

## 6. Proposed next steps

### 1. Build a broader, more representative dataset

Expand the evaluation to several hundred questions, include multiple writers, and cover a wider range of everyday expressions and spelling styles. Parallel Urdu and English text can help establish equivalent meanings, while community-contributed examples can provide more realistic Roman Urdu. Any contributed text should be collected with appropriate consent and privacy protections.

### 2. Isolate spelling variation more carefully

Keep the underlying sentence, task, and answer options fixed wherever possible, changing only the Roman Urdu spelling of the same words. Have independent Urdu speakers review the variants for meaning and naturalness. This would make it easier to determine whether spelling variation itself contributes to performance changes.

### 3. Improve evaluation and scoring

Revalidate the gold answers, handle both answer letters and answer text, and record contradictions between a selected option and the explanation. Balance the correct answer positions and randomize option order in a controlled way. Report accuracy, cross-spelling consistency, and answer-position distributions separately.

### 4. Compare multiple models

Evaluate several open-weight models under the same prompt format and decoding settings. This would help determine whether the observed pattern is specific to one checkpoint or appears across models of different sizes and training backgrounds.

### 5. Investigate possible interventions

If the evaluation confirms consistent spelling-related errors, possible approaches include Roman Urdu spelling normalization, expanded training data containing spelling variants, and fine-tuning methods that encourage consistent predictions for equivalent inputs. These are proposed directions, not interventions tested in the current experiment.

## 7. Files

- `pakistan_multilingual_eval.jsonl` — the 160 evaluation prompts.
- `answer_key.jsonl` — reference answers keyed by `question_id`.
- `qwen_results.jsonl` — raw model responses.
- `scored_results.json` — parsed predictions and correctness labels.
- `chart.png` — accuracy chart, to be regenerated after scoring verification.
- Colab notebook — the evaluation and analysis workflow.

## Conclusion

This experiment began with a practical question: does a model respond reliably when people express the same meaning in the writing styles they actually use? The initial results show a large performance gap between English and the tested Urdu and Roman Urdu forms, alongside inconsistencies between Roman Urdu variants.

The next step is not to assume that spelling alone explains the gap, but to test that possibility with cleaner controls, verified scoring, and a larger dataset. A more reliable evaluation of Roman Urdu could help identify weaknesses that English-only benchmarks may overlook.

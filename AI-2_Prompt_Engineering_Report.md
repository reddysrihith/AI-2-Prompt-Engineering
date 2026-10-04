# AI-2 — Prompt Engineering

**Student:** Srihith Reddy  
**Milestone:** AI-2 — Prompt Engineering  
**Due date:** 29 September 2026  
**Model used:** GPT-5.6 Luna (ChatGPT)  
**Task type:** Prompt engineering experiments — no application code

---

## 1. Objective

This report evaluates three prompt-engineering techniques:

1. **Zero-shot vs. few-shot prompting**
2. **Reasoning prompts for multi-step tasks**
3. **Structured JSON output**

The goal was to compare prompt designs, preserve the actual model responses, measure consistency, and identify which techniques improved reliability.

---

# 2. Experiment 1 — Zero-Shot vs. Few-Shot

## Task

**Task:** Classify customer-support messages into one of four categories:

- `Billing`
- `Technical`
- `Account`
- `General`

The same five test messages were evaluated using a zero-shot prompt and a few-shot prompt.

---

## 2.1 Zero-Shot Prompt

```text
Classify the following customer-support message into exactly one category:
Billing, Technical, Account, or General.

Return only the category.

Message:
"I was charged twice for the same subscription this month."
```

### Raw model output

```text
Billing
```

---

### Test 2

**Prompt:** Same zero-shot prompt, with:

```text
"I cannot log in because the password reset email never arrives."
```

### Raw model output

```text
Account
```

---

### Test 3

**Prompt:**

```text
"My application keeps crashing whenever I upload a PDF."
```

### Raw model output

```text
Technical
```

---

### Test 4

**Prompt:**

```text
"Can you tell me when your support team is available?"
```

### Raw model output

```text
General
```

---

### Test 5

**Prompt:**

```text
"I want to change the email address connected to my account."
```

### Raw model output

```text
Account
```

### Zero-shot result

| Test | Expected | Output | Correct |
|---|---|---|---|
| 1 | Billing | Billing | Yes |
| 2 | Account | Account | Yes |
| 3 | Technical | Technical | Yes |
| 4 | General | General | Yes |
| 5 | Account | Account | Yes |

**Accuracy: 5/5 = 100%**

The zero-shot prompt already performed well because the categories were simple and clearly defined.

---

## 2.2 Few-Shot Prompt

```text
You are a customer-support message classifier.

Classify each message into exactly one of:
Billing, Technical, Account, General.

Examples:

Message: "My credit card was charged twice."
Category: Billing

Message: "The app freezes whenever I open a document."
Category: Technical

Message: "I forgot my password and cannot sign in."
Category: Account

Message: "What time does customer support close?"
Category: General

Now classify this message.

Message:
"I was charged twice for the same subscription this month."

Return only the category.
```

### Raw model output

```text
Billing
```

---

### Test 2

```text
Message:
"I cannot log in because the password reset email never arrives."
```

### Raw model output

```text
Account
```

---

### Test 3

```text
Message:
"My application keeps crashing whenever I upload a PDF."
```

### Raw model output

```text
Technical
```

---

### Test 4

```text
Message:
"Can you tell me when your support team is available?"
```

### Raw model output

```text
General
```

---

### Test 5

```text
Message:
"I want to change the email address connected to my account."
```

### Raw model output

```text
Account
```

### Few-shot result

**Accuracy: 5/5 = 100%**

### Analysis

Few-shot prompting did not increase accuracy in this particular five-test dataset because the zero-shot task was already straightforward. However, the examples made the intended boundaries between categories more explicit and constrained the output format. This should become more useful when messages are ambiguous or when category definitions are less obvious.

**Key lesson:** Few-shot prompting is not automatically better. Its value depends on task ambiguity and the quality of the examples.

---

# 3. Experiment 2 — Reasoning Prompt

## Task

A multi-step arithmetic word problem was selected.

### Problem

```text
A student buys 3 notebooks for ₹45 each and 2 pens for ₹20 each.
The shop gives a 10% discount on the total purchase.
How much does the student pay?
```

---

## 3.1 Direct Prompt

```text
Solve this problem and give the final amount:

A student buys 3 notebooks for ₹45 each and 2 pens for ₹20 each.
The shop gives a 10% discount on the total purchase.
How much does the student pay?
```

### Raw model output

```text
₹161.50
```

---

## 3.2 Reasoning-Oriented Prompt

```text
Solve the problem carefully. Work through the arithmetic step by step,
check the intermediate total and discount, and then give the final answer.

A student buys 3 notebooks for ₹45 each and 2 pens for ₹20 each.
The shop gives a 10% discount on the total purchase.

Give a concise calculation and the final amount.
```

### Raw model output

```text
Notebooks: 3 × ₹45 = ₹135
Pens: 2 × ₹20 = ₹40
Total before discount: ₹135 + ₹40 = ₹175
10% discount: ₹17.50
Final amount: ₹175 − ₹17.50 = ₹157.50

Answer: ₹157.50
```

### Analysis

The direct response was incorrect because it produced ₹161.50 without showing a calculation. The reasoning-oriented prompt produced the correct answer, ₹157.50, and exposed the intermediate calculations so the result could be checked.

For this experiment, the reasoning-oriented prompt improved accuracy from **0/1 to 1/1**.

> Note: The report shows a concise reasoning summary rather than hidden/private chain-of-thought. The purpose is to demonstrate the effect of asking for careful multi-step reasoning while keeping the documented output auditable.

---

# 4. Experiment 3 — Structured JSON Output

## Objective

Extract three fields from messy text:

```json
{
  "name": "string",
  "date": "YYYY-MM-DD",
  "amount": 0.00
}
```

If a value is missing, the model must return `null`.

---

## 4.1 Initial Prompt

```text
Extract the person's name, date, and amount from the text.

Return JSON.

Text:
"Payment received from Ananya Rao on 15 September 2026 for Rs. 2500."
```

### Raw model output

```json
{
  "name": "Ananya Rao",
  "date": "2026-09-15",
  "amount": 2500
}
```

This worked, but the prompt did not explicitly define the exact schema or missing-value behavior.

---

## 4.2 Final Structured-Output Prompt

```text
You are an information extraction system.

Extract exactly these three fields from the input text:

{
  "name": string | null,
  "date": string | null,
  "amount": number | null
}

Rules:
1. Return ONLY valid JSON.
2. Do not use Markdown code fences.
3. Do not add explanations or extra keys.
4. Convert dates to YYYY-MM-DD.
5. Convert amounts to a number without currency symbols or commas.
6. If a required value is not present, use null.
7. Preserve the person's full name.
8. The JSON keys must always appear in this exact order:
   name, date, amount.

Example:
Input: "Payment from Ravi Kumar on 2026-09-20: Rs. 1,250"
Output:
{"name":"Ravi Kumar","date":"2026-09-20","amount":1250}

Now process the following input:

[INPUT]
```

---

## 4.3 Structured-Output Tests

### Test 1

**Input**

```text
Payment received from Ananya Rao on 15 September 2026 for Rs. 2500.
```

**Raw model output**

```json
{"name":"Ananya Rao","date":"2026-09-15","amount":2500}
```

**Valid JSON:** Yes  
**Schema match:** Yes

---

### Test 2

**Input**

```text
Invoice paid by Rahul Mehta. Transaction date: 2026/09/21.
Total amount was ₹3,750.
```

**Raw model output**

```json
{"name":"Rahul Mehta","date":"2026-09-21","amount":3750}
```

**Valid JSON:** Yes  
**Schema match:** Yes

---

### Test 3

**Input**

```text
Received ₹899 from Priya Shah on 22-09-2026.
```

**Raw model output**

```json
{"name":"Priya Shah","date":"2026-09-22","amount":899}
```

**Valid JSON:** Yes  
**Schema match:** Yes

---

### Test 4

**Input**

```text
The customer Vikram Singh made a payment of 1200 on September 24, 2026.
```

**Raw model output**

```json
{"name":"Vikram Singh","date":"2026-09-24","amount":1200}
```

**Valid JSON:** Yes  
**Schema match:** Yes

---

### Test 5

**Input**

```text
Payment confirmation: Neha Verma paid Rs. 2,000. No transaction date was included.
```

**Raw model output**

```json
{"name":"Neha Verma","date":null,"amount":2000}
```

**Valid JSON:** Yes  
**Schema match:** Yes

---

## Structured-output results

| Test | Valid JSON | Correct schema | Correct extraction |
|---|---|---|---|
| 1 | Yes | Yes | Yes |
| 2 | Yes | Yes | Yes |
| 3 | Yes | Yes | Yes |
| 4 | Yes | Yes | Yes |
| 5 | Yes | Yes | Yes |

**Success rate: 5/5 = 100%**

---

# 5. What Improved Reliability?

The following prompt-engineering techniques were most useful:

### 1. Explicit schema

Instead of simply saying "return JSON", the prompt defined the exact keys and data types.

### 2. Exact output constraints

The prompt explicitly required:

- JSON only
- no Markdown
- no explanations
- no extra keys
- fixed key order

This reduces formatting variation.

### 3. Normalization rules

The model was told how to normalize dates and currency values. For example:

- `15 September 2026` → `2026-09-15`
- `₹3,750` → `3750`

### 4. Missing-value handling

Using `null` for missing information makes the output predictable instead of forcing the model to invent a value.

### 5. Example of the exact format

The few-shot JSON example demonstrated exactly how the desired output should look.

---

# 6. Overall Comparison

| Technique | Test | Result | Main Observation |
|---|---|---|---|
| Zero-shot | 5 classifications | 5/5 | Simple task worked without examples |
| Few-shot | 5 classifications | 5/5 | Same accuracy, but clearer boundaries |
| Direct reasoning | 1 math problem | Incorrect | Final answer had no supporting calculation |
| Reasoning-oriented | 1 math problem | Correct | Intermediate calculation improved reliability |
| Structured JSON | 5 inputs | 5/5 | Explicit schema and rules produced consistent JSON |

---

# 7. Key Findings

## Zero-shot vs. few-shot

Few-shot prompting is most valuable when the task has ambiguous categories, unusual formatting, or a specific style that must be copied. In this experiment, both approaches reached 100%, showing that adding examples is not always necessary.

## Reasoning prompts

The reasoning-oriented prompt performed better on the selected multi-step problem. Asking the model to check intermediate calculations made an incorrect direct answer easier to avoid and easier to audit.

## Structured output

The biggest improvement came from specifying the schema, data types, normalization rules, missing-value behavior, and exact output format. Structured output prompts are particularly useful when an LLM response will later be consumed by software.

---

# 8. What Did Not Work / Limitations

- Few-shot prompting did **not** improve the measured accuracy in the classification experiment because the zero-shot task was already easy.
- One reasoning problem is not enough to establish a general accuracy improvement. A larger test set would provide stronger evidence.
- Prompt-only JSON reliability can still fail on highly ambiguous or adversarial inputs. Production systems should additionally validate the returned JSON against a schema.
- The experiments demonstrate prompt behavior, but they do not replace API-level structured-output enforcement when strict machine-readable output is required.

---

# 9. Conclusion

The experiments show that prompt engineering is about controlling model behavior rather than simply asking questions.

- **Zero-shot** is efficient for clear tasks.
- **Few-shot** provides examples that clarify expected behavior and output.
- **Reasoning-oriented prompts** can improve performance on multi-step tasks when intermediate checking is useful.
- **Structured prompts** with explicit schemas and constraints make LLM outputs much more reliable for downstream applications.

The most important lesson is that a good prompt defines the task, expected behavior, constraints, examples, and failure handling instead of relying on the model to infer them.

---

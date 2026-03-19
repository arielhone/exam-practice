---
name: pdf-to-exam-json
description: Convert a PDF exam file into the project's exam JSON format and register it in exams_list.json. Use when the user provides a PDF exam and wants it converted.
argument-hint: <path-to-pdf> <exam-title> [output-filename]
---

You are converting a PDF exam into this project's JSON format.

## Your task

1. **Read the PDF** at the path provided in the arguments.
2. **Extract all questions** and convert them to the JSON schema below.
3. **Write the JSON file** to `exams/<output-filename>.json` (derive filename from exam title if not given).
4. **Register the exam** by adding an entry to `exams_list.json`.

---

## Arguments

Parse the user's argument string:
- **Arg 1**: path to the PDF file
- **Arg 2**: exam title in English (e.g. `"Moed 13.3.2008"`) — used in `exams_list.json` and as the exam display name
- **Arg 3** *(optional)*: output filename without extension (e.g. `exam_13_3_2008`). If omitted, derive it from the title: lowercase, spaces→underscores, remove special chars.

---

## JSON Schema

The output file must be a **JSON array** of question objects. Every question follows this exact structure:

```json
{
  "id": "q1",
  "preamble": "...",
  "question_text": "...",
  "options": ["...", "...", "...", "..."],
  "correct_option_index": 0,
  "image_url": null
}
```

### Field rules

| Field | Type | Rules |
|-------|------|-------|
| `id` | string | `"q1"`, `"q2"`, ... sequential |
| `preamble` | string | Shared context printed before a group of questions. Empty string `""` if there is no preamble for this question. **Repeat** the same preamble text for every question in the group (do not reference a previous question). |
| `question_text` | string | The question itself. |
| `options` | array of 4 strings | Always exactly 4 answer choices. |
| `correct_option_index` | integer or null | 0-based index of the correct answer. Set to `null` if the PDF does not include an answer key. |
| `image_url` | string or null | `"NEEDS_IMAGE"` if the question references a figure/diagram that is in the PDF but cannot be extracted as text. `null` if no image is needed. |

### Math formatting

All mathematical expressions **must** be wrapped in `<span class="math-container">...</span>`:

- Inline math: `<span class="math-container">$E[X^2]$</span>`
- Display (block) math: `<span class="math-container">$$f(x) = \frac{1}{\sqrt{2\pi}}e^{-x^2/2}$$</span>`

Escape all backslashes as `\\` inside JSON strings (e.g. `\\frac`, `\\cos`, `\\int`).

Use `<br>` for line breaks within preamble/question text where needed.

### Example question

```json
{
  "id": "q4",
  "preamble": "נתונה מערכת מסוג AM המייצרת את המתח האקראי:<br><span class=\"math-container\">$$Y_{AM}(t) = [A_0 + X(t)]\\cos(\\omega_0 t + \\Theta_0)$$</span><br>כאשר האמפליטודה <span class=\"math-container\">$A_0$</span> והתדר <span class=\"math-container\">$\\omega_0$</span> קבועים.",
  "question_text": "חשב את פונקצית האוטוקורלציה של <span class=\"math-container\">$Y_{AM}(t)$</span>.",
  "options": [
    "<span class=\"math-container\">$R_{AM}(\\tau) = \\frac{1}{2}(A_0^2 - R_X(\\tau))\\cos(\\omega_0\\tau)$</span>",
    "<span class=\"math-container\">$R_{AM}(\\tau) = (A_0^2 + R_X(\\tau))\\cos(\\omega_0\\tau)$</span>",
    "<span class=\"math-container\">$R_{AM}(\\tau) = (2A_0^2 + R_X(\\tau))\\cos(\\omega_0\\tau)$</span>",
    "<span class=\"math-container\">$R_{AM}(\\tau) = \\frac{1}{2}(A_0^2 + R_X(\\tau))\\cos(\\omega_0\\tau)$</span>"
  ],
  "correct_option_index": 3,
  "image_url": null
}
```

---

## Preamble grouping

In Israeli engineering exams, several consecutive questions often share a common problem statement ("נתון...").

- Identify these groups by reading the PDF structure.
- Copy the **full preamble text** into each question in the group — do not leave it empty or refer back to a previous question.
- If a question stands alone with no shared context, set `preamble` to `""`.

---

## After writing the JSON

Read the current `exams_list.json`, append a new entry, and write it back:

```json
{
  "id": "<output-filename>",
  "title": "<exam-title-from-arg2>",
  "file_path": "exams/<output-filename>.json"
}
```

---

## Output to user

When done, report:
- The path of the created JSON file
- How many questions were extracted
- How many have `correct_option_index` set vs. `null`
- How many have `image_url: "NEEDS_IMAGE"` (images that still need to be added manually)

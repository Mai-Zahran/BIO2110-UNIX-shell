# Assignment 3 – Pipes & Filters for Biomedical Vitals Data

> **Type:** Graded individual assignment  
> **Suggested points:** 20 points  
> **Environment:** Google Cloud Shell (BIO2110-UNIX-shell repo)  
> **Related module:** Module 3 – Pipes & Filters

---

## 1. Purpose and learning outcomes

This assignment checks that you can:

- Use `cat`, `head`, `tail`, `wc`, `cut`, `sort`, and `uniq` on real CSV files.
- Combine these tools into **multi-step pipelines** that answer specific questions.
- Explain, in words, why each step in a pipeline is needed.
- Use AI tools (if you choose) in a transparent, verifiable way by keeping a short AI-use & verification log.

After completing this assignment, you should feel confident reading, writing, and debugging basic Unix pipelines for biomedical-style tabular data.

---

## 2. Files and environment

You will work in **Google Cloud Shell** using the BIO2110 UNIX repo.

- Cloud Shell: <https://shell.cloud.google.com>  
- Repo (browser view): <https://github.com/Mai-Zahran/BIO2110-UNIX-shell>

1. Open Cloud Shell and move into the Module 3 directory (if you have already cloned):

   ```bash
   cd BIO2110-UNIX-shell/contents/module3
   ```

   If you have not cloned the repo on this Cloud Shell account yet, run:

   ```bash
   git clone https://github.com/Mai-Zahran/BIO2110-UNIX-shell.git
   cd BIO2110-UNIX-shell/contents/module3
   ```

2. The starter data bundle for this assignment includes (names may vary slightly depending on the final release):

   - `vitals_small.csv` — small sample for debugging and testing.
   - `vitals_large.csv` — larger vitals dataset for final answers.
   - `lab_logs.txt` — simulated lab log entries (for one task).

3. Download the **Assignment 3 worksheet** (docx) from Brightspace/OpenLab. You will paste commands and write short answers in this document.

> **Important:** Do *not* edit any of the CSV files by hand. All work should be done using command-line tools.

---

## 3. Rules about collaboration and AI tools

- You may discuss general concepts with classmates (e.g., how `uniq -c` works), but the **commands and explanations you submit must be your own**.
- You may use AI tools (e.g., ChatGPT) to help you **draft** pipelines, but you must:
  - Write your own **prompts**,
  - Check the AI’s output carefully,
  - Document your use in a short **AI-use & verification log** (Task 4).
- You may not paste the entire assignment prompt into an AI tool and copy answers without understanding them. That is considered academic dishonesty in this course.

---

## 4. Tasks and grading

### Task 1 – Count and sanity check (4 pts)

Use `vitals_large.csv` in `contents/module3`.

1. Use `wc -l` to find:

   a. The total number of lines in `vitals_large.csv`.  
   b. The number of data rows (excluding the header).

2. Use `head` and `tail` to:

   - Show the header and first **3** data rows.
   - Show the last **3** data rows.

3. In your worksheet, record:

   - The commands you ran.
   - The numeric answers you obtained.
   - One short sentence per command explaining what you checked  
     (e.g., “This confirms that the file has exactly one header line and 50,000 data rows”).

> **Grading focus:** correct commands, correct numbers, and clear explanations.

---

### Task 2 – Unique tests and their frequencies (6 pts)

Again use `vitals_large.csv`.

#### 2a. Unique test names (2 pts)

Write a pipeline that prints all **unique values** of the `test` column (column #2), excluding the header, one per line, sorted alphabetically.

- Paste your final pipeline into the worksheet.
- In 2–3 sentences, explain how your pipeline:
  - Skips the header,
  - Selects the correct column,
  - Ensures uniqueness.

*(Tip: practice this first on `vitals_small.csv` to check your logic.)*

#### 2b. Test frequencies and top-5 tests (4 pts)

Write a pipeline that:

1. Counts how many times each test appears in `vitals_large.csv` (excluding the header), and  
2. Prints the **top 5** tests in descending order of count.

Your pipeline will likely include `tail -n +2`, `cut`, `sort`, `uniq -c`, `sort -nr`, and `head -n 5`.

In your worksheet, provide:

- The full pipeline.
- The **top-5 table** it prints (copy from your terminal).
- Short answers to the following questions:

  1. Which test is the most common, and how many times does it appear?  
  2. Why is `sort` placed **before** `uniq -c` in your pipeline?  
  3. Why do we use `sort -nr` at the end (what do `-n` and `-r` mean)?

> **Grading focus:** correctness of the pipeline, correct understanding of the order of operations, and clear explanation.

---

### Task 3 – Patient-level summary (6 pts)

Now summarize by `patient_id` (column #1).

1. Write a pipeline that prints:

   - Each unique `patient_id`, and
   - The number of records for that patient,

   ordered from most to least.

2. In your worksheet, include:

   - The pipeline.
   - The first **10** lines of its output (copy from your terminal).

3. Answer:

   - Which patient has the most records, and how many?  
   - Which patient has the second-most records, and how many?

4. In 2–3 sentences, explain at least **one** way a clinician or analyst could use this information  
   (for example, identifying high-risk patients or heavy service users, or checking for suspicious data duplication).

> **Grading focus:** correct aggregation logic and ability to connect the summary to a real biomedical informatics use case.

---

### Task 4 – AI prompt and verification log (4 pts)

This task checks that you can use AI tools *responsibly*, not that you use them at all. If you choose **not** to use AI for the rest of the assignment, you must still complete this small exercise with at least one AI interaction.

1. Choose **one** of your pipelines from Task 2 or Task 3.

2. Write a clear AI prompt that describes the problem and asks for a Unix pipeline. For example (you must write your own):

   > “I have a CSV file named `vitals_large.csv` with columns `patient_id,test,result,timestamp`. I need a single Unix pipeline that skips the header, groups by `test`, counts how many times each test appears, and prints the top 5 tests with counts in descending order. Please give just the command, and briefly explain each step.”

3. Run your prompt in an allowed AI tool and copy the AI’s suggested command into your worksheet.

4. Verify the AI command by running at least these checks in Cloud Shell:

   - Show the first few lines of the file (e.g., `head -n 3 vitals_large.csv`) and confirm the column order.
   - Run a **partial** pipeline (for example, up to `cut` or `uniq -c`) and inspect the output.
   - Run the full pipeline and compare it to your own solution and/or a small manual spot-check on `vitals_small.csv`.

5. In the worksheet, write a short **AI-use & verification log** (about half a page) including:

   - The exact prompt you used.
   - The exact command returned by the AI.
   - The specific commands you ran to verify it (copy/paste them).
   - Whether you needed to modify the AI’s command (if yes, how and why).
   - A 2–3 sentence reflection: What did this teach you about pipelines and AI tools?

> **Grading focus:** depth of verification, clarity of reflection, evidence that you understood and evaluated the AI output rather than copying it blindly.

---

## 5. Deliverables

Unless your instructor specifies otherwise, submit the following to Brightspace/OpenLab:

1. **Completed Assignment 3 worksheet** (docx or PDF) containing:
   - All commands requested.
   - All numerical answers.
   - Explanations and reflections.
   - The AI-use & verification log for Task 4.

2. (Optional, if required) **Screenshot** of your terminal showing the final pipeline and output for at least one task (for example, the top-5 tests or the patient summary).

Make sure your name and course information appear on the first page of the worksheet.

---

## 6. Rubric (suggested)

| Criterion                                      | Points |
|----------------------------------------------|--------|
| Task 1 – counts & sanity checks              | 4      |
| Task 2 – unique tests & frequencies          | 6      |
| Task 3 – patient-level summary & reflection  | 6      |
| Task 4 – AI prompt, verification & reflection| 4      |
| **Total**                                    | **20** |

Your instructor may adjust point values or add extra credit, but the **skills** assessed will remain the same.

---

## 7. Hints and common pitfalls

- Always **skip the header** before counting records or grouping by a column. A common mistake is to forget `tail -n +2`.
- `uniq` only removes or counts **adjacent** duplicates. Use `sort` before `uniq` when you want global counts.
- When something looks wrong, run only the **first half** of your pipeline and inspect the output. Most bugs are visible early.
- When using AI, never assume the answer is correct. Treat AI as a junior assistant whose work must be checked.

If you get stuck on a specific step, note which command is confusing and bring that question to lab or office hours.

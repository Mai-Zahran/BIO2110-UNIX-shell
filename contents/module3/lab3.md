# Lab 3 – Pipes & Filters Practice (UNIX Shell)

> **Status:** Ungraded practice lab (strongly recommended)  
> **Estimated time:** 60–75 minutes  
> **Environment:** Google Cloud Shell (real Bash), using the BIO2110 UNIX repo

---

## 1. Goals

By the end of this lab you should be able to:

- Navigate into a project directory using basic shell commands.
- Use `cat`, `head`, `tail`, `wc`, `cut`, `sort`, and `uniq` on real files.
- Combine these tools into pipelines that answer specific questions about vitals data.
- Capture and interpret the outputs of your commands.
- Write a short AI-use & verification note for one command.

This lab directly reinforces the concepts from the **Module 3 book chapter** and prepares you for **Assignment 3**.

---

## 2. Setup – Google Cloud Shell

For Lab 3 and future labs, we will use **Google Cloud Shell** for a “real Bash” experience that can preserve your files between sessions.

1. Open a browser and go to **Google Cloud Shell**: <https://shell.cloud.google.com>  
   Sign in with your City Tech / CUNY Google account if needed.

2. If you are prompted to start a new Cloud Shell session, accept the defaults.

2. When the terminal appears at the bottom of the page, check that you are in your home directory:

   ```bash
   pwd
   ```

3. Clone the BIO2110 Unix repo (you only need to do this **once**):

   ```bash
   git clone https://github.com/Mai-Zahran/BIO2110-UNIX-shell.git
   ```

   The repo is public here: <https://github.com/Mai-Zahran/BIO2110-UNIX-shell>


4. Move into the repository and into the Module 3 directory:

   ```bash
   cd BIO2110-UNIX-shell
   ls

   cd contents/module3
   ls
   ```

You should see files such as:

```text
vitals.csv
vitals_large.csv
lab3_notes.txt    (optional scratch file you can edit)
...
```

You will run **all** lab commands from inside `contents/module3`.

> **Tip:** If you return to Cloud Shell later, you do *not* need to clone again. Just run:
> ```bash
> cd BIO2110-UNIX-shell/contents/module3
> ```

---

## 3. Recording your work

For this practice lab, you do **not** need to upload anything, but we strongly recommend that you keep a record of:

- Commands you ran, and
- Short notes about what each one did.

You can:

- Use a local text editor, **or**
- Append notes to a simple log file in Cloud Shell, for example:

  ```bash
  nano lab3_log.txt
  ```

  or

  ```bash
  echo "Command: wc -l vitals_large.csv  # counts lines" >> lab3_log.txt
  ```

This habit will make graded assignments and future debugging much easier.

---

## 4. Tasks

### Task 1 – Sanity check the data

Use `vitals.csv` and `vitals_large.csv` in `contents/module3`.

1. Use `head` and `tail` to inspect the files.

   - Show the first 5 lines of each file.
   - Show the last 5 lines of each file.

   Example pattern:

   ```bash
   head -n 5 vitals_large.csv
   tail -n 5 vitals_large.csv
   ```

2. Use `wc -l` to count lines:

   - How many total lines are in `vitals.csv`?
   - How many total lines are in `vitals_large.csv`?
   - How many **data rows** (excluding the header) does each file have?

3. In your notes, write a short sentence for each command explaining what it checks  
   (for example: “This confirms there is exactly one header line.”).

> **Checkpoint:** You should now know how big each file is and what the columns look like.

---

### Task 2 – Unique test types

Now focus on the `test` column (column #2).

1. Write a pipeline that prints **all unique test names** in `vitals_large.csv`, one per line, sorted alphabetically. Remember to skip the header.

   - Start with something like:
     ```bash
     tail -n +2 vitals_large.csv | cut -d ',' -f 2 | sort | uniq
     ```
   - Adapt as needed and test in small steps.

2. Add a brief note to your log explaining:
   - How you skipped the header.
   - Why `sort` is used before `uniq`.

3. Optional stretch: Save the list of unique test names to a file called `unique_tests.txt` using redirection (`>`), then confirm its contents with `cat` or `less`.

---

### Task 3 – Test frequencies

Now we want to know how *often* each test appears.

1. Build a pipeline that shows **each test name and its count**, in any order.

   Hint: Use `tail -n +2`, `cut`, `sort`, `uniq -c`.

2. Extend your pipeline so that it prints only the **top 5** most frequent tests in descending order of count.

   Hint: Add `sort -nr` and `head -n 5`.

3. In your log, answer:

   - Which test is most common?
   - How many times does it appear?
   - Why do we use `sort -nr` at the end (what do `-n` and `-r` mean)?

> **Optional:** Redirect the top-5 table to a file called `test_frequencies_top5.txt` and check it with `cat` or `less`.

---

### Task 4 – Patient-level activity

Now summarize by `patient_id` (column #1).

1. Build a pipeline that prints **each patient ID** and the **number of records** for that patient, ordered from most to least.

   Hint: The pattern is very similar to Task 3, but using column 1 instead of column 2.

2. Answer in your log:

   - Which patient has the most records?
   - How many records do they have?

3. In 2–3 sentences, explain how a clinician or data analyst could use this summary  
   (for example, to identify high-utilization patients or check for data-entry issues).

---

### Task 5 – AI prompt and verification (mini practice)

This is a small, ungraded practice version of what you will do in **Assignment 3**.

1. Pick **one** of your pipelines from Task 2, 3, or 4.
2. Open an AI assistant (such as ChatGPT, if allowed by course policy) and write a **clear prompt** describing the problem.
   - Include:
     - File name,
     - Column names,
     - What output you want (e.g., “top 5 tests with counts, sorted by frequency”).

3. Paste the AI’s suggested command into your log (or write it down).

4. Verify the AI’s pipeline by running at least these checks:

   - Check the header and columns with `head -n 3`.
   - Run part of the pipeline (for example, up to `cut` or `uniq -c`) and inspect the output.
   - Run the full pipeline and see if the result matches what you expect from a manual spot-check.

5. In 2–3 sentences, write a mini AI-use & verification note:

   - Was the AI output correct the first time?
   - Did you need to fix anything (e.g., skipping the header, wrong delimiter, wrong column)?
   - What did you learn about pipelines from this?

> **Reminder:** You are practicing how to be the *scientist in charge* of the pipeline, not the person who just copies commands.

---

## 5. Wrapping up

When you are done:

- Review your `lab3_log.txt` (or your own notes).
- Make sure you can explain each major pipeline **in words**, not just as code.
- Keep your notes somewhere you can refer back to for Assignment 3 and quizzes.

This lab is for practice, but it is highly recommended. The patterns you build here will show up again and again in the rest of the course and in real biomedical data work.

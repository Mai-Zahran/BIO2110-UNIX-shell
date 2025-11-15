# Module 3 – Practice Quiz: Pipes & Filters

> **Status:** Practice only (not graded)  
> **Goal:** Help you self-check your understanding before the graded quiz.

You may complete this quiz as many times as you like. Try to answer questions **without** looking at your notes first, then check your answers.

---

## Part A – Multiple choice

**Q1.** What does the following command print?

```bash
tail -n +2 vitals.csv | wc -l
```

A. The total number of lines in `vitals.csv`, including the header.  
B. The total number of lines in `vitals.csv`, excluding the header.  
C. The number of characters in `vitals.csv`.  
D. The number of unique tests in `vitals.csv`.

---

**Q2.** Which pipeline correctly lists all **unique test names** (excluding the header) in alphabetical order?

A.
```bash
cat vitals.csv | cut -d ',' -f 2 | uniq
```

B.
```bash
tail -n +2 vitals.csv | cut -d ',' -f 2 | sort | uniq
```

C.
```bash
cut -d ',' -f 2 vitals.csv | sort -n | uniq
```

D.
```bash
tail -n +2 vitals.csv | sort | uniq -c
```

---

**Q3.** Why do we usually put `sort` **before** `uniq` when counting frequencies?

A. `uniq` cannot read from standard input.  
B. `sort` makes the output prettier for humans.  
C. `uniq` only removes or counts **adjacent** duplicates, and `sort` groups identical lines together.  
D. It does not matter; the order is arbitrary.

---

**Q4.** Consider the pipeline:

```bash
cat vitals_large.csv | head -n 50 | tail -n 10
```

What does this print?

A. The first 10 lines of `vitals_large.csv`.  
B. The last 10 lines of `vitals_large.csv`.  
C. Lines 41–50 of `vitals_large.csv`.  
D. Lines 10–20 of `vitals_large.csv`.

---

**Q5.** Which pipeline prints the **3 most common test names and their counts**?

A.
```bash
tail -n +2 vitals_large.csv \
  | cut -d ',' -f 2 \
  | uniq -c \
  | sort -nr \
  | head -n 3
```

B.
```bash
tail -n +2 vitals_large.csv \
  | cut -d ',' -f 2 \
  | sort \
  | uniq \
  | wc -l
```

C.
```bash
tail -n +2 vitals_large.csv \
  | cut -d ',' -f 2 \
  | sort \
  | uniq -c \
  | sort -nr \
  | head -n 3
```

D.
```bash
wc -l vitals_large.csv
```

---

**Q6.** What is the main effect of the `-n` flag to `sort`, as in `sort -n`?

A. Sorts by name of the file instead of its contents.  
B. Sorts lines in numeric order instead of alphabetical order.  
C. Sorts lines in reverse alphabetical order.  
D. Limits the output to *n* lines.

---

**Q7.** In a pipeline, what does the pipe symbol `|` do?

A. Redirects output to a file.  
B. Connects the standard output of one command to the standard input of the next.  
C. Comments out the rest of the line.  
D. Tells the shell to ignore errors.

---

## Part B – Short answer

**Q8.** Write a pipeline that prints all **unique patient IDs** in `vitals_large.csv` (excluding the header), one per line, sorted alphabetically.

*(Write your answer in the form of a shell command.)*

---

**Q9.** Write a pipeline that prints the number of **data rows** (excluding the header) in `vitals_large.csv`.

*(Write your answer as a shell command.)*

---

**Q10.** In 3–4 sentences, explain how you would verify that a pipeline using `tail -n +2`, `cut`, `sort`, `uniq -c`, and `sort -nr` is correctly answering the question:

> “What are the 5 most common lab tests in this file?”

Mention at least **two** concrete commands you would run as part of your verification.

---

## Answer key (fold/unfold)

Use this section only after you’ve attempted the questions.

<details>
<summary><strong>Show/Hide answers</strong></summary>

### Part A

**Q1.** B — It prints the number of data rows (all lines except the first/header line).

**Q2.** B — It skips the header with `tail -n +2`, selects column 2, sorts, then removes duplicates.

**Q3.** C — `uniq` only removes or counts *adjacent* duplicates; `sort` groups identical lines so `uniq` can work correctly.

**Q4.** C — `head -n 50` keeps the first 50 lines; `tail -n 10` of those prints lines 41–50.

**Q5.** C — Only this pipeline skips the header, extracts the test column, sorts, counts duplicates, sorts by count descending, and keeps the top 3.

**Q6.** B — It sorts lines according to numeric value instead of text order.

**Q7.** B — It connects the stdout of one command to the stdin of the next.

---

### Part B

**Q8.** One possible answer:

```bash
tail -n +2 vitals_large.csv \
  | cut -d ',' -f 1 \
  | sort \
  | uniq
```

**Q9.** One possible answer:

```bash
tail -n +2 vitals_large.csv | wc -l
```

**Q10.** Example reasoning (your wording can vary):  
First, I would run `head -n 3 vitals_large.csv` to confirm the header and column order, and verify that the test name is in the column I’m cutting. Then I would run only part of the pipeline, for example:

```bash
tail -n +2 vitals_large.csv | cut -d ',' -f 2 | head
```

to check that only test names appear and the header is gone. Next, I would run:

```bash
tail -n +2 vitals_large.csv \
  | cut -d ',' -f 2 \
  | sort \
  | uniq -c \
  | head
```

to see some raw counts and confirm they match manual spot checks on a small subset of the data. Only after those checks look reasonable would I trust the full pipeline with `sort -nr | head -n 5`.

</details>

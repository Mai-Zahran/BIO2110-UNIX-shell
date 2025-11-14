# Module 3 – Pipes & Filters: Building Power with UNIX Pipelines

> **How to use this chapter**
>
> - Read it like a **short textbook chapter** – not just slides.
> - Open the **interactive shell** in your browser (Binder) and **type along** with the examples.
> - Use the **AI & Prompt Testing** section near the end to practice working *with* AI, not letting it think for you.
> - Treat the examples as a **template library**: you will reuse these patterns all semester.

---

## Table of Contents

1. [Module 3 Learning Outcomes](#module-3-learning-outcomes)  
2. [Why Pipes & Filters Matter in Biomedical Informatics](#why-pipes--filters-matter-in-biomedical-informatics)  
3. [Getting Ready: Opening the Cloud Shell (Binder)](#getting-ready-opening-the-cloud-shell-binder)  
4. [Core Concept: Streams, Filters, and Pipes](#the-core-concept-streams-filters-and-pipes)  
5. [Our Running Example Dataset: `vitals.csv`](#our-running-example-dataset-vitalscsv)  
6. [Meet the Core Filters (with Examples)](#meet-the-core-filters-with-examples)  
7. [Combining Filters into Pipelines](#combining-filters-into-pipelines)  
8. [Debugging Pipelines: Think Like a Scientist](#debugging-pipelines-think-like-a-scientist)  
9. [Practice: Guided Exercises](#practice-guided-exercises)  
10. [Using AI as a Lab Partner (Not a Shortcut)](#using-ai-as-a-lab-partner-not-a-shortcut)  
    - [Case Study: ICU Alert Log & Distinct Alert Types per Patient](#case-study-icu-alert-log--distinct-alert-types-per-patient)  
    - [The Prompt → Test → Refine Cycle](#the-prompt--test--refine-cycle)  
    - [Your AI-Use Artifact](#your-ai-use-artifact)  
    - [Optional: Embedded AI Sandbox](#optional-embedded-ai-sandbox)  
11. [Summary & Checklist](#summary--checklist)  
12. [Attributions & Further Reading (OER)](#attributions--further-reading-oer)  

---

## Module 3 Learning Outcomes

By the end of this module, you will be able to:

1. Explain the “**pipes and filters**” design pattern in the UNIX shell.
2. Describe **standard input**, **standard output**, and **standard error**.
3. Use `cat`, `head`, `tail`, `cut`, `sort`, `uniq`, and `wc` as filters.
4. Construct multi-step pipelines with `|` that:
   - Select columns from CSV files.
   - Combine filters to summarize biomedical data.
   - Count and rank frequencies (e.g., most common lab test).
5. Use output redirection (`>`, `>>`) to save pipeline results to new files.
6. Debug and verify a pipeline using **small test cases** and inspection commands.
7. Draft and refine effective **AI prompts** that stay within a defined toolbox (Modules 1–3 only) and write a **verification log** showing how you checked the AI’s output.

---

## Why Pipes & Filters Matter in Biomedical Informatics

Imagine:

- You are given a **5 GB CSV file** named `all_patient_vitals.csv`.
- Your mentor asks two questions:
  1. *“How many patient records are in this file?”*
  2. *“What are the 10 most common lab tests recorded?”*

You cannot open this in Excel.  
You *could* write a Python script, but it might take:

- 10–15 minutes to write,
- More time to debug,
- Extra time to run if it loads the entire file into memory.

With the UNIX shell, you can often answer both questions in **under 30 seconds**, by chaining small commands together.

> **Key idea:**  
> The shell encourages **small, single-purpose tools** (filters) that you connect with **pipes** (`|`) so that the output of one becomes the input of the next. This “small pieces, loosely joined” idea is central in many Unix shell lessons (e.g., Software Carpentry’s “Pipes and Filters” and Data Carpentry’s shell genomics materials).

This module adapts those ideas to **Biomedical Informatics**, using examples like:

- Vitals data (`heart_rate`, `bp`, `temp`).
- ICU-style alerts.
- Simple summary tables.

---

## Getting Ready: Opening the Cloud Shell (Binder)

In this course, you will use a **real Bash shell in your browser** (no installation needed) via [myBinder](https://mybinder.org).

> ## Interactive Environment – Real Bash in Your Browser
>
> 1. Click this button to open the BIO2110 Unix course repo on Binder in a new tab:  
>    [![Launch BIO2110 Unix shell on Binder (opens in new tab)](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/Mai-Zahran/BIO2110-UNIX-shell/HEAD?urlpath=lab/tree/contents/module3)
>
> 2. Wait for **JupyterLab** to start (this may take 30–90 seconds the first time).
>
> 3. In the top menu, choose **File → New → Terminal**.  
>    A new tab with a shell prompt will open.
>
> 4. In most cases, your prompt will already show something like:
>    ```bash
>    jovyan@...:~/contents/module3$
>    ```
>    If you see that, you are already in the correct folder. Try:
>    ```bash
>    ls
>    cat vitals.csv
>    ```
>
> 5. If your prompt instead shows `~/` or `/home/jovyan`, first move into the module folder:
>    ```bash
>    cd contents/module3
>    ls
>    ```
>
> You should see a file named `vitals.csv` – we will use this in many examples in this chapter.

---

## The Core Concept: Streams, Filters, and Pipes

Before we learn specific commands, we need to talk about **streams**.

### Streams: stdin, stdout, stderr

Most shell programs interact with three streams:

- **Standard input (stdin)** – where data comes *in* (by default: the keyboard).
- **Standard output (stdout)** – where normal results go *out* (by default: the screen).
- **Standard error (stderr)** – where error messages go (by default: also the screen).

You can think of them as three labeled pipes connected to your program.

Many shell tools are **filters**:

- They **read from stdin**,  
- Transform the data,  
- **Write to stdout**.

Because they follow this pattern, the shell can **connect them together** using `|`.

### Filters

A **filter** is a small program that transforms a stream of text:

- `cat` – show the contents of a file (or multiple files).
- `head` – show the first lines.
- `tail` – show the last lines.
- `cut` – extract specific columns (fields).
- `sort` – sort lines.
- `uniq` – de-duplicate or count repeated lines.
- `wc` – count lines, words, or characters.

We will use all of these in increasingly complex ways.

### Pipes (`|`)

The **pipe** operator `|` connects the output of one command directly to the input of the next.

For example:

```bash
cut -d ',' -f 2 vitals.csv | sort | uniq -c
```

You can read this left to right as:

1. “Take column 2 from `vitals.csv`,”
2. “then sort the results,”
3. “then count how often each unique line appears.”

This pattern—**simple pieces composed into a larger pipeline**—is the heart of the “pipes and filters” approach.

---

## Our Running Example Dataset: `vitals.csv`

In your Binder shell, run:

```bash
ls
cat vitals.csv
```

You should see:

```text
patient_id,test,result
p001,heart_rate,72
p002,heart_rate,80
p001,temp,98.6
p003,heart_rate,65
p002,bp,120/80
```

Interpretation in a biomedical context:

- Each line after the header is a **single measurement**.
- `patient_id` is a de-identified code.
- `test` is the type of observation (e.g., heart rate, blood pressure).
- `result` is the measurement value itself.

We will use this file to practice pipelines before moving to larger real-world files.

---

## Meet the Core Filters (with Examples)

In this module we restrict ourselves to:

> `cat`, `head`, `tail`, `cut`, `sort`, `uniq`, `wc`, plus `>`, `>>`, and `|`

This keeps your mental “toolbox” focused. Regex, `grep`, `sed`, and `awk` arrive **later** in the course.

---

### 1. `cat` – Concatenate / Show Files

Think: **“dump the file contents.”**

```bash
cat vitals.csv
```

You should see the whole file. This is often used at the *start* of a pipeline, or to **quickly inspect** a small file.

> 🔎 **Practice:**
> - Create a new file `tests.txt`:
>   ```bash
>   echo "heart_rate" > tests.txt
>   echo "temp"       >> tests.txt
>   echo "bp"         >> tests.txt
>   ```
> - View it:
>   ```bash
>   cat tests.txt
>   ```

---

### 2. `head` – First Lines

Think: **“peek at the start.”**

By default, `head` shows the first 10 lines.

```bash
head vitals.csv
```

To show only the first 3 lines:

```bash
head -n 3 vitals.csv
```

> 🧠 **Why this matters in BMI:**  
> Real hospital logs can be millions of lines. `head` lets you verify the **file format**, **column order**, and **delimiter** quickly, without opening a giant file in Excel.

---

### 3. `tail` – Last Lines

Think: **“peek at the end.”**

```bash
tail vitals.csv
```

To show the last 2 lines:

```bash
tail -n 2 vitals.csv
```

A very useful variant to **skip the header row**:

```bash
tail -n +2 vitals.csv
```

- `-n +2` means: start at line 2 and show everything from there.

> 🔁 **Common pattern:**  
> You can combine `head` and `tail` to grab a slice of lines:
> ```bash
> head -n 110 big_log.txt | tail -n 10
> ```
> This shows lines 101–110 of `big_log.txt`.

---

### 4. `wc` – Count Lines, Words, Characters

Think: **“the accountant.”**

```bash
wc vitals.csv
```

You might see something like:

```text
      6      6     132 vitals.csv
```

This output means:

- 6 lines
- 6 words
- 132 characters (approximate; exact number isn’t important here)

You will almost always use the **`-l`** flag to count just lines:

```bash
wc -l vitals.csv
```

Output:

```text
6 vitals.csv
```

We know there is 1 header row, so there are **5 patient records**.

> 🧪 **Try it:**
> - Combine `tail` and `wc`:
>   ```bash
>   tail -n +2 vitals.csv | wc -l
>   ```
>   This should output `5` with no filename attached. Now you are counting data rows only.

---

### 5. `cut` – Extract Columns

Think: **“the scalpel.”**

`cut` is perfect for CSV and TSV files.

Basic usage:

- `-d ','` – set the delimiter to a comma.
- `-f 2` – select field (column) 2.

Example: extract the `test` column:

```bash
cut -d ',' -f 2 vitals.csv
```

Output:

```text
test
heart_rate
heart_rate
temp
heart_rate
bp
```

We often want to **skip the header** and only see data:

```bash
tail -n +2 vitals.csv | cut -d ',' -f 2
```

> 🧪 **Try it:**
> - Extract just the `result` column (field 3):
>   ```bash
>   tail -n +2 vitals.csv | cut -d ',' -f 3
>   ```

---

### 6. `sort` – Sort Lines

Think: **“the organizer.”**

```bash
tail -n +2 vitals.csv | cut -d ',' -f 2 | sort
```

Output (sorted test names):

```text
bp
heart_rate
heart_rate
heart_rate
temp
```

Useful flags:

- `sort -n` – numeric sort.
- `sort -r` – reverse order.
- `sort -nr` – numeric + reverse (largest to smallest).

> 🧪 **Try it:**
> - Sort `patient_id`s:
>   ```bash
>   tail -n +2 vitals.csv | cut -d ',' -f 1 | sort
>   ```

---

### 7. `uniq` – Remove or Count Duplicates

Think: **“the de-duplicator / counter.”**

`uniq` only removes **adjacent** duplicate lines. This is the “gotcha.”

Watch:

```bash
tail -n +2 vitals.csv | cut -d ',' -f 2 | uniq
```

Output:

```text
heart_rate
temp
heart_rate
bp
```

This is **wrong** if we wanted unique test names, because the two `heart_rate` entries are not next to each other in the input.

Correct pattern:

```bash
tail -n +2 vitals.csv | cut -d ',' -f 2 | sort | uniq
```

Output:

```text
bp
heart_rate
temp
```

To **count** how many times each unique line appears:

```bash
tail -n +2 vitals.csv | cut -d ',' -f 2 | sort | uniq -c
```

Example output:

```text
      1 bp
      3 heart_rate
      1 temp
```

> 💡 **Remember:**  
> `sort | uniq -c` is one of the **most important patterns** in data wrangling with the shell. It appears in many Carpentries shell lessons as a core building block.

---

## Combining Filters into Pipelines

Now let’s build multi-step pipelines that answer realistic questions, still using only:

> `cat`, `head`, `tail`, `cut`, `sort`, `uniq`, `wc`, `>`, `>>`, `|`

---

### Example 1 – How Many Unique Test Types?

**Question:** In `vitals.csv`, how many unique test types are present (excluding the header)?

Pipeline:

```bash
tail -n +2 vitals.csv | cut -d ',' -f 2 | sort | uniq | wc -l
```

Step by step:

1. `tail -n +2 vitals.csv`  
   → Skip the header, keep the 5 data rows.

2. `| cut -d ',' -f 2`  
   → Keep only the `test` column.

3. `| sort`  
   → Group identical test names together.

4. `| uniq`  
   → Keep one instance of each test name.

5. `| wc -l`  
   → Count how many unique names remain.

Expected output:

```text
3
```

So we have **3 test types**: `heart_rate`, `temp`, `bp`.

---

### Example 2 – Top 3 Most Common Test Types

**Question:** In a larger file `vitals_large.csv`, what are the **3 most common** test types?

(For practice, we’ll pretend `vitals.csv` is large.)

Pipeline:

```bash
tail -n +2 vitals.csv | cut -d ',' -f 2 | sort | uniq -c | sort -nr | head -n 3
```

Step by step:

1. `tail -n +2 vitals.csv`  
   → Remove header.

2. `| cut -d ',' -f 2`  
   → Select `test`.

3. `| sort`  
   → Group identical test names.

4. `| uniq -c`  
   → Count how many times each test appears.

5. `| sort -nr`  
   → Sort counts numerically, highest first.

6. `| head -n 3`  
   → Keep the top 3 lines.

Expected output:

```text
      3 heart_rate
      1 temp
      1 bp
```

---

### Example 3 – Selecting a Slice of a Larger File

**Scenario:** You have a long ICU log, and you want to inspect lines 101–120.

Pipeline pattern:

```bash
cat big_log.txt | head -n 120 | tail -n 20
```

- `head -n 120` keeps the first 120 lines.
- `tail -n 20` keeps the last 20 of *those* → original lines 101–120.

> 🧪 **Practice:**
> - If you had a 10,000-line lab result file, how would you view lines 501–550?  
>   *Hint:* Replace `120` and `20` with the appropriate numbers.

---

### Example 4 – Writing Pipeline Output to a File

Sometimes you want to **save** the result of a pipeline so you can use it later (or attach it to an email, or import it into another tool).

Use `>` (overwrite) or `>>` (append).

**Task:** Save the test frequency table to a new file called `test_counts.txt`.

```bash
tail -n +2 vitals.csv   | cut -d ',' -f 2   | sort   | uniq -c   | sort -nr > test_counts.txt
```

Now check:

```bash
cat test_counts.txt
```

You should see the same frequency table as before.

> ⚠️ **Be careful:**  
> `>` will overwrite the file if it already exists.  
> Use `>>` if you want to **append** to the file instead.

---

## Debugging Pipelines: Think Like a Scientist

When a pipeline doesn’t behave as expected, treat it like an experiment:

1. **Check assumptions.**
2. **Observe intermediate results.**
3. **Adjust and retest.**

### Strategy 1 – Print Intermediate Stages

Suppose you wrote:

```bash
tail -n +2 vitals.csv | cut -d ',' -f 2 | uniq -c
```

and got:

```text
      2 heart_rate
      1 temp
      1 heart_rate
      1 bp
```

This is suspicious. You forgot to `sort` before `uniq -c`.

Try adding one command at a time:

```bash
tail -n +2 vitals.csv
```

Then:

```bash
tail -n +2 vitals.csv | cut -d ',' -f 2
```

Then:

```bash
tail -n +2 vitals.csv | cut -d ',' -f 2 | sort
```

You’ll see exactly when the result becomes clean.

### Strategy 2 – Use Small Test Files

Instead of testing on a 5 GB file, create a tiny version:

```bash
head -n 20 big_file.csv > small_test.csv
```

Then work on `small_test.csv` until your pipeline looks correct.  
Once it works on the small file, you can apply it to the full dataset with more confidence.

---

## Practice: Guided Exercises

Try these directly in your Binder shell. Type your answers; then compare with the sample solutions.

Assume you are in `contents/module3` and `vitals.csv` exists.

---

### Exercise 1 – Number of Rows per Patient

**Goal:** Find how many measurements each patient has and list them from **most** to **least**.

1. Start by extracting the `patient_id` column (skip the header).
2. Use `sort` and `uniq -c`.
3. Sort by count (numerically, reverse).

**Write your pipeline below (in your notes) before running it.**

**Sample solution:**

```bash
tail -n +2 vitals.csv   | cut -d ',' -f 1   | sort   | uniq -c   | sort -nr
```

---

### Exercise 2 – Distinct Test Types per Patient

**Goal:** For each patient, count how many **different test types** they have (heart rate, temp, bp, etc.) and sort patients from most to fewest test types.

Hints:

1. Start from `patient_id,test` (columns 1 and 2).
2. De-duplicate `(patient_id,test)` pairs.
3. Count how many pairs each patient has.

**Sample solution:**

```bash
tail -n +2 vitals.csv   | cut -d ',' -f 1,2   | sort   | uniq   | cut -d ',' -f 1   | sort   | uniq -c   | sort -nr
```

---

### Exercise 3 – Lines 3–5 of `vitals.csv` (Data Only)

**Goal:** Show lines 3–5 of the data (excluding the header). That is, third through fifth **data rows**, not counting the header line.

1. Use `tail -n +2` to drop the header.  
2. Combine `head` and `tail` to take a slice from the data.

**Sample solution:**

```bash
tail -n +2 vitals.csv | head -n 5 | tail -n 3
```

---

### Exercise 4 – Save Unique Test Names to a File

**Goal:** Save the unique test names (excluding the header) to a file `test_names.txt`, one per line, sorted alphabetically.

**Sample solution:**

```bash
tail -n +2 vitals.csv   | cut -d ',' -f 2   | sort   | uniq > test_names.txt
```

Check:

```bash
cat test_names.txt
```

---

## Using AI as a Lab Partner (Not a Shortcut)

AI tools (like ChatGPT, Copilot, or other campus-provided assistants) are now everywhere. Some students use them to *avoid* thinking:

> “Just give me the pipeline so I can paste it and be done.”

In this course, we want the opposite:

> **Use AI to *amplify* your thinking, not replace it.**

For Module 3, that means:

- You stick to the **Module 1–3 toolbox**:
  - `cat`, `head`, `tail`, `cut`, `sort`, `uniq`, `wc`, `>`, `>>`, `|`
- You **explicitly say this** in your prompts.
- If AI suggests tools we haven’t learned yet (`grep`, `sed`, `awk`, etc.), you:
  - Mark that as **“not allowed (Module 4–5 content)”**,
  - Ask AI to try again with only allowed commands.

This section gives you a structured way to do that using a slightly more complex pipeline problem that is *not* obvious at first glance, but still solvable with our limited toolbox.

---

### Case Study: ICU Alert Log & Distinct Alert Types per Patient

Imagine you have an ICU alert log file named `icu_alerts.csv`:

```text
timestamp,patient_id,alert_type,value,unit,location
2024-04-10 01:15,p001,HR_HIGH,132,bpm,ICU-1
2024-04-10 01:17,p001,HR_NORMAL,98,bpm,ICU-1
2024-04-10 02:20,p002,BP_LOW,80/50,mmHg,ICU-2
2024-04-10 03:05,p003,HR_HIGH,145,bpm,ICU-3
2024-04-10 11:30,p002,HR_HIGH,130,bpm,ICU-2
2024-04-10 23:50,p004,HR_HIGH,140,bpm,ICU-1
2024-04-11 00:10,p004,BP_LOW,85/55,mmHg,ICU-1
2024-04-11 05:45,p001,HR_HIGH,135,bpm,ICU-1
2024-04-11 07:05,p001,HR_HIGH,120,bpm,ICU-1
```

In your Binder environment, create this file:

```bash
cat > icu_alerts.csv <<EOF
timestamp,patient_id,alert_type,value,unit,location
2024-04-10 01:15,p001,HR_HIGH,132,bpm,ICU-1
2024-04-10 01:17,p001,HR_NORMAL,98,bpm,ICU-1
2024-04-10 02:20,p002,BP_LOW,80/50,mmHg,ICU-2
2024-04-10 03:05,p003,HR_HIGH,145,bpm,ICU-3
2024-04-10 11:30,p002,HR_HIGH,130,bpm,ICU-2
2024-04-10 23:50,p004,HR_HIGH,140,bpm,ICU-1
2024-04-11 00:10,p004,BP_LOW,85/55,mmHg,ICU-1
2024-04-11 05:45,p001,HR_HIGH,135,bpm,ICU-1
2024-04-11 07:05,p001,HR_HIGH,120,bpm,ICU-1
EOF
```

#### The Task (Complex but Toolbox-Limited)

> Using only the commands from Modules 1–3 (`cat`, `head`, `tail`, `cut`, `sort`, `uniq`, `wc`, redirection, and pipes), write a **single pipeline** that:
>
> 1. Ignores the header.  
> 2. Treats each unique combination of `patient_id` and `alert_type` as a distinct “kind of problem”.  
> 3. Counts how many **different alert types** each patient has ever had.  
> 4. Sorts patients from **most** to **fewest** distinct alert types.

This is more complicated than just counting lines. You have to:

- Work with **two columns** together (`patient_id,alert_type`).
- Remove duplicated combinations.
- Then count per patient.

#### Step 1 – Your First Prompt

A good “Module 3-safe” prompt might be:

> “I have a CSV file `icu_alerts.csv` with columns `timestamp,patient_id,alert_type,value,unit,location`. I am only allowed to use these Unix commands: `cat`, `head`, `tail`, `cut`, `sort`, `uniq`, `wc`, `>`, `>>`, and pipes `|`.  
> I want a single pipeline that:  
> 1. ignores the header row,  
> 2. considers each unique `(patient_id,alert_type)` pair only once,  
> 3. counts how many different alert types each patient has had, and  
> 4. sorts patients from highest to lowest count.  
> Please explain each step briefly.”

You might have to **re-ask** if the AI suggests `grep`, `awk`, or `sed`. Part of the exercise is saying:

> “That’s outside our current toolbox. Please rewrite using only the commands listed.”

#### Step 2 – One Possible Solution (Within the Toolbox)

Here is a pipeline that satisfies the constraints:

```bash
tail -n +2 icu_alerts.csv   | cut -d ',' -f 2,3   | sort   | uniq   | cut -d ',' -f 1   | sort   | uniq -c   | sort -nr
```

Let’s unpack it:

1. `tail -n +2 icu_alerts.csv`  
   → Remove the header, keep data rows only.

2. `| cut -d ',' -f 2,3`  
   → Keep only `patient_id,alert_type`.

   Example output lines:
   ```text
   p001,HR_HIGH
   p001,HR_NORMAL
   p002,BP_LOW
   ...
   ```

3. `| sort`  
   → Sort by patient, then alert type. This groups identical pairs together.

4. `| uniq`  
   → Remove duplicate `(patient_id,alert_type)` pairs.  
     Now each combination occurs at most once.

5. `| cut -d ',' -f 1`  
   → Drop `alert_type`, keep only the patient IDs.  
     Each line now represents “one distinct alert type for this patient”.

6. `| sort`  
   → Group the same patient IDs together.

7. `| uniq -c`  
   → Count how many times each patient ID appears (i.e., how many distinct alert types they have).

8. `| sort -nr`  
   → Sort in numeric, reverse order (most types first).

Try running it and interpret the output.

#### Step 3 – Verification: Did It Work?

Following the AI transparency ideas from Module 1–2, you should keep a short **verification log**.

Example:

> **AI-Use & Verification Log (Module 3)**
>
> **1. Toolbox constraint I enforced**
>
> - Only allowed: `cat`, `head`, `tail`, `cut`, `sort`, `uniq`, `wc`, `>`, `>>`, and `|`.
> - I asked AI to avoid `grep`, `sed`, `awk`, and any other commands.
>
> **2. Final pipeline I used**
>
> ```bash
> tail -n +2 icu_alerts.csv >   | cut -d ',' -f 2,3 >   | sort >   | uniq >   | cut -d ',' -f 1 >   | sort >   | uniq -c >   | sort -nr
> ```
>
> **3. Verification steps**
>
> - I ran:
>   ```bash
>   tail -n +2 icu_alerts.csv | cut -d ',' -f 2,3
>   ```
>   to confirm that I was seeing `patient_id,alert_type` pairs.
>
> - Then I ran:
>   ```bash
>   tail -n +2 icu_alerts.csv >     | cut -d ',' -f 2,3 >     | sort >     | uniq
>   ```
>   to see the set of unique pairs and checked them against the original file.
>
> - Finally, I ran the full pipeline and manually checked one patient (e.g., `p001`):
>   - `p001` has `HR_HIGH` and `HR_NORMAL` alerts → 2 distinct types.
>   - The pipeline’s output showed a count of 2 next to `p001`, so the result is consistent.

The goal is not that you memorize this exact pipeline, but that you:

- Understand the **strategy**,
- Are able to **check** AI’s suggestions,
- Can say, with evidence, “this command actually did what I wanted.”

---

### The Prompt → Test → Refine Cycle

You can treat working with AI like running an experiment with a very talkative lab partner.

A simple cycle:

1. **Think first.**
   - What question am I trying to answer?
   - Which columns do I need?
   - What would a correct answer *look like* on a tiny test file?

2. **Set your constraints.**
   - “In this module, I can only use `cat`, `head`, `tail`, `cut`, `sort`, `uniq`, `wc`, `>`, `>>`, `|`.”
   - Put that in your prompt.

3. **Ask for a pipeline and an explanation.**
   - Ask AI to explain each step in one short sentence.

4. **Test on a tiny file.**
   - Use `head`, `tail`, `cut`, `sort`, `uniq`, `wc` to inspect intermediate output.
   - Compare results with your own manual reasoning.

5. **Refine the prompt.**
   - If AI uses disallowed commands, tell it why:
     - “We haven’t learned `grep` yet; please don’t use it.”
   - If the pipeline gives a wrong answer, describe what went wrong in plain language and ask for a revision.

6. **Stop when *you* are convinced.**
   - You should be able to explain every stage of the pipeline in your own words.

> ### Quick Checklist: Did I Use AI *Well* in Module 3?
>
> - [ ] I clearly listed the commands I’m allowed to use.  
> - [ ] I asked AI to **explain** the pipeline, not just print it.  
> - [ ] I tested AI’s suggestion on a small dataset.  
> - [ ] I inspected intermediate stages using our Module 3 commands.  
> - [ ] I kept a short **AI-use log** (prompt + pipeline + verification notes).

---

### Your AI-Use Artifact

For graded work later in the course, you may be asked to submit an **AI-use artifact** that includes:

1. **Your prompt(s)**  
   - Paste the prompt you used.
   - If you iterated, include the most important versions.

2. **The AI’s suggestion(s)**  
   - The pipeline AI gave you.
   - Any explanation it provided that you found helpful or confusing.

3. **Your final pipeline**  
   - The version you actually used to answer the question.
   - Highlight what you changed compared to the AI suggestion.

4. **Verification log** (3–8 sentences or bullet points)  
   - How you tested the pipeline.
   - What intermediate commands you ran.
   - How you know the result is correct.

You can reuse this template:

> **Prompt I used:**  
> *(paste here)*  
>
> **AI’s suggested pipeline:**  
> ```bash
> # paste here
> ```  
>
> **Issues I found:**  
> -  
> -  
>
> **My final pipeline (Module 3-safe):**  
> ```bash
> # paste here
> ```  
>
> **Verification steps:**  
> 1.  
> 2.  
> 3.  

---

### Optional: Embedded AI Sandbox

On the online version of this OER, your instructor may provide an **embedded AI assistant** so you can experiment with prompts *inside* the course site.

A generic embed (for instructors) might look like:

```html
<iframe
  src="https://example.com/your-ai-widget"
  title="BIO2110 AI Assistant – Prompt Practice"
  width="100%"
  height="500"
  style="border: 1px solid #ccc;"
>
  Your browser does not support iframes.
  You can open the AI assistant in a new tab instead:
  <a href="https://example.com/your-ai-widget">
    Open AI Assistant
  </a>
</iframe>
```

> **Accessibility note:**  
> - The `title` attribute helps screen readers identify this region.  
> - The fallback link inside provides an alternative if iframes are blocked.  
> - Your grade will be based on your **thinking and verification**, not on how fancy the AI’s wording is.

In this course, the embedded AI (if provided) is a **practice partner**, not an answer machine.

---

## Summary & Checklist

Before moving on, you should be comfortable with:

### Concepts

- Streams: **stdin**, **stdout**, **stderr**.
- Filters: small programs that transform data.
- Pipes: chaining filters with `|`.
- Redirection: `>` overwrites, `>>` appends.
- AI as a **tool**, with clear limits, not a replacement for your reasoning.

### Commands

- `cat`, `head`, `tail`
- `cut`, `sort`, `uniq`, `wc`
- Redirection: `>`, `>>`
- Command chaining with `|`

### Patterns

- Skip header:  
  `tail -n +2 file.csv`
- Frequency table:  
  `... | sort | uniq -c | sort -nr`
- Slice lines:  
  `head -n N | tail -n K`
- Save results:  
  `pipeline > results.txt`
- AI workflow (Module 1–3):  
  **Think → Constrain toolbox → Prompt → Test → Refine → Log**

> ### Module 3 Checklist
>
> - [ ] I can explain, in plain language, what a **pipe** and a **filter** are.  
> - [ ] I can use `cat`, `head`, `tail`, `cut`, `sort`, `uniq`, and `wc` on my own.  
> - [ ] I can build a pipeline to answer a simple question about `vitals.csv`.  
> - [ ] I can write a **frequency table** of test types and sort it by count.  
> - [ ] I can save pipeline output to a file and open it again.  
> - [ ] I can ask AI for help **without** letting it introduce tools we haven’t learned yet.  
> - [ ] I can write and verify a Module 3-safe AI-generated pipeline.

---

## Attributions & Further Reading (OER)

This chapter was inspired by, and loosely adapted from, several open educational resources:

- **The Unix Shell: Pipes and Filters – Software Carpentry**  
  CC BY 4.0. Classic introduction to pipes, filters, and redirection.  
  <https://swcarpentry.github.io/shell-novice/04-pipefilter.html>

- **Introduction to the Command Line for Genomics – Data Carpentry**  
  CC BY 4.0. Shell lessons designed for genomics data, including pipe-and-filter workflows on realistic biological datasets.  
  <https://datacarpentry.github.io/shell-genomics/>

- **Pipes and Filters – UCSB Carpentry / Library Carpentry**  
  CC BY 4.0. Exercises and explanations for combining commands with pipes in research and library contexts.  
  <https://carpentry.library.ucsb.edu/2021-08-09-ucsb-bash-online/04-pipefilter/index.html>

- **Scientific Computing for Chemists with Python – Charles J. Weiss**  
  CC BY-NC-SA. We borrow stylistic ideas from this book’s clear chapter structure and Jupyter/HTML integration for STEM learners.  
  <https://weisscharlesj.github.io/SciCompforChemists/>

All referenced OER are used under their respective **Creative Commons licenses**.  
This BIO2110 material is intended to be released by the instructor under a compatible Creative Commons license.

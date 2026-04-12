# Teaching Workflow — Detailed Procedure

This file defines the exact actions for each phase of the Java Tutor skill. Load this file when detailed guidance on a specific phase is needed.

---

## Phase 0 — Session Start / Resume

**Trigger:** Learner types `/teach` or the skill is invoked.

**Actions:**
1. Read `java-multiple-threads-concurrency/outline.md` — load the 14-module structure into context.
2. Check if `java-multiple-threads-concurrency/learner/process.md` exists:

   **File exists (returning learner):**
   - Read `learner/process.md` in full.
   - Read `learner/learner-level.md` — note language and style preference.
   - Greet in the learner's language:
     > "Welcome back! You're on **Module XX — [Title]**, section [X.Y]. Let's continue from where you left off."
   - If current module has a customized file (`learner/module-XX-customized.md`) → go to **Phase 3** at the saved section.
   - If current module does NOT have a customized file yet → go to **Phase 2**.

   **File does not exist (new learner):**
   - Greet:
     > "Welcome to the Java Multithreading & Concurrency course! Before we dive in, I'd like to ask a few quick questions to personalize the experience for you."
   - Go to **Phase 1**.

---

## Phase 1 — Level Evaluation *(first time only)*

**Trigger:** New learner (no `process.md` found).

**Actions:**

1. Read `java-multiple-threads-concurrency/learner-level-evaluating.md`.

2. Ask in **Round A** (background & knowledge) — present these together as a short friendly form:
   ```
   Quick setup — answer these however feels natural:
   1. What language do you prefer for explanations? (English / Vietnamese / other — just type it)
   2. Overall Java experience? (Just starting out / 1–2 years / 3+ years / Senior dev)
   3. Have you used threads in Java before? (Never / A little / Regularly)
   4. Have you ever hit a race condition or deadlock in a real project?
   5. Are you comfortable with `synchronized`? With `CompletableFuture`?
   ```

3. After learner answers Round A, ask **Round B** (goals & style):
   ```
   Almost done — two more:
   6. What's your main goal? (Job interview prep / Solving a real project problem / General curiosity / Other)
   7. What's your target domain? (Web backend / Data processing / Systems programming / Other)
   8. How do you prefer to learn? (Theory first, then code / Show me code first, then explain / Ask me questions to guide me)
   9. How much time can you spend per session? (~30 min / ~1 hour / Flexible)
   ```

4. After all answers, summarize the profile and confirm:
   > "Here's your profile: You're a [level] Java developer targeting [domain], preferring [style] in [language]. Your sessions are about [time]. I'll tailor the course accordingly. Does that sound right?"

5. Determine overall level:
   - **Beginner:** "Just starting out" or "Never used threads" → start from Module 1, full depth.
   - **Intermediate:** 1–2 years, some thread experience → fast-track Modules 1–3 option.
   - **Advanced:** 3+ years, used `synchronized`/`CompletableFuture` → skip to Module 4+ option.

6. Create `learner/learner-level.md` using the template in `assets/learner-level-template.md`. Fill in all fields from answers + inferred level.

7. Create `learner/process.md` using the template in `assets/process-template.md`. Set starting module.

8. If **Intermediate or Advanced**, ask:
   > "Modules 1–3 cover basics: what a thread is, lifecycle, sleep/join/interrupt. Based on your experience, would you like to: (a) fast-track them in a few minutes, (b) skip to Module 4 directly, or (c) go through them fully?"
   - Handle the choice: update starting module in `process.md` accordingly.

9. Go to **Phase 2** with the determined starting module.

---

## Phase 2 — Module Preparation *(before each new module)*

**Trigger:** About to start a new module (new learner after Phase 1, or completing a module and moving to next).

**Actions:**

1. Determine current module number from `process.md`.
2. Read `java-multiple-threads-concurrency/learner/learner-level.md` — note: `language`, `style`, `level`, `domain`, `strengths`, `gaps`.
3. Identify the original module folder: `java-multiple-threads-concurrency/module-XX-<name>/module-XX.md`. Read it.
4. Assess if module is below learner's level:
   - Modules 1–3 for Intermediate/Advanced learners → ask:
     > "Module XX ([Title]) looks foundational for your experience. Want to: (a) fast-track it, (b) skip it entirely, or (c) go through it fully?"
   - Handle choice accordingly and update `process.md`.
5. Generate `learner/module-XX-customized.md`:
   - Copy the module structure but adapt:
     - **Language:** all prose in learner's preferred language; all technical terms in English.
     - **Examples domain:** swap generic examples for learner's domain (e.g., web backend → HTTP request handler examples; data processing → batch job examples).
     - **Depth:** Beginner → more explanation per concept; Advanced → concise text, add "deeper dive" pointers.
     - **Style `theory-first` (default):** theory → sample → homework.
     - **Style `code-first`:** sample code → explanation → theory → homework.
     - **Style `socratic`:** replace each explanation with guiding questions that lead the learner to the answer.
   - Keep all code, class names, method names, and technical terms in English.
6. Update `process.md`:
   - Set `Current Module` = XX
   - Set `Current Section` = 1
   - Add row to Module History table with status "🔄 In Progress" and today's date.
7. Announce:
   > "Let's start **Module XX — [Title]**. [One-sentence preview of what they'll learn.] This module has [N] sections."
8. Go to **Phase 3**.

---

## Phase 3 — Teaching

**Trigger:** Customized module file is ready. Teaching starts at `Current Section` from `process.md`.

**Actions:**

1. Open `learner/module-XX-customized.md`. Start at the section saved in `process.md`.

2. For each section:
   a. Present the section content (prose + code blocks) in the learner's language.
   b. After presenting: pause and ask:
      > "Any questions on this before we move on? You can also type `/explain <topic>` if you want more detail on anything."
   c. Wait for learner response:
      - **Question/clarification asked** → answer fully, stay in the learner's language, keep technical terms in English. Then ask "Ready to continue?"
      - **`/explain <message>`** → deliver focused explanation on exactly what was asked, then ask "Ready to continue?"
      - **`/repeat`** → re-explain the current section using a different analogy, metaphor, or code example.
      - **"Yes / OK / ready / let's go"** → proceed to next section.
   d. After moving to the next section, update `process.md` → `Current Section` = current section number. (This ensures crash-safe progress.)

3. **Handling commands during teaching (any time):**

   | Command | Immediate action |
   |---------|-----------------|
   | `/progress` | Print `process.md` summary in a readable format. Ask "Ready to continue?" |
   | `/language <lang>` | Acknowledge. Update `learner-level.md` → `language`. Switch immediately. Re-state the last sentence in new language to confirm. Continue. |
   | `/style <style>` | Acknowledge. Update `learner-level.md` → `style`. Adapt the next section to new style. Continue. |
   | `/skip` | Ask: "Are you sure you want to skip Module XX? It covers [key topics]. I'll mark it as skipped." On confirm → update `process.md` (status = ⏭️ Skipped) → go to Phase 2 with next module. |
   | `/hint` | "Hints are for exercises! We're in the theory section — feel free to ask your question directly." |
   | `/explain <msg>` | Pause. Explain the topic. Ask "Ready to continue?" |

4. When all theory sections are done → tell learner "Theory done! Let's look at the code." → go to **Phase 4**.
   - Exception: Module 1 has no code exercise → go directly to **Phase 5**.

---

## Phase 4 — Code Exercises

**Trigger:** Theory sections of a module are complete and the module has code samples/exercises.

**Gradle Project Setup** *(one-time, create if `learner/code/` does not exist):*

Create the following files:

**`learner/code/settings.gradle`:**
```groovy
rootProject.name = 'java-concurrency-exercises'
```

**`learner/code/build.gradle`:**
```groovy
plugins {
    id 'java'
    id 'application'
}

repositories {
    mavenCentral()
}

java {
    sourceCompatibility = JavaVersion.VERSION_21
    targetCompatibility = JavaVersion.VERSION_21
}

application {
    mainClass = project.findProperty('mainClass') ?: 'module02.HelloThreads'
}
```

Then run in terminal to generate the Gradle wrapper:
```bash
cd java-multiple-threads-concurrency/learner/code
gradle wrapper --gradle-version 8.7
```
If `gradle` CLI is not available, create the wrapper files manually (gradlew script + gradle/wrapper/gradle-wrapper.properties).

**Actions for each module's code exercise:**

1. Create the source file: `learner/code/src/main/java/moduleXX/ClassName.java`
   - Use package: `package moduleXX;`
   - File name matches the class demonstrated in the customized module.
2. Show the code to learner with explanation.
3. Run via terminal:
   ```bash
   cd java-multiple-threads-concurrency/learner/code
   ./gradlew run -PmainClass=moduleXX.ClassName --quiet
   ```
4. Show the terminal output to learner. Explain what it demonstrates.
5. Give a modification challenge:
   > "Now try: [specific change]. Predict what will happen, then make the change and run it."
6. Wait for learner to modify the file and ask the AI to run it.
7. Run the modified file, show results, confirm understanding.
8. Go to **Phase 5**.

**Multiple source files in one module:**
- Name them descriptively: `BuggyCounter.java`, `FixedCounter.java`, `ProducerConsumer.java`.
- Run each one separately with `-PmainClass=moduleXX.FileName`.

---

## Phase 5 — Quiz & Homework

**Trigger:** Code exercises done (or Phase 3 complete for theory-only module 1).

**Actions:**

1. Announce: "Time for the module challenge!"

2. Present the homework from `learner/module-XX-customized.md` (homework section):
   - **Coding problem** (always first — developers prefer code):
     > "Here's your challenge: [problem statement]. Write your solution in `src/main/java/moduleXX/Homework.java`."
   - **Conceptual question** (optional, after coding):
     > "Quick question: [question]?"

3. Wait. **Do NOT volunteer hints.** Let learner attempt on their own.

4. If learner submits code or asks to run it:
   - Run it via terminal.
   - If correct output: praise and explain why it works.
   - If wrong output: give targeted feedback:
     > "The output shows [X]. Think about what happens when [Y]... Try again!"
   - Do not reveal the full answer — guide toward it.

5. If learner types `/hint`:
   - Give ONE specific, directional hint (not the answer).
   - Keep a mental count of hints for this exercise. After 3 hints, offer to walk through it together.

6. On satisfactory completion:
   - Briefly recap: "Great! In Module XX you learned: (1) [concept], (2) [concept], (3) [concept]."
   - Update `process.md`:
     - Module XX row → status = ✅ Completed, date = today, add any notes.
     - Increment Overall Progress counter.
     - Clear `Current Section`.
   - Update `learner-level.md` if the session revealed new information:
     - Learner struggled with X → add to `Gaps`.
     - Learner grasped Y quickly → add to `Strengths`.
   - Preview next module:
     > "Module XX+1 is **[Title]** — [one-sentence preview]. Ready to continue?"
   - On "yes" → go to **Phase 2** with Module XX+1.
   - On "not now" → say "No problem! Type `/teach` to resume anytime." Save current state in `process.md` and end session.

---

## process.md Update Rules

- **Update after every section taught** (Phase 3): change `Current Section` — never let it lag.
- **Update on module start** (Phase 2): add module row with "🔄 In Progress".
- **Update on module complete** (Phase 5): change status to "✅ Completed", add date.
- **Update on module skip**: status = "⏭️ Skipped".
- **Update on language/style change**: update the Status block in `process.md` to note the change.

## learner-level.md Update Rules

- **Update after Phase 1**: full creation from template.
- **Update on `/language` command**: change `language` field immediately.
- **Update on `/style` command**: change `teaching-style` field immediately.
- **Update after Phase 5**: append to `Strengths` or `Gaps` if new evidence found.
- **Do not rewrite the file** — use targeted updates to individual fields.

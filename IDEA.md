# AI Course Indexer & Streamlined Player Blueprint

## 1. Purpose & Core Philosophy
You are an **Expert Technical Curriculum Architect and Frontend Engineer**. When given a link to any online course (Udemy, Coursera, edX, YouTube playlist, etc.), your job is to:
1. Extract the complete syllabus (every module and video lecture title with duration).
2. Audit the curriculum to eliminate bloat, repetitive setup fluff, and deprecated code—**without ever breaking the learning continuity**.
3. Generate **`index.md`**: A structured audit and re-ordered chronological learning path.
4. Generate **`index.html`**: A standalone, zero-dependency, **Gemini Obsidian Black** interactive course launcher and progress tracker with direct deep-linking to the course player.

---

## 2. The Golden Rule: "Never Break the Learning Chain" (Continuity Guardrail)
When auditing technical courses that have been updated over time, instructors often append "v1 / v2 Update" modules later in the course while leaving older modules in place. Follow these strict rules when deciding what to keep vs. skip:

1. **Outdated AND Replaced $\rightarrow$ `🚫 [SKIP - OUTDATED]`**
   - *Rule:* Only mark an outdated video/module as `[SKIP]` if a newer module in the same course teaches the modern replacement.
   - *Case Study Example:* In the *Complete Agentic AI Bootcamp*, Module 8 teaches legacy `RunnableWithMessageHistory`, which is completely replaced by Module 10 (LangChain v1) and Module 12 (LangGraph Checkpointers). Therefore, Module 8 is safe to skip.
2. **Outdated Rosette / Prerequisite Bridge (NOT Replaced) $\rightarrow$ `⚠️ [OLD BUT REQUIRED]`**
   - *Rule:* If a video uses older syntax or an older library version, **but no newer video in the course replaces that foundational concept** and subsequent modules depend on it, **DO NOT SKIP IT**.
   - *Action:* Keep it in the active "Must Watch" path, tag it with **`⚠️ [OLD BUT REQUIRED FOR NEXT SECTION]`**, and add a concise 1-line note explaining *what* is legacy and *why* it is still required so the learner's continuity is never broken.
   - *Case Study Example:* Module 5 teaches Document Loaders, Text Splitters, FAISS, and ChromaDB using older LangChain v0.x imports, and Module 7 teaches the LCEL `|` pipe operator. Even though LangChain v1 is introduced in Module 10, the course never re-teaches Vector Stores or LCEL pipes in Module 10—and Module 16 (Agentic RAG) assumes you know them. Thus, those core lectures are kept in the active path with a continuity note.
3. **Repetitive Boilerplate & Fluff $\rightarrow$ `🚫 [SKIP - BOILERPLATE]`**
   - *Rule:* If a course repeats virtual environment creation (`conda create`, `uv init`), API key `.env` setup, basic Python syntax, or identical Streamlit UI shells across 4+ modules, keep **only the first modern setup video** and skip all subsequent duplicates.

---

## 3. End-to-End Execution Pipeline

### Phase 1: Generate `index.md` (The Curriculum Audit & Streamlined Path)
When the user provides a course URL:

#### Step 1.1: Full Curriculum Extraction
- Fetch the course syllabus. If the main platform URL is paywalled (e.g., Udemy), search public mirrors, Class Central, course aggregators, or student GitHub repositories for that exact course title to retrieve all Module numbers, Lecture titles, and Video durations.
- Index **100% of the modules and sub-modules** in their original order first so the user has a complete reference map.

#### Step 1.2: Assign Precision Status Badges
Tag every lecture in Step 1 with one of five badges:
- `🔥 [MUST WATCH]` — Core modern concepts, architecture, and primary code implementations.
- `⚠️ [OLD BUT REQUIRED]` — Uses older syntax/version, but is **not replaced** elsewhere in the course and is required to understand the next section. Always include a short note: *(Note: Old vX syntax, but required for [Next Topic])*.
- `⚡ [2x SPEED]` — Good conceptual overview, secondary tool demo, or UI wiring worth skimming at 2x speed.
- `🚫 [SKIP - OUTDATED]` — Deprecated APIs/patterns that are explicitly superseded by a newer module in the course.
- `🚫 [SKIP - FLUFF/REDUNDANT]` — Basic language prerequisites, duplicate IDE/environment setups, or repetitive project intros.

#### Step 1.3: Chronological Re-Ordering (The 5-Section Streamlined Path)
- Re-organize the active lectures (`🔥 MUST WATCH`, `⚠️ OLD BUT REQUIRED`, and `⚡ 2x SPEED`) into **4 to 6 logical Sections** with clean subsections (`1.1`, `1.2`, etc.).
- **Crucial Ordering Rule:** If the instructor placed a modern version update in the middle of the course (e.g., Module 10: LangChain v1 Updates), move it to **Section 1** *before* the foundational RAG/Tool modules so the student learns modern syntax first.

---

### Phase 2: Generate `index.html` (The Interactive Course Launcher)
When the user asks to convert `index.md` into `index.html` (or asks for both upfront), generate a single, self-contained `index.html` file adhering to these exact architectural requirements:

#### 1. Data Structure & Strict Alignment with `index.md`
Each video object in the `CURRICULUM` JavaScript array must include:
```javascript
{
  mod: 10, // Exact original course Module/Unit number (1..N) for collision-free link matching
  title: "Understanding Retrievers And Chains",
  duration: 21, // Duration in minutes
  pace: "legacy", // "must" | "legacy" | "skim" | "skip"
  note: "Old v0.x chain syntax, but required context for LCEL & Agentic RAG" // Required for "legacy" and "skip"
}
```

#### 2. Dynamic Filter Pills & Top Bar Behavior
- **Filter Pills Order & Default State:**
  1. `🔥 Must Watch` (**Default active view on load** — displays both `pace === "must"` AND `pace === "legacy"` so required bridge lectures are never missed).
  2. `⚡ 2x Speed` (Displays `pace === "skim"`).
  3. `All Active` (Displays `must` + `legacy` + `skim`).
  4. `🚫 Skipped` (Displays `pace === "skip"` with reasons why each lecture was cut).
- **100% Dynamic Top Bar:**
  - Switching filter pills must immediately recalculate:
    - **Top Progress Text:** `X / Y [Filter Name] Completed (Z%)`
    - **Top Progress Bar Fill:** `width: Z%`
    - **Remaining Time Display:** Exact hours and minutes left in the active filter (for `⚡ 2x Speed`, show both raw duration and actual watch time at `2x` speed).
    - **Section & Subsection Counters:** `Done / Total • Xh Ym left` per section for the active filter.
    - **Up Next Queue Banner:** Points to the next uncompleted video inside the currently active filter.
    - **Sequential Numbering:** Dynamically numbers visible rows `1, 2, 3...` without exposing complex underlying module numbers.

#### 3. One-Time Udemy / LMS Deep-Link Sync (Self-Replacing Setup UI)
Udemy's internal `/learn/lecture/<ID>` URLs require the numeric Course ID and subscriber curriculum JSON. Build a **Self-Replacing Setup View** (`#setupView`) that appears on first launch and automatically replaces itself with the main index (`#appView`) once pasted:
- **Step 1 Button (`Open Course ID Page ↗`):**
  Opens `[https://www.udemy.com/api-2.0/courses/](https://www.udemy.com/api-2.0/courses/)<COURSE-SLUG>/?fields[course]=id,title` in a new tab.
- **Step 2 Button (`Open Curriculum JSON ↗`):**
  When the user pastes the Step 1 JSON (`{"_class":"course","id":1234567}`), automatically extract `id`, unlock Step 2, and open:
  `[https://www.udemy.com/api-2.0/courses/](https://www.udemy.com/api-2.0/courses/)<COURSE_ID>/cached-subscriber-curriculum-items/?page_size=1000&fields[lecture]=title,id&fields[chapter]=title,id`
- **Module-Scoped Lecture Matcher (Prevents Chapter/Duplicate Collisions):**
  - Udemy's JSON contains both `_class: "chapter"` and `_class: "lecture"`.
  - **CRITICAL:** Never match `_class === "chapter"`. Increment a `currentMod` counter on each `chapter`, group only `_class === "lecture"` items by `currentMod`, and match each video inside its exact `video.mod` bucket first. This guarantees duplicate lecture titles across modules (e.g., *"Introduction To The Project"*) always link to the right video.

#### 4. `file://` Safe Storage & Cross-Browser Portable HTML Export
- **SafeStorage Wrapper:** Wrap `localStorage` in `try/catch` with an in-memory fallback so opening `file:///C:/.../index.html` in strict browsers never throws a `SecurityError` or halts rendering.
- **Portable State Tag (`<script id="portable-state" type="application/json">`):**
  Include a `💾 Save Portable HTML` button in the top bar that serializes `{ configured, courseId, progress, ids }` into `#portable-state` and downloads an updated `index.html`. Re-opening that file in any browser restores all links and completion states immediately.
- **Reset Progress Button:** Clears completion checkboxes after a `window.confirm()` prompt while preserving synced Udemy lecture IDs.

#### 5. Gemini Obsidian Black UI Specification
- **Color Palette:**
  - Canvas Background: `#090a0f`
  - Card Surface: `#13151c`
  - Elevated Header: `#1a1d26`
  - Glass Sticky Bar: `rgba(13, 15, 20, 0.9)` with `backdrop-filter: blur(18px)`
  - Gemini Gradient Accent: `linear-gradient(135deg, #4285f4 0%, #9b72cb 52%, #d96570 100%)`
- **Visual Pace Badges:**
  - `🔥 MUST WATCH` — Blue pill (`rgba(138, 180, 248, 0.11)`)
  - `⚠️ OLD • REQUIRED FOR NEXT SECTION` — Amber/Orange bridge pill (`rgba(249, 115, 22, 0.14)`, border `#fb923c`, text `#fdba74`) with tooltip/label showing what it unlocks.
  - `⚡ 2x SPEED` — Gold pill (`rgba(251, 191, 36, 0.11)`)
  - `🚫 SKIP` — Rose pill (`rgba(244, 63, 94, 0.1)`) showing the skip reason.

---

## 4. Reference Implementation Template (`index.html` Skeleton)
Use this exact logic for the `⚠️ [OLD BUT REQUIRED]` badge and filter handling in `index.html`:

```javascript
// Filter inclusion logic: "must" view includes both "must" AND "legacy" (Old but Required)
function isVideoInFilter(video, filterMode) {
  if (filterMode === "must") return video.pace === "must" || video.pace === "legacy";
  if (filterMode === "skim") return video.pace === "skim";
  if (filterMode === "all")  return video.pace !== "skip";
  if (filterMode === "skip") return video.pace === "skip";
  return video.pace === "must" || video.pace === "legacy";
}

// Badge generator with continuity support
function createBadgeEl(video) {
  const span = document.createElement("span");
  if (video.pace === "must") {
    span.className = "pace-badge pace-must";
    span.innerHTML = '🔥 <span class="label-text">MUST WATCH</span>';
  } else if (video.pace === "legacy") {
    span.className = "pace-badge pace-legacy";
    span.title = video.note || "Outdated syntax, but required for the next section";
    span.innerHTML = '⚠️ <span class="label-text">' + (video.note || "OLD • REQUIRED FOR NEXT SECTION") + '</span>';
  } else if (video.pace === "skim") {
    span.className = "pace-badge pace-skim";
    span.innerHTML = '⚡ <span class="label-text">2x SPEED</span>';
  } else {
    span.className = "pace-badge pace-skip";
    span.innerHTML = '🚫 <span class="label-text">' + (video.note || "SKIP") + '</span>';
  }
  return span;
}
```

---

## 5. How to Use This Blueprint in Future Chats
Copy and paste the prompt below whenever you start a new course:

> **Prompt:**
> "Follow the specification in `AICourseIndexer.md` for this course: `[INSERT COURSE URL]`.
> 1. First, generate `index.md` (Step 1: Full Module/Video Index with badges, Step 2: Audit of what is outdated vs. `⚠️ OLD BUT REQUIRED FOR NEXT SECTION` so no learning link is broken, Step 3: Streamlined 5-Section Path).
> 2. Next, generate the complete, self-contained `index.html` following the Gemini Obsidian UI, 2-step Udemy JSON link sync, module-scoped `_class === 'lecture'` matcher, and dynamic `🔥 Must Watch` default filter bar."

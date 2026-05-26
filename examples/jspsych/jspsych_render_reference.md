© 2026 SRP Solutions SRP LLC. All rights reserved.

# SRP Survey DSL Language Reference — jsPsych Edition

Syntax specification for the SRP Survey DSL **as it applies to jsPsych experiment export**. This document covers features and limitations specific to jsPsych rendering. For the complete DSL reference, see `language_reference.md`.

---

## Table of Contents

1. [Overview](#1-overview)
2. [Use Cases and Capabilities](#2-use-cases-and-capabilities)
3. [CLI Usage](#3-cli-usage)
4. [Question Type Support Matrix](#4-question-type-support-matrix)
5. [Output Format](#5-output-format)
6. [Supported Question Attributes](#6-supported-question-attributes)
7. [Limitations and Unsupported Features](#7-limitations-and-unsupported-features)
8. [Output Structure and Examples](#8-output-structure-and-examples)

---

## 1. Overview

The **jsPsych renderer** converts SRP surveys into browser-based behavioral experiments using the jsPsych framework. It generates a self-contained HTML file that runs entirely in the browser with no build step required.

### Key Characteristics

| Aspect | Details |
|--------|---------|
| **Output** | Single standalone HTML file (4–8 KB typical) |
| **Runtime** | Browser-based, no server required for execution |
| **Dependencies** | jsPsych 8.2.3 (loaded via CDN: unpkg.com) |
| **Data Collection** | Auto-captured and exported as JSON or CSV |
| **Response Timing** | Millisecond precision on all questions |
| **Randomization** | Supported via jsPsych timeline configuration |

### Invocation

```bash
bin/survey_srp survey.srp --jspsych -o output.html
bin/survey_srp survey.srp --jspsych > output.html  # stdout
bin/survey_srp survey.srp --jspsych               # auto-names survey_jspsych.html
```

---

## 2. Use Cases and Capabilities

### Primary Use Cases

1. **Academic Research** — Surveys with response timing, randomization, and attention checks.
2. **UX Research** — Collect timing data on interface feedback and decision-making.
3. **Behavioral Science** — Factorial designs, adaptive flows, and conditional branching.
4. **Online Experiments** — Deploy without server infrastructure (responses logged locally).

### Built-in Features

- **Welcome Trial** — Displays survey title; respondents press any key to begin.
- **Multi-Question Pages** — All questions from an SRP page rendered in a single jsPsych trial (using SurveyJS plugin).
- **Auto-Advance Trials** — Respondents proceed page-to-page automatically after answering all required questions.
- **Completion Screen** — Final screen with options to download responses as JSON or CSV.
- **Browser Console Logging** — All response data also logged to `console.log()` and stored in `window.responseData`.
- **No Server Required** — Experiments run entirely client-side; optional POST to endpoint if configured.

---

## 3. CLI Usage

### Basic Syntax

```bash
bin/survey_srp <input.srp> --jspsych [options]
```

### Options

| Flag | Argument | Description |
|------|----------|-------------|
| `--jspsych` | None | Enable jsPsych rendering |
| `-o`, `--outfile` | `FILE` | Write HTML to specified file |

### Examples

```bash
# Write to auto-generated filename (survey_jspsych.html)
bin/survey_srp examples/survey.srp --jspsych

# Specify output file
bin/survey_srp examples/survey.srp --jspsych -o experiments/my_survey.html

# Output to stdout (pipe to file or browser)
bin/survey_srp examples/survey.srp --jspsych > survey.html

# Read from stdin
cat survey.srp | bin/survey_srp --jspsych -o output.html
```

---

## 4. Question Type Support Matrix

### Supported Question Types

| SRP Type | jsPsych Mapping | Status | Notes |
|----------|-----------------|--------|-------|
| OpenEnded | `text` element | ✅ Supported | Single-line text input |
| Discussion | `comment` element | ✅ Supported | Multi-line textarea (6 rows) |
| Number | `text` with `inputType: 'number'` | ✅ Supported | Numeric input with validation |
| SingleSelect | `radiogroup` element | ✅ Supported | Radio button options |
| MultiSelect | `checkbox` element | ✅ Supported | Checkbox options |
| Dropdown | `dropdown` element | ✅ Supported | Drop-down select |
| ButtonCheckbox | `checkbox` element | ✅ Supported | Checkbox options (button style) |
| ThisOrThat | `radiogroup` (2 choices) | ✅ Supported | Binary choice (first 2 options used) |
| Ranking | `ranking` element | ✅ Supported | Drag-and-drop item ranking |
| Rating | `rating` element | ✅ Supported | 1–5 star/numeric scale (customizable) |
| NPS | `rating` (0–10 scale) | ✅ Supported | Net Promoter Score (0–10) |
| Slider | `sliders` element | ✅ Supported | Range slider (configurable min/max) |
| SingleSelectMatrix | `matrix` element | ✅ Supported | Radio buttons per row |
| MultiSelectMatrix | `matrixdropdown` element | ✅ Supported | Checkboxes per row |
| RatingMatrix | `matrix` element | ✅ Supported | Rating scale per row |
| SingleSelectBipolar | `matrix` (2 columns) | ✅ Supported | Agree/Disagree scale per item |
| SingleSelectBipolarMatrix | `matrix` (2 columns) | ✅ Supported | Bipolar matrix |
| FillInTheBlank | `text` with placeholder | ✅ Supported | Single-line text with hint |
| DatePicker | `text` with `inputType: 'date'` | ✅ Supported | HTML5 date input |
| Notification | `html` element | ✅ Supported | Display-only message (no input) |
| HeatMap | `html` placeholder | ⚠️ Unsupported | Renders as "not yet supported" message |
| TimedHeatMap | `html` placeholder | ⚠️ Unsupported | Renders as "not yet supported" message |
| StickyNote | `html` placeholder | ⚠️ Unsupported | Renders as "not yet supported" message |
| CardSort | `html` placeholder | ⚠️ Unsupported | Renders as "not yet supported" message |
| CardRating | `html` placeholder | ⚠️ Unsupported | Renders as "not yet supported" message |
| MaxDiff | `html` placeholder | ⚠️ Unsupported | Renders as "not yet supported" message |
| ConstantSum | `html` placeholder | ⚠️ Unsupported | Renders as "not yet supported" message |
| TextHighlighter | `html` placeholder | ⚠️ Unsupported | Renders as "not yet supported" message |
| Autosuggest | `html` placeholder | ⚠️ Unsupported | Renders as "not yet supported" message |
| ConjointChoice | `html` placeholder | ⚠️ Unsupported | Renders as "not yet supported" message |
| EmotionSelector | `html` placeholder | ⚠️ Unsupported | Renders as "not yet supported" message |
| MediaUpload | `html` placeholder | ⚠️ Unsupported | Renders as "not yet supported" message |
| USAddress, InternationalAddress, Age, PhoneNumber | `html` placeholder | ⚠️ Unsupported | Renders as "not yet supported" message |

### Unsupported Question Types

When an unsupported question type is encountered, the renderer generates an HTML placeholder:

```html
<div style="padding: 16px; background: #fef3c7; border-left: 4px solid #f59e0b; border-radius: 4px; color: #78350f;">
  <strong>QuestionTypeName Question Type</strong><br/>
  This question type is not yet supported in jsPsych export.
</div>
```

The experiment continues normally; respondents see the placeholder but cannot enter a response for that item.

---

## 5. Output Format

### HTML Structure

The generated file is a complete, valid HTML5 document with:

- **DOCTYPE and meta tags** — Standard HTML5 boilerplate with viewport settings.
- **External CDN links** — jsPsych, plugin libraries, and CSS stylesheet (unpkg.com CDN).
- **Single `<body>` with `<script>` block** — All JavaScript inline; no external JS files.
- **Minimal CSS** — Essential styles for layout (responsive container, completion message styling).

### File Size

Typical survey output: **4–8 KB** (plain HTML + JSON timeline, no compression).

### CDN Dependencies

```html
<script src="https://unpkg.com/jspsych@8.2.3/dist/jspsych.js"></script>
<script src="https://unpkg.com/@jspsych/plugin-survey@2.0.0/index.browser.min.js"></script>
<script src="https://unpkg.com/jspsych@8.2.3/dist/plugins/plugin-html-keyboard-response.js"></script>
<link href="https://unpkg.com/jspsych@8.2.3/css/jspsych.css" rel="stylesheet" />
```

**Note:** Internet connectivity is required at runtime for CDN resources to load. For offline use, host jsPsych libraries locally and modify the HTML CDN URLs.

---

## 6. Supported Question Attributes

### Common Attributes Recognized

Most common SRP question attributes are recognized but not all affect the jsPsych output.

| Attribute | Type | Usage in jsPsych | Notes |
|-----------|------|------------------|-------|
| `text` | String | ✅ Rendered as element title | Question prompt |
| `required` | Boolean | ✅ Applied to all elements | Enforced by jsPsych survey plugin |
| `hidden` | Boolean | ✅ Filtered at render time | Hidden questions excluded from timeline |
| `randomize_items` | Boolean | ⚠️ Ignored | jsPsych randomization handled client-side (future feature) |
| `add_why_field` | Boolean | ⚠️ Ignored | "Why" text fields not rendered |
| `min_value`, `max_value` | Integer | ✅ Used for Rating/Slider | Sets scale bounds |
| `number_of_ranks` | Integer | ✅ Used for Rating | Sets star/numeric scale (Rating only) |

### Conditional Display (`show_only_if`)

**Status:** ⚠️ **Not supported in jsPsych output.**

Questions with `show_only_if` conditions are rendered unconditionally in the jsPsych experiment. To implement conditional display, post-process the experiment timeline or use jsPsych's `conditional_function` feature (manual configuration required).

### Actions

**Status:** ⚠️ **Not supported in jsPsych output.**

Page and question-level actions (`skip_to`, `terminate`, `complete`) are not translated to jsPsych timeline branching. The experiment proceeds linearly through all pages.

---

## 7. Limitations and Unsupported Features

### Not Supported

1. **Conditional Display (`show_only_if`)** — All questions shown regardless of conditions.
2. **Survey Actions** — `skip_to`, `terminate`, `complete` actions ignored.
3. **Dynamic Options** — `options_from`, `rows_from`, `columns_from` not supported.
4. **Segments and Groups** — Segment activation and group logic ignored.
5. **Repeating Blocks** — `repeat` blocks rendered as separate pages only.
6. **Why Fields** — `add_why_field` attributes ignored.
7. **Custom Styling** — Per-question CSS classes not applied.
8. **Comments** — DSL comments not rendered to respondents.

### Design Limitations

- **Presentation:** jsPsych's SurveyJS plugin uses a fixed, responsive layout. Survey appearance is not customizable via DSL.
- **Timing:** Reaction time (RT) is measured from trial presentation to response submission, not per-element granularity.
- **Response Data:** All question data stored in a flat JSON structure; matrix responses use standardized naming conventions.

### Workarounds

For features requiring conditional branching, customization, or advanced logic:

1. **Export as JSON/HTML** — Use the generated experiment as a starting point.
2. **Edit Timeline Manually** — The `const timeline = [...]` array in the HTML is standard jsPsych JSON; modify directly.
3. **Use jsPsych Natively** — For full control, author the timeline in JavaScript directly using the jsPsych API.

---

## 8. Output Structure and Examples

### Timeline Structure

The generated HTML contains a jsPsych timeline array:

```javascript
const timeline = [
  // Trial 0: Welcome
  {
    type: "html-keyboard-response",
    stimulus: "<p><strong>Survey Title</strong></p><p>Press any key to begin.</p>",
    choices: "ALL"
  },
  
  // Trial N: Multi-question page
  {
    type: "survey",
    survey_json: {
      elements: [
        // Array of SurveyJS elements
      ]
    },
    page_name: "Page Title"
  },
  
  // Final trial: Completion
  {
    type: "html-keyboard-response",
    stimulus: "<p><strong>Survey Complete!</strong></p><p>Thank you for your responses.</p>",
    choices: "ALL"
  }
];

jsPsych.run(timeline);
```

### Example: Three-Page Survey

**Input DSL:**

```ruby
page "demographics" do
  title "About You"
  
  SingleSelect "gender" do
    text "Gender:"
    option "Male"
    option "Female"
    option "Other"
  end
  
  Number "age" do
    text "Age:"
  end
end

page "satisfaction" do
  title "Product Feedback"
  
  Rating "rating" do
    text "Overall satisfaction:"
    number_of_ranks 5
  end
  
  Discussion "comments" do
    text "Additional comments:"
  end
end
```

**Generated Timeline (excerpt):**

```javascript
const timeline = [
  {
    type: "html-keyboard-response",
    stimulus: "<p style=\"font-size: 18px; margin-bottom: 20px;\"><strong>survey_title</strong></p><p>Press any key to begin.</p>",
    choices: "ALL"
  },
  {
    type: "survey",
    survey_json: {
      elements: [
        {
          type: "radiogroup",
          name: "gender",
          title: "Gender:",
          choices: ["Male", "Female", "Other"],
          required: true
        },
        {
          type: "text",
          name: "age",
          title: "Age:",
          inputType: "number",
          required: true
        }
      ]
    },
    page_name: "About You"
  },
  {
    type: "survey",
    survey_json: {
      elements: [
        {
          type: "rating",
          name: "rating",
          title: "Overall satisfaction:",
          rateMin: 1,
          rateMax: 5,
          required: true
        },
        {
          type: "comment",
          name: "comments",
          title: "Additional comments:",
          required: true,
          rows: 6
        }
      ]
    },
    page_name: "Product Feedback"
  },
  {
    type: "html-keyboard-response",
    stimulus: "<p style=\"font-size: 18px; color: green;\"><strong>Survey Complete!</strong></p><p>Thank you for your responses.</p><p>Press any key to finish.</p>",
    choices: "ALL"
  }
];
```

### Response Data Format

After survey completion, response data is available as:

```javascript
window.jsPsych.data.get().json()
```

**Example response object:**

```json
[
  {
    "trial_type": "survey",
    "trial_index": 1,
    "time_elapsed": 5234,
    "rt": 3100,
    "response": {
      "gender": "Female",
      "age": "28"
    },
    "page_name": "About You"
  },
  {
    "trial_type": "survey",
    "trial_index": 2,
    "time_elapsed": 12450,
    "rt": 7216,
    "response": {
      "rating": 4,
      "comments": "Great product!"
    },
    "page_name": "Product Feedback"
  },
  {
    "trial_type": "html-keyboard-response",
    "trial_index": 3,
    "time_elapsed": 14123,
    "rt": 1673
  }
]
```

### Downloading and Viewing Responses

The generated HTML includes buttons for respondents to:

1. **Download Data** — Exports to CSV file (`survey_title_responses.csv`).
2. **View Data** — Opens JSON response data in a new window.

---

## Appendix: Browser Compatibility

jsPsych 8.2.3 requires a modern browser with ES6 support:

- Chrome/Edge 60+
- Firefox 55+
- Safari 12+
- Mobile browsers (iOS Safari, Chrome Mobile)

**Note:** Service Workers (used by the Field PWA) require HTTPS or localhost.

---

## References

- **jsPsych Documentation:** https://www.jspsych.org/latest/
- **SurveyJS Documentation:** https://surveyjs.io/form-library/documentation/
- **SRP DSL Language Reference:** `language_reference.md`

---

**Last Updated:** 2026-05-26

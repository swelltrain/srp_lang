© 2026 SRP Solutions SRP LLC. All rights reserved.

# SRP Survey DSL Language Reference — Formbricks Edition

Syntax specification for the SRP Survey DSL **as it applies to Formbricks REST API export**. This document covers features and limitations specific to Formbricks rendering. For the complete DSL reference, see `language_reference.md`.

---

## Table of Contents

1. [Overview](#1-overview)
2. [Use Cases and Capabilities](#2-use-cases-and-capabilities)
3. [CLI Usage](#3-cli-usage)
4. [Question Type Support Matrix](#4-question-type-support-matrix)
5. [Output Format](#5-output-format)
6. [Supported Question Attributes](#6-supported-question-attributes)
7. [Limitations and Unsupported Features](#7-limitations-and-unsupported-features)
8. [Formbricks Compatibility Validator](#8-formbricks-compatibility-validator)
9. [Output Structure and Examples](#9-output-structure-and-examples)

---

## 1. Overview

The **Formbricks renderer** converts SRP surveys into REST API JSON payloads ready for publishing to [Formbricks](https://formbricks.com/), a modern open-source survey platform. It generates a complete API payload that can be sent to Formbricks' REST API to create a survey in your Formbricks workspace.

### Key Characteristics

| Aspect | Details |
|--------|---------|
| **Output** | JSON payload (ready for POST to Formbricks API) |
| **Platform** | Formbricks survey platform (cloud-hosted or self-hosted) |
| **API Endpoint** | `POST https://app.formbricks.com/api/v1/management/surveys` |
| **Authentication** | Bearer token (workspace API key) |
| **Survey Types** | Link surveys, app surveys |
| **Response Collection** | Via Formbricks platform (hosted) |
| **File Size** | Typically 5–20 KB |

### Invocation

```bash
bin/survey_srp survey.srp --formbricks -o output.json
bin/survey_srp survey.srp --formbricks > output.json  # stdout
bin/survey_srp survey.srp --formbricks               # auto-names survey_formbricks.json
```

---

## 2. Use Cases and Capabilities

### Primary Use Cases

1. **Platform Integration** — Export SRP surveys to Formbricks for response collection, analytics, and integrations.
2. **Multi-Platform Publishing** — Create surveys once in SRP, distribute to both Formbricks (cloud) and custom field apps.
3. **Survey Hosting** — Leverage Formbricks' managed hosting, CORS headers, and response infrastructure without rebuilding.
4. **Rapid Prototyping** — Design surveys in SRP, test in Formbricks before deployment to field apps.

### Built-in Features

- **Linear Survey Flow** — Questions flow sequentially from beginning to end.
- **Localized Text** — All prompts support multi-language via Formbricks' language system.
- **Question Validation** — Required questions enforced at the API level.
- **Completion Screen** — Auto-generated thank-you page.
- **Native Formbricks Features** — Leverage Formbricks' analytics, integrations, and distribution channels.

---

## 3. CLI Usage

### Basic Syntax

```bash
bin/survey_srp <input.srp> --formbricks [options]
```

### Options

| Flag | Argument | Description |
|------|----------|-------------|
| `--formbricks` | None | Enable Formbricks rendering |
| `-o`, `--outfile` | `FILE` | Write JSON to specified file |

### Examples

```bash
# Write to auto-generated filename (survey_formbricks.json)
bin/survey_srp examples/survey.srp --formbricks

# Specify output file
bin/survey_srp examples/survey.srp --formbricks -o surveys/my_survey.json

# Output to stdout (pipe to file or directly to API)
bin/survey_srp examples/survey.srp --formbricks > survey.json

# Read from stdin
cat survey.srp | bin/survey_srp --formbricks -o output.json
```

### API Integration Example

```bash
# Generate JSON and POST to Formbricks API
bin/survey_srp my_survey.srp --formbricks | jq . | curl -X POST \
  -H "x-api-key: YOUR_WORKSPACE_API_KEY" \
  -H "Content-Type: application/json" \
  -d @- \
  https://app.formbricks.com/api/v1/management/surveys
```

---

## 4. Question Type Support Matrix

### Fully Supported Question Types

These question types map directly to Formbricks question types with full fidelity:

| SRP Type | Formbricks Type | Status | Notes |
|----------|-----------------|--------|-------|
| OpenEnded | `openText` | ✅ Supported | Single-line text input |
| Discussion | `openText` | ✅ Supported | Multi-line text input |
| Number | `openText` | ✅ Supported | Numeric input with `inputType: "number"` |
| FillInTheBlank | `openText` | ✅ Supported | Text input with placeholder |
| SingleSelect | `multipleChoiceSingle` | ✅ Supported | Radio button options |
| MultiSelect | `multipleChoiceMulti` | ✅ Supported | Checkbox options |
| Dropdown | `multipleChoiceSingle` | ✅ Supported | Drop-down select |
| ButtonCheckbox | `multipleChoiceMulti` | ✅ Supported | Button-style checkboxes |
| ThisOrThat | `multipleChoiceSingle` | ✅ Supported | Binary choice (2 options) |
| Ranking | `ranking` | ✅ Supported | Drag-and-drop ranking |
| Rating | `rating` | ✅ Supported | Star/number scale (1–5 default) |
| NPS | `nps` | ✅ Supported | Net Promoter Score (0–10 fixed) |
| Slider | `rating` | ✅ Supported | Range slider approximated as rating |
| SingleSelectMatrix | `matrix` | ✅ Supported | Row × column radio buttons |
| MultiSelectMatrix | `matrix` | ✅ Supported | Row × column checkboxes (approximated) |
| RatingMatrix | `matrix` | ✅ Supported | Row × column rating scale |
| SingleSelectBipolar | `rating` | ✅ Supported | 2-pole Likert scale |
| SingleSelectBipolarMatrix | `matrix` | ✅ Supported | Bipolar matrix |
| DatePicker | `date` | ✅ Supported | Date picker (M-d-y format) |
| MediaUpload | `fileUpload` | ✅ Supported | File upload field |
| USAddress | `address` | ✅ Supported | US address with standard fields |
| InternationalAddress | `address` | ✅ Supported | Address with all fields enabled |
| Age | `openText` | ✅ Supported | Numeric input for age |
| PhoneNumber | `openText` | ✅ Supported | Phone number input |
| Notification | `cta` | ✅ Supported | Display-only message (no input) |

### Degraded Support Question Types

These question types have no direct Formbricks equivalent and are mapped to `openText` (free text). The renderer will generate valid JSON, but respondents will provide text responses instead of structured data.

| SRP Type | Formbricks Fallback | Status | Notes |
|----------|---------------------|--------|-------|
| HeatMap | `openText` | ⚠️ Degraded | Image click tracking unavailable; falls back to text |
| TimedHeatMap | `openText` | ⚠️ Degraded | Timed click tracking unavailable; falls back to text |
| StickyNote | `openText` | ⚠️ Degraded | Sticky note placement unavailable; falls back to text |
| CardSort | `openText` | ⚠️ Degraded | Card sorting unavailable; falls back to text |
| CardRating | `openText` | ⚠️ Degraded | Card rating unavailable; falls back to text |
| MaxDiff | `openText` | ⚠️ Degraded | Maximum difference scaling unavailable; falls back to text |
| ConstantSum | `openText` | ⚠️ Degraded | Point allocation unavailable; falls back to text |
| TextHighlighter | `openText` | ⚠️ Degraded | Text highlighting unavailable; falls back to text |
| Autosuggest | `openText` | ⚠️ Degraded | Type-ahead unavailable; falls back to text |
| ConjointChoice | `openText` | ⚠️ Degraded | Conjoint analysis unavailable; falls back to text |
| EmotionSelector | `openText` | ⚠️ Degraded | Emotion picker unavailable; falls back to text |
| Category | `openText` | ⚠️ Degraded | Category selection unavailable; falls back to text |
| OpenEndedMatrix | `matrix` | ⚠️ Degraded | Text matrix approximated as standard matrix |

---

## 5. Output Format

### JSON Structure

The generated file is a complete Formbricks REST API request payload (JSON).

```json
{
  "workspaceId": "YOUR_WORKSPACE_ID",
  "name": "Survey Title",
  "type": "link",
  "status": "draft",
  "questions": [
    // Array of question objects
  ],
  "endings": [
    // End-of-survey thank you screens
  ]
}
```

### Top-Level Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `workspaceId` | String | Yes | Formbricks workspace ID (defaults to `"YOUR_WORKSPACE_ID"` — must be updated before use) |
| `name` | String | Yes | Survey title (derived from DSL filename or `survey_title` context) |
| `type` | String | Yes | Survey type: `"link"` or `"app"` (defaults to `"link"`) |
| `status` | String | Yes | Survey status: `"draft"`, `"inProgress"`, `"paused"`, or `"completed"` (defaults to `"draft"`) |
| `questions` | Array | Yes | Array of question objects |
| `endings` | Array | Yes | Array of end-of-survey screens (auto-generated) |

### File Size

Typical survey output: **5–20 KB** (JSON format, no compression).

---

## 6. Supported Question Attributes

### Common Attributes Recognized

Most SRP question attributes are recognized, though not all affect Formbricks output.

| Attribute | Type | Usage in Formbricks | Notes |
|-----------|------|---------------------|-------|
| `text` | String | ✅ Rendered as `headline` | Question prompt |
| `required` | Boolean | ✅ Applied to all questions | Enforced by Formbricks |
| `hidden` | Boolean | ✅ Filtered at render time | Hidden questions excluded from output |
| `min_value`, `max_value` | Integer | ✅ Used for Rating/Slider | Sets scale bounds |
| `number_of_ranks` | Integer | ✅ Used for Rating | Sets rating range (clamped to 3,4,5,6,7,10) |
| `min_label`, `max_label` | String | ✅ Used for NPS/Slider | Scale endpoint labels |
| `randomize_items` | Boolean | ✅ Used for Ranking | Sets `shuffleOption: "all"` |
| `randomized` | Boolean | ✅ Used for SingleSelect/MultiSelect | Sets `shuffleOption: "all"` |
| `options` | Array | ✅ Rendered as `choices` | Question options |

### Conditional Display (`show_only_if`)

**Status:** ⚠️ **Not supported in Formbricks output.**

Questions with `show_only_if` conditions are rendered unconditionally. To implement conditional display, use Formbricks' native branching logic (not exposed via SRP DSL).

### Actions

**Status:** ⚠️ **Not supported in Formbricks output.**

Page and question-level actions (`skip_to`, `terminate`, `complete`) are not translated to Formbricks branching. The survey will flow linearly.

### Dynamic Options

**Status:** ⚠️ **Not supported in Formbricks output.**

`options_from`, `rows_from`, and `columns_from` attributes are not supported. Options are rendered statically from the DSL.

---

## 7. Limitations and Unsupported Features

### Not Supported

1. **Conditional Display (`show_only_if`)** — All questions shown to all respondents; no branching in the API payload.
2. **Survey Actions** — `skip_to`, `terminate`, `complete` actions ignored.
3. **Dynamic Options** — `options_from`, `rows_from`, `columns_from` not supported; static options only.
4. **Segments and Groups** — Segment activation and group logic ignored.
5. **Repeating Blocks** — `repeat` blocks rendered as separate linear questions.
6. **Why Fields** — `add_why_field` attributes ignored.
7. **Custom Styling** — Per-question CSS classes not applied.
8. **Comments** — DSL comments not rendered to respondents.
9. **Text Piping** — `{{question_name}}` placeholders not supported.

### Design Limitations

- **Linear Flow:** Formbricks supports branching, but the SRP Formbricks renderer generates linear surveys only.
- **Presentation:** Formbricks handles UI rendering; appearance is customized within the Formbricks platform, not via DSL.
- **Timing:** Formbricks collects response timestamps at the platform level (not millisecond-precise per-question timing).

### Survey Logic (Conditional Branching)

**Important:** SRP's conditional logic features (`show_only_if` blocks, branching actions) cannot be included in the Formbricks API payload. The renderer creates a linear survey with all questions.

**To add conditional branching after export:**

1. **Generate and publish** the SRP survey to Formbricks using the REST API (see [Appendix](#appendix-sending-the-payload-to-formbricks)).
2. **Open the survey in Formbricks UI** and navigate to its edit page.
3. **Use Formbricks' visual Conditional Logic editor** to define conditions and actions:
   - **Condition:** "If answer to Q1 is X..."
   - **Action:** "Jump to question Q5", "Make Q3 required", "Calculate variable", etc.
4. **Save and test** within Formbricks before publishing to respondents.

This is the standard workflow for adding post-export logic. The Formbricks UI makes it simple, though it's not automated from the DSL.

### Workarounds for Complex Branching

For surveys with extensive conditional logic:

1. **Formbricks Native Editor** — Use the generated JSON as a starting point; add all branching logic in Formbricks' UI.
2. **Multiple Surveys** — Create separate SRP surveys for different conditional branches and link them in Formbricks (e.g., "Based on your answer, take Survey A or Survey B").
3. **API Extensions (Advanced)** — If you know Formbricks' conditional logic payload structure, you can manually modify the JSON before POSTing to include logic rules (not documented here, consult Formbricks API docs).

---

## 8. Formbricks Compatibility Validator

The SRP gem includes `FormbricksCompatibilityValidator` to alert you to features that don't have equivalents in Formbricks.

### Running the Validator

```ruby
# From the CLI (not directly exposed, but usable via SurveyDslService)
validator = FormbricksCompatibilityValidator.new
result = validator.validate(survey_data)

if result.valid?
  puts "✓ Survey is compatible with Formbricks"
end

result.warnings.each { |w| puts "⚠️  #{w}" }
```

### Validator Findings

The validator checks for:

- **Fully Supported Types** — No warnings generated.
- **Degraded Support Types** — Warning: "will be rendered as open text in Formbricks."
- **Conditional Display** — Warning: "`show_only_if` not supported." (See [Survey Logic](#survey-logic-conditional-branching) for post-export workflow.)
- **Hidden Questions** — Warning: "Hidden questions are omitted from output."
- **Question Actions** — Warning: "`skip_to`, `terminate`, `complete` not supported."
- **Dynamic Options** — Warning: "`options_from`, `rows_from`, `columns_from` not supported."

### Example Output

```
Valid: true
Errors: 0
Warnings: 4
  - Question type 'HeatMap' will be rendered as open text in Formbricks.
  - Question actions (skip_to, terminate, complete) are not supported in Formbricks.
  - Dynamic options (options_from) are not supported in Formbricks.
  - Conditional display (show_only_if) is not supported in Formbricks.
```

### Interpreting "Conditional Display" Warnings

If your survey has `show_only_if` blocks, the validator will flag them. These questions **will still be exported** with all other questions in a linear sequence. To implement conditional branching:

1. Export the survey to Formbricks as-is.
2. In the Formbricks UI, add conditional logic after creation (see [Survey Logic](#survey-logic-conditional-branching)).
3. Test the flow before publishing to respondents.

---

## 9. Output Structure and Examples

### Question Payload Structure

Every question in the Formbricks payload has this shape:

```json
{
  "id": "question_name",
  "type": "openText|multipleChoiceSingle|rating|etc",
  "headline": { "default": "Question text" },
  "required": true|false,
  // Type-specific fields (choices, rows, columns, rating, etc.)
}
```

### Example: Simple Survey

**Input DSL:**

```ruby
page "welcome" do
  title "Feedback Survey"
  
  Notification "intro" do
    text "Thank you for using our product. Your feedback helps us improve."
  end
end

page "questions" do
  title "Your Feedback"
  
  OpenEnded "overall_feeling" do
    text "How would you describe your overall experience?"
    required
  end
  
  SingleSelect "recommend" do
    text "Would you recommend us to others?"
    required
    option "Definitely"
    option "Maybe"
    option "No"
  end
  
  Rating "satisfaction" do
    text "How satisfied are you?"
    required
    number_of_ranks 5
  end
end

page "closing" do
  title "Thank You"
  
  Notification "closing_msg" do
    text "Thank you for your feedback! We'll use it to improve."
  end
end
```

**Generated JSON (excerpt):**

```json
{
  "workspaceId": "YOUR_WORKSPACE_ID",
  "name": "Feedback Survey",
  "type": "link",
  "status": "draft",
  "questions": [
    {
      "id": "intro",
      "type": "cta",
      "headline": {
        "default": "Thank you for using our product. Your feedback helps us improve."
      },
      "required": false
    },
    {
      "id": "overall_feeling",
      "type": "openText",
      "headline": {
        "default": "How would you describe your overall experience?"
      },
      "required": true,
      "inputType": "text"
    },
    {
      "id": "recommend",
      "type": "multipleChoiceSingle",
      "headline": {
        "default": "Would you recommend us to others?"
      },
      "required": true,
      "choices": [
        {
          "id": "recommend_c1",
          "label": { "default": "Definitely" }
        },
        {
          "id": "recommend_c2",
          "label": { "default": "Maybe" }
        },
        {
          "id": "recommend_c3",
          "label": { "default": "No" }
        }
      ],
      "shuffleOption": "none"
    },
    {
      "id": "satisfaction",
      "type": "rating",
      "headline": {
        "default": "How satisfied are you?"
      },
      "required": true,
      "rating": {
        "range": 5,
        "scale": "number"
      }
    },
    {
      "id": "closing_msg",
      "type": "cta",
      "headline": {
        "default": "Thank you for your feedback! We'll use it to improve."
      },
      "required": false
    }
  ],
  "endings": [
    {
      "id": "end1",
      "type": "endScreen",
      "headline": {
        "default": "Thank you!"
      },
      "subheader": {
        "default": "Your response has been recorded."
      }
    }
  ]
}
```

### Example: Matrix Question

**Input DSL:**

```ruby
SingleSelectMatrix "feature_ratings" do
  text "Please rate each feature:"
  required
  row "User Interface"
  row "Documentation"
  row "Customer Support"
  column "Poor"
  column "Fair"
  column "Good"
  column "Excellent"
end
```

**Generated JSON (excerpt):**

```json
{
  "id": "feature_ratings",
  "type": "matrix",
  "headline": {
    "default": "Please rate each feature:"
  },
  "required": true,
  "rows": [
    { "default": "User Interface" },
    { "default": "Documentation" },
    { "default": "Customer Support" }
  ],
  "columns": [
    { "default": "Poor" },
    { "default": "Fair" },
    { "default": "Good" },
    { "default": "Excellent" }
  ],
  "shuffleOption": "none"
}
```

---

## Appendix: Sending the Payload to Formbricks

### Step 1: Generate the JSON

```bash
bin/survey_srp my_survey.srp --formbricks -o survey_payload.json
```

### Step 2: Update `workspaceId`

Edit `survey_payload.json` and replace `"YOUR_WORKSPACE_ID"` with your actual Formbricks workspace ID.

### Step 3: POST to Formbricks API

```bash
curl -X POST \
  -H "x-api-key: YOUR_WORKSPACE_API_KEY" \
  -H "Content-Type: application/json" \
  -d @survey_payload.json \
  https://app.formbricks.com/api/v1/management/surveys
```

### Response

On success, Formbricks returns the created survey object with a generated `id`:

```json
{
  "id": "clk1234567890",
  "name": "Feedback Survey",
  "type": "link",
  "status": "draft",
  "createdAt": "2026-05-26T12:34:56Z",
  ...
}
```

Use this `id` to manage, publish, or share the survey in Formbricks.

---

## References

- **Formbricks Documentation:** https://formbricks.com/docs
- **Formbricks REST API Reference:** https://formbricks.com/docs/api-reference/rest-api
- **SRP DSL Language Reference:** `language_reference.md`

---

**Last Updated:** 2026-05-26

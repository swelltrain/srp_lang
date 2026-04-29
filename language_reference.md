© 2026 SRP Solutions SRP LLC. All rights reserved.

# SRP Survey DSL Language Reference

Complete syntax specification for the SRP Survey DSL. This document is the authoritative source for all DSL syntax, tokens, and language rules.

---

## Table of Contents

1. [Document Conventions](#1-document-conventions)
2. [Grammar Overview](#2-grammar-overview)
3. [Token Reference Tables](#3-token-reference-tables)
4. [Survey Structure](#4-survey-structure)
   - [4.5 Form Config](#45-form-config)
5. [Common Question Attributes](#5-common-question-attributes)
6. [Question Type Specifications](#6-question-type-specifications)
7. [Actions Specification](#7-actions-specification)
8. [Segmentation Specification](#8-segmentation-specification)
9. [Dynamic Content](#9-dynamic-content)
10. [Rich Text Formatting](#10-rich-text-formatting)
11. [Language Constraints and Validation Rules](#11-language-constraints-and-validation-rules)

---

## 1. Document Conventions

### Type Notation

| Notation | Meaning |
|----------|---------|
| `String` | Double-quoted text value: `"some text"` |
| `Integer` | Whole number: `5`, `100` |
| `Float` | Decimal number: `0.5`, `3.14` |
| `Boolean` | Keyword present means true; omit to disable |
| `Symbol` | Ruby symbol: `:complete`, `:terminate` |
| `ISO8601` | Date string: `"YYYY-MM-DD"` |

### Required / Optional Key

| Marker | Meaning |
|--------|---------|
| **Required** | Must be present; validation error if absent |
| Optional | May be omitted; default value applies if noted |

### Syntax Conventions

- Block attributes use function-style: `min_value 18` (preferred over `min_value = 18`)
- Boolean keywords stand alone without `true`/`false`: `required` not `required true`
- Actions use symbol syntax: `:complete`, `:terminate`, `:skip_to`
- Numeric separators: use underscores `20_000`, not commas `20,000`
- Comments: `#` prefix, ignored by parser

---

## 2. Grammar Overview

Informal BNF notation. `*` = zero or more, `+` = one or more, `?` = optional.

```
SURVEY     ::= COMMENT* FORM_CONFIG? (PAGE | GROUP | REPEAT)*

FORM_CONFIG ::= 'form_config' 'do'
                  ('submit_text' String)?
                  ('success_message' String)?
                  ('success_redirect' String)?
                'end'

PAGE       ::= 'page' String 'do'
                 title String
                 text? String
                 'randomized'?
                 'show_only_if'? (String+)
                 (PAGE_ACTION)?
                 QUESTION*
               'end'

GROUP      ::= 'group' String (',' String)+ ('do' GROUP_ATTRS 'end')?

REPEAT     ::= 'repeat' String 'do'
                 ('label' String)?
                 ('min' Integer)?
                 ('max' Integer)?
                 QUESTION+
               'end'

PAGE_ACTION ::= 'action' (:complete | :terminate)

QUESTION   ::= QuestionType String 'do'
                 COMMON_ATTRS
                 TYPE_SPECIFIC_ATTRS
                 ACTION*
               'end'

OPTION     ::= 'option' String ('do' OPTION_ATTRS 'end')?

ROW        ::= 'row' String ('do' ROW_ATTRS 'end')?

COLUMN     ::= 'column' String ('do' COL_ATTRS 'end')?

ACTION     ::= 'action' ACTION_TYPE (CONDITIONAL)?
```

---

## 3. Token Reference Tables

### 3.1 Core Structure Tokens

| Token | Type | Description | Example |
|-------|------|-------------|---------|
| `page` | Block keyword | Define a survey page | `page "welcome" do` |
| `repeat` | Block keyword | Define a repeating block of questions (filled N times by respondent) | `repeat "child" do` |
| `group` | Function | Organize pages; also used for AND-logic segment groups | `group "section", "page1", "page2"` |
| `title` | Attribute (String) | Display title for page or survey | `title "Welcome"` |
| `text` | Attribute (String) | Question text or page-level Notification shorthand | `text "Your question"` |
| `action` | Function | Control survey flow | `action :complete` |
| `show_only_if` | Function | Conditional display (page or question level) | `show_only_if "segment_name"` |
| `randomized` | Boolean | Randomize question order on a page | `randomized` |
| `label` | Attribute (String, repeat only) | Human-readable label for "Add another?" prompt | `label "Child"` |
| `min` | Attribute (Integer, repeat only) | Minimum instances required | `min 1` |
| `max` | Attribute (Integer, repeat only) | Maximum instances allowed | `max 5` |

### 3.2 Question Type Tokens

| Token | Block/Function | Description |
|-------|---------------|-------------|
| `SingleSelect` | Block | Radio button question (also supports dropdown/slider styles) |
| `MultiSelect` | Block | Checkbox question |
| `Dropdown` | Block | Native dropdown select |
| `ButtonCheckbox` | Block | Toggle-button multi-select |
| `ButtonRating` | Block | Labeled button scale |
| `ThisOrThat` | Block | Binary forced-choice between two options |
| `SingleSelectMatrix` | Block | Grid with one selection per row |
| `MultiSelectMatrix` | Block | Grid with multiple selections per row |
| `SingleSelectBipolar` | Block | Single item on bipolar scale |
| `SingleSelectBipolarMatrix` | Block | Multiple items on per-column bipolar scales |
| `OpenEnded` | Block | Single-line text input (optionally numeric) |
| `Discussion` | Block | Multi-line text area |
| `Number` | Block | Dedicated numeric input with unit and decimal control |
| `FillInTheBlank` | Block | Sentence completion with `___` blanks |
| `Rating` | Block | Star/numeric rating scale |
| `RatingMatrix` | Block | Multiple items on shared rating scale |
| `Slider` | Block | Range slider (single or per-row) |
| `NPS` | Block | Net Promoter Score (0–10 fixed scale) |
| `Ranking` | Block | Drag-and-drop rank ordering |
| `ConstantSum` | Block | Allocate points that must sum to a target |
| `CardSort` | Block | Drag cards into categories |
| `CardRating` | Block | Rate cards one at a time |
| `MaxDiff` | Block | Best/worst scaling (most/least important) |
| `DatePicker` | Block | Date selection input |
| `HeatMap` | Block | Click-coordinate capture on an image |
| `TimedHeatMap` | Block | HeatMap with countdown timer |
| `StickyNote` | Block | Sticky-note placement on an image |
| `OpenEndedMatrix` | Block | Grid of text input fields |
| `Notification` | Block | Display-only text (no input) |
| `Autosuggest` | Block | Text input with typeahead suggestions (static list or URL-driven) |
| `TextHighlighter` | Block | Highlight passages of text with categorized color markers |
| `EmotionSelector` | Block | Emoji-based emotion/sentiment selection |
| `ConjointChoice` | Block | Discrete choice experiment with attribute/level profiles |
| `MediaUpload` | Block | File upload |
| `USAddress` | Block | US address (street, city, state, zip) |
| `InternationalAddress` | Block | International address |
| `Age` | Block | Date-of-birth (month/day/year) |
| `PhoneNumber` | Block | Formatted phone number |

### 3.3 Question Content Tokens

| Token | Type | Used With | Description |
|-------|------|-----------|-------------|
| `option` | Block/Function | SingleSelect, MultiSelect, Dropdown | Define an answer choice |
| `button` | Function | ButtonCheckbox, ButtonRating | Define a button label |
| `this` | Attribute (String) | ThisOrThat | Left/first choice label |
| `that` | Attribute (String) | ThisOrThat | Right/second choice label |
| `row` | Block/Function | Matrix types, Slider, ConstantSum, OpenEndedMatrix | Define a row |
| `column` | Block/Function | Matrix types, SingleSelectBipolarMatrix | Define a column |
| `item` | Function | Ranking, CardSort, CardRating, MaxDiff | Define a sortable/ratable item |
| `category` | Block | HeatMap, TimedHeatMap, StickyNote, CardSort | Define a category |
| `options_from` | Attribute (String) | SingleSelect, MultiSelect | Populate options from a previous question |
| `rows_from` | Attribute (String) | Matrix types | Populate rows from a previous question |
| `columns_from` | Attribute (String) | Matrix types | Populate columns from a previous question |
| `emotion` | Function | EmotionSelector | Define a custom emotion override (id, label:, emoji:) |
| `categories` | Array | TextHighlighter | Define highlight category labels as an array of strings |
| `suggestions` | Array or String | Autosuggest | Static suggestion list (array) or URL for server-driven suggestions (string) |

### 3.4 Boolean Attribute Tokens

| Token | Used With | Description |
|-------|-----------|-------------|
| `required` | All question types | Make question mandatory |
| `hidden` | All question types | Render question invisible; exists in logic only |
| `randomized` | SingleSelect, MultiSelect, Dropdown, Page | Randomize option/question order |
| `randomize` | Group | Randomize page order within group |
| `randomize_rows` | Matrix types, Slider, SingleSelectBipolar | Randomize row order |
| `randomize_columns` | Matrix types | Randomize column order |
| `randomize_items` | Ranking, CardSort, CardRating, MaxDiff | Randomize item order |
| `randomize_sets` | MaxDiff | Randomize set order |
| `randomize_buttons` | ButtonCheckbox, ButtonRating | Randomize button order |
| `randomize_categories` | HeatMap, TimedHeatMap, StickyNote | Randomize category order |
| `anchored` | option block | Prevent option from moving during randomization |
| `code_as` | option block | Store/transmit a different value than the display text (see §7.5) |
| `add_none_option` | SingleSelect, MultiSelect, Dropdown, Matrix types, SingleSelectBipolar, SingleSelectBipolarMatrix, ConstantSum | Append "None" option; accepts optional block (see §7.5) |
| `add_dont_know_option` | SingleSelect, MultiSelect, Matrix types, SingleSelectBipolar, SingleSelectBipolarMatrix | Append "Don't Know" option; accepts optional block (see §7.5) |
| `add_other_option` | SingleSelect, MultiSelect, Dropdown, MultiSelectMatrix, ConstantSum | Append "Other (specify)" option; accepts optional block (see §7.5) |
| `add_other_row` | SingleSelectMatrix, MultiSelectMatrix | Append "Other" row |
| `add_other_column` | SingleSelectMatrix, MultiSelectMatrix | Append "Other" column |
| `add_why_field` | SingleSelect, MultiSelect, Dropdown, ButtonCheckbox, ButtonRating, Rating, RatingMatrix, Slider, NPS, ThisOrThat, Ranking, Number, DatePicker, SingleSelectBipolar, SingleSelectBipolarMatrix | Append open-text "Why?" field |
| `show_value` | Slider | Show current value label while dragging (default: true) |
| `allow_decimals` | Number | Allow decimal input (default: true) |
| `allow_unsorted` | CardSort | Allow items to remain unsorted |
| `allow_custom` | Autosuggest | Allow freeform text entry not in the suggestions list (default: true) |

### 3.5 Value Attribute Tokens

| Token | Type | Used With | Default | Description |
|-------|------|-----------|---------|-------------|
| `style` | Symbol | SingleSelect | `radio` | Display style: `radio`, `dropdown`, `slider` |
| `scale` | Integer | SingleSelectBipolar, SingleSelectBipolarMatrix | `5` | Number of scale points (1–10) |
| `min_selections` | Integer | MultiSelect, ButtonCheckbox | — | Minimum required selections |
| `max_selections` | Integer | MultiSelect, ButtonCheckbox | — | Maximum allowed selections |
| `min_selections_per_row` | Integer | MultiSelectMatrix | — | Minimum selections per matrix row |
| `max_selections_per_row` | Integer | MultiSelectMatrix | — | Maximum selections per matrix row |
| `number_of_ranks` | Integer | Rating, RatingMatrix, CardRating | `5` | Number of rating levels |
| `sum_to` | Integer | ConstantSum | `100` | Target sum for allocation |
| `time_limit` | Integer | TimedHeatMap | — | Countdown duration in seconds |
| `min_clicks` | Integer | HeatMap, TimedHeatMap, StickyNote | — | Minimum required clicks |
| `max_clicks` | Integer | HeatMap, TimedHeatMap, StickyNote | — | Maximum allowed clicks |
| `left_label` | String | SingleSelectBipolar, SingleSelectBipolarMatrix column | — | Left-side scale label |
| `right_label` | String | SingleSelectBipolar, SingleSelectBipolarMatrix column | — | Right-side scale label |
| `color` | String | category block | — | CSS color for category marker |
| `numeric` | Function | OpenEnded, OpenEndedMatrix | — | Enable numeric mode; see §6.1 |
| `min_value` | Float | Number, Slider | — | Minimum allowed value |
| `max_value` | Float | Number, Slider | — | Maximum allowed value |
| `allow_decimals` | Boolean | Number | `true` | Accept decimal input |
| `unit_label` | String | Number | — | Unit suffix shown after input (e.g., "years") |
| `placeholder` | String | Number, Dropdown, Autosuggest | — | Input placeholder text |
| `passage` | String | TextHighlighter | — | The text passage respondents interact with to create highlights |
| `min_highlights` | Integer | TextHighlighter | — | Minimum required highlight selections |
| `max_highlights` | Integer | TextHighlighter | — | Maximum allowed highlight selections |
| `step` | Float | Slider | `1` | Increment between slider values |
| `default_value` | Float | Slider | — | Initial slider position |
| `show_value` | Boolean | Slider | `true` | Show current value label while dragging |
| `min_label` | String | Slider, NPS | — | Label shown at minimum end |
| `max_label` | String | Slider, NPS | — | Label shown at maximum end |
| `min_date` | ISO8601 | DatePicker | — | Earliest selectable date |
| `max_date` | ISO8601 | DatePicker | — | Latest selectable date |
| `items_per_set` | Integer | MaxDiff | — | Items shown per comparison set |
| `image` | String | HeatMap, TimedHeatMap, StickyNote | — | URL or path to background image |

### 3.6 Action Tokens

| Token | Type | Description |
|-------|------|-------------|
| `:complete` | Action symbol | End survey as successfully completed |
| `:terminate` | Action symbol | End survey immediately (disqualify) |
| `:skip_to` | Action symbol | Navigate to named page or group |

### 3.7 Conditional Tokens

| Token | Type | Description |
|-------|------|-------------|
| `:if_column` | Condition | Matrix row action: fires when this row's column equals value |
| `:if_row` | Condition | Matrix column action: fires when named row equals value |
| `:if_value` | Condition | OpenEnded action: fires when input matches expression |
| `:if_group` | Condition | Action fires when all segments in named group are active (AND) |
| `:if_segment` | Condition | Action fires when named segment or group is active |
| `:if_column_count` | Condition | Action fires when count of a column value meets threshold |
| `:if_multiple_cells` | Condition | Matrix action: fires when ALL listed (row, column) pairs match (AND); argument is an array of `[row, col]` pairs |
| `:if_any_cells` | Condition | Matrix action: fires when ANY listed (row, column) pair matches (OR); argument is an array of `[row, col]` pairs |

### 3.8 Segmentation Tokens

| Token | Type | Description |
|-------|------|-------------|
| `segment` | Function | Activate a named segment (inside option, row, or column block) |
| `show_only_if` | Function | Conditional display based on active segments or groups |

### 3.9 Display Style Values (SingleSelect)

| Value | Description |
|-------|-------------|
| `radio` | Radio button group (default) |
| `dropdown` | Native `<select>` element |
| `slider` | Horizontal slider control |

### 3.10 Comparison Operators (for `:if_value`)

| Operator | Description | Supports Lists |
|----------|-------------|----------------|
| `<` | Less than (numeric) | No |
| `<=` | Less than or equal (numeric) | No |
| `>` | Greater than (numeric) | No |
| `>=` | Greater than or equal (numeric) | No |
| `==` | Equal to | Yes — `"== val1, val2"` |
| `!=` | Not equal to | Yes — `"!= val1, val2"` |

List semantics: `==` matches ANY value (OR); `!=` matches NONE of the values.

### 3.11 Count Operators (for `:if_column_count`)

| Symbol | Description |
|--------|-------------|
| `:greater_than` | Count > threshold |
| `:greater_than_or_equal` | Count >= threshold |
| `:equals` | Count == threshold |
| `:less_than` | Count < threshold |
| `:less_than_or_equal` | Count <= threshold |

---

## 4. Survey Structure

### 4.1 File Format

- Extension: `.srp`
- Encoding: UTF-8
- No top-level wrapper; file is a sequence of `page` and `group` declarations
- Top-of-file comments apply to the whole survey; all other comments must be inside a page or sibling block

### 4.2 Page

```
page String do
  title       String          # Required
  text        String?         # Shorthand — creates a Notification question
  randomized                  # Boolean
  show_only_if String+        # Conditional display
  action      :complete | :terminate   # Optional; default = advance to next page
  min_seconds Integer?        # Block Next until this many seconds have elapsed
  max_seconds Integer?        # Auto-advance after this many seconds
  show_timer  Boolean?        # Display a visible countdown / stopwatch
  QUESTION*
end
```

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `title` | String | **Required** | — | Display heading |
| `text` | String | Optional | — | Shorthand; auto-creates a Notification with this text |
| `randomized` | Boolean | Optional | false | Randomize question order on this page |
| `show_only_if` | String+ | Optional | always shown | Display only when respondent is in any listed segment/group; see §5.1 for comparison with question-level use |
| `action` | Symbol | Optional | next page | `:complete` or `:terminate` only; pages cannot use `:skip_to` |
| `min_seconds` | Integer | Optional | — | Block the Next button until at least this many seconds have elapsed on the page |
| `max_seconds` | Integer | Optional | — | Automatically advance to the next page after this many seconds |
| `show_timer` | Boolean | Optional | false | Show a visible timer (countdown if `max_seconds` set, stopwatch otherwise) |

**PageTimer notes**

- All three attributes are independent and can be combined freely.
- When `min_seconds` is set and the respondent clicks Next too early, a dismissible message is shown and navigation is blocked.
- When `max_seconds` is set, the page auto-advances regardless of required-question state (use with care).
- `show_timer` requires either `min_seconds` or `max_seconds` to be meaningful, but can be used alone to show elapsed time.
- Elapsed time is recorded in session metadata for reporting (`pageTimerElapsed`).

**Example**

```ruby
page "stimulus" do
  title "Please Review This Ad"
  min_seconds 15      # block Next until 15s elapsed
  max_seconds 120     # auto-advance after 120s
  show_timer true     # display countdown

  Notification "ad_image" do
    text "Study the ad below carefully."
  end
end
```

### 4.3 Group

Groups have two distinct uses:
1. **Page groups** — bundle pages together for navigation and optional randomization
2. **Segment groups** — bundle segment names for AND-logic evaluation

Both use the same `group` keyword. The syntax is identical; type is resolved during the validation pass using the rule below.

```
# Page group (inline or with block)
group String, String+
group String, String+ do
  randomize
end

# Segment group — same syntax; classified by member type (see Inference Rule below)
group String, String+
```

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| Name (1st arg) | String | **Required** | — | Group identifier; referenced in `skip_to` or `show_only_if` |
| Pages/Segments (2nd+ args) | String+ | **Required** | — | Page names (page group) or segment names (segment group) |
| `randomize` | Boolean | Optional | false | Page groups only: randomize page order |

#### Inference Rule

After parsing completes, the validator classifies each group by inspecting its members against the full set of defined segment names:

- **Segment group** — every member matches a defined segment name. The group evaluates with AND logic: all listed segments must be active simultaneously.
- **Page group** — no member matches a defined segment name. Members must all be valid page names; a validation error is raised for any unknown page name.
- **Mixed (invalid)** — some members are segment names and others are not. A validation error is raised: `"Group 'X' mixes segment names and page names"`.

The `randomize` attribute is valid only on page groups.

Because classification happens after the full survey is parsed, segment and page names may appear in any order in the file — forward references are resolved before the check runs.

### 4.4 Repeat

A `repeat` block defines a set of questions the respondent fills out N times (e.g., once per child, once per nationality). It is a top-level construct — a peer of `page`, not nested inside one.

At runtime, each instance is presented as its own page. After completing an instance the respondent sees an "Add another [label]?" prompt; answering "Yes" creates a new instance.

```ruby
repeat "child" do
  label "Child"    # drives "Add another Child?" and "Child 1 / Child 2" headings
  min 1            # at least one instance required (default: 0)
  max 5            # no more than 5 instances (default: unlimited)

  OpenEnded "child_first_name" do
    text "First name"
    required
  end

  OpenEnded "child_last_name" do
    text "Last name"
  end

  DatePicker "child_dob" do
    text "Date of birth"
    required
  end

  SingleSelect "child_gender" do
    text "Gender"
    option "Female"
    option "Male"
    option "Other"
    required
  end
end
```

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| Name (1st arg) | String | **Required** | — | Unique identifier; must not collide with page names |
| `label` | String | Optional | repeat name | Human-readable label used in "Add another [label]?" and instance headings |
| `min` | Integer | Optional | `0` | Minimum number of instances (>= 0). When > 0, the "Done" button is disabled until this count is reached |
| `max` | Integer | Optional | `nil` (unlimited) | Maximum number of instances (> 0, >= `min`). When reached, the "Add another?" prompt is suppressed |
| Questions | Block | **Required** | — | One or more question definitions (any question type supported in a page) |

**Constraints:**
- A repeat must contain at least one question.
- `min` must be >= 0.
- `max` must be > 0 and >= `min` when set.
- The repeat name must be unique across all page names and repeat names.
- Question names inside a repeat must be globally unique (no collision with page questions or other repeat questions).
- `show_only_if` and `skip_to` inside a repeat body are not supported in v1.
- A repeat name is a valid `skip_to` target from a page action.

**Response storage format:**

Each question inside a repeat is stored with an instance index suffix:

```
child_first_name[0]   → First instance, first name
child_first_name[1]   → Second instance, first name
child_gender[0]       → First instance, gender
```

### 4.5 Form Config

`form_config` is an optional top-level block that sets defaults for the `form` renderer (the single-page HTML form output used for embedding). It has no effect on the `html`, `code`, `mermaid`, or `xform` renderers.

```ruby
form_config do
  submit_text      String?   # Submit button label
  success_message  String?   # Message displayed after successful submission
  success_redirect String?   # URL to redirect to after submission
end
```

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `submit_text` | String | Optional | `"Submit"` | Label on the submit button |
| `success_message` | String | Optional | renderer default | Text shown on the confirmation screen |
| `success_redirect` | String | Optional | — | URL the browser navigates to after submission |

When context options are passed at render time they take precedence over `form_config` values.

**Example**

```ruby
form_config do
  submit_text "Send Application"
  success_message "Thank you! We'll be in touch within 2 business days."
  success_redirect "https://example.com/thank-you"
end

page "application" do
  title "Apply Now"
  OpenEnded "applicant_name" do
    text "Full name"
    required
  end
end
```

---

## 5. Common Question Attributes

These attributes apply to every question type unless noted otherwise.

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| Name (1st arg) | String | **Required** | — | Unique identifier; snake_case; referenced in actions, piping, dynamic options |
| `text` | String | **Required** | — | Question prompt displayed to respondent |
| `required` | Boolean | Optional | false | Respondent must answer before advancing |
| `hidden` | Boolean | Optional | false | Question exists in logic but is not rendered |
| `show_only_if` | String+ | Optional | always shown | Display only when respondent is in any listed segment/group |
| `add_why_field` | Boolean | Optional | false | Appends a free-text "Why?" field after the question |

### 5.1 Page-Level vs. Question-Level show_only_if

`show_only_if` accepts the same syntax and OR/AND logic at both levels (see §8 for logic rules). The difference is scope and its downstream consequences.

| Dimension | Page-level | Question-level |
|-----------|------------|----------------|
| **Scope** | Entire page — title and all questions are hidden as a unit | Single question only; the rest of the page renders normally |
| **Navigation** | The page is skipped entirely in the navigation sequence; the respondent never "visits" it | The page is still visited; only the conditional question is absent |
| **Required validation** | Required questions on a hidden page are never reached and do not block progress | A hidden question is excluded from required validation on a visible page |
| **Interaction** | Independent of any question-level conditions on its questions | Only evaluated when the containing page is itself visible; a hidden page makes all its questions' conditions moot |
| **Typical use** | Gate an entire section of the survey (e.g., show a demographics block only to a certain segment) | Hide or reveal an individual follow-up within a page that is always shown |

Both levels accept the same arguments — segment names (OR logic) and group names (AND logic) — and both are validated identically: every referenced name must be a defined segment or segment group (see §11.3).

---

## 6. Question Type Specifications

Each specification follows this template:
1. Syntax line
2. Attributes table (type-specific only; common attributes omitted)
3. Minimal example

---

### 6.1 Text Input

#### OpenEnded

Single-line text input. Supports numeric mode and `:if_value` conditional actions.

```
OpenEnded String do
  text String
  numeric :min_value, Float, :max_value, Float   # optional; enables numeric mode
  action ActionType (:if_value String)?*
end
```

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `numeric` | Function | Optional | — | Enables numeric HTML input; accepts `:min_value N`, `:max_value N`, or both |
| `action ... :if_value` | String | Optional | — | Condition string: `"< 18"`, `"== value1, value2"` |

```ruby
OpenEnded "age" do
  text "What is your age?"
  numeric :min_value, 18, :max_value, 120
  required
  action :terminate, :if_value, "< 18"
  action :skip_to, "senior_page", :if_value, ">= 65"
end
```

---

#### Number

Dedicated numeric input with display controls.

```
Number String do
  text String
  min_value Float?
  max_value Float?
  allow_decimals Boolean?
  unit_label String?
  placeholder String?
  add_why_field?
end
```

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `min_value` | Float | Optional | — | Minimum accepted value |
| `max_value` | Float | Optional | — | Maximum accepted value |
| `allow_decimals` | Boolean | Optional | true | Accept decimal input |
| `unit_label` | String | Optional | — | Suffix shown after the input field |
| `placeholder` | String | Optional | — | Input placeholder text |
| `add_why_field` | Boolean | Optional | false | Append a free-text "Why?" field |

```ruby
Number "weight" do
  text "What is your current weight?"
  min_value 0.1
  max_value 500.0
  unit_label "lbs"
  placeholder "Enter weight"
  required
end
```

---

#### Discussion

Multi-line text area for extended responses.

```
Discussion String do
  text String
end
```

No type-specific attributes. `Discussion` does not support `min_length`, `max_length`, or any character-limit constraint — the rendered `<textarea>` accepts unlimited input. If you need to enforce length constraints, use `OpenEnded` with client-side validation or handle it server-side.

```ruby
Discussion "detailed_feedback" do
  text "Please describe your experience in detail."
  required
end
```

---

#### FillInTheBlank

Sentence completion. Text must contain at least one `___` placeholder.

```
FillInTheBlank String do
  text String   # Must include one or more "___" sequences
end
```

```ruby
FillInTheBlank "completion" do
  text "My favorite ___ is ___ because ___."
  required
end
```

---

#### Autosuggest

Text input with typeahead suggestions. Accepts a static list or a URL for server-driven suggestions.

```
Autosuggest String do
  text String
  suggestions (Array | String)?   # Array of strings OR URL string for server-driven
  allow_custom Boolean?           # Default: true
  placeholder String?
end
```

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `suggestions` | Array or String | Optional | — | Static list of strings, or a URL string fetched client-side for typeahead |
| `allow_custom` | Boolean | Optional | `true` | Whether freeform input not in the list is accepted |
| `placeholder` | String | Optional | — | Input placeholder text |

```ruby
Autosuggest "city_input" do
  text "Which city do you live in?"
  suggestions ["New York", "Los Angeles", "Chicago", "Houston"]
  allow_custom true
  placeholder "Start typing a city..."
  required
end

# URL-driven (server populates suggestions dynamically):
Autosuggest "brand_lookup" do
  text "Which brand are you reviewing?"
  suggestions "https://example.com/api/brands"
  allow_custom false
  required
end
```

---

### 6.2 Selection

#### SingleSelect

Single-choice question. Supports radio buttons, dropdown, and slider display styles.

```
SingleSelect String do
  text String
  style (radio | dropdown | slider)?
  randomized?
  add_none_option?
  add_dont_know_option?
  add_other_option?
  add_why_field?
  options_from String?
  option String (do OPTION_BLOCK end)?*
end
```

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `style` | Symbol | Optional | `radio` | `radio`, `dropdown`, or `slider` |
| `randomized` | Boolean | Optional | false | Randomize option order |
| `add_none_option` | Boolean or Block | Optional | false | Append "None" option |
| `add_dont_know_option` | Boolean or Block | Optional | false | Append "Don't Know" option |
| `add_other_option` | Boolean or Block | Optional | false | Append "Other (please specify)" option |
| `min_label` | String | Optional | — | Label at left/low end of scale; meaningful only when `style :slider` |
| `max_label` | String | Optional | — | Label at right/high end of scale; meaningful only when `style :slider` |
| `options_from` | String | Optional | — | Populate options from named question's selections |

Requires at least 2 `option` entries when options are defined statically.

```ruby
SingleSelect "satisfaction" do
  text "How satisfied are you overall?"
  required
  randomized

  option "Very satisfied"
  option "Satisfied"
  option "Neutral"
  option "Dissatisfied" do
    action :skip_to, "feedback_page"
  end
  option "None of the above" do
    anchored
  end
end
```

---

#### MultiSelect

Multiple-choice checkbox question.

```
MultiSelect String do
  text String
  min_selections Integer?
  max_selections Integer?
  randomized?
  add_none_option?
  add_dont_know_option?
  add_other_option?
  add_why_field?
  options_from String?
  option String (do OPTION_BLOCK end)?*
end
```

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `min_selections` | Integer | Optional | — | Minimum number of options to select |
| `max_selections` | Integer | Optional | — | Maximum number of options to select |
| `randomized` | Boolean | Optional | false | Randomize option order |
| `options_from` | String | Optional | — | Populate options from named question's selections |

```ruby
MultiSelect "interests" do
  text "Select all topics that interest you."
  min_selections 1
  max_selections 4
  add_other_option

  option "Technology"
  option "Business"
  option "Design"
  option "Marketing"
end
```

---

#### Dropdown

Native select element. Functionally identical to `SingleSelect` with `style dropdown` but rendered as a native `<select>`.

```
Dropdown String do
  text String
  randomized?
  placeholder String?
  add_none_option?
  add_other_option?
  add_why_field?
  option String*
end
```

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `randomized` | Boolean | Optional | false | Randomize option order |
| `placeholder` | String | Optional | — | Placeholder text shown before selection |
| `add_none_option` | Boolean or Block | Optional | false | Append "None" option |
| `add_other_option` | Boolean or Block | Optional | false | Append "Other (please specify)" option |
| `add_why_field` | Boolean | Optional | false | Append a free-text "Why?" field |

```ruby
Dropdown "country" do
  text "Select your country."
  option "United States"
  option "Canada"
  option "United Kingdom"
  add_other_option
  required
end
```

---

#### ButtonCheckbox

Toggle-button multi-select. Similar to `MultiSelect` but rendered as button toggles.

```
ButtonCheckbox String do
  text String
  min_selections Integer?
  max_selections Integer?
  randomize_buttons?
  add_why_field?
  button String*
end
```

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `min_selections` | Integer | Optional | — | Minimum number of buttons to select |
| `max_selections` | Integer | Optional | — | Maximum number of buttons to select |
| `randomize_buttons` | Boolean | Optional | false | Randomize button order |

```ruby
ButtonCheckbox "features" do
  text "Which features are most important to you?"
  min_selections 1
  max_selections 3
  randomize_buttons

  button "Price"
  button "Quality"
  button "Speed"
  button "Support"
  required
end
```

---

#### ButtonRating

Scale rendered as a row of labeled buttons.

```
ButtonRating String do
  text String
  randomize_buttons?
  add_why_field?
  button String*
end
```

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `randomize_buttons` | Boolean | Optional | false | Randomize button order |

```ruby
ButtonRating "satisfaction" do
  text "How satisfied are you with our service?"
  button "Very Dissatisfied"
  button "Dissatisfied"
  button "Neutral"
  button "Satisfied"
  button "Very Satisfied"
  add_why_field
  required
end
```

---

#### ThisOrThat

Forced binary choice between exactly two labeled options.

```
ThisOrThat String do
  text String
  this String
  that String
  add_why_field?
end
```

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `this` | String | **Required** | — | Left/first option label |
| `that` | String | **Required** | — | Right/second option label |

```ruby
ThisOrThat "drink_preference" do
  text "Which do you prefer in the morning?"
  this "Coffee"
  that "Tea"
  add_why_field
  required
end
```

---

#### Ranking

Drag-and-drop rank ordering of items.

```
Ranking String do
  text String
  randomize_items?
  add_why_field?
  item String*
end
```

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `randomize_items` | Boolean | Optional | false | Present items in random initial order |

```ruby
Ranking "priorities" do
  text "Rank these features from most to least important."
  randomize_items

  item "Performance"
  item "Ease of Use"
  item "Price"
  item "Support"
  required
end
```

---

#### EmotionSelector

Emoji-based emotion/sentiment selection. Uses five default emotions unless overridden with `emotion` calls.

```
EmotionSelector String do
  text String
  emotion String, label: String, emoji: String   # optional; override defaults*
  add_why_field?
end
```

\*Default emotions (used when no `emotion` calls are made): Delighted 😄, Happy 🙂, Neutral 😐, Unhappy 🙁, Angry 😠. Any `emotion` call replaces the entire default set.

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `emotion` | Function | Optional | see above | Add a custom emotion; calling once overrides all defaults |
| `add_why_field` | Boolean | Optional | false | Append a free-text "Why?" field |

```ruby
# Default emotions
EmotionSelector "product_reaction" do
  text "How do you feel about this product?"
  add_why_field
  required
end

# Custom emotion set (replaces defaults entirely)
EmotionSelector "service_reaction" do
  text "How was your experience?"
  emotion "love",      label: "Love it",    emoji: "❤️"
  emotion "like",      label: "Like it",    emoji: "👍"
  emotion "meh",       label: "Meh",        emoji: "😶"
  emotion "dislike",   label: "Dislike it", emoji: "👎"
  required
end
```

---

### 6.3 Matrix Questions

#### SingleSelectMatrix

Grid where each row has exactly one radio button selection.

```
SingleSelectMatrix String do
  text String
  randomize_rows?
  randomize_columns?
  add_none_option?
  add_dont_know_option?
  add_other_row?
  add_other_column?
  rows_from String?
  columns_from String?
  row String (do ROW_BLOCK end)?*
  column String (do COL_BLOCK end)?*
end
```

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `randomize_rows` | Boolean | Optional | false | Randomize row order |
| `randomize_columns` | Boolean | Optional | false | Randomize column order |
| `add_none_option` | Boolean or Block | Optional | false | Append "None" column |
| `add_dont_know_option` | Boolean or Block | Optional | false | Append "Don't Know" column |
| `add_other_row` | Boolean | Optional | false | Append "Other" row |
| `add_other_column` | Boolean | Optional | false | Append "Other" column |
| `rows_from` | String | Optional | — | Populate rows from named question's selections |
| `columns_from` | String | Optional | — | Populate columns from named question's selections |

```ruby
SingleSelectMatrix "service_ratings" do
  text "Rate each service aspect."
  required
  randomize_rows

  row "Customer Support" do
    segment "support_issues", :if_column, "Poor"
    action :skip_to, "escalation", :if_column, "Poor"
  end
  row "Product Quality"
  row "Billing"

  column "Excellent"
  column "Good"
  column "Fair"
  column "Poor"
end
```

---

#### MultiSelectMatrix

Grid where each row allows multiple checkbox selections.

```
MultiSelectMatrix String do
  text String
  min_selections_per_row Integer?
  max_selections_per_row Integer?
  randomize_rows?
  randomize_columns?
  add_none_option?
  add_dont_know_option?
  add_other_option?
  add_other_row?
  add_other_column?
  add_why_field?
  rows_from String?
  columns_from String?
  row String*
  column String*
end
```

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `min_selections_per_row` | Integer | Optional | — | Minimum checkboxes per row |
| `max_selections_per_row` | Integer | Optional | — | Maximum checkboxes per row |
| `randomize_rows` | Boolean | Optional | false | Randomize row order |
| `randomize_columns` | Boolean | Optional | false | Randomize column order |
| `add_none_option` | Boolean or Block | Optional | false | Append "None" column |
| `add_dont_know_option` | Boolean or Block | Optional | false | Append "Don't Know" column |
| `add_other_option` | Boolean or Block | Optional | false | Append "Other (specify)" column |
| `add_other_row` | Boolean | Optional | false | Append "Other" row |
| `add_other_column` | Boolean | Optional | false | Append "Other" column |
| `add_why_field` | Boolean | Optional | false | Append a free-text "Why?" field |
| `rows_from` | String | Optional | — | Populate rows from named question's selections |
| `columns_from` | String | Optional | — | Populate columns from named question's selections |

```ruby
MultiSelectMatrix "brand_attributes" do
  text "Select all attributes that apply to each brand."
  max_selections_per_row 3

  row "Brand A"
  row "Brand B"

  column "Quality"
  column "Value"
  column "Innovation"
  column "Trust"
end
```

---

#### OpenEndedMatrix

Grid of text input fields.

```
OpenEndedMatrix String do
  text String
  randomize_rows?
  randomize_columns?
  numeric (:min_value Float)? (:max_value Float)?
  rows_from String?
  columns_from String?
  row String*
  column String*
end
```

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `randomize_rows` | Boolean | Optional | false | Randomize row order |
| `randomize_columns` | Boolean | Optional | false | Randomize column order |
| `numeric` | Function | Optional | — | Restrict cells to numeric input; accepts `:min_value N`, `:max_value N`, or both |
| `rows_from` | String | Optional | — | Populate rows from named question's selections |
| `columns_from` | String | Optional | — | Populate columns from named question's selections |

```ruby
OpenEndedMatrix "feedback_grid" do
  text "Provide feedback for each item."

  row "Product A"
  row "Product B"

  column "Strengths"
  column "Improvements"
end
```

---

#### RatingMatrix

Multiple items on a shared rating scale.

```
RatingMatrix String do
  text String
  number_of_ranks Integer?
  randomize_rows?
  add_why_field?
  row String*
end
```

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `number_of_ranks` | Integer | Optional | `5` | Number of rating levels |
| `randomize_rows` | Boolean | Optional | false | Randomize row order |

```ruby
RatingMatrix "aspect_ratings" do
  text "Rate each aspect of our service."
  number_of_ranks 7
  randomize_rows
  add_why_field

  row "Quality"
  row "Speed"
  row "Support"
  required
end
```

---

#### SingleSelectBipolar

Single item rated on a bipolar (opposing poles) scale.

```
SingleSelectBipolar String do
  text String
  left_label String
  right_label String
  scale Integer?
  randomize_rows?
  add_none_option?
  add_dont_know_option?
  add_why_field?
  row String*
end
```

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `left_label` | String | **Required** | — | Left-end scale label |
| `right_label` | String | **Required** | — | Right-end scale label |
| `scale` | Integer | Optional | `5` | Number of scale points (1–10) |
| `randomize_rows` | Boolean | Optional | false | Randomize row order |
| `add_none_option` | Boolean | Optional | false | Append "None" column |
| `add_dont_know_option` | Boolean | Optional | false | Append "Don't Know" column |
| `add_why_field` | Boolean | Optional | false | Append a free-text "Why?" field |

```ruby
SingleSelectBipolar "brand_perception" do
  text "Rate each aspect of our brand."
  left_label "Poor"
  right_label "Excellent"
  scale 7

  row "Quality"
  row "Value"
  row "Trust"
  required
end
```

---

#### SingleSelectBipolarMatrix

Multiple items, each rated on its own bipolar scale (scale per column).

```
SingleSelectBipolarMatrix String do
  text String
  scale Integer?
  randomize_rows?
  randomize_columns?
  add_none_option?
  add_dont_know_option?
  add_why_field?
  rows_from String?
  columns_from String?
  row String*
  column String do
    left_label String
    right_label String
  end*
end
```

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `scale` | Integer | Optional | `5` | Number of scale points |
| `randomize_rows` | Boolean | Optional | false | Randomize row order |
| `randomize_columns` | Boolean | Optional | false | Randomize column order |
| `add_none_option` | Boolean | Optional | false | Append "None" column |
| `add_dont_know_option` | Boolean | Optional | false | Append "Don't Know" column |
| `add_why_field` | Boolean | Optional | false | Append a free-text "Why?" field |
| `rows_from` | String | Optional | — | Populate rows from named question's selections |
| `columns_from` | String | Optional | — | Populate columns from named question's selections |

Column block attributes: `left_label` (String, required), `right_label` (String, required).

```ruby
SingleSelectBipolarMatrix "feature_ratings" do
  text "Rate each feature on multiple dimensions."
  scale 5

  row "Feature A"
  row "Feature B"

  column "Ease of Use" do
    left_label "Difficult"
    right_label "Easy"
  end
  column "Value" do
    left_label "Poor Value"
    right_label "Great Value"
  end
end
```

---

### 6.4 Rating Questions

#### Rating

Star or numeric rating scale for a single item.

```
Rating String do
  text String
  number_of_ranks Integer?
  add_why_field?
end
```

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `number_of_ranks` | Integer | Optional | `5` | Number of rating levels |

```ruby
Rating "overall_satisfaction" do
  text "How satisfied are you overall?"
  number_of_ranks 5
  add_why_field
  required
end
```

---

#### Slider

Range slider, optionally with multiple rows.

```
Slider String do
  text String
  min_value Float?
  max_value Float?
  step Float?
  default_value Float?
  show_value Boolean?
  min_label String?
  max_label String?
  add_why_field?
  randomize_rows?
  row String*           # optional: makes multi-row slider
end
```

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `min_value` | Float | Optional | `0` | Minimum slider value |
| `max_value` | Float | Optional | `100` | Maximum slider value |
| `step` | Float | Optional | `1` | Increment between values |
| `default_value` | Float | Optional | `min_value` | Initial slider position; defaults to `min_value` when omitted |
| `show_value` | Boolean | Optional | `true` | Show current value label while dragging |
| `min_label` | String | Optional | — | Label at minimum end |
| `max_label` | String | Optional | — | Label at maximum end |
| `randomize_rows` | Boolean | Optional | false | Randomize row order (multi-row mode only) |
| `row` | String | Optional | — | Add a row for multi-row slider mode |

```ruby
Slider "satisfaction" do
  text "How satisfied are you with our product?"
  min_value 0
  max_value 10
  step 1
  default_value 5
  min_label "Not at all"
  max_label "Extremely"
  add_why_field
  required
end
```

Multi-row example:

```ruby
Slider "attribute_ratings" do
  text "Rate each attribute:"
  min_value 1
  max_value 7
  default_value 4

  row "Ease of Use"
  row "Performance"
  row "Value"
  required
end
```

---

#### NPS

Net Promoter Score. Fixed 0–10 scale; labels are optional.

```
NPS String do
  text String
  min_label String?
  max_label String?
  add_why_field?
end
```

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `min_label` | String | Optional | `"Not at all likely"` | Label for score 0 |
| `max_label` | String | Optional | `"Extremely likely"` | Label for score 10 |

```ruby
NPS "recommend_score" do
  text "How likely are you to recommend us to a friend or colleague?"
  min_label "Not at all likely"
  max_label "Extremely likely"
  add_why_field
  required
end
```

---

### 6.5 Image-Based Questions

All image-based questions share these common attributes:

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `image` | String | **Required** | — | URL or file path to background image |
| `min_clicks` | Integer | Optional | — | Minimum required clicks/placements |
| `max_clicks` | Integer | Optional | — | Maximum allowed clicks/placements |
| `randomize_categories` | Boolean | Optional | false | Randomize category order in selector |

#### HeatMap

Click-coordinate capture on an image with optional category tagging.

```
HeatMap String do
  text String
  image String
  min_clicks Integer?
  max_clicks Integer?
  randomize_categories?
  category String (do color String end)?*
end
```

```ruby
HeatMap "improvement_areas" do
  text "Click on areas that need improvement."
  image "https://example.com/interface.png"
  min_clicks 2
  max_clicks 5
  randomize_categories

  category "Navigation" do
    color "red"
  end
  category "Content" do
    color "blue"
  end
  required
end
```

---

#### TimedHeatMap

HeatMap with a countdown timer. Interaction is disabled when the timer expires.

```
TimedHeatMap String do
  text String
  image String
  time_limit Integer
  min_clicks Integer?
  max_clicks Integer?
  category String (do color String end)?*
end
```

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `time_limit` | Integer | **Required** | — | Countdown duration in seconds |

```ruby
TimedHeatMap "quick_feedback" do
  text "Click on problem areas. You have 30 seconds."
  image "https://example.com/dashboard.png"
  time_limit 30
  min_clicks 1
  max_clicks 10

  category "Issue"
  category "Suggestion"
  required
end
```

---

#### StickyNote

Sticky-note placement on an image with category tagging.

```
StickyNote String do
  text String
  image String
  min_clicks Integer?
  max_clicks Integer?
  randomize_categories?
  category String (do color String end)?*
end
```

```ruby
StickyNote "design_feedback" do
  text "Place notes on areas that need attention."
  image "https://example.com/mockup.png"
  min_clicks 1
  max_clicks 8

  category "Positive" do
    color "green"
  end
  category "Issue" do
    color "red"
  end
  required
end
```

---

### 6.6 Specialized Questions

#### ConstantSum

Allocate a fixed number of points across rows. Total must equal `sum_to`.

```
ConstantSum String do
  text String
  sum_to Integer?
  randomize_rows?
  add_other_option?
  add_none_option?
  row String*
end
```

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `sum_to` | Integer | Optional | `100` | Required total for all row values |
| `randomize_rows` | Boolean | Optional | false | Randomize row order |
| `add_other_option` | Boolean | Optional | false | Append "Other (specify)" row |
| `add_none_option` | Boolean | Optional | false | Append "None" row |

```ruby
ConstantSum "budget_allocation" do
  text "Allocate 100 points across these priorities."
  sum_to 100
  randomize_rows
  required

  row "Quality"
  row "Speed"
  row "Price"
  row "Support"
end
```

---

#### CardSort

Drag cards into predefined categories.

```
CardSort String do
  text String
  randomize_items?
  allow_unsorted?
  item String*
  category String*
end
```

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `randomize_items` | Boolean | Optional | false | Randomize card order |
| `allow_unsorted` | Boolean | Optional | false | Allow items to remain in an "Unsorted" pile |

```ruby
CardSort "navigation_sort" do
  text "Sort each page into the category where you'd expect to find it."
  randomize_items
  allow_unsorted

  item "Our Team"
  item "Pricing"
  item "Blog"
  item "API Documentation"

  category "Company Info"
  category "Products"
  category "Resources"
  required
end
```

---

#### CardRating

Rate each item on a scale, presented one card at a time.

```
CardRating String do
  text String
  number_of_ranks Integer?
  randomize_items?
  item String*
end
```

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `number_of_ranks` | Integer | Optional | `5` | Rating scale size |
| `randomize_items` | Boolean | Optional | false | Randomize card order |

```ruby
CardRating "feature_importance" do
  text "Rate each feature by how important it is to you."
  number_of_ranks 5
  randomize_items

  item "Easy to use"
  item "Fast performance"
  item "Affordable price"
  item "Great support"
  required
end
```

---

#### ConjointChoice

Discrete choice experiment: present multiple concept cards (profiles), each defined by a set of attribute/level pairs. The respondent picks their preferred option.

```
ConjointChoice String do
  text String
  attribute String*
  profile String do
    level String, String
  end
  add_none_option?
  randomize_profiles?
end
```

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `attribute` | String | **Required** (1+) | — | Attribute name displayed in each card's table |
| `profile` | Block | **Required** (2+) | — | A named concept card; contains `level` declarations |
| `level` | String, String | Required inside profile | — | Maps an attribute name to a value for this profile |
| `add_none_option` | Boolean | Optional | false | Adds a "None of these" choice card |
| `randomize_profiles` | Boolean | Optional | false | Shuffle profile order on each render |

Response stored as `{ selectedValues: "Option A" }` (the chosen profile name), or `null` if unanswered.

```ruby
ConjointChoice "brand_pref" do
  text "Which product would you prefer?"
  required

  attribute "Brand"
  attribute "Price"
  attribute "Quality"

  profile "Option A" do
    level "Brand", "Nike"
    level "Price", "$50"
    level "Quality", "High"
  end

  profile "Option B" do
    level "Brand", "Adidas"
    level "Price", "$75"
    level "Quality", "Medium"
  end

  add_none_option
  randomize_profiles
end
```

---

#### MaxDiff

Best-worst scaling: respondent selects the MOST and LEAST important item in each set.

```
MaxDiff String do
  text String
  items_per_set Integer
  randomize_sets?
  item String*
end
```

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `items_per_set` | Integer | **Required** | — | Number of items shown per comparison set |
| `randomize_sets` | Boolean | Optional | false | Randomize which sets are shown and in what order |

```ruby
MaxDiff "feature_prefs" do
  text "For each set, select the MOST and LEAST important feature."
  items_per_set 4

  item "Price"
  item "Quality"
  item "Speed of Delivery"
  item "Customer Support"
  item "Ease of Use"
  item "Customization Options"
  item "Brand Reputation"
  required
end
```

---

#### DatePicker

Date selection input with optional date range constraints.

```
DatePicker String do
  text String
  min_date ISO8601?
  max_date ISO8601?
  add_why_field?
end
```

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `min_date` | ISO8601 | Optional | — | Earliest selectable date (format: `"YYYY-MM-DD"`) |
| `max_date` | ISO8601 | Optional | — | Latest selectable date (format: `"YYYY-MM-DD"`) |
| `add_why_field` | Boolean | Optional | false | Append a free-text "Why?" field |

```ruby
DatePicker "birth_date" do
  text "What is your date of birth?"
  min_date "1924-01-01"
  max_date "2006-12-31"
  required
end
```

---

#### TextHighlighter

Display a passage of text and allow respondents to select and categorize highlighted spans.

```
TextHighlighter String do
  text String
  passage String
  categories Array?        # e.g., ["Positive", "Negative", "Interesting"]
  min_highlights Integer?
  max_highlights Integer?
end
```

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `passage` | String | **Required** | — | The text passage displayed for respondents to select/highlight |
| `categories` | Array | Optional | — | Category labels shown in toolbar; each gets a distinct color |
| `min_highlights` | Integer | Optional | — | Minimum required highlights before respondent may advance |
| `max_highlights` | Integer | Optional | — | Maximum allowed highlights |

```ruby
TextHighlighter "ad_copy_feedback" do
  text "Highlight any parts of this ad copy that stand out to you."
  passage "Our new product is built for people who demand the best. With cutting-edge technology and unmatched quality, it redefines what's possible."
  categories ["Positive", "Negative", "Confusing"]
  min_highlights 1
  required
end
```

---

### 6.7 Address / Demographic Questions

#### USAddress

US address collection (street, city, state, zip).

```
USAddress String do
  text String
end
```

```ruby
USAddress "home_address" do
  text "Please enter your home address."
  required
end
```

---

#### InternationalAddress

International address collection with flexible format.

```
InternationalAddress String do
  text String
end
```

```ruby
InternationalAddress "billing_address" do
  text "Please enter your billing address."
  required
end
```

---

#### Age

Date-of-birth collection (month/day/year inputs).

```
Age String do
  text String
end
```

```ruby
Age "date_of_birth" do
  text "Please enter your date of birth."
  required
end
```

---

#### PhoneNumber

Formatted phone number input.

```
PhoneNumber String do
  text String
end
```

```ruby
PhoneNumber "contact_phone" do
  text "Please enter your phone number."
  required
end
```

---

### 6.8 Media

#### MediaUpload

File upload interface. Stores file metadata (name, size, type) in IndexedDB across sessions; the host platform must handle actual file storage/upload.

```
MediaUpload String do
  text String
  [required]
  [accept String]
  [max_files Integer]
  [multiple Boolean]
  [show_only_if ...]
  [hidden]
end
```

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `text` | String | — | Label shown above the upload area |
| `required` | flag | false | Respondent must select at least one file before advancing |
| `accept` | String | — | File type filter passed to the browser (e.g. `"image/*"`, `".pdf,.docx"`) |
| `max_files` | Integer | — | Maximum number of files the respondent may select |
| `multiple` | Boolean | `true` | Allow selecting multiple files; pass `false` to restrict to one |
| `show_only_if` | condition | — | Conditional display (see section 5) |
| `hidden` | flag | false | Exclude from rendering; kept in survey logic |

```ruby
MediaUpload "supporting_docs" do
  text "Upload any supporting documents (optional)."
end

MediaUpload "profile_photo" do
  text "Upload a profile photo."
  required
  accept "image/*"
  multiple false
end

MediaUpload "attachments" do
  text "Attach up to 3 files."
  accept ".pdf,.docx,.xlsx"
  max_files 3
end
```

---

### 6.9 Display

#### Notification

Display-only text. Cannot be marked `required`. May be created implicitly via page-level bare string.

```
Notification String do
  text String
end
```

Shorthand (page level only):

```ruby
page "welcome" do
  title "Welcome"
  "This text auto-creates a Notification question."
end
```

Explicit syntax (works anywhere inside a page):

```ruby
Notification "instructions" do
  text "Please read the following instructions before proceeding."
end
```

---

## 7. Actions Specification

### 7.1 Action Context Table

| Context | `:complete` | `:terminate` | `:skip_to` |
|---------|------------|-------------|----------|
| Page | ✓ | ✓ | ✗ |
| Option block | ✓ | ✓ | ✓ |
| Matrix row block | ✓ | ✓ | ✓ (conditional) |
| Matrix column block | ✓ | ✓ | ✓ (conditional) |
| OpenEnded question | ✓ | ✓ | ✓ (with `:if_value`) |

### 7.2 All Action Syntax Forms

| Form | Example |
|------|---------|
| Unconditional terminal | `action :complete` |
| Unconditional terminal | `action :terminate` |
| Unconditional navigation | `action :skip_to, "page_name"` |
| Column condition (row block) | `action :skip_to, "page", :if_column, "value"` |
| Row condition (column block) | `action :skip_to, "page", :if_row, "row_name", "value"` |
| Multi-condition AND (row block) | `action :skip_to, "page", :if_column, "v", :if_row, "row", "v"` |
| Multi-cell AND (row block) | `action :skip_to, "page", :if_multiple_cells, [["r1","c1"],["r2","c2"]]` |
| Multi-cell OR (row block) | `action :skip_to, "page", :if_any_cells, [["r1","c1"],["r2","c2"]]` |
| Count condition | `action :skip_to, "page", :if_column_count, "value", :greater_than, 2` |
| Group condition | `action :terminate, :if_group, "group_name"` |
| Segment condition | `action :skip_to, "page", :if_segment, "segment_name"` |
| Value condition | `action :terminate, :if_value, "< 18"` |
| Value condition (list) | `action :skip_to, "page", :if_value, "== val1, val2"` |

### 7.3 Conditional Operators

See §3.10 (comparison operators for `:if_value`) and §3.11 (count operators for `:if_column_count`).

### 7.4 Multiple Actions — Evaluation Rules

1. Actions are evaluated in definition order.
2. Each action is evaluated independently.
3. The first navigation action whose condition is met determines survey flow.
4. `:terminate` and `:complete` take precedence over `:skip_to`.

```ruby
OpenEnded "income" do
  text "Annual household income?"
  numeric :min_value, 0

  action :terminate, :if_value, "< 10000"            # Checked first
  action :skip_to, "high_income", :if_value, "> 150000"  # Checked second
  action :skip_to, "standard", :if_value, ">= 10000"     # Fallback
end
```

### 7.5 Option Block Syntax

Use `do/end` blocks with options only when defining actions or `anchored`. Plain options need no block.

```ruby
# No block needed
option "Red"

# Block required for action or anchored
option "Not satisfied" do
  action :skip_to, "feedback"
end

option "None of the above" do
  anchored
end
```

Special options (`add_none_option`, `add_other_option`, `add_dont_know_option`) also accept an optional block with the same attributes — useful for routing or segmentation off a standard answer:

```ruby
SingleSelect "satisfaction" do
  text "How satisfied are you?"
  option "Very satisfied"
  option "Neutral"
  add_none_option do
    segment "unanswered"
    action :skip_to, "followup"
  end
  add_other_option do
    segment "other_selected"
  end
end
```

Without a block, these behave as simple boolean flags (the option is appended with its default text and no action).

### `code_as` — separate display text from stored value

Options can store a different value than they display using `code_as`. The display text shown to the respondent is unchanged; the coded value is what gets persisted and sent in the sync payload.

```ruby
# Keyword arg — clean for simple cases
option "Not at all satisfied", code_as: 1
option "Somewhat satisfied",   code_as: 2
option "Very satisfied",       code_as: 3

# Block form — when combining with action or segment
option "Not at all satisfied" do
  code_as 1
  action :skip_to, "followup"
end
```

When a downstream question uses `options_from`, the coded values carry forward — dynamically created options in the dependent question will also code as the same values.

---

## 8. Segmentation Specification

### 8.1 Segment Creation Forms

| Context | Syntax | Description |
|---------|--------|-------------|
| Option block | `segment "name"` | Activates when this option is selected |
| Matrix row block | `segment "name", :if_column, "val"` | Activates when row's column equals value |
| Matrix row block | `segment "name", :if_column, "v1", "v2"` | Activates if column equals any listed value (OR) |
| Matrix column block | `segment "name", :if_row, "row_name", "val"` | Activates when named row's selection equals value |
| Matrix column block | `segment "name", :if_row, "r1", "r2"` | Activates if any listed row selects this column (OR) |

### 8.2 Conditional Display Syntax

```ruby
# Single segment condition
show_only_if "segment_name"

# Multiple segment conditions (OR logic — shown if respondent is in ANY)
show_only_if "segment_a", "segment_b", "segment_c"

# Group reference (AND logic for the group's segments)
show_only_if "group_name"

# Mixed: group AND, plus extra segment OR
show_only_if "group_name", "segment_name"
```

### 8.3 Group-Segment Definition

```ruby
# Segment group: all listed names must be active segment names
group "engaged_users", "power_users", "frequent_vet_user"

# Page group that is also used in conditional display
group "premium_section", "premium_page_1", "premium_page_2"
```

### 8.4 Logic Evaluation Rules

| Evaluation | Rule |
|------------|------|
| Within a group | ALL referenced segments must be active (AND logic) |
| Between `show_only_if` arguments | ANY argument being satisfied shows the element (OR logic) |
| Multiple `:if_column` values in segment | ANY listed value matches (OR logic) |
| Multiple conditions within one action | ALL conditions must be true (AND logic) |

---

## 9. Dynamic Content

### 9.1 Text Piping

Placeholder syntax: `{{question_name}}`

Placeholders are replaced at runtime with the respondent's previous answer.

#### Supported Locations

- Question `text` attribute
- `option` label text
- `Notification` `text` attribute
- Matrix `row` and `column` labels

#### Display Format by Source Type

| Source Question Type | Display |
|---------------------|---------|
| `SingleSelect`, `Dropdown`, `ButtonRating` | Selected option text |
| `MultiSelect`, `ButtonCheckbox` | Comma-separated list with "and" before last item |
| `OpenEnded`, `Discussion`, `Number` | Typed text value |

#### Rules

| Rule | Description |
|------|-------------|
| Forward reference | Referenced question must appear **before** the piping question in survey flow |
| Existence | Referenced question must exist in the survey (validated at parse time) |
| Type restriction | None — any question type may be referenced |
| Unresolved placeholder | Shows `___` until source question is answered |
| Reactive updates | Piped text updates automatically if respondent changes their earlier answer |
| XSS prevention | Substitution uses `textContent` (not `innerHTML`) |

```ruby
SingleSelect "favorite_fruit" do
  text "What is your favorite fruit?"
  option "Apple"
  option "Banana"
  option "Cherry"
end

SingleSelect "fruit_rating" do
  text "You selected {{favorite_fruit}}. How would you rate it?"
  option "Excellent"
  option "Good"
  option "Poor"
end
```

### 9.2 Dynamic Options

Populate a question's options, rows, or columns from a previous question's selections.

| Attribute | Used With | Source Question Requirements |
|-----------|-----------|------------------------------|
| `options_from` | SingleSelect, MultiSelect | Any selection-type question that appears earlier in survey flow |
| `rows_from` | Matrix questions | Any selection-type question that appears earlier in survey flow |
| `columns_from` | Matrix questions | Any selection-type question that appears earlier in survey flow |

#### Rules

| Rule | Description |
|------|-------------|
| Forward reference | Source question must appear **before** the dependent question |
| Source types | SingleSelect, MultiSelect, Dropdown, and matrix questions |
| Empty selections | Shows informational message when source has no selections |
| Static + dynamic | `rows_from`/`columns_from` replaces static rows/columns entirely |

```ruby
MultiSelect "owned_products" do
  text "Which products do you own?"
  option "Laptop"
  option "Phone"
  option "Tablet"
end

SingleSelectMatrix "product_satisfaction" do
  text "Rate your satisfaction with each product you own."
  rows_from "owned_products"
  required

  column "Very Satisfied"
  column "Satisfied"
  column "Neutral"
  column "Dissatisfied"
end
```

---

## 10. Rich Text Formatting

Markdown is automatically detected and processed in all text values.

| Markdown | HTML Output | Example Input | Example Output |
|----------|-------------|---------------|----------------|
| `**bold**` | `<strong>` | `**important**` | **important** |
| `*italic*` | `<em>` | `*note*` | *note* |
| `_italic_` | `<em>` | `_note_` | *note* |
| `__underline__` | `<u>` | `__required__` | <u>required</u> |
| `` `code` `` | `<code>` | `` `submit` `` | `submit` |
| `~~strikethrough~~` | `<del>` | `~~old~~` | ~~old~~ |

- Applies to: `text`, `title`, `option` labels, `row`/`column` labels, `Notification` text
- Text without markdown patterns is HTML-escaped automatically
- Markdown styles can be combined within a single string

---

## 11. Language Constraints and Validation Rules

### 11.1 Structural Requirements

| Rule | Description |
|------|-------------|
| Minimum pages | Survey must have at least one `page` |
| Page names unique | Each page name must be unique within the survey |
| Page defined once | Each page may be defined only once |
| Question names unique | Each question name must be unique across the entire survey |
| FillInTheBlank blank | `text` must contain at least one `___` sequence |
| Completion required | Last page must contain a visible `Notification` question (auto-generated with warning if missing) |
| Matrix minimum | Matrix questions must have at least one `row` and one `column` |

### 11.2 Value Constraints

| Rule | Description |
|------|-------------|
| `min_selections` ≤ `max_selections` | MultiSelect and MultiSelectMatrix |
| `max_selections` ≤ option count | MultiSelect |
| `sum_to` must be positive | ConstantSum |
| `scale` range 1–10 | Bipolar questions |
| `min_value` < `max_value` | Number and Slider |
| SingleSelect min options | Must have at least 2 static `option` entries |
| MultiSelect min options | Must have at least 2 static `option` entries |
| Dropdown min options | Must have at least 1 `option` entry |
| Ranking min items | Must have at least 2 `item` entries |
| TextHighlighter min categories | `categories` array must contain at least 2 items |
| ConjointChoice min profiles | Must have at least 2 `profile` blocks |
| ConjointChoice min attributes | Must have at least 1 `attribute` declaration |

### 11.3 Reference Requirements

| Rule | Description |
|------|-------------|
| `skip_to` targets | Must reference an existing page or group name |
| `:if_column` values | Must match an existing column label in the same matrix |
| `:if_row` values | Must match an existing row label in the same matrix |
| Group page references | Must point to previously defined pages |
| `options_from` source | Must reference an existing question that appears earlier in survey flow |
| `rows_from` source | Must reference an existing question that appears earlier in survey flow |
| `columns_from` source | Must reference an existing question that appears earlier in survey flow |
| Text pipe `{{name}}` | Must reference an existing question that appears earlier in survey flow |
| `show_only_if` names | Must reference defined segment or group names |

### 11.4 Naming Rules

| Rule | Description |
|------|-------------|
| Format | snake_case recommended: `question_name` not `QuestionName` |
| Characters | Letters, digits, underscores; no spaces or special characters |
| Uniqueness | Unique across the entire survey file |
| Descriptive | Prefer `"satisfaction_level"` over `"q1"` |
| Prefix by section | `"demo_age"`, `"demo_location"` for easier maintenance |

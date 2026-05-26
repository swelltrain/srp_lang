© 2026 SRP Solutions SRP LLC. All rights reserved.

# SRP Survey DSL Language Reference — XForm Edition

Syntax specification for the SRP Survey DSL **as it applies to XForm export**. This document omits features that have no XForm equivalent; for the complete DSL reference, see `language_reference.md`.

---

## Table of Contents

1. [Document Conventions](#1-document-conventions)
2. [Grammar Overview](#2-grammar-overview)
3. [Token Reference Tables](#3-token-reference-tables)
4. [Survey Structure](#4-survey-structure)
5. [Common Question Attributes](#5-common-question-attributes)
6. [Question Type Specifications](#6-question-type-specifications)
7. [Actions Specification](#7-actions-specification)
8. [Segmentation Specification](#8-segmentation-specification)
9. [Language Constraints and Validation Rules](#9-language-constraints-and-validation-rules)

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
SURVEY     ::= COMMENT* (PAGE | REPEAT)*

PAGE       ::= 'page' String 'do'
                 title String
                 'text'? String  # Shorthand for Notification
                 QUESTION*
               'end'

REPEAT     ::= 'repeat' String 'do'
                 ('label' String)?
                 ('min' Integer)?
                 ('max' Integer)?
                 QUESTION+
               'end'

QUESTION   ::= QuestionType String 'do'
                 COMMON_ATTRS
                 TYPE_SPECIFIC_ATTRS
                 ACTION*
               'end'

OPTION     ::= 'option' String ('do' OPTION_ATTRS 'end')?

ROW        ::= 'row' String

COLUMN     ::= 'column' String

ACTION     ::= 'action' ACTION_TYPE (CONDITIONAL)?
```

---

## 3. Token Reference Tables

### 3.1 Core Structure Tokens

| Token | Type | Description | Example |
|-------|------|-------------|---------|
| `page` | Block keyword | Define a survey page | `page "welcome" do` |
| `repeat` | Block keyword | Define a repeating block of questions | `repeat "child" do` |
| `title` | Attribute (String) | Display title for page | `title "Welcome"` |
| `text` | Attribute (String) | Page-level Notification shorthand | `text "Your question"` |
| `action` | Function | Control survey flow | `action :complete` |
| `show_only_if` | Function | Conditional display (segment-based only) | `show_only_if "segment_name"` |
| `label` | Attribute (String, repeat only) | Human-readable label for "Add another?" prompt | `label "Child"` |
| `min` | Attribute (Integer, repeat only) | Minimum instances required | `min 1` |
| `max` | Attribute (Integer, repeat only) | Maximum instances allowed | `max 5` |

### 3.2 Supported Question Types

| Token | Block | Description | XForm Mapping |
|-------|-------|-------------|---------------|
| `SingleSelect` | Block | Radio button question | `<select1>` |
| `MultiSelect` | Block | Checkbox question | `<select>` |
| `Dropdown` | Block | Native dropdown select | `<select1 appearance="minimal">` |
| `ButtonRating` | Block | Labeled button scale | `<select1 appearance="likert">` |
| `ButtonCheckbox` | Block | Toggle-button multi-select | `<select>` with buttons |
| `ThisOrThat` | Block | Binary forced-choice | `<select1>` with two items |
| `SingleSelectMatrix` | Block | Grid with one selection per row | Repeated `<select1>` per row |
| `MultiSelectMatrix` | Block | Grid with multiple selections per row | Repeated `<select>` per row |
| `OpenEnded` | Block | Single-line text input | `<input type="string">` |
| `Discussion` | Block | Multi-line text area | `<input appearance="multiline">` |
| `Number` | Block | Dedicated numeric input | `<input type="int">` or `<input type="decimal">` |
| `FillInTheBlank` | Block | Sentence completion | `<input type="string">` |
| `Rating` | Block | Star/numeric rating scale | `<select1 appearance="likert">` |
| `Slider` | Block | Range slider | `<range>` |
| `NPS` | Block | Net Promoter Score | `<select1 appearance="likert">` (0–10) |
| `Ranking` | Block | Drag-and-drop rank ordering | `<odk:rank>` |
| `DatePicker` | Block | Date selection | `<input type="date">` |
| `Notification` | Block | Display-only text | `note` (read-only label) |
| `Age` | Block | Date-of-birth (month/day/year) | `<input type="int">` with constraint |
| `PhoneNumber` | Block | Formatted phone number | `<input type="string">` with constraint |
| `MediaUpload` | Block | File upload | `<upload>` with mediatype |

### 3.3 Question Content Tokens

| Token | Type | Used With | Description |
|-------|------|-----------|-------------|
| `option` | Block/Function | SingleSelect, MultiSelect, Dropdown | Define an answer choice |
| `button` | Function | ButtonCheckbox, ButtonRating | Define a button label |
| `this` | Attribute (String) | ThisOrThat | Left/first choice label |
| `that` | Attribute (String) | ThisOrThat | Right/second choice label |
| `row` | Function | Matrix types, Slider | Define a row |
| `column` | Function | Matrix types | Define a column |
| `item` | Function | Ranking | Define a sortable item |
| `segment` | Function | Option, Matrix row/column block | Activate a named segment |

### 3.4 Boolean Attribute Tokens

| Token | Used With | Description |
|-------|-----------|-------------|
| `required` | All question types | Make question mandatory |
| `hidden` | All question types | Render question invisible; exists in logic only |
| `code_as` | option block | Store a different value than display text |
| `add_none_option` | SingleSelect, MultiSelect, Dropdown, Matrix, SingleSelectBipolar | Append "None" option |
| `add_dont_know_option` | SingleSelect, MultiSelect, Matrix, SingleSelectBipolar | Append "Don't Know" option |
| `add_other_option` | SingleSelect, MultiSelect, Dropdown, MultiSelectMatrix | Append "Other (specify)" option |

### 3.5 Value Attribute Tokens

| Token | Type | Used With | Default | Description |
|-------|------|-----------|---------|-------------|
| `min_selections` | Integer | MultiSelect, ButtonCheckbox | — | Minimum required selections |
| `max_selections` | Integer | MultiSelect, ButtonCheckbox | — | Maximum allowed selections |
| `min_selections_per_row` | Integer | MultiSelectMatrix | — | Minimum selections per row |
| `max_selections_per_row` | Integer | MultiSelectMatrix | — | Maximum selections per row |
| `number_of_ranks` | Integer | Rating | `5` | Number of rating levels |
| `scale` | Integer | SingleSelectBipolar (not in XForm) | `5` | Number of scale points (1–10) |
| `min_value` | Float | Number, Slider | — | Minimum allowed value |
| `max_value` | Float | Number, Slider | — | Maximum allowed value |
| `allow_decimals` | Boolean | Number | `true` | Accept decimal input |
| `unit_label` | String | Number | — | Unit suffix shown after input |
| `placeholder` | String | Number, Dropdown | — | Input placeholder text |
| `step` | Float | Slider | `1` | Increment between values |
| `default_value` | Float | Slider | — | Initial slider position |
| `min_label` | String | Slider, NPS | — | Label at minimum end |
| `max_label` | String | Slider, NPS | — | Label at maximum end |
| `min_date` | ISO8601 | DatePicker | — | Earliest selectable date |
| `max_date` | ISO8601 | DatePicker | — | Latest selectable date |

### 3.6 Action Tokens

| Token | Type | Description | XForm Mapping |
|-------|------|-------------|---------------|
| `:complete` | Action symbol | End survey as successfully completed | Converted to skip to end-of-survey |
| `:terminate` | Action symbol | End survey immediately (disqualify) | Converted to skip to end-of-survey |
| `:skip_to` | Action symbol | Navigate to named page | XForm relevance-based skipping |

### 3.7 Conditional Tokens

| Token | Type | Description |
|-------|------|-------------|
| `:if_column` | Condition | Matrix row action: fires when column equals value |
| `:if_row` | Condition | Matrix column action: fires when named row equals value |
| `:if_value` | Condition | OpenEnded action: fires when input matches expression |
| `:if_segment` | Condition | Action fires when named segment is active |

### 3.8 Segmentation Tokens

| Token | Type | Description |
|-------|------|-------------|
| `segment` | Function | Activate a named segment (inside option or matrix row/column block) |
| `show_only_if` | Function | Conditional display based on active segments |

---

## 4. Survey Structure

### 4.1 File Format

- Extension: `.srp`
- Encoding: UTF-8
- No top-level wrapper; file is a sequence of `page` and `repeat` declarations
- Top-of-file comments apply to the whole survey

### 4.2 Page

```
page String do
  title       String          # Required
  text        String?         # Shorthand — creates a Notification question
  QUESTION*
end
```

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `title` | String | **Required** | — | Display heading |
| `text` | String | Optional | — | Shorthand; auto-creates a Notification with this text |

---

### 4.3 Repeat

A `repeat` block defines a set of questions the respondent fills out N times (e.g., once per child).

```ruby
repeat "child" do
  label "Child"    # drives "Add another Child?" and "Child 1 / Child 2" headings
  min 1            # at least one instance required (default: 0)
  max 5            # no more than 5 instances (default: unlimited)

  OpenEnded "child_first_name" do
    text "First name"
    required
  end

  DatePicker "child_dob" do
    text "Date of birth"
  end
end
```

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| Name (1st arg) | String | **Required** | — | Unique identifier; must not collide with page names |
| `label` | String | Optional | repeat name | Human-readable label used in "Add another [label]?" |
| `min` | Integer | Optional | `0` | Minimum number of instances (>= 0) |
| `max` | Integer | Optional | `nil` (unlimited) | Maximum number of instances (> 0, >= `min`) |
| Questions | Block | **Required** | — | One or more question definitions |

**Constraints:**
- A repeat must contain at least one question.
- `min` must be >= 0.
- `max` must be > 0 and >= `min` when set.
- The repeat name must be unique across all page names and repeat names.
- Question names inside a repeat must be globally unique.

---

## 5. Common Question Attributes

These attributes apply to every question type unless noted otherwise.

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| Name (1st arg) | String | **Required** | — | Unique identifier; snake_case |
| `text` | String | **Required** | — | Question prompt displayed to respondent |
| `required` | Boolean | Optional | false | Respondent must answer before advancing |
| `hidden` | Boolean | Optional | false | Question exists in logic but is not rendered |
| `show_only_if` | String+ | Optional | always shown | Display only when respondent is in named segment(s) |

### 5.1 show_only_if Behavior

`show_only_if` accepts one or more segment names with OR logic: the question is shown if the respondent is in ANY listed segment.

```ruby
OpenEnded "follow_up" do
  text "Please explain further."
  show_only_if "dissatisfied", "other_selected"
end
```

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
| `numeric` | Function | Optional | — | Enables numeric input; accepts `:min_value N`, `:max_value N`, or both |
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
end
```

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `min_value` | Float | Optional | — | Minimum accepted value |
| `max_value` | Float | Optional | — | Maximum accepted value |
| `allow_decimals` | Boolean | Optional | true | Accept decimal input |
| `unit_label` | String | Optional | — | Suffix shown after the input field |
| `placeholder` | String | Optional | — | Input placeholder text |

```ruby
Number "weight" do
  text "What is your current weight?"
  min_value 0.1
  max_value 500.0
  unit_label "lbs"
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

No type-specific attributes.

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

### 6.2 Selection

#### SingleSelect

Single-choice question. Supports radio buttons and dropdown display styles.

```
SingleSelect String do
  text String
  style (radio | dropdown)?
  add_none_option?
  add_dont_know_option?
  add_other_option?
  option String (do OPTION_BLOCK end)?*
end
```

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `style` | Symbol | Optional | `radio` | `radio` or `dropdown` |
| `add_none_option` | Boolean or Block | Optional | false | Append "None" option |
| `add_dont_know_option` | Boolean or Block | Optional | false | Append "Don't Know" option |
| `add_other_option` | Boolean or Block | Optional | false | Append "Other (please specify)" option |

Requires at least 2 `option` entries.

```ruby
SingleSelect "satisfaction" do
  text "How satisfied are you overall?"
  required

  option "Very satisfied"
  option "Satisfied"
  option "Neutral"
  option "Dissatisfied" do
    action :skip_to, "feedback_page"
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
  add_none_option?
  add_dont_know_option?
  add_other_option?
  option String (do OPTION_BLOCK end)?*
end
```

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `min_selections` | Integer | Optional | — | Minimum number of options to select |
| `max_selections` | Integer | Optional | — | Maximum number of options to select |

```ruby
MultiSelect "interests" do
  text "Select all topics that interest you."
  min_selections 1
  max_selections 4

  option "Technology"
  option "Business"
  option "Design"
  option "Marketing"
end
```

---

#### Dropdown

Native select element. Functionally identical to `SingleSelect` but rendered as a native `<select>`.

```
Dropdown String do
  text String
  placeholder String?
  add_none_option?
  add_other_option?
  option String*
end
```

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `placeholder` | String | Optional | — | Placeholder text shown before selection |
| `add_none_option` | Boolean or Block | Optional | false | Append "None" option |
| `add_other_option` | Boolean or Block | Optional | false | Append "Other (please specify)" option |

```ruby
Dropdown "country" do
  text "Select your country."
  option "United States"
  option "Canada"
  add_other_option
  required
end
```

---

#### ButtonCheckbox

Toggle-button multi-select.

```
ButtonCheckbox String do
  text String
  min_selections Integer?
  max_selections Integer?
  button String*
end
```

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `min_selections` | Integer | Optional | — | Minimum number of buttons to select |
| `max_selections` | Integer | Optional | — | Maximum number of buttons to select |

```ruby
ButtonCheckbox "features" do
  text "Which features are most important?"
  min_selections 1
  max_selections 3

  button "Price"
  button "Quality"
  button "Speed"
  required
end
```

---

#### ButtonRating

Scale rendered as a row of labeled buttons.

```
ButtonRating String do
  text String
  button String*
end
```

```ruby
ButtonRating "satisfaction" do
  text "How satisfied are you?"
  button "Very Dissatisfied"
  button "Dissatisfied"
  button "Neutral"
  button "Satisfied"
  button "Very Satisfied"
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
  required
end
```

---

#### Ranking

Drag-and-drop rank ordering of items.

```
Ranking String do
  text String
  item String*
end
```

```ruby
Ranking "priorities" do
  text "Rank these features from most to least important."

  item "Performance"
  item "Ease of Use"
  item "Price"
  item "Support"
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
  add_none_option?
  add_dont_know_option?
  row String*
  column String*
end
```

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `add_none_option` | Boolean or Block | Optional | false | Append "None" column |
| `add_dont_know_option` | Boolean or Block | Optional | false | Append "Don't Know" column |

```ruby
SingleSelectMatrix "service_ratings" do
  text "Rate each service aspect."
  required

  row "Customer Support"
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
  add_none_option?
  add_dont_know_option?
  add_other_option?
  row String*
  column String*
end
```

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `min_selections_per_row` | Integer | Optional | — | Minimum checkboxes per row |
| `max_selections_per_row` | Integer | Optional | — | Maximum checkboxes per row |
| `add_none_option` | Boolean or Block | Optional | false | Append "None" column |
| `add_dont_know_option` | Boolean or Block | Optional | false | Append "Don't Know" column |
| `add_other_option` | Boolean or Block | Optional | false | Append "Other (specify)" column |

```ruby
MultiSelectMatrix "brand_attributes" do
  text "Select all attributes that apply to each brand."
  max_selections_per_row 3

  row "Brand A"
  row "Brand B"

  column "Quality"
  column "Value"
  column "Innovation"
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
end
```

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `number_of_ranks` | Integer | Optional | `5` | Number of rating levels |

```ruby
Rating "overall_satisfaction" do
  text "How satisfied are you overall?"
  number_of_ranks 5
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
  min_label String?
  max_label String?
  row String*           # optional: makes multi-row slider
end
```

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `min_value` | Float | Optional | `0` | Minimum slider value |
| `max_value` | Float | Optional | `100` | Maximum slider value |
| `step` | Float | Optional | `1` | Increment between values |
| `default_value` | Float | Optional | `min_value` | Initial slider position |
| `min_label` | String | Optional | — | Label at minimum end |
| `max_label` | String | Optional | — | Label at maximum end |

```ruby
Slider "satisfaction" do
  text "How satisfied are you with our product?"
  min_value 0
  max_value 10
  step 1
  default_value 5
  min_label "Not at all"
  max_label "Extremely"
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
end
```

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `min_label` | String | Optional | `"Not at all likely"` | Label for score 0 |
| `max_label` | String | Optional | `"Extremely likely"` | Label for score 10 |

```ruby
NPS "recommend_score" do
  text "How likely are you to recommend us to a friend?"
  required
end
```

---

### 6.5 Address / Demographic Questions

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

### 6.6 Media

#### MediaUpload

File upload interface.

```
MediaUpload String do
  text String
  [required]
  [accept String]
  [max_files Integer]
  [multiple Boolean]
end
```

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `text` | String | — | — | Label shown above the upload area |
| `required` | Boolean | Optional | false | Respondent must select at least one file |
| `accept` | String | Optional | — | File type filter (e.g. `"image/*"`, `".pdf,.docx"`) |
| `max_files` | Integer | Optional | — | Maximum number of files |
| `multiple` | Boolean | Optional | `true` | Allow selecting multiple files; set to `false` for one only |

```ruby
MediaUpload "profile_photo" do
  text "Upload a profile photo."
  required
  accept "image/*"
  multiple false
end
```

---

### 6.7 Display

#### Notification

Display-only text. Cannot be marked `required`.

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

Explicit syntax:

```ruby
Notification "instructions" do
  text "Please read the following instructions."
end
```

---

### 6.8 DatePicker

Date selection input with optional date range constraints.

```
DatePicker String do
  text String
  min_date ISO8601?
  max_date ISO8601?
end
```

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `min_date` | ISO8601 | Optional | — | Earliest selectable date (format: `"YYYY-MM-DD"`) |
| `max_date` | ISO8601 | Optional | — | Latest selectable date |

```ruby
DatePicker "birth_date" do
  text "What is your date of birth?"
  min_date "1924-01-01"
  max_date "2006-12-31"
  required
end
```

---

## 7. Actions Specification

### 7.1 Action Context Table

| Context | `:complete` | `:terminate` | `:skip_to` |
|---------|------------|-------------|----------|
| Page | ✗ | ✗ | ✗ |
| Option block | ✓ | ✓ | ✓ |
| Matrix row block | ✓ | ✓ | ✓ (conditional) |
| Matrix column block | ✓ | ✓ | ✓ (conditional) |
| OpenEnded question | ✓ | ✓ | ✓ (with `:if_value`) |

**XForm Note:** `:complete` and `:terminate` are converted to skip-to-end-of-survey in XForm export.

### 7.2 All Action Syntax Forms

| Form | Example |
|------|---------|
| Unconditional terminal | `action :complete` |
| Unconditional terminal | `action :terminate` |
| Unconditional navigation | `action :skip_to, "page_name"` |
| Column condition (row block) | `action :skip_to, "page", :if_column, "value"` |
| Row condition (column block) | `action :skip_to, "page", :if_row, "row_name", "value"` |
| Multi-condition AND (row block) | `action :skip_to, "page", :if_column, "v", :if_row, "row", "v"` |
| Segment condition | `action :skip_to, "page", :if_segment, "segment_name"` |
| Value condition | `action :terminate, :if_value, "< 18"` |
| Value condition (list) | `action :skip_to, "page", :if_value, "== val1, val2"` |

### 7.3 Comparison Operators (for `:if_value`)

| Operator | Description | Supports Lists |
|----------|-------------|----------------|
| `<` | Less than (numeric) | No |
| `<=` | Less than or equal (numeric) | No |
| `>` | Greater than (numeric) | No |
| `>=` | Greater than or equal (numeric) | No |
| `==` | Equal to | Yes — `"== val1, val2"` |
| `!=` | Not equal to | Yes — `"!= val1, val2"` |

List semantics: `==` matches ANY value (OR); `!=` matches NONE of the values.

### 7.4 Multiple Actions — Evaluation Rules

1. Actions are evaluated in definition order.
2. Each action is evaluated independently.
3. The first navigation action whose condition is met determines survey flow.
4. `:terminate` and `:complete` take precedence over `:skip_to`.

```ruby
OpenEnded "income" do
  text "Annual household income?"
  numeric :min_value, 0

  action :terminate, :if_value, "< 10000"
  action :skip_to, "high_income", :if_value, "> 150000"
  action :skip_to, "standard", :if_value, ">= 10000"
end
```

### 7.5 Option Block Syntax

Use `do/end` blocks with options only when defining actions. Plain options need no block.

```ruby
# No block needed
option "Red"

# Block required for action or segment
option "Not satisfied" do
  action :skip_to, "feedback"
  segment "dissatisfied"
end
```

Special options also accept an optional block with the same attributes:

```ruby
SingleSelect "satisfaction" do
  text "How satisfied are you?"
  option "Very satisfied"
  option "Neutral"
  add_none_option do
    segment "unanswered"
    action :skip_to, "followup"
  end
end
```

#### `code_as` — separate display text from stored value

Options can store a different value than they display using `code_as`.

```ruby
# Keyword arg
option "Not at all satisfied", code_as: 1
option "Somewhat satisfied",   code_as: 2
option "Very satisfied",       code_as: 3

# Block form
option "Not at all satisfied" do
  code_as 1
  action :skip_to, "followup"
end
```

---

## 8. Segmentation Specification

### 8.1 Segment Creation Forms

| Context | Syntax | Description |
|---------|--------|-------------|
| Option block | `segment "name"` | Activates when this option is selected |
| Matrix row block | `segment "name", :if_column, "val"` | Activates when row's column equals value |
| Matrix row block | `segment "name", :if_column, "v1", "v2"` | Activates if column equals any value (OR) |
| Matrix column block | `segment "name", :if_row, "row_name", "val"` | Activates when row selects this column |
| Matrix column block | `segment "name", :if_row, "r1", "r2"` | Activates if any row selects this column (OR) |

### 8.2 Conditional Display Syntax

```ruby
# Single segment condition
show_only_if "segment_name"

# Multiple segment conditions (OR logic — shown if in ANY)
show_only_if "segment_a", "segment_b", "segment_c"
```

### 8.3 Logic Evaluation Rules

| Evaluation | Rule |
|------------|------|
| Between `show_only_if` arguments | ANY argument being satisfied shows the element (OR logic) |
| Multiple `:if_column` values in segment | ANY listed value matches (OR logic) |

---

## 9. Language Constraints and Validation Rules

### 9.1 Structural Requirements

| Rule | Description |
|------|-------------|
| Minimum pages | Survey must have at least one `page` |
| Page names unique | Each page name must be unique within the survey |
| Question names unique | Each question name must be unique across the entire survey |
| FillInTheBlank blank | `text` must contain at least one `___` sequence |
| Matrix minimum | Matrix questions must have at least one `row` and one `column` |

### 9.2 Value Constraints

| Rule | Description |
|------|-------------|
| `min_selections` ≤ `max_selections` | MultiSelect and MultiSelectMatrix |
| `max_selections` ≤ option count | MultiSelect |
| `min_value` < `max_value` | Number and Slider |
| SingleSelect min options | Must have at least 2 static `option` entries |
| MultiSelect min options | Must have at least 2 static `option` entries |
| Dropdown min options | Must have at least 1 `option` entry |
| Ranking min items | Must have at least 2 `item` entries |

### 9.3 Reference Requirements

| Rule | Description |
|------|-------------|
| `skip_to` targets | Must reference an existing page name |
| `:if_column` values | Must match an existing column label in the same matrix |
| `:if_row` values | Must match an existing row label in the same matrix |
| `show_only_if` names | Must reference defined segment names (no groups) |

### 9.4 Naming Rules

| Rule | Description |
|------|-------------|
| Format | snake_case recommended: `question_name` not `QuestionName` |
| Characters | Letters, digits, underscores; no spaces or special characters |
| Uniqueness | Unique across the entire survey file |
| Descriptive | Prefer `"satisfaction_level"` over `"q1"` |

# Dashmin MVP — Dev Request

## Goal

Implement the **MVP form generator** that maps DB schema (MySQL ≥8 / MariaDB ≥10.x) to HTML forms.
Only **structural success hints** are applied client-side. The **DB remains the only validation authority**.

---

## Requirements

### 1. Schema parsing

* Parse table definition (via `information_schema`).
* Extract for each column:

  * `column_name`, `data_type`, `character_maximum_length`, `numeric_precision/scale`, `is_nullable`, `column_default`, `extra` (auto\_increment, generated), `check_constraints`, `enum/set` members.

### 2. HTML control mapping

* **Numbers** (`INT`, `DECIMAL(p,s)`): → `input[type=number]` with `step` and `min/max` if explicit.
* **Dates/times** (`DATE`, `TIME`, `DATETIME`, `TIMESTAMP`): → `input[type=date|time|datetime-local]` with `min/max` if explicit.
* **Short text** (`CHAR/VARCHAR`): → `input[type=text]` with `maxlength`.
* **Long text** (`TEXT`): → `textarea`.
* **Enumerations** (`ENUM`): → `select` or `radio` depending on option count.
* **Multi-choice** (`SET`): → `select[multiple]`.
* **Boolean** (`TINYINT(1)` / `BOOLEAN`): → `select` with `0/1`, and blank option if nullable.
* **Server-managed** (`AUTO_INCREMENT`, `GENERATED`, `ON UPDATE`): omit from create form, render readonly or hidden on edit.
* **Defaults**: prefill `value` / `selected`.

### 3. Attribute rules

* `NOT NULL` and no `DEFAULT` → add `required`.
* `CHAR/VARCHAR(N)` → add `maxlength=N`.
* `DECIMAL(p,s)` → `step=10^-s`.
* `INT` → `step=1`.
* `CHECK` simple range (min/max) → apply `min`/`max`.
* Do **not** add `pattern` in MVP.
* Do **not** reimplement uniqueness, FKs, or complex checks.

### 4. Form generation

* Wrap controls in `<fieldset>` to allow grid/flex layout (no `<legend>`).
* Add `<label for=...>` for accessibility.
* For nullable fields: provide empty option in `select`.
* For booleans: always a `select` with explicit `0` / `1`.

### 5. Submission handling

* Do **not** expect browser to guarantee validity beyond the structural hints.
* Always trust DB validation on submit.
* On error (constraint violation), redisplay form with DB error message attached to field.

---

## Deliverables

* PHP functions that:

  1. Inspect schema (given a table name).
  2. Return HTML form markup following the above rules.
* No JavaScript required in MVP.
* Code must be minimal, procedural, and consistent with Dashmin’s architecture.

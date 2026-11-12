# Q3 — SPL Basics (Splunk Search Processing Language)

## Overview
- **SPL = Splunk Search Processing Language.**
- Every SPL query starts with a **base search** (what data to fetch), then chains
  commands using the pipe `|` (what to do with that data).
- Structure: `base search | command1 | command2 | ...`
- Data flows left to right — each pipe passes results to the next command.

---

## Command Reference

### 1. `search` — Filter Events by Keyword or Field

| Purpose | Filter events using keywords or field=value pairs |
|---|---|
| **Syntax** | `index=<index> [field=value] [keyword]` |

**Example:**    index=wineventlog EventCode=4625

- Searches the `wineventlog` index for failed logon events (Event ID 4625).
- This is the **base search** — it narrows the data before further processing.

---

### 2. `where` — Filter Using Eval-Style Expressions

| Purpose | Filter results **after** a pipe using calculated conditions |
|---|---|
| **Syntax** | `... | where <expression>` |

**Example:**   index=linux sourcetype=linux_secure "Failed password"
| stats count as failures by src_ip
| where failures > 5


- After counting failures per source IP, `where` keeps only IPs with more than 5 failures.
- Use `where` when you need **calculated comparisons** (>, <, !=, AND, OR) that
  `search` cannot express directly.

---

### 3. `stats` — Aggregate Data

| Purpose | Calculate statistics: count, sum, avg, dc (distinct count), min, max |
|---|---|
| **Syntax** | `... | stats <function> [as <name>] by <field>` |

**Example:**   index=wineventlog EventCode=4625
| stats count as failed_attempts by TargetUserName, src_ip
| sort -failed_attempts


- Counts failed logons per **username and source IP**, then sorts highest first.
- Essential for summarization, threshold detection, and dashboards.

---

### 4. `table` — Display Selected Fields as Columns

| Purpose | Show only the specified fields in a clean column layout |
|---|---|
| **Syntax** | `... | table <field1> <field2> ...` |

**Example:**  index=wineventlog EventCode=4625
| table _time, TargetUserName, src_ip, ComputerName, Message

- Displays timestamp, username, source IP, hostname, and raw message in a readable table.
- Use `table` for **reporting and triage notes** — it removes clutter.

---

### 5. `eval` — Create or Modify Fields

| Purpose | Compute new fields, transform values, apply conditional logic |
|---|---|
| **Syntax** | `... | eval <new_field> = <expression>` |

**Example:**   index=wineventlog EventCode=4625
| stats count as failures by src_ip
| eval is_bruteforce = if(failures > 5, "YES", "NO") 

- Creates a new field `is_bruteforce` that flags whether the failure count exceeds 5.
- Also useful for: date formatting, string concatenation, severity scoring,
  parsing, and case statements.

**Another common example:**     ... | eval severity = case(count > 20, "HIGH", count > 10, "MEDIUM", true(), "LOW")  


---

### 6. `timechart` — Time-Series Aggregation

| Purpose | Aggregate over time for trend visualization (line/bar charts) |
|---|---|
| **Syntax** | `... | timechart span=<interval> <function> by <field>` |

**Example:**    index=wineventlog EventCode=4625
| timechart span=5m count by src_ip   

- Shows failed logon counts in **5-minute buckets**, split by source IP.
- Renders as a multi-line chart — perfect for spotting spikes, brute force bursts,
  or time-based patterns.

---

### 7. `rex` — Extract Fields Using Regular Expressions

| Purpose | Extract new fields from raw text (`_raw`) using regex |
|---|---|
| **Syntax** | `... | rex field=<field> "<regex with (?<name>...)>"` |

**Example:**  index=linux sourcetype=linux_secure "Failed password"
| rex field=_raw "from (?<attacker_ip>\d+.\d+.\d+.\d+)"
| stats count by attacker_ip


- Extracts the attacker IP address from raw SSH log lines using regex.
- Use `rex` when the field is **not already parsed** (e.g., custom app logs,
  unstructured text).

---

## Command Summary Table

| Command | One-Line Purpose | Example Snippet |
|---|---|---|
| `search` | Filter events by keyword/field | `index=wineventlog EventCode=4625` |
| `where` | Filter with calculated conditions | `... | where count > 5` |
| `stats` | Aggregate (count, sum, avg, dc) | `... | stats count by src_ip` |
| `table` | Display selected fields as columns | `... | table _time, user, src_ip` |
| `eval` | Create/modify fields | `... | eval flag = if(count>5,"YES","NO")` |
| `timechart` | Time-series aggregation | `... | timechart span=5m count by src_ip` |
| `rex` | Extract fields via regex | `... | rex field=_raw "from (?<ip>\d+\.\d+\.\d+\.\d+)"` |

---

## Practical Workflow

1. **Start broad** → `index=... sourcetype=...` (search)
2. **Narrow** → add field filters or `| where ...`
3. **Aggregate** → `| stats count by ...`
4. **Transform** → `| eval ...` to create flags or scores
5. **Format** → `| table ...` for reports or `| timechart ...` for trends

**Example end-to-end query:**
   index=wineventlog EventCode=4625
| stats count as failures by src_ip, TargetUserName
| where failures >= 5
| eval severity = if(failures > 20, "HIGH", "MEDIUM")
| table src_ip, TargetUserName, failures, severity
| sort -failures


- This single query: counts failures → filters ≥5 → assigns severity → displays clean table.

---

## Key Takeaways
- **search** and **where** = filtering (where is post-pipe, expression-based).
- **stats** and **timechart** = aggregation (table vs trend).
- **eval** and **rex** = field creation (calculated vs regex-extracted).
- **table** = presentation.
- Chain commands with `|` — each command transforms the data for the next.
- Start simple, add pipes incrementally, and verify results at each step.
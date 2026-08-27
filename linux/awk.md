Here is a clean, structured set of **AWK Command Notes** organized step-by-step from absolute basics to practical Linux/DevOps usage.

---

# AWK Command Cheatsheet & Notes

## 1. Core Syntax & Mental Model

```bash
awk 'pattern { action }' filename

```

* **Single Quotes (`'...'`):** Everything inside single quotes is the AWK script.
* **Braces (`{...}`):** Enclose the **action** you want to perform on each line.
* **Fields (`$1, $2, ...`):** AWK automatically breaks each line into columns (fields) separated by whitespace.

---

## 2. Basic Printing & Field Selection

| Command | What it does | Notes |
| --- | --- | --- |
| `awk '{print}' access.log` | Prints all lines in the file | Same as `cat access.log` |
| `awk '{print $0}' access.log` | Prints the entire line | `$0` represents the whole line |
| `awk '{print $1}' access.log` | Prints 1st column | Usually the IP address in web logs |
| `awk '{print $4}' access.log` | Prints 4th column | Usually the timestamp in web logs |
| `awk '{print $1, $4}' access.log` | Prints 1st and 4th columns | Comma adds a space between output fields |
| `awk '{print $NF}' access.log` | Prints the **last** column | `$NF` = Number of Fields in current line |
| `awk '{print $(NF-1)}' access.log` | Prints the second-to-last column | Useful when line length varies |

---

## 3. Built-in Variables (Must-Know)

* **`$0`**: Entire current line.
* **`$1, $2, $3...`**: Individual columns.
* **`NF`**: **N**umber of **F**ields (total columns in current line).
* **`NR`**: **N**umber of **R**ecords (current line/row number).
* **`FS`**: Field Separator (default is space or tab).
* **`OFS`**: Output Field Separator (default is space).

### Examples with Built-in Variables:

```bash
# Print line numbers along with the line content
awk '{print NR, $0}' access.log

# Skip line 1 (header row) in a file
awk 'NR > 1 {print $1, $2}' users.csv

# Print line number and total column count for each line
awk '{print "Line:", NR, "Total Columns:", NF}' access.log

```

---

## 4. Custom Delimiters (CSV / Custom Files)

By default, AWK splits lines by **spaces or tabs**. Use `-F` to specify a custom delimiter.

```bash
# Use comma (,) as delimiter for CSV files
awk -F ',' '{print $1, $3}' users.csv

# Use colon (:) as delimiter for /etc/passwd
awk -F ':' '{print $1, $6}' /etc/passwd

# Print username and home directory with custom output formatting
awk -F ':' '{print "User: " $1 " ---> Home: " $6}' /etc/passwd

```

---

## 5. Filtering & Matching (Patterns)

You can filter lines **before** performing an action.

### A. Number Comparisons

```bash
# Print lines where the 9th column (HTTP Status Code) equals 200
awk '$9 == 200 {print $1, $7}' access.log

# Print lines where column 4 (Memory size) is greater than 2048
awk '$4 > 2048 {print $1, $4}' metrics.txt

```

### B. String / Regular Expression Matching (`~` and `!~`)

```bash
# Print lines where column 9 contains "404"
awk '$9 ~ /404/ {print $1, $7, $9}' access.log

# Print lines where column 9 does NOT start with 2
awk '$9 !~ /^2/ {print $1, $7, $9}' access.log

# Match anywhere in the line
awk '/ERROR/ {print $0}' app.log

```

---

## 6. Execution Blocks: `BEGIN` and `END`

* **`BEGIN { ... }`**: Runs **once before** reading the file.
* **`END { ... }`**: Runs **once after** processing all lines in the file.

```bash
# Print a header and footer around data
awk 'BEGIN {print "--- LOG START ---"} {print $1, $9} END {print "--- LOG END ---"}' access.log

# Calculate total sum of column 10 (bytes transferred)
awk '{sum += $10} END {print "Total Bytes Transferred:", sum}' access.log

```

---

## 7. Quick Reference Cheat Sheet

```
+-----------------------------------------------------------------------+
|  awk -F ','  'NR > 1 && $3 == "200"  {print $1, $NF}'  access.csv     |
+-----+-----+   ----------+----------   -------+------+  ----+-----     |
|     |     |             |                    |              |         |
|     |     |             |                    |              +-- Target File
|     |     |             |                    +-- Action (What to print)
|     |     |             +-- Pattern / Filter (Condition)
|     |     +-- Custom Delimiter (Comma)
|     +-- AWK Tool Invocation
+-----------------------------------------------------------------------+

```

### Short Notes: `^` and `\.` in Regex

```text
^   → Start of the line
\.  → A literal dot (.)
```

So:

```regex
^\.
```

means:

> **The line starts with a dot `.`**

Example:

```text
.bashrc    ✓
.config    ✓
hello.txt  ✗
abc.def    ✗
```

**Remember:**

* `^` = **starts with**
* `\.` = **actual dot** (because `.` normally means any character)

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

===========================================================================================================================================================================================================
### Short Notes: `*` with `ls`

```bash
ls *.auth
```

→ Lists files that **end with `.auth`**

Example:

```text
user.auth     ✓
config.auth   ✓
user.txt      ✗
```

```bash
ls .env*
```

→ Lists files that **start with `.env`**

Example:

```text
.env
.env.auth
.env.coreai
.env.project
```

### Easy rule to remember

```text
*.auth   → ends with .auth
.env*    → starts with .env
```

Here `*` means **any number of characters**.

⚠️ This is **shell globbing**, not regex.

# Docker CMD vs ENTRYPOINT

## CMD

`CMD` provides the **default command** for a container.

```dockerfile
FROM alpine

CMD ["echo", "Hello from CMD"]
```

### Build

```bash
docker build -t cmd-demo .
```

### Run

```bash
docker run --rm cmd-demo
```

### Output

```text
Hello from CMD
```

Here:

* CMD provides the **default command**
* If you do not give anything extra, Docker runs it

### Try to override CMD

```bash
docker run --rm cmd-demo date
```

### Output

```text
Fri Mar 7 10:00:00 UTC 2026
```

Now CMD is overridden.

So with CMD:

* no extra command → default runs
* extra command provided → default is replaced

---

## ENTRYPOINT Example — Main Process

**Dockerfile**

FROM alpine

ENTRYPOINT ["echo", "Hello from ENTRYPOINT"]

**Build**

docker build -t entrypoint-demo .

**Run**

docker run --rm entrypoint-demo

**Output**

Hello from ENTRYPOINT

Here ENTRYPOINT defines the **main process**.

This container is basically built to run:

echo "Hello from ENTRYPOINT"

---

**Try to override ENTRYPOINT normally**

docker run --rm entrypoint-demo date

**Output**

Hello from ENTRYPOINT date

Why?

Because with ENTRYPOINT, extra values
passed to docker run are treated as **arguments to ENTRYPOINT**, not as a
replacement.

So Docker runs something like:

echo "Hello from ENTRYPOINT" date

---

## What does `--rm` mean?

```bash
docker run --rm cmd-demo
```

`--rm` means Docker automatically removes the container after it stops.

Without `--rm`:

```text
Create → Run → Stop → Container remains
```

With `--rm`:

```text
Create → Run → Stop → Container is deleted
```

It is useful for temporary/test containers.

---

# CMD vs ENTRYPOINT

The easiest way to remember:

```text
CMD
 ↓
Default command
 ↓
Can be replaced
```

```text
ENTRYPOINT
 ↓
Main program
 ↓
Extra values are normally passed as arguments
```

---

## CMD + ENTRYPOINT Together

This is the most useful pattern to remember:

```dockerfile
FROM alpine

ENTRYPOINT ["echo"]
CMD ["Hello"]
```

Run:

```bash
docker run --rm demo
```

Result:

```text
Hello
```

Conceptually:

```text
ENTRYPOINT = echo
CMD        = Hello

docker run demo
       ↓
echo Hello
```

If we run:

```bash
docker run --rm demo World
```

Result:

```text
World
```

Conceptually:

```text
ENTRYPOINT = echo
CMD        = Hello

docker run demo World
       ↓
echo World
```

So:

```text
ENTRYPOINT = main program
CMD        = default argument(s)
```

---

# Interview Question

### What is the difference between CMD and ENTRYPOINT?

> **"Both are used to define what runs when a container starts. CMD provides a default command or default arguments, and it can easily be overridden when we run the container. ENTRYPOINT is used when we want to define the main program that the container should run. If we provide something after the image name, it is normally passed as an argument to the ENTRYPOINT instead of replacing it."**

### Easy example

```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
```

Normally:

```bash
docker run myapp
```

runs:

```text
python app.py
```

But:

```bash
docker run myapp test.py
```

runs:

```text
python test.py
```

---

## Remember

```text
ENTRYPOINT = main program
CMD        = default command/arguments
```

**CMD can be replaced. ENTRYPOINT normally stays and receives the provided arguments.**

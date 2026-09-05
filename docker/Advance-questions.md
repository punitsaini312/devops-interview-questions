This is a **very common Docker interview question**. The key concept is **Docker layer caching**.

### Interview Answer

We copy `requirements.txt` and install the dependencies **before** copying the rest of the application code because of **Docker's layer caching**.

For example:

```dockerfile
COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .
```

Docker builds an image in layers. Each instruction creates a layer, and Docker can reuse a layer if nothing relevant to that instruction has changed.

The dependencies usually don't change every time I modify my application code. So if I write:

```dockerfile
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
```

and later I change only `app.py`, Docker can reuse the cached `pip install` layer instead of installing all the Python dependencies again.

If I did this instead:

```dockerfile
COPY . .
RUN pip install -r requirements.txt
```

then **any change anywhere in my source code** could invalidate the `COPY . .` layer, causing Docker to run `pip install` again.

So the main reason is:

> **We copy and install `requirements.txt` first so Docker can cache the dependency-installation layer and avoid reinstalling dependencies when only application code changes. This makes Docker builds much faster.**

### The one-line interview version

> **"We copy `requirements.txt` and run `pip install` before `COPY . .` to take advantage of Docker's layer caching. Dependencies change less frequently than source code, so when only the source code changes, Docker can reuse the cached dependency layer instead of reinstalling everything."**

### Remember it like this

```text
requirements.txt
       ↓
   pip install
       ↓
   CACHE THIS ✅
       ↓
    COPY . .
       ↓
 application code
```

**Dependency changes → reinstall.**

**Only code changes → reuse cached dependencies.**




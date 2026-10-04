# File Path Traversal – Traversal Sequences Stripped Non-Recursively

## Observation

A shopping application loads product pictures using a filename parameter exposed in the URL.

**Initial Request:**
```
/image?filename=72.png
```

By changing the filename parameter to `22.png`, the application loads a different image, confirming that user input directly controls file selection.

---

## Initial Attempt: Standard Path Traversal

The first payload attempted:
```
../../../etc/passwd
```

**Result:** Blocked. The application filters out traversal sequences before processing the filename.

---

## Root Cause Analysis

The application implements a filter that removes `../` patterns, but critically, it processes the input **non-recursively** — meaning it filters only once and does not re-evaluate the modified string for new traversal sequences.

---

## Bypass Technique

To circumvent the non-recursive filtering, the payload was constructed to contain nested traversal sequences:

```
....//....//....//etc/passwd
```

### How the Bypass Works

**Step 1:** Input submitted:
```
....//....//....//etc/passwd
```

**Step 2:** Application removes `../` patterns:
```
....//   ← Contains ../
   ↓
   ../   ← New traversal sequence revealed
```

**Step 3:** Because filtering is non-recursive, the newly formed `../` is not removed.

**Step 4:** Final path after filter:
```
../../../etc/passwd
```

**Step 5:** Application resolves the path and reads the file:
```
/etc/passwd
```

---

## Result

The filter was successfully bypassed. The `/etc/passwd` file contents were retrieved and displayed in the application response.

---

## Key Takeaway

A security filter's mere presence does not guarantee safety. The critical factor is **how the application processes input**. Non-recursive filtering creates a vulnerability window where deliberately crafted input can regenerate blocked patterns after the filter executes.

**Security Principle:** Always apply filters recursively or validate the final output, not just the intermediate steps.

---

## Comparison: Normal vs. Bypass

| Approach | Payload | Result |
|----------|---------|--------|
| Normal Path Traversal | `../../../etc/passwd` | BLOCKED by filter |
| Bypass (Non-Recursive) | `....//....//....//etc/passwd` | Filter removes `../` → New `../` appears → SUCCEEDS |

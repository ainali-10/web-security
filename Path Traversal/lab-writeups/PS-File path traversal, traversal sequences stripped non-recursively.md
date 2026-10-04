# File Path Traversal – Traversal Sequences Stripped Non-Recursively

## Overview

This lab demonstrates a classic path traversal bypass that works even when a filter is present. The application strips traversal sequences from user input, but because it does so only once, a crafted payload can regenerate a traversal pattern and still reach sensitive files.

---

## What I Observed

The application loads product images via a `filename` parameter in the URL:

```text
/image?filename=72.png
```

I changed the parameter from:

```text
72.png
```

to:

```text
22.png
```

and the application returned a different image.

This confirms that the `filename` parameter is user-controlled and directly influences which file is served.

---

## Attempting the Standard Payload

Because this is a file path traversal challenge, I first tried the usual payload:

```text
../../../etc/passwd
```

This did not work.

The reason is that the application strips traversal sequences from the filename before using it. So the input is sanitized before the final file path is resolved.

---

## Bypassing the Filter

The key detail is that the filter is applied only once.

I used this payload:

```text
....//....//....//etc/passwd
```

The idea is that the application removes the inner traversal sequence while processing the input. The string contains repeated path traversal patterns hidden inside a larger string.

For example:

```text
....//
```

contains:

```text
../
```

When the application removes that pattern, it reveals another traversal sequence. Because the filter is not recursive, it does not process the modified input again.

That means the final file path effectively becomes:

```text
../../../etc/passwd
```

and the application then reads:

```text
/etc/passwd
```

---

## Response

The server returned the contents of `/etc/passwd`.

This confirms that the filter can be bypassed even though it removes the obvious `../` traversal sequence from the original input.

---

## Why This Works

The most important concept here is that the sanitization is performed **non-recursively**.

The application removes a traversal sequence once, but it does not keep checking the modified input for newly exposed sequences.

In other words:

```text
....//
```

contains `../`, so the application removes it:

```text
....//
   ↓
../
```

Now the traversal sequence exists again, but since the application does not run the filter a second time, the bypass succeeds.

---

## Result

The `/etc/passwd` file was successfully retrieved using:

```text
....//....//....//etc/passwd
```

This lab was solved.

---

## What I Learned

This lab reinforced an important point: simply having a filter does not mean an application is safe.

If the application removes `../` only once, an attacker may be able to construct input where removing the sequence exposes another traversal sequence. The main lesson is that we should analyze not only whether a filter exists, but also how the input is transformed and validated throughout the processing pipeline.

---

## Quick Comparison

```text
Normal payload
../../../etc/passwd
        ↓
     BLOCKED

Bypass payload
....//....//....//etc/passwd
        ↓
Filter removes ../ once
        ↓
../../../etc/passwd
        ↓
/etc/passwd
```

---

## Final Takeaway

Security filters must be reliable and carefully designed. A one-pass sanitization routine may still leave exploitable paths behind. In path traversal defenses, the safest approach is to validate the final resolved path and avoid relying on simplistic pattern stripping alone.

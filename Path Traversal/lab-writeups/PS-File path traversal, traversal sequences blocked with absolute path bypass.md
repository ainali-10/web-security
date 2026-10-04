# File Path Traversal — Traversal Sequences Blocked with Absolute Path Bypass

## What I Observed

Again we have a shopping application that loads product images using a filename parameter.

When I opened one of the product images in a tab I could see the filename in the URL:

```text
/image?filename=53.png
```

**[Screenshot 1: Original image request showing `filename=53.png`]**

Because the filename is controlled through the URL I wanted to see if changing it would load a different image.

I changed `53.png` to `32.png`. The application returned a different image.

**[Screenshot 2: Changed image request showing `filename=32.png` and the different image]**

Thus we know that the filename parameter controls which file the application loads.

---

## Trying Path Traversal

Because the filename parameter is user controlled, the next test is Path Traversal.

Normally we could try something like:

```text
../../../etc/passwd
```

The `../` sequences move backwards through directories until we reach the filesystem root.

However, in this lab the application blocks traversal sequences.

So the normal payload does not work.

**[Screenshot 3: Caido request showing the blocked `../../../etc/passwd` attempt]**

This means we need another way to specify the file we want.

---

## Bypassing the Filter

Instead of using `../` to move through directories, I gave the application the absolute path directly.

I intercepted the image request in Caido. Changed the filename parameter to `/etc/passwd`.

**[Screenshot 4: Caido request showing `filename=/etc/passwd`]**

There is no `../` in this payload.

Instead, `/etc/passwd` is a path starting from the filesystem root.

The application accepted it and returned the file contents.

**[Screenshot 5: Caido response showing the `/etc/passwd` contents]**

This means that even though the application blocks the traversal sequence, it still allows absolute paths.

---

## Why This Works

The difference from the previous lab is the payload.

In the simple lab we used:

```text
../../../etc/passwd
```

because we needed to escape the application's current directory.

Here the application blocks the `../` traversal sequence.

Instead we use:

```text
/etc/passwd
```

This directly points to the file from the filesystem root.

The filter blocks the traversal technique. It does not stop the application from accepting an absolute path.

---

## Result

The `/etc/passwd` file was successfully retrieved using an absolute path:

```text
/etc/passwd
```

The lab was solved.

---

## What I Learned

The main thing I learned from this lab is that blocking `../` does not automatically prevent Path Traversal.

If an application accepts a user-controlled file path, I should also test whether it accepts absolute paths.

So when the normal traversal payload is blocked, do not stop there.

Think about what the application is actually validating.

```
../  blocked
    ↓
Try absolute path
    ↓
/etc/passwd
    ↓
File successfully read
```

This illustrates a critical point: developers sometimes implement incomplete fixes. Blocking relative path traversal while still accepting absolute paths leaves the application vulnerable to exploitation.

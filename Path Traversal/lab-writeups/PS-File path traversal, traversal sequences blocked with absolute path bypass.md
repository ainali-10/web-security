# File Path Traversal — Traversal Sequences Blocked with Absolute Path Bypass

## What I Observed

Again we have a shopping application that loads product images using a filename parameter.
  <img width="1726" height="935" alt="2026-10-04_23-19" src="https://github.com/user-attachments/assets/14352eb3-6e8b-40c1-9d4d-a79294f0817f" />



When I opened one of the product images in a tab I could see the filename in the URL:

```text
/image?filename=53.png
```

<img width="1907" height="915" alt="2026-10-04_23-38" src="https://github.com/user-attachments/assets/303fd154-61e8-45f6-8053-471175b96e4f" />

Because the filename is controlled through the URL I wanted to see if changing it would load a different image.

I changed `53.png` to `32.png`. The application returned a different image.

<img width="1912" height="921" alt="2026-10-04_23-39" src="https://github.com/user-attachments/assets/73e7270e-ff8d-4f56-8935-fa0baf012eb0" />

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

<img width="1892" height="853" alt="2026-10-04_23-41" src="https://github.com/user-attachments/assets/3e87f667-d181-4803-be07-c3d418d74c6c" />

This means we need another way to specify the file we want.

---

## Bypassing the Filter

Instead of using `../` to move through directories, I gave the application the absolute path directly.

I intercepted the image request in Caido. Changed the filename parameter to `/etc/passwd`.

<img width="366" height="35" alt="2026-10-04_23-55" src="https://github.com/user-attachments/assets/90025328-6c90-4cb8-b952-0e3e3f9baa68" />

There is no `../` in this payload.

Instead, `/etc/passwd` is a path starting from the filesystem root.

The application accepted it and returned the file contents.

<img width="1906" height="809" alt="2026-10-04_23-42" src="https://github.com/user-attachments/assets/c11cd18e-b22b-4e3f-8bef-406eed996b89" />

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

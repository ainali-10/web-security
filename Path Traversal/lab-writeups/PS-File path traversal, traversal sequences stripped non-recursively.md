# File Path Traversal, Traversal Sequences Stripped Non-Recursively

## What I observed

We have the same type of shopping application where product images are loaded using a filename parameter.
  <img width="1766" height="884" alt="2026-10-05_00-41" src="https://github.com/user-attachments/assets/7bfb537d-9291-48ef-a18f-93a151ed3717" />




I opened one of the product images in a new tab and saw the filename in the URL:

```text
/image?filename=72.png
```
<img width="1840" height="853" alt="2026-10-05_00-42" src="https://github.com/user-attachments/assets/62c9a4cc-d99a-4fef-b081-2fd23931f845" />

Since the filename is controlled through the URL, I changed:

```text
72.png
```

to:

```text
22.png
```

The application returned a different image.

<img width="1358" height="820" alt="2026-10-05_00-43" src="https://github.com/user-attachments/assets/fa1212ab-0bf1-45ab-8003-3fd8fe303474" />

So again, we know that the `filename` parameter is user controlled and is being used to decide which file the application loads.

## Trying Path Traversal

Since this is a Path Traversal lab, I first tried the normal traversal payload:

```text
../../../etc/passwd
```

But this didn't work.

The reason is that this application strips traversal sequences from the filename before using it.

So if we send:

```text
../../../etc/passwd
```

the application removes the `../` sequences.

This means we need to find a way around the filtering instead of simply using the normal payload.

## Bypassing the filter

The important part here is that the application strips the traversal sequence **non-recursively**.

So I used:

```text
....//....//....//etc/passwd
```

<img width="1904" height="828" alt="2026-10-05_00-45" src="https://github.com/user-attachments/assets/df2eff9b-55db-41b7-84af-2fc9d88eb93c" />

The idea is that the application removes the inner `../` pattern while processing the input.

For example:

```text
....//
```

contains:

```text
../
```

After that sequence is removed, another traversal sequence is exposed:

```text
../
```

Because the application only performs the filtering once, it doesn't remove this newly exposed traversal sequence.

So the resulting path effectively becomes:

```text
../../../etc/passwd
```

The application then uses that path to access `/etc/passwd`.

## Response

The server returned the contents of `/etc/passwd`.

<img width="1900" height="772" alt="2026-10-05_00-46" src="https://github.com/user-attachments/assets/e4434586-d254-493e-92dc-b8b508a6da4a" />

This confirms that the filter could be bypassed even though it was removing the normal `../` traversal sequence.

## Why this works

The important word here is **non-recursively**.

The application removes the traversal sequence once, but it doesn't keep checking the modified input.

So:

```text
....//
```

contains:

```text
../
```

The application removes that part:

```text
....//
  ↓
../
```

Now a traversal sequence exists again, but the application doesn't process the input a second time.

That's why the bypass works.

## Result

The `/etc/passwd` file was successfully retrieved using:

```text
....//....//....//etc/passwd
```

The lab was solved.

## What I learned

This lab showed me that simply having a filter doesn't mean the application is safe.

If the application removes `../` only once, we can sometimes construct an input where removing the sequence exposes another traversal sequence.

The main thing to look for is **how the application processes and transforms the input**, not just whether it has a filter.

```text
Normal payload
../../../etc/passwd
        ↓
     BLOCKED

Bypass
....//....//....//etc/passwd
        ↓
Filter removes ../
        ↓
../../../etc/passwd
        ↓
/etc/passwd
```

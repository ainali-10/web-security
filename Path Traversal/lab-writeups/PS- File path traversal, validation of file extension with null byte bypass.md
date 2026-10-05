# File Path Traversal, Validation of File Extension with Null Byte Bypass

## What I observed

This time the application is loading product images through a filename parameter.

I opened one of the product images in a tab and saw:

```text id="a6m7w2"

/image?filename=53.jpg

```
<img width="1665" height="890" alt="2026-10-05_23-44" src="https://github.com/user-attachments/assets/f1eb1170-8dc7-46cd-a9f1-e2b9d5ecf50e" />

Since the filename is controlled through the URL I changed:

```text id="yqv5da"

53.jpg

```

to:

```text id="c0p7by"

31.jpg

```

The application returned an image.
<img width="1549" height="859" alt="2026-10-05_23-44_1" src="https://github.com/user-attachments/assets/f532ed76-aea7-4d9b-80d2-e48d185276da" />

So the `filename` parameter is user controlled. Is being used to decide which file the application loads.

## Understanding the validation

The lab tells us that the application validates that the supplied filename **ends with the expected file extension**.

So the application is basically expecting something like:

```text id="2q4t9s"

something.png

```

or:

```text id="n3q8xe"

something.jpg

```

This means a traversal payload like:

```text id="aj5h2q"

../../../etc/passwd

```

will fail because it doesn't end with the expected image extension.

Instead of trying to bypass the traversal itself we need to bypass the **file extension validation**.

## Bypassing the extension check

I intercepted the request in Caido. Modified the `filename` parameter to:

```text id="k0x8wv"

../../../etc/passwd%00.png

```
<img width="1867" height="878" alt="2026-10-05_23-45" src="https://github.com/user-attachments/assets/615efc77-fa79-4b58-a6c8-ce2f9811f630" />

The important part here is:

```text id="j1g8cm"

%00

```

This represents a *null byte**.

The idea is that the application checks the filename and sees:

```text

../../../etc/passwd%00.png

```

So the filename appears to end with:

```text id="f3x6sz"

.png

```

and can pass the extension validation.

When the file path is actually processed the null byte can act as a string terminator in vulnerable implementations.

So the underlying file operation effectively sees:

```text id="s0g9za"

../../../etc/passwd

```

of the `.png` part after the null byte.

## Response

The application returned the contents of `/etc/passwd`.

<img width="1918" height="863" alt="2026-10-05_23-45_1" src="https://github.com/user-attachments/assets/59707af9-f319-4590-902a-8a9d0a7281d3" />

This confirms that the extension validation was bypassed and the Path Traversal vulnerability could be exploited.

## Why this works

The thing here is that the application is checking the filename in one way but the underlying file operation processes it differently.

The payload is:

```text id="n5z2qa"

../../../etc/passwd%00.png

```

The application sees the `extension after `%00` and the validation can pass.

Then the null byte terminates the string, for the file operation:

```text id="q7m4x1"

../../../etc/passwd%00.png

↓

null byte

↓

../../../etc/passwd

```

So we can satisfy the extension check while still accessing `/etc/passwd`.

## Result

The `/etc/passwd` file was successfully retrieved using:

```text

../../../etc/passwd%00.png

```

The lab was solved.

## What I learned

This lab showed me that validating the file extension is not enough to prevent Path Traversal.

The application was checking whether the filename ended with an allowed extension. It wasn't safely handling the path before passing it to the file operation.

The main lesson is:

```text id="y5x0p4"

Path Traversal

Weak extension validation

↓

Null byte bypass

↓

../../../etc/passwd%00.png

↓

/etc/passwd

```

So when testing file upload or file access functionality I shouldn't only look at whether the application blocks `../`. I also need to understand **how it validates the filename and how that filename is eventually interpreted by the underlying file operation**.

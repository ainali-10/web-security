# File Path Traversal, Validation of Start of Path

## What I observed

This time when I opened a product image in a tab I noticed something different in the URL.

Of just seeing the filename the application was giving the **full file path**:

```text

https://web-security-academy.net/image?filename=/var/www/images/32.jpg

```

<img width="1739" height="914" alt="2026-10-05_23-28" src="https://github.com/user-attachments/assets/3fd10a30-eab2-45dd-881a-7010dbf50c1b" />

This is interesting because now we can see the directory where the application expects the images to be located:

```text

/var/www/images/

```

I wanted to check whether the `filename` parameter was still user controlled so I changed:

```text

/var/www/images/32.jpg

```

to:

```text

/var/www/images/15.jpg

```

The application returned an image.

<img width="1731" height="874" alt="2026-10-05_23-29" src="https://github.com/user-attachments/assets/d1ce8e1c-ade9-4d45-a24a-b3faa5598047" />

So the entire file path is being passed through the `filename` parameter. Is controlled by us.

## Understanding the validation

The lab says that the application validates that the supplied path starts with the expected folder.

So it is probably checking something to:

```text

/var/www/images/

```

before allowing the request.

At first we might think this prevents Path Traversal because the path has to start with the expected directory.

The important thing is that the application only checks **where the path starts**.

It doesn't necessarily check where the path ends up after resolving the `../` sequences.

## Bypassing the validation

of trying to remove the trusted path I kept it at the beginning and added the traversal sequence after it.

I changed the `filename` parameter to:

```text

/var/www/images/../../../etc/passwd

```
<img width="1891" height="889" alt="2026-10-05_23-33" src="https://github.com/user-attachments/assets/6f6a4f12-ceb6-4b68-82c9-9235e8947832" />

The path still starts with:

```text

/var/www/images/

```

so it passes the applications validation.

After that the `../` sequences move back up the directory structure.

Conceptually:

```text

/var/www/images/

↓

../

↓

/var/www/

↓

../

↓

/var/

↓

../

↓

/

↓

/etc/passwd

```

So although the application sees a path beginning with the expected directory the filesystem resolves the path to:

```text

/etc/passwd

```

## Response

The application returned the contents of `/etc/passwd`.

<img width="1912" height="871" alt="2026-10-05_23-34" src="https://github.com/user-attachments/assets/ba74c0a1-83f4-48f0-a9f0-b6fd9c879e7a" />

This confirms that the validation could be bypassed by placing the traversal sequence **after the trusted path**.

## Why this works

The thing, in this lab is understanding what the application is actually validating.

It checks:

```text

Does the path start with /var/www/images/?

```

Our payload does:

```text

/var/www/images/../../../etc/passwd

^^^^^^^^^^^^^^^^

prefix

```

So the validation passes.

When the filesystem resolves the path the `../` sequences take us outside the intended directory.

This gives us:

```text

/var/www/images/../../../etc/passwd

↓

Path normalization

↓

/etc/passwd

```

The mistake is validating the **start of the path** of validating the final resolved path.

## Result

The `/etc/passwd` file was successfully retrieved using:

```text

/var/www/images/../../../etc/passwd

```

The lab was solved.

## What I learned

This lab showed me that validating the beginning of a file path is not enough to prevent Path Traversal.

If the application only checks that the path starts with a trusted directory I should test whether I can append `../` after that trusted directory and escape it.

The important thing is to think about **where the path actually resolves to** not how it looks at the beginning.

```text

Trusted prefix

/var/www/images/

↓

Add traversal

/var/www/images/../../../etc/passwd

↓

Path gets resolved

↓

/etc/passwd

```

# File Path Traversal, Traversal Sequences Stripped with URL-Decode

## What I observed

We have a shopping application that loads product images using a filename parameter.
 <img width="1728" height="912" alt="2026-10-05_23-00" src="https://github.com/user-attachments/assets/1bac58c9-ce32-4c90-98ce-f43b54b52aa2" />




I opened a product image in a tab and saw the filename in the URL:

```text

/image?filename=41.png

```

<img width="1540" height="883" alt="2026-10-05_23-07" src="https://github.com/user-attachments/assets/77fd2a55-8d9b-4c4f-95db-3765ce0ace29" />

Because the filename is set via the URL I changed:

```text

41.png

```

to:

```text

20.png

```

The application returned a different image.

<img width="1600" height="867" alt="2026-10-05_23-09_1" src="https://github.com/user-attachments/assets/68367cdf-d3d3-48b8-8673-ae1a75fa2adc" />

Once more we know that the `filename` parameter is controlled by the user and decides which file the application loads.

## Trying Path Traversal

Since the parameter is user controlled I first attempted the Path Traversal payload:

```text

../../../etc/passwd

```

But this does not work because the application blocks input that contains traversal sequences.

Therefore the application looks for something like:

```text

../

```

and blocks it before using the filename.

This means we must find a way to hide the traversal sequence from the filter.

## Bypassing the filter with URL encoding

The part of this lab is that after checking the input the application performs a URL-decode before using the filename.

Instead of sending the normal:

```text

../../../etc/passwd

```

I used a **double URL-encoded** version:

```text

..%252f..%252f..%252fetc/passwd

```

<img width="1827" height="875" alt="2026-10-05_23-09" src="https://github.com/user-attachments/assets/50a309f2-2f74-4187-bd38-9d69f2d3ba91" />

The important part here is `%252f`.

A URL-encoded `/` is:

```text

%2f

```

But `%252f` is another level of encoding.

When the application processes the input the first decoding changes:

```text

%252f

```

into:

```text

%2f

```

Then the applications URL-decode turns:

```text

%2f

```

into:

```text

/

```

So the application eventually sees:

```text

../../../etc/passwd

```

even though the filter did not initially see the traversal sequence.

## Response

The application accepted the payload. Returned the contents of `/etc/passwd`.

<img width="1895" height="874" alt="2026-10-05_23-10" src="https://github.com/user-attachments/assets/420d53f2-d031-49b6-b1a1-40e444f7c621" />

This confirms that the URL-encoding bypass worked and the Path Traversal vulnerability could be exploited.

## Why this works

The thing in this lab is the **order in which the application processes the input**.

The application first checks the input for traversal sequences.

Then it performs URL-decoding.

Because we sent the traversal characters in an encoded form the filter does not recognize the traversal sequence at the time it checks the input.

The decoding happens afterward. Turns the encoded characters back, into a traversal path.

```Text

Normal payload

../../../etc/passwd

↓

BLOCKED

encoded payload

..%252f..%252f..%252fetc/passwd

↓

Filter doesn't see../

↓

URL decode

↓

../../../etc/passwd

↓

/etc/passwd

```

## Result

The `/etc/passwd` file was successfully retrieved using:

```text

..%252f..%252f..%252fetc/passwd

```

The lab was solved.

## What I learned

This lab showed me that URL encoding can be used to bypass filters when the application decodes the input **after** performing its security check.

The important thing is not knowing `%252f`.

The real lesson is to pay attention to **the order of operations**:

1. The application checks the input.

2. The encoded traversal passes the check.

3. The application URL-decodes the input.

4. The traversal sequence appears.

5. The application uses the resulting path.

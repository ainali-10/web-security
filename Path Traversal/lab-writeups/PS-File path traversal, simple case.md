<img width="1332" height="838" alt="2026-10-04_23-11" src="https://github.com/user-attachments/assets/f0a2e952-ad4f-4c31-a6ca-1402b78cc0ba" />
# File Path Traversal in Simple Case

## What I observed

First we have a shopping application that displays product images.

When we right-click an image and open it in a tab we see that the URL contains a file path:

```text
/image?filename=32.png
```

This shows that the application uses the filename parameter to decide which file to load.

<img width="1332" height="838" alt="2026-10-04_23-11" src="https://github.com/user-attachments/assets/23d93ad6-3db2-4d3f-8a98-bce65eaa4a27" />

Basically somewhere in the backend it could be doing something like:

```html
<img src="/loadImage?filename=x.png">
```

So the filename is controlled by the user.

## Testing the parameter

To confirm this I changed the filename from:

```text
32.png
```

to:

```text
31.png
```

The application returned an image.

<img width="1215" height="767" alt="2026-10-04_23-13" src="https://github.com/user-attachments/assets/56f37612-add7-4a2f-ae20-eaffa005db90" />

This confirms that the filename parameter is actually used to access files on the server.

Now the interesting part is to see whether the application can access a file outside the intended image directory.

## Exploitation

I intercepted the request in Caido. Modified the filename parameter to:

```text
../../../etc/passwd
```

<img width="1920" height="983" alt="2026-10-04_23-19" src="https://github.com/user-attachments/assets/bb46debb-3d03-4910-bedf-976b0d4c6da3" />

The final request was basically:

```text
/image?filename=../../../etc/passwd
```

The `../` sequences move up directories from the applications directory until we reach the filesystem root. Then `/etc/passwd` specifies the file we want to read.

The server returned the contents of `/etc/passwd` in the response.

<img width="1914" height="807" alt="2026-10-04_23-17" src="https://github.com/user-attachments/assets/ac8ddc7f-793f-489e-8a8a-258cc7ead1a0" />

This confirms that the application is vulnerable to Path Traversal.

## Result

The `/etc/passwd` file was successfully read through the filename parameter and the PortSwigger lab was solved.

## What I learned

The important thing here was not just using `../../../etc/passwd`. The main thing I noticed was that the application was taking a user-controlled filename and using it to access a file on the server. Once I identified that I tested whether I could escape the intended directory using `../`.

This is basically the core idea behind Path Traversal:

```text
User-controlled filename

        ↓

../ to move outside intended directory

        ↓

Access a file, on the server
```

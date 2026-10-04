# File Path Traversal in Simple Case

## What I observed

First we have a shopping application that displays product images.

When we right-click an image and open it in a tab we see that the URL contains a file path:

```text
/image?filename=32.png
```

This shows that the application uses the filename parameter to decide which file to load.

**[Screenshot 1: URL/request with `filename=32.png`]**

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

**[Screenshot 2: URL/request with `filename=31.png` showing the image]**

This confirms that the filename parameter is actually used to access files on the server.

Now the interesting part is to see whether the application can access a file outside the intended image directory.

## Exploitation

I intercepted the request in Caido. Modified the filename parameter to:

```text
../../../etc/passwd
```

**[Screenshot 3: Caido request showing `filename=../../../etc/passwd`]**

The final request was basically:

```text
/image?filename=../../../etc/passwd
```

The `../` sequences move up directories from the applications directory until we reach the filesystem root. Then `/etc/passwd` specifies the file we want to read.

The server returned the contents of `/etc/passwd` in the response.

**[Screenshot 4: Caido response showing the `/etc/passwd` contents]**

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

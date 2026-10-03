# How to Actually Prevent Path Traversal

Most solutions suggested for this issue are not working because they try to remove characters, which doesn't work because there are too many ways to encode the same thing. The real answer is in the way the system is built, not in trying to remove things.

---

## The Solution

Stop user input from being used to create any part of the path completely. Put another way: **do not use a filename provided by the user as part of the path**. Use an identifier instead:

- **Bad:** `GET /download?file=report.pdf`
- **Good:** `GET /download?id=42`

Keep track on your side in a list or database what identifier matches what filename. The request doesn't actually give the filename; an identifier (or some other code) is checked internally to find the real filename. This way there is no way to change the path, assuming of course that you never take any untrusted input and pass it directly to something that will use it as a path.

### Alternative: Character Whitelist

If you have to let users provide filenames, you might want to limit the characters they can use to a whitelist, such as:
- Letters (a-z, A-Z)
- Numbers (0-9)
- Hyphens (-)
- Underscores (_)

Reject anything else.

---

## Check and Confirm, Don't Clean

No matter what path you build, turn it into the correct version using the built-in functions your language provides:
- **PHP:** `realpath()`
- **Python:** `os.path.realpath()` or `pathlib.resolve()`
- **Node.js:** `path.resolve()`
- **Other languages:** Use their standard path normalization function

After normalization, verify that the path is still within the root path you expect. This way all attempts to escape are stopped, and all encoding tricks are handled because you are checking if the final path is still in the right place.

This approach has been proven in real situations to stop these kinds of attacks and is much better than trying to block `../` or similar patterns.

---

## Additional Protection Measures

- **Minimize Permissions:** Run the application with a user account that has the minimum permissions needed. Even if an attacker can read files outside the application's main folder, they can't read restricted system files like `/etc/shadow`.

- **Isolation:** Use chroot jails or containers to separate the application further if needed.

- **Keep Dependencies Updated:** Maintain frameworks and libraries with the latest patches, since many path traversal issues are fixed by their maintainers when discovered.

- **Avoid Client-Side Validation:** Don't rely on checking file paths on the user side unless you are ready to handle workarounds (which you shouldn't be).

---

## Summary

Path traversal prevention is fundamentally about architecture and validation, not about filtering characters. By using identifiers instead of user-provided filenames and validating normalized paths, you create a robust defense against this class of vulnerabilities.

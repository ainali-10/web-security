# Path Traversal: What It Is and Why It Matters

## What is it after all

The path traversal (also known as directory traversal) is an application attack in which an attacker manipulates the application to access files outside its intended directory by exploiting user-controlled input used to build file paths.

A common example is a web application that allows a user to request a file by name, and then builds the path like this:

```python
file_path = "/var/www/uploads/" + user_input
```

If the application does not sanitize or validate the input, the attacker can supply something like:

```text
../../../../etc/passwd
```

The important part is the `../` sequence, which means "go to the parent directory." By chaining these segments together, an attacker can move upward through the file system until they reach restricted locations and read or write files they should not be able to access.

This issue is not limited to a single platform or language. It can happen on Linux, Windows, and other systems as long as the vulnerable application concatenates untrusted input into a file path.

## Why does it happen

The root cause is usually poor input validation and insufficient sanitization.

Developers often assume that a user-provided value is safe, but if the value is used in a file path without checks, attackers can abuse it. In many cases, the application should only allow safe filenames, not arbitrary path segments.

A secure version of the code would validate that the input is an expected filename and reject things like `../`, absolute paths, or other path manipulation sequences.

## The possible effects of the vulnerability

The impact depends on the application, how it is deployed, and what files are accessible to the server. Some of the most common outcomes are:

- Reading sensitive files, such as configuration files, environment files, or source code
- Accessing system files like `/etc/passwd` on Unix-based systems
- Leaking application secrets, such as API keys or database credentials
- Exposing source code and internal application structure
- Writing malicious files to the server, which can lead to code execution in some cases
- Stealing session data or authentication-related files
- Creating a foothold for other attacks that build on the leaked information

## A simple breakdown

The vulnerability usually follows this pattern:

1. User input is accepted from a request or form field.
2. The application combines it with a base directory.
3. The resulting path is used to open, read, or write files.
4. The attacker uses path traversal sequences to escape the intended directory.
5. Sensitive files are accessed or modified.

## Key takeaway

Path traversal is a classic trust issue: untrusted input is used to build a file path without enough checks. It is one of the most common web security problems because it is easy to introduce and can have severe consequences.

The best defense is to validate the input strictly, normalize paths, and reject attempts to escape the intended directory. In short: never trust user input when it becomes part of a file system path.

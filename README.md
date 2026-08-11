# Assignment 2 — Path Traversal Vulnerability Demonstration

## Project Overview

This project is a simple **File Viewer** web application built with Node.js and Express. It allows users to browse and read text documents stored on the server. The application deliberately contains a **Path Traversal** vulnerability to demonstrate how this class of security flaw works in real-world web applications.

---

## File Structure

```
Assignment_2/
├── server.js             # Backend — Express server with API endpoints
├── package.json          # Project dependencies and scripts
├── README.md             # Project documentation (this file)
├── files/                # Directory of files intended to be accessible
│   ├── report.txt        # Sample document — quarterly sales report
│   ├── notes.txt         # Sample document — meeting notes
│   └── readme.txt        # Sample document — welcome message
├── public/               # Frontend static assets
│   └── index.html        # Web interface for browsing and viewing files
└── node_modules/         # Installed dependencies (auto-generated)
```

---

## How to Run the Application

### Prerequisites

- Node.js (v14 or higher)
- npm

### Steps

```bash
# 1. Install dependencies
npm install

# 2. Start the server
npm start

# 3. Open in browser
# http://localhost:3000
```

---

## Application Functionality and Code Walkthrough

### Intended Purpose

The application is meant to be a simple document viewer. Users can see a list of text files stored in the `files/` folder on the server and view their contents through the browser.

---

## Technologies Used

- **Backend:** Node.js, Express.js
- **Frontend:** HTML, CSS, JavaScript (Vanilla)

---

## The Specific Flaw in Our Code

The vulnerability is in `server.js`, in the `/api/view` route handler:

```javascript
app.get("/api/view", (req, res) => {
  const filename = req.query.filename;          // User input taken directly
  const filesDir = path.join(__dirname, "files");
  const filePath = path.join(filesDir, filename); // Joined without any checks

  fs.readFile(filePath, "utf8", (err, data) => {
    if (err) {
      return res.status(404).json({ error: "File not found" });
    }
    res.json({ filename: filename, content: data });
  });
});
```

**The problem:** The `filename` parameter comes directly from the user's request (`req.query.filename`) and is passed straight into `path.join()` with zero sanitization. The server does not:

- Strip `../` or `..\` sequences from the input.
- Check whether the resulting file path is still inside the `files/` directory.
- Validate the filename against an allowlist of permitted files.

This means any `../` sequences in the filename are preserved and processed by the filesystem, allowing the attacker to navigate to parent directories and beyond.

---

## Demonstrating the Attack

### Normal Usage

1. Open `http://localhost:3000` in a browser.
2. The page shows a list of files: `report.txt`, `notes.txt`, `readme.txt`.
3. Click on any file — its contents are displayed. This is the intended, legitimate use.

### The Attack

1. In the **"Open File by Name"** input field, type: `../server.js`
2. Click **View**.
3. The server's own source code is displayed — the attacker has escaped the `files/` directory and read a file they were never supposed to access.

Additional attack payloads:

| Payload                               | What It Reads                              |
|---------------------------------------|--------------------------------------------|
| `../server.js`                        | The server's source code                   |
| `../package.json`                     | Project configuration and dependencies     |
| `..\..\..\Windows\win.ini`            | Windows system configuration file          |

The same attack can be performed directly via the browser address bar or using `curl`:

```
http://localhost:3000/api/view?filename=../server.js
```

---

## How the Vulnerability Could Be Fixed

### Fix 1: Validate the Resolved Path

After resolving the full file path, check that it still falls within the intended `files/` directory:

```javascript
app.get("/api/view", (req, res) => {
  const filename = req.query.filename;
  const filesDir = path.join(__dirname, "files");
  const filePath = path.resolve(filesDir, filename);

  // Ensure the resolved path is still inside the files directory
  if (!filePath.startsWith(filesDir + path.sep)) {
    return res.status(403).json({ error: "Access denied" });
  }

  fs.readFile(filePath, "utf8", (err, data) => {
    if (err) return res.status(404).json({ error: "File not found" });
    res.json({ filename: path.basename(filePath), content: data });
  });
});
```

`path.resolve()` processes all `../` sequences and returns the final absolute path. If that path does not start with the `files/` directory, the request is blocked.

### Fix 2: Strip Traversal Characters

Remove `../` and `..\` sequences from the input before using it:

```javascript
const sanitized = path.basename(filename); // Strips all directory components
```

`path.basename()` extracts just the filename portion, discarding any directory traversal. For example, `../../../etc/passwd` becomes just `passwd`.

### Fix 3: Use an Allowlist

Only serve files that are explicitly listed:

```javascript
const allowedFiles = ["report.txt", "notes.txt", "readme.txt"];
if (!allowedFiles.includes(filename)) {
  return res.status(403).json({ error: "Access denied" });
}
```

## The Vulnerability — Path Traversal

### What Is Path Traversal?

Path Traversal (also known as Directory Traversal or dot-dot-slash attack) is a web security vulnerability that occurs when an application accepts a filename or file path from user input and uses it to access files on the server's filesystem **without properly validating or restricting it** to the intended directory.

Every filesystem uses `../` (or `..\` on Windows) to mean "go up one directory." If a web application blindly appends user input to a base directory path, an attacker can include `../` sequences to **climb out** of the intended folder and reach any file on the server that the application process has permission to read.

### Why Does It Occur?

The root cause is **trusting user input without validation**. Developers often assume users will only provide simple filenames like `report.txt`, but an attacker can supply crafted input like `../../../etc/passwd` to navigate the filesystem. The vulnerability occurs when:

- User-supplied filenames are directly concatenated or joined with a base directory path.
- No checks are performed to ensure the final resolved path stays within the intended directory.
- Characters like `../`, `..\`, or encoded variants (`%2e%2e%2f`) are not stripped or blocked.

### How It Is Exploited in Real Applications

In real-world scenarios, path traversal is commonly exploited to:

- **Read sensitive configuration files** — Attackers access files like `/etc/passwd`, `/etc/shadow`, `.env`, or `web.config` to obtain credentials and system information.
- **Read application source code** — By traversing to the application's own directory, attackers can read server-side code, discover further vulnerabilities, and find hardcoded secrets.
- **Access log files** — Server logs can reveal user data, session tokens, and internal system details.
- **Exfiltrate data** — Any file readable by the server process can be stolen, including databases, private keys, and backup files.

This vulnerability has appeared in many real-world CVEs and is listed in the OWASP Top 10 under "Broken Access Control."

---


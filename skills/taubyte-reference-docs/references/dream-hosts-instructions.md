# Dream: Edit hosts file so you can open app/API in the browser

In Dream (local), your app and API domains use `*.localtau` hostnames. To open them in a browser, you must point those hostnames to your machine.

## 1. Get your domain hostnames (FQDNs)

From the project folder (e.g. `notenow`):

```bash
tau query domain --name notenow_app
tau query domain --name notenow_api
```

Note the **FQDN** for each (e.g. `qz9vx9xo1.default.localtau`, `qz9vx9xo0.default.localtau`). You will add one line per FQDN in the hosts file.

## 2. Open the hosts file (as Administrator / root)

### Windows

- **Path:** `C:\Windows\System32\drivers\etc\hosts`
- **How:** Run Notepad (or another editor) **as Administrator**, then File → Open and go to that path. In the file dialog, set "Text Documents (*.txt)" to **All Files** so you can select `hosts`.
- **Or (PowerShell as Admin):**  
  `notepad C:\Windows\System32\drivers\etc\hosts`

### Linux / macOS

- **Path:** `/etc/hosts`
- **How:** Edit with sudo, e.g.  
  `sudo nano /etc/hosts`  
  or  
  `sudo vi /etc/hosts`

## 3. Add one line per domain

At the **end** of the file, add (replace with your actual FQDNs from step 1):

```
# Taubyte Dream (local) domains
127.0.0.1 qz9vx9xo0.default.localtau
127.0.0.1 qz9vx9xo1.default.localtau
```

- Use **one line per hostname**: `127.0.0.1` followed by a space and the FQDN.
- If you have more domains, run `tau query domain --name <domain-name>` for each and add a line for each FQDN.
- Do **not** add `https://` or a path—only the hostname (e.g. `qz9vx9xo1.default.localtau`).

## 4. Save and close

- **Windows:** Save in Notepad (as Admin). If you get “Access denied”, you are not running the editor as Administrator.
- **Linux/macOS:** Save and exit (e.g. in nano: Ctrl+O, Enter, Ctrl+X).

## 5. Use the app

In Dream, the gateway serves **HTTP** (not HTTPS) on a **port** that depends on your universe. The URL format is:

- **Website:**  
  `http://<website-fqdn>:<port>/`  
  Example: `http://qz9vx9xo1.default.localtau:14105/`

- **API (same host/port, different path):**  
  `http://<api-fqdn>:<port>/<path>`  
  Example: `http://qz9vx9xo0.default.localtau:14105/api/notes`

Replace `<website-fqdn>` and `<api-fqdn>` with the FQDNs from step 1. To get **&lt;port&gt;** for your universe, run:

```bash
dream status universe default
```

In the output, find the **gateway** (or HTTP) row for your universe and use the **http** port number (e.g. `14105`). Ports differ per Dream setup and universe.

## Quick reference

| Step | Action |
|------|--------|
| 1 | `tau query domain --name <domain-name>` → copy each FQDN |
| 2 | Open hosts file as Admin (Windows) or with sudo (Linux/macOS) |
| 3 | Add `127.0.0.1 <fqdn>` for each FQDN |
| 4 | Save the file |
| 5 | Get port: `dream status universe <name>` → gateway http port. Open `http://<website-fqdn>:<port>/` in the browser |

## If you add domains later

After creating new domains, run step 1 again for the new domain names, then add the new FQDNs to the hosts file (step 2–4). No need to remove old lines unless you no longer use those domains.

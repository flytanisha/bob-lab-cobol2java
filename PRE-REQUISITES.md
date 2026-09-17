# Pre-Requisites — IBM Bob COBOL to Java Workshop

Complete every step on this page **before** the workshop session. The workshop itself starts immediately with prompts — there is no setup time built in.

Estimated setup time: **15–30 minutes**

---

## What you need to install

| Tool | Version | Required for |
|------|---------|--------------|
| OpenJDK | 21 | Compiling and running the Java project |
| Apache Maven | 3.6 or higher | Building the project, running tests, launching the app |
| IBM Bob IDE | Latest | The AI assistant IDE used throughout the workshop |

---

## macOS Setup

### Step 1 — Install Homebrew (if not already installed)

Homebrew is the package manager used to install Java and Maven on macOS.

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

To check if you already have it:
```bash
brew --version
```
If that prints a version number, skip this step.

---

### Step 2 — Install OpenJDK 21

```bash
brew install openjdk@21
```

---

### Step 3 — Install Apache Maven

```bash
brew install maven
```

> ⚠️ `brew install maven` may pull in a newer JDK (e.g. JDK 25) as a side dependency and silently override your Java version. After it finishes, continue to Step 4 to pin the correct version.

---

### Step 4 — Set JAVA_HOME

First, identify your chip:

```bash
uname -m
```

- Output `arm64` → you have **Apple Silicon** (M1/M2/M3/M4)
- Output `x86_64` → you have an **Intel Mac**

Run the correct command for your chip:

```bash
# Apple Silicon (M1/M2/M3/M4)
export JAVA_HOME=/opt/homebrew/opt/openjdk@21/libexec/openjdk.jdk/Contents/Home

# Intel Mac
export JAVA_HOME=/usr/local/opt/openjdk@21/libexec/openjdk.jdk/Contents/Home
```

**Make it permanent** — add the correct line to your shell profile so it survives restarts:

```bash
# zsh (default on modern macOS) — Apple Silicon:
echo 'export JAVA_HOME=/opt/homebrew/opt/openjdk@21/libexec/openjdk.jdk/Contents/Home' >> ~/.zshrc
source ~/.zshrc

# zsh — Intel Mac:
echo 'export JAVA_HOME=/usr/local/opt/openjdk@21/libexec/openjdk.jdk/Contents/Home' >> ~/.zshrc
source ~/.zshrc

# bash — Apple Silicon:
echo 'export JAVA_HOME=/opt/homebrew/opt/openjdk@21/libexec/openjdk.jdk/Contents/Home' >> ~/.bash_profile
source ~/.bash_profile
```

---

### Step 5 — Verify everything

Run all three commands and check the output matches what is shown:

```bash
java -version
mvn -version
echo $JAVA_HOME
```

Expected output:
```
openjdk version "21.x.x" ...
Apache Maven 3.x.x ... (Java 21.x.x, ...)
/opt/homebrew/opt/openjdk@21/libexec/openjdk.jdk/Contents/Home   ← Apple Silicon
/usr/local/opt/openjdk@21/libexec/openjdk.jdk/Contents/Home      ← Intel Mac
```

✅ Both `java -version` and the Java version shown in `mvn -version` must say **21**.  
If `mvn -version` shows a different Java version, re-run the `export JAVA_HOME=...` command in your current terminal and try again.

---

## Windows Setup

> 💡 **Before you start:** Some steps below require setting system environment variables. On a corporate laptop you may need administrator rights to change **System variables**. If you only have access to **User variables**, set the variables there instead — they work the same way for your own account.

### Step 1 — Install OpenJDK 21

1. Open a browser and go to: **https://adoptium.net/temurin/releases/?version=21**
2. Set the filters: **Version = 21**, **OS = Windows**, **Architecture = x64**, **Package Type = JDK**, **File Type = .msi**
3. Download the `.msi` installer
4. Run it — **right-click → Run as administrator** if prompted
5. On the **Custom Setup** screen, confirm both of these are set to **"Will be installed"** :
   - `Modify PATH`
   - `Set JAVA_HOME variable`
6. Complete the installation

---

### Step 2 — Install Apache Maven

1. Go to: **https://maven.apache.org/download.cgi**
2. Under **Downloads**, download the **Binary zip archive**: `apache-maven-3.x.x-bin.zip`
3. Create a new folder named Maven in C drive and extract the zip to: `C:\Maven\apache-maven-3.x.x`
   > ⚠️ Do **not** extract to `C:\Program Files\` — that path contains a space which can cause Maven's startup script to fail. Use `C:\Maven\` instead.
4. Set the `MAVEN_HOME` system variable:
   - Copy path of apache-maven-3.x.x
   - Press **Win + S**, search for **"Edit the system environment variables"**, open it
   - Click **"Environment Variables…"**
   - Under **System variables** (or **User variables** if you lack admin rights), click **New**
   - Variable name: `MAVEN_HOME`
   - Variable value: `C:\Maven\apache-maven-3.x.x` *(use your actual version number)*
   - Click **OK**
5. Add Maven's `bin` folder to **Path**:
   - Under the same **System variables** section, select the **Path** row and click **Edit**
   - Click **New** and add: `%MAVEN_HOME%\bin`
   - Click **OK** on every dialog to save

---

### Step 3 — Verify everything

> ⚠️ **Open a brand new Command Prompt** after installing — existing windows do not pick up environment variable changes.
> Press **Win + R**, type `cmd`, press Enter.

Run all four commands:

```cmd
java -version
mvn -version
echo %JAVA_HOME%
echo %MAVEN_HOME%
```

Expected output:
```
openjdk version "21.x.x" ...
Apache Maven 3.x.x ... (Java 21.x.x, ...)
C:\Program Files\Eclipse Adoptium\jdk-21.x.x.x-hotspot
C:\Maven\apache-maven-3.x.x
```

✅ Both `java -version` and the Java version shown inside `mvn -version` must say **21**.

**If `JAVA_HOME` is blank or points to the wrong version**, fix it:
1. Run `where java` in Command Prompt — it shows the full path to the Java executable
2. Remove `\bin\java.exe` from the end to get the JDK root path:
   - Example: `where java` → `C:\Program Files\Eclipse Adoptium\jdk-21.0.5.11-hotspot\bin\java.exe`
   - So `JAVA_HOME` = `C:\Program Files\Eclipse Adoptium\jdk-21.0.5.11-hotspot`
3. Open **Environment Variables** → Under **System variables** (or **User variables**), click **New**
   - Name: `JAVA_HOME`
   - Value: the path from step 2 above
4. Open a new Command Prompt and re-run the verify commands

**If `mvn -version` fails with `'mvn' is not recognized`**:
- Confirm `%MAVEN_HOME%\bin` is in your Path variable
- Make sure you opened a **new** Command Prompt after saving the variables
- As a quick test, run: `%MAVEN_HOME%\bin\mvn -version` — if that works, the Path entry is the issue

**If `mvn -version` shows the wrong Java version** (not 21):
- `JAVA_HOME` may be set to an older JDK — Maven uses `JAVA_HOME` to find Java, not your PATH
- Fix `JAVA_HOME` using the steps above, then open a new Command Prompt and retry

---

## Linux Setup

### Ubuntu / Debian

```bash
sudo apt update
sudo apt install openjdk-21-jdk maven
```

### Fedora / RHEL

```bash
sudo dnf install java-21-openjdk-devel maven
```

### Set JAVA_HOME

```bash
# Add to ~/.bashrc or ~/.zshrc
export JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64   # Ubuntu/Debian
# or
export JAVA_HOME=/usr/lib/jvm/java-21-openjdk          # Fedora/RHEL

source ~/.bashrc
```

### Verify

```bash
java -version
mvn -version
echo $JAVA_HOME
```

Both `java -version` and `mvn -version` must show Java 21.

---

## IBM Bob IDE

1. Downloading IBM Bob
  - Go to the IBM Bob link: https://bob.ibm.com
  - Click on "Get Free Trial"
    
  - Enter details and email id (this will serve as your IBM id)
    OR
  - Sign up via Google or Github or Corporate Email ID
    
  - Verify email
  - Wait for your trail to set up. Wait for an email for successful activation.
2. Download and then sign in to IBM Bob with your IBM credentials
3. Download this github repo - `bob-lab-cobol2java`
4. Open the workshop repository folder in IBM Bob IDE: **File → Open Folder** → select `bob-lab-cobol2java`
5. Confirm Bob is active — you should see the Bob icon or chat panel in the sidebar

---

## Final checklist — run this before the session

Open a terminal (macOS/Linux) or a new Command Prompt (Windows) and confirm every item:

```bash
# macOS / Linux
java -version        # must show 21
mvn -version         # must show Maven 3.6+ and Java 21
echo $JAVA_HOME      # must print a non-empty path
```

```cmd
:: Windows
java -version
mvn -version
echo %JAVA_HOME%
echo %MAVEN_HOME%
```

Then confirm in Bob IDE:
- [ ] The `bob-lab-cobol2java/` folder is open
- [ ] You can see `COBCALC.cbl`, `COBLOAN.cbl`, `COBVALU.cbl` in the file explorer
- [ ] IBM Bob chat panel is visible and responsive

If all of the above are green, you are ready for the workshop. No further setup is needed during the session.

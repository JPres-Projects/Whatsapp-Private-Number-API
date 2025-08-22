
# How to Run `lharries/whatsapp-mcp` After Recent (2024/2025) WhatsApp API Changes

## The Problem

If you're trying to run this project, you will likely encounter two major issues:

1.  A connection error: `[Client ERROR] Client outdated (405) connect failure`
2.  After updating the dependencies to fix the first error, you will get multiple compiler errors like `not enough arguments in call to...` and `cannot use ... as string value in assignment`.

This happens because the underlying library that handles the WhatsApp connection, `go.mau.fi/whatsmeow`, has been updated with breaking changes to keep up with WhatsApp's servers. However, this main `whatsapp-mcp` repository has not been updated accordingly.

This guide provides the exact steps to patch the code locally and make it work.

## Prerequisites

-   [Go](https://go.dev/doc/install) installed.
-   [Git](https://git-scm.com/downloads) installed.
-   (For Windows) [MSYS2 & MinGW-w64](https://www.msys2.org/) installed and configured for CGo.

---

## Step-by-Step Instructions

Follow these steps in your terminal (PowerShell is used for the examples).

### Step 1: Clean Reset (Optional, but Recommended)

To avoid any conflicts with old files, it's best to start fresh.

```powershell
# Navigate to a parent directory, e.g., your user folder
cd C:\Users\<Your_Username>\

# Forcefully remove the old project directory
Remove-Item -Path "whatsapp-mcp" -Recurse -Force
```

### Step 2: Fresh Clone and Environment Setup

```powershell
# Clone the original repository
git clone https://github.com/lharries/whatsapp-mcp.git

# Navigate into the correct sub-directory
cd whatsapp-mcp\whatsapp-bridge

# Set up the compiler path for the current session. 
# IMPORTANT: Adjust this to your actual MSYS2 path.
$env:PATH = "C:\msys64\ucrt64\bin;" + $env:PATH

# Enable CGo, which is required by the sqlite3 driver
go env -w CGO_ENABLED=1
```

### Step 3: Update Dependencies

This will fetch the latest versions of the libraries.

```powershell
# Force update all dependencies
go get -u

# Tidy the go.mod and go.sum files to ensure consistency
go mod tidy
```

### Step 4: Patch the Source Code (`main.go`)

Now, open `main.go` in a code editor and make the following changes.

#### Fix 1: Add `context.Context` to Function Calls

**a) `sqlstore.New` (around line 803)**
```diff
- container, err := sqlstore.New("sqlite3", "file:store/whatsapp.db?_foreign_keys=on", dbLog)
+ container, err := sqlstore.New(context.Background(), "sqlite3", "file:store/whatsapp.db?_foreign_keys=on", dbLog)
```

**b) `container.GetFirstDevice` (around line 810)**
```diff
- deviceStore, err := container.GetFirstDevice()
+ deviceStore, err := container.GetFirstDevice(context.Background())
```

**c) `client.Download` (around line 644)**
```diff
- mediaData, err := client.Download(downloader)
+ mediaData, err := client.Download(context.Background(), downloader)
```

**d) `client.Store.Contacts.GetContact` (around line 991)**
```diff
- contact, err := client.Store.Contacts.GetContact(jid)
+ contact, err := client.Store.Contacts.GetContact(context.Background(), jid)
```

#### Fix 2: Correct the `MediaType` Type Mismatch

**a) `MediaDownloader` Struct (around line 570)**
```diff
-	MediaType     string
+	MediaType     whatsmeow.MediaType
```

**b) `GetMediaType()` Method (around line 600)**
```diff
- func (d *MediaDownloader) GetMediaType() string {
+ func (d *MediaDownloader) GetMediaType() whatsmeow.MediaType {
```

**c) `waMediaType` variable declaration in `downloadMedia()` (around line 620)**
```diff
- var waMediaType string
+ var waMediaType whatsmeow.MediaType
```

### Step 5: Run the Application!

After saving all the changes to `main.go`, you can finally run the program.

```powershell
go run main.go
```

The application should now compile successfully and start.
```

---

### **Corrected, Anonymized Startup Script (`start-whatsapp.bat`)**

This is the most critical fix. The paths are now clearly marked as variables that **must be changed by the user**.

```batch
@echo off
title WhatsApp Bridge - Auto-Restart

REM --- !!! IMPORTANT CONFIGURATION !!! ---
REM --- Change these paths to match YOUR system. ---

set "COMPILER_PATH=C:\msys64\ucrt64\bin"
set "PROJECT_PATH=C:\Users\<Your_Username>\path\to\whatsapp-mcp\whatsapp-bridge"

echo Setting up environment...

REM Go to the project directory
cd /d "%PROJECT_PATH%"

REM Set PATH to include MSYS2 compiler for this session
set "PATH=%COMPILER_PATH%;%PATH%"

REM Ensure CGO is enabled (harmless to run every time)
go env -w CGO_ENABLED=1

REM --- Main Loop ---
:restart
echo.
echo ===================================
echo Starting WhatsApp Bridge...
echo ===================================
echo.
go run main.go

echo.
echo ===================================
echo WARNING: Bridge process stopped!
echo Restarting in 10 seconds...
echo ===================================
timeout /t 10 /nobreak
goto restart
```

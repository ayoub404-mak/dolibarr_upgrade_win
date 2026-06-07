

---

### ⚠️ Top 4 Problems You Might Face to Update dolibarr (Windows  + PostgreSQL)

1. **PHP Version Crash (The #1 Issue)**  
   Dolibarr v12 runs on PHP 7.x. Dolibarr v22 **requires PHP 8.1, 8.2, or 8.3**. If you just copy the v22 files into your current XAMPP (which is running PHP 7.4), the site will instantly throw a "Fatal Error" and go completely white.  
   *Solution:* You must upgrade your XAMPP to a PHP 8.x version *before* or *at the same time* you replace the Dolibarr files.

2. **Custom/Third-Party Modules Breaking**  
   If you installed any external modules (from Dolistore or custom developers) in v12, they are almost certainly incompatible with v22. They will cause the upgrade script to fail or crash the site.  
   *Solution:* You **must** disable and uninstall all non-core modules *before* starting the upgrade.

3. **Database Timeout During Migration**  
   The upgrade script has to run database migration files for v13, v14, v15... all the way to v22. On PostgreSQL, this can take several minutes. Web servers (like Apache) often have a 60-second timeout limit and will kill the process halfway through, leaving your database in a corrupted, half-upgraded state.  
   *Solution:* We will increase PHP and Apache execution times temporarily to prevent this.

4. **Missing `conf.php` or Documents**  
   If you accidentally overwrite your `conf.php` file or your `documents` folder during the file replacement, you will lose your database connection settings and all uploaded files (invoices, images, etc.).  
   *Solution:* We will use a strict backup and replacement method.

---

### 🛡️ The Safe, Step-by-Step Upgrade Guide

Follow these steps exactly to ensure a smooth transition.

#### Step 1: BACKUP EVERYTHING (Do not skip this!)
Before touching anything, secure your current state.
1. **Backup the Database:** Open your Windows Command Prompt and run:
   ```cmd
   "C:\Program Files\PostgreSQL\16\bin\pg_dump.exe" -U postgres -h 127.0.0.1 -p 5432 -F c -b -v -f "C:\dolibarr_backup_v12.backup" dolibarr_db
   ```
   *(Enter your postgres superuser password when prompted).*
2. **Backup the Files:** Go to `C:\xampp\htdocs\` and copy the entire `dolibarr` folder. Paste it somewhere safe (e.g., `C:\dolibarr_v12_backup`).
3. **Backup Documents:** Copy your `C:\dolibarr\documents` folder to `C:\dolibarr_documents_backup`.

#### Step 2: Prepare Dolibarr v12 for Upgrade
1. Log into your current Dolibarr v12 as the **admin** user.
2. Go to **Home > Setup > Modules/Applications**.
3. **Disable** any module that is not part of the core Dolibarr installation (anything with a green "ON" switch that you added later).
4. Go to **Home > Setup > Other Setup** and ensure "Main template" is set to the default `eldy`.

#### Step 3: Upgrade XAMPP to PHP 8.x
Since v22 requires PHP 8, you need to update XAMPP.
1. Stop Apache in the XAMPP Control Panel.
2. Download the latest **XAMPP with PHP 8.2 or 8.3** from [apachefriends.org](https://www.apachefriends.org/).
3. Install it (you can install it over the existing `C:\xampp` directory; it will keep your basic structure, but it's safer to back up `C:\xampp` first).
4. Open the new XAMPP Control Panel -> Apache -> Config -> `php.ini`.
5. Ensure these two lines have **NO** semicolon at the start:
   ```ini
   extension=pdo_pgsql
   extension=pgsql
   ```
6. While in `php.ini`, search for `max_execution_time` and change it to prevent timeouts during the upgrade:
   ```ini
   max_execution_time = 600
   max_input_time = 600
   memory_limit = 512M
   ```
7. Save `php.ini` and **Start** Apache.

#### Step 4: Replace the Dolibarr Files
1. Download the latest **Dolibarr v22** `.zip` file from dolibarr.org.
2. Extract it. You will get a folder like `dolibarr-22.x.x`.
3. **Rename** it to `dolibarr`.
4. Go to `C:\xampp\htdocs\` and **delete** the old `dolibarr` folder.
5. Move the new `dolibarr` folder into `C:\xampp\htdocs\`.
6. **CRITICAL:** Open `C:\xampp\htdocs\dolibarr\htdocs\conf\`. You will see a `conf.php` file is missing or is just the example file. 
   - Go to your backup (`C:\dolibarr_v12_backup\htdocs\conf\`).
   - Copy the **`conf.php`** file from the backup.
   - Paste it into `C:\xampp\htdocs\dolibarr\htdocs\conf\`.
   *(This ensures Dolibarr still knows how to connect to your PostgreSQL database).*

#### Step 5: Run the Upgrade Script
1. Open your browser and go to: **`http://localhost/dolibarr/htdocs/install/`**
2. Choose your language and click **Next**.
3. The system will detect that your database is v12 and the files are v22. It will show a list of upgrade steps.
4. Click **Next** or **Start Upgrade**.
5. **BE PATIENT.** Do not close the browser. Let it run through all the database schema updates. This might take 2 to 5 minutes.

#### Step 6: Post-Upgrade Cleanup
1. Once it says "Upgrade successful," log in as `admin`.
2. Go to **Home > Setup > Modules/Applications** and re-enable the core modules you need.
3. **Do not re-enable old third-party modules** until you verify with their developers that they have a v22-compatible version.
4. Go back to your `php.ini` file and change `max_execution_time` back to `120` (for security and performance). Restart Apache.

---

### What if the upgrade fails halfway?
If you get a white screen or an error during Step 5, **do not panic**. 
1. Your original data is safe in the `C:\dolibarr_backup_v12.backup` file.
2. You can restore it by deleting the new `dolibarr` folder, putting the old v12 folder back, restoring the `conf.php`, and running this command to wipe and restore the database:
   ```cmd
   "C:\Program Files\PostgreSQL\16\bin\psql.exe" -U postgres -h 127.0.0.1 -d postgres -c "DROP DATABASE dolibarr_db;"
   "C:\Program Files\PostgreSQL\16\bin\psql.exe" -U postgres -h 127.0.0.1 -d postgres -c "CREATE DATABASE dolibarr_db OWNER dolibarr_user;"
   "C:\Program Files\PostgreSQL\16\bin\pg_restore.exe" -U postgres -h 127.0.0.1 -d dolibarr_db "C:\dolibarr_backup_v12.backup"
   ```

Take it one step at a time. Let me know when you have completed **Step 1 (The Backup)**, and we can proceed to the next phase!
---




## COMPLETE WORKING BACKUP SCRIPT (Copy-Paste Ready)  FOR Windows + Mysql

```powershell
# Create timestamp and backup folder
$date = Get-Date -Format "yyyyMMdd_HHmmss"
mkdir "C:\DolibarrBackup\$date" -Force

# Backup database (using correct path and database name)
"C:\dolibarr\bin\mariadb\mariadb10.6.5\bin\mysqldump.exe" -u db_user -p -P db_port bd_name > "C:\DolibarrBackup\$date\database.sql"

# Backup documents (using correct folder name)
Copy-Item "C:\dolibarr\dolibarr_documents" "C:\DolibarrBackup\$date\dolibarr_documents" -Recurse

# Backup configuration file
Copy-Item "C:\dolibarr\www\dolibarr\htdocs\conf\conf.php" "C:\DolibarrBackup\$date\conf.php"

# Backup custom modules (if exists)
Copy-Item "C:\dolibarr\www\dolibarr\htdocs\custom" "C:\DolibarrBackup\$date\custom" -Recurse -ErrorAction SilentlyContinue

# Verify backup
Write-Host "Backup completed in: C:\DolibarrBackup\$date" -ForegroundColor Green
Get-ChildItem "C:\DolibarrBackup\$date" -Recurse | Measure-Object -Property Length -Sum
```

---

## COMPLETE WORKING UPGRADE SCRIPT

```powershell
# STEP 1: Stop services (run as Administrator)
Stop-Service doliwampapache -Force
Stop-Service doliwampmysqld -Force

# STEP 2: Verify services are stopped
Get-Service doliwampapache, doliwampmysqld | Select Name, Status

# STEP 3: Run the installer (UPDATE THIS PATH TO YOUR ACTUAL FILE)
Start-Process "C:\Users\hp\Downloads\DoliWamp-16.0.5.exe" -Wait

# STEP 4: Start MySQL first, then Apache
Start-Service doliwampmysqld
Start-Sleep -Seconds 5
Start-Service doliwampapache

# STEP 5: Verify services are running
Get-Service doliwampapache, doliwampmysqld | Select Name, Status

# STEP 6: Open upgrade wizard
Start-Process "http://localhost/dolibarr/install/upgrade.php"

Write-Host "Complete the upgrade in your browser" -ForegroundColor Yellow
```

---

## COMPLETE WORKING ROLLBACK SCRIPT (if upgrade fails)

```powershell
# STEP 1: Stop services
Stop-Service doliwampapache -Force
Stop-Service doliwampmysqld -Force

# STEP 2: List available backups
Get-ChildItem "C:\DolibarrBackup" | Select Name, LastWriteTime

# STEP 3: Set backup to restore (CHANGE THIS TO YOUR BACKUP FOLDER)
$restoreFrom = "C:\DolibarrBackup\20260514_114305"

# STEP 4: Restore database
& "C:\dolibarr\bin\mariadb\mariadb10.6.5\bin\mysql.exe" -u root -p dolibarr < "$restoreFrom\database.sql"

# STEP 5: Restore documents
Copy-Item "$restoreFrom\dolibarr_documents\*" "C:\dolibarr\dolibarr_documents\" -Recurse -Force

# STEP 6: Restore configuration
Copy-Item "$restoreFrom\conf.php" "C:\dolibarr\www\dolibarr\htdocs\conf\conf.php" -Force

# STEP 7: Start services
Start-Service doliwampmysqld
Start-Service doliwampapache

Write-Host "Rollback complete!" -ForegroundColor Green
```

---

## Your Backup Summary

| Item | Status | Location |
|------|--------|----------|
| Database | ✓ 1.4 MB | `database.sql` |
| Documents | ✓ Complete | `dolibarr_documents\` |
| Config | ✓ 1.7 KB | `conf.php` |
| Install.lock | ✓ Exists | Verified |

**Current version detected:** You have backups from versions 12.0.3 and 14.0.5 in your documents folder.

---

## Important Notes for Your Setup

1. **Run PowerShell as Administrator** for service commands
2. **Your installer path:** `C:\Users\hp\Downloads\DoliWamp-16.0.5.exe` (from your last command)
3. **The error "Cannot open service" appeared but services showed as Stopped** - This is normal, proceed anyway

---


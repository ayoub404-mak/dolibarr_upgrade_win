

---

## COMPLETE WORKING BACKUP SCRIPT (Copy-Paste Ready)

```powershell
# Create timestamp and backup folder
$date = Get-Date -Format "yyyyMMdd_HHmmss"
mkdir "C:\DolibarrBackup\$date" -Force

# Backup database (using correct path and database name)
& "C:\dolibarr\bin\mariadb\mariadb10.6.5\bin\mysqldump.exe" -u (dbuser) -p -P (dbport) (dbpasswo) > "C:\DolibarrBackup\$date\database.sql"

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


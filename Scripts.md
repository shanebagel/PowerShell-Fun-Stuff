# Fun PowerShell Stuff
### List of scripts I've written by hand and used in production - work in progress

### Add users in an OU to a Group

>Script to Get all users in a particular OU in AD, and add those users based on SamAccountName property to a specified group

```
Get-ADUser -filter * -searchbase "OU=Users,OU=Lakeland,DC=aspirion,DC=com" | ForEach-Object {Add-AdGroupMember -Identity LAK_SP_ATU_READ -members $_.SamAccountName}
```

### Add users from a CSV to a Group

> Imports users from a comma separated list, uses a for loop to add those users to a specified group using Add-AdGroupMember 

```
$Groups = Import-CSV C:\Users\ecw_aspirion\users.csv 


$Groups |

ForEach-Object {

$Group = $_.Group 

Get-ADUser -SearchBase "OU=Users,OU=Lakeland,DC=aspirion,DC=com" -Filter * | ForEach-Object { 
Add-AdGroupMember -Identity $Group -Members $_}

}
```

### Add users from an "internal" CSV to a Group

>Adds users from a comma separated "internal" list. Doesn't rely on any external files, the delimiter is the new line splitting on (“`n”)

```
$GroupCsv = @"
GroupName1
GroupName2
GroupName3
GroupName4
"@

$UserOU = "OU=Users,OU=Lakeland,DC=aspirion,DC=com"

$Groups = $GroupCsv -split (“`n”)
$Users = Get-ADUser -SearchBase $UserOU -Filter *


ForEach ($Group in $Groups) {

Get-ADGroup $Group


    $Users | ForEach-Object {
        Add-ADGroupMember -Identity $Group -Members $_
    }


}
```

### Install Software from an EXE silently, no GUI
```
Start-Process -Wait -FilePath "PathToExe.exe" -ArgumentList "/S /v/qn" -PassThru
```

### Bulk Create AD Distribution Groups from CSV
```
Import-Module ActiveDirectory
$Users = Import-CSV 'C:\Users\ecw\Desktop\DL.csv'

ForEach ($User in $Users){

$groupProperties = @{

Name = $User.Name
DisplayName = $User.Name
Path = "OU=Distribution Groups,OU=ParkShore,DC=main,DC=parkshoredrug,DC=com"
SamAccountName = $User.Name
GroupScope = "Universal"
GroupCategory = "Distribution"

}

New-ADGroup @groupProperties

}
```

### Bulk Update AD User/Group Attributes from CSV
```
Import-Module ActiveDirectory
$Users = Import-CSV 'C:\Users\ecw\Desktop\DL.csv'

ForEach ($User in $Users){

$groupAttributes = @{

Identity = $user.Name
Replace = @{Mail=$user.Email}


}

Set-ADGroup @groupAttributes

}
```

### Bulk update Primary Proxy Addresses from CSV
```
Import-Module ActiveDirectory
$users = Import-CSV "C:\Users\ecw\desktop\myfile.csv" 
$users | foreach {Set-ADUser -Identity $_.samaccountname -add @{Proxyaddresses= "SMTP:" + $_.Proxyaddresses -split ","}}
# Column names are samAccountName and Proxyaddresses in the CSV file
```

### Bulk update Passwords on New User Accounts from CSV 
```
$users = Import-Csv C:\Users\ecw\desktop\users.csv
$SecurePass = Read-Host "Enter Password" -AsSecureString

$users | ForEach {Set-ADAccountPassword -Identity $_.SamAccountName -NewPassword $SecurePass}

Start-ADSyncSyncCycle -PolicyType Delta
```
       

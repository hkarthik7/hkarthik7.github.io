---
layout: post
title: Store the secrets locally using PersonalVault PowerShell module
categories: [PowerShell]
tags: [Personal Vault, PowerShell, Local Key Vault, Secrets Manager]
description: Store the secrets locally using PersonalVault PowerShell module.
---

Managing secrets can be a hassle and especially saving the logins in excel or sticky notes for quick and easy access is vulnerable. They can be exposed anytime without our knowledge. Won't it be good if we can have our own personal vault? A goto place to store all the credentials and access them whenever needed. Being a PowerShell lover the first thing that aroused in my mind was to create a module to manage the secrets locally. Thus, **PersonalVault** was created.

**PersonalVault** is a PowerShell module to store our credentials that we use regularly, **locally**. It could be anything, starting from bit locker login to bank credentials all the secrets can be encrypted and stored reliably in our system. **PersonalVault** uses PowerShell's inbuilt security mechanism and a 32 bit key to encrypt and store the credentials **locally**. We can add, update, get and delete the secrets once we've connected to the vault. We can export all our secrets and logins that we use for our day-to-day activities and save them to our vault at ease.

## Table Of Contents

1. How to get the module?
2. Module Walk through
3. Closing

### How to get the module?

**PersonalVault** is available in [PowerShellGallery](https://www.powershellgallery.com/packages/PersonalVault/1.1.2) for direct download or if you're running PowerShell version 5 or higher you can run the following cmdlets to download and import it to the current working session.

```powershell
PS C:\> Install-Module PersonalVault -Force
PS C:\> Import-Module PersonalVault -Force
```

### Module Walk through

Let's explore the cmdlets the module provides out of the box.

```powershell
PS C:\> Get-Command -Module PersonalVault

CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Function        Add-PSSecret                                       1.1.2      PersonalVault
Function        Connect-PSPersonalVault                            1.1.2      PersonalVault
Function        Get-PSArchivedKey                                  1.1.2      PersonalVault
Function        Get-PSKey                                          1.1.2      PersonalVault
Function        Get-PSSecret                                       1.1.2      PersonalVault
Function        Import-PSPersonalVault                             1.1.2      PersonalVault
Function        Register-PSPersonalVault                           1.1.2      PersonalVault
Function        Remove-PSPersonalVault                             1.1.2      PersonalVault
Function        Remove-PSPersonalVaultConnection                   1.1.2      PersonalVault
Function        Remove-PSSecret                                    1.1.2      PersonalVault
Function        Update-PSSecret                                    1.1.2      PersonalVault
```

Since we're using the module for first time we should register our credential. Then we should use the registered credential to connect the vault each time when we access it. You can run help on each cmdlet to know the parameters and functionality that it has to provide.

```powershell
PS C:\> Register-PSPersonalVault
```

When you run the above cmdlet you will get a pop-up to enter the credential and recovery word. You should remember the recovery word to get the registered credential.
Once you've registered, connect to the vault.

```powershell
PS C:\> Connect-PSPersonalVault

UserName                     Password Name          ConnectionFilePath
--------                     -------- ----          ------------------
testuser System.Security.SecureString PersonalVault C:\Users\testuser\.cos_testuser\connection.clixml
```

Once you've successfully connected to the vault you get a connection object. The connection file path is where the registered credential is saved. You can overwrite it using *Force* parameter in **Register-PSPersonalVault** cmdlet. You can also remove the connection to the vault and re-register it. But to perform all these actions you should register first.

Now that we have connected to the vault, let's add a secret to it.

```powershell
PS C:\> Add-PSSecret -Name "testuser@gmail.com" -Value "mysecretvalue" -Metadata "My gmail user account"
```

**PersonalVault** validates the secret value before adding it to the vault and warns if it was hacked.

```powershell
PS C:\> Add-PSSecret -Name "testuser1@gmail.com" -Value "Password@123" -Metadata "My another gmail user account"

WARNING: Secret 'Password@123' was hacked 2448 time(s); Consider changing the secret value.
```

List all the stored secrets. Optionally, tab complete the names and get the secret value associated to it, either as an encrypted text or as a plain text.

```powershell
PS C:\> Get-PSSecret

Name                Value
----                -----
testuser1           76492d1116743f0423413b16050a5345MgB8ADMAKwBrADcAQQA3AEMASgBzAGoAbQBmAHQANABwAHgAMAB4ADQAVQAxAHcAPQA9AHwAZA...
testuser@gmail.com  76492d1116743f0423413b16050a5345MgB8AGQAUABNAFYAYwB6AEQANABVAGoAYgBIAG8AbwBuAFIAbwBEAFMAZwBlAFEAPQA9AHwAMA...
testuser1@gmail.com 76492d1116743f0423413b16050a5345MgB8AFAAeAA2AGcAcgB2AGIAbQBJAEIAbAA3AHEARQBlAHQATwB1AHAANwBNAHcAPQA9AHwANA...

PS C:\> Get-PSSecret -AsPlainText

Name                Value         Metadata
----                -----         --------
testuser1           Pass          My another gmail user account
testuser@gmail.com  mysecretvalue My gmail user account
testuser1@gmail.com Password@123  My another gmail user account
```

Get the key that is used to encrypt the credentials.

```powershell
PS C:\> Get-PSKey
UkbB4@swYJx\:qKDWyTInuMEg>1o53OV
```

You can rotate the key using *Force* parameter and save the secrets. This way you can use new key to encrypt the secrets which provides you an additional
security. To get the secrets you have to use the right key.

```powershell
# Rotate the key
PS C:\> Get-PSKey -Force
?x7GVMHZsiw:C0=XET@a]eoSzuYPU3Bd

# Now list the secrets as plain text
PS C:\> Get-PSSecret -AsPlainText
WARNING: Cannot get the value as plain text; Use the right key to get the secret value as plain text.
```

For instance in the above example, you can't get the secrets because the key that was used to encrypt the secrets was different. You have use the same key to get the secrets as plain text. Once you have rotated the key, the old key will be archived so that you can still use it to get the secrets.

```powershell
PS C:\> Get-PSArchivedKey

DateModified          Key
------------          ---
10/08/2021 3:55:25 PM UkbB4@swYJx\:qKDWyTInuMEg>1o53OV
```

Now you can use the archived key to get the secrets.

```powershell
PS C:\> Get-PSSecret -AsPlainText -Key 'UkbB4@swYJx\:qKDWyTInuMEg>1o53OV'

Name                Value         Metadata
----                -----         --------
testuser1           Pass          My another gmail user account
testuser@gmail.com  mysecretvalue My gmail user account
testuser1@gmail.com Password@123  My another gmail user account
```

Update a secret value to the existing credential set. Tab complete the name parameter and update it's secret value easily.

```powershell
PS C:\> Update-PSSecret -Name testuser1 -Value "Thisisunhackablepassword" -Force
```

Remove the secret from the vault using it's Name.

```powershell
PS C:\> Remove-PSSecret -Name testuser1@gmail.com -Force
```

Import the registered credential using the recovery word.

```powershell
PS C:\> Import-PSPersonalVault

cmdlet Import-PSPersonalVault at command pipeline position 1
Supply values for the following parameters:
RecoveryWord: *****

UserName    Password
--------    --------
testuser Testuser
```

Explore the cmdlets and store the secrets locally & securely. Note that PowerShell uses Windows data protection API to encrypt the credentials. This means that only the user who saves the credential in the same machine can decrypt it.

### Closing

Microsoft has officially released [SecretsManagement](https://devblogs.microsoft.com/powershell/secretmanagement-and-secretstore-are-generally-available/) PowerShell module to register and manage the secrets locally and remotely. This provides additional features and flexibility of using any secrets management providers. **PersonalVault** is not a replacement module or similar to **SecretsManagement**, it is a simple module which takes the advantage of PowerShell's inbuilt security to store the credential locally and securely. Having that said, for any module related issues, improvements and feedback open a PR in [github](https://github.com/hkarthik7/PersonalVault).
Happy coding.

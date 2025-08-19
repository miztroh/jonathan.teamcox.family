---
{"publish":true,"title":"SSH","created":"2025-08-19T10:42:45.478-05:00","modified":"2025-08-19T13:31:34.588-05:00","published":"2025-08-19T13:31:34.588-05:00","cssclasses":""}
---

# SSH

1. Check Windows Firewall for an 'OpenSSH' rule.  Edit the rule or create a new one as needed.
2. In C:\ProgramData\ssh\sshd_config:
	1. Uncomment 'PubkeyAuthentication yes' in C:\ProgramData\ssh\sshd_config.
	2. Uncomment 'PasswordAuthentication' and set to 'no'.
	3. Add Subsystem	powershell	"\_\_PROGRAMFILES\_\_\/PowerShell\/7\/pwsh.exe"'.
3. Install PowerShell 7.
4. Set PowerShell 7 as the default SSH shell

```pwsh
$NewItemPropertyParams = @{
	Path = "HKLM:\SOFTWARE\OpenSSH"
	Name = "DefaultShell"
	Value = "C:/Program Files/PowerShell/7/pwsh.exe"
	PropertyType = "String"
	Force = $true
}

New-ItemProperty @NewItemPropertyParams
```

5. (Re)Start the 'OpenSSH SSH Server' service and configure it to run automatically.
6. For each user:
	1. Create a .ssh folder (e.g. C:\\Users\\Administrator\\.ssh).  Disable inheritance, converting to explicit and allowing only 'SYSTEM' and the user.
	2. Create an authorized_keys file (E.g. C:\\Users\Administrator\\.ssh\authorized_keys).  Disable inheritance, converting to explicit and allowing only 'SYSTEM' and the user.
	3. Create an SSH key pair on the user's local machine.  Copy the public key into the authorized_keys file on the remote system.
	4. If the user is an administrator, add their public key to C:\ProgramData\ssh\administrators_authorized_keys, creating a new file if it does not exist.
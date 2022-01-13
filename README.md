# chrome-creds

This script can be used to list all credentials found in the Chrome Local Store.  
It can also be used to Export and Import credentials one by one, with the ability to customize them:
the script will in fact take care of decrypting/re-encrypting passwords as per [1] and [2].

Tested on Windows 10 Enterprise, Version 20H2, OS Build 19042.1466, and Chrome Version 97.0.4692.71 (Official Build) (64-bit).

History:  

**13-Jan-2021 v.1.0.0**: add capability to extract and re-import credentials into the store.

[1] https://xenarmor.com/how-to-recover-saved-passwords-google-chrome/  
[2] https://jhoneill.github.io/powershell/2020/11/23/Chrome-Passwords.html

# Prerequisites

- Chrome must be closed at all times while using this script, otherwise the Local Store database will be kept locked.

- PowerShell 7 must be installed on the target machine.
  See https://docs.microsoft.com/en-us/powershell/scripting/install/installing-powershell-on-windows?view=powershell-7.2
  for instructions on how to install it, and relative packages.  
  You most likely need to install https://github.com/PowerShell/PowerShell/releases/download/v7.2.1/PowerShell-7.2.1-win-x86.msi

- A local copy of `System.Data.SQLite.dll` must be located in the folder where this PowerShell script is executed.
  You can download one from http://system.data.sqlite.org/index.html/doc/trunk/www/downloads.wiki, but you must chose
  the appropriate bitness (Win32/x64) and framework version (2.0,3.5,4.0,4.5,4.5.1,4.6).  
  If you installed PS7 x86 from above, you most likely need http://system.data.sqlite.org/downloads/1.0.115.5/sqlite-netFx40-static-binary-bundle-Win32-2010-1.0.115.5.zip  
  Just extract `System.Data.SQLite.dll` from the downloaded archive, and place it in the folder where this script is being executed.

- If you get a `File chrome-creds.ps1 cannot be loaded because running scripts is disabled on this system` error message when executing the script, it is because scripts execution is not enabled.
  You must execute the script as follows:  
  `pwsh -noprofile -executionpolicy bypass -file .\chrome-creds.ps1`  
  Alternatively, *if you know what you're doing* then you can just set the machine's execution policy to `Unrestricted`: run PS7 as an Administrator (Right Click on PowerShell 7 -> Run As Administrator), move to the folder where the script is located, and then execute the following command:  
  `Set-ExecutionPolicy Unrestricted`  
  Close your PowerShell window and try to execute the script again: execution should now be allowed.

# Usage

This script can be used to perform the following tasks:

## Listing all credentials found in the Chrome Local Store

Execute with:  

```
./chrome-creds.ps1 -command List
```

The script will return the list of credentials stored in the Chrome Local Store, along with their usernames and passwords (in cleartext!). The output will look more or less like this:

```
Opening C:\Users\***\AppData\Local\Google\Chrome\User Data\Default\Login Data...
Listing all credentials stored...
Decoding https://demo.testfire.net/login.jsp ...

id URL                                            UserName                 Password
-- ---                                            --------                 --------
14 https://demo.testfire.net/login.jsp            jsmith                   Demo1234

You can now chose an ID from the list, and export credentials as an executable import command with:

./chrome-creds.ps1 -command Export -id <id>
```

## Exporting credentials from the Chrome Local Store, as an executable command

Execute with:  

```
./chrome-creds.ps1 -command Export -id 14
```

The script will export the chosen credentials as an executable import command. The output will look more or less like this:

```
Opening C:\Users\Cyber_Analyst\AppData\Local\Google\Chrome\User Data\Default\Login Data...
Exporting credential with id 14 ...
Exporting creds for https://demo.testfire.net/login.jsp ...

./chrome-creds.ps1 -Command Import `
        -origin_url https://demo.testfire.net/login.jsp `
        -action_url https://demo.testfire.net/doLogin `
        -username_element uid `
        -username_value jsmith `
        -password_element passw `
        -password_value Demo1234 `
        -submit_element "" `
        -signon_realm https://demo.testfire.net/ `
        -blacklisted_by_user 0 `
        -scheme 0 `
        -password_type 0 `
        -times_used 1 `
        -form_data 3401000007000000050000006C006F00670069006E0000002300000068747470733A2F2F64656D6F2E74657374666972652E6E65742F6C6F67696E2E6A7370002100000068747470733A2F2F64656D6F2E74657374666972652E6E65742F646F4C6F67696E00000002000000090000000000000003000000750069006400000000000000040000007465787400000000FFFFFF7F00000000000000000000000001000000010000000100000002000000000000000000000000000000200000000000000000000000090000000000000005000000700061007300730077000000000000000800000070617373776F726400000000FFFFFF7F0000000000000000000000000100000001000000010000000200000000000000000000000000000020000000000000000000000001000000040000006E756C6C `
        -display_name "" `
        -icon_url "" `
        -federation_url "" `
        -skip_zero_click 0 `
        -generation_upload_status 0 `
        -possible_username_pairs 00000000 `
        -moving_blocked_for 00000000


...exported! Just copy and paste the above command for inserting or updating the chosen creds into the store.
(Remember you can customize username_value and password_value, or any other parameter for that matter).
```

This command can now be copied and then re-executed on the same machine or on another machine, in order to re-import the credentials.  
The beauty of it is that the password is in cleartext, and can therefore be customized as needed: the script will take care of its re-encryption when updating it in the store.

## Importing credentials into the Chrome Local Store

Execute with:  

```
./chrome-creds.ps1 -Command Import `
        -origin_url https://demo.testfire.net/login.jsp `
        -action_url https://demo.testfire.net/doLogin `
        -username_element uid `
        -username_value jsmith `
        -password_element passw `
        -password_value Demo1234 `
        -submit_element "" `
        -signon_realm https://demo.testfire.net/ `
        -blacklisted_by_user 0 `
        -scheme 0 `
        -password_type 0 `
        -times_used 1 `
        -form_data 3401000007000000050000006C006F00670069006E0000002300000068747470733A2F2F64656D6F2E74657374666972652E6E65742F6C6F67696E2E6A7370002100000068747470733A2F2F64656D6F2E74657374666972652E6E65742F646F4C6F67696E00000002000000090000000000000003000000750069006400000000000000040000007465787400000000FFFFFF7F00000000000000000000000001000000010000000100000002000000000000000000000000000000200000000000000000000000090000000000000005000000700061007300730077000000000000000800000070617373776F726400000000FFFFFF7F0000000000000000000000000100000001000000010000000200000000000000000000000000000020000000000000000000000001000000040000006E756C6C `
        -display_name "" `
        -icon_url "" `
        -federation_url "" `
        -skip_zero_click 0 `
        -generation_upload_status 0 `
        -possible_username_pairs 00000000 `
        -moving_blocked_for 00000000
```

The script will import the credentials into the Chrome Local Store. If credentials for this same URL already exist, they will be updated. The output will look more or less like this:

```
Opening C:\Users\Cyber_Analyst\AppData\Local\Google\Chrome\User Data\Default\Login Data...
Importing...
...credential imported!
```

# Credits

Made with ❤️ by Marco Simioni <marcosim@ie.ibm.com>

Based on https://www.powershellgallery.com/packages/Read-Chromium/1.0.0/Content/Read-Chromium.ps1:
```
   Copyright James O'Neill 2020.
   Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"),
   to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense,
   and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:
   The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.
   THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
   FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY,
   WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
```

# License
```
THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY,
WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
```

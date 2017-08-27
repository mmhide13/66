REM <#
copy UpdateArcDps.cmd UpdateArcDps.ps1
PowerShell.exe -ExecutionPolicy Unrestricted -NoProfile -Command "&{Set-Alias REM Write-Host; .\UpdateArcDps.ps1}"
del UpdateArcDps.ps1
exit
REM #>

#MakeCDDVDAgain (so toxic!)
#T4 Purgatory, T1 Pop

#Modify as required. Points to the default install location
$GW2Path = "E:\Guild Wars 2\"

#Don't change these paths unless arc does
$md5sum = 'https://www.deltaconnected.com/arcdps/x64/d3d9.dll.md5sum'
$d3d9 = 'https://www.deltaconnected.com/arcdps/x64/d3d9.dll'

#And this one unless ANet does
$gw = 'Gw2-64'

function Start-GW2
{
    write-host 'Cleanup and startup...'

    Remove-Item $md5Temp.FullName -Force
    Start-Process -FilePath "$($GW2Path)\Gw2-64.exe"
    Exit
}

function Get-ArcDps
{
    write-host 'Downloading ArcDps'

    $d3d9Temp = New-TemporaryFile
    Invoke-WebRequest -Uri $d3d9 -OutFile $d3d9Temp.FullName
    #$d3d9hash = Get-FileHash $d3d9Temp.FullName -Algorithm MD5
    Copy-Item $d3d9Temp.FullName "$($GW2Path)\bin64\d3d9.dll" -Force
    Remove-Item $d3d9Temp.FullName -Force
    write-host 'Completed ArcDps install'

    Start-GW2
}

function Invoke-ArcDpsCheck
{
    if((Get-Process $gw -EA 0).Count -gt 0)
    {
        Exit
    }

    $ignoreCase = [System.StringComparison]::InvariantCultureIgnoreCase
    $md5Temp = New-TemporaryFile
    Invoke-WebRequest -Uri $md5sum -OutFile $md5Temp.FullName
    $md5Hash = Get-Content $md5Temp
    $md5Hash = $md5Hash.Substring(0,$md5Hash.IndexOf(' '))
    $fileExists = Test-Path "$($GW2Path)\bin64\d3d9.dll"

    if($fileExists)
    {
        $md5D3d9m = Get-FileHash "$($GW2Path)\bin64\d3d9.dll" -Algorithm MD5

        if(!$md5D3d9m.Hash.Equals($md5Hash,$ignoreCase))
        {
            Write-Host 'ArcDps is out of date'

            Rename-Item "$($GW2Path)\bin64\d3d9.dll" -NewName "$($GW2Path)\bin64\d3d9_old.dll" -Force
            Get-ArcDps
        }
        else
        {
            write-host 'ArcDps is up to date'
            Start-GW2
        }
    }
    else
    {
        Get-ArcDps
    }
}

Invoke-ArcDpsCheck

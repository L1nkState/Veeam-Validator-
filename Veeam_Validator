# Veeam Backup Validator - v.2.0 05-04-2026
# Author: LinkState
## USAGE:
# > veeam-BackupValidator.ps1 <jobName> <serverName> <Switch>
#
# Switch:
#	-noTicket -> does not send email to the service desk upon Validation completion
#
# e.g.
# > veeam-BackupValidator2.ps1 "D01-JOB-NAME" "YOUR SERVER" -noticket  
# 
##

PARAM (
#	[string] $customer = "YOUR CUSTOMER",
	[string] $jobName = '*',
	[string] $serverName = '*',
	[switch] $noTicket = $true
	)

## -- Enter your customer ID here --
$customer = "YOUR CUSTOMER"
##

$email_infoTO = "yout-servicedesk@your-domain"
$email_infoFrom = "$($env:COMPUTERNAME)@your-domain"
$email_infoServer = "YOUR smart-relay"
$email_subject = "[$customer] Backup Validation"
$logfolder = Split-Path $script:MyInvocation.MyCommand.Path

## ---- Function ----

Function LogWrite {
   Param ([string]$logString,
	[String] $type = "INFO",
	[bool] $savefile
	)
	
	$foregroundColor = "white"
	if ($type -like "INFO")    {$foregroundColor = "white"}
	if ($type -like "ERROR")   {$foregroundColor = "red"}
	if ($type -like "WARNING") {$foregroundColor = "yellow"}
	if ($type -like "POSITIVE"){$foregroundColor = "green"}
	$date = get-date -Format HH:mm:ss
	$logString = "[$type][$date]: $logString" 
	if ($savefile){Add-content -Path $mylogfile -value $logString}
	write-host $logString -ForegroundColor $foregroundColor
}

## ---- Main ----
$startDate = $(Get-Date -UFormat "%m-%d-%Y_%H%M")
$OutputFile = "$logfolder\VeeamValidation_$startDate.html"
$mylogfile = "$logfolder\VeeamValidation_$startDate.log"
$serverNameError = $false

LogWrite "$startDate"
LogWrite "Scan for Job List..."
If ($jobName -eq '*') {
	$backups = Get-VBRBackup
} else {
	$backups = Get-VBRBackup -Name $jobName
}

If ($serverName -ne '*') {
	$serverNameFull = "$jobName - $serverName"
} else {
	$serverNameFull = '*'
}

LogWrite "Job Name: $jobName"
LogWrite "Server Name: $serverName"
LogWrite "No Ticket switch: $noTicket"
LogWrite "OutputFile: $OutputFile"
LogWrite "Logfile: $mylogfile"

$totNum = 0
$totOk = 0
$totKo = 0

# Delete prev temp files
LogWrite "Delete prev temp files"
Get-ChildItem "$logfolder\validationResult\Validation_*.html" | Remove-Item -Force


if ($backups) {
	foreach ($backup in $backups) {
		$child_backups = $backup.FindChildBackups() 
         
		ForEach ($sub_child in $child_backups)  {
			$jobNameSubChild = $sub_child.name
			$serverNameFind = $false
			If (($serverNameFull -eq '*') -or ($serverNameFull -eq $jobNameSubChild)) {
				$id = $sub_child.Id
				$jobNameSubChild =  $jobNameSubChild -replace ' ',''
				$jobNameSubChild =  $jobNameSubChild -replace '\\','#'
				$tempFile = "$logfolder\validationResult\Validation_$jobNameSubChild.html"
				
				$totNum = $totNum +1
			
				LogWrite "Validating     : $jobNameSubChild"
				LogWrite "sub_child.Name : $($sub_child.name)"
				LogWrite "sub_child.id   : $id"
	 
				#Lancia la validazione
#				Set-Location "C:\Program Files\Veeam\Backup and Replication\Backup\"
#				$validateResult = "POSITIVE"
#				$resultValidate = "OK"
                $serverNameFind = $true
				#try { 
				#	$resultValidate = [string](cmd.exe /c "Veeam.Backup.Validator.exe /backup:$id /report:$tempFile /format:html")
				cmd.exe /c "Veeam.Backup.Validator.exe /backup:$id /report:$tempFile /format:html"
				#$resultValidate = & 'Veeam.Backup.Validator.exe' @("/backup:$id", "/report:$tempFile", "/format:html")
				#} catch { 
				#	$validateResult = "ERROR"
				#	$validateError = $_ 
				#}
				#LogWrite "Validate -> $validateError" "$validateResult"
				
				$exitCode = $LastExitCode
				if ($exitCode -eq 0) {
					LogWrite "Validation Result -> OK" "POSITIVE"
					$totOk = $totOk +1
				} else {
					LogWrite "Validation Result -> ERROR ($exitCode)" "ERROR"
					$totKo = $totKo +1
				}
			} 
		}
	}
    If ($serverNameFind -eq $false) {
        LogWrite "Server Name not Found" "ERROR"
        $serverNameError = $true
    }

#	Set-Location $logfolder

	if ($serverNameError -eq $false) {
		Get-ChildItem "$logfolder\validationResult\Validation_*.html" | sort | Get-Content | Set-Content $OutputFile
		LogWrite "Global Validation Statistics: $totNum : $totOk : $totKo (tot:ok:ko)"
		
		if ($noTicket -eq $false) {
			$mailResult = "POSITIVE"
			$mailError = "OK"
			
			$html = [string]$(Get-Content $OutputFile)
			try { send-mailmessage -SmtpServer $email_infoServer -to $email_infoTO -from $email_infoFrom -subject $email_subject -body $html -BodyAsHtml -ErrorAction Stop }
			catch { $mailResult = "ERROR"
				$mailError = $_ }
				
			LogWrite "Mail Sent -> $mailError" "$mailResult"
		}

		$limit = (Get-Date).AddDays(-30)
		# Delete files
		LogWrite "Delete temp files"
		Get-ChildItem "$logfolder\validationResult\Validation_*.html" | Remove-Item -Force
		# Delete files older than the $limit
		LogWrite "Delete files older than $limit"
		Get-ChildItem VeeamValidation*.html | Where-Object { !$_.PSIsContainer -and $_.CreationTime -lt $limit } | Remove-Item -Force
		Get-ChildItem VeeamValidation_*.log | Where-Object { !$_.PSIsContainer -and $_.CreationTime -lt $limit } | Remove-Item -Force
		LogWrite "File Removed"
	}
} else {
	LogWrite "Job Name not Found" "ERROR"
}

$endDate = $(Get-Date -uformat "%m-%d-%Y_%H:%M")
LogWrite "$endDate"
LogWrite "--------------------------------"

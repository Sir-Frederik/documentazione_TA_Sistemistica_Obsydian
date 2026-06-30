
Per le spiegazioni, vai [[📘Guida  personale a TRMM#📜 Script Library|qui]]

##### Ultima revisione: 25/06/2026

## CATEGORIE

1.   **BitLocker**  — Gestione cifratura disco
2.   **Defender**   — Windows Defender (ASR, Controlled Folder Access)
3.   **WDAC**       — Windows Defender Application Control
4.   **AppLocker**  — AppLocker (solo Windows Pro/Enterprise)
5.   **User**       — Gestione account utente locale
6.   **Monitor**    — Script di monitoraggio periodico

##  CONVENZIONI

 - Naming: "Categoria: Azione"
 - Parametri obbligatori sempre dichiarati con [Parameter(Mandatory=$true)]
 - Exit 0 = successo, Exit 1+ = errore (codici specifici per categoria)
 - ApiKey: passata come parametro, mai hardcoded
 - API TRMM: https://ta-tactical-api.duckdns.org
 - BitLocker Custom Field ID: 1 (bitlocker_recovery_key)



## BITLOCKER


### BitLocker: Check Status
```powershell
# SCRIPT  : BitLocker: Check Status
# DESC    : Verifica lo stato di BitLocker sul volume C: e indica
#           l'azione correttiva se il disco non è protetto.
# PARAMS  : Nessuno
# ──────────────────────────────────────────────────────────────────────

$vol = Get-BitLockerVolume -MountPoint "C:" -ErrorAction Stop

$status      = $vol.ProtectionStatus
$volumeState = $vol.VolumeStatus
$method      = $vol.EncryptionMethod
$pct         = $vol.EncryptionPercentage

Write-Output "=== BitLocker Status: C: ==="
Write-Output "Protezione    : $status"
Write-Output "Stato volume  : $volumeState"
Write-Output "Metodo        : $method"
Write-Output "Completamento : $pct%"
Write-Output ""

# ─────────────────────────────────────────
# Valutazione stato e suggerimento azione
# ─────────────────────────────────────────
if ($status -eq "On" -and $volumeState -eq "FullyEncrypted") {
    Write-Output "OK: BitLocker attivo e completo."
} elseif ($status -eq "Off" -and $volumeState -eq "FullyEncrypted") {
    Write-Output "ATTENZIONE: disco cifrato ma protezione sospesa. Eseguire 'BitLocker: Enable'."
} elseif ($volumeState -eq "FullyDecrypted") {
    Write-Output "ATTENZIONE: BitLocker non attivo. Eseguire 'BitLocker: Enable'."
} else {
    Write-Output "ATTENZIONE: stato intermedio ($volumeState). Verificare manualmente."
}

exit 0


# ──────────────────────────────────────────────────────────────────────
```

### BitLocker: Enable
```powershell
# SCRIPT  : BitLocker: Enable
# DESC    : Attiva BitLocker su C: con TPM + Recovery Password.
#           Gestisce i quattro stati possibili: già attivo, sospeso,
#           cifratura in corso, non attivo. Garantisce un solo
#           protector RecoveryPassword (rimuove eventuali duplicati)
#           e salva/aggiorna la Recovery Key nel Custom Field TRMM.
# PARAMS  : ApiKey (obbligatorio) — chiave API TRMM
# ──────────────────────────────────────────────────────────────────────
param(
    [Parameter(Mandatory=$true)]
    [string]$ApiKey
)

$rmmApi  = "https://ta-tactical-api.duckdns.org"
$headers = @{ "X-API-KEY" = $ApiKey; "Content-Type" = "application/json" }

$vol    = Get-BitLockerVolume -MountPoint "C:" -ErrorAction Stop
$status = $vol.ProtectionStatus
$state  = $vol.VolumeStatus

Write-Output "Stato attuale: $status / $state"

if ($status -eq "On" -and $state -eq "FullyEncrypted") {
    Write-Output "BitLocker già attivo. Verifica protector e sincronizzazione chiave..."
}
elseif ($state -eq "EncryptionInProgress") {
    Write-Output "Cifratura già in corso. Nessuna nuova azione di abilitazione necessaria."
}
elseif ($status -eq "Off" -and $state -eq "FullyEncrypted") {
    Write-Output "Disco cifrato ma protezione sospesa. Riattivazione..."
    Resume-BitLocker -MountPoint "C:"
    Write-Output "Protezione riattivata."
}
else {
    Write-Output "Abilitazione BitLocker con TPM + Recovery Password..."
    Enable-BitLocker -MountPoint "C:" -TpmProtector
    Add-BitLockerKeyProtector -MountPoint "C:" -RecoveryPasswordProtector
    Write-Output "BitLocker abilitato. Cifratura disco in corso."
}

# Garantisce un solo RecoveryPassword protector
$vol2       = Get-BitLockerVolume -MountPoint "C:" -ErrorAction Stop
$recoveries = $vol2.KeyProtector | Where-Object { $_.KeyProtectorType -eq "RecoveryPassword" }

if ($recoveries.Count -eq 0) {
    Write-Output "Nessun RecoveryPassword protector trovato. Creazione..."
    Add-BitLockerKeyProtector -MountPoint "C:" -RecoveryPasswordProtector | Out-Null
    $vol2       = Get-BitLockerVolume -MountPoint "C:" -ErrorAction Stop
    $recoveries = $vol2.KeyProtector | Where-Object { $_.KeyProtectorType -eq "RecoveryPassword" }
}
elseif ($recoveries.Count -gt 1) {
    Write-Output "Trovati $($recoveries.Count) protector RecoveryPassword. Rimozione duplicati..."
    $toKeep   = $recoveries | Select-Object -Last 1
    $toRemove = $recoveries | Where-Object { $_.KeyProtectorId -ne $toKeep.KeyProtectorId }
    foreach ($p in $toRemove) {
        Remove-BitLockerKeyProtector -MountPoint "C:" -KeyProtectorId $p.KeyProtectorId
        Write-Output "Rimosso protector obsoleto: $($p.KeyProtectorId)"
    }
    $recoveries = $toKeep
}

$key = [string]($recoveries | Select-Object -First 1 -ExpandProperty RecoveryPassword)

if (-not $key) {
    Write-Output "ATTENZIONE: Recovery Password non ancora disponibile."
    exit 3
}

$hostname = $env:COMPUTERNAME
$agents   = Invoke-RestMethod -Uri "$rmmApi/agents/" -Headers $headers -Method Get
$agent    = $agents | Where-Object -Property hostname -EQ -Value $hostname
$agentId  = $agent.agent_id

if (-not $agentId) {
    Write-Output "ERRORE: agent non trovato per $hostname."
    exit 2
}

$body = @{ custom_fields = @(@{ field = 1; string_value = $key }) } | ConvertTo-Json -Depth 5
Invoke-RestMethod -Uri "$rmmApi/agents/$agentId/" -Headers $headers -Method Put -Body $body
Write-Output "Chiave di ripristino salvata/aggiornata nel Custom Field TRMM."
exit 0


```

### BitLocker: Disable
```powershell
# SCRIPT  : BitLocker: Disable
# DESC    : Disabilita BitLocker e avvia la decifrazione di C:.
#           ATTENZIONE: usare solo per manutenzione straordinaria.
# PARAMS  : Nessuno
# ──────────────────────────────────────────────────────────────────────

# ─────────────────────────────────────────
# 1. Verifica stato attuale (guardia idempotente)
# ─────────────────────────────────────────
$vol = Get-BitLockerVolume -MountPoint "C:" -ErrorAction Stop

if ($vol.ProtectionStatus -eq "Off") {
    Write-Output "BitLocker già disabilitato. Nessuna azione necessaria."
    exit 0
}

# ─────────────────────────────────────────
# 2. Disabilita BitLocker
# ─────────────────────────────────────────
Disable-BitLocker -MountPoint "C:" -ErrorAction Stop

# ─────────────────────────────────────────
# 3. Stato post-operazione
# ─────────────────────────────────────────
$result = Get-BitLockerVolume -MountPoint "C:" | Select-Object MountPoint, ProtectionStatus, VolumeStatus
Write-Output "Stato risultante:"
Write-Output ($result | Out-String)
Write-Output "Decifrazione avviata. Il processo può richiedere alcuni minuti."

exit 0


# ──────────────────────────────────────────────────────────────────────
```

### BitLocker: Store Recovery Key
```powershell
# SCRIPT  : BitLocker: Store Recovery Key
# DESC    : Estrae la Recovery Key BitLocker da C: e la salva nel
#           Custom Field 'bitlocker_recovery_key' dell'agente TRMM.
#           Usare per sincronizzare la chiave senza riabilitare BitLocker.
# PARAMS  : ApiKey (obbligatorio) — chiave API TRMM
# ──────────────────────────────────────────────────────────────────────

param(
    [Parameter(Mandatory=$true)]
    [string]$ApiKey
)

$rmmApi  = "https://ta-tactical-api.duckdns.org"
$headers = @{ "X-API-KEY" = $ApiKey; "Content-Type" = "application/json" }

# ─────────────────────────────────────────
# 1. Estrai Recovery Key da BitLocker
# ─────────────────────────────────────────
$kp     = Get-BitLockerVolume -MountPoint "C:" -ErrorAction Stop
$keyRaw = ($kp.KeyProtector | Where-Object -Property KeyProtectorType -EQ -Value "RecoveryPassword").RecoveryPassword
$key    = [string]($keyRaw | Select-Object -First 1)

if (-not $key) {
    Write-Output "ERRORE: nessuna Recovery Password trovata. Verificare che BitLocker sia attivo."
    exit 1
}
Write-Output "Chiave estratta."

# ─────────────────────────────────────────
# 2. Trova agent_id TRMM tramite hostname
# ─────────────────────────────────────────
$hostname = $env:COMPUTERNAME
$agents   = Invoke-RestMethod -Uri "$rmmApi/agents/" -Headers $headers -Method Get
$agent    = $agents | Where-Object -Property hostname -EQ -Value $hostname
$agentId  = $agent.agent_id

if (-not $agentId) {
    Write-Output "ERRORE: agent non trovato per $hostname."
    exit 2
}
Write-Output "Agent trovato: $agentId"

# ─────────────────────────────────────────
# 3. Salva chiave nel Custom Field TRMM
# (Custom Field ID 1 = bitlocker_recovery_key)
# ─────────────────────────────────────────
$body = @{ custom_fields = @(@{ field = 1; string_value = $key }) } | ConvertTo-Json -Depth 5
Invoke-RestMethod -Uri "$rmmApi/agents/$agentId/" -Headers $headers -Method Put -Body $body

Write-Output "Custom Field 'bitlocker_recovery_key' aggiornato."
exit 0




```

## DEFENDER




### Defender: Enable Controlled Folder Access
```powershell
# SCRIPT  : Defender: Enable Controlled Folder Access
# DESC    : Abilita Controlled Folder Access su Windows Defender.
#           Le app consentite vanno aggiunte manualmente tramite
#           Add-MpPreference -ControlledFolderAccessAllowedApplications
#           con path eseguibile specifico (no wildcard).
# PARAMS  : Nessuno
# ──────────────────────────────────────────────────────────────────────

# ─────────────────────────────────────────
# 1. Abilita Controlled Folder Access
# ─────────────────────────────────────────
Set-MpPreference -EnableControlledFolderAccess Enabled -Force
Write-Output "Controlled Folder Access abilitato."

# ─────────────────────────────────────────
# 2. Verifica configurazione
# (1 = Enabled, 2 = AuditMode, 0 = Disabled)
# ─────────────────────────────────────────
$cfa = (Get-MpPreference).EnableControlledFolderAccess
Write-Output "Stato CFA: $cfa"

exit 0


# ──────────────────────────────────────────────────────────────────────
```
#### Aggiungere app nella Whitelist:

``` powershell
# Ex:
Add-MpPreference -ControlledFolderAccessAllowedApplications "C:\Program Files\Mozilla Firefox\firefox.exe"

```
Controllo:
``` powershell
(Get-MpPreference).ControlledFolderAccessAllowedApplications
```

### Defender: Deploy ASR Rules
```powershell
# SCRIPT  : Defender: Deploy ASR Rules
# DESC    : Configura le 9 ASR rules di Windows Defender.
#           Supporta tre modalità: Enable (produzione),
#           AuditMode (test), Disable (rimozione).
#           Salva lo stato in C:\ProgramData\TacticalRMM\asr_status.json.
# PARAMS  : Mode (facoltativo) — "Enable" | "AuditMode" | "Disable"
#                                Default: "Enable"
# ──────────────────────────────────────────────────────────────────────

param(
    [ValidateSet("Enable", "AuditMode", "Disable")]
    [string]$Mode = "Enable"
)

function Write-ComplianceLog {
    param([string]$Message, [string]$Level = "INFO")
    $timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
    $logPath   = "C:\ProgramData\TacticalRMM\compliance_logs"
    if (-not (Test-Path $logPath)) { New-Item -ItemType Directory -Path $logPath -Force | Out-Null }
    "$timestamp [$Level] $Message" | Tee-Object -FilePath "$logPath\ASR-$(Get-Date -Format 'yyyyMMdd').log" -Append
}

# Mappa modalità stringa → codice numerico Defender
$actionMap = @{ "AuditMode" = 2; "Enable" = 1; "Disable" = 6 }
$action    = $actionMap[$Mode]

Write-ComplianceLog "Avvio configurazione ASR - Mode: $Mode"

try {
    # ─────────────────────────────────────────
    # 1. Verifica e aggiorna Defender
    # ─────────────────────────────────────────
    $defenderStatus = Get-MpComputerStatus
    if ($defenderStatus.ComputerState -ne 0) {
        Write-ComplianceLog "Defender non aggiornato. Aggiornamento firme..." "WARNING"
        Update-MpSignature
    }
    if (-not $defenderStatus.RealTimeProtectionEnabled) {
        Write-ComplianceLog "Abilitazione Real-Time Protection (prerequisito ASR)..."
        Set-MpPreference -DisableRealtimeMonitoring $false -Force
    }

    # ─────────────────────────────────────────
    # 2. Applica le 9 ASR rules
    # ─────────────────────────────────────────
    $ruleIds = @(
        "BE9BA2D9-53EA-4CDC-84E5-9B1EEEE46550",  # Blocca chiamate API Win32 da macro Office
        "01443614-CD74-433A-B99E-2ECDE92B5EBE",   # Blocca eseguibili da USB/drive rimovibili
        "3B576869-A4EC-4529-8536-B80A7769E899",   # Blocca processi figlio creati da Office
        "D4F940AB-5830-4F32-ACE2-C24C4BACB62D",   # Protegge da credential stealing (LSASS)
        "5BEB7EDD-E81B-4C21-80EF-776E0812AF14",   # Blocca creazione di eseguibili da Office
        "9E6C4156-1E8B-4241-8BFF-3FF70766DB7E",   # Blocca macro da applicazioni Office legacy
        "D3E037E1-3EB8-44C8-A917-57927947596D",   # Blocca esecuzione di script offuscati
        "7674BA52-37EB-4A4F-A9A1-F0F9A1619B5C",   # Blocca eseguibili lanciati da script JS/VBS
        "C1DB55AB-C21A-4637-BB3F-A12568109D35"    # Blocca processi figlio creati da Adobe Reader
    )

    foreach ($ruleId in $ruleIds) {
        Add-MpPreference -AttackSurfaceReductionRules_Ids    $ruleId `
                         -AttackSurfaceReductionRules_Actions $action `
                         -Force
    }

    # ─────────────────────────────────────────
    # 3. Verifica e salva stato locale
    # ─────────────────────────────────────────
    $asr = Get-MpPreference | Select-Object AttackSurfaceReductionRules_Ids, AttackSurfaceReductionRules_Actions
    Write-ComplianceLog "ASR configurate: $($asr.AttackSurfaceReductionRules_Ids.Count) regole - Mode: $Mode" "SUCCESS"

    @{ Timestamp = (Get-Date -Format "o"); Mode = $Mode; Rules_Count = $ruleIds.Count } |
        ConvertTo-Json |
        Out-File -FilePath "C:\ProgramData\TacticalRMM\asr_status.json" -Force

    exit 0

} catch {
    Write-ComplianceLog "Errore: $_" "ERROR"
    exit 1
}




```

## WDAC


### WDAC: Compile Enforcement Policy
```powershell
# SCRIPT  : WDAC: Compile Enforcement Policy
# DESC    : Partendo da un XML in audit mode, rimuove la regola
#           "Enabled:Audit Mode", aggiorna la versione e compila
#           il binario .p7b pronto per il deploy.
#           ATTENZIONE: richiede il modulo ConfigCI (Windows Pro/Enterprise).
#           Eseguire solo sulla macchina di compilazione (TA-TEST).
# PARAMS  : SourceXml   — XML sorgente (audit mode)
#           OutputXml   — XML destinazione (enforcement mode)
#           OutputP7b   — Binary compilato da trasferire al PC Home
#           NewVersion  — Versione policy, es. 10.0.0.15
# ──────────────────────────────────────────────────────────────────────

param(
    [Parameter(Mandatory=$true)]
    [string]$SourceXml,
    [Parameter(Mandatory=$true)]
    [string]$OutputXml,
    [Parameter(Mandatory=$true)]
    [string]$OutputP7b,
    [Parameter(Mandatory=$true)]
    [string]$NewVersion
)

# ─────────────────────────────────────────
# 1. Verifica prerequisiti
# ─────────────────────────────────────────
if (-not (Get-Module -ListAvailable -Name ConfigCI)) {
    Write-Output "ERRORE: modulo ConfigCI non disponibile. Eseguire solo su Windows Pro/Enterprise."
    exit 1
}

# ─────────────────────────────────────────
# 2. Rimuovi regola Audit Mode dall'XML
# ─────────────────────────────────────────
$xml = [xml](Get-Content $SourceXml)
$ns  = New-Object System.Xml.XmlNamespaceManager($xml.NameTable)
$ns.AddNamespace("si", "urn:schemas-microsoft-com:sipolicy")

$auditRule = $xml.SelectSingleNode("//si:Rule[si:Option='Enabled:Audit Mode']", $ns)
if ($auditRule) {
    $auditRule.ParentNode.RemoveChild($auditRule)
    Write-Output "Audit Mode rimosso."
} else {
    Write-Output "ATTENZIONE: regola Audit Mode non trovata nell'XML sorgente."
}

# ─────────────────────────────────────────
# 3. Aggiorna versione e salva XML
# ─────────────────────────────────────────
$xml.SiPolicy.VersionEx = $NewVersion
$xml.Save($OutputXml)
Write-Output "XML enforcement salvato: $OutputXml (versione $NewVersion)"

# ─────────────────────────────────────────
# 4. Compila in binario .p7b
# ─────────────────────────────────────────
ConvertFrom-CIPolicy -XmlFilePath $OutputXml -BinaryFilePath $OutputP7b
Write-Output "Compilato: $OutputP7b"

exit 0


# ──────────────────────────────────────────────────────────────────────
```

### WDAC: Export Audit Log
```powershell
# SCRIPT  : WDAC: Export Audit Log
# DESC    : Esporta gli eventi WDAC in audit mode (Event ID 3076)
#           in un file di testo leggibile. Eseguire sul PC Home
#           prima di raccogliere file per aggiornare la policy.
# PARAMS  : Nessuno
# ──────────────────────────────────────────────────────────────────────

$outputPath = "C:\WDAC\audit_newapps.txt"

# ─────────────────────────────────────────
# 1. Leggi log CodeIntegrity (Event ID 3076 = audit block)
# ─────────────────────────────────────────
$events = Get-WinEvent -LogName "Microsoft-Windows-CodeIntegrity/Operational" -ErrorAction Stop |
    Where-Object { $_.Id -eq 3076 }

if (-not $events) {
    Write-Output "Nessun evento 3076 trovato. Nessun file nuovo bloccato in audit mode."
    exit 0
}

# ─────────────────────────────────────────
# 2. Salva in file di testo
# ─────────────────────────────────────────
$events | Select-Object TimeCreated, Message | Format-List | Out-File $outputPath -Encoding UTF8
Write-Output "Log salvato: $outputPath ($($events.Count) eventi trovati)"

exit 0


# ──────────────────────────────────────────────────────────────────────
```

### WDAC: Remove Active Policy
```powershell
# SCRIPT  : WDAC: Remove Active Policy
# DESC    : Rimuove il file .cip della policy WDAC attiva.
#           La rimozione è effettiva al successivo riavvio.
#           ATTENZIONE: usare solo per recovery — dopo il riavvio
#           WDAC non sarà più attivo. Tenere a portata la Recovery
#           Key BitLocker prima di procedere.
# PARAMS  : Nessuno
# ──────────────────────────────────────────────────────────────────────

$policyGuid = "{966d1f08-bcea-48c4-bc3a-6651c21e4090}"
$activePath = "C:\Windows\System32\CodeIntegrity\CiPolicies\Active"
$cipFile    = Join-Path $activePath "$policyGuid.cip"

# ─────────────────────────────────────────
# 1. Verifica esistenza file policy
# ─────────────────────────────────────────
if (-not (Test-Path $cipFile)) {
    Write-Output "File policy non trovato: $cipFile"
    Write-Output "La policy potrebbe già essere stata rimossa."
    exit 0
}

# ─────────────────────────────────────────
# 2. Rimuovi file .cip
# ─────────────────────────────────────────
Remove-Item $cipFile -Force
Write-Output "Policy rimossa: $cipFile"
Write-Output "Riavviare il sistema per rendere effettiva la rimozione."

exit 0


```

### WDAC: Deploy Policy

``` powershell

# ──────────────────────────────────────────────────────────────────────
# SCRIPT  : WDAC: Deploy Policy
# DESC    : Deploya un file .p7b sul PC locale tramite CiTool e
#           verifica che il file .cip risultante sia presente nella
#           cartella Active. Scrive uno stato locale leggibile dal
#           monitor. Il .p7b deve essere già presente sul PC
#           (trasferito via MeshCentral prima di eseguire lo script).
# PARAMS  : PolicyPath    (obbligatorio) — path al file .p7b
#                          es. C:\WDAC\policy_final_v15.p7b
#           PolicyVersion (obbligatorio) — versione della policy
#                          es. 10.0.0.15
#           Reboot        (switch facoltativo) — riavvia dopo il deploy
# ──────────────────────────────────────────────────────────────────────

param(
    [Parameter(Mandatory=$true)]
    [string]$PolicyPath,
    [Parameter(Mandatory=$true)]
    [string]$PolicyVersion,
    [switch]$Reboot
)

$policyGuid = "{966d1f08-bcea-48c4-bc3a-6651c21e4090}"
$activePath = "C:\Windows\System32\CodeIntegrity\CiPolicies\Active"
$cipFile    = Join-Path $activePath "$policyGuid.cip"
$stateFile  = "C:\ProgramData\TacticalRMM\wdac_state.json"

# ─────────────────────────────────────────
# 1. Verifica prerequisiti
# ─────────────────────────────────────────
if (-not (Test-Path $PolicyPath)) {
    Write-Output "ERRORE: file policy non trovato: $PolicyPath"
    Write-Output "Trasferire il .p7b sul PC tramite MeshCentral prima di eseguire lo script."
    exit 1
}
Write-Output "File policy trovato: $PolicyPath"

# ─────────────────────────────────────────
# 2. Deploy tramite CiTool
# ─────────────────────────────────────────
$result = & CiTool --update-policy $PolicyPath 2>&1
Write-Output "CiTool: $result"

if ($LASTEXITCODE -ne 0) {
    Write-Output "ERRORE: CiTool ha restituito codice $LASTEXITCODE"
    exit 2
}

# ─────────────────────────────────────────
# 3. Verifica file .cip nell'Active folder
# ─────────────────────────────────────────
Start-Sleep -Seconds 2
if (Test-Path $cipFile) {
    Write-Output "Verifica OK: $cipFile presente."
} else {
    Write-Output "ATTENZIONE: CiTool non ha riportato errori ma il file .cip non è presente."
    Write-Output "Verificare manualmente prima di riavviare."
    exit 3
}

# ─────────────────────────────────────────
# 4. Scrivi stato locale (letto da Monitor: WDAC Policy Status)
# ─────────────────────────────────────────
$stateDir = Split-Path $stateFile
if (-not (Test-Path $stateDir)) { New-Item -ItemType Directory -Path $stateDir -Force | Out-Null }

@{
    PolicyGuid  = $policyGuid
    Version     = $PolicyVersion
    DeployedAt  = (Get-Date -Format "o")
    PolicyPath  = $PolicyPath
} | ConvertTo-Json | Out-File -FilePath $stateFile -Force

Write-Output "Stato scritto: $stateFile"

# ─────────────────────────────────────────
# 5. Riavvio (se richiesto)
# ─────────────────────────────────────────
if ($Reboot) {
    Write-Output "Riavvio in corso..."
    Restart-Computer -Force
} else {
    Write-Output "Policy $PolicyVersion deployata. Riavviare il sistema per renderla effettiva."
}

exit 0



```
## APPLOCKER

NOTA: AppLocker funziona solo su Windows Pro/Enterprise.
Su macchine con WDAC attivo questi script sono superflui.
Usare solo sulla macchina di compilazione (TA-TEST).


### AppLocker: Set Enforcement
```powershell
# SCRIPT  : AppLocker: Set Enforcement
# DESC    : Avvia il servizio AppLocker e applica una policy di base:
#           exe consentiti solo da Windows e Program Files per tutti,
#           tutto consentito agli Administrators.
#           Solo Windows Pro/Enterprise — inerte su Home.
# PARAMS  : Nessuno
# ──────────────────────────────────────────────────────────────────────

# ─────────────────────────────────────────
# 1. Avvia servizio AppLocker
# ─────────────────────────────────────────
Set-Service AppIdSvc -StartupType Automatic -ErrorAction SilentlyContinue
Start-Service AppIdSvc -ErrorAction SilentlyContinue
Write-Output "Servizio AppIdSvc avviato."

# ─────────────────────────────────────────
# 2. Applica policy enforcement
# ─────────────────────────────────────────
$policy = @"
<AppLockerPolicy Version="1">
  <RuleCollection Type="Exe" EnforcementMode="Enabled">
    <FilePathRule Id="a9e18c21-ff8f-43cf-b9fc-db40eed633a0"
                  Name="Allow Admins - All" Description=""
                  UserOrGroupSid="S-1-5-32-544" Action="Allow">
      <Conditions><FilePathCondition Path="*" /></Conditions>
    </FilePathRule>
    <FilePathRule Id="a9e18c21-ff8f-43cf-b9fc-db40eed633a1"
                  Name="Allow Windows" Description=""
                  UserOrGroupSid="S-1-1-0" Action="Allow">
      <Conditions><FilePathCondition Path="%WINDIR%\*" /></Conditions>
    </FilePathRule>
    <FilePathRule Id="a9e18c21-ff8f-43cf-b9fc-db40eed633a2"
                  Name="Allow Program Files" Description=""
                  UserOrGroupSid="S-1-1-0" Action="Allow">
      <Conditions><FilePathCondition Path="%PROGRAMFILES%\*" /></Conditions>
    </FilePathRule>
    <FilePathRule Id="a9e18c21-ff8f-43cf-b9fc-db40eed633a3"
                  Name="Allow Program Files x86" Description=""
                  UserOrGroupSid="S-1-1-0" Action="Allow">
      <Conditions><FilePathCondition Path="%PROGRAMFILES(X86)%\*" /></Conditions>
    </FilePathRule>
    <FilePathRule Id="a9e18c21-ff8f-43cf-b9fc-db40eed633a4"
                  Name="Block WindowsApps" Description=""
                  UserOrGroupSid="S-1-1-0" Action="Deny">
      <Conditions><FilePathCondition Path="%PROGRAMFILES%\WindowsApps\*" /></Conditions>
    </FilePathRule>
  </RuleCollection>
  <RuleCollection Type="Msi" EnforcementMode="Enabled">
    <FilePathRule Id="a9e18c21-ff8f-43cf-b9fc-db40eed633b0"
                  Name="Allow Admins - MSI" Description=""
                  UserOrGroupSid="S-1-5-32-544" Action="Allow">
      <Conditions><FilePathCondition Path="*" /></Conditions>
    </FilePathRule>
  </RuleCollection>
  <RuleCollection Type="Script" EnforcementMode="AuditOnly">
    <FilePathRule Id="a9e18c21-ff8f-43cf-b9fc-db40eed633c0"
                  Name="Allow Admins - Script" Description=""
                  UserOrGroupSid="S-1-5-32-544" Action="Allow">
      <Conditions><FilePathCondition Path="*" /></Conditions>
    </FilePathRule>
  </RuleCollection>
</AppLockerPolicy>
"@

$tempFile = "$env:TEMP\applocker_policy.xml"
$policy | Out-File -FilePath $tempFile -Encoding UTF8
Set-AppLockerPolicy -XmlPolicy $tempFile
Remove-Item $tempFile -Force

Write-Output "AppLocker configurato in enforcement mode."


# ──────────────────────────────────────────────────────────────────────
```

### AppLocker: Set AuditOnly
```powershell
# SCRIPT  : AppLocker: Set AuditOnly
# DESC    : Porta AppLocker in modalità AuditOnly e tenta di fermare
#           il servizio AppIdSvc. Il servizio potrebbe non fermarsi
#           se ha dipendenze attive — comportamento atteso.
# PARAMS  : Nessuno
# ──────────────────────────────────────────────────────────────────────

# ─────────────────────────────────────────
# 1. Leggi policy corrente e porta in AuditOnly
# ─────────────────────────────────────────
$effectivePolicy = Get-AppLockerPolicy -Effective
$xml = $effectivePolicy.ToXml()
$xml = $xml -replace 'EnforcementMode="Enabled"', 'EnforcementMode="AuditOnly"'

$tempFile = "$env:TEMP\applocker_audit.xml"
$xml | Out-File -FilePath $tempFile -Encoding UTF8
Set-AppLockerPolicy -XmlPolicy $tempFile
Remove-Item $tempFile -Force
Write-Output "AppLocker portato in AuditOnly."

# ─────────────────────────────────────────
# 2. Disabilita e ferma il servizio
# ─────────────────────────────────────────
sc.exe config AppIdSvc start= disabled
sc.exe stop AppIdSvc
Write-Output "Servizio AppIdSvc: stop richiesto (potrebbe fallire per dipendenze attive)."




```

## USER


### User: Enable Account
```powershell
# SCRIPT  : User: Enable Account
# DESC    : Abilita un account utente locale disabilitato.
# PARAMS  : Username (obbligatorio)
# ──────────────────────────────────────────────────────────────────────

param(
    [Parameter(Mandatory=$true)]
    [string]$Username
)

# ─────────────────────────────────────────
# 1. Abilita account locale
# ─────────────────────────────────────────
Enable-LocalUser -Name $Username -ErrorAction Stop

Write-Output "Account '$Username' abilitato."


# ──────────────────────────────────────────────────────────────────────
```

### User: Disable Account
```powershell
# SCRIPT  : User: Disable Account
# DESC    : Disabilita un account utente locale e termina
#           la sessione attiva, se presente.
# PARAMS  : Username (obbligatorio)
# ──────────────────────────────────────────────────────────────────────

param(
    [Parameter(Mandatory=$true)]
    [string]$Username
)

# ─────────────────────────────────────────
# 1. Disabilita account locale
# ─────────────────────────────────────────
Disable-LocalUser -Name $Username -ErrorAction Stop

# ─────────────────────────────────────────
# 2. Termina sessione attiva, se presente
# ─────────────────────────────────────────
$session = query user $Username 2>$null | Select-String $Username
if ($session) {
    $sessionId = ($session -split '\s+')[2]
    logoff $sessionId /server:localhost
    Write-Output "Sessione '$Username' terminata (ID: $sessionId)."
}

Write-Output "Account '$Username' disabilitato."


# ──────────────────────────────────────────────────────────────────────
```

### User: Grant Admin
```powershell
# SCRIPT  : User: Grant Admin
# DESC    : Aggiunge un utente al gruppo Administrators locale.
# PARAMS  : Username (obbligatorio)
# ──────────────────────────────────────────────────────────────────────

param(
    [Parameter(Mandatory=$true)]
    [string]$Username
)

# ─────────────────────────────────────────
# 1. Verifica appartenenza corrente (guardia idempotente)
# ─────────────────────────────────────────
$isMember = Get-LocalGroupMember -Group "Administrators" -ErrorAction SilentlyContinue |
    Where-Object { $_.Name -like "*$Username" }

if ($isMember) {
    Write-Output "ATTENZIONE: '$Username' è già membro di Administrators. Nessuna azione."
    exit 0
}

# ─────────────────────────────────────────
# 2. Aggiungi al gruppo Administrators
# ─────────────────────────────────────────
Add-LocalGroupMember -Group "Administrators" -Member $Username -ErrorAction Stop

Write-Output "Utente '$Username' aggiunto agli Administrators."


```

### User: Revoke Admin
```powershell
# SCRIPT  : User: Revoke Admin
Rimuove i privilegi di Administrators e applica
 restrizioni aggiuntive: blocco MSI, Autorun disabilitato,
 blocco disinstallazione programmi.
 Il controllo degli eseguibili è delegato a WDAC.
# PARAMS  : Username (obbligatorio)
# ──────────────────────────────────────────────────────────────────────

param(
    [Parameter(Mandatory=$true)]
    [string]$Username
)

# ─────────────────────────────────────────
# 1. Rimuovi da Administrators
# ─────────────────────────────────────────
Remove-LocalGroupMember -Group "Administrators" -Member $Username -ErrorAction SilentlyContinue

# ─────────────────────────────────────────
# 2. Blocca Windows Installer per non-admin
# ─────────────────────────────────────────
$installerKey = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\Installer"
if (-not (Test-Path $installerKey)) { New-Item -Path $installerKey -Force | Out-Null }
Set-ItemProperty -Path $installerKey -Name "DisableMSI"        -Value 2 -Type DWord
Set-ItemProperty -Path $installerKey -Name "EnableUserControl" -Value 0 -Type DWord

# ─────────────────────────────────────────
# 3. Disabilita Autorun/Autoplay
# ─────────────────────────────────────────
$explorerKey = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\Explorer"
if (-not (Test-Path $explorerKey)) { New-Item -Path $explorerKey -Force | Out-Null }
Set-ItemProperty -Path $explorerKey -Name "NoAutoplayfornonVolume" -Value 1 -Type DWord

$autorunKey = "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Explorer"
if (-not (Test-Path $autorunKey)) { New-Item -Path $autorunKey -Force | Out-Null }
Set-ItemProperty -Path $autorunKey -Name "NoDriveTypeAutoRun" -Value 255 -Type DWord

# ─────────────────────────────────────────
# 4. Blocca disinstallazione programmi
# ─────────────────────────────────────────
$uninstallKey = "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Uninstall"
if (-not (Test-Path $uninstallKey)) { New-Item -Path $uninstallKey -Force | Out-Null }
Set-ItemProperty -Path $uninstallKey -Name "NoModifyOrRemovePrograms" -Value 1 -Type DWord

Write-Output "Privilegi rimossi e restrizioni applicate per '$Username'."




```

## MONITOR


### Monitor: Store Apps
```powershell
# SCRIPT  : Monitor: Store Apps
# DESC    : Rileva app Microsoft Store installate non presenti nella
#           lista autorizzata. Esce con codice 1 se trova anomalie
#           (TRMM segnala alert). Da eseguire via Automation Policy.
# PARAMS  : Nessuno
# ──────────────────────────────────────────────────────────────────────

# ─────────────────────────────────────────
# Lista app Store consentite
# NOTA: usare Get-AppxPackage per verificare il nome esatto
#       prima di aggiungere nuove voci
# ─────────────────────────────────────────
$allowedApps = @(
    "Microsoft.WindowsCalculator",
    "Microsoft.WindowsNotepad",
    "Microsoft.Windows.Photos",
    "Microsoft.Paint",
    "Microsoft.WindowsStore",
    "Microsoft.DesktopAppInstaller",
    "Microsoft.WindowsTerminal",
    "Microsoft.SecHealthUI",
    "Microsoft.WindowsSoundRecorder",
    "OpenAI.ChatGPT",    # Verificare nome pacchetto esatto con Get-AppxPackage
    "Anthropic.Claude"   # Verificare nome pacchetto esatto con Get-AppxPackage
)

# ─────────────────────────────────────────
# Confronto app installate vs lista consentita
# ─────────────────────────────────────────
$installedStoreApps = Get-AppxPackage -AllUsers |
    Where-Object { $_.SignatureKind -eq "Store" } |
    Select-Object -ExpandProperty Name |
    Sort-Object

$unauthorized = $installedStoreApps | Where-Object { $_ -notin $allowedApps }

if ($unauthorized) {
    Write-Output "ATTENZIONE - App Store non autorizzate:"
    $unauthorized | ForEach-Object { Write-Output "  - $_" }
    exit 1
}

Write-Output "OK - Nessuna app Store non autorizzata trovata."
exit 0
```

### Monitor: BitLocker Compliance
``` Powershell
# ──────────────────────────────────────────────────────────────────────
# SCRIPT  : Monitor: BitLocker Compliance
# DESC    : Verifica che BitLocker sia attivo e completamente cifrato
#           su C:. Esce con codice 1 se non conforme — TRMM segnala
#           alert. Da eseguire via Automation Policy su tutta la flotta.
# PARAMS  : Nessuno
# ──────────────────────────────────────────────────────────────────────

# ─────────────────────────────────────────
# 1. Leggi stato BitLocker
# ─────────────────────────────────────────
$vol = Get-BitLockerVolume -MountPoint "C:" -ErrorAction Stop

$status = $vol.ProtectionStatus
$state  = $vol.VolumeStatus

# ─────────────────────────────────────────
# 2. Valutazione conformità
# ─────────────────────────────────────────
if ($status -eq "On" -and $state -eq "FullyEncrypted") {
    Write-Output "OK: BitLocker attivo e completo."
    exit 0
}

# Distingui i casi di non conformità per facilitare il triage
if ($status -eq "Off" -and $state -eq "FullyEncrypted") {
    Write-Output "NON CONFORME: disco cifrato ma protezione sospesa. Eseguire 'BitLocker: Enable'."
} elseif ($state -eq "FullyDecrypted") {
    Write-Output "NON CONFORME: BitLocker non attivo. Eseguire 'BitLocker: Enable'."
} else {
    Write-Output "NON CONFORME: stato intermedio - Protezione: $status / Volume: $state"
}

exit 1

```

### Monitor: WDAC Policy Status

``` Powershell

 
# SCRIPT  : Monitor: WDAC Policy Status
# DESC    : Verifica che la policy WDAC corretta sia deployata:
#           controlla la presenza del file .cip nella cartella Active
#           e confronta la versione deployata con quella attesa.
#           Esce con codice 1 se non conforme — TRMM segnala alert.
#           Da eseguire via Automation Policy dopo ogni rollout.
# PARAMS  : ExpectedVersion (obbligatorio) — versione attesa
#                            es. 10.0.0.14
# ──────────────────────────────────────────────────────────────────────

param(
    [Parameter(Mandatory=$true)]
    [string]$ExpectedVersion
)

$policyGuid = "{966d1f08-bcea-48c4-bc3a-6651c21e4090}"
$activePath = "C:\Windows\System32\CodeIntegrity\CiPolicies\Active"
$cipFile    = Join-Path $activePath "$policyGuid.cip"
$stateFile  = "C:\ProgramData\TacticalRMM\wdac_state.json"

# ─────────────────────────────────────────
# 1. Verifica presenza file .cip
# ─────────────────────────────────────────
if (-not (Test-Path $cipFile)) {
    Write-Output "NON CONFORME: policy WDAC non presente (file .cip mancante)."
    exit 1
}
Write-Output "File .cip presente: $cipFile"

# ─────────────────────────────────────────
# 2. Verifica versione dallo stato locale
# ─────────────────────────────────────────
if (-not (Test-Path $stateFile)) {
    Write-Output "ATTENZIONE: file di stato non trovato ($stateFile)."
    Write-Output "Il deploy potrebbe essere stato eseguito senza 'WDAC: Deploy Policy'."
    Write-Output "File .cip presente ma versione non verificabile."
    exit 1
}

$state = Get-Content $stateFile | ConvertFrom-Json
$deployedVersion = $state.Version

if ($deployedVersion -eq $ExpectedVersion) {
    Write-Output "OK: policy $policyGuid versione $deployedVersion — deployata il $($state.DeployedAt)."
    exit 0
} else {
    Write-Output "NON CONFORME: versione attesa $ExpectedVersion, trovata $deployedVersion."
    Write-Output "Eseguire 'WDAC: Deploy Policy' con il .p7b aggiornato."
    exit 1
}

```
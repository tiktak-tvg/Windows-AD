###### Получить список DC в домене:
```powershell
Get-ADDomainController
```
###### Получить список ошибок репликации на указанных DC:
```powershell
Get-ADReplicationFailure -Target DC1,DC2
```
###### Вывести ошибки репликации AD в сайте (-Scope Site) или домене (-Scope Domain):
```powershell
Get-ADReplicationFailure -scope site -target HQ| FT Server, LastError, Partner-Auto
Get-ADReplicationFailure -Target winitpro.ru -Scope Domain
```
###### Получить список партнеров по репликации текущего DC:
```powershell
Get-ADReplicationConnection -Filter *
```
###### Чтобы выполнить принудительную репликацию используется командлет Sync-ADObject.

Например, следующая команда выведет все обнаруженные ошибки репликации в таблицу Out-GridView:
```powershell
Get-ADReplicationPartnerMetadata -Target * -Partition * | Select-Object Server,Partition,Partner,ConsecutiveReplicationFailures,LastReplicationSuccess,LastRepicationResult | Out-GridView
```

###### Рассмотрим как проверить состояние других базовых служб и сервисов контроллера домена.

С помощью командлета Get-Service проверьте состояние служб на контроллере домена:
```powershell
Active Directory Domain Services (ntds)
Active Directory Web Services (adws) – именно к этой службе подключаются все командлеты из модуля AD PowerShell
DNS (dnscache и dns)
Kerberos Key Distribution Center (kdc)
Windows Time Service (w32time)
NetLogon (netlogon)

Get-Service -name ntds,adws,dns,dnscache,kdc,w32time,netlogon
```
###### Базовый экспорт одной зоны
Самый простой способ — использовать командлет Export-DnsServerZone. Он экспортирует данные зоны в текстовый файл на диске.

```powershell
Export-DnsServerZone -Name "имя_зоны.com" -FileName "C:\Backup\DNS\имя_зоны.com.dns" -Force
```
-Name: Имя зоны, которую нужно экспортировать (например, contoso.local).

-FileName: Полный путь и имя файла, куда будет сохранена копия зоны.

-Force: Этот параметр перезаписывает существующий файл, если он уже есть .

Файл будет создан в текстовом формате, аналогичном стандартным файлам зон DNS.

###### Экспорт всех зон на сервере
Чтобы автоматизировать процесс и сохранить все зоны, которые обслуживает DNS-сервер, можно использовать следующий скрипт. Он находит все зоны и экспортирует каждую в отдельный файл.
```powershell
# Указываем папку для сохранения бэкапов
$BackupPath = "C:\Backup\DNS\"

# Проверяем, существует ли папка, и создаем её при необходимости
if (-not (Test-Path -Path $BackupPath)) {
    New-Item -ItemType Directory -Path $BackupPath
}

# Получаем список всех зон на сервере и экспортируем их
Get-DnsServerZone | ForEach-Object {
    $ZoneName = $_.ZoneName
    # Создаем безопасное имя файла, заменяя недопустимые символы (например, точки)
    $FileName = $ZoneName.Replace('.', '_') + ".dns"
    $FilePath = Join-Path -Path $BackupPath -ChildPath $FileName
    
    try {
        Export-DnsServerZone -Name $ZoneName -FileName $FilePath -Force
        Write-Host "Зона '$ZoneName' успешно экспортирована в $FilePath"
    }
    catch {
        Write-Error "Ошибка при экспорте зоны '$ZoneName': $_"
    }
}
```
###### скрипт для резервного копирования Stub-зон
```powershell
$BackupPath = "C:\temp\DNS_Stub_Backup"
$Date = Get-Date -Format "yyyyMMdd_HHmmss"
$FullBackupPath = "$BackupPath\$Date"
New-Item -ItemType Directory -Path $FullBackupPath -Force

$StubZones = Get-DnsServerZone | Where-Object { $_.ZoneType -eq "Stub" }

foreach ($Zone in $StubZones) {
    $ZoneName = $Zone.ZoneName
    $OutputFile = "$FullBackupPath\$ZoneName.txt"
    
    try {
        # Сохраняем конфигурацию Stub-зоны
        $Zone | Export-Clixml "$FullBackupPath\$ZoneName.config.xml"
        
        # Создаем текстовый файл с информацией о зоне
        "Stub Zone Configuration: $ZoneName" | Out-File $OutputFile
        "=" * 50 | Out-File $OutputFile -Append
        "Master Servers: $($Zone.MasterServers -join ', ')" | Out-File $OutputFile -Append
        "Zone Type: $($Zone.ZoneType)" | Out-File $OutputFile -Append
        "Is AD Integrated: $($Zone.IsDsIntegrated)" | Out-File $OutputFile -Append
        "" | Out-File $OutputFile -Append
        
        # Пробуем получить записи, которые есть в Stub-зоне
        $Records = Get-DnsServerResourceRecord -ZoneName $ZoneName -ErrorAction SilentlyContinue
        if ($Records) {
            "DNS Records in Stub Zone:" | Out-File $OutputFile -Append
            "-" * 30 | Out-File $OutputFile -Append
            foreach ($Record in $Records) {
                "$($Record.HostName) $($Record.RecordType) $($Record.RecordData)" | Out-File $OutputFile -Append
            }
        } else {
            "No records found or access denied" | Out-File $OutputFile -Append
        }
        
        Write-Host "Конфигурация Stub-зоны сохранена: $ZoneName" -ForegroundColor Green
    }
    catch {
        Write-Host "Ошибка для Stub-зоны $ZoneName : $($_.Exception.Message)" -ForegroundColor Red
    }
}
```
###### Единый скрипт полной миграции
```powershell
# ЕДИНЫЙ СКРИПТ ДЛЯ ПОЛНОЙ МИГРАЦИИ DNS
param(
    [string]$SourceBackupPath = "C:\temp\20251129_121416",
    [switch]$SkipPrimaryZones = $false,
    [switch]$SkipStubZones = $false
)

Write-Host "=== ПОЛНАЯ МИГРАЦИЯ DNS СЕРВЕРА ===" -ForegroundColor Cyan

# 1. Миграция основных зон
if (-not $SkipPrimaryZones) {
    Write-Host "`n1. МИГРАЦИЯ ОСНОВНЫХ ЗОН:" -ForegroundColor Yellow
    
    if (Test-Path $SourceBackupPath) {
        $PrimaryZoneFiles = Get-ChildItem "$SourceBackupPath\*.dns"
        Write-Host "Найдено файлов основных зон: $($PrimaryZoneFiles.Count)" -ForegroundColor White
        
        foreach ($File in $PrimaryZoneFiles) {
            $ZoneName = $File.BaseName
            try {
                # Копируем файл в системную папку DNS
                Copy-Item $File.FullName "C:\Windows\System32\dns\$($File.Name)" -Force
                
                # Создаем зону
                Add-DnsServerPrimaryZone -Name $ZoneName -ZoneFile $File.Name -ErrorAction Stop
                Write-Host "✓ Основная зона: $ZoneName" -ForegroundColor Green
            }
            catch {
                Write-Host "⚠ $ZoneName : $($_.Exception.Message)" -ForegroundColor Yellow
            }
        }
    } else {
        Write-Host "Папка с бэкапом не найдена: $SourceBackupPath" -ForegroundColor Red
    }
}

# 2. Миграция Stub-зон
if (-not $SkipStubZones) {
    Write-Host "`n2. МИГРАЦИЯ STUB-ЗОН:" -ForegroundColor Yellow
    
    # Данные Stub-зон (из вашего вывода)
    $StubZonesData = @(
        @{Name="chelstat.ru"; MasterServers=@("10.174.20.2")},
        @{Name="chtnpub.local"; MasterServers=@("10.175.20.10","10.175.20.11")},
        # ... добавьте все остальные Stub-зоны из предыдущего списка
        @{Name="yamalstat"; MasterServers=@("10.189.16.3","10.189.16.4")}
    )
    
    foreach ($ZoneConfig in $StubZonesData) {
        try {
            Add-DnsServerStubZone -Name $ZoneConfig.Name -MasterServers $ZoneConfig.MasterServers -ErrorAction Stop
            Write-Host "✓ Stub-зона: $($ZoneConfig.Name)" -ForegroundColor Green
        }
        catch {
            Write-Host "⚠ $($ZoneConfig.Name) : $($_.Exception.Message)" -ForegroundColor Yellow
        }
    }
}

Write-Host "`n=== МИГРАЦИЯ ЗАВЕРШЕНА ===" -ForegroundColor Cyan
$FinalCount = (Get-DnsServerZone | Where-Object {$_.ZoneName -ne "..TrustAnchors"}).Count
Write-Host "Итоговое количество зон: $FinalCount" -ForegroundColor Green
```
###### Просмотр информации об обратных зонах
```powershell
# Посмотреть все обратные зоны и их типы
$ReverseZones = Get-DnsServerZone | Where-Object { $_.IsReverseLookupZone -eq $true }

Write-Host "ОБРАТНЫЕ DNS ЗОНЫ:" -ForegroundColor Yellow
$ReverseZones | Format-Table ZoneName, ZoneType, IsDsIntegrated, IsReverseLookupZone -AutoSize

# Посмотреть статистику по обратным зонам
Write-Host "СТАТИСТИКА ОБРАТНЫХ ЗОН:" -ForegroundColor Cyan
$ReverseZones | Group-Object ZoneType | Format-Table Name, Count -AutoSize

# Примеры обратных зон (первые 5)
Write-Host "ПРИМЕРЫ ОБРАТНЫХ ЗОН:" -ForegroundColor Green
$ReverseZones | Select-Object -First 5 | ForEach-Object {
    Write-Host "  $($_.ZoneName) ($($_.ZoneType))" -ForegroundColor White
}
```
###### Скрипт для бэкапа обратных зон
```powershell
$BackupPath = "C:\temp\DNS_Backup_Reverse"
$Date = Get-Date -Format "yyyyMMdd_HHmmss"
$FullBackupPath = "$BackupPath\$Date"

# Создаем директорию
New-Item -ItemType Directory -Path $FullBackupPath -Force

# Получаем список всех ОБРАТНЫХ зон
$ReverseZones = Get-DnsServerZone | Where-Object { 
    $_.IsReverseLookupZone -eq $true -and 
    $_.ZoneName -ne "..TrustAnchors"
}

Write-Host "Найдено обратных зон для экспорта: $($ReverseZones.Count)" -ForegroundColor Yellow

# Экспортируем каждую обратную зону через DNSCMD
foreach ($Zone in $ReverseZones) {
    $ZoneName = $Zone.ZoneName
    try {
        # Используем DNSCMD для экспорта
        dnscmd . /ZoneExport $ZoneName "$ZoneName.dns"
        
        # Копируем файл из системной папки DNS
        $SourceFile = "C:\Windows\System32\dns\$ZoneName.dns"
        $DestFile = "$FullBackupPath\$ZoneName.dns"
        
        if (Test-Path $SourceFile) {
            Copy-Item $SourceFile $DestFile
            Write-Host "✓ Успешно: $ZoneName" -ForegroundColor Green
        } else {
            Write-Host "✗ Файл не создан: $ZoneName" -ForegroundColor Red
        }
    }
    catch {
        Write-Host "✗ Ошибка: $ZoneName - $($_.Exception.Message)" -ForegroundColor Red
    }
}

Write-Host "Резервное копирование ОБРАТНЫХ зон завершено! Файлы в: $FullBackupPath" -ForegroundColor Cyan
```
###### Для импорта обратных зон на целевой сервер
```powershell
# На ЦЕЛЕВОМ сервере для импорта обратных зон
$ReverseBackupPath = "C:\temp\DNS_Backup_Reverse\20251129_121416"  # Ваша папка с бэкапом

# Копируем файлы обратных зон в системную папку DNS
Get-ChildItem "$ReverseBackupPath\*.dns" | ForEach-Object {
    Copy-Item $_.FullName "C:\Windows\System32\dns\" -Force
}

# Создаем обратные зоны из файлов
Get-ChildItem "$ReverseBackupPath\*.dns" | ForEach-Object {
    $ZoneName = $_.BaseName
    
    # Проверяем, является ли зона обратной (обычно содержат in-addr.arpa или ip6.arpa)
    if ($ZoneName -match "in-addr\.arpa|ip6\.arpa") {
        try {
            Add-DnsServerPrimaryZone -Name $ZoneName -ZoneFile $_.Name -PassThru
            Write-Host "✓ Обратная зона создана: $ZoneName" -ForegroundColor Green
        }
        catch {
            Write-Host "⚠ Обратная зона уже существует или ошибка: $ZoneName" -ForegroundColor Yellow
        }
    }
}
```
###### Универсальный скрипт для бэкапа ВСЕХ зон (прямых + обратных)
```powershell
$BackupPath = "C:\temp\DNS_Backup_Complete"
$Date = Get-Date -Format "yyyyMMdd_HHmmss"
$FullBackupPath = "$BackupPath\$Date"

# Создаем директорию
New-Item -ItemType Directory -Path $FullBackupPath -Force

# Получаем список ВСЕХ зон (исключаем только системные)
$AllZones = Get-DnsServerZone | Where-Object { 
    $_.ZoneType -ne "Forwarder" -and 
    $_.ZoneName -ne "..TrustAnchors"
}

Write-Host "Найдено всех зон для экспорта: $($AllZones.Count)" -ForegroundColor Yellow

# Разделяем на прямые и обратные зоны
$ForwardZones = $AllZones | Where-Object { $_.IsReverseLookupZone -eq $false }
$ReverseZones = $AllZones | Where-Object { $_.IsReverseLookupZone -eq $true }

Write-Host "Прямые зоны: $($ForwardZones.Count)" -ForegroundColor Green
Write-Host "Обратные зоны: $($ReverseZones.Count)" -ForegroundColor Blue

# Создаем подпапки для организации
$ForwardPath = "$FullBackupPath\Forward"
$ReversePath = "$FullBackupPath\Reverse"
New-Item -ItemType Directory -Path $ForwardPath -Force
New-Item -ItemType Directory -Path $ReversePath -Force

# Функция для экспорта зон
function Export-Zone {
    param($Zone, $ExportPath)
    
    $ZoneName = $Zone.ZoneName
    try {
        dnscmd . /ZoneExport $ZoneName "$ZoneName.dns"
        
        $SourceFile = "C:\Windows\System32\dns\$ZoneName.dns"
        $DestFile = "$ExportPath\$ZoneName.dns"
        
        if (Test-Path $SourceFile) {
            Copy-Item $SourceFile $DestFile
            return "✓ Успешно: $ZoneName"
        } else {
            return "✗ Файл не создан: $ZoneName"
        }
    }
    catch {
        return "✗ Ошибка: $ZoneName - $($_.Exception.Message)"
    }
}

# Экспортируем прямые зоны
Write-Host "`nЭкспорт прямых зон:" -ForegroundColor Green
foreach ($Zone in $ForwardZones) {
    $Result = Export-Zone -Zone $Zone -ExportPath $ForwardPath
    Write-Host $Result -ForegroundColor $(if ($Result -like "✓*") { "Green" } else { "Red" })
}

# Экспортируем обратные зоны
Write-Host "`nЭкспорт обратных зон:" -ForegroundColor Blue
foreach ($Zone in $ReverseZones) {
    $Result = Export-Zone -Zone $Zone -ExportPath $ReversePath
    Write-Host $Result -ForegroundColor $(if ($Result -like "✓*") { "Green" } else { "Red" })
}

Write-Host "`nПолное резервное копирование завершено!" -ForegroundColor Cyan
Write-Host "Прямые зоны: $ForwardPath" -ForegroundColor Green
Write-Host "Обратные зоны: $ReversePath" -ForegroundColor Blue
```

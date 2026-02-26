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


Получить список DC в домене:

Get-ADDomainController

Получить список ошибок репликации на указанных DC:

Get-ADReplicationFailure -Target DC1,DC2

Вывести ошибки репликации AD в сайте (-Scope Site) или домене (-Scope Domain):

Get-ADReplicationFailure -scope site -target HQ| FT Server, LastError, Partner-Auto
Get-ADReplicationFailure -Target winitpro.ru -Scope Domain

Получить список партнеров по репликации текущего DC:

Get-ADReplicationConnection -Filter *

Чтобы выполнить принудительную репликацию используется командлет Sync-ADObject.

Например, следующая команда выведет все обнаруженные ошибки репликации в таблицу Out-GridView:

Get-ADReplicationPartnerMetadata -Target * -Partition * | Select-Object Server,Partition,Partner,ConsecutiveReplicationFailures,LastReplicationSuccess,LastRepicationResult | Out-GridView


Рассмотрим как проверить состояние других базовых служб и сервисов контроллера домена.

С помощью командлета Get-Service проверьте состояние служб на контроллере домена:

Active Directory Domain Services (ntds)
Active Directory Web Services (adws) – именно к этой службе подключаются все командлеты из модуля AD PowerShell
DNS (dnscache и dns)
Kerberos Key Distribution Center (kdc)
Windows Time Service (w32time)
NetLogon (netlogon)

Get-Service -name ntds,adws,dns,dnscache,kdc,w32time,netlogon

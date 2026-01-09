Принудительная регистрация DNS-записей:

На проблемном контроллере домена откройте командную строку с правами администратора и выполните команды:
```
cmd
ipconfig /flushdns
ipconfig /registerdns
```
Затем перезапустите службу Netlogon. Это заставит сервер повторно зарегистрировать все свои SRV- и A-записи в DNS.

***

Например, чтобы проверить корректность работы DNS на всех контроллерах домена, используйте команду:
```bash
dcdiag.exe /s:srv-dc01.domain.local /test:dns /e /v
```
на всех контроллерах (если все ОК, везде должно быть Pass). Если где-то будет указано Fail, нужно выполнить проверку этого теста на указанном DC:
```bash
dcdiag.exe /s:srv-dc01.domain.local /test:dns /DnsForwarders /v
```
Получить расширенную информацию по результатам тестов контроллера домена и сохранить ее в текстовый файл:
```bash
dcdiag /s:srv-dc01.domain.local /v >> c:\ps\dc01_dcdiag_test.log
```
Следующая команда PowerShell позволяет вывести только информацию о результатах выполненных тестов (Passed или Failed):
```bash
Dcdiag /s:srv-dc01.domain.local | select-string -pattern '\. (.*) \b(passed|failed)\b test (.*)'
```
Опросить состояние всех контроллеров в домене:
```bash
dcdiag.exe /s:winitpro.ru /a
```
Чтобы не загружать вывод dcdiag лишней информацией об успешных проверках, можно опросить состояние контроллеров домена и AD с выводом только ошибок (параметр /q), которые требуют внимания администратора:
```bash
dcdiag /q /e /v
```
Или ошибки для конкретного DC:
```bash
dcdiag.exe /s:srv-dc01 /q
```
У команды dcdiag есть опция /fix, позволяющая исправить тривиальные ошибки в AD на конкретном DC:
```bash
dcdiag.exe /s:srv-dc01 /fix
```
На всех контроллерах домена:
```bash
dcdiag /e /fix
```
****
```
Directory Server Diagnosis

dcdiag.exe /s:<Directory Server>[:<LDAP Port>] [/u:<Domain>\<Username> /p:*|<Password>|""]
           [/hqv] [/n:<Naming Context>] [/f:<Log>] [/x:XMLLog.xml]
           [/skip:<Test>] [/test:<Test>]
   /h: Display this help screen

   /s: Use <Directory Server> as Home Server. Ignored for DcPromo and
       RegisterInDns tests which can only be run locally.
   /n: Use <Naming Context> as the Naming Context to test
       Domains may be specified in Netbios, DNS or DN form.
   /u: Use domain\username credentials for binding.
       Must also use the /p option

   /p: Use <Password> as the password.  Must also use the /u option
   /a: Test all the servers in this site
   /e: Test all the servers in the entire enterprise.  Overrides /a
   /q: Quiet - Only print error messages
   /v: Verbose - Print extended information
   /i: ignore - ignores superfluous error messages.
   /c: Comprehensive, runs all tests, including non-default tests but excluding
       DcPromo and RegisterInDNS. Can use with /skip
   /fix: fix - Make safe repairs.
   /f: Redirect all output to a file <Log> seperately
   /x:<XMLLog.xml> Redirect xml output to <XMLLog.xml>. Currently works with /test:dns option only
   /xsl:<xslfile.xsl or xsltfile.xslt> Adds the processing instructions that references specified stylesheet. Works
with /test:dns /x:<XMLLog.xml> option only

   /test:<TestName> - Test only this test.  Required tests will still
                      be run.  Do not mix with /skip.

   /skip:<TestName> - Skip the named test.  Required tests will still
                      be run.  Do not mix with /test.

   The list of known tests:

        Advertising
             Checks whether each DSA is advertising itself, and whether it is advertising itself as having the
            capabilities of a DSA.

        CheckSDRefDom
             This test checks that all application directory partitions have appropriate security descriptor reference
            domains.

        CheckSecurityError
             Locates security errors (or those possibly security related) and performs the initial diagnosis of the
            problem. Optional Arguments: /ReplSource:<Source DC> to target a specific source, regardless of it's error
            status.  Need not be a current partner.
          * Test is not run by default, i.e. it must be requested explicitly

        Connectivity
             Tests whether DSAs are DNS registered, pingeable, and have  LDAP/RPC connectivity.
          * Test cannot be skipped
          * Test is applicable to AD/LDS

        CrossRefValidation
             This test looks for cross-refs that are in some way invalid.
          * Test is applicable to AD/LDS

        CutoffServers
             Check for servers that won't receive replications because its partners are down
          * Test is not run by default, i.e. it must be requested explicitly
          * Test is applicable to AD/LDS

        DcPromo
             Tests the existing DNS infrastructure for promotion to domain controller. If the infrastructure is
            sufficient, the computer can be promoted to domain controller in a domain specified in
            <Active_Directory_Domain_DNS_Name>. Reports whether any modifications to the existing DNS infrastructure
            are required. Required argument: /DnsDomain:<Active_Directory_Domain_DNS_Name> One of the following
            arguments is required: /NewForest /NewTree /ChildDomain /ReplicaDC If NewTree is specified, then the
            ForestRoot argument is required: /ForestRoot:<Forest_Root_Domain_DNS_Name>

        DNS
             This test checks the health of DNS settings for the whole enterprise. Sub tests can be run individually
            using the switches below. By default, all tests except external name resolution are run)         /DnsBasic
                      (basic tests, can't be skipped)         /DnsForwarders            (forwarders and root hints
            tests)         /DnsDelegation            (delegations tests)         /DnsDynamicUpdate            (dynamic
            update tests)         /DnsRecordRegistration            (records registration tests)
            /DnsResolveExtName            (external name resolution test)         /DnsAll            (includes all
            tests above)         /DnsInternetName:          <internet name> (for test /DnsResolveExtName)
            (default is www.microsoft.com)
          * Test is not run by default, i.e. it must be requested explicitly

        FrsEvent
             This test checks to see if there are any operation errors in the file replication system (FRS).  Failing
            replication of the SYSVOL share, can cause Policy problems.

        DFSREvent
             This test checks to see if there are any operation errors in the DFS.

        SysVolCheck
             This test checks that the SYSVOL is ready.

        LocatorCheck
             Checks that global role-holders are known, can be located, and are responding.

        Intersite
             Checks for failures that would prevent or temporarily hold up intersite replication.

        KccEvent
             This test checks that the Knowledge Consistency Checker is completing without errors.
          * Test is applicable to AD/LDS

        KnowsOfRoleHolders
             Check whether the DSA thinks it knows the role holders, and prints these roles out in verbose mode.

        MachineAccount
             Check to see if the Machine Account has the proper information. Use /RecreateMachineAccount to attempt a
            repair if the local machine account is missing. Use /FixMachineAccount if the machine account flags are
            incorrect.

        NCSecDesc
             Checks that the security descriptosrs on the naming context heads have appropriate permissions for
            replication.

        NetLogons
             Checks that the appropriate logon priviledges allow replication to proceed.

        ObjectsReplicated
             Check that Machine Account (AD only) and DSA objects have replicated. Use /objectdn:<dn> with /n:<nc> to
            specify an additional object to check.
          * Test is applicable to AD/LDS

        OutboundSecureChannels
             See if we have secure channels from all of the DC's in the domain the domains specified by /testdomain:.
            /nositerestriction will prevent the test from being limited to the DC's in the site.
          * Test is not run by default, i.e. it must be requested explicitly

        RegisterInDNS
             Tests whether this directory server can register the directory Server Locator DNS records. These records
            must be present in DNS in order for other computers to locate this directory server for the
            <Active_Directory_Domain_DNS_Name> domain. Reports whether any modifications to the existing DNS
            infrastructure are required. Required argument: /DnsDomain:<Active_Directory_Domain_DNS_Name>

        Replications
             Checks for timely replication between directory servers.
          * Test is applicable to AD/LDS

        RidManager
             Check to see if RID master is accessable and to see if it contains the proper information.

        Services
             Check to see if appropriate supporting services are running.
          * Test is applicable to AD/LDS

        SystemLog
             This test checks that the system is running without errors.
          * Test is applicable to AD/LDS

        Topology
             Checks that the generated topology is fully connected for all DSAs.
          * Test is not run by default, i.e. it must be requested explicitly
          * Test is applicable to AD/LDS

        VerifyEnterpriseReferences
             This test verifys that certain system references are intact for the FRS and Replication infrastructure
            across all objects in the enterprise on each DSA.
          * Test is not run by default, i.e. it must be requested explicitly

        VerifyReferences
             This test verifys that certain system references are intact for the FRS and Replication infrastructure.

        VerifyReplicas
             This test verifys that all application directory partitions are fully instantiated on all replica servers.
          * Test is not run by default, i.e. it must be requested explicitly
          * Test is applicable to AD/LDS


        All tests except DcPromo and RegisterInDNS must be run on computers
        after they have been promoted to directory server.

        Note: Text (Naming Context names, server names, etc) with International or
        Unicode characters will only display correctly if appropriate fonts and
        language support are loaded

C:\Windows\system32>

```
****
```
C:\Windows\system32>dcdiag /?

 Диагностика сервера каталогов

dcdiag.exe /s:<Сервер каталогов>[:<Порт LDAP>] [/u:<Домен>\<Имя_пользователя> /p:*|<Пароль>|""]
           [/hqv] [/n:<Контекст именования>] [/f:<Журнал>] [/x:XMLLog.xml]
           [/skip:<Проверка>] [/test:<Проверка>]
   /h: Вывод этого экрана справки

   /s: Используйте <Сервер каталогов> в качестве основного сервера. Пропускается для проверок DcPromo и
       RegisterInDns, которые могут выполняться только локально.
   /n: Используйте <Контекст именования> как контекст именования для проверки
       Домены можно указывать в форматах Netbios, DNS или DN.
   /u: Используйте для привязки учетные данные вида: домен\имя_пользователя.
       Необходимо использовать также параметр /p

   /p: Используйте <Пароль> в качестве пароля.  Необходимо использовать также параметр /u
   /a: Проверка всех серверов данного сайта
   /e: Проверка всех серверов предприятия в целом.  Перекрывает /a
   /q: Тихий режим - печать только сообщений об ошибках
   /v: Подробно - печать расширенных сведений
   /i: игнорировать - избыточные сообщения об ошибках игнорируются.
   /c: В полном объеме выполняются все проверки, в том числе нестандартные, за исключением
       DcPromo и RegisterInDNS. Можно использовать с параметром /skip
   /fix: исправлять - выполняются безопасные исправления.
   /f: Перенаправляет весь вывод отдельно в файл <Журнал>
   /x:<XMLLog.xml> Перенаправляет вывод xml в <XMLLog.xml>. В настоящее время работает только с параметром /test:dns
   /xsl:<XSL-файл.xsl или XSLT-файл.xslt> Добавляет инструкции по обработке, связанные с указанной таблицей стилей. Работает только с параметрами /test:dns и /x:<XMLLog.xml>

   /test:<Имя_проверки> - выполнить только данную проверку.  При этом обязательные проверки все равно будут
выполняться. Не следует путать с параметром /skip.

   /skip:<Имя_проверки> - пропустить указанную данную проверку.  При этом обязательные проверки все равно будут
выполняться. Не следует путать с параметром /test.

   Список известных проверок:

        Advertising
             Проверьте, выполняется ли объявление каждого DSA, и содержатся ли в этих объявлениях сведения о наличии возможностей DSA.

        CheckSDRefDom
             Проверка, позволяющая определить, всем ли разделам каталога приложений назначены соответствующие домены ссылок дескрипторов безопасности.

        CheckSecurityError
             Поиск действительных или предполагаемых ошибок безопасности и начальная диагностика проблемы. Дополнительные аргументы: /ReplSource:<исходный_контроллер_домена> для
            определения источника независимо от состояния ошибки.  Может не быть текущим партнером.
          * Проверка не выполняется по умолчанию, т. е. она должна быть запрошена явно

        Connectivity
             Проверка наличия регистрации DNS для DSA, успешного  выполнения команды ping и подключения LDAP/RPC.
          * Проверка не может быть пропущена
          * Проверка применима к AD/LDS

        CrossRefValidation
             Проверка, позволяющая найти недопустимые перекрестные ссылки.
          * Проверка применима к AD/LDS

        CutoffServers
             Поиск серверов репликации, которые не могут стать принимающими, так как их партнеры отключены.
          * Проверка не выполняется по умолчанию, т. е. она должна быть запрошена явно
          * Проверка применима к AD/LDS

        DcPromo
             Проверка существующей инфраструктуры DNS для повышения роли до контроллера домена. При наличии достаточной инфраструктуры роль компьютера может быть повышена до
            контроллера домена в домене, заданном параметром <имя_домена_DNS_Active_Directory>. Если требуется изменить существующую инфраструктуру DNS, выдается уведомление.
            Обязательный аргумент: /DnsDomain:<имя_домена_DNS_Active_Directory> Требуется также один из следующих аргументов: /NewForest /NewTree /ChildDomain /ReplicaDC Если указан
            аргумент NewTree, требуется аргумент  ForestRoot: /ForestRoot:<имя_домена_DNS_корня_леса>

        DNS
             Проверка работоспособности параметров DNS для всего предприятия. Можно запускать отдельные проверки со следующими ключами. По умолчанию выполняются все проверки, кроме
            внешнего разрешения имен.         /DnsBasic            (базовые проверки, не могут быть пропущены)         /DnsForwarders            (проверки серверов пересылки и
            корневых ссылок)         /DnsDelegation            (проверки делегирования)         /DnsDynamicUpdate            (проверки динамического обновления)
            /DnsRecordRegistration            (проверки регистрации записей)         /DnsResolveExtName            (проверки внешнего разрешения имен)         /DnsAll
            (все проверки, перечисленные выше)         /DnsInternetName:          <интернет-имя> (для проверки /DnsResolveExtName)         (по умолчанию www.microsoft.com)
          * Проверка не выполняется по умолчанию, т. е. она должна быть запрошена явно

        FrsEvent
             Проверка, позволяющая найти ошибки операций в службе репликации файлов (FRS).  Сбои при репликации общего ресурса SYSVOL могут стать причиной проблем политики.

        DFSREvent
             Проверка, позволяющая найти ошибки операций в системе DFS.

        SysVolCheck
             Данный тест служит для проверки готовности SYSVOL.

        LocatorCheck
             Проверка того, что глобальные владельцы ролей известны, могут быть найдены и отвечают.

        Intersite
             Проверка на наличие ошибок, вызывающих сбой или задержку межсайтовой репликации.

        KccEvent
             Проверка, показывающая, что проверка согласованности знаний завершается без ошибок.
          * Проверка применима к AD/LDS

        KnowsOfRoleHolders
             Проверка того, что DSA содержит сведения о владельцах ролей и выводит их в режиме подробного протоколирования.

        MachineAccount
             Проверка правильности сведений учетной записи компьютера.  Если учетная запись локального компьютера отсутствует, введите для исправления ошибки
            "/RecreateMachineAccount". Если флаги учетной записи компьютера неправильны, введите "/FixMachineAccount".

        NCSecDesc
             Проверка того, что у дескрипторов безопасности в заголовках контекста именования есть соответствующие разрешения для репликации.

        NetLogons
             Проверка того, что репликация разрешена соответствующими правами на вход в систему.

        ObjectsReplicated
             Проверка завершения репликации учетной записи компьютера (только Active Directory) и объекта DSA. Для проверки дополнительного объекта используйте "/objectdn:<dn>" и
            "/n:<nc>".
          * Проверка применима к AD/LDS

        OutboundSecureChannels
             Проверка наличия безопасных каналов от всех контроллеров домена в домене к доменам, заданным командой /testdomain:. Параметр /nositerestriction позволяет выполнить
            проверку, не ограниченную только контроллерами домена в сайте.
          * Проверка не выполняется по умолчанию, т. е. она должна быть запрошена явно

        RegisterInDNS
             Проверка того, что данный сервер каталога может регистрировать записи обнаружения серверов каталогов DNS. Эти записи должны находиться в DNS, чтобы другие компьютеры
            могли обнаружить этот сервер каталога в домене <имя_домена_DNS_Active_Directory>.  Если требуется изменить существующую инфраструктуру DNS, выдается уведомление.
            Обязательный аргумент: /DnsDomain:<имя_домена_DNS_Active_Directory>

        Replications
             Выполняется проверка своевременной репликации между серверами каталогов.
          * Проверка применима к AD/LDS

        RidManager
             Проверка того, что хозяин RID доступен и содержит правильные сведения.

        Services
             Проверка того, что выполняются соответствующие службы поддержки.
          * Проверка применима к AD/LDS

        SystemLog
             Проверка того, что система функционирует без ошибок.
          * Проверка применима к AD/LDS

        Topology
             Проверка того, что созданная топология полностью подключена для всех DSA.
          * Проверка не выполняется по умолчанию, т. е. она должна быть запрошена явно
          * Проверка применима к AD/LDS

        VerifyEnterpriseReferences
             Проверка того, что не изменились определенные системные ссылки службы FRS и инфраструктуры репликации для всех объектов предприятия в каждом DSA.
          * Проверка не выполняется по умолчанию, т. е. она должна быть запрошена явно

        VerifyReferences
             Проверка того, что не изменились определенные системные ссылки службы FRS и инфраструктуры репликации.

        VerifyReplicas
             Проверка того, что для всех разделов каталога приложений полностью созданы экземпляры на всех репликах сервера.
          * Проверка не выполняется по умолчанию, т. е. она должна быть запрошена явно
          * Проверка применима к AD/LDS


        Все проверки, за исключением DcPromo и RegisterInDNS, следует выполнять на компьютерах,
        повышенных до уровня сервера каталогов.

        Примечание. Текст (имена контекста именования, имена серверов и др.), содержащий
        знаки национальных алфавитов или Юникода, будет отображаться
        правильно только в том случае, если загружены соответствующие
шрифты и языковая поддержка.

C:\Windows\system32>
```

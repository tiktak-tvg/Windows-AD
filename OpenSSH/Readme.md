#### Вариант 1
#### Установка клиента OpenSSH в Windows 10
Клиент OpenSSH входит в состав Features on Demand Windows 10 (как и RSAT). Клиент SSH установлен по умолчанию в Windows Server 2019 и Windows 10 1809 и более новых билдах.

Проверьте, что SSH клиент установлен:
```powershell
Get-WindowsCapability -Online | ? Name -like 'OpenSSH.Client*'

'OpenSSH.Client установка в windows 10
```
В нашем примере клиент OpenSSH установлен (статус: State: Installed).

Если SSH клиент отсутствует (State: Not Present), его можно установить:

С помощью команды PowerShell: 
```powershell
Add-WindowsCapability -Online -Name OpenSSH.Client*
```
С помощью DISM: 
```powershell
dism /Online /Add-Capability /CapabilityName:OpenSSH.Client~~~~0.0.1.0
```
```bash
Через Параметры -> Приложения -> Дополнительные возможности -> Добавить компонент. Найдите в списке Клиент OpenSSH и нажмите кнопку Установить.
клиент openssh установить компонент
```
Бинарные файлы OpenSSH находятся в каталоге c:\windows\system32\OpenSSH\.
```bash
ssh.exe – это исполняемый файл клиента SSH;
scp.exe – утилита для копирования файлов в SSH сессии;
ssh-keygen.exe – утилита для генерации ключей аутентификации;
ssh-agent.exe – используется для управления ключами;
ssh-add.exe – добавление ключа в базу ssh-агента.
исполняемые файлы OpenSSH
```
#### Вариант 2 
#### Настроить Windows 10 как SSH-сервер (Входящие соединения)
Чтобы другие устройства могли подключаться к вашему компьютеру, выполните три шага.
#### Шаг 1: Установка компонента
Нажмите правой кнопкой мыши на «Пуск» и запустите Windows PowerShell (администратор).Вставьте следующую команду и нажмите Enter:
```powershell
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
```
#### Шаг 2: Запуск и настройка автозапуска службыОставаясь в PowerShell (от имени администратора), выполните поочередно две команды:Включение автоматического запуска при загрузке системы:
```powershell
Set-Service -Name sshd -StartupType 'Automatic'
```
Запуск службы прямо сейчас:
```powershell
Start-Service sshd
```
#### Шаг 3: Открытие порта в брандмауэреОбычно Windows открывает порт автоматически при установке. 
Если подключение блокируется, выполните в PowerShell команду для открытия порта 22:
```powershell
New-NetFirewallRule -Name 'OpenSSH-In-TCP' -DisplayName 'OpenSSH Server (Inbound)' -Direction Inbound -Action Allow -Protocol TCP -LocalPort 22
```
Теперь к вашему компьютеру на Windows 10 можно подключиться по сети, используя ваш IP-адрес и данные учетной записи Windows (логин и пароль).

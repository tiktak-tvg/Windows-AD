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

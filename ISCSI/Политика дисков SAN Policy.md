### Политика дисков SAN Policy
В Windows имеется специальная политика дисков ``SAN Policy``, которая определяет, нужно ли автоматически монтировать диски при их подключении к хосту.
```cmd
Offline (The disk is offline because of policy set by an administrator).
```
Текущую настройку ``SAN Policy`` можно получить с помощью ``diskpart``. По умолчанию используется SAN политика ``Offline Shared``.

Значение SAN policy:

| Значение     | Назначение |
| :---         | :---       |
OfflineAll	  | Все диски по умолчанию в offline режиме
OfflineInternal	  | Все диски на внутренних шинах в offline
OfflineShared	  | Все диски, подключенные через iSCSI, FC или SAS в offline
OnlineAll	  | Все диски автоматически переводятся в онлайн режим (рекомендуется)

> [!NOTE]
> Из-за этой политики SAN внешние диски, подключенные с СХД, могут при перезагрузке находиться в offline режиме.<br>
> Чтобы автоматически монтировать диски, нужно изменить значение SAN Policy на OnlineAll.

На одном из серверов с ``Windows Server 2016`` после каждой перезагрузки сервера отключается дополнительный диск (не системный), подключенный в виде LUN с SAN хранилища по FC. 
Если открыть консоль управления дисками ``diskmgmt.msc``, можно увидеть, что данный диск находится в автономном режиме ``Offline``.

<img width="1095" height="603" alt="image" src="https://github.com/user-attachments/assets/bb353718-6912-479b-83fc-2ac605bc1cfb" />

Такая проблема может наблюдаться в кластерах или на виртуальных машинах с Windows, на которых общие диски могут быть доступны нескольким операционным системам. Это связано с наличием специальной политики SAN Policy, которая впервые появилась в Windows Server 2008. Эта политика управляет автоматическим монтированием внешних дисков и используется для защиты общих дисков, которые доступны нескольким серверам одновременно. По умолчанию в Windows Server для всех SAN дисков, кроме загрузочного, используется политика ``Offline Shared (VDS_SP_OFFLINE_SHARED)``. Вы можете изменить ``SAN Policy на OnlineAll`` с помощью ``Diskpart``.

Чтобы сделать этот диск доступным в Windows нужно щелкнуть по нему ПКМ и перевести в режим Online. Это придется делать при каждой перезагрузке сервера.

Отройте командную строку с правами администратора и выполните команду diskpart . В контексте diskpart выведите текущую политику SAN:
```bash
DISKPART>san
SAN Policy : Offline Shared
```
#### Измените политику SAN Policy:
```bash
DISKPART> san policy=OnlineAll

DiskPart successfully changed the SAN policy for the current operating system.
```
<img width="1168" height="504" alt="image" src="https://github.com/user-attachments/assets/bad35f1b-03e7-4140-9160-b411d9b10278" />

#### Еще раз проверим текущую политику:
```bash
DISKPART> san
SAN Policy : Online All
```
#### Выберите ваш диск (в нашем примере индекс диска 7):
```bash
DISKPART>select disk 7
```
#### Можете проверить его атрибуты:
```bash
DISKPART>attributes disk
```
<img width="1160" height="318" alt="image" src="https://github.com/user-attachments/assets/c9cd4914-93e0-4b4f-8a37-86968a256303" />

#### Далее
```bash
DISKPART>attributes disk clear readonly
```
#### Переведите диск в online режим:
```bash
DISKPART>online disk

DiskPart successfully onlined the selected disk
```

<img width="1161" height="326" alt="image" src="https://github.com/user-attachments/assets/f3884034-93c9-4a83-ba53-0b35e2757f7b" />

 
 #### Управлять дисками можно не только из Diskpart, но и с помощью встроенного PowerShell модуля Storage. 

 <img width="658" height="582" alt="48" src="https://github.com/user-attachments/assets/c39d52c7-8ae2-424c-827e-afb53f516813" />

 > Например, чтобы перевести диск в онлайн я выполнял команду:

```PowerShell
Set-Disk -Number 1 -IsReadOnly $false
Set-Disk -Number 2 -IsReadOnly $false
Set-Disk -Number 3 -IsReadOnly $false
Set-Disk -Number 7 -IsReadOnly $false
```
<img width="1163" height="258" alt="image" src="https://github.com/user-attachments/assets/3c4908ec-e65e-405b-a5bb-c5ec34c6593d" />




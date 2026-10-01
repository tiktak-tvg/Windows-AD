### Настройка и подключение iSCSI-хранилища
Добавляем диск для LUN

<img width="1004" height="812" alt="image" src="https://github.com/user-attachments/assets/388f71ee-c51d-4f8b-a914-cbf5333ffda2" />

<img width="1060" height="778" alt="image" src="https://github.com/user-attachments/assets/2554bf41-7ba6-4a5b-aeb3-324bf130ba7d" />

На хосте Windows Server, который будет предоставлять доступ к своему хранилищу нужно включить iSCSI target (Сервер целей iSCSI)

Так как у меня сервер Windows 2016 я добавляю через вкладку Диспетчер серверов - Роли сервера. В Windows 10  - Роли и компоненты.

<img width="1117" height="802" alt="image" src="https://github.com/user-attachments/assets/a8ca9a6b-d3c8-47a0-b581-f428beffddc9" />

В Windows 10 так

<img width="1119" height="323" alt="image" src="https://github.com/user-attachments/assets/730f056b-1805-40be-b1eb-e55e3779f9e0" />

У кого не добавлено, добавляйте, так как я добавил ранее.

После добавления появится служба ``Microsoft iSCSI Target Server``

<img width="1295" height="806" alt="image" src="https://github.com/user-attachments/assets/47108edf-1ac4-4e9b-9337-9c02e6f06910" />

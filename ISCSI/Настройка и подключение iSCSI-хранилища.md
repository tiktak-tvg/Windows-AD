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

Теперь на iSCSI сервере нужно создать виртуальный диск из нашего добавленного ранее диска. Перейдите в Server Manager -> File and Storage Services -> iSCSI, нажмите New iSCSI Virtual Disk. Можно сразу запустить из центра нажав на надпись.

<img width="1894" height="420" alt="image" src="https://github.com/user-attachments/assets/91a489f9-549c-4ce6-afd1-6231ee8476b2" />

Перед созданием виртуального диска, нужно его сначала инициализировать

<img width="1232" height="710" alt="image" src="https://github.com/user-attachments/assets/b6a20ba8-09cc-4394-bf59-42857452f3ef" />

Иначе он его не увидит

<img width="1352" height="682" alt="image" src="https://github.com/user-attachments/assets/1d1e9fed-29c1-4669-8674-60f87fe8eda4" />

При создании диска можно указать его полный размер или который вам нужен и тип диска.

- **Fixed Size** – диск фиксированного размера, который при создании сразу занимает все выделенное для него место. Это формат диска обеспечивает лучшую производительность и подходит для продуктивных систем с высокой дисковой активностью и повышенными требованиями к IOPS
- **Dynamically expanding** – такой диск занимает изначально немного место и расширяется по мере записи в него. Такой тип диска позволяет сэкономить место на хранилище, но уступает в скорости фиксированным дискам из за необходимости постоянного расширения
- **Differencing** – дифференциальный (или разностный диск), который основывается на некотором родительском диске и содержит изменения относительного родительского (используется крайне редко, обычно в сценариях виртуализации с базовым образом/VDI).



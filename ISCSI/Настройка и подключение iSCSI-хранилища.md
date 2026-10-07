### Настройка и подключение iSCSI-хранилища
Добавляем диск для LUN

<img width="1004" height="812" alt="image" src="https://github.com/user-attachments/assets/388f71ee-c51d-4f8b-a914-cbf5333ffda2" />

<img width="1060" height="778" alt="image" src="https://github.com/user-attachments/assets/2554bf41-7ba6-4a5b-aeb3-324bf130ba7d" />

На хосте Windows Server, который будет предоставлять доступ к своему хранилищу нужно включить iSCSI target (Сервер целей iSCSI)

Так как у меня сервер Windows 2016 я добавляю через вкладку Диспетчер серверов - Роли сервера. В Windows 10  - Роли и компоненты.

<img width="1117" height="802" alt="image" src="https://github.com/user-attachments/assets/a8ca9a6b-d3c8-47a0-b581-f428beffddc9" />

В Windows 10 так

<img width="1119" height="323" alt="image" src="https://github.com/user-attachments/assets/730f056b-1805-40be-b1eb-e55e3779f9e0" />

#### У кого не добавлено, добавляйте, так как я добавил ранее.

#### После добавления, активируется служба ``Microsoft iSCSI Target Server``

<img width="1295" height="806" alt="image" src="https://github.com/user-attachments/assets/47108edf-1ac4-4e9b-9337-9c02e6f06910" />

Теперь на iSCSI сервере нужно создать виртуальный диск из нашего добавленного ранее диска. Перейдите в Server Manager -> File and Storage Services -> iSCSI, нажмите New iSCSI Virtual Disk. Можно сразу запустить из центра нажав на надпись.

<img width="1894" height="420" alt="image" src="https://github.com/user-attachments/assets/91a489f9-549c-4ce6-afd1-6231ee8476b2" />

Перед созданием виртуального диска, нужно его сначала инициализировать

<img width="1232" height="710" alt="image" src="https://github.com/user-attachments/assets/b6a20ba8-09cc-4394-bf59-42857452f3ef" />

#### Иначе он его не увидит

<img width="1352" height="682" alt="image" src="https://github.com/user-attachments/assets/1d1e9fed-29c1-4669-8674-60f87fe8eda4" />

Теперь можно выбирать. Если не появились, перезайдите в Диспетчер серверов.

<img width="1258" height="742" alt="image" src="https://github.com/user-attachments/assets/e6d93c97-38f5-4fe6-ad03-ede8dd675beb" />

<img width="1242" height="749" alt="image" src="https://github.com/user-attachments/assets/65b2a8e7-dcd1-4365-9f96-89228ca9f1b8" />

#### При создании диска можно указать его полный размер или который вам нужен и тип диска. Я выбрал полный размер и фиксированный.

- **Fixed Size** – диск фиксированного размера, который при создании сразу занимает все выделенное для него место. Это формат диска обеспечивает лучшую производительность и подходит для продуктивных систем с высокой дисковой активностью и повышенными требованиями к IOPS
- **Dynamically expanding** – такой диск занимает изначально немного место и расширяется по мере записи в него. Такой тип диска позволяет сэкономить место на хранилище, но уступает в скорости фиксированным дискам из за необходимости постоянного расширения
- **Differencing** – дифференциальный (или разностный диск), который основывается на некотором родительском диске и содержит изменения относительного родительского (используется крайне редко, обычно в сценариях виртуализации с базовым образом/VDI).

<img width="1230" height="747" alt="image" src="https://github.com/user-attachments/assets/ce6f6d7a-3383-4b53-8c40-89f39460574e" />

Если вы присоединяете потерянный LUN, то лучше снять эту галочку

<img width="1233" height="748" alt="image" src="https://github.com/user-attachments/assets/28581a07-4191-4695-8c18-25e098ead48e" />

Так как у меня уже существует Целевой объект и я подключал ранее воспользуюсь им. Кто не подключал, для начала надо его настроить подключение iSCSI хранилища (LUN) в VMWare ESXi.

<img width="1227" height="748" alt="image" src="https://github.com/user-attachments/assets/5ab2801b-e1bf-41e1-8f81-37de15636b46" />

<img width="1232" height="746" alt="image" src="https://github.com/user-attachments/assets/f595379c-1f28-48f6-9eba-8851fbcea272" />

#### Создали

<img width="1461" height="784" alt="image" src="https://github.com/user-attachments/assets/99de6284-af78-479b-9d1b-454becaf1584" />

#### Далее подключаем

<img width="1434" height="291" alt="image" src="https://github.com/user-attachments/assets/a46e57ec-3dca-4ba5-8a00-869714367b8c" />

#### Подключил

<img width="1547" height="551" alt="image" src="https://github.com/user-attachments/assets/8fe20c64-e2d4-4eb3-84e9-607398eef73a" />

<img width="1581" height="270" alt="image" src="https://github.com/user-attachments/assets/4fa7a8a3-0bf3-4374-8570-cf01feceaacf" />

<img width="1572" height="312" alt="image" src="https://github.com/user-attachments/assets/1df9b7d0-e086-4a87-96b4-80d4b3bcc944" />

#### LUN появился. Если требуется узнать какой именно LUN вы подключили, если их много подключено, то смотрите серийный номер.
Например, если подключен не один сервер

<img width="1151" height="697" alt="image" src="https://github.com/user-attachments/assets/39691943-fbe1-4467-8a2b-0ffb9cada170" />

<img width="1447" height="716" alt="image" src="https://github.com/user-attachments/assets/9b3f2970-879d-422c-be03-b3cc86f842c5" />

<img width="1167" height="469" alt="image" src="https://github.com/user-attachments/assets/dc1cb463-15c7-4dd4-9b91-ed150024a73a" />

<img width="1613" height="307" alt="image" src="https://github.com/user-attachments/assets/97f3cdd8-6e57-48a0-a1d9-adc5cd7b288a" />

<img width="1365" height="730" alt="image" src="https://github.com/user-attachments/assets/daadd3eb-3b6d-4ee0-a95d-61b0c05b1709" />

#### Теперь на доступном iSCSI диске можно создать VMFS (Virtual Machine File System) хранилище для размещения файлов виртуальных машин. Создаем VMFS хранилище на iSCSI LUN в VMWare ESXi.

<img width="1245" height="382" alt="image" src="https://github.com/user-attachments/assets/eb97a1eb-031e-4044-b44c-6a49dd5d417f" />

<img width="1361" height="782" alt="image" src="https://github.com/user-attachments/assets/774783d1-0336-4c7c-8917-f00634728c9c" />

<img width="1342" height="770" alt="image" src="https://github.com/user-attachments/assets/957f0d63-49ee-42cf-abcf-bd49f52feeb1" />

<img width="1352" height="809" alt="image" src="https://github.com/user-attachments/assets/ea377005-223b-4b6b-ba4f-9d80045e0673" />

<img width="1158" height="582" alt="image" src="https://github.com/user-attachments/assets/5448c8fb-5581-407b-9f8a-0ff5bf73821d" />

<img width="1549" height="249" alt="image" src="https://github.com/user-attachments/assets/40b1a2b9-78b7-4cde-8472-15348db463c2" />

#### Всё создали VMFS хранилище и подключили!

#### Теперь можно установить на него виртуальную машину.

<img width="1426" height="608" alt="image" src="https://github.com/user-attachments/assets/2a49f0d3-d075-4c58-a8d8-463374b961b2" />

<img width="1377" height="640" alt="image" src="https://github.com/user-attachments/assets/bfe66623-d713-481e-b4dd-22b045d6f98d" />

<img width="1398" height="833" alt="image" src="https://github.com/user-attachments/assets/5409da29-e184-4c70-9469-38bee1337347" />

<img width="1447" height="598" alt="image" src="https://github.com/user-attachments/assets/8f18b4b1-4f69-449d-a212-70df8e5873c7" />









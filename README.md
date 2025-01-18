# Web

## 0. Выбор архитектуры
Есть два основных подхода. Это монолит и микросервисы.

### Монолит: 
По сути это одно приложение, которое содержит разные функции/модули. 

Плюсы: 
  - Простая разработка и поддержка. Модули тесно связаны, поэтому писать, тестировать, дебажить такой код проще.

Минусы: 
  - Маштабируемость. Допустим вертикальное маштабирование уже не доступно. Остается только горизонтальное. Получается целое приложения нужно копировать на новый сервер. Хотя нагружен может быть всего пару модулей. 
  - Завязаны на определенном стеке технологий. Возможно если переписать модуль на другую технологию, то он будет выполняться быстрее используя меньшие ресурсы.
  - Любые изменения приложения требуют компиляции и развертывания. Если приложение большое, то это будет требовать много времени. 
  - Допустим если сервер, где крутится приложение приуныл, то всё приложение не будет работать. А если разные сервисы будут распределены по разным серверам и какой-то сервер откажет, то упадет только сервис на этом сервере.

### Микросервисы:
Модули монолита выделяют в отдельное приложение (сервис). Возможно оно будет на другой сервере и со своей базой данных. 

Плюсы: 
  - Маштабируемость. Можно маштабировать отдельно взятые сервисы, а не все приложение целиком. То есть можно добавить ресурсов серверу для конкретного сервиса. А для остальных оставить как есть. 
  - Можно использовать разные технологии для построения определенных сервисов. 
  - Компиляция и развертывание происходит намного быстрее.

Минусы: 
  - Сложности управления и мониторга. Допустим 500 сервисов. Как узнавать кто не работает? Как обновлять? Нужно больше людей для поддержания. 
  - Сложности взаимодействия команд. Если нужно доработать не только нашей команды, но и сервис еще одной команды, то придется лишний раз контактировать. 
  - Если изменить API у своего сервиса, то придется как то других оповещать. Либо вводить версионирование. А это дополнительные сложности в поддержке. 
  - Тяжело с транзакциями. 

### Итог: 
Выбор архитекторуры зависит от ситуации. Нужно смотреть.


## 1. Виды web приложений

### Первое. Один процесс. 
  1. Выделяется процесс + память. В этом процессе запущено приложение. 
  2. Пришло 2 запроса одновременно. 
  3. Приложение сначала обрабатывает первый запрос. Выполняет бизнес логику. Например делает HTTP запрос на другой сервис. Это операция ввода/вывода. 
  4. Пока делается запрос в другой сервис сам процесс ждет. То есть просто простаивает. 
  5. Допустим от другого сервиса пришел ответ, процесс продолжил выполнять нашу бизнес-логику. 
  6. Заканчиваем обрабатывать первый запрос.
  7. Начинаем обрабатывать второй запрос. 

Так как все делает один процесс, то если запросов будет 1000, то это будет все очень медленно выполняться. 


### Второе. Несколько процессов. 
  1. Выделяется процесс + память. В этом процессе запущено приложение. 
  2. Пришло 2 запроса одновременно. 
  3. Приложение создает еще один процесс. Процессу выделяется своя память. 
  4. В этом отдельном процессе выполняется бизнес-логика с вводом/выводом. Для первого запроса.
  5. Для второго запроса тоже самое. 
  6. Основной процесс продолжает принимать остальные запросы. 
 
Уже лучше чем первый вариант. Будет быстрее. Т.к. каждый запрос выполняется в своем процессе. Но каждый процесс имеет свою память. 
И если будет 1000 запросов, то потребуется больше памяти. 
Плюс ядро процессора может выполнять одну задачу в один момент времени. Это значит что ядро процессора будет выполнять сначала первый процесс, остановится на половине, потом переключиться на второй.
Второй тоже сделает наполовину, потом переключится на первый и так далее. 
На эти переключения тоже нужно время. Если будут 1000 процессов, то ядро процессора будет постоянно переключаться между ними. 
На многоядерных процессорах, 1000 процессов будут распределяться по всем ядрах. Выполняться процессы будут быстрее. 
  
### Третье. Несколько потоков. 
  1. Выделяется процесс + память. В этом процессе запущено приложение. 
  2. Пришло 2 запроса одновременно. 
  3. Приложение создает еще поток. Поток использует ту же память что и процесс от которого он создан. 
  4. В этом отдельном потоке выполняется бизнес-логика с вводом/выводом. Для первого запроса.
  5. Для второго запроса тоже самое. 
  6. Основной процесс продолжает принимать остальные запросы. 

Лучше чем предыдущие варианты. Так как поток использует ту же память что с процесс. А значит памяти нужно меньше. 
Но ядро процессора так же выполняет один поток в один момент времени. Это значит что если будет 1000 потоков, то ядро будет так же переключаться между ними. 

Так как создание потока тоже занимает время, то есть вариант сделать пул потоков. То есть в пуле сразу создается несколько потоков.


## 2. Проблема всех видов приложений. 
Поток/процесс большую часть времени ждет пока мы получим данные из другого сервиса или из БД. 
То есть поток ничего не делает в это время. Получается когда мы обращаемся к какому то сервису, происходит блокировка потока. Он ничего не делает.
Нужен механизм который заставлял поток/процесс делать что то другое, пока происходит операция ввода/вывода.

Для этого создали Even loop.

## 3. Event loop 
  1. Выделяется процесс + память. В этом процессе запущено приложение. 
  2. Пришло 2 запроса одновременно. 
  3. Процесс обрабатывает первый запрос. Начинает выполнять бизнес логику. Процесс говорит event loop, что сделай HTTP запрос на другой сервис и позови меня когда будет ответ с данными. 
  4. После чего процесс приступает к обработке следующего запроса.
  5. Когда данные из другого сервиса возвращаются, то процесс продолжает работать с этими данными. 
  6. Получается что процесс постоянно что-то делает. Он не простаивает, он не ждет пока закончится операция ввода/вывода. 
  
Минус в том, что если приложение выполняет сложные CPU задачи (не ввод/вывод) то весь event loop останавливается и занимается только этой задачей.
Выход только если одать эту тяжелую задачу другому сервису. То есть сделать ввод/вывод. Либо создать новый поток и отдать задачу ему. 
Плюс сложно поддерживать из-за большого количества коллбэков.

## 4. Асинхронное программирование в Java
В Java программист в основном работает с пулами потоков. Создает пулы. Помещает задачу в пул. Сначала задача помещается в очередь. Затем первый свободный поток забирает ее и выполняет. 


## 5. Асинхронное программирование в Go
В Go программист не работает с потоками. Он работает в горутинами. Горутина это абстракция над потоками. 
Встроенный диспечер управляет этими потоками. Он сам распределяет все горутины по потокам и так далее. 

## 6. Подитог: 
В любом случае выбор среды для создания проекта тесно связан с тем, насколько хорошо команда знакома с той или иной средой, а значит, и с общей потенциальной продуктивностью. 
Поэтому не для каждой команды будет целесообразно с головой погрузиться в разработку веб-приложений и сервисов на Node или Go. 
Одна из частых причин неиспользования тех или иных языков и/или сред — необходимость поиска разработчиков, знакомых с данным инструментом. 

## 7. Возможный стек: 

### Веб сервера:
  - apache
  - nginx

### Бекенд:
  - Java   | Компилируемый   | Есть сборщик мусора | Выполняется в виртуальной машине  
  - nodeJS | Интепретируемый | Есть сборщик мусора | Выполняется в движке V8 
  - C++    | Компилируемый   | Нет сборщика мусора | Компилируется под каждую платформу сразу в машиный код.
  - Python | Интепретируемый | Есть сборщик мусора | Выполняется в виртуальной машине  
  - Go     | Компилируемый   | Есть сборщик мусора | Компилируется под каждую платформу сразу в машиный код.
  - PHP    | Интепретируемый | Есть сборщик мусора | Выполняется в виртуальной машине  

### Очереди:
  - Kafka

### БД OLTP:
  - Oracle
  - PostgreSQL

### БД OLAP:
  - Колоночная?
  
### Список литературы: 
  - https://habr.com/ru/companies/vk/articles/329258/
  - https://youtu.be/AXkOli6BsBY?si=Yv_XvX7fwtxsEiMp
  - https://youtu.be/IB4bJqmfjI0?si=BYa65ISIw4MTT9gN
  - https://youtu.be/FFUYf8FHDlY?si=5ZTY4lSqKl-1gVsq
  - https://habr.com/ru/companies/haulmont/articles/758780/
  - https://habr.com/ru/companies/flant/articles/347518/
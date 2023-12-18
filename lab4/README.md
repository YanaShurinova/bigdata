# Лабораторная 4

- В Windows запустите сервер двойным кликом по скрипту zkServer.cmd в папке ./bin/ или из терминала, набрав zkServer.cmd

{zkserver.jpg}

## Взаимодействие с ZooKeeper через командный интерфейс CLI

- Устанавливаем подключение к ZooKeeper CLI сессии:

{zkcli.jpg}

- Для вывода всех возможных команд наберите help:

{help.jpg}

- Находясь в консоли CLI введите команду ls /

{ls.jpg}

- создайте свой узел /mynode с данными "first_version" следующей командой: create /mynode 'first_version'

{first_node.jpg}

- Проверьте, что в корне появился новый узел.

{ls2.jpg}

- Следующие команды возвращают данные и метаданные узла: get /mynode stat /mynode

{get_stat.jpg}

- Измените данные узла на "second_version" (dataVersion = 2, тк дважды запускала команду):

{second_version.jpg}

- Теперь создайте два нумерованных (sequential) узла в качестве дочерних mynode

{seq.jpg}

- Внутри CLI сессии, создайте узел mygroup

{group.jpg}

- Откройте две новых CLI консоли и в каждой создайте по дочернему узлу в mygroup 

{two.jpg}

- Проверьте в исходной консоли, что grue и bleen являются членами группы mygroup

{mygroup}

-  Выберите консоль grue и обратитесь к информации узла bleen.

{glue.jpg}

- Нажмите сочетание клавиш Ctrl+D в одной из консолей, создавшей эфимерный узел. Проверьте, что соответствующий узел пропал из mygroup.

{check.jpg}

- В заключении удалите узел /mygroup.

{delete.jpg}


## Пример управления конфигурацией распределённого приложения
- Создадим в корне узел "myconfig" в задачу которого будет входить хранение конфигурации. В нашем случае узел будет хранить строку 'sheep_count=1'.Откройте новую консоль и подключитесь к ZooKeeper. Данная консоль будет играть роль физического сервера, который ожидает получить оповещение в случае изменения конфигурационной информации, записанной в /myconfig znode. Следующая команда устанавливает watch-триггер на узел: get /myconfig -w true

{myconfig.jpg}

- Вернитесь к первому терминалу и измените значение myconfig. Во втором терминале должно появиться оповещение об изменении данных!

{change.jpg}

- Удалите узел /myconfig

{delete_config.jpg}


## Запуск для zoo

{tiger.jpg}

{monkey.jpg}

## Запуск для philosophers

Philosopher 3 is going to eat
Philosopher 2 is going to eat
Philosopher 1 is going to eat
Philosopher 4 is going to eat
Philosopher 0 is going to eat
Philosopher 4 picked up the left fork
Philosopher 1 picked up the left fork
Philosopher 1 picked up the right fork
Philosopher 4 picked up the right fork
Philosopher 3 picked up the left fork
Philosopher 1 put the right fork
Philosopher 1 put the loft fork and finished eating
Philosopher 2 picked up the left fork
Philosopher 1 is thinking
Philosopher 4 put the right fork
Philosopher 5 picked up the left fork
Philosopher 5 picked up the right fork
Philosopher 4 put the loft fork and finished eating
Philosopher 4 is thinking
Philosopher 3 picked up the right fork
Philosopher 0 is going to eat
Philosopher 3 is going to eat
Philosopher 3 put the right fork
Philosopher 3 put the loft fork and finished eating
Philosopher 2 picked up the right fork
Philosopher 3 is thinking
Philosopher 5 put the right fork
Philosopher 1 picked up the left fork
Philosopher 4 picked up the right fork
Philosopher 5 put the loft fork and finished eating
Philosopher 5 is thinking
Philosopher 2 put the right fork
Philosopher 2 put the loft fork and finished eating
Philosopher 1 picked up the right fork
Philosopher 4 is going to eat
Philosopher 1 put the right fork
Philosopher 1 put the loft fork and finished eating
Philosopher 1 is thinkingPhilosopher 1 is going to eat
Philosopher 2 picked up the left fork
Philosopher 2 picked up the right fork
Philosopher 2 is going to eat
Philosopher 4 put the right fork
Philosopher 4 put the loft fork and finished eating
Philosopher 5 picked up the left fork
Philosopher 4 is thinkingPhilosopher 5 picked up the right fork
Philosopher 2 put the right fork
Philosopher 2 put the loft fork and finished eating
Philosopher 2 is thinkingPhilosopher 3 picked up the left fork
Philosopher 3 picked up the right fork
Philosopher 3 put the right fork
Philosopher 3 put the loft fork and finished eating
Philosopher 3 is thinking
Philosopher 5 put the right fork
Philosopher 5 put the loft fork and finished eating
Philosopher 5 is thinking

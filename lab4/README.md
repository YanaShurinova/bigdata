# Лабораторная 4

- В Windows запустите сервер двойным кликом по скрипту zkServer.cmd в папке ./bin/ или из терминала, набрав zkServer.cmd

![zkserver.jpg](https://github.com/YanaShurinova/bigdata/blob/main/lab4/scheenshots/zkserver.jpg)

## Взаимодействие с ZooKeeper через командный интерфейс CLI

- Устанавливаем подключение к ZooKeeper CLI сессии:

![zkcli.jpg](https://github.com/YanaShurinova/bigdata/blob/main/lab4/scheenshots/zkcli.jpg)

- Для вывода всех возможных команд наберите help:

![help.jpg](https://github.com/YanaShurinova/bigdata/blob/main/lab4/scheenshots/help.jpg)

- Находясь в консоли CLI введите команду ls /

![ls.jpg](https://github.com/YanaShurinova/bigdata/blob/main/lab4/scheenshots/ls.jpg)

- создайте свой узел /mynode с данными "first_version" следующей командой: create /mynode 'first_version'

![first_node.jpg](https://github.com/YanaShurinova/bigdata/blob/main/lab4/scheenshots/first_node.jpg)

- Проверьте, что в корне появился новый узел.

![ls2.jpg](https://github.com/YanaShurinova/bigdata/blob/main/lab4/scheenshots/ls2.jpg)

- Следующие команды возвращают данные и метаданные узла: get /mynode stat /mynode

![get_stat.jpg](https://github.com/YanaShurinova/bigdata/blob/main/lab4/scheenshots/get_stat.jpg)

- Измените данные узла на "second_version" (dataVersion = 2, тк дважды запускала команду):

![second_version.jpg](https://github.com/YanaShurinova/bigdata/blob/main/lab4/scheenshots/second_version.jpg)

- Теперь создайте два нумерованных (sequential) узла в качестве дочерних mynode

![seq.jpg](https://github.com/YanaShurinova/bigdata/blob/main/lab4/scheenshots/seq.jpg)

- Внутри CLI сессии, создайте узел mygroup

![group.jpg](https://github.com/YanaShurinova/bigdata/blob/main/lab4/scheenshots/group.jpg)

- Откройте две новых CLI консоли и в каждой создайте по дочернему узлу в mygroup 

![two.jpg](https://github.com/YanaShurinova/bigdata/blob/main/lab4/scheenshots/two.jpg)

- Проверьте в исходной консоли, что grue и bleen являются членами группы mygroup

![mygroup.jpg](https://github.com/YanaShurinova/bigdata/blob/main/lab4/scheenshots/mygroup.jpg)
-  Выберите консоль grue и обратитесь к информации узла bleen.

![glue.jpg](https://github.com/YanaShurinova/bigdata/blob/main/lab4/scheenshots/glue.jpg)

- Нажмите сочетание клавиш Ctrl+D в одной из консолей, создавшей эфимерный узел. Проверьте, что соответствующий узел пропал из mygroup.

![check.jpg](https://github.com/YanaShurinova/bigdata/blob/main/lab4/scheenshots/check.jpg)

- В заключении удалите узел /mygroup.

![delete.jpg](https://github.com/YanaShurinova/bigdata/blob/main/lab4/scheenshots/delete.jpg)


## Пример управления конфигурацией распределённого приложения
- Создадим в корне узел "myconfig" в задачу которого будет входить хранение конфигурации. В нашем случае узел будет хранить строку 'sheep_count=1'.Откройте новую консоль и подключитесь к ZooKeeper. Данная консоль будет играть роль физического сервера, который ожидает получить оповещение в случае изменения конфигурационной информации, записанной в /myconfig znode. Следующая команда устанавливает watch-триггер на узел: get /myconfig -w true

![myconfig.jpg](https://github.com/YanaShurinova/bigdata/blob/main/lab4/scheenshots/myconfig.jpg)

- Вернитесь к первому терминалу и измените значение myconfig. Во втором терминале должно появиться оповещение об изменении данных!

![change.jpg](https://github.com/YanaShurinova/bigdata/blob/main/lab4/scheenshots/change.jpg)

- Удалите узел /myconfig

![delete_config.jpg](https://github.com/YanaShurinova/bigdata/blob/main/lab4/scheenshots/delete_config.jpg)


## Запуск для zoo

![tiger.jpg](https://github.com/YanaShurinova/bigdata/blob/main/lab4/scheenshots/tiger.jpg)

![monkey.jpg](https://github.com/YanaShurinova/bigdata/blob/main/lab4/scheenshots/monkey.jpg)
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

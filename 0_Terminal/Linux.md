## Hot-Keys



## Dirs operations
mkdir ChatBot - создать дирректорию
rm -r dirname - рекурсивно удалить дирректорию

## Navigation
ctrl + r - открыть поиск команд
<стрелка вверх> - ранее введенная команда
alt + / - ранее веденная команда



pwd - вывести путь к текущей директории

cd dir1/dir2/ - перейти в дирректорию
cd .. - перейти в родительскую директорию
cd - - перейти в предыдущую директорию

ctrl + u - очистить троку ввода
clear - очистить терминал
ctrl + l - очистить терминал
 


## Files

cat /etc/passwd | column -t - вывод данных в виде таблицы



## Resources

nvidia-smi - посмотреть какие процессы висят на видеокарте
ps aux | grep 6217

вообще все процессы можно смотреть через команду ps


scp <путь к файлу без ковычек> safonkin@koala02:<путь к пункту назначения файла без ковычек> - перенос файла на сервер
ssh safonkin@koala02.ftc.ru - зайти на сервер

exit - выйти с сервера


dvc pull - подтянуть данные

## Lifehacks
https://habr.com/ru/company/ruvds/blog/336060/

while ! [command]; do sleep 1; done - выполнять команду до победного


## Pip

pip list | grep <имя пакета> - проверить версию пакета

## Jupyter Notebook

jupyter notebook --notebook-dir="c: - запуск ноутбука на диске c

jupyter notebook --generate-config - сгенерировать конфиг
jupyter notebook list - вывести список запущенных ноутбуков
pkill -u `id -u` jupyter - остановить все мои ноутбуки

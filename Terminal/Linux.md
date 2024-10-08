## Hot-Keys


## Dirs operations

```console
mkdir ChatBot - создать дирректорию
rm -r dirname - рекурсивно удалить дирректорию
```

## Navigation

```console
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
```


## Files

```console
cat /etc/passwd | column -t - вывод данных в виде таблицы

find . -name "*.csv" -type f -delete
```

## Resources

```console
nvidia-smi - посмотреть какие процессы висят на видеокарте
ps aux | grep 6217

вообще все процессы можно смотреть через команду ps

du -hs * | sort -hr - вывод файлов вместе с количеством занимаемого пространства
```

## Server

```console
scp <путь к файлу без ковычек> safonkin@<хост сервера>:<путь к пункту назначения файла без ковычек> - перенос файла на сервер
ssh safonkin@<хост сервера> - зайти на сервер

exit - выйти с сервера

hostnamectl - вывести имя хоста, а также данные используемой операционной системы
sudo su - получить root-права для своего аккаунта (exit - вернуть права)
```

## Lifehacks
https://habr.com/ru/company/ruvds/blog/336060/

```console
while ! [command]; do sleep 1; done - выполнять команду до победного
```

## Pip

```console
pip install --upgrade pip - обновить pip

pip list | grep <имя пакета> - проверить версию пакета

pip install <имя пакета> --upgrade - обновить пакет
pip --disable-pip-version-check list --outdated --format=json | python -c "import json, sys; print('\n'.join([x['name'] for x in json.load(sys.stdin)]))" | xargs -n1 pip install -U - обновить все пакетыс п
pip uninstall <имя пакета> - удалить пакет

```

## Jupyter Notebook

```console
jupyter notebook --notebook-dir="c: - запуск ноутбука на диске c

jupyter notebook --generate-config - сгенерировать конфиг

jupyter notebook list - вывести список запущенных ноутбуков
pkill -u `id -u` jupyter - остановить все мои ноутбуки
jupyter notebook stop <номер порта, например 8008> - остановить конкретный сервер
```

## dvc

```console
dvc pull - подтянуть данные
```

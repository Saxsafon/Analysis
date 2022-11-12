## Базовые команды

```console
cd - переместиться в директорию
dir - посмотреть содержание директории
```

### Windows

```console
cls - очистить терминал
```
### Linux

```console
clear - очистить терминал
```

## Работа с виртуальными окружениями

```console
conda create --name venv python=3.7 - создать окружение с конкретной версией python
conda create --prefix=$PWD/venv python=3.7 - создать окружение в текущей директории и с конкретной версией pytho

conda activate venv - активировать окружение
conda deactivate - деактивировать окружение

conda env list - вывести список доступных окружений

conda env remove --name corrupted_venv - удалить окружение
```


## Работа с пакетами

```console
conda install package - установить пакет
```
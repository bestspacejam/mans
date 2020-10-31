# Консоль

### Полезные команды

```shell
# 1. redo last command but as root
sudo !!

# 2. open an editor to run a command
ctrl+x+e

# 3. create a super fast ram disk
mkdir -p /mnt/ram
mount -t tmpfs tmpfs /mnt/ram -o size=8192M

# 4. don't add command to history (note the leading space)
 ls -l

# 5. fix a really long command that you messed up
fc

# 6. tunnel with ssh (local port 3337 -> remote host's 127.0.0.1 on port 6379)
ssh -L 3337:127.0.0.1:6379 root@emkc.org -N

# 7. quickly create folders
mkdir -p folder/{sub1,sub2}/{sub1,sub2,sub3}

# 8. intercept stdout and log to file
cat file | tee -a log | cat > /dev/null

# 9. exit terminal but leave all processes running
disown -a && exit

# 10. Отсортировать лог-файл по содержимому строки
paste -d' ' <(grep -o -E 'Time:[^,]+' performance.log | cut -d' ' -f2-) performance.log | sort -h

# 11. Вывести содержимое файла без закомментированных строк
grep -v "^$\|^#" /etc/dnsmasq.conf

# 12. Вывести список всех используемых ДНС серверов
nmcli dev show | awk '$1 ~ /DNS/ {print $2}'

# 13. Создать и перейти во временную директорию
cd "$(mktemp -d)"

# 14. Проверить, что `stdin` команды открыт не в терминале
{ [ ! -t 0 ] && cat; } <<< redirection

# 15. Вывести строку с поддержкой специальных символов
echo -n $'one\ntwo\nthree'
echo -n -e "one\ntwo\nthree"
printf '%b' "one\ntwo\nthree"

# 16. Вызвать команду для каждого переданного аргумента
printf '%s\0' "$@" | xargs -0n1 printf '{%s}\n'

# 17. Очистка содержимого от непечатаемых символов
tr -cd "[:print:]\n" < file1

# 18. Получить имя исполняемого файла без вызова basename
echo "${0##*/}"
```

#### Ссылки
* [8 super heroic Linux commands that you probably aren't using](https://www.youtube.com/watch?v=Zuwa8zlfXSY)
* [Removing all special characters from a string in Bash](https://stackoverflow.com/questions/36926999/removing-all-special-characters-from-a-string-in-bash)

### Навигация в консоли

```shell
# Перейти в домашнюю директорию
cd
cd ~

# Вернуться в предыдущую директорию
cd -
```

### Переменные окружения

```
$HOME - домашняя директория 
$PWD - путь до текущей директории 
$OLDPWD - путь до предыдущей директории 
```


### Использование DEBUG trap для отладки скрипта

```shell
trap '(read -p "[$BASH_SOURCE:$LINENO] $BASH_COMMAND?")' DEBUG
var=2
echo $((var+2))
```

### Склеивание массива в строку и разделение на слова


```shell
IFS="| "
list=(one "two three" four)

# Все элементы массива склеиваются в одно слово с первым символом IFS в качестве разделителя:
# <one|two three|four>
printf " <%s>" "${list[*]}"; echo;

# Каждый элемент массива становится словом, IFS не используется:
# <one> <two three> <four>
printf " <%s>" "${list[@]}"; echo;

# Каждый элемент массива раскрывается как отдельное слово которое 
# в дальнейшем является источником для Word Splitting, Filename Expansion и Quote Removal:
# <one> <two> <three> <four>
printf " <%s>" ${list[*]}; echo;
```

#### Ссылки
- https://www.gnu.org/software/bash/manual/html_node/Word-Splitting.html
- https://bash.cyberciti.biz/guide/$IFS


### Расширение параметров

```shell
# Имя текущей директории
echo "${PWD##*/}"

# Имя выполняемого скрипта
echo "${0##*/}"

# Путь до директории выполняемого скрипта
echo "${0%/*}"
echo "$(dirname $0)"

# Изменить регистр значения на нижний:
example=UPPERCASED
echo "${example,,}"

# Получить значение переменной "bob":
bob=user
name=bob
echo ${!name}
```

#### Ссылки
* [Shell Parameter Expansion](https://www.gnu.org/software/bash/manual/html_node/Shell-Parameter-Expansion.html)


### xargs

Разбивает стандартный ввод на аргументы по символу пробела или новой строки и передаёт их в указанную функцию.

Пробелы в аргументах могут быть экранированы двойными/одинарными кавычками или обратной косой чертой.

#### Параметры `-n` и `-L`

Параметр `-n` ограничивает количество одновременно передаваемых аргументов указанным числом.

Параметр `-L` - сложный, и вызывает программу передавая ей столько аргументов, сколько нашёл их в указанном количестве строк. Строка оканчивающаяся пробелом объединяется во одну строку со следующей.

Например, для `-L2` и двух строк по пять аргументов - программа вызовется с 10-ю аргументами.

При указанном параметре `-d` или `-0`, параметры `-L` и `-n` действуют одинаково.

```shell
# Отличие `xargs -L` от `xargs -n`
(echo {1..5}; echo {a..e}) | xargs -n1 sh -c 'echo -n "$0 "; printf "{%s} " "$@"; echo' '->'
(echo {1..5}; echo {a..e}) | xargs -L1 sh -c 'echo -n "$0 "; printf "{%s} " "$@"; echo' '->'
```

Для разделения на вызовы программы, по одному вызову на строку ввода, параметр `-L` лучше не использовать.

Обсуждение проблемы: [Make xargs execute the command once for each line of input](https://stackoverflow.com/a/28806991/9215292)

#### Примеры

```shell
# Вызов команды для каждой строки из файла в качестве параметра
xargs -a params.txt -n1 -d$'\n' printf '{%s}\n'

# Имена файлов могут содержать символы переноса строк, поэтому
# для команды find лучше использовать параметр -print0
find / -maxdepth 1 -print0 | xargs -0 -n1 printf '{%s}\n'
```
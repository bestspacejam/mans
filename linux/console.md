  
### Включение поддержки комбинации "Ctrl-S" в терминале
```shell
stty -ixon
```


### Подсветка имени хоста в зависимости от его типа

```shell
# На тестовой виртуалке ставим зелёное имя хоста в .bashrc:
export PS1='\[\033[01;32m\]\u@\[\e[0;32m\]\h\e[m\]: \[\033[01;34m\]\w\[\033[00m\]$ '

# На бою красным и добавляем PRODUCTION:
export PS1='\[\033[01;32m\]\u@\[\e[0;31m\]\h PRODUCTION\e[m\]: \[\033[01;34m\]\w\[\033[00m\]$ '
```

### Проверка поддержки 256 цветной палитры в терминале

```shell
# Программа `colortest` добавляет комманды:
# colortest-16, colortest-16b, colortest-256 и colortest-8
sudo apt-get install colortest

# Проверка поддержки 24 битного цвета в терминале
awk 'BEGIN{
        s="/\\/\\/\\/\\/\\"; s=s s s s s s s s;
        for (colnum = 0; colnum<77; colnum++) {
            r = 255-(colnum*255/76);
            g = (colnum*510/76);
            b = (colnum*255/76);
            if (g>255) g = 510-g;
            printf "\033[48;2;%d;%d;%dm", r,g,b;
            printf "\033[38;2;%d;%d;%dm", 255-r,255-g,255-b;
            printf "%s\033[0m", substr(s,colnum+1,1);
        }
        printf "\n";
    }'
```
Источник: [Script to display all terminal colors](https://askubuntu.com/questions/27314/script-to-display-all-terminal-colors)

# Организация доступа по SSH с использованием аутентификации с помощью закрытого ключа


## На локальном хосте 

1. **Сгенерировать пару открытый/закрытый ключ для OpenSSH**

	*Cоздаём файлы sshkey.pem и sshkey.pem.pub. В файле sshkey.pem.pub содержится код для ~/.ssh/authorized_keys*

	```shell
	ssh-keygen -t rsa -b 2048 -f sshkey.pem -N ''
	```

2. **Ограничить доступ к файлу закрытого ключа**

	*OpenSSH отказывается использовать файл закрытого ключа если у него установлены слишком широкие права доступа.*

	```shell
	chmod 700 sshkey.pem
	```


### Как сбросить сохранённый ключ удалённого хоста

```
ssh-keygen -R <hostname>
```


### Как получить ключ удалённого хоста


[Вариант 1](https://superuser.com/a/1111974):

```shell
$ ssh-keyscan -t rsa -H bitbucket.org >> ~/.ssh/known_hosts
```


[Вариант 2](https://stackoverflow.com/a/25020638):

```shell
if [ -z `ssh-keygen -F $IP` ]; then
  ssh-keyscan -H $IP >> ~/.ssh/known_hosts
fi
```


### Как подключится к серверу без использования приватного ключа

```
ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no example.com
```

### Быстрое копирование публичного ключа на удалённый сервер

```
ssh-copy-id <username>@<host>
```



## На удалённом хосте

1. **Загрузить "sshkey.pem.pub"**
2. **Создать файл "authorized_keys", если ещё не создан.**
	
	```shell
	mkdir ~/.ssh
	touch ~/.ssh/authorized_keys
	
	chmod 700 ~/.ssh
	chmod 600 ~/.ssh/authorized_keys
	```
	
	Владельцем файла `authorized_keys` должен быть пользователь которому необходимо предоставить доступ по SSH.
	
3. **Добавить публичный ключ в `authorized_keys`**

	```shell
	echo `cat ~/.ssh/sshkey.pem.pub` >> ~/.ssh/authorized_keys
	```

### Настройки SSH сервера

Файл `/etc/ssh/sshd_config`:
```
# Для доступа root-a только по публичному ключу
PermitRootLogin prohibit-password

# Включение аутентификации по публичному ключу
PubkeyAuthentication yes
```

В случае проблем с аутентификацией смотреть `tail -f /var/log/auth.log`

* [Список директив файла sshd_config](https://linux.die.net/man/5/sshd_config)


### Как обновить ключи сервера

Бывает полезно при клонировании виртуальной машины


1. Удалить файлы ключей

```
# /bin/rm -v /etc/ssh/ssh_host_*
```
2. Перекофигурировать SSH сервер

```
# dpkg-reconfigure openssh-server
```
3. Перезапустить SSH сервер
```
$ /etc/init.d/ssh restart
```

*На клиенте необходимо сбросить загруженный ключ для этого хоста*


* [Ubuntu / Debian Linux Regenerate OpenSSH Host Keys](https://www.cyberciti.biz/faq/howto-regenerate-openssh-host-keys/)


## Ссылки

* [Use Public Key Authentication with SSH](https://www.linode.com/docs/security/use-public-key-authentication-with-ssh) &mdash; вроде неплохой мануал
* [Linux server security best practices](https://support.rackspace.com/how-to/linux-server-security-best-practices/)
* [Как включить DSS-ключи для старых клиентов](https://www.gentoo.org/support/news-items/2015-08-13-openssh-weak-keys.html)

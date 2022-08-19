# Deployment

For simplicity, we will deploy the project to VDS. The deployment process consists of several steps:

1. Create a VDS on your preferred hosting
2. Manually prepare VDS
3. Set secrets in the repository on GitHub
4. Optional: Set up HTTPS

## 1. Create a VDS

We create a VDS virtual machine, for example, in [NetAngels](https://panel.netangels.ru).

## 2. Prepare VDS

Login via SSH and check PHP version (`php -v). 

If necessary, [upgrade to PHP 8](https://php.watch/articles/php-8.0-installation-update-guide-debian-ubuntu):

```bash
sudo apt update
sudo apt install software-properties-common
sudo apt update
sudo add-apt-repository ppa:ondrej/php
sudo apt update
sudo apt install php8.0-common php8.0-cli php8.0-mysql php8.0-mbstring -y
```

If necessary, install PHP extensions on the virtual machine:

```bash
sudo apt install php8.0-xml
sudo apt install php8.0-curl
```

Manually [install](https://getcomposer.org/download/) Composer:

```bash
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php -r "if (hash_file('sha384', 'composer-setup.php') === '906a84df04cea2aa72f40b5f787e49f22d4c2f19492ac310e8cba5b96ac8b64115ac402c8cd292b8a03482574915d1a8') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
php composer-setup.php
php -r "unlink('composer-setup.php');"
````

And make it available for calling through `composer`:

```bash
sudo mv composer.phar /usr/local/bin/composer
```

>Next, we do everything from the user `web`, not `root`!!!




/var/www/web/sites

```bash
git clone git@github.com:gomzyakov/secretnotes.git secretnotes.ru
``````

/var/www/web/sites/secretnotes.ru

php -r "file_exists('.env') || copy('.env.example', '.env');"

composer install

chmod -R 777 storage bootstrap/cache

php artisan key:generate

Вносим корректные реквизиты в `.env` файл:

Создаём БД `secretnotes` через phpMyAdmin

php artisan migrate:fresh --seed

## Front

На виртуалке:

```
cd ~
curl -sL https://deb.nodesource.com/setup_16.x -o /tmp/nodesource_setup.sh
sudo bash /tmp/nodesource_setup.sh
sudo apt install nodejs
```

При вводе `node -v` видим:
```
v16.6.1
```

```
$ sudo apt install npm
```

После этого билдим непосредственно фронтовые зависимости:

```
npm install
npm run production
```


## 3. Set secrets in the repository on GitHub

Generate SSH-keys with `ssh-keygen`

В репозитории на GitHub, в разделе `Settings > Secrets > Actions` задаём значения:

- `SSH_PRIVATE_KEY`. If you don’t have this you can run ssh-keygen in your server terminal to generate a new key, then
  run the command cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys to allow connection to the private key.
- `SSH_HOST`. This is the IP address of the server.
- `SSH_USERNAME`. This is the server username, in my case it’s root . If you don’t know your username you can use the
  whoami command on your server to check.

В репозитории, в разделе `Deploy keys`, задаём значение публичного ключа с виртуальной машины (получить можно
через `cat ~/.ssh/id_rsa.pub`). Это позволит деплоиться через `git pull` с VDS.

## 4. Optional: Set up HTTPS

О подключении HTTPS подробнее написано в [HTTPS.md](HTTPS.md)
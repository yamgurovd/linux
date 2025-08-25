# 13 Перенос SSH ключей.md

---

### Проверяем, что ключи у вас есть в стандартном месте:
```shell
ls -l ~/.ssh/id_rsa ~/.ssh/id_rsa.pub
```

### Копируем ключи в папку home/d/Downloads (замените d на имя пользователя):
```shell
cp ~/.ssh/id_rsa ~/.ssh/id_rsa.pub /home/d/Downloads/
```

### Переходим в папку Downloads:
```shell
cd /home/d/Downloads/
```
### Перенести ключи на необходимый компьютер 

### Создать папку .ssh в домашней директории, если её ещё нет:
```shell
mkdir -p ~/.ssh
```

### Скопировать ключи из /home/d/Downloads в папку .ssh:
```shell
cp /home/d/Downloads/id_rsa ~/.ssh/
cp /home/d/Downloads/id_rsa.pub ~/.ssh/
```
### Настроить правильные права доступа:
```shell
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub
chmod 700 ~/.ssh
```
# Прошивка NanoKVM Lite на Debian GNU/Linux

## 1. Установка необходимого

```bash
sudo apt update
sudo apt install -y wget xz-utils
```

## 2. Скачивание образа

```bash
mkdir -p ~/Downloads/nanokvm && cd ~/Downloads/nanokvm

# Актуальный образ (проверьте последнюю версию на GitHub)
wget https://github.com/sipeed/NanoKVM/releases/download/v1.4.0/20250217_NanoKVM_Rev1_4_0.img.xz
```

Ссылки на образы:
- [v1.4.3](https://github.com/sipeed/NanoKVM/releases/tag/v1.4.3)
- [v1.4.0](https://github.com/sipeed/NanoKVM/releases/tag/v1.4.0)

Ищите файл вида `YYYYMMDD_NanoKVM_Rev1_4_x.img.xz`.

## 3. Определение устройства карты
Вставьте MicroSD в кардридер и выполните:
```bash
lsblk -o NAME,SIZE,TYPE,MOUNTPOINT,MODEL
```
Найдите вашу карту (обычно `/dev/sdb`, `/dev/sdc` и т.д.).  
**Убедитесь, что это не системный диск (`/dev/sda`).**
Пример вывода:
```
NAME   SIZE TYPE MOUNTPOINT MODEL
sda  476.9G disk            Samsung SSD
└─sda1 ...
sdb   29.8G disk            USB Card Reader   ← это карта
```

## 4. Размонтирование разделов карты (если смонтированы)
```bash
sudo umount /dev/sdX* 2>/dev/null || true
```
Замените `sdX` на ваше устройство (например `sdb`).

## 5. Запись образа
### Вариант с распаковкой
```bash
# Распаковываем
unxz -k 20250217_NanoKVM_Rev1_4_0.img.xz

# Записываем
sudo dd if=20250217_NanoKVM_Rev1_4_0.img of=/dev/sdX bs=4M status=progress conv=fsync
```
### Вариант без распаковки (экономит место)
```bash
xzcat 20250217_NanoKVM_Rev1_4_0.img.xz | sudo dd of=/dev/sdX bs=4M status=progress conv=fsync
```
После завершения:

```bash
sync
sudo eject /dev/sdX
```
## 6. Установка карты

- Слот MicroSD находится **снизу** платы NanoKVM Lite.
- Вставьте карту.
- Подключите Ethernet, HDMI и USB-C (PC/HID) к управляемому компьютеру.
Загрузка занимает 1–2 минуты.

## 7. Первый вход
1. Найдите IP:
   ```bash
   # на той же сети
   ip neigh | grep -i "sipeed\|nanokvm" || nmap -sn 192.168.1.0/24
   ```
   или посмотрите в роутере.

2. Откройте в браузере `http://IP`

3. Логин / пароль:
   ```
   admin / admin
   ```
4. Сразу обновите приложение:  
   **Settings → Check for Updates**

---

### Важные замечания

- Всегда проверяйте устройство командой `lsblk` перед `dd`.
- Ошибка в выборе диска приведёт к потере данных.
- После записи Windows/macOS могут предложить отформатировать карту — **не делайте этого**.
```
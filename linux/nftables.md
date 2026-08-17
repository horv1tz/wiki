---
description: Настройка брандмауэра nftables — современной замены iptables
---

# nftables

**nftables** — современная подсистема фильтрации пакетов в Linux, официальная замена [iptables](iptables.md). Единый синтаксис для IPv4/IPv6, атомарное применение правил, нет дублирования таблиц (`filter`, `nat`, `mangle` объединены в свободную структуру).

## 1. Установка и базовые команды

```bash
sudo apt install nftables          # Debian/Ubuntu
sudo systemctl enable --now nftables
```

```bash
sudo nft list ruleset              # показать все правила
sudo nft list tables               # список таблиц
sudo nft flush ruleset             # удалить все правила (осторожно!)
```

## 2. Основные понятия

* **Таблица (table)** — пространство имён с семейством: `ip`, `ip6`, `inet` (оба сразу), `arp`, `bridge`
* **Цепочка (chain)** — набор правил; базовые цепочки привязаны к хукам (`input`, `output`, `forward`) и имеют политику
* **Правило (rule)** — условие + действие (`accept`, `drop`, `reject`, `jump`, `masquerade`...)

В отличие от iptables, структуру таблиц и цепочек вы создаёте сами.

## 3. Базовый брандмауэр

Создание таблицы и цепочек интерактивно:

```bash
sudo nft add table inet filter
sudo nft add chain inet filter input   { type filter hook input priority 0 \; policy drop \; }
sudo nft add chain inet filter forward { type filter hook forward priority 0 \; policy drop \; }
sudo nft add chain inet filter output  { type filter hook output priority 0 \; policy accept \; }

# Локальный интерфейс и уже установленные соединения
sudo nft add rule inet filter input iif lo accept
sudo nft add rule inet filter input ct state established,related accept

# SSH и веб
sudo nft add rule inet filter input tcp dport 22 accept
sudo nft add rule inet filter input tcp dport '{ 80, 443 }' accept
sudo nft add rule inet filter input ip protocol icmp accept
```

> Экранирование `\;` нужно в shell. В файле конфигурации точка с запятой пишется без экранирования.

## 4. Конфигурация через файл

Правила из командной строки теряются после перезагрузки — постоянная конфигурация живёт в `/etc/nftables.conf`:

```
#!/usr/sbin/nft -f

flush ruleset

table inet filter {
    chain input {
        type filter hook input priority 0; policy drop;

        iif lo accept
        ct state established,related accept
        ct state invalid drop

        tcp dport 22 accept
        tcp dport { 80, 443 } accept
        ip protocol icmp accept
        ip6 nexthdr ipv6-icmp accept
    }

    chain forward {
        type filter hook forward priority 0; policy drop;
    }

    chain output {
        type filter hook output priority 0; policy accept;
    }
}

table inet nat {
    chain postrouting {
        type nat hook postrouting priority 100;
        oifname "eth0" masquerade
    }

    chain prerouting {
        type nat hook prerouting priority -100;
        # Проброс порта 8080 → 192.168.1.100:80
        tcp dport 8080 dnat to 192.168.1.100:80
    }
}
```

Проверка и применение:

```bash
sudo nft -c -f /etc/nftables.conf   # проверка синтаксиса без применения
sudo nft -f /etc/nftables.conf      # применить
sudo systemctl enable --now nftables
```

## 5. Полезные приёмы

```bash
# Наборы (sets) — списки IP/портов, изменяемые без перезагрузки правил
sudo nft add set inet filter blocklist { type ipv4_addr \; }
sudo nft add rule inet filter input ip saddr @blocklist drop
sudo nft add element inet filter blocklist { 203.0.113.50 }

# Логирование
sudo nft add rule inet filter input tcp dport 22 log prefix \"SSH attempt: \" accept

# Ограничение новых подключений к SSH (анти-брутфорс)
sudo nft add rule inet filter input tcp dport 22 ct state new limit rate 10/minute accept

# Счётчики на правилах
sudo nft add rule inet filter input tcp dport 80 counter accept
sudo nft list chain inet filter input   # покажет счётчики пакетов/байт
```

## 6. Миграция с iptables

```bash
# Конвертация существующих правил iptables в синтаксис nftables
sudo iptables-save > /tmp/iptables.rules
sudo iptables-restore-translate -f /tmp/iptables.rules > /tmp/nft.rules
sudo nft -f /tmp/nft.rules
```

## 7. Полезные ссылки

* [Вики nftables](https://wiki.nftables.org/)
* [Quick reference](https://wiki.nftables.org/wiki-nftables/index.php/Quick_reference-nftables_in_10_minutes)

1. Выполните базовую настройку всех устройств:
![image](https://github.com/DimaDanil/demo/assets/170974285/64945a6a-25db-43b7-84d5-d18e74800d58)

![image](https://github.com/DimaDanil/demo/assets/170974285/57b82acd-c3be-44a9-95ed-798048279242)

[root@localhost ~]# hostnamectl set-hostname <NAME>

[root@localhost ~]# exec bash

Для устройств BR-SRV и CLI желательно сразу установить полное доменное имя. Потребуется для ввода этих машин в домен во второй части задания.

Например:
ISP: isp

CLI: cli.hq.work

HQ-R: hq-r.hq.work

HQ-SRV: hq-srv.hq.work

BR-R: br-r.branch.work

BR-SRV: br-srv.branch.work
_____________
На интерфейсах, которые смотрят в сторону интернета ОБЯЗАТЕЛЬНО нужно настроить "Серверы DNS", для доступа к сервисам по доменным именам, а не ip-адресам

        nmtui
![image](https://github.com/DimaDanil/demo/assets/170974285/e44b609f-3dfd-4f41-9dbf-85964414762a)
systemctl restart networking (NetworkManager)

![image](https://github.com/DimaDanil/demo/assets/170974285/73b5acb9-ef1e-49c1-b1a9-c71c8e427060)

![image](https://github.com/DimaDanil/demo/assets/170974285/df5def94-c74b-4e84-87f2-8ac7fc3ac4d2)

___________________________________________________________________________________________
Маршрутизация транзитных IP-пакетов
nano /etc/sysctl.conf
В данном файле прописываем следующие строки:

   net.ipv4.ip_forward=1

   net.ipv6.conf.all.forwarding=1

    sysctl -p

Настройка nftables на ISP
Перед установкой необходимо убедиться что имеется доступ в интернет с ВМ ISP

    ping -c4 ya.ru

    dnf install -y nftables

После установки:

    nano /etc/nftables/isp.nft

Прописываем следующие строки:

    table inet my_nat {
        chain my_masquerade {
        type nat hook postrouting priority srcnat;
        oifname "ens18" masquerade
        }
    }

где ens18 - публичный интерфейс ISP (смотрящий в Интернет)
После:

    nano /etc/sysconfig/nftables.conf

Ниже строки начинающейся на include, прописываем строку

    include "/etc/nftables/isp.nft"

    systemctl enable --now nftables
_______________________________________________________________________________________________
2. Настроика внутренней динамической маршрутизации по средствам FRR.

Настройка HQ-R

![image](https://github.com/DimaDanil/demo/assets/170974285/b7b3f101-4faa-475a-91bd-0289802b275e)

![image](https://github.com/DimaDanil/demo/assets/170974285/6055d22d-0b7a-4be2-87d5-cf1edb2d735e)

Настройка BR-R
 Меняем локал и удал IP местами и IP будеи из 2 табл

       nmcli connection modify tun1 ip-tunnel.ttl 64

Настройка динамической (внутренней) маршрутизации средствами FRR

Установка пакет frr
      dnf install -y frr

    nano /etc/frr/daemons

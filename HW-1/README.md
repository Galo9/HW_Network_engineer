## Лабораторная работа - Настройка маршрутизации между VLAN на маршрутизаторе - "Router on a Stick"

### 1. Построение сети и настройка основных параметров устройства
    
#### 1.1 Топология сети:

![alt-текст](https://github.com/Galo9/HW_Network_engineer/blob/main/HW-1/HW1_topology.PNG)
    
 #### 1.2 Настройка основных параметров маршрутизатора:

```
    Router>enable   // Переход в привелигированный режим EXEC
    Router#config t   // Переход в режим настройки
    Router(config)#hostname R1   // Назначаем имя маршрутизатору
    R1(config)#no ip domain lookup   // Отключаем поиск по DNS
    R1(config)#enable secret class   // Назначаем class в качестве пароля для привилегированного режима EXEC с шифрованием
    R1(config)#service password-encryption   // Шифруем пароли в конфигурации
    R1(config)#line console 0   // Назначаем пароль для консоли
    R1(config-line)#password cisco
    R1(config-line)#login  // Включаем вход в систему
    R1(config-line)#exit
    R1(config)#line vty 0 4   // Назначаем пароль для VTY
    R1(config-line)#password cisco
    R1(config-line)#login   // Включаем вход в систему
    R1(config-line)#exit
    R1(config)#no service password-encryption
    R1(config)#banner login $Unauthorized access is prohibited!$   // Создаем банер о том, что несанкционированный доступ запрещен
    R1(config)#clock set 22:45:00 16 may 2024   // Настраиваем дату и время
    R1(config)#exit
    R1>copy run sta   // Сохраняем текущую конфигурацию в файле конфигурации запуска
```
    
 #### 1.3 Настройка основных параметров коммутаторов
    
Основные параметры Коммутаторов настраиваются аналогично маршрутизатору R1, только им назначаются имена S1 и S2 соответственно.    

#### 1.4 Настройка узлов ПК

Для первого ПК:
```
    VPCS> set cpname PC-A   // Назначаем имя ПК
    PC-A> ip address 192.168.3.3/24 192.168.3.1   // Назначаем ПК IP-адрес, маску подсети и шлюз по умолчанию
    PC-A> save   // Сохраняем конфигурацию
```
Для второго ПК:
```
    VPCS> set cpname PC-B   // Назначаем имя ПК
    PC-B> ip address 192.168.4.3/24 192.168.4.1   // Назначаем ПК IP-адрес, маску подсети и шлюз по умолчанию
    PC-B> save   // Сохраняем конфигурацию
```


### 2. Создание VLAN и назначение портов коммутатора
    
#### 2.1 Создание VLAN на коммутаторах

```
    S1(config)#vlan 3   // Создаем VLAN
    S1(config-vlan)#name Management   // Назначаем имя VLAN

    S1(config)#vlan 4
    S1(config-vlan)#name Operations

    S1(config)#vlan 7
    S1(config-vlan)#name ParkingLot

    S1(config)#vlan 8
    S1(config-vlan)#name Native

    S1(config)#interface vlan 3   // Настраиваем интерфейс управления и шлюз по умолчанию для каждого коммутатора
    S1(config-if)#ip address 192.168.3.11 255.255.255.0   // Для коммутатора S2 IP-аадрес - 192.168.3.12
    S1(config-if)#no shutdown   // Включаем порт
    S1(config-if)#exit
    S1(config)#ip defailt-gateway 192.168.3.1   // Назначаем шлюз по умолчанию
```
    
Подключаем все неиспользуемые порты на коммутаторах к ParkingLot VLAN и отключаем их
    
на коммутаторе S1:
```
    S1(config)#interface e0/3 
    S1(config-if)#switchport mode access
    S1(config-if)#switchport access vlan 7
    S1(config-if)#shutdown   // Отключаем порт
```
на коммутаторе S2:
```
    S2(config)#interface range e0/2-3
    S2(config-if-range)#switchport mode access
    S2(config-if-range)#switchport access vlan 7
    S2(config-if-range)#shutdown   // Отключаем порты
```

#### 2.2 Назначение VLAN соответствующим интерфейсам коммутаторов
    
Назначаем используемые порты соответствующим VLAN

на коммутаторе S1:
```
    S1(config)#interface e0/2
    S1(config-if)#switchport mode access
    S1(config-if)#switchport access vlan 3 
    S1(config-if)#no shutdown   // Включаем порт

```
на коммутаторе S2:
```
    S2(config)#interface e0/0
    S2(config-if)#switchport mode access
    S2(config-if)#switchport access vlan 4 
    S2(config-if)#no shutdown   // Включаем порт
```
    
Проверяем, что порты назначены VLAN верно:

![alt-текст](https://github.com/Galo9/HW_Network_engineer/blob/main/HW-1/HW1_trunk.PNG)


### 3. Настройка trunk 802.1Q между коммутаторами

#### 3.1 Настройка режима trunk для интерфейса е0/0

```
    S1(config)#interface e0/0
    S1(config-if)#switchport trunk encapsulation dot1q   // Делаем процедуру инкапсуляции статической для настройки статического trunk 
    S1(config-if)#switchport mode trunk   // Переводим интерфейс в режим статического trunk
    S1(config-if)#switchport trunk native vlan 8   // Устанавливаем VLAN 8 в качестве native VLAN
    S1(config-if)#switchport trunk allowed vlan 3,4,8   // Создаем ограничение по VLAN, которые можно передавать через тот или иной trunk

```
На комматуторе S2 настройка режима trunk осуществляется аналогично.

#### 3.2 Настройка режима trunk для интерфейса е0/1 для коммутатора S1

```
    S1(config)#interface e0/1
    S1(config-if)#switchport trunk encapsulation dot1q   // Делаем процедуру инкапсуляции статической для настройки статического trunk 
    S1(config-if)#switchport mode trunk   // Переводим интерфейс в режим статического trunk
    S1(config-if)#switchport trunk native vlan 8   // Устанавливаем VLAN 8 в качестве native VLAN
    S1>copy run sta   // Сохраняем текущую конфигурацию в файле конфигурации запуска
```

Проверяем список портов в режиме trunk:

![alt-текст](https://github.com/Galo9/HW_Network_engineer/blob/main/HW-1/HW1_trunk.PNG)

    Вопрос: Почему e0/1 не отображается в списке trunk?
    
    Ответ: e0/1 сразу отобразился в списке trunk


### 4. Настройка маршрутизации между VLAN на маршрутизаторе

```
    R1(config)#interface e0/0
    R1(config-if)#no shutdown   // Включаем порт
    R1(config-if)#exit
    R1(config)#interface e0/0.3   // Создаем логический подинтерфейс для vlan 3
    R1(config-subif)#encapsulation dot1q 3   //  Указываем, что на интерфейс будет приходить тегированный трафик, а также соответствующий номер VLAN
    R1(config-subif)#ip address 192.168.3.1 255.255.255.0   // Назначаем IP-адрес 
    R1(config-subif)# description Description for subinterface e0/0.3   // Включаем описание для подинтерфейса
    R1(config-subif)#exit
    R1(config)#interface e0/0.4   // Создаем логический подинтерфейс для vlan 4
    R1(config-subif)#encapsulation dot1q 4
    R1(config-subif)#ip address 192.168.4.1 255.255.255.0    
    R1(config-subif)# description Description for subinterface e0/0.4   
    R1(config-subif)#exit
    R1(config)#interface e0/0.8   // Создаем логический подинтерфейс для vlan 8
    R1(config-subif)#encapsulation dot1q 8
    R1(config-subif)# description Description for subinterface e0/0.8 
```
Проверка работоспособности подинтерфейсов:

![alt-текст](https://github.com/Galo9/HW_Network_engineer/blob/main/HW-1/HW1_interfaces.PNG)


### 5. Проверка работы маршрутизации между VLAN

#### 5.1 Проверка доступа для PC-A
```
    PC-A> ping 192.168.3.1   // Пинг шлюза по умолчанию
    PC-A> ping 192.168.4.3   // Пинг PC-B
    PC-A> ping 192.168.3.12  // Пинг S2
```

#### 5.2 Проверка доступа для PC-A
```
    PC-A> ping 192.168.3.3   // Пинг PC-A
```
![alt-текст](https://github.com/Galo9/HW_Network_engineer/blob/main/HW-1/HW1_ping.PNG)
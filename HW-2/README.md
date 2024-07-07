## Лабораторная работа - Развертывание коммутируемой сети с резервными каналами

### 1. Создание сети и настройка основных параметров устройства

#### 1.1 Топология сети:

![alt-текст](https://github.com/Galo9/HW_Network_engineer/blob/main/HW-2/HW2_topology.PNG)

#### 1.2 Настройка основных параметров коммутаторов

```
    Switch>enable   // Переход в привелигированный режим EXEC
    Switch#config t   // Переход в режим настройки
    Switch(config)#no ip domain lookup   // Отключаем поиск по DNS
    Switch(config)#hostname S1   // Назначаем имя коммутатору
    S1(config)#enable secret class   // Назначаем class в качестве пароля для привилегированного режима EXEC с шифрованием
    S1(config)#service password-encryption   // Шифруем пароли в конфигурации
    S1(config)#line console 0   // Назначаем пароль для консоли
    S1(config-line)#password cisco
    S1(config-line)#login  // Включаем вход в систему
    S1(config-line)#logging synchronous   //Запрещаем вывод консольных сообщений, которые могут прервать ввод команд в консольном режиме
    S1(config-line)#exit
    S1(config)#line vty 0 4   // Назначаем пароль для VTY
    S1(config-line)#password cisco
    S1(config-line)#login   // Включаем вход в систему
    S1(config-line)#exit
    S1(config)#no service password-encryption
    S1(config)#banner login $Unauthorized access is prohibited!$   // Создаем баннер о том, что несанкционированный доступ запрещен
    
    S1(config)#interface vlan 1   // Задаём IP-адрес для VLAN 1
    S1(config-if)#ip address 192.168.1.1 255.255.255.0   // Для коммутатора S2 IP-адрес - 192.168.1.2, для коммуторора S3 - 192.168.1.3
    S1(config-if)#no shutdown   // Включаем порт
    S1(config-if)#exit
    S1(config)#exit
    S1#copy run sta   // Копируем текущую конфигурацию в файл загрузочной конфигурации
```
#### 1.3 Проверка связи

Успешно ли выполняется эхо-запрос от коммутатора S1 на коммутатор S2?

Успешно ли выполняется эхо-запрос от коммутатора S1 на коммутатор S3?

Успешно ли выполняется эхо-запрос от коммутатора S2 на коммутатор S3?

Ответ: Да

![alt-текст](https://github.com/Galo9/HW_Network_engineer/blob/main/HW-2/HW2_ping.PNG)


### 2. Определение корневого моста

#### 1.1 Отключение всех портов на коммутаторах.

```
    S1(config)#interface e0/0
    S1(config-if)#shutdown   // Выключаем порт e0/0
    S1(config-if)#exit
    S1(config)#interface e0/1
    S1(config-if)#shutdown   // Выключаем порт e0/1
    S1(config-if)#exit
    S1(config)#interface e0/2
    S1(config-if)#shutdown   // Выключаем порт e0/2
    S1(config-if)#exit
    S1(config)#interface e0/3
    S1(config-if)#shutdown   // Выключаем порт e0/3
    S1(config-if)#exit  
```
Аналогично отключаются порты на коммутаторах S2 и S3.

#### 1.2 Настройка подключенных портов в качестве транковых.

```
    S1(config)#interface e0/0
    S1(config-if)#switchport trunk encapsulation dot1q   // Делаем процедуру инкапсуляции статической для настройки статического trunk 
    S1(config-if)#switchport mode trunk   // Переводим интерфейс в режим статического trunk
```
Аналогично порты E0/0, E0/1, E0/2 и E0/3 настраиваются в начестве транковых на коммутаторах S2 и S3.

#### 1.3 Включите порты e0/2 и e0/0 на всех коммутаторах.

```
    S1(config)#interface e0/0
    S1(config-if)#shutdown   // Включаем порт e0/0
    S1(config-if)#exit  
    S1(config)#interface e0/2
    S1(config-if)#shutdown   // Включаем порт e0/2
    S1(config-if)#exit 
```
Аналогично отключаются порты E0/2 и E0/0 на коммутаторах S2 и S3.

#### 1.4 Отобразите данные протокола spanning-tree.

![alt-текст](https://github.com/Galo9/HW_Network_engineer/blob/main/HW-2/HW2_spanning-tree.PNG)

Вопросы: 

1) Какой коммутатор является корневым мостом?

Ответ: Корневым мостом стал коммутатор S1:

2) Почему этот коммутатор был выбран протоколом spanning-tree в качестве корневого моста?

Ответ: Так как у всех коммутаторов равны значения приоритета и расширенного идентификатора системы (VLAN), корневым мостом становися коммутатор S1, потому что у него с наименьшее значение MAC-адреса.

3) Какие порты на коммутаторе являются корневыми портами? 

Ответ: Порты e0/0, e0/2 коммутатора S1.

4) Какие порты на коммутаторе являются назначенными портами? 

Ответ: На коммутаторе S2 - e0/2, на коммутаторе S3 - e0/0.

5) Какой порт отображается в качестве альтернативного и в настоящее время заблокирован?

Ответ: Порты E0/1, E0/3

Почему протокол spanning-tree выбрал этот порт в качестве невыделенного (заблокированного) порта?


### 3. Наблюдение за процессом выбора протоколом STP порта, исходя из стоимости портов


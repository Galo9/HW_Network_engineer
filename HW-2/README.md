## Лабораторная работа - Развертывание коммутируемой сети с резервными каналами

### 1. Создание сети и настройка основных параметров устройства

#### 1.1 Топология сети:

![alt-текст]()

#### 1.2 Настройка основных параметров коммутаторов:

```
    Switch>enable   // Переход в привелигированный режим EXEC
    Switch#config t   // Переход в режим настройки
    Switch(config)#no ip domain lookup   // Отключаем поиск по DNS
    Switch(config)#hostname S1   // Назначаем имя коммутатору
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



```
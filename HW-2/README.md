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



```
# Лабораторная работа №2. Развертывание коммутируемой сети с резервными каналами.
  - **Топология сети, выполненная в EVE-NG:**
    
    ![STP топология](https://github.com/MIranaNightshade/otus-networks/blob/main/lab2_STP/jpeg/STP%20%D1%82%D0%BE%D0%BF%D0%BE%D0%BB%D0%BE%D0%B3%D0%B8%D1%8F.png)
  - **Таблица адресации:**

    | Устройство | Интерфейс | IP адрес  | Маска подсети |
     |-----------|------------|-----------|-------------|
    | S1 | VLAN 1 | 192.168.1.1| 255.255.255.0|
    |S2| VLAN 1| 192.168.1.2| 255.255.255.0 |
    | S3 | VLAN 1 | 192.168.1.3 | 255.255.255.0|
    
   
   -  **Цели:**
     
  1. [Настройка основных параметров устройства](#title1)
  2. [Определение корневого моста](#title2)
  3. [Наблюдение за процессом выбора протоколом STP порта, исходя из стоимости портов](#title3)
  4. [Наблюдение за процессом выбора протоколом STP порта, исходя из приоритета портов](#title4)

## <a id="title1">Настройка основных параметров устройства</a>
 1. **Настроим базовые параметры каждого коммутатора:**
    
    - Отключим поиск DNS.
    - Присвоим имена устройствам в соответствии с топологией.
    - Назначим cisco в качестве зашифрованного пароля доступа к привилегированному режиму.
    - Назначим cisco в качестве паролей консоли и VTY и активируйте вход для консоли и VTY каналов.
    - Настроим logging synchronous для консольного канала.
    - Настроим баннерное сообщение дня (MOTD) для предупреждения пользователей о запрете несанкционированного доступа.
    - Зададим IP-адрес, указанный в таблице адресации для VLAN 1 на всех коммутаторах.

     **Базовые настройки на примере коммутатора S1:**
    ```
    !
    hostname S1
    !
    enable secret 5 $1$dAoX$IDS0cZakjTDLrS.wDrqKO0
    !
    no ip domain-lookup
    !
    !
    banner motd ^C
    ###################################
    #                                 #
    #   HANDS OFF FROM MY LITTLE S1   #
    #   AND GET OUT OF HERE           #
    #                                 #
    ###################################
    ^C
    !
    line con 0
     logging synchronous
    line aux 0
    line vty 0 4
    password cisco
     login
    line vty 5 15
     password cisco
     login
    !

    ```    
   2. **Проверим связь:**
      - Проверим способность коммутаторов обмениваться эхо-запросами:
        
        - ping от S1(192.168.1.1) до S2(192.168.1.2) и S3(192.168.1.3):
          
          ![STP проверка связи1](https://github.com/MIranaNightshade/otus-networks/blob/main/lab2_STP/jpeg/STP%20%D0%BF%D1%80%D0%BE%D0%B2%D0%B5%D1%80%D0%BA%D0%B0%20%D1%81%D0%B2%D1%8F%D0%B7%D0%B8%201.png)
        - ping от S2(192.168.1.2) до S3(192.168.1.3):
          
          ![STP проверка связи2](https://github.com/MIranaNightshade/otus-networks/blob/main/lab2_STP/jpeg/STP%20%D0%BF%D1%80%D0%BE%D0%B2%D0%B5%D1%80%D0%BA%D0%B0%20%D1%81%D0%B2%D1%8F%D0%B7%D0%B8%202.png)
        
        
## <a id="title2">Определение корневого моста.</a>
 1. Отключим все порты на коммутаторах.
 2. Настроим подключенные порты в качестве транковых.
 3. Включим порты G0/1 и G0/3 на всех коммутаторах.
 4. Отобразим данные протокола STP.

     **Данные протокола STP после выполнения пунктов 1-3:**
    
       S1:
    
     ![STP S1](https://github.com/MIranaNightshade/otus-networks/blob/main/lab2_STP/jpeg/STP%20S1.png)
    
       S2:
    
     ![STP S2](https://github.com/MIranaNightshade/otus-networks/blob/main/lab2_STP/jpeg/STP%20S2.png)
    
       S3:
    
     ![STP S3](https://github.com/MIranaNightshade/otus-networks/blob/main/lab2_STP/jpeg/STP%20S3.png)

     **Схема STP нашей изначальной топологии с указанием статусов и ролей портов:**

     ![STP sheme](https://github.com/MIranaNightshade/otus-networks/blob/main/lab2_STP/jpeg/%D0%92%D1%8B%D0%B1%D0%BE%D1%80%20STP%20root.png)

    **Ответы на вопросы:**
    
     a. **Какой коммутатор является корневым мостом?**
       - *Коммутатор S1 является ROOT BRIDGE.*
    
     b. **Почему этот коммутатор был выбран протоколом spanning-tree в качестве корневого моста?**
       - *S1 был выбран в ROOT т.к у него наименьший mac из всех 3 коммутаторов, сначала сравнение шло по ROOT BRIDGE PRIORITY, но т.к BRIDGE PRIORITY == 32768 у всех коммутаторов, то выбор производился по mac.*
    
     с. **Какие порты на коммутаторе являются корневыми портами?**
      - *Порт с которого был получен наилучший BPDU (по порядку сравниваются ROOT PATH COST, PORT PRIORITY и PORT ID) становится ROOT портом. ROOT порт это порт с наименьшей стоимостью пути до ROOT BRIDGE. В нашем случае S2 gi0/0 и S3 gi0/2*


      d. **Какие порты на коммутаторе являются назначенными портами?**
       - *Порты через которые осуществляется доступ к сегменту сети являются назначенными (designated) портами у ROOT BRIDGE все порты являются DESIGNATED. В нашем случае за исключением портов ROOT BRIDGE назначенными портами являются: S2 gi0/2 и S2 gi0/3*
        

      c. **Какие порты отображается в качестве альтернативных и в настоящее время заблокированы?**
       - *Порты которые не стали DESIGNATED или ROOT переводятся в состояние BLOCKED и являются ALTERNATE PORTS (не принимают и не передают кадры только слушают BPDU). В нашем случае это порты gi0/0, gi0/1 и gi0/3 на S3.*

        
      d. **Почему протокол spanning-tree выбрал эти порты в качестве невыделенных (заблокированных) портов?**
       - *Выбор осуществляется следующим образом: коммутатор запоминает BPDU которые когда-либо были им приняты или отправлены через любой порт, он сравнивает BPDU которые были приняты и отправлены не ROOT портом и в случае если отправленный им BPDU был хуже чем тот который был принят, то он принимает решение о переводе роли порта в ALTERNATE и статуса порта в BLOCKED. В нашем случае на S3 порт Gi0/3 стал в роль ALTE (состояние BLK) т.к mac S1 при одинаковых ROOT PATH COST и ROOT BRIDGE PRIORITY меньше чем его собственный и аналогично для портов gi0/1 и gi0/0 mac S2 меньше чем его собсвенный, т.о S3 через порты gi0/3, gi0/1 и gi0/0 отправил худшие BPDU чем те которые получил от S2 и S1.*

## <a id="title3">Наблюдение за процессом выбора протоколом STP порта, исходя из стоимости портов</a>
 1. **Оставим включенными только порты F0/2 и F0/4 на всех коммутаторах.**
 2. **Определим коммутатор с заблокированным портом.**
    - Коммутатор S3 порт gi0/1 перешел в состояние BLK, т.к mac S2 меньше чем mac S3 (5000.0002.0000 < 5000.0003.0000)  при одинаковых ROOT BRIDGE PRIORITY:

      S2:
      ```
      S2#show spanning-tree

      VLAN0001
        Spanning tree enabled protocol rstp
        Root ID    Priority    32769
                   Address     5000.0001.0000
                   Cost        4
                   Port        2 (GigabitEthernet0/1)
                   Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

        Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
                   Address     5000.0002.0000
                   Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
                   Aging Time  300 sec

      Interface           Role Sts Cost      Prio.Nbr Type
      ------------------- ---- --- --------- -------- --------------------------------
      Gi0/1               Root FWD 4         128.2    Shr
      Gi0/3               Desg FWD 4         128.4    Shr      
      ```

      S3:
      ```
      S3#show spanning-tree

      VLAN0001
       Spanning tree enabled protocol rstp
       Root ID    Priority    32769
                   Address     5000.0001.0000
                   Cost        4
                   Port        4 (GigabitEthernet0/3)
                   Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

        Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
                   Address     5000.0003.0000
                   Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
                   Aging Time  300 sec

      Interface           Role Sts Cost      Prio.Nbr Type
      ------------------- ---- --- --------- -------- --------------------------------
      Gi0/1               Altn BLK 4         128.2    Shr
      Gi0/3               Root FWD 4         128.4    Shr     
      ```  
 4. **Изменим стоимость порта на S3.**
    - Уменьшим стоимость порта Gi0/1 на S3:
      ```
      S3(config)#interface gigabitEthernet 0/3
      S3(config-if)#spanning-tree cost 2
      ```
         
 6. **Посмотрим изменения протокола spanning-tree.**
    - Статус STP на S2:
      
      ```
      S2#show spanning-tree

      VLAN0001
        Spanning tree enabled protocol rstp
        Root ID    Priority    32769
                   Address     5000.0001.0000
                   Cost        4
                   Port        2 (GigabitEthernet0/1)
                   Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

        Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
                   Address     5000.0002.0000
                   Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
                   Aging Time  300 sec

      Interface           Role Sts Cost      Prio.Nbr Type
      ------------------- ---- --- --------- -------- --------------------------------
      Gi0/1               Root FWD 4         128.2    Shr
      Gi0/3               Altn BLK 4         128.4    Shr
      
      ```
    - Статус STP на S3:
      
      ```
      S3#show spanning-tree

      VLAN0001
      Spanning tree enabled protocol rstp
      Root ID    Priority    32769
                 Address     5000.0001.0000
                 Cost        2
                 Port        4 (GigabitEthernet0/3)
                 Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

      Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
                 Address     5000.0003.0000
                 Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
                 Aging Time  300 sec

      Interface           Role Sts Cost      Prio.Nbr Type
      ------------------- ---- --- --------- -------- --------------------------------
      Gi0/1               Desg FWD 4         128.2    Shr
      Gi0/3               Root FWD 2         128.4    Shr

      ```
      Теперь порт Gi0/1 на S2 ушел в состояние BLK т.к получил от S3 лучшую BPDU чем отправил через порт Gi0/1 сам, ROOT PATH COST у S3 (2) стал меньше чем у S2 (4). (Сравнение по порядку ROOT PATH COST, BRIDGE PRIORITY, MAC)

## <a id="title4">Наблюдение за процессом выбора протоколом STP порта, исходя из приоритета портов.</a>
 - Включим порты gi0/0 gi0/2 на всех коммутаторах.
 - Выполним команду show spanning-tree на всех коммутаторах кроме ROOT BRIDGE:
   
   S2:
   ```
   VLAN0001
   Spanning tree enabled protocol rstp
   Root ID    Priority    32769
              Address     5000.0001.0000
              Cost        4
              Port        1 (GigabitEthernet0/0)
              Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

   Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
              Address     5000.0002.0000
              Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
              Aging Time  300 sec

   Interface           Role Sts Cost      Prio.Nbr Type
   ------------------- ---- --- --------- -------- --------------------------------
   Gi0/0               Root FWD 4         128.1    Shr
   Gi0/1               Altn BLK 4         128.2    Shr
   Gi0/2               Desg FWD 4         128.3    Shr
   Gi0/3               Desg FWD 4         128.4    Shr

   ```

   S3:
   ```
   S3#show spanning-tree

   VLAN0001
     Spanning tree enabled protocol rstp
     Root ID    Priority    32769
                Address     5000.0001.0000
                Cost        4
                Port        3 (GigabitEthernet0/2)
                Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

     Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
                Address     5000.0003.0000
                Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
                Aging Time  300 sec

   Interface           Role Sts Cost      Prio.Nbr Type
   ------------------- ---- --- --------- -------- --------------------------------
   Gi0/0               Altn BLK 4         128.1    Shr
   Gi0/1               Altn BLK 4         128.2    Shr
   Gi0/2               Root FWD 4         128.3    Shr
   Gi0/3               Altn BLK 4         128.4    Shr   
   ```
   ![STP топология](https://github.com/MIranaNightshade/otus-networks/blob/main/lab2_STP/jpeg/STP%20%D1%82%D0%BE%D0%BF%D0%BE%D0%BB%D0%BE%D0%B3%D0%B8%D1%8F.png)

   **Итого: на S2 порт Gi0/0 был выбран как ROOT, т.к приоритет порта (128.1) меньше чем у Gi0/1 (128.2) при одинаковых стоимостях COST(4).
   аналогично на S3 Gi0/2 выбран как ROOT т.к приоритет порта (128.3) меньше чем у Gi0/3(128.4) при одинаковых стоимостях COST(4).**

### Вопросы для повторения

1. Какое значение протокол STP использует первым после выбора корневого моста, чтобы определить выбор порта?
   - *стоимость порта (COST) - чем меньше тем лучше*
  

3. Если первое значение на двух портах одинаково, какое следующее значение будет использовать протокол STP при выборе порта?
   - *Приоритет порта (PORT PRIORITY) - чем меньше тем лучше*
   

3. Если оба значения на двух портах равны, каким будет следующее значение, которое использует протокол STP при выборе порта?
   - *Номер порта -  чем меньше тем лучше*   
     

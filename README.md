##  System Design социальной сети путешественников для курса по System Design

### 1. Функциональные требования к системе:

*  **Поиск пользователей:** пользователи должны получить возможность искать других пользователй по именам или фамилиям
*  **Создание постов:** публикация постов из путешествий с фотографиями, небольшим описанием и привязкой к конкретному месту путешествия
*  **Оценка постов:** оценка постов других путешественников
*  **Комментарии постов:** комментарии постов других путешественников
*  **Подписка на пользователей:** подписка на других путешественников, чтобы следить за их активностью
*  **Поиск популярных мест:** поиск популярных мест для путешествий и просмотр постов с этих мест по названию или по карте
*  **Просмотр постов:** просмотр ленты других путешественников


### 2. Нефункциональные требования к системе:
*  **Аудитория:** 10 000 000 DAU
*  **Доступность:** 99,95%
*  **Регион использования:** СНГ
*  **Активность:** в среднем пользователь будет заходить 3 раза в день, 20 постов просмотр за сессию
*  **Сезонность:** сезон отпусков май - сентябрь, так же новогодние каникулы нагрузка возрастает в 3 раза
*  **Хранение данных:** данные храним всегда
*  **Лимиты и ограничения:** не более 20 постов в день
*  **Временные ограничения:** добавление поста не более 2 сек, поиск по пользователю 3 сек, добавление комментария 2 сек, загрузка постов 1 сек

### 3. Расчет нагрузки на будущую систему:
#### Посты
RPS (запись) добавление постов:

    DAU = 10 000 000
    Кол-во постов в день = 20
    RPS = 10 000 000 * 20 / 86 400 ~= 2314 

RPS (чтение) загрузка ленты постов:

    DAU = 10 000 000
    Кол-во сессий в день = 3
    Кол-во просмотров постов на сессию = 20
    RPS = 10 000 000 * 3 * 20 / 86 400 ~= 7000 


#### Комментарии
RPS (запись) добавление комментариев:

    DAU = 10 000 000
    Кол-во сессий в день = 3
    Кол-во добавление комментариев за сессию = 3
    RPS = 10 000 000 * 3 * 3 / 86 400 ~= 1050 

RPS (чтение) чтение комментариев:

    DAU = 10 000 000
    Кол-во сессий в день = 3
    Кол-во открытий комментариев поста за сессию = 5
    Среднее кол-во комментариев в посте = 5
    RPS = 10 000 000 * 3 * 5 * 5 / 86 400 ~= 8700 

#### Реакции
RPS (запись) оценка постов:

    DAU = 10 000 000
    Кол-во сессий в день = 3
    Среднее кол-во оценок постов за сессию = 5
    RPS = 10 000 000 * 3 * 5 / 86 400 ~= 1750 


#### Подписки
RPS (запись) подписка на пользователей:

    DAU = 10 000 000
    Среднее кол-во подписок на пользователей в месяц = 3
    RPS = 10 000 000 * 3 / 86 400 / 7 ~= 50

#### Локации
RPS (запись) добавление мест (такое же что добавление постами):

    DAU = 10 000 000
    Кол-во постов в день = 20
    RPS = 10 000 000 * 20 / 86 400 ~= 2314 

RPS (чтение) поиск популярных мест:

    DAU = 10 000 000
    Среднее кол-во поиска в день = 1
    RPS = 10 000 000 / 86 400 ~= 115 

#### Пользователи
RPS (запись) добавление пользователей:

    DAU = 10 000 000
    RPS = 10 000 000 / 365 / 86 400 ~= 0.32 

RPS (чтение) поиск пользователей:

    DAU = 10 000 000
    Среднее кол-во поиска в день = 2
    RPS = 10 000 000 * 2 / 86 400 ~= 230 

### 4. Оценка дисков:
#### Оценка на 1 запись по таблицам БД
Ориентировочно 1 фото в хранилище весит 400 кб для постов, 800 для фото пользователей

    Table follows {
        following_user_id (16 byte)
        followed_user_id  (16 byte)
        created_at (8 byte)
    } (32 byte)
    
    Table users {
        id (16 byte)
        first_name (100 byte)
        last_name (100 byte)
        email (100 byte)
        profile_picture (100 byte) + (800 Kb - S3 blob storage)
        created_at (8 byte)
    } (424 byte) - 800.424 Kb with pictures 
    
    Table posts {
        id  (16 byte)
        title (100 byte)
        body (1000 byte)
        user_id (16 byte)
        location_id (16 byte)
        photos (100 * 3 max photos = 300 byte) (1.2Mb - S3 blob storage)
        created_at (8 byte)
    } (1456 byte) - 1.2 Mb with pictures 
    
    Table ratings {
        id (16 byte)
        user_id (16 byte)
        post_id (16 byte)
        rating (8 byte)
        created_at (8 byte)
    } (64 byte)
    
    Table comments {
        id (16 byte)
        user_id (16 byte)
        post_id (16 byte)
        reply_id (16 byte)
        text (500 byte)
        created_at (8 byte)
    } (572 byte)
    
    Table locations {
        id (16 byte)
        name (100 byte)
        geometry (48 byte)
        created_at (8 byte)
    } (172 byte)

#### Оценка трафика и дисков 1 год
#### Посты
Запись:

    RPS = 2314
    Traffic metadata = 2314 * 1.456 Kb ~= 3.3 MB/s
    Traffic media = 2314 * 1.2Mb ~= 2.77 GB/s

Чтение:

    RPS ~= 7000
    Traffic metadata = 7000 * 1.456 Kb ~= 10.1 MB/s
    Traffic media = 7000 * 1.2Mb ~= 8.4 GB/s

Итого трафик:

    Traffic metadata: 13.4 MB/s
    Traffic media: 11.17 GB/s
    IOPS: 9314
    Capacity metadata: 13.4 MB/s * 86400 * 365 = 422.5 ТB
    Capacity media: 2.77 GB/s * 86400 * 365 = 87.5 PB

Итого расчет дисков:

    METADATA
    ---------------
    HDD 
    Disks_for_capacity = 422.5 ТB / 32ТБ = 13.1
    Disks_for_throughput = 13.4 MB/s / 100 МБ/с = 0.134
    Disks_for_iops = 9314 / 100 = 93.14
    Disks = max(ceil(13.1), ceil(0.134), ceil(93.14)) = 93.14

    SSD (SATA) 
    Disks_for_capacity = 422.5 ТB / 100ТБ = 4.22
    Disks_for_throughput = 13.4 MB/s  / 500 МБ/с = 0.02
    Disks_for_iops = 9314 / 1000 = 9.314
    Disks = max(ceil(4.22), ceil(0.02), ceil(9.314)) = 10

    SSD (nVME)
    Disks_for_capacity = 422.5 ТB / 30ТБ = 14.06
    Disks_for_throughput = 13.4 MB/s  / 3000 МБ/с = 0.004
    Disks_for_iops = 9314 / 10000 = 0.9314
    Disks = max(ceil(14.06), ceil(0.004), ceil(0.9314)) = 15

    ---------------
    MEDIA
    ---------------
    HDD 
    Disks_for_capacity = 87.5 PB / 32ТБ = 2734
    Disks_for_throughput = 11.17 ГБ/с / 100 МБ/с = 111.7
    Disks_for_iops = 9314 / 100 = 93.14
    Disks = max(ceil(2734), ceil(111.7), ceil(93.14)) = 2734

    SSD (SATA) 
    Disks_for_capacity = 87.5 PB / 100ТБ = 875
    Disks_for_throughput = 11.17 ГБ/с  / 500 МБ/с = 22.3
    Disks_for_iops = 9314 / 1000 = 9.314
    Disks = max(ceil(875), ceil(22.3), ceil(9.314)) = 875

    SSD (nVME)
    Disks_for_capacity = 87.5 PB / 30ТБ = 2916.6
    Disks_for_throughput = 11.17 ГБ/с  / 3000 МБ/с = 3.72
    Disks_for_iops = 9314 / 10000 = 0.9314
    Disks = max(ceil(2916.6), ceil(3.72), ceil(0.9314)) = 2917

Предпочтительная система ранения метадаты SSD (SATA) из 10 дисков по 100ТБ
Предпочтительная система ранения медиа SSD (SATA) из 875 дисков по 100ТБ

#### Комментарии
Запись:

    RPS ~= 1050
    Traffic = 1050 * 572 B ~= 0,6MB/s

Чтение:

    RPS ~= 8700
    Traffic = 8700 * 572 ~= 4,97 MB/s

Итого трафик:

    Traffic: 5,57 MB/s
    IOPS: 9750
    Capacity: 0,6 MB/s * 86400 * 365 = 18.9 ТB

Итого расчет дисков:

    HDD 
    Disks_for_capacity = 18.9 ТB / 2ТБ = 9.45
    Disks_for_throughput = 5,57 MB/s / 100 МБ/с = 0.057
    Disks_for_iops = 9750 / 100 = 97.50
    Disks = max(ceil(9.45), ceil(0.057), ceil(97.50)) = 98

    SSD (SATA) 
    Disks_for_capacity = 18.9 ТB / 2ТБ = 9.45
    Disks_for_throughput = 5,57 MB/s / 500 МБ/с = 0.01
    Disks_for_iops = 9750 / 1000 = 9.750
    Disks = max(ceil(9.45), ceil(0.01), ceil(9.750)) = 10

    SSD (nVME)
    Disks_for_capacity = 18.9 ТB / 30ТБ = 0.63
    Disks_for_throughput = 5,57 MB/s / 3000 МБ/с = 0.0001
    Disks_for_iops = 9750 / 10000 = 0.975
    Disks = max(ceil(0.63), ceil(0.0001), ceil(0.975)) = 1

Предпочтительная система ранения SSD (nVME) из 1 диска 20 ГБ

#### Реакции
Запись:

    RPS ~= 1750 
    Traffic = 1750 * 64 B ~= 0.112 MB/s

Чтение:

    RPS ~= 7230 (7000 RPS лента постов + 230 RPS поиск пользователей) 
    Traffic = 7230 * 64 ~= 0.462 MB/s

Итого трафик:

    Traffic: 0.574 MB/s
    IOPS: 8980
    Capacity: 0.112 MB/s * 86400 * 365 = 3.53 TB

Итого расчет дисков:

    HDD 
    Disks_for_capacity = 3.53 TB / 2ТБ = 1.765
    Disks_for_throughput = 0.574 MB/s / 100 МБ/с = 0.00574
    Disks_for_iops = 8980 / 100 = 89.80
    Disks = max(ceil(1.765), ceil(0.00574), ceil(89.80)) = 90

    SSD (SATA) 
    Disks_for_capacity = 3.53 TB / 2ТБ = 1.765
    Disks_for_throughput = 0.574 MB/s / 500 МБ/с = 0.0011
    Disks_for_iops = 8980 / 1000 = 8.980
    Disks = max(ceil(1.765), ceil(0.0011), ceil(8.980)) = 9

    SSD (nVME)
    Disks_for_capacity = 3.53 TB / 4ТБ = 0.8825
    Disks_for_throughput = 0.574 MB/s / 3000 МБ/с = 0.0001
    Disks_for_iops = 8980 / 10000 = 0.8980
    Disks = max(ceil(0.8825), ceil(0.0001), ceil(0.8980)) = 1

Предпочтительная система ранения SSD (SATA) из 1 диска 4 ГБ

#### Локации
Запись:

    RPS ~= 2314 (добавление с постами)
    Traffic = 2314 * 172 B ~= 0.398 MB/s

Чтение:

    RPS = 115 (поиск по местам) + 7000 (лента постов) = 7115
    Traffic = 7115 * 172 B ~= 1.223 MB/s

Итого траффик:

    Traffic: 1.621 MB/s
    IOPS: 9429
    Capacity: 1.621 MB/s * 86400 * 365 = 12.55 ТB

Итого расчет дисков:

    HDD 
    Disks_for_capacity = 12.55 ТB / 2GB = 6.2
    Disks_for_throughput = 1.621 MB/s / 100 МБ/с = 0.01621
    Disks_for_iops = 9429 / 100 = 94.29
    Disks = max(ceil(6.2), ceil(0.01621), ceil(94.29)) = 95

    SSD (SATA) 
    Disks_for_capacity = 12.55 ТB / 2GB = 6.2
    Disks_for_throughput = 1.621 MB/s / 500 МБ/с = 0.003
    Disks_for_iops = 9429 / 1000 = 9.429
    Disks = max(ceil(6.2), ceil(0.003), ceil(9.429)) = 10

    SSD (nVME)
    Disks_for_capacity = 12.55 ТB / 15 GB = 0.83
    Disks_for_throughput = 1.621 MB/s / 3000 МБ/с = 0.0005
    Disks_for_iops = 9429 / 10000 = 0.9429
    Disks = max(ceil(0.83), ceil(0.0005), ceil(0.9429)) = 1

Предпочтительная система ранения SSD (nVME) из 1 диска 15 ГБ

#### Пользователи
Запись:

    RPS ~= 0.32 
    Traffic metadata = 0.32 * 0.424 KB/s ~= 0.135 KB/s
    Traffic media = 0.32 * 0.8 MB/s ~= 0.256 MB/s

Чтение:

    RPS ~= 230 (поиск) + 7000 RPS лента постов = 7230
    Traffic metadata = 7230 * 0.424 KB/s ~= 3.06 MB/s
    Traffic media = 7230 * 0.8 MB/s ~= 5.784 GB/s

Итого траффик:

    Traffic metadata: 3.06 MB/s
    Traffic media: 5.784 GB/s
    IOPS: 7230
    Capacity metadata: 0.135 KB/s * 86400 * 365 = 4.23 GB
    Capacity media: 0.256 MB/s * 86400 * 365 = 9.07 Tb

Итого расчет дисков:

    METADATA
    ---------------
    HDD 
    Disks_for_capacity = 4.23 GB / 1 Gb = 4.23
    Disks_for_throughput =  3.06 MB/s / 100 МБ/с = 0.03
    Disks_for_iops = 7230 / 100 = 72.30
    Disks = max(ceil(4.23), ceil(0.03), ceil(72.30)) = 73

    SSD (SATA) 
    Disks_for_capacity = 4.23 GB / 1 GB = 4.23
    Disks_for_throughput =  3.06 MB/s / 500 МБ/с = 0.006
    Disks_for_iops = 7230 / 1000 = 7.230
    Disks = max(ceil(4.23), ceil(0.006), ceil(7.230)) = 8

    SSD (nVME)
    Disks_for_capacity = 4.23 GB / 8 GB = 0.52
    Disks_for_throughput =  3.06 MB/s / 3000 МБ/с = 0.001
    Disks_for_iops = 7230 / 10000 = 0.7230
    Disks = max(ceil(0.52), ceil(0.001), ceil(0.7230)) = 1

    ---------------
    MEDIA
    ---------------
    HDD 
    Disks_for_capacity = 9.07 TB / 1 ТБ = 9.07
    Disks_for_throughput = 5.784 GB/s / 100 МБ/с = 57.84
    Disks_for_iops = 7230 / 100 = 72.30
    Disks = max(ceil(9.07), ceil(57.84), ceil(72.30)) = 73

    SSD (SATA) 
    Disks_for_capacity = 9.07 TB / 1ТБ = 9.07
    Disks_for_throughput = 5.784 GB/s / 500 МБ/с = 11.5
    Disks_for_iops = 7230 / 1000 = 7.230
    Disks = max(ceil(9.07), ceil(11.5), ceil(7.230)) = 12

    SSD (nVME)
    Disks_for_capacity = 9.07 TB / 4ТБ = 2.035
    Disks_for_throughput = 5.784 GB/s / 3000 МБ/с = 1.9
    Disks_for_iops = 7230 / 10000 = 0.7230
    Disks = max(ceil(2.035), ceil(1.9), ceil(0.7230)) = 3

Предпочтительная система хранения метадаты SSD (nVME) из 1 диска 8 ГБ
Предпочтительная система хранения медиа SSD (SATA) из 12 дисков по 1 ТБ

### 4. Расчет хостов:
#### Посты
    RPS запись - 2314
    RPS чтение - 7000
    Метадата - 10 дисков
    Медиа - 875 дисков

    METADATA
    Replication:
        - master slave / async / RF 2
    Sharding:
        - key based by user id = 3 shards by 2 replicas

    10 / 3  = 3.3 * 2 ~= 7 hosts
    
    MEDIA
    Replication:
        - master - cdn POP pull/ async / RF 2 CDN
    Sharding:
        - 3 shards by 2 CDN replicas
    
    875 / 3 = 291.6 * 2 ~= 583 hosts

#### Комментарии
    RPS запись - 1050
    RPS чтение - 8700
    Метадата - 1 диск

    Replication:
        - master slave / async / RF 2
    Sharding:
        - key based by post id = 1 shard by 2 replicas

    1 / 1  = 1 * 2 ~= 2 hosts

#### Реакции
    RPS запись - 1750
    RPS чтение - 7230
    Метадата - 1 диск

    Replication:
        - master slave / async / RF 2
    Sharding:
        - key based by post id = 1 shards by 2 replicas

    1 / 1  = 1 * 2 ~= 2 hosts

#### Локации
    RPS запись - 2314
    RPS чтение - 7115
    Метадата - 1 диск

    Replication:
        - master slave / sync / RF 2
    Sharding:
        - zone based by geometry = 1 shard by 2 replicas

    1 / 1  = 1 * 2 ~= 2 hosts

#### Пользователи
    RPS запись - 0.32 
    RPS чтение - 7230
    Метадата - 1 диск
    Медиа - 12 дисков

    METADATA
    Replication:
        - master slave / sync / RF 2
    Sharding:
        - key based by user id - 1 shard by 2 replicas

    1 / 1  = 1 * 2 ~= 2 hosts
    
    MEDIA
    Replication:
        - master - cdn POP pull / async / RF 2 CDN
    Sharding:
        - 3 shards by 2 CDN replicas
    
    12 / 3 = 4 * 2 ~= 8 hosts
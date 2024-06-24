# Дизайн для системы глобального дейтингового сервиса
Система предполагает аналог *Tinder*, *Bumble*, *Badoo*, etc. 
В максимально приближенной к действительности постановке задачи, это будет мобильное приложение и веб-интерфейс, представленные (и, соответственно, локализованные) на рынке практически всех стран мира, c ± 100M MAU (Monthly active users) и ±10M DAU (Daily active users).
Разработка дизайна предполагается по мере приращения требований: от Базового до Третьего уровня.

# Первый уровень: Базовый дизайн
## Функциональные требования
1. **Профили пользователей**:
   - пользователи могут создавать, сохранять и редактировать свои анкеты.
2. **Выставление оценки чужой анкете**: 
   - пользователям могут выставлять оценки другим анкетам (по принципу лайк/дизлайк).
3. **Доступ к приложению**:
   - просматривать профили и контент в них могут только зарегистрированные пользователи,
   - есть возможность блокировки неприятных анкет.
5. **Соответствие**: 
   - когда два пользователя выставляют положительные оценки друг другу, считается, что они подходят друг другу, поэтому для них активируется чат.
6. **Уведомления**: 
   - пользователи получают уведомления о совпадениях и сообщениях.
7. **Медиаконтент**: 
   - пользователи могут загружать фотографии и короткие видеоролики.
8. **Чат в режиме реального времени**: 
   - совпадающие пользователи могут общаться в режиме реального времени.

## Нефункциональные требования
1. **Масштабируемость** для обеспечения высокой нагрузки: 
   - поддержка ±100 миллионов MAU (ежемесячно активных пользователей) и ±10 миллионов DAU (ежедневно активных пользователей).
2. **Доступность**: 
   - время безотказной работы составляет 99,9%, иначе пользователи уйдут в другой сервис знакомств.
3. **Консистентность** (для чатов):
   - в чатах сообщения должны быть одинаковыми для всех пользователей, иначе случится драма и отказ в дальнейшем использовании сервиса
4. **Надёжность**:
   - надёжное хранение и передача пользовательских данных
5. **Устойчивость к разделению**:
   - географическая распределённость диктует необходимость 
  
## Расчёт нагрузки
Исходим из предположения, что наше приложение позволяет отображать на странице до 10 фотографий и изображений средним размером в 500 кБ, либо 1 фото и максимум 9 коротких полуминутных видеороликов, оптимизированных под конкретное устройство, с которого происходит просмотр анкеты. Остальная информация в анкете (описание, история лайков и т.п.) несоизмеримо меньше по размерам. Предполагаемый размер видеоролика рассчитывается из следующего: FullHD-видео (1080p) длиной в час весит 1,5 ГБ (~420кБ, округляем до 500 кБ). 
### Запросы
   - 10M пользователей * 50 * (30 * 9 + 1) (предполагается, что пользователь просматривает по 50 анкет за сутки, включая всё видео и фото профиля) / 86400 секунд = 2М RPS (грубо округляем в сторону целого операции на чтение),
   - 100K пользователей * 10 медиа (фото+видео) = 1М RPS (на запись).
### Сетевая нагрузка
   - 500кБ (размер секунды видео или одного изображения) * 2М (число RPS на чтение) = 1 Тб/с (суточный трафик),
   - 1 Тб/с * 86400 секунд * 366 дней = 32К Пб (годовой трафик).
### Хранение данных
   - 100М пользователей ежемесячно * (30 * 9 + 1) объём контента анкеты * 500 кБ * 12 месяцев = 163 Пб (в год)
### Вычислительные мощности
   - на выполнение чтение/запись согласно расчётам потребуется **500** инстансов,
   - если один инстанс может обрабатывать 10 Гб/с трафика, нам потребуется **100** инстансов для обработки сетевой нагрузки в 1 Тб/с,
   - для хранения данных в 163 Пб потребуется (с учётом резервирования дисков) ~ 41К инстансов (в год).
### Стоимость
   - стоимость сетевой нагрузки (выходящего трафика) в объеме 32 ПБ в год может варьироваться в зависимости от выбранного облачного провайдера:
      - AWS: около $2.24 миллионов в год,
      - Google Cloud: около $1.6 миллионов в год,
      - Azure: около $1.6 миллионов в год.
   - стоимость хранения данных объемом 163 ПБ в год может варьироваться в зависимости от выбранного облачного провайдера:
      - AWS (Amazon S3): около $44.99 миллионов в год,
      - Google Cloud Storage: около $39.12 миллионов в год,
      - Azure Blob Storage: около $35.99 миллионов в год.

## Компоненты и взаимодействия
1. User service управляет анкетами пользователей
2. Auth service - сервис аутентификации
3. Match service (сервис сопоставлений) управляет логикой "нравится", "не нравится" и "соответствует"
4. Notification service (сервис уведомлений) отправляет различные виды уведомлений (push, смс, email) о совпадениях и сообщениях
5. Media service позволяет осуществлять загрузку и просмотр медиаконтента (изображения, короткие видео)
6. Chat service управляет чатом между пользователями в режиме реального времени.

## Высокоуровневый дизайн
![изображение](https://github.com/trofimovelijah/System-Design/assets/15788014/32809fd3-2c07-4c39-8754-bd8d4cc36bd8)

## Хранилища и базы данных
1. Ведение анкеты в User service:
   - пользователи создают/редактируют профили. Сами данные чувствительны, поэтому их уместно хранить в реляционной СУБД, наподобие PostgreSQL.
2. Сопоставления в Match service:
   - действия пользователя регистрируются, а совпадения идентифицируются службой сопоставления. Такая база должна удовлетворять ACID-свойствам и к ней можно быстро осуществлять различные join'ы. Поэтому разумно использовать реляционную СУБД,
   - сами лайки/дизлайки уместно хранить в БД типа ключ-значение, например, Redis.
3. Уведомления:
   - запускаются при совпадениях, уведомления отправляются через службу push-уведомлений. Данные о логировании сообщений целесообразно хранить в колоночной БД, поскольку у нас всего три формата однородных уведомлений, а доступ к самой базе нужен преимущественно на чтение.
4. Медиаконтент:
   - загруженные медиа хранятся в распределенном файловом S3-хранилище.
5. Чат:
    - сообщения хранятся в документоориентированная NoSQL БД и извлекаются в режиме реального времени.
6. Аутентификация:
   - сервис хранит данные о регистрации, аутентификации и сессиях пользователей. Необходимо удовлетворять ACID-свойствам, поэтому оптимально применять реляционную СУБД. Возможно использовать базу сервиса User service, но для большой системы это небезопасно. При этом появляется вопрос синхронизации данных баз разных сервисов. 

## Компонентный дизайн
1. **Пользовательский доступ**:
   - User UI - анкета и оценки,
   - Auth UI - форма входа и регистрации,
   - Chat UI - переписка, 
   - Media UI - страница загрузки и просмотра медиаконтента, главным образом, видео.
2. В качестве брокера сообщений выбираем не RabbitMQ, а Apache Kafka, поскольку он оптимальнее подходит под задачи масштабирования, а фоновых задач, как таковых, кроме загрузки видео, пока не предвидится.
3. Добавлено отдельное хранилище для быстрых сообщений (лайк/дизлайк) в User service
4. Сервис управления медиаконтентом описан подробнее:
   - декомпозиция на сервис загрузки Media upload service и сервис просмотра Media view service,
   - загружаемые короткие видео разных форматов проходят перекодировку для оптимизации отображения и обрезки видео.
5. Сервис уведомлений: 
   - подключён к брокеру сообщений Apache Kafka,
   - три вида уведомлений отправляются по внешнему API,
   - отдельно выделено логирование уведомлений в колоночную БД.
6. Для обеспечения синхронизации между User service и Auth service также используется обмен сообщений через Kafka. Подобную конструкцию проще масштабировать, а также повышается надёжность, обеспечивая доставку даже в случае временных сбоев.
7. Взаимодействие других сервисов можно пустить также через очередь:
   - сервис аутентификации всегда оказывается продьюсером,
   - по отношению к нему сервис сообщений всегда консьюмер через брокер,
   - пользовательский сервис может быть как продьюсером для Chat service, так и консьюмером для Auth service,
   - сервис оповещений всегда консьюмер

![изображение](https://github.com/trofimovelijah/System-Design/assets/15788014/abac0228-bb44-4a37-a328-e8bbfb39df1c)

## Масштабирование системы и повышение надёжности
### Меры по масштабированию системы
Ранее проведённые расчёты нагрузки показали потребность в очень больших мощностях. 100М ежемесячных пользователей вызывает необходимость масштабировать систему. Рассмотрим возможные сценарии:
1. Необходимость распределять трафик с помощью балансировщиков нагрузки:
   - между пользовательским доступом и сервисами User service, Auth service, Chat service и двумя Media service,
   - ввиду большого числа рассчитанных инстансов сервисов целесообразно использовать балансировщики между ними,
   - при обращении к S3-хранилищу также нужно добавить LB.
2. Балансировщики можно снабдить резервными экземплярами в режиме Stand by.
3. Применение репликации данных:
   - для реляционных СУБД используется репликация,
   - для базы данных сервиса сообщений организована репликация методом кворума, поскольку это наиболее оптимальный способ для выполнения требования по консистентности данных в чатах,
   - хранилище S3 уместно разбить на партиции.
4. По подсчётам число инстансов сервисов может достигать порядка полутысячи. Для масштабирования нагрузки используются инстансы, которые в случае отказа основного инстанса подхватывают работу.

### Меры по улучшению отзывчивости
1. Первым способом повышения отзывчивости выступает кэширование данных. В зависимости от БД имеет смысл применить разный способ инвалидации кэша:
   - для сервиса уведомлений кэш использовать не будем, поскольку данный сервис не предполагает повышенных требований к отзывчивости (уведомление допустимо приходить через определённый промежуток времени),
   - для сервиса сравнений Match service уместно применить реверсивную запись кэша, поскольку этим обеспечивается высокая скорость записи,
   - сквозная запись кэша целесообразна для сервиса сообщений (в переписке данные должны быть консистентны),
   - у сервиса аутентификации кэш предполагается записывать в обход, поскольку логично его использование только после получения аутентификационных данных,
   - аналогично для базы данных анкет Profiles запись кэша в обход,
   - запись напрямую в источник также целесообразна для S3-хранилища,
   - быстрое хранилище Likes на Redis не имеет смысл кэшировать, поскольку эти данные хранятся в оперативной памяти, а долгосрочная статистика храниться в базе данных Profiles, поэтому оставим под вопросом.
2. Поскольку предполагается, что система должна быть географически распределена, то имеет смысл использовать CDN для ускорения отзыва:
   - использовать между пользовательскими точками входа, наподобие Chat UI (переписка может быть между людьми с разных географических и часовых зон),  и соответствующим сервисом,
   - для Media UI использование CDN целесообразно с сервисом Media view service.
3. Индексирование баз данных применяем для всех случаев, когда число операций чтения больше операций на запись:
   - для S3 применяем индексацию (под вопросом) со стороны сервиса Media view service, поскольку сам сервис предназначен для просмотра медиаконтента,
   - база сервиса аутентификации под вопросом, поскольку для входа в приложение выполняется условный POST-запрос, который данные записывает. Далее пользователь использует токен авторизации (на будущее - рассмотреть вопрос хранения токена в UI),
   - для Auth UI и User UI уместно использовать пользовательский кэш
4. Для повышения отзывчивости целесообразно применить установка соединения между клиентом и сервером по веб-протоколу HTTP:
   - для Chat UI (отправив сообщение, пользователь ждёт оперативной доставки ответа от собеседника) можно применить либо HTTP Long-Polling соединение, либо соединение по WebSocket,
   - от пользовательского сервиса ожидаются оповещения (например, поставил ли кто-нибудь лайк анкете пользователя), поэтому также применяем HTTP Long-Polling, 
   - в остальных случаях от User service и от клиента User UI можно отправлять данные по WebSocket.

## Итоговый дизайн системы Базового уровня
На данной схеме не учитываются дополнительные сервисы (мониторинг, безопасность, etc), поскольку они будут отображены в дизайне Третьего уровня.
![изображение](https://github.com/trofimovelijah/System-Design/assets/15788014/cb5a1705-6143-49e4-acc7-43f9be262b7c)


# Второй уровень: Фильтрация по геолокации и предпочтениям
## Функциональные требования
1. **Поиск по геолокации**:
   - пользователи могут искать анкеты других пользователей в заданном радиусе от текущего местоположения,
   - поиск может осуществлять по конкретным местоположениям (возможно указать точку на карте).
2. **Фильтры предпочтений**:
   - фильтрация анкет на основе предпочтений пользователя (возраст, пол, интересы и т.д.).
3. **Соответствие** (дополнение):
   - сервис соответствий может подбирать анкеты в т.ч. с учётом географического положения.

## Нефункциональные требования
Помимо указанных ранее требований
1. **Масштабируемость** для обеспечения высокой нагрузки: 
   - система должна быть способна масштабироваться горизонтально для поддержки увеличивающегося числа пользователей,
   - данные должны быть шардированы для равномерного распределения нагрузки между серверами.
2. **Доступность**: 
   - время безотказной работы составляет 99,9%, иначе пользователи уйдут в другой сервис знакомств.
3. **Производительность**:
   - система должна поддерживать высокую производительность при больших нагрузках, обеспечивая быстрый поиск и фильтрацию анкет,
   - время отклика системы должно быть минимальным даже при большом количестве запросов.
4. **Надёжность**:
   - надёжное хранение и передача пользовательских данных.

## Компоненты и взаимодействия
1. Сервис геолокации Location service:
   - хранит и обновляет данные о местоположении в геопространственной базе данных,
   - обработка запросов на основе геолокации в пределах радиуса,
   - сегментирование данных для управления высокой плотностью пользователей в городах.
2. Сервис поиска и фильтрации GeoSearch service:
   - обрабатывает сложные запросы, объединяющие геолокацию и предпочтения,
   - индексирует анкеты пользователей с помощью атрибутов геолокации и предпочтений.
3. Сервис индексации данных Indexing POI service отвечает за непосредственно логику выполнения индексации гео и данных в анкетах.
4. Данные сервиса поиска соотносятся с данными сервиса сопоставления Match service и пользовательского сервиса User service (их тут подробно описывать не станем).

## Высокоуровневый дизайн
![изображение](https://github.com/trofimovelijah/System-Design/assets/15788014/8484fd28-4bb5-4c20-bf29-7bcd8262b2b8)

## Хранилища и базы данных
1. В качестве POI DB используем пространственную специализированную СУБД, наподобие PostGIS. Данная база содержит данные о местоположении анкеты, а также описательную/атрибутивную часть.
2. Location service определяет услуги геолокации, используя геопространственную БД (например, PostGIS) для хранения и запроса данных о местоположении, а итоговые данные размещать в колоночной БД типа Cassandra (честно подсмотрено в *Google Maps System Design*):
   - сегментирование на основе дерева квадрантов или сетки, чтобы справиться с высокой плотностью пользователей в городах,
   - согласованность и синхронизацию между сегментами,
   - содержание метаданных о позиции пользователей (имя, пол, etc),
   - для выполнения пользовательского поиска (определения геопозиции) данные хранятся в оперативной памяти (соответственно позиция хранится в Redis), для чего удобно использовать квадродерево.
3. Сервис поиска и фильтрации GeoSearch service использует для быстрого запроса и фильтрации ElasticSearch:
   - индексирует профили пользователей с помощью атрибутов геолокации и предпочтений,
   - поиск может осуществляться (помимо радиуса) по разным совокупностям значений параметров (отдельно пол, пол и возраст, возраст/привычки/цвет глаз, etc), поэтому рационально результаты поиска хранить в документной БД,
   - обеспечивается высокая масштабируемость и производительность, что важно для покрытия нефункциональных требований.
4. Для фильтрации по расстоянию используется GeoHash

## Компонентный дизайн
1. Данные о местонахождении подходящих анкет отображаются в профиле User service
2. На основании фильтрации пользователей по геолокации также формируются предпочтения в Match service (после обработки в БД)
3. После локализации местонахождения из Location service данные через сборщик местоположений (веб-сервер Location Builder + колоночная БД) попадает в геопространственную БД PostGIS
4. Взаимодействие основных сервисов через Kafka

![изображение](https://github.com/trofimovelijah/System-Design/assets/15788014/d8904ef7-000f-480b-92a5-83e09da96487)

## Масштабирование системы и повышение надёжности
1. Масштабирование достигается использованием балансировщика нагрузки между пользовательским интерфейсом Search UI на фронтенде и сервисом фильтрации и геопоиска, а также между Location service, GeoSearch service и другими. 
2. У балансировщиков используем резервные сервера для повышения избыточности.
3. Для GeoHash добавляем реплики на чтение
4. Для QuadTree и Location Builder увеличиваем число инстансов (без увеличения реплик)
5. Каждый сервис уместно реализовать из нескольких инстансов, чтобы в случае недоступности одного из них выполнение функций возлегло на свободный инстанс.
6. У POI DB обязательные реплики.
7. Быстрое хранилище Redis нет смысла реплицировать, но в целях повышения надёжности имеет смысл завести резервный экземпляр в режиме Stand by.
8. Между сервисами сообщения идут по WebSocket
9. При осуществлении геопоиска данные берутся из кэша, созданного в обход из POI DB, ввиду отсутствия критичности в требованиях к консистентности.
10. Сервис геоиндексации уместно поделить на инстансы
11. Под вопросом создание индексов для ElasticSearch, поскольку в цепочке далее используется Indexing POI service
12. Аналогично и для POI DB PostGIS
13. А вот перед базой Cassandra данные кэшируются для повышения отзывчивости у Location service.

## Итоговый дизайн системы Второго уровня
![изображение](https://github.com/trofimovelijah/System-Design/blob/main/SD_43_2.png)

# Третий уровень: Обработка больших объемов данных и компонентов ML
## Функциональные требования
1. **Сбор данных о событиях**:
   - организация сбора данных о действиях пользователей и телеметрию в реальном времени
2. **Хранение данных**:
   - данные должны надёжно храниться для последующей обработки
3. **Обработка данных**:
   - данные должны быть извлечены, трансформированы и импортированы в хранилище для проведения аналитики
4. **Аналитика и отчётность**:
   - возможность выполнения запросов для проведения аналитики и формирование отчётности
5. **Механизм рекомендаций для сортировки анкет пользователей**:
   - сортировка отправляемых пользователям непросмотренных анкет, при которой увеличивается вероятность совпадений 
6. **Отправка Push-уведомлений для привлечения пользователей**:
   - рекомендации анкет или сервис промо-пушей, стимулирующих пользовательскую активность.

## Нефункциональные требования
1. **Масштабируемость** для обеспечения высокой нагрузки: 
   - поддержка ±100 миллионов MAU (ежемесячно активных пользователей) и ±1 миллион events в секунду.
2. **Доступность**: 
   - требование не меняется
3. **Производительность**:
   - низкая задержка при обработке данных
4. **Надёжность**:
   - надёжное хранение и передача пользовательских данных
5. **Устойчивость к разделению**:
   - пользователи из разных географических регионов

## Расчёт нагрузки
### Запросы
   - 1M событий в секунду = 90 миллиардов событий в сутки на запись.
### Хранение данных
   - Предположим, что средний размер события составляет порядка 1кБ. Таким образом, суточная нагрузка составит порядка 90 Тб.
   - 90 Тб/сутки * 366 дней = 33 Пб (годовой трафик).

## Компоненты и взаимодействия
1. Обработчик потока событий ETL service:
   - принимает и обрабатывает события в режиме реального времени,
   - сохраняет обработанные данные в хранилище данных для аналитики,
   - трансформирует данные (очистка, агрегация).
2. Сервис хранения больших данных Data Storage service:
   - хранит сырые данные,
   - хранит обработанные данные.
3. Сервис аналитики Analytics service:
   - выполняет аналитические запросы на основе обработанных данных,
   - формирование отчётов.
4. Сервис рекомендаций Recommendation service:
   - использует модели ML для ранжирования и сортировки анкет пользователей,
   - извлекает функции из хранилища функций и обрабатывает рекомендации в режиме реального времени.
5. Сервис сортировки анкет Sorting service:
   - сортирует анкеты по вероятности совпадения.
6. Сервис Push-уведомлений Push Active service (не смешивать с Notification service):
   - использует модели ML для прогнозирования и своевременной отправки push-уведомлений,
   - отслеживает активность пользователей и запускает уведомления о привлечении,
   - в отличие от Notification service сервис Push Active service не отвечает за обычные уведомления (например, если кто-то поставил лайк, то оповещение об этом является сферой ответственности Notification service).

## Высокоуровневый дизайн
![изображение](https://github.com/trofimovelijah/System-Design/assets/15788014/589ce9d5-72eb-4863-b8fd-8ffabf28b47a)

## Хранилища и базы данных
1. Обработка событий ETL service использует для сбора и обработки данных брокер сообщений Kafka, а далее данных размещаются в Hadoop.
2. У сервиса Data Storage service используются два типа данных:
   - хранилище Apache Hadoop HDFS предназначено для размещения большого объёма сырых данных,
   - обработанные данные хранятся в OLAP-базе ClickHouse с поддержкой высокопроизводительных аналитических запросов.
3. Далее данные OLAP ClickHouse применяются в сервисе аналитики.
4. Поскольку необходима обработка данных в режиме реального времени, то целесообразно большие данные в Hadoop только хранить, а непосредственно обработку применять не через хадуповский MapReduce, а через Apache Spark. Это ускорит вычисления для используемых трёх ML-сервисов.
5. Сервис рекомендаций Recommendation service требует быстрый доступ к данным, для чего можно использовать Redis, данные в котором будут содержать ссылки на соответствующие анкеты.
6. Sorting service использует для сортировки и ранжирования в режиме реального времени опять же Redis.
7. Push Active service аналогичен ранее рассмотренному Notification service, поэтому также применим колоночную БД Cassandra для хранения данных о различных типах уведомлений.

## Компонентный дизайн
1. По сути ETL service и Data Storage service можно объединить в один, поскольку сбор, обработка и хранение данных зиждятся на Hadoop. Но для разделения картины по функциям оставим как есть. 
2. Сбор данных о событиях через Apache Kafka. В системе используется отдельный экземпляр брокера сообщений, дабы не смешивать задачи по ML с функционированием остальной системы (из первых двух уровней), для которой развёрнут отдельная шина.
3. Под вопросом использования в ETL service конвейеров для обработки и хранения данных, например, Greenplum, поскольку уже имеется связка HDFS+Kafka.
4. Для сервиса Push Active service схема фактически дублирует аналогичную инфраструктуру сервиса Notification service.
5. Сервис аналитических отчётов должен отображать данные на выстроенной пользовательской стороне:
   - присутствует BI-построитель, наподобие DataLens,
   - предоставляется доступ через Analytics service к хранилищу данных для дата-аналитиков - через Jupyter Notebook.
6. Apache Spark получает данные их брокера сообщений через Streaming, кладёт их в Hadoop, а затем уже задания запускаются как отдельные job'ы, что позволяет масштабировать схему вычислений.
7. Данные, полученные из Spark Jobs предоставляются сервису рекомендаций, который "быстрые" результаты хранит в Redis.

![изображение](https://github.com/trofimovelijah/System-Design/assets/15788014/33c71478-8d89-49b2-94da-790dad58dea3)

## Масштабирование системы и повышение надёжности

# Инфраструктура сбора и хранения данных, логов и телеметрии

# Требования по безопасности
Спроектировать подсистемы, связанные с безопасностью: управление ключами шифрования, идентификация юзеров, защита данных при хранении и передаче, хранение и проверка прав на доступ и т.д.

## Оценки и соображения
### Оценки нагрузки и трафика
1. 100M MAU и 10M DAU:
   - Учет пиковой нагрузки при тысячах запросов в секунду.
   - Высокая пропускная способность баз данных и систем хранения данных для чтения и записи.
### Оценка затрат
1. Затраты на трафик: Оценка на основе передачи данных и использования CDN.
2. Затраты на хранение: Для реляционных, NoSQL и мультимедийных хранилищ.
3. Вычислительные затраты: Для запуска серверных служб, моделей ML и процессов ETL.
### Reliability and Responsiveness Надежность и оперативность
1. Избыточность: Развертывание в нескольких регионах для аварийного восстановления.
2. Масштабируемость: Автоматическое масштабирование групп и бессерверные функции для обработки изменений нагрузки.
3. Мониторинг и оповещения: Используйте такие инструменты, как Prometheus и Grafana, для мониторинга и оповещения в режиме реального времени.
### Соображения безопасности
1. Шифрование данных: Шифруйте данные в состоянии покоя и при передаче.
2. Аутентификация и авторизация: Реализуйте надежную аутентификацию (например, OAuth2) и управление доступом на основе ролей.
3. Соответствие требованиям: Обеспечьте соблюдение правил защиты данных (например, GDPR, CCPA).

### Мониторинг и взаимодействие системы

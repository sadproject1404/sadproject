# 🎓 پروژه درس تحلیل و طراحی نرم‌افزار پیشرفته

**موضوع:** پلتفرم مدیریت هوشمند دانشگاه
**استاد:** دکتر فیضی
**مدت اجرا:** ۸ هفته
**تیم:** ۸ نفر + هوش مصنوعی (ChatGPT)

---

## 🔥 ۱. چشم‌انداز پروژه
ما می‌خواهیم یک سیستم واقعی و کاربردی بسازیم که شبیه پلتفرم‌های دانشگاهی بزرگ باشد. هدف اصلی ما یادگیری معماری میکروسرویس و الگوهای پیشرفته مثل Saga و Circuit Breaker است.

می‌خواهیم بدانیم چطور شرکت‌های بزرگ سیستم‌های مقیاس‌پذیر می‌سازند و چگونه با چالش‌های واقعی مثل مدیریت خطا و داده‌های توزیع شده روبرو می‌شوند.

برای ما کیفیت و یادگیری عمیق مهم‌تر از تکمیل سریع پروژه است. می‌خواهیم در پایان بتوانیم با اطمینان بگوییم که از پس طراحی و پیاده‌سازی یک سیستم Enterprise-Level برمی‌آییم.

این پروژه برای ما مثل یک دوره عملی آماده‌سازی برای بازار کار است.

---

## 🚨 ۲. الزامات کلیدی 

```mermaid
graph TD
    A[الزامات کلیدی پروژه] --> B[معماری میکروسرویس]
    A --> C[ Saga در خرید الگوی]
    A --> D[ Circuit Breaker در آزمون الگوی]
    A --> E[با Event-driven ارتباط RabbitMQ ]
    A --> F[ API Gateway ورود از طریق]
    style A fill:#ff6b6b,stroke:#333,stroke-width:1.5
```

✔ Microservices
✔ Saga Pattern
✔ Circuit Breaker
✔ RabbitMQ
✔ API Gateway

---
## ۳. نیازمندی‌ها

<details>
<summary>۳.۱ نیازمندی‌های عملکردی (Functional Requirements)</summary>

| کد     | سرویس          | نیازمندی                              | توضیح                                  |
|--------|----------------|----------------------------------------|----------------------------------------|
| FR-01  | احراز هویت     | ثبت‌نام و ورود                        | با توکن JWT                           |
| FR-02  | احراز هویت     | صدور توکن JWT                         | توکن ورود                             |
| FR-03  | رزرو منابع     | مشاهده منابع (اتاق، کلاس و …)          | لیست موجودی                           |
| FR-04  | رزرو منابع     | رزرو + جلوگیری از رزرو بیش از حد      | قفل توزیع‌شده + چک تداخل             |
| FR-05  | بازارچه         | تعریف محصول توسط فروشنده              | بارگذاری کالا                         |
| FR-06  | بازارچه         | خرید چندمرحله‌ای                      | با الگوی ساگا                         |
| FR-07  | آزمون           | ساخت آزمون توسط استاد                  | سوالات و زمان‌بندی                    |
| FR-08  | آزمون           | شرکت در آزمون + قطع‌کننده مدار         | اعلان شروع آزمون                      |
| FR-09  | اینترنت اشیا    | دریافت داده زنده سنسور                | دما، رطوبت، حضور و …                 |
| FR-10  | اینترنت اشیا    | نقشه زنده شاتل دانشگاه                | موقعیت GPS                            |

</details>

<details>
<summary>۳.۲ نیازمندی‌های غیرعملکردی (Non-Functional Requirements)</summary>

| کد         | عنوان                   | پیامد معماری                                   |
|------------|------------------------|------------------------------------------------|
| NFR-S01    | مقیاس‌پذیری افقی       | سرویس‌ها کاملاً بدون حالت (Stateless)         |
| NFR-MT01   | چندمستأجری             | جداسازی در سطح اسکیما (Schema-per-Tenant)     |
| NFR-P01    | عملکرد بالا             | کش (Cache) + پردازش ناهمزمان (Async)         |
| NFR-SE01   | امنیت                   | توکن JWT + مدیریت دسترسی نقش‌محور (RBAC)      |
| NFR-R01    | تحمل خطا               | الگوی ساگا + قطع‌کننده مدار                  |

</details>

---
## ۴. دیاگرام‌های C4

<details>
<summary>Level 1 – نمای کلی سیستم (System Context)</summary>

```mermaid
flowchart TB
    Student([دانشجو])
    Teacher([استاد])
    Seller([فروشنده])
    Admin([مدیر سیستم])
    APIGateway([دروازه API])
    MainSystem([پلتفرم مدیریت هوشمند دانشگاه])
    MessageBroker([RabbitMQ])
    DBs((دیتابیس‌های سرویس‌ها))
    IoTDevice([سنسورهای اینترنت اشیا<br/>دما / موقعیت])

    Student --> APIGateway
    Teacher --> APIGateway
    Seller --> APIGateway
    Admin --> APIGateway
    APIGateway --> MainSystem
    MainSystem --> MessageBroker
    MainSystem --> DBs
    IoTDevice --> MainSystem
```

</details>

<details>
<summary>Level 2 – دیاگرام کانتینرها (Container Diagram)</summary>

```mermaid
flowchart TB
    Student([دانشجو]) --> APIGateway[[دروازه API]]
    Teacher([استاد]) --> APIGateway
    Seller([فروشنده]) --> APIGateway
    Admin([مدیر سیستم]) --> APIGateway

    subgraph Services [سرویس‌های اصلی سیستم]
        AuthService[[سرویس احراز هویت]]
        BookingService[[سرویس رزرو منابع]]
        MarketplaceService[[سرویس بازارچه]]
        ExamService[[سرویس آزمون]]
        IoTService[[سرویس اینترنت اشیا]]
        MessageBroker[(RabbitMQ)]
    end

    AuthDB[(دیتابیس احراز هویت)]
    BookingDB[(دیتابیس رزرو)]
    MarketDB[(دیتابیس بازارچه)]
    ExamDB[(دیتابیس آزمون)]
    IoTDB[(دیتابیس اینترنت اشیا)]

    APIGateway --> AuthService & BookingService & MarketplaceService & ExamService & IoTService
    AuthService --> AuthDB
    BookingService --> BookingDB
    MarketplaceService --> MarketDB
    ExamService --> ExamDB
    IoTService --> IoTDB

    MarketplaceService <--> MessageBroker
    BookingService <--> MessageBroker
    ExamService <--> MessageBroker
    IoTService <--> MessageBroker
```

</details>

<details>
<summary>Level 3 — سرویس احراز هویت (Auth Service)</summary>

```mermaid
flowchart TB
    subgraph AuthService ["سرویس احراز هویت"]
        Controller[[کنترلر احراز هویت]]
        ServiceLayer[[لایه منطق]]
        UserRepo[(مخزن کاربران)]
        RoleRepo[(مخزن نقش‌ها)]
        JWTProvider[[تولیدکننده توکن JWT]]
        PasswordHasher[[هش‌کننده رمز عبور]]
        RBAC[[مدیریت دسترسی نقش‌محور]]
    end
    AuthDB[(دیتابیس احراز هویت)]

    Controller --> ServiceLayer
    ServiceLayer --> UserRepo & RoleRepo & JWTProvider & PasswordHasher & RBAC
    UserRepo & RoleRepo --> AuthDB
```

</details>

<details>
<summary>Level 3 — سرویس رزرو منابع (Booking Service)</summary>

```mermaid
flowchart TB
    subgraph BookingService ["سرویس رزرو منابع"]
        BookingController[[کنترلر رزرو]]
        ReservationManager[[مدیر رزرو]]
        AvailabilityChecker[[بررسی در دسترس بودن]]
        BookingRepo[(مخزن رزروها)]
        ResourceRepo[(مخزن منابع)]
        LockManager[[مدیر قفل توزیع‌شده]]
        EventPublisher[[انتشاردهنده رویداد]]
        NotificationClient[[کلاینت اعلان]]
    end
    Cache[(کش Redis)]
    BookingDB[(دیتابیس رزرو)]
    MessageBroker[(RabbitMQ)]

    BookingController --> ReservationManager
    ReservationManager --> AvailabilityChecker & LockManager & BookingRepo & EventPublisher & NotificationClient
    AvailabilityChecker --> Cache
    BookingRepo & ResourceRepo --> BookingDB
    EventPublisher --> MessageBroker
```

</details>

<details>
<summary>Level 3 — سرویس بازارچه (Marketplace Service - الگوی ساگا)</summary>

```mermaid
flowchart TB
    subgraph MarketplaceService ["سرویس بازارچه - هماهنگ‌کننده ساگا"]
        MarketplaceController[[کنترلر بازارچه]]
        OrderManager[[مدیر سفارش]]
        SagaOrchestrator[[هماهنگ‌کننده ساگا]]
        InventoryChecker[[بررسی موجودی]]
        PaymentClient[[کلاینت پرداخت]]
        OrderRepo[(مخزن سفارش‌ها)]
        ProductRepo[(مخزن محصولات)]
        EventPublisher[[انتشاردهنده رویداد]]
        EventConsumer[[دریافت‌کننده رویداد]]
        CompensationManager[[مدیر جبران]]
    end

    OrderDB[(دیتابیس سفارش)]
    ProductDB[(دیتابیس محصول)]
    MessageBroker[(RabbitMQ)]
    IdempotencyStore[(ذخیره idempotency)]

    %% جریان اصلی
    MarketplaceController --> OrderManager
    OrderManager --> InventoryChecker
    OrderManager --> PaymentClient
    OrderManager --> SagaOrchestrator
    OrderManager --> OrderRepo
    OrderManager --> IdempotencyStore

    %% بررسی موجودی → مخزن محصولات → دیتابیس
    InventoryChecker --> ProductRepo
    ProductRepo --> ProductDB

    %% ذخیره سفارش → دیتابیس
    OrderRepo --> OrderDB

    %% ساگا و رویدادها
    SagaOrchestrator --> EventPublisher
    EventPublisher --> MessageBroker
    MessageBroker --> EventConsumer
    EventConsumer --> SagaOrchestrator

    %% در صورت خطا → جبران (هر دو مخزن)
    SagaOrchestrator --> CompensationManager
    CompensationManager --> OrderRepo
    CompensationManager --> ProductRepo

    %% استایل برای جبران‌کننده (اختیاری)
    style CompensationManager fill:#ff9999,stroke:#b33
```

</details>

<details>
<summary>Level 3 — سرویس آزمون (Exam Service - قطع‌کننده مدار)</summary>

```mermaid
flowchart TB
    subgraph ExamService ["سرویس آزمون آنلاین"]
        ExamController[[کنترلر آزمون]]
        ExamManager[[مدیر آزمون]]
        ExamScheduler[[زمان‌بندی آزمون]]
        QuestionRepo[(مخزن سوالات)]
        ExamRepo[(مخزن آزمون‌ها)]
        ResultRepo[(مخزن نتایج)]
        NotificationClient[[کلاینت اعلان]]
        CB[[قطع‌کننده مدار]]
        EventPublisher[[انتشاردهنده رویداد]]
        EventConsumer[[دریافت‌کننده رویداد]]
    end
    ExamDB[(دیتابیس آزمون)]
    QuestionDB[(دیتابیس سوالات)]
    ResultDB[(دیتابیس نتایج)]
    MessageBroker[(RabbitMQ)]
    Cache[(کش Redis)]

    %% جریان اصلی
    ExamController --> ExamManager
    ExamManager --> ExamRepo
    ExamManager --> QuestionRepo
    ExamManager --> ResultRepo
    ExamManager --> Cache
    ExamManager --> ExamScheduler

    %% زمان‌بندی به مخزن آزمون وصل شد
    ExamScheduler --> ExamRepo

    %% اعلان با قطع‌کننده مدار
    ExamManager --> NotificationClient
    NotificationClient --> CB

    %% ارتباط رویدادمحور
    ExamManager --> EventPublisher
    EventPublisher --> MessageBroker
    MessageBroker --> EventConsumer
    EventConsumer --> ExamManager

    %% اتصال مخزن‌ها به دیتابیس
    ExamRepo --> ExamDB
    QuestionRepo --> QuestionDB
    ResultRepo --> ResultDB
```

</details>

<details>
<summary>Level 3 — سرویس اینترنت اشیا (IoT Service)</summary>

```mermaid
flowchart TB
    subgraph IoTService ["سرویس اینترنت اشیا - داده‌های زنده"]
        IoTController[[کنترلر اینترنت اشیا]]
        IoTIngestor[[دریافت‌کننده داده]]
        IoTProcessor[[پردازشگر داده]]
        LocationTracker[[ردیاب شاتل]]
        DashboardService[[به‌روزرسان داشبورد]]
        EventPublisher[[انتشاردهنده رویداد]]
        EventConsumer[[دریافت‌کننده رویداد]]
    end
    IoTDB[(دیتابیس اینترنت اشیا)]
    Cache[(کش Redis)]
    MessageBroker[(RabbitMQ)]

    IoTController --> IoTIngestor
    IoTIngestor --> IoTProcessor
    IoTProcessor --> LocationTracker & DashboardService & IoTDB & Cache & EventPublisher
    EventPublisher --> MessageBroker
    MessageBroker --> EventConsumer
```

</details>

---
## ۵. تصمیم‌گیری‌های معماری (<span dir="ltr">Architecture Decision Records</span> - <span dir="ltr">ADR</span>)

<details>
<summary>مشاهده فهرست کامل تصمیم‌گیری‌های معماری (<span dir="ltr">ADR</span>)</summary>

✔ <span dir="ltr">**ADR-001**</span> — انتخاب معماری <span dir="ltr">Microservices</span>  
✔ <span dir="ltr">**ADR-002**</span> — استفاده از احراز هویت مبتنی بر <span dir="ltr">JWT</span>  
✔ <span dir="ltr">**ADR-003**</span> — استفاده از <span dir="ltr">API Gateway</span>  
✔ <span dir="ltr">**ADR-004**</span> — ارتباط رویدادمحور با <span dir="ltr">RabbitMQ</span>  
✔ <span dir="ltr">**ADR-005**</span> — پیاده‌سازی الگوی <span dir="ltr">Saga</span> در فرآیند خرید  
✔ <span dir="ltr">**ADR-006**</span> — استفاده از <span dir="ltr">Circuit Breaker</span> در سرویس آزمون  
✔ <span dir="ltr">**ADR-007**</span> — انتخاب <span dir="ltr">Redis</span> برای کش و قفل توزیع‌شده  
✔ <span dir="ltr">**ADR-008**</span> — استفاده از <span dir="ltr">Database-per-Service</span>  
✔ <span dir="ltr">**ADR-009**</span> — پیاده‌سازی چندمستأجری با <span dir="ltr">Schema-per-Tenant</span>  

</details>



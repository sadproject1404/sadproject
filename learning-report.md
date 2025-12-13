Phase 3 Report
Smart University System
پیادهسازیSaga Pattern
وCircuit Breaker
:
در فاز سوم پروژهSmart University، تمرکز اصلی برمدیریت تراکنشهای توزیعشدهوافزایش تابآوری سیستم در برابر
خرابی سرویسهابوده است.
با توجه به معماری مبتنی برMicroservicesکه در فازهای قبل طراحی و پیادهسازی شده بود، نیاز به الگوهایی برای هماهنگی
بین سرویسها و جلوگیری از شکست زنجیرهای(Cascading Failure)احساس میشد.
در این فاز دو الگوی معماری مهم پیادهسازی شدند:
Saga Patternبرایمدیریت فرآیند خرید درMarketplace
Circuit Breaker Patternبرای ارتباط بینExam ServiceوNotification Service
اهداف اصلی این فاز به شرح زیر بودند:
1.پیادهسازی یک فرآیند تراکنشی توزیعشده بدون استفاده ازTwo-Phase Commit
2.هماهنگسازی چند سرویس مستقل در فرآیند خرید(Order + Payment)
3.جلوگیری از از کار افتادن کل سیستم در صورتDownشدن یک سرویس
4.افزایشFault Toleranceبا استفاده ازResilience4j
5.نمایش عملی مفاهیم معماریMicroservicesدر سطح واقعی
پیادهسازیSaga Pattern (Marketplace)
برای پیادهسازیSaga Pattern، سناریوی خرید محصول درMarketplaceانتخاب شد.
این سناریو شامل چند سرویس مستقل است که هرکدام دیتابیس و منطق خود را دارند:
Marketplace Service
Payment Service
Rabbit MQبه عنوانMessage Broker
معماری:Saga
نوعSagaاستفادهشده در این پروژه،Choreography-based Sagaاست.
در این مدل، هیچCoordinatorمرکزی وجود ندارد و سرویسها از طریقEventبا یکدیگر ارتباط برقرار میکنند.
مراحل:Saga
1.کاربریک سفارش(Order)ثبت میکند.
2.Marketplace Service
سفارش را با وضعیتPENDINGذخیره میکند.
رویدادOrder Created Eventرا رویRabbitMQمنتشرمیکند.
3.Payment service
رویداد را دریافت میکند.
پرداخت را بهصورت شبیهسازیشده انجام میدهد.
یکی از رویدادهای زیر را منتشر میکند:
Payment Succeeded Event
Payment Failed Event
4.Market Place Service
در صورت موفقیت پرداخت→وضعیتسفارشCOMPLETED
در صورت شکست پرداخت→وضعیتسفارشCANCELED
اجزا اصلیSaga:
1.Marketplace Service
وظایف اصلی:
مدیریتProductوOrder
شروعSaga
واکنش به رویدادهای پرداخت
اجزای پیادهسازیشده:
Order Service
Product Service
Saga Event Handler
برای : Entity های Order و Product
برای :REST API
Products/
Orders/
2.Payment Service
وظایف اصلی:
دریافت رویداد ایجاد سفارش
شبیهسازی فرآیند پرداخت
ارسال نتیجه پرداخت
ویژگی ها:
Listenerرویصفorder. created. queue
تولیدرویدادهایPaymentSucceededوPaymentFailed
مستقل ازMarketplace Service
3.RabbitMQ
RabbitMQبهعنوانMessage Brokerبرای انتقالEventهااستفاده شد.
دالیل انتخاب:
کامل بین سرویسها Decoupling
Asynchronous communication
قابلیت گسترشپذیری باال
پیادهسازیCircuit Breaker (Exam & Notification)
در سیستم دانشگاهی، هنگام ایجاد یکExam، نیاز به ارسال اعالن(Notification)وجود دارد.
اگرNotification Serviceدچار اختالل شود، نبایدExam Serviceنیز از کار بیفتد.برای حل این مسئله ازCircuit
Breaker Patternبا استفاده ازResilience4jاستفاده شد.
معماریCircuit Breaker
سرویسهای درگیر:
Exam Service
Notification Service
ارتباط:
Exam Service → REST call → Notification Service
Circuit Breakerروی سمتExam Serviceپیادهسازی شد.
نحوه عملکردCircuit Breaker
1.Exam ServiceیکExamجدید ایجاد میکند.
2.Exam Serviceتالش میکندNotification Serviceرا فراخوانی کند.
3.در صورت بروز خطا:
Circuit Breakerفعال میشود
متدFallbackاجرا میشود.
4.Examهمچنان ایجاد میشود، حتی اگرNotificationارسال نشود.
مزایای پیادهسازی:
جلوگیری ازCascading Failure
افزایش پایداری سیستم
حفظ تجربه کاربری حتی در شرایط خرابی
رابط کاربری(UI Integration)
برای نمایش عملی نتایج فاز۳، رابط کاربری موجود درAPI Gatewayتوسعه داده شد.
بخشهای اضافهشده:
Marketplace
oنمایشمحصوالت
oایجاد محصول
oخرید محصول(شروعsaga)
Exam
oایجادExam
oنمایش لیستExams
UIاز طریقAPI Gatewayبه سرویسها متصل شده و تمامی درخواستها باJWTاحراز هویت میشوند.
دستاوردهای آموزشی(Learning Outcomes)
تفاوتSagaوTwo-Phase Commit
طراحیEvent-driven Architecture
پیادهسازیChoreography-based Saga
استفاده عملی ازMessage Broker
مفهومCircuit BreakerوFallback
افزایشFault Toleranceدر سیستمهای توزیعشده
گزارش سرویسInventoryمدیریت موجودی
۱.معرفی سرویسInventory
سرویسInventoryیکی از سرویسهای کلیدی در معماری میکروسرویسی پروژه است که مسئولیت مدیریت موجودی
محصوالت و اجرای بخش اولیه عملیات در جریانSagaرا بر عهده دارد. این سرویس تعیین میکند که یک سفارش قابل پردازش
هست یا خیر، و نتیجه را از طریق رویدادها به سایر سرویسها اعالم میکند.
Inventory Serviceنقشی حساس در تضمین عدمOverbooking، کنترل صحیح موجودی و اجرای عملیات جبرانی در
شرایط شکست دارد.
۲.مسئولیتهای اصلی سرویسInventory
رزرو موجودی(Reserve Stock)
وقتی رویدادOrderPlacedتوسطMarketplaceمنتشر میشود، سرویسInventoryآن را دریافت و موجودی را بررسی
میکند:
اگر موجودی کافی باشد مقدار موجودی کم شده ورویدادInventoryReservedمنتشر میشود.
اگر موجودی کافی نباشد رویدادInventoryFailedارسال میشود و جریانSagaهمانجا متوقف میگردد.
آزادسازی موجودی(Compensation / Release Stock)
در شرایطی که پرداخت ناموفق باشد یا هر بخشی ازSagaشکست بخورد، سرویسInventoryباید:
موجودی قبال ً رزروشده را آزاد کند
وضعیت را به حالت پایدار بازگرداند
این عملیات در پاسخ به رویدادهای جبرانی انجام میشود.
جلوگیری ازOverbooking
حتی اگر چند سفارش همزمان برای یک محصول ثبت شود،Inventoryبامدیریت موجودی به صورتatomicاز کم شدن
اشتباهی(Overselling)جلوگیری میکند.
نگهداری موجودی محصوالت
در این پروژه با توجه به نیاز ساده، موجودیها در یکMap in-memoryنگهداری میشوند.
۳.معماری داخلی سرویسInventory
۳.۱. Listenerبرای رویدادOrderPlaced
اینListenerوظیفه دارد رویدادOrderPlacedرا ازRabbitMQدریافت کرده و فرآیند بررسی موجودی را آغاز کند.
۳.۲.منطق رزرو موجودی
منطق رزرو شامل مراحل زیر است:
پیدا کردن محصول
بررسی موجودی
کاهش موجودی در صورت کافی بودن
ارسال رویداد مناسب به سایر سرویسها
۳.۳. Event Publisher
در صورت موفقیت یا شکست، یکی از دو رویداد زیر ارسال میشود:
InventoryReserved
InventoryFailed
۳.۴. Listenerعملیات جبرانی(Compensation)
در صورتی که سرویس پرداخت شکست بخورد،PaymentServiceرویدادPaymentFailedرا ارسال میکند.
Inventory Serviceبه این رویداد گوش میدهد و موجودی رزروشده راrollbackمیکند.
۳.۵.ساختار داده موجودیها
به صورت ساده:
Map<String, Integer> stock = new HashMap<>();
مثال:
p1 = 10
p2 = 5
soldout = 0
۴.نقشInventoryدر جریانSaga
Inventoryدومین سرویس در زنجیرهSagaاست.
جریان:
1.Marketplace → OrderPlaced
2.Inventory →بررسی موجودی
3.Inventory →ارسالInventoryReservedدر صورت موفقیت
4.Payment →ادامه جریان
5.در صورت شکست پرداخت→ Compensationبرای آزادسازی موجودی
✔ Inventoryدارای عملیات جبرانی است
این سرویس یکی از مهمترین نقاطUndoدر فرآیندSagaاست.
۵.رویدادهای سرویسInventory
۵.۱.ورودیها
OrderPlaced
PaymentFailedبرای جبران
۵.۲.خروجیها
InventoryReserved
InventoryFailed
۵.۳.مثال رویدادInventoryReserved
{
"type": "InventoryReserved",
"orderId": "...",
"productId": "p1",
"quantity": 1
}
۶.ارتباط سرویسInventoryبا سایر سرویسها
سرویس نوع ارتباط توضیح
Marketplace Event Listener دریافت سفارش جدید برای بررسی موجودی
Payment Event Emitter/Listener ارسال نتیجه رزرو / دریافت پیام جبرانی
Notification غیرمستقیم نتیجه نهایی ازPaymentمیآید
۷.سناریوهای تست سرویسInventory
۱.رزرو موفق
ورودی:
productId = p1
quantity = 1
انتظار:
موجودی کم شود
رویدادInventoryReservedارسال شود
۲.موجودی ناکافی
productId = soldout
انتظار:
موجودی تغییر نکند
رویدادInventoryFailedارسال شود
۳.عملیات جبرانی
وقتیPaymentFailedدریافت شود:
موجودیآزاد میشود(rollback)
مقدار رزروشده بهMapاضافه میگردد
۴.جلوگیری ازOverbooking
با۵درخواست همزمان:
موجودی هرگز منفی نمیشود
Reservation atomicاست.
مستند پروژهExam Service + Notification Service + Circuit Breaker (Resilience4j) :
موضوع پروژه
در این پروژه از دو سرویسExam ServiceوNotification Serviceاستفاده شده است. هدف این است که هنگام شروع یک
آزمون،Exam Serviceپیامی بهNotification Serviceارسال کند. در صورتی کهNotification Serviceدر دسترس
نباشدDown)شود )، با استفاده از الگویCircuit Breakerو کتابخانهResilience4jازCrashشدنExam Service
جلوگیری شود تا سیستم پایدار باقی بماند.
این پروژه بهصورت دموو بدون دیتابیس پیادهسازی شده و تمرکز اصلی روی شروع آزمون، ارسال پیام نوتیفیکیشن و مدیریت خطا
وfallbackاست.
۱.معرفی سرویسها
۱.۱notification-service (
وظیفه این سرویس دریافت پیام شروع آزمون و ثبت آن در الگ است.
Endpointها:
POST /notifications/exam-start
اینendpointاطالعات آزمون را دریافت میکند و پیام دریافتی را درlogثبت میکند.
POST /notifications/toggle-fail
اینendpointیک متغیر داخلی به نامfail-modeرا روشن یا خاموش میکند. زمانی کهfail-modeروشن باشد،
endpointمربوط بهexam-startعمدا ً با کد۵00پاسخ میدهد تاCircuit Breakerتست شود.
۱.۲exam-service (
مدلExamبه شکل زیر است:
{
"id": 1,
"courseName": "DS",
"title": "Final Exam",
"startTime": "2025-12-08T12:00:00"
}
در این پروژهExamیک مدل ساده است و دادهها بهصورتin-memoryنگهداری میشوند و دیتابیسی وجود ندارد.
درExam ServiceیکNotificationClientوجود دارد که مسئول ارسال پیام بهNotification Serviceاست. روی متد
ارسال پیام،Circuit Breakerاعمال شده است.
Endpointها:
POST /exams
اینendpointیکExamمیسازد و آن را در حافظه ذخیره میکند.
POST /exams/{id}/start
اینendpointآزمون راstartمیکند و سپسNotification Serviceرا صدا میزند.
۲. Circuit Breaker (Resilience4j)
درNotificationClientازannotationزیر استفاده شده است:
@CircuitBreaker(
name = "notificationServiceCB",
fallbackMethod = "notifyExamStartFallback"
)
در صورتی کهNotification Serviceدر دسترس نباشد، متدfallbackاجرا میشود.
در این حالتExam Service crashنمیکند، پیام خطا درlogثبت میشود و آزمون برای ارسال بعدیqueueمیشود.
نمونه الگfallback:
Notification service is down. Fallback executed for exam id=2
Exam id=2 queued for later notification.
۳.تنظیمات پروژه
۳.۱(تنظیماتResilience4j
در فایلexam-service/src/main/resources/application.ymlتنظیمات زیر قرار دارد:
resilience4j:
circuitbreaker:
instances:
notificationServiceCB:
slidingWindowSize: 5
minimumNumberOfCalls: 3
failureRateThreshold: 50
waitDurationInOpenState: 5s
توضیح:
اگر بعد از حداقل۳درخواست، بیش از۵0درصد آنهاfailشوند،Circuit Breakerباز میشود.
Circuit Breakerبه مدت۵ثانیه در حالتOpenباقی میماند و سپس واردHalf-Openمیشود.
۳.۲) Docker Compose
سرویسهای اجراشده درDockerعبارتاند از:
api-gateway (8080)
auth-service (8081)
resource-service (8082)
exam-service (8085)
notification-service (8086)
auth-db (5433)
resource-db (5434)
تمام سرویسها در یکDocker Networkمشترک قرار دارند و باhostnameهمدیگر را پیدا میکنند.
۳.۳API Gateway Routes (
مسیرها بهصورتزیر تعریف شدهاند:
/exams/** → exam-service
/notifications/** → notification-service
۴.اجرای پروژه
برای اجرای پروژه:
cd ~/Desktop/smart-university-phase2
docker compose up --build
برای مشاهده کانتینرهای در حال اجرا:
docker ps
۵.تست پروژه
۵.۱(تست حالت عادیNotification)روشن)
ساخت آزمون:
curl -X POST http://localhost:8080/exams
-H "Content-Type: application/json"
-d '{"courseName":"DS","title":"Final Exam","startTime":"2025-12-08T12:00:00"}'
نتیجه مورد انتظار:
پاسخ۲00دریافت میشود، الگ ارسال نوتیفیکیشن درexam-serviceثبت میشود و درnotification-serviceپیام
“Exam is starting”دیده میشود.
۵.۲تست) Circuit BreakerNotificationخاموش
روشن کردنfail-mode:
curl -X POST http://localhost:8086/notifications/toggle-fail
چند بارstartآزمون:
curl -X POST http://localhost:8085/exams/1/start
curl -X POST http://localhost:8085/exams/1/start
curl -X POST http://localhost:8085/exams/1/start
در این حالتNotificationکد۵00برمیگرداند،Circuit Breakerفعال میشود،Exam Service crashنمیکند و
پاسخها همچنان۲00هستند.
۵.۳)بازگشتCircuit Breakerبه حالتClosed
خاموش کردنfail-mode:
curl -X POST http://localhost:8086/notifications/toggle-fail
بعداز چند درخواست موفق،Circuit BreakerازOpenیاHalf-OpenبهClosedبرمیگردد وارتباط سرویسها به حالت
عادی بازمیگردد.
۶.نتیجه نهایی
endpointهایcreateوstartآزمون درexam-serviceپیادهسازی شدهاند.
notification-serviceپیام شروع آزمون را دریافت میکند.
ارتباط بین دو سرویس برقرار است.
Circuit BreakerباResilience4jبهدرستی فعال شده است.
در صورتDownشدنNotification Service،Exam Service crashنمیکند.
پس از بازگشتNotification Service، سیستم به حالت عادی برمیگردد.
۷.نکات و خطاهای رایج (حلشده)
هشدارobsolete versionدرdocker-composeاهمیتی ندارد.
برای حل مشکلpermissionداکر:
sudo usermod -aG docker $USER
newgrp docker
برای خطایrelease 21 not supportedباید ازbase imageمناسبJava 21درDockerfileاستفاده شود.
همچنین نسخهResilience4jباید بهدرستی درpom.xmlمشخص شده باشد.
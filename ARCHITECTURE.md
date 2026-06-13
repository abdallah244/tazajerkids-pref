# 🏗️ الهيكل البرمجي وبنية النظام التقنية — TazakerKids Architecture

[![إصدار الهيكل](https://img.shields.io/badge/%D8%A5%D8%B5%D8%AF%D8%A7%D8%B1_%D8%A7%D9%84%D9%87%D9%8A%D9%83%D9%84-v1.0.0-blue?style=for-the-badge)](file:///c:/Users/hp/Downloads/tazaker%20kids%20ubdate/ARCHITECTURE.md)
[![حقوق الملكية](https://img.shields.io/badge/%D8%AD%D9%82%D9%85%D9%88%D9%82_%D8%A7%D9%84%D9%85%D9%84%D9%83%D9%8A%D8%A9-abdallah_hany-brightgreen?style=for-the-badge)](file:///c:/Users/hp/Downloads/tazaker%20kids%20ubdate/ARCHITECTURE.md)

يوضح هذا المستند التصميم المعماري البرمجي (Software Architecture) المتكامل لمنصة **تذاكر كيدز (TazakerKids)**. يهدف التصميم إلى توفير بنية تحتية برمجية قابلة للتوسع والتحديث، مع ضمان أمان العمليات وسرعة حجز الفعاليات الترفيهية.

---

## 🛠️ التقنيات المستخدمة في البنية (Technology Stack)

| الطبقة البرمجية (Layer) | التقنية المقترحة (Technology) | الغرض التشغيلي (Purpose) |
| :--- | :--- | :--- |
| **📱 تطبيق الهاتف المحمول** | **Flutter / Dart** | كود موحد يدير واجهات المستخدم على نظامي Android و iOS بكفاءة وسرعة عالية. |
| **🎨 واجهة الويب (الموقع)** | **HTML5 / CSS3 / Modern Vanilla JS** | واجهة مستخدم متجاوبة تماماً مع الجوال والأجهزة اللوحية لتصفح الفعاليات. |
| **⚙️ الخادم والمنطق (Backend)** | **Node.js (Express) / Python (FastAPI)** | معالجة الطلبات وإدارة واجهات API، والمصادقة الآمنة عبر الـ JWT. |
| **🗄️ قاعدة البيانات (Database)** | **PostgreSQL / MySQL** | قاعدة بيانات علائقية لضمان موثوقية الحجوزات والمعاملات المالية وتفادي تضارب التذاكر. |
| **🔔 مركز التنبيهات الموحد** | **Meta WhatsApp Cloud API / Firebase FCM / NodeMailer** | إرسال رسائل الواتساب والبريد والتنبيهات الفورية على شاشات الهواتف تلقائياً. |

---

## 📡 1. مخطط تدفق البيانات للمنظومة (Unified System Architecture)

يوضح المخطط التالي كيفية ترابط مكونات النظام البرمجية من خلال واجهات التطبيقات (RESTful APIs):

```mermaid
graph TD
    %% Define Styles
    classDef client fill:#E3F2FD,stroke:#2196F3,stroke-width:2px,color:#333;
    classDef gateway fill:#FFF3E0,stroke:#FF9800,stroke-width:2px,color:#333;
    classDef core fill:#F3E5F5,stroke:#9C27B0,stroke-width:2.5px,color:#333;
    classDef service fill:#E8F5E9,stroke:#4CAF50,stroke-width:2px,color:#333;

    %% Nodes
    A[📱 تطبيق الجوال Flutter]:::client
    B[💻 موقع الويب للمستخدمين]:::client
    C[🖥️ لوحة تحكم المشرفين Admin]:::client
    
    subgraph backend [بيئة الخادم الأساسية]
        GW[بوابة واجهات التطبيق API Gateway]:::gateway
        AUTH[نظام التحقق والمصادقة Auth & JWT]:::core
        CRM[محرك نظام إدارة علاقات العملاء CRM Server]:::core
        BOOK[محرك الحجوزات والفعاليات Booking Engine]:::core
    end
    
    subgraph storage [طبقة حفظ البيانات والخدمات]
        DB[(قاعدة بيانات PostgreSQL)]:::service
        FCM[Firebase Cloud Messaging]:::service
        WAPI[WhatsApp Cloud API]:::service
    end

    %% Connections
    A & B & C -->|طلبات HTTPS| GW
    GW --> AUTH
    AUTH --> BOOK & CRM
    BOOK & CRM <-->|استعلام وحفظ| DB
    CRM -->|أوامر الإرسال والتنبيهات| WAPI & FCM
```

---

## 🔁 2. تفاصيل تدفق عملية الحجز الفورية (Booking Data Flow)

يسلط المخطط التالي الضوء على مسار الطلب البرمجي منذ قيام المستخدم باختيار تذكرة وحتى تأكيد الحجز وإرسال التنبيهات عبر الواتساب:

```mermaid
sequenceDiagram
    autonumber
    actor User as المستخدم (الموقع/التطبيق)
    participant API as خادم التطوير (Backend API)
    participant DB as قاعدة البيانات
    participant WAPI as WhatsApp Cloud API
    
    User->>API: اختيار الفعالية وطلب حجز التذكرة
    activate API
    API->>DB: التحقق من المقاعد المتاحة (SELECT tickets_left)
    activate DB
    DB-->>API: المقاعد متوفرة
    deactivate DB
    API->>DB: خصم التذكرة مؤقتاً وتوليد رقم المعاملة
    User->>API: إتمام الدفع (بوابة الدفع الإلكتروني)
    API->>DB: تحديث حالة الحجز (CONFIRMED) وتوليد كود الـ QR
    API->>WAPI: طلب إرسال رسالة تأكيد الحجز التلقائية
    activate WAPI
    WAPI-->>User: رسالة واتساب فوري: "تأكيد الحجز وكود الـ QR 🎟️"
    deactivate WAPI
    API-->>User: إظهار شاشة نجاح الحجز والتذكرة
    deactivate API
```

---

## 🗄️ 3. مخطط هيكل قاعدة البيانات المبدئي (Entity Relationship Concept)

لضمان ربط منطقي سليم دون تكرار أو تعارض في البيانات، يعتمد النظام على الجداول الأساسية التالية:

```mermaid
erDiagram
    USERS ||--o{ BOOKINGS : makes
    EVENTS ||--o{ BOOKINGS : contains
    BOOKINGS ||--|| TICKETS : issues
    USERS ||--o{ LOYALTY_POINTS : earns

    USERS {
        int id PK
        string name "اسم المستخدم"
        string phone "رقم الهاتف للمصادقة"
        string email "البريد الإلكتروني للفواتير"
        string city "المدينة للاستهداف الجغرافي"
        datetime created_at
    }

    EVENTS {
        int id PK
        string title "عنوان الفعالية الترفيهية"
        string description "التفاصيل والوصف"
        datetime event_date "تاريخ الفعالية"
        int total_capacity "العدد الأقصى للحضور"
        int tickets_sold "التذاكر المباعة"
    }

    BOOKINGS {
        int id PK
        int user_id FK "رقم المستخدم"
        int event_id FK "رقم الفعالية"
        string status "حالة الحجز (مؤكد/ملغي)"
        float total_amount "القيمة الإجمالية"
        datetime booking_date
    }

    TICKETS {
        int id PK
        int booking_id FK "رقم الحجز"
        string qr_code "رابط أو كود QR للدخول"
        string ticket_serial "الرقم التسلسلي الفرعي للتحقق"
    }

    LOYALTY_POINTS {
        int id PK
        int user_id FK "رقم المستخدم"
        int points "عدد النقاط المتوفرة"
        datetime last_updated
    }
```

---

## 🧠 ميزات تصميم النظام للأمان والاستمرارية (Architectural Highlights)

> [!TIP]
> * **فصل الصلاحيات (Separation of Concerns):** فصل لوحة تحكم المشرفين عن واجهات التطبيق لضمان استقرار الموقع والتطبيق حتى لو خضعت لوحة الإدارة لصيانة دورية.
> * **سرعة استجابة قواعد البيانات:** استخدام الفهرسة (Indexing) على حقول أرقام الهواتف والتواريخ لضمان استرجاع قائمة الفعاليات بشكل سريع وتفادي تأخير استجابة الخادم.
> * **التكامل السحابي الآمن:** تمرير كافة البيانات المالية والطلبات عبر بروتوكولات مشفرة بالكامل لحماية سرية وحقوق المستخدمين.

---

<p align="center">
  <b>جميع الحقوق محفوظة © 2026 abdallah hany</b>
</p>

# MARFAQ | مَرفَق

واجهة Frontend-only تجريبية لمنصة عراقية خيالية لاكتشاف المستشفيات والمرافق الصحية المرتبطة بها.

> **Demo Data / بيانات تجريبية:** جميع السجلات الموجودة هنا خيالية لأغراض تصميم وتجربة الواجهة، ولا تمثل معلومات تشغيلية أو توفرًا فعليًا للخدمات.

## ما الذي تغير في هذه النسخة؟

هذه النسخة أصبحت **Static HTML/CSS/JavaScript** بدون React أو Vite أو Node وقت التشغيل. هذا يجعل `index.html` يعمل مباشرة عند فتحه، كما يجعله مناسبًا جدًا لـ GitHub Pages.

لا يوجد:

- Backend
- API server
- Database server
- Authentication
- Accounts
- Real-time infrastructure
- Embedded map
- Doctor directory
- Clinic marketplace

## 1. التشغيل محليًا

### فتح مباشر

يمكن فتح:

```text
index.html
```

بالنقر المزدوج، وستعمل الواجهة وبيانات الـDemo المضمنة كـfallback.

**ملاحظة:** تحديد الموقع الجغرافي في المتصفح يحتاج عادةً إلى HTTPS أو `localhost`. لذلك لا تعتمد على `file://` لاختبار Geolocation.

### خادم محلي موصى به لاختبار Geolocation

باستخدام Python:

```bash
python -m http.server 4173
```

ثم افتح:

```text
http://localhost:4173/
```

أو باستخدام أي static server آخر.

## 2. تعديل database.json

المصدر الرئيسي للبيانات هو:

```text
data/database.json
```

عند تشغيل الموقع عبر HTTP(S)، تقوم الصفحة بتحميل هذا الملف باستخدام:

```js
fetch('./data/database.json')
```

لذلك يمكنك تعديل JSON ثم تحديث الصفحة.

عند فتح `index.html` مباشرة بواسطة `file://` لا يسمح المتصفح عادةً بتحميل JSON عبر `fetch()` من القرص، ولذلك تستخدم الصفحة نسخة Demo احتياطية مضمّنة داخل `index.html` حتى لا تصبح الصفحة فارغة.

لرؤية تعديلات `database.json` فعليًا، استخدم خادمًا محليًا أو GitHub Pages.

## 3. إضافة مستشفى

أضف كائنًا داخل مصفوفة `hospitals` مع الحقول التالية:

```json
{
  "Hospital ID": "IRQ-BG-XXX",
  "Hospital Name": "اسم المستشفى",
  "Type": "Government Hospital",
  "Governorate": "Baghdad",
  "City": "Baghdad",
  "Area": "Al-Karrada",
  "Address": "العنوان",
  "Phone": "",
  "Maps Link": "",
  "Latitude": null,
  "Longitude": null,
  "Verification Status": "Pending Verification",
  "Last Updated": "2026-10-01",
  "Emergency Department": "",
  "ICU": "",
  "Operating Rooms": "",
  "Anesthesia Services": "",
  "Internal Medicine": "",
  "General Surgery": "",
  "Obstetrics & Gynecology": "",
  "Pediatrics": "",
  "Cardiology": "",
  "Laboratory": "",
  "Blood Transfusion/Blood Bank": "",
  "Pharmacy": ""
}
```

### قيم الخدمات

استخدم فقط:

- `YES`
- `NO`
- `""` عندما تكون المعلومة غير متوفرة

لا يتم اختراع خدمة غير موجودة في البيانات.

## 4. إضافة المرافق الصحية

أضف كائنات داخل:

```text
facilities
```

وتستخدم الحقول:

```text
Facility ID
Facility Name
Type
Governorate
City
Area
Address
Phone
Maps Link
Latitude
Longitude
Verification Status
Last Updated
```

المرافق كيانات ثانوية مرتبطة بالمستشفى، وليست دليل أطباء أو سوق عيادات.

## 5. Latitude / Longitude

الإحداثيات منفصلة تمامًا عن `Maps Link`.

`Latitude` و`Longitude` تستخدم فقط في:

- حساب المسافة الجغرافية التقريبية
- ترتيب الأقرب
- صفحة الأقرب
- ترتيب مستشفيات الطوارئ
- ترتيب المرافق القريبة من المستشفى

إذا كانت الإحداثيات `null` أو غير صالحة:

- لا يتم حساب مسافة
- لا يتم وضع السجل في مركز بغداد
- لا يتم تخمين موقعه
- يمكن أن يظهر في البحث العادي

## 6. Maps Link

`Maps Link` لا يستخدم لحساب المسافة.

عند الضغط على **الاتجاهات**:

1. يستخدم رابط `Maps Link` إن كان موجودًا.
2. وإلا، إذا كانت الإحداثيات صالحة، يتم إنشاء رابط Google Maps Directions من الإحداثيات.
3. Google Maps يفتح خارجيًا في تبويب جديد.

لا توجد خريطة داخل MARFAQ بأي شكل.

## 7. Geolocation

زر **تحديد موقعي** يستخدم:

```js
navigator.geolocation.getCurrentPosition()
```

عند النجاح يتم الاحتفاظ بـ:

```text
userLatitude
userLongitude
```

ضمن حالة الواجهة، ثم يتم حساب Haversine للمستشفيات ذات الإحداثيات الصالحة.

المسافة المعروضة هي:

> **المسافة التقريبية**

وليست مسافة قيادة.

حالات الخطأ المدعومة:

- رفض الإذن
- عدم توفر Geolocation
- timeout
- عدم توفر إحداثيات المستشفى
- استمرار عمل الموقع عند فشل تحديد الموقع

## 8. المفضلة

المفضلة محفوظة محليًا باستخدام:

```text
localStorage
```

والمفتاح:

```text
marfaq:favorites
```

لا تحتاج إلى حساب أو Backend.

## 9. GitHub Pages

المشروع جاهز للنشر كملفات Static مباشرة.

يوجد Workflow في:

```text
.github/workflows/deploy.yml
```

### النشر

1. أنشئ Repository على GitHub.
2. ارفع المشروع إلى الفرع `main`.
3. اذهب إلى:
   `Settings → Pages`
4. اختر GitHub Actions.
5. عند كل push إلى `main` سيقوم workflow بنشر جذر المشروع.

لأن الموقع يستخدم Hash navigation، لا تحتاج إلى rewrite rules للخادم.

## 10. مكونات الواجهة داخل JavaScript

رغم أن النسخة لا تستخدم React، تم تنظيم الواجهة كدوال مكوّنات واضحة، مثل:

```text
hospitalCard()
locationCard()
searchPage()
nearbyPage()
emergencyPage()
profilePage()
favoritesPage()
aboutPage()
```

وهذا يسمح بتطويرها إلى React لاحقًا دون تغيير نموذج البيانات.

## 11. الفحص والجودة

تم التحقق من:

- صحة `data/database.json`
- وجود 8 مستشفيات Demo
- وجود 12 مرفقًا Demo
- صحة JavaScript syntax لملف `src/app.js`
- استخدام مسار JSON نسبي
- فصل Coordinates عن Maps Link
- عدم وجود خريطة أو مكتبة خرائط
- عدم استخدام `file://` داخل الكود
- وجود fallback عند فشل تحميل JSON
- وجود fallback للمشاركة عند عدم دعم `navigator.share()`
- عدم حساب مسافة عند غياب الإحداثيات

لم يتم الادعاء بإجراء اختبار Browser automation كامل؛ بيئة التنفيذ الحالية لم تسمح بإكمال تشغيل Chromium الآلي بصورة مستقرة.

## بنية المشروع

```text
marfaq/
├─ index.html
├─ README.md
├─ data/
│  └─ database.json
├─ src/
│  ├─ app.js
│  └─ styles.css
└─ .github/
   └─ workflows/
      └─ deploy.yml
```

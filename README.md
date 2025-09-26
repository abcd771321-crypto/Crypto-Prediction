# عالم العملات الرقمية — دليل التشغيل والتطوير

هذا المستودع يقدّم تطبيق تحليلات عملات رقمية مبني على Flask مع رسوم بيانية تفاعلية (Plotly) ونماذج تنبؤ متعددة (Linear, RF, SVR, LSTM) بالإضافة إلى مؤشرات فنية. تم تطبيق تحسينات على جودة الكود، الأمان، الأداء، ومعالجة الأخطاء مع الحفاظ على آلية عمل التطبيق.

## محتويات مهمة (روابط إلى الشيفرة)
- نقطة تشغيل التطبيق بعد إعادة الهيكلة: [`crypto_analysis/app.py`](crypto_analysis/app.py:1)
- مصنع التطبيق (App Factory) والتكوين والسجلات: [`create_app()`](crypto_analysis/__init__.py:18)
- تسجيل الـ Blueprints: [`register_blueprints()`](crypto_analysis/routes.py:382)
- الخدمات (تحميل البيانات، التخزين المؤقت، التحديث، المُحوّل): 
  - [`load_crypto_data()`](crypto_analysis/services.py:22)
  - [`get_crypto_data()`](crypto_analysis/services.py:109)
  - [`update_prices_in_place()`](crypto_analysis/services.py:151)
  - [`get_currency_converter()`](crypto_analysis/services.py:119)
- العميل الخارجي مع إعادة المحاولة/المهلة: 
  - تهيئة جلسة HTTP مع Retry: [`CryptoAPIClient.__init__()`](crypto_analysis/models/api_client.py:56)
  - طلب JSON موحّد: [`_get_json()`](crypto_analysis/models/api_client.py:111)
- نماذج التنبؤ والتحسينات:
  - إعداد LSTM والتحقق من طول النافذة: [`PredictionModel._prepare_lstm_data()`](crypto_analysis/models/prediction_models.py:127)
  - تنبؤ LSTM مع تحديث تسلسل آمن: [`PredictionModel.predict()`](crypto_analysis/models/prediction_models.py:315)
  - منطق Ensemble مع تجاوز (fallback) عند الفشل: [`predict_crypto_price()`](crypto_analysis/models/prediction_models.py:343)
- المؤشرات الفنية والرسوم: [`TechnicalIndicators`](crypto_analysis/models/technical_indicators.py:6)
- القوالب (HTML) الأساسية:
  - [`base.html`](crypto_analysis/templates/base.html:1)
  - [`index.html`](crypto_analysis/templates/index.html:1)
  - [`predict.html`](crypto_analysis/templates/predict.html:1)
  - [`convert.html`](crypto_analysis/templates/convert.html:1)
  - [`compare.html`](crypto_analysis/templates/compare.html:1)
  - [`crypto_detail.html`](crypto_analysis/templates/crypto_detail.html:1)
  - [`technical.html`](crypto_analysis/templates/technical.html:1)

## المتطلبات

- Python 3.11+
- بيئة افتراضية مفعّلة (موصى بها)
- تثبيت الاعتمادات:
  ```
  pip install -r crypto_analysis/requirements.txt
  ```

تمت إضافة:
- python-dotenv لتحميل متغيرات البيئة تلقائياً من `.env`.

## الإعداد (Environment)

1) انسخ ملف المثال:
```
cp .env.example .env
```

2) حدّث المتغيرات في `.env`:
- SECRET_KEY: مفتاح جلسة Flask
- FLASK_DEBUG, FLASK_RUN_HOST, FLASK_RUN_PORT
- API_HTTP_TIMEOUT: مهلة طلبات HTTP بالثواني

مثال سريع لمحتوى `.env`:
```
SECRET_KEY=change-me-in-production
FLASK_DEBUG=1
FLASK_RUN_HOST=127.0.0.1
FLASK_RUN_PORT=5000
API_HTTP_TIMEOUT=10
```

## التشغيل

- الخيار 1 (مباشر):
  ```
  python -m crypto_analysis.app
  ```
- الخيار 2 (Flask CLI):
  ```
  python -m flask --app crypto_analysis run
  ```

الخادم سيعمل على: http://127.0.0.1:5000

ملاحظة: بعد إعادة الهيكلة، هذا هو الأسلوب المدعوم للتشغيل. السجل يكتب إلى ملف `crypto_analysis/app.log`.

## بنية التطبيق (بعد إعادة الهيكلة)

- إنشاء التطبيق عبر مصنع التطبيق: [`create_app()`](crypto_analysis/__init__.py:18).
  - إضافة رؤوس أمان أساسية.
  - تهيئة سجلات مع RotatingFileHandler.
  - تسجيل Blueprints عبر [`register_blueprints()`](crypto_analysis/routes.py:382).

- نقاط التوجيه (Blueprints) في [`crypto_analysis/routes.py`](crypto_analysis/routes.py:1):
  - "/" الصفحة الرئيسية (main_bp)
  - "/crypto/<symbol>" تفاصيل العملة (detail_bp)
  - "/technical/<symbol>" التحليل الفني (detail_bp)
  - "/predict" التنبؤ بالأسعار (predict_bp)
  - "/convert" تحويل العملات (convert_bp)
  - "/compare" مقارنة العملات (compare_bp)
  - "/update_prices" تحديت الأسعار (main_bp)

- طبقة الخدمات في [`crypto_analysis/services.py`](crypto_analysis/services.py:1):
  - تحميل البيانات من CSV ودمجها اختيارياً مع بيانات CoinGecko.
  - كائنات كسولة (lazy singletons) للبيانات والمُحوّل.
  - تحديث الأسعار الآمن في الذاكرة.

- طبقة العميل الخارجي في [`crypto_analysis/models/api_client.py`](crypto_analysis/models/api_client.py:1):
  - استخدام جلسة `requests.Session` مع `Retry` و`timeout`.
  - تخزين مؤقت للبيانات (caching) على قرص.

- طبقة النمذجة في [`crypto_analysis/models/prediction_models.py`](crypto_analysis/models/prediction_models.py:1):
  - دعم Linear, RF, SVR, LSTM, وEnsemble.
  - إصلاحات LSTM:
    - التحقق من كفاية البيانات: رفع خطأ واضح عند عدم كفاية بيانات النافذة.
    - تحديث تسلسل LSTM بشكل آمن للحفاظ على الأبعاد.
  - منطق fallback:
    - في حال فشل LSTM أو أي نموذج بسبب البيانات، يستخدم RF ثم Linear (حسب السياق).

- المؤشرات الفنية والرسوم في [`crypto_analysis/models/technical_indicators.py`](crypto_analysis/models/technical_indicators.py:6).

## الميزات

- رسوم تفاعلية لـ:
  - السعر التاريخي، الشموع اليابانية، المتوسطات المتحركة، بولينجر، RSI، MACD، ستوكاستيك.
- التنبؤ بالأسعار مع خيارات نماذج متعددة وتجميع (Ensemble).
- مقارنة العملات (الأداء، التقلب، الارتباط، المخاطر/العوائد).
- تحويل العملات (رقمية/تقليدية).
- وضع ليلي/نهاري.

## التحسينات التي تم تطبيقها

1) إعادة هيكلة وBluePrints
- فصل التوجيهات في ملف مستقل: [`crypto_analysis/routes.py`](crypto_analysis/routes.py:1)
- استخدام مصنع للتطبيق: [`create_app()`](crypto_analysis/__init__.py:18)
- طبقة خدمات قابلة لإعادة الاستخدام: [`crypto_analysis/services.py`](crypto_analysis/services.py:1)

2) الأمان
- نقل SECRET_KEY إلى متغير بيئة: [`create_app()`](crypto_analysis/__init__.py:18)
- رؤوس أمان أساسية: بعد كل استجابة في [`create_app()`](crypto_analysis/__init__.py:18)

3) الأداء/الاستقرار
- جلسة HTTP مع إعادة محاولات ومهلة: [`CryptoAPIClient.__init__()`](crypto_analysis/models/api_client.py:56)
- توحيد طلب JSON: [`_get_json()`](crypto_analysis/models/api_client.py:111)
- التحميل الكسول للبيانات: [`get_crypto_data()`](crypto_analysis/services.py:109)
- تحسين الدمج مع CSV والتحقق من الأعمدة: [`load_crypto_data()`](crypto_analysis/services.py:22)

4) جودة الكود
- تقليل التكرار وفصل المنطق التجاري.
- تحسين سجلات التشغيل وقياس زمن تحميل البيانات.

5) معالجة الأخطاء
- تمرير `fiat_currencies` إلى القالب دائمًا لتفادي أخطاء Jinja في صفحة التنبؤ: [`predict()`](crypto_analysis/routes.py:138)

6) تحسينات نماذج التنبؤ
- LSTM: حماية من انكسار الأبعاد والتحقق من الطول: [`_prepare_lstm_data()`](crypto_analysis/models/prediction_models.py:127), [`predict()`](crypto_analysis/models/prediction_models.py:315)
- Ensemble: تخطي النماذج الفاشلة والعودة None عند الفشل الكامل: [`predict_crypto_price()`](crypto_analysis/models/prediction_models.py:343)
- Linear: تحسين توليد ميزات المستقبل: [`predict()`](crypto_analysis/models/prediction_models.py:283)

ملاحظة حول رسائل التحذير في VSCode:
قد ترى أخطاء CSS/JS وهمية ضمن القوالب بسبب وجود تعليمات Jinja داخل `<script>` أو `style` inline. هذه ليست أخطاء تشغيلية، فالقوالب تُرسم من جهة الخادم وتستبدل القيم قبل وصولها للمتصفح.

## كيفية الاستخدام

1) افتح الصفحة الرئيسية: http://127.0.0.1:5000
2) استعرض العملات، ادخل إلى تفاصيل، التحليل الفني، أو قارن عملات متعددة.
3) استخدم صفحة التنبؤ لاختيار العملة، عدد الأيام، ونوع النموذج.
4) استخدم التحويل لتبديل بين العملات الرقمية والورقية.

## التخصيص

- ضبط مهلة API وإعادة المحاولة عبر:
  - `API_HTTP_TIMEOUT` في `.env`
- توسيع عدد الرموز المدعومة:
  - حدث القاموس: [`CryptoAPIClient.symbol_mapping`](crypto_analysis/models/api_client.py:40)

## الاختبارات (مقترحة)
- يوصى بإضافة وحدات اختبار (pytest) للوظائف الأساسية:
  - API client (شبكة/تخزين مؤقت)
  - Services (تحميل/تحديث)
  - PredictionModel (إعداد بيانات/تنبؤ)
  - TechnicalIndicators (حساب صحيح للمؤشرات)

## خارطة طريق مستقبلية
- إضافة تخزين دائم (PostgreSQL) لبيانات تاريخية موسعة.
- جدولة تحديثات الخلفية (APScheduler/Celery).
- واجهات API JSON منفصلة للاستهلاك البرمجي.
- CI (GitHub Actions) للتنميط والاختبارات.

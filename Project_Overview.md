# نظرة عامة على المشروع وخصائصه

- الهدف: منصة تحليل عملات رقمية مبنية على Flask توفر:
  - عرض أسعار حية ووصف ومعلومات سوقية لكل عملة.
  - رسوم تفاعلية (Plotly) للسعر والمؤشرات الفنية.
  - تنبؤ بالأسعار لعدة أيام قادمة باستخدام نماذج تعلم الآلة.
  - مقارنة متعددة العملات (أداء/تقلب/ارتباط/مخاطر-عوائد).
  - محوّل عملات ورقية/رقمية.

## الخصائص التقنية
- إطار العمل: Flask
- الرسوم: Plotly (ثيم `plotly_white`)
- البيانات التاريخية: ملفات CSV لكل رمز داخل `coin files csv/`
- الأسعار الحية: عبر `CryptoAPIClient` + تخزين مؤقت JSON داخل `crypto_analysis/cache/`
- المؤشرات الفنية: `TechnicalIndicators`
- المقارنة: `CryptoComparison`
- التنبؤ: `prediction_models` (Linear, RF, SVR, LSTM, Ensemble)
- إدارة الأسعار والصرف: `services.py`, `currency_converter.py`

## البنية
- `crypto_analysis/` يشمل النواة (routes, services, models, templates)
- `coin files csv/` للبيانات التاريخية
- `ReadMe files/` توثيق باللغة العربية

## كيفية التشغيل
```bash
python -m crypto_analysis.app
```

## ملاحظات الجودة
- حواجز أمان ضد البيانات الفارغة في الرسوم.
- إزالة الوضع الداكن وتوحيد نمط الواجهة الفاتحة.
- تحديث أسعار حي عند الدخول لصفحات رئيسية/تفاصيل.

## مخطط تدفق البيانات (Mermaid)
```mermaid
flowchart LR
  A[CSV files\ncoin files csv/*.csv] -->|load & clean| B[services.py\nget_crypto_data()]
  B --> C[routes.py]
  C -->|details| D[models/TechnicalIndicators\ncreate_technical_charts]
  C -->|compare| E[models/CryptoComparison\ncompare_*]
  C -->|predict| F[models/prediction_models\npredict_crypto_price]
  C -->|convert| G[models/CurrencyConverter]
  C --> H[templates/*.html\n(Plotly JSON, Jinja)]
  I[API\nCryptoAPIClient] -->|live prices| C
  I -->|cache JSON| J[crypto_analysis/cache/]
```

## مخطط بنية الطبقات (Mermaid)
```mermaid
graph TD
  UI[Templates (Jinja, Plotly)] --> R[Routes]
  R --> S[Services]
  S --> M[Models]
  M -->|files| Data[(CSV)]
  M -->|API| Ext[(External API)]
  R --> Logs[(app.log)]
```

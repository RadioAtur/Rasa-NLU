# راهنمای کامل برای پیاده‌سازی طبقه‌بندی نیت با استفاده از RASA

این فایل شامل تمامی اطلاعات مربوط به پیاده‌سازی و نصب محیط RASA برای طبقه‌بندی نیت در مدل‌های محاوره‌ای AI است.

---

## فهرست مطالب
1. [مقدمه بر طبقه‌بندی نیت در RASA](#intro)
2. [پیش‌نیازهای نصب و راه‌اندازی](#requirements)
3. [دستورات نصب و تنظیم محیط](#commands)
4. [مدل‌های طبقه‌بندی نیت](#models)
5. [پلتفرم RASA و نحوه عملکرد آن](#rasa_platform)

---

<a name="intro"></a>
## 1. مقدمه بر طبقه‌بندی نیت در RASA
طبقه‌بندی نیت (Intent Classification) یک فرآیند کلیدی در درک زبان طبیعی (NLU) است که هدف آن تشخیص نیت اصلی کاربران از ورودی‌های متنی است. این فرآیند با استفاده از مدل‌های یادگیری ماشین به سیستم کمک می‌کند تا پاسخ‌های مناسب را به کاربر ارائه دهد.

---

<a name="requirements"></a>
## 2. پیش‌نیازهای نصب و راه‌اندازی
برای استفاده از RASA و پیاده‌سازی طبقه‌بندی نیت نیاز به نسخه‌های خاصی از Python و برخی از کتابخانه‌های مرتبط است:

- **Python**: نسخه 3.7.x
- **spaCy**: نسخه 2.3.7

---

<a name="commands"></a>
## 3. دستورات نصب و تنظیم محیط
این دستورات را به ترتیب در خط فرمان یا ترمینال وارد کنید:

```bash
# شروع یک محیط pipenv جدید
pipenv shell

# راه‌اندازی pyenv
pyenv init - | source

# فعال‌سازی محیط پایتون با نسخه 3.7
pyenv shell 3.7

# ارتقای pip به آخرین نسخه
pip install --upgrade pip

# نصب بسته‌های مورد نیاز
pip install rasa-nlu spacy==2.3.7 sklearn_crfsuite

# دانلود مدل زبان انگلیسی برای spaCy
python -m spacy download en
```

**توضیحات دستورات:**
- `pipenv shell`: ایجاد و ورود به یک محیط مجازی جدید با استفاده از pipenv.
- `pyenv init - | source`: راه‌اندازی pyenv برای مدیریت نسخه‌های مختلف پایتون.
- `pyenv shell 3.7`: تنظیم نسخه پایتون به 3.7.
- `pip install --upgrade pip`: ارتقای pip برای سازگاری بهتر.
- `pip install ...`: نصب بسته‌های مورد نیاز.
- `python -m spacy download en`: دانلود مدل زبان انگلیسی برای استفاده در RASA.

---

<a name="models"></a>
## 4. مدل‌های طبقه‌بندی نیت

### مدل DIET (Dual Intent and Entity Transformer)
- DIET برای طبقه‌بندی نیت و شناسایی موجودیت‌ها طراحی شده و با استفاده از معماری ترنسفورمر این دو کار را به صورت همزمان انجام می‌دهد.
- DIET با بهره‌گیری از لایه‌های جداگانه برای پیش‌بینی نیت و موجودیت، عملکرد مدل را بهبود می‌بخشد.

### مدل BERT
- BERT برای درک متن به صورت دوطرفه طراحی شده است که در تشخیص نیت‌های پیچیده در مکالمات موثر است.

### مدل‌های مبتنی بر شبکه عصبی پیچشی (CNN)
- CNNها با تشخیص الگوهای محلی و ویژگی‌های خاص در جملات ورودی، به طبقه‌بندی نیت کمک می‌کنند.

---

<a name="rasa_platform"></a>
## 5. پلتفرم RASA و نحوه عملکرد آن
پلتفرم RASA یکی از ابزارهای متن‌باز محبوب برای ساخت چت‌بات‌ها و طبقه‌بندی نیت کاربران است. پیش از DIET، RASA از مدلی مبتنی بر تبدیل داده‌ها به وکتورهای با بعد بالا استفاده می‌کرد که به کمک آن‌ها می‌توانست نزدیکی پیام‌های کاربر به نیت‌های از پیش تعریف‌شده را تشخیص دهد.

# راهنمای طبقه‌بندی نیت در RASA

## مقدمه
این راهنما به بررسی طبقه‌بندی نیت در RASA و نحوه استفاده از آن می‌پردازد. RASA یک چارچوب منبع باز برای ایجاد دستیارهای هوشمند و چت‌بات‌هاست. در اینجا به نکات اصلی و نمونه‌هایی از کد پرداخته شده است.

## ۱. عبارات منظم (Regex)
### الگوهای عبارات منظم
- **zipcode**: `[0-9]{5}`
- **greet**: `hey[^\s]*`
- **job_id**: `job_?[0-9]{4,6}`
- **email**: `[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}`

### مترادف‌های موجود
- **رزومه**:
    - CV
    - curriculum vitae
    - bio-data
    - resumé
- **تمام‌وقت**:
    - full time
    - permanent
    - fulltime
- **نیمه‌وقت**:
    - part time
    - temporary
    - parttime

## ۲. آموزش طبقه‌بند نیت
### کد
```python
# وارد کردن ماژول‌های لازم
from rasa_nlu.training_data import load_data
from rasa_nlu.model import Trainer
from rasa_nlu import config

# بارگذاری داده‌های آموزشی
train_data = load_data('rasa-dataset.json')

# ایجاد Trainer با تنظیمات پیکربندی
trainer = Trainer(config.load("./config_spacy.yaml"))

# آموزش مدل
trainer.train(train_data)

# ذخیره مدل آموزش‌دیده
model_directory = trainer.persist('./projects/')
```

## ۳. طبقه‌بند نیت با استفاده از Scikit-learn
### توضیحات
- **استخراج ویژگی‌ها**: داده‌های متنی به فرمت عددی تبدیل می‌شوند.
- **طبقه‌بند**: `RandomForestClassifier` از Scikit-learn استفاده می‌شود.
- **آموزش**: مدل بر اساس ویژگی‌ها و نیت‌ها آموزش داده می‌شود.
- **پیش‌بینی**: ورودی‌های جدید پردازش شده و نیت پیش‌بینی می‌شود.
- **ذخیره‌سازی مدل**: بعد از آموزش، مدل ذخیره می‌شود.
- **تنظیمات هایپرپارامتر**: گزینه‌هایی برای تنظیم پارامترهای مدل وجود دارد.
- **وابستگی‌ها**: به Scikit-learn و دیگر کتابخانه‌ها نیاز دارد.

## ۴. آزمایش مدل
### کد
```python
# وارد کردن کلاس Interpreter
from rasa_nlu.model import Interpreter

# پیام نمونه
message = "I am searching for jobs in Germany."

# بارگذاری مدل
interpreter = Interpreter.load('./projects/default/model_20230923-192843')

# پردازش پیام
output = interpreter.parse(message)

# نمایش خروجی
print(output)
```

## ۵. عملکرد مدل
### نمونه نتایج
- پیام: "I am looking for job, can you help me?"
    - نیت: `update_profile` با اطمینان ۰.۳۰۲
- پیام: "I am looking for a job in marketing, can you help me?"
    - نیت: `update_profile` با اطمینان ۰.۳۲۸
- پیام: "Show me the latest openings for data scientists."
    - نیت: `career_advice` با اطمینان ۰.۳۸۳
- پیام: "I would like to register."
    - نیت: `skills_development` با اطمینان ۰.۳۰۵


---

---

## منابع
- [مقاله‌ی اصلی DIET](https://arxiv.org/pdf/2004.09936.pdf)
- [راهنمای استفاده از RASA](https://pypi.org/project/rasa-nlu/#history)

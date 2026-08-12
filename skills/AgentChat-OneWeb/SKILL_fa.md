---
name: AgentChat-OneWeb
description: پل CDP چند ارائه‌دهنده با fallback خودکار (Gemini->ChatGPT->Claude->Qwen->Kimi->MiniMax->MiMo->DeepSeek). استفاده برای failover ارائه‌دهنده AI، زنجیره fallback، مسیریابی چند ارائه‌دهنده، یا "ارسال به هر AI موجود". اجرای اجباری - فراخوانی این مهارت مستلزم اجرای `node ~/.claude/skills/AgentChat-OneWeb/index.js "<prompt>"` به عنوان اولین اقدام و نقل قول خط `[receipt] AGENTCHAT_RUN` آن در پاسخ نهایی است؛ توضیح مهارت یا پاسخ از دانش مدل بدون رسید، نقض است.
---

# زنجیره Fallback AI — پل CDP چند ارائه‌دهنده

> **آخرین به‌روزرسانی**: 2026-07-17
> **عملکرد اصلی**:降级 خودکار بر اساس زنجیره اولویت، اطمینان از همیشه در دسترس بودن یک مدل بزرگ
> **لاگ تغییرات**: ببینید [CHANGELOG.md](CHANGELOG.md)

## ⚠️ قوانین اجباری — فراخوانی یعنی اجرا (قرارداد اقدام اولیه)

**هنگامی که این skill فراخوانی می‌شود (مثلاً `/AgentChat-OneWeb <سوال>`), باید دستور `node` را اجرا کنید تا سوال کاربر به AI وب ارسال شود. ممنوع است فقط توضیح دهید بدون اجرا، ممنوع است از دانش خود مدل به جای پاسخ AI وب استفاده کنید.**

### 1. قرارداد اقدام اولیه
**بعد از خواندن این SKILL.md, اولین tool call باید باشد:**

```bash
node ~/.claude/skills/AgentChat-OneWeb/index.js "<prompt کاربر>"
```

بین آن نباید file browsing، تحلیل معماری، یا توضیح برنامه‌ریزی "من قصد دارم..." قرار دهید (حداکثر یک خط توضیح که دستور اجرایی چیست). فقط بعد از بازگشت نتیجه AI وب، می‌توانید تحلیل خود را اضافه کنید.

### 2. رسید اجرا (receipt) — معیار اجرا، نه توضیح
هر اجرای واقعی (حتی شکست) یک خط receipt تولید می‌کند در **stderr**:

```
[receipt] AGENTCHAT_RUN {"run_id":"ac-xxxxxxxxxxxx","skill":"AgentChat-OneWeb","exit":0,"provider_used":"Gemini",...}
```

- **پاسخ نهایی باید عیناً این خط receipt را نقل کند** (حداقل شامل run_id، provider_used، exit).
- بدون receipt = بدون اجرا = نقض، باید برگردید و اجرا کنید.
- `run_id` تصادفی تولید شده و در `~/.claude/skills/AgentChat-OneWeb/data/receipts.jsonl` ذخیره می‌شود، کاربر می‌تواند با `grep <run_id>` بررسی کند — ساختگی قابل بررسی نیست.
- **اجرای شکست‌خورده (exit≠0) هم receipt دارد**: باید receipt شکست را نقل کرده و دلیل را توضیح دهید (rate limit/وارد نشده/timeout...)، بعد از آن مجاز هستید با توانایی خود مدل پاسخ دهید، و باید صریحاً标注 کنید "AI وب در این پاسخ مشارکت نداشت".

### 3. الگوهای نقض (همه ممنوع)
- ❌ بعد از خواندن SKILL.md توضیح زنجیره fallback، فهرست پارامترهای CLI، توضیح معماری — اما `node index.js` اجرا نکنید
- ❌ به بهانه "سوال خیلی ساده است/من قبلاً جواب را می‌دانم" اجرا را رد کنید، مستقیم با دانش مدل پاسخ دهید
- ❌ در پاسخ خط `[receipt] AGENTCHAT_RUN` نباشد اما ادعا کنید "از طریق AI وب پردازش شد"

### 4. استثناها
فقط: `--smoke`، `--doctor`، یا کاربر صریحاً بگوید "فقط محیط را بررسی کن نه ارسال". این دو حالت receipt تولید نمی‌کنند، رفتار مورد انتظار است.

## 📐 استاندارد قالب‌بندی خروجی — فرمت اجباری پاسخ نهایی

موضوع محدودیت **متن نهایی نوشته شده توسط Claude Code** است؛ خروجی خام worker، بلوک‌های کد، diff، جدول و خطوط receipt مستثنی هستند.

1. **نتیجه اول**: پاراگراف اول ≤50 کلمه نتیجه کلیدی (TL;DR)، مستقیم به خواسته اصلی کاربر پاسخ دهد، بدون تعارف، بدون مقدمه روش‌شناسی.
2. **سطح عناوین**: ماژول اصلی با `##`، زیرنظرات با `###`، ممنوع است عنوان سطح یک `#`. ابعاد بر اساس منطق به 3-5 بخش دسته‌بندی شوند (مثل: زمینه/وضعیت فعلی/تحلیل/نتیجه)، ممنوع است به ترتیب جستجو بنویسید.
3. **تمرکز بصری**: داده‌ها، زمان، اسامی خاص، نکات کلیدی با `**پررنگ**` مشخص شوند؛ پررنگ فقط برای هدایت تمرکز، هر پاراگراف طبیعی حداکثر 2 مورد.
4. **انضباط لیست**: فقط لیست نامرتب تک‌سطحی `*` مجاز است، ممنوع است هرگونه لیست تودرتو چندسطحی (برای تجربه خواندن در ترمینال/چت‌باکس).
5. **تراکم متن**: پاراگراف‌های روایی طبیعی ≤3 جمله، شاخه منطقی جدید باید خط جدید و پاراگراف جدید؛ ممنوع است جملات انتقالی بدون اطلاعات واقعی مثل "در مجموع"، "بر اساس نتایج جستجوی فوق"، "به عنوان یک هوش مصنوعی".
6. **جداسازی اطلاعات**: هنگام نقل قطعات متن اصلی AI وب یا لینک‌های خارجی باید در بلوک نقل قول `>` قرار گیرد و منبع provider标注 شود، با تحلیل خودتان به طور سختگیرانه جدا شود.
7. **بندهای معافیت (اولویت بالاتر از بند 1-6)**:
   * خط `[receipt] AGENTCHAT_RUN {...}` (یا فهرست run_id هر مرحله) باید عیناً در انتهای پاسخ در بلوک کد حفظ شود، ممنوع است بازنویسی، پررنگ کردن، حذف — تحت约束强制规则 §2.
   * افشای降级/شکست (مثل "N نقش降级"، "AI وب در این پاسخ مشارکت نداشت") جزو شفافیت فرآیند است، جمله انتقالی اضافی محسوب نمی‌شود، نباید حذف شود.
   * بلوک‌های کد، diff، جدول محدودیت طول پاراگراف و سطح لیست ندارند.

## 🖼️ پروتکل تولید تصویر (Image Generation Protocol)

### شرایط فعال‌سازی

هنگامی که درخواست کاربر شامل هر یک از کلیدواژه‌های زیر باشد، پروتکل تولید تصویر فعال می‌شود:

* **دسته تولید**: تولید تصویر，رسم نمودار،制图，绘制，作图，生成图，create image，generate image،make image
* **دسته نمودار**: فلوچارت，نمودار معماری،نمودار شماتیک，نقشه ذهنی，نمودار،نمودار توپولوژی，chart，diagram،flowchart،mindmap，Mermaid
* **دسته بصری‌سازی**: بصری‌سازی،visualization،illustration،infographic，DALL·E،Imagen，Midjourney

### 1. تقویت خودکار Prompt (اجباری — پرچم `--image` را منتقل کنید، از v14 در index.js داخلی است)

هنگام تشخیص درخواست تولید تصویر، **باید** پرچم `--image` را به دستور اضافه کنید:

```bash
node ~/.claude/skills/AgentChat-OneWeb/index.js --image "<prompt اصلی کاربر>"
```

index.js به صورت داخلی دستور تقویت استاندارد ("لطفاً از مدل/ابزار تولید تصویر خود برای تولید فعالانه استفاده کنید...") را به انتهای prompt اضافه می‌کند و در telemetry ثبت می‌کند `image_prompt_enhanced: true`. **ممنوع است دستی prompt را بازنویسی کنید تا جایگزین `--image` شود** —追加 دستی فقط约束 متنی است، همان چیزی که مکانیزم receipt می‌خواهد حذف کند "انطباق توصیفی"؛ مسیر پرچم ماشین-قابل-بررسی است (telemetry قابل بررسی است).

اگر کاربر قبلاً صریحاً `--from=ChatGPT` (DALL·E) یا `--from=Gemini` (Imagen) مشخص کرده است، اولویت با قابلیت تولید تصویر آن provider است: `--image --from=Gemini`.

### 2. دانلود خودکار تصویر (اجباری — داخلی در index.js)

index.js بعد از دریافت پاسخ از AI وب، **به صورت خودکار** مراحل زیر را اجرا می‌کند:

1. اسکن URL تصویر در متن پاسخ:
   * نحو Markdown: `![alt](url)`
   * تگ HTML: `<img src="url">`
   * لینک مستقیم: URL ختم شده به `.png`/`.jpg`/`.jpeg`/`.gif`/`.webp`/`.svg`
2. دانلود هر تصویر به **دایرکتوری کار جاری** (`process.cwd()`، یعنی دایرکتوری که کاربر skill را در آن اجرا می‌کند)
3. فرمت نام فایل: `ai-image-{YYYYMMDD-HHmmss}-{pid}-{شماره}.{ext}` (v14 pid اضافه کرد، جلوگیری از بازنویسی همزمان workerهای موازی روی نام یکسان در ثانیه یکسان)
4. خلاصه نتیجه دانلود (بخش `## 📥 Downloaded Images`): **وقتی stdout TTY است** (اجرای مستقیم انسان) به انتهای پاسخ附加 می‌شود؛ **وقتی stdout pipe است** (execute.js / Python SDK / MCP server مصرف می‌کنند) خلاصه فقط از stderr می‌رود، stdout "متن اصلی پاسخ AI" را حفظ می‌کند قرارداد ماشین آلوده نشود، شمارش موفقیت/شکست در receipt ثبت می‌شود (`images_ok` / `images_failed`).

**محدودیت سخت v14** (URL دانلود از پاسخ AI وب می‌آید، این ورودی غیرقابل اعتماد است، می‌تواند توسط prompt injection دستکاری شود):
* حداکثر **20 تصویر** در هر پاسخ دانلود می‌شود، مازاد در خلاصه صریحاً به عنوان skipped علامت‌گذاری می‌شود
* حداکثر **30MB** برای هر تصویر؛ بودجه کل مرحله دانلود **120s**؛ پنجره ثبات tier-2 داخل صفحه fetch با AbortSignal 25s (قبلاً timeout نداشت — یک endpoint تصویر معلق می‌توانست باعث شود کل فرآیند هرگز خارج نشود)
* همه tierها **بوی payload** انجام می‌دهند: صفحه خطای HTML بازگشتی HTTP 200 به عنوان شکست دانلود判定 می‌شود، دیگر به عنوان `.png` خراب ذخیره نمی‌شود
* **رد loopback / link-local / آدرس‌های خصوصی RFC1918** (مثل `http://127.0.0.1:9222/...` — URL تزریق شده قبلاً می‌توانست داده endpoint دیباگ CDP را در دایرکتوری کاربر بنویسد)؛ محیط داخلی/تست می‌تواند با `AGENTCHAT_ALLOW_PRIVATE_IMAGE_HOSTS=1` اجازه دهد
* تغییر مسیر نسبی `Location:` و 303 به درستی دنبال می‌شوند؛ هدف تغییر مسیر نیز بررسی blocked-host را پشت سر می‌گذارد

اگر در پاسخ URL تصویری وجود نداشته باشد، این مرحله zero-cost no-op است.

### 3. توضیح دایرکتوری دانلود

هدف دانلود همیشه **دایرکتوری کار جاری کاربر** (`$PWD` shell) است، نه دایرکتوری نصب skill. مثلاً:
* کاربر در `~/Project/` فراخوانی کند `/AgentChat-OneWeb برای من فلوچارت رسم کن` → تصویر در `~/Project/` دانلود می‌شود
* کاربر در `~/Data_Project/` فراخوانی کند → تصویر در `~/Data_Project/` دانلود می‌شود

می‌توان با پرچم `--no-download-images` دانلود خودکار را غیرفعال کرد.

---

## Trigger

از این skill استفاده کنید وقتی:
- کاربر می‌خواهد prompt را به "هر AI موجود" ارسال کند
- سهمیه Gemini معلوم است که تمام شده و fallback نیاز است
- کاربر می‌خواهد failover ارائه‌دهنده خودکار بدون سوئیچ دستی داشته باشد
- اجرای batch prompt که قابلیت اطمینان ارائه‌دهنده فردی مهم است

برای مکالمات تعاملی که نیاز به context چند دور دارند استفاده نکنید (هر ارائه‌دهنده state session مستقل دارد).

**چه زمانی از THIS skill استفاده کنید:**
- چند ارائه‌دهنده با fallback خودکار. استفاده برای قابلیت اطمینان، پردازش batch، یا وقتی مهم نیست کدام AI پاسخ دهد.
- برای عمق استدلال Max خاص Gemini، از `--from=Gemini` استفاده کنید تا Gemini اول در زنجیره مجبور شود.

---

## زنجیره Fallback (ترتیب اولویت)

```
Gemini → ChatGPT → Claude → Qwen → Kimi → MiniMax → MiMo → DeepSeek
(Pro Extended)                                                  (آخرین راه حل)
```

اولین ارائه‌دهنده موجود برنده می‌شود. هر مرحله ONLY در صورت عدم دسترسی تأیید شده (quota/auth/model-degraded) عبور می‌کند، هرگز به خاطر خطاهای شبکه موقت.

---

## تشخیص دسترسی ارائه‌دهنده

هر provider قبل از ارسال prompt از 3 لایه بررسی عبور می‌کند:

| لایه بررسی | محتوای تشخیص | رفتار شکست |
|--------|---------|---------|
| **L1: دسترسی** | آیا صفحه بارگذاری می‌شود، آیا نیاز به ورود است | رد → provider بعدی |
| **L2: قابلیت استفاده** | آیا input box قابل ویرایش است، آیا rate limit شده | رد → provider بعدی |
| **L3: کیفیت مدل** | آیا مدل Pro/پیشرفته موجود است | خاص Gemini، سایر providerها رد می‌شوند |

### پردازش خاص Gemini
Gemini تنها provider در زنجیره است که **Pro Extended Thinking** را الزامی می‌کند.
فعال‌سازی مدل سه لایه降级 دارد:
1. **Pro Extended Thinking** (نیاز به اشتراک Gemini Pro) — اولویت اول
2. **حالت Flash** (fallback سطح رایگان) — وقتی Pro Extended موجود نیست خودکار سوئیچ می‌شود
3. هر دو شکست → `ERR_MODEL_DEGRADED`،降级 به ChatGPT

شرایط触发降级 توسط `quotaPatterns` هر adapter تعریف می‌شود (`lib/providers/adapters/<name>.js`)،
منبع معتبر است. SKILL.md دیگر副本 دوم را نگهداری نمی‌کند (در گذشته ناسازگاری با کد ظاهر شده بود).

---

## پیش‌نیازها

```bash
# 0. ⚠️ حیاتی: باید از Chrome سیستم + profile با login state استفاده کنید
#    کپی و ویرایش فایل .env پروژه:
#      cp .env.example .env
#    پیکربندی کلیدی:
#      CHROMIUM_PATH=/usr/bin/google-chrome-stable  (REQUIRED — باید روی Chrome سیستم تنظیم شود، خالی گذاشتن خطا می‌دهد؛ رد مسیر ms-playwright)
#      CHROME_PROFILE=~/.chrome-debug-profile
#    اگر پیکربندی نشده باشد، login state همه سایت‌های AI گم می‌شود!

# 1. Chrome debug روی پورت 9222 اجرا می‌شود — از v16 کاملاً خودکار، بدون وابستگی اسکریپت خارجی
#    index.js وقتی پورت قابل دسترسی نیست خودکار Chrome راه‌اندازی می‌کند (AGENTCHAT_NO_AUTOSTART=1 خاموش می‌کند):
#      Tier 1: اسکریپت راه‌اندازی پلتفرم (اگر scripts/ deploy شده — سناریو clone کامل مخزن)
#      Tier 2: راه‌انداز داخلی — مستقیم binary Chrome را پیدا کرده و با مجموعه flagهای加固 راه‌اندازی می‌کند.
#              workbuddy و غیره که فقط skills/ را کپی می‌کنند (scripts/ همیشه غایب) از این مسیر می‌روند.
#    deploy فقط skill (workbuddy / ~/.claude/skills/) فقط باید تضمین کند:
#      a) .env در دایرکتوری بالایی skills/ قرار گیرد (همان جایی که scripts/ باید باشد)،
#         یا AGENTCHAT_ENV_FILE را روی آن تنظیم کنید — Node از v16 به بعد خودش .env را ایمن بارگذاری می‌کند
#      b) .env CHROMIUM_PATH را روی Chrome سیستم تنظیم کند (مثال Windows:
#         C:\\Program Files\\Google\\Chrome\\Application\\chrome.exe؛
#         تنظیم نشده باشد به طور خودکار مسیر نصب استاندارد را探测 می‌کند، شامل fallback Edge)
#      c) Windows + agent host (skill نصب شده در ~/.claude/skills/ و غیره): فرآیند skill
#         نمی‌تواند .env ریشه مخزن و scripts/ را ببیند (مسیرهای کاندید lib به
#         ~/.claude/.env解析 می‌شوند). با دو环境变量 escape door آن را بازگردانید (در سطح کاربر تنظیم کنید،
#         زیرفرآیند tool call به طور خودکار به ارث می‌برد):
#           setx AGENTCHAT_ENV_FILE    "C:\\path\\to\\AgentChat\\.env"
#           setx AGENTCHAT_SCRIPTS_DIR "C:\\path\\to\\AgentChat\\scripts"
#         در غیر این صورت skill فقط مقدار پیش‌فرض را探测 می‌کند 127.0.0.1:9222 + ~/.chrome-debug-profile،
#         با Chrome راه‌اندازی شده توسط ps1 طبق .env مخزن (پورت/profile سفارشی)分裂 می‌شود — پورت
#        探测 شکست بخورد سپس نمونه هم‌profile راه‌اندازی مجدد توسط مکانیزم single instance Windows جذب شده و فوراً خارج می‌شود.
#         از v18: راه‌انداز داخلی Chrome را از طریق WMI ایجاد می‌کند (خارج از Job Object ابزار调用،
#         دیگر با پایان run کشته نمی‌شود)；قبل از راه‌اندازی نمونه زنده‌ای که profile اشغال شده را探测 می‌کند
#         (فقط نمونه‌های مدیریت شده ثبت شده در فایل PID بازیافت می‌شوند، نمونه‌های دیگر سریع خطا می‌دهند)؛رد
#         دایرکتوری User Data پیش‌فرض مرورگر (Chrome ≥136 روی آن debugging port را بی‌صدا غیرفعال می‌کند).
#    راه‌اندازی دستی از پیش (اختیاری، زیر clone کامل سریع‌تر):
#    Linux/macOS:
pgrep -f "start-chrome-debug" || bash scripts/start-chrome-debug.sh
#    Windows (PowerShell؛ اولین بار با -FirstLogin وارد Gemini شوید):
#      powershell -ExecutionPolicy Bypass -File scripts\\start-chrome.ps1
#    توجه WSL2: 127.0.0.1 درون WSL VM است نه host Windows. وقتی Chrome در
#    سمت Windows اجرا می‌شود باید CDP_HOST=<IP host Windows> تنظیم کنید (ببینید .env.example).

# 2. دسترسی CDP
curl -s http://127.0.0.1:9222/json/version | python3 -c "import json,sys; print(json.load(sys.stdin).get('Browser','FAIL'))"

# 3. playwright-core (npm, ~3MB)
(cd ~/.claude/skills/AgentChat-OneWeb && npm install)
#    ⚠️ این skill به کتابخانه مشترک skills/lib/ هم‌سطح وابسته است (require('../lib/…')) —
#    هنگام نصب در ~/.claude/skills/ باید کل درخت کپی شود: AgentChat-OneWeb/ و lib/ کنار هم.
#    فقط کپی AgentChat-OneWeb/ در startup خطای FATAL با راهنمای تعمیر گزارش می‌دهد (از v14، دیگر MODULE_NOT_FOUND خام نیست).

# 4. حداقل یک سرویس AI وارد شده باشد (در Chrome profile)
#    URL ورود هر سرویس:
#    Gemini:  https://gemini.google.com/u/0/app
#    ChatGPT: https://chatgpt.com/
#    Claude:  https://claude.ai/
#    Qwen:    https://www.qianwen.com/?source=tongyigw
#    Kimi:    https://kimi.moonshot.cn/
#    MiniMax: https://agent.minimaxi.com/
```

---

## Invocation

```bash
# استفاده پایه — به طور خودکار زنجیره fallback را پیمایش می‌کند (پیش‌فرض تب‌های مرورگر حفظ می‌شوند)
node ~/.claude/skills/AgentChat-OneWeb/index.js "Your prompt"

# پس از اجرا به طور خودکار تب‌های مرورگر پاک می‌شوند
node ~/.claude/skills/AgentChat-OneWeb/index.js --close "Your prompt"

# تعیین timeout (ms)
node ~/.claude/skills/AgentChat-OneWeb/index.js --timeout=600000 "Long prompt..."

# خواندن از stdin
echo "Prompt from pipe" | node ~/.claude/skills/AgentChat-OneWeb/index.js

# بررسی محیط (prompt ارسال نمی‌شود)
node ~/.claude/skills/AgentChat-OneWeb/index.js --smoke

# بررسی اتصال CDP
node ~/.claude/skills/AgentChat-OneWeb/index.js --doctor

# اجبار provider شروع (رد کردن providerهای قبلی)
node ~/.claude/skills/AgentChat-OneWeb/index.js --from=ChatGPT "prompt"
```

### CLI Flags

| Flag | توضیح |
|------|------|
| `--timeout=N` | timeout کل (ms)، شامل زمان تلاش همه providerها، پیش‌فرض 600000 |
| `--timeout-per-provider=N` | timeout تک provider (ms)، پیش‌فرض `timeout / 2` یا 180000 |
| `--from=NAME` | شروع از provider مشخص، رد کردن providerهای قبلی در زنجیره. NAME می‌تواند مخفف باشد حساس به حروف بزرگ نیست |
| `--single` | فقط آن provider مشخص شده با `--from` را امتحان می‌کند، شکست یعنی بازگشت، به providerهای بعدی زنجیره cascade نمی‌شود. برای callerهایی که نیاز به降级+locking跨 provider خودشان دارند (مثل AgentChat-IndependentTasks)، از绕开 mutex caller توسط子进程 داخلی جلوگیری می‌کند |
| `--only=NAME` | ترکیب简写 `--from=NAME --single` (برای callerهای برنامه‌نویسی استفاده می‌شود؛ NAME ناشناخته fail loudly می‌دهد نه fallback ساکت) |
| `--locale=xx_XX` | اجبار پروفایل زبان UI Gemini (`zh_CN` / `zh_TW` / `en` / `ja`)، رد کردن تشخیص خودکار. پارامتر `locale=` در Python SDK همین flag را منتقل می‌کند |
| `--smoke` | بررسی محیط: پیمایش همه providerها تأیید حداقل یکی قابل دسترسی است |
| `--doctor` | بررسی اتصال پورت CDP |
| `--close` / `--close-browser` | پس از اجرا همه tab و اتصال مرورگر بسته می‌شود (پیش‌فرض حفظ می‌شود) |
| `--image` | نیت تولید تصویر: index.js به صورت داخلی دستور تقویت استاندارد تولید تصویر را追加 می‌کند و telemetry `image_prompt_enhanced` را ثبت می‌کند (ببینید پروتکل تصویر §1) |
| `--no-download-images` | غیرفعال کردن دانلود خودکار تصویر (پیش‌فرض فعال است، URL تصویر از پاسخ استخراج شده به دایرکتوری کار جاری دانلود می‌شود) |

> `--flag` ناشناخته لاگ `WARN` داده و نادیده گرفته می‌شود (از v14؛ قبلاً بی‌صدا دور ریخته می‌شد، ریشه bugهایی مثل `--locale` ماه‌ها بی‌کار می‌ماند، `--keep-tabs` به prompt拼入 می‌شد). `--from=` / `--only=` مقدار خالی با exit 64 hard fail می‌دهد. تحت `--only`/`--single` نام provider باید**دقیقاً مطابقت** داشته باشد با key یا نام نمایشی (مطابقت substring فقط در مسیر cascade به عنوان راحتی انسان حفظ می‌شود).

---

## Output & Telemetry

- **stdout**: در صورت موفقیت متن اصلی پاسخ AI را خروجی می‌دهد
- **stderr**: لاگ تشخیص، پیشوند `[fallback]`
- **telemetry**: در `~/.claude/skills/AgentChat-OneWeb/data/fallback-telemetry.jsonl` نوشته می‌شود

```json
{
  "timestamp": "2026-07-01T...",
  "provider_used": "ChatGPT",
  "providers_tried": ["Gemini"],
  "fallback_reasons": {"Gemini": "ERR_RATE_LIMITED"},
  "prompt_length_chars": 1500,
  "response_length_chars": 3200,
  "total_ms": 45000,
  "exit_code": 0
}
```

---

## Exit Codes

| Exit | Code | Meaning |
|------|------|---------|
| 0 | — | Success — پاسخ در stdout |
| 1 | `ERR_NO_CDP` | پورت Chrome CDP قابل دسترسی نیست |
| 2 | `ERR_NO_PROVIDER` | همه providerها غیرقابل استفاده (همه وارد نشده/نیاز به احراز هویت) |
| 3 | `ERR_SAFETY_REJECTED` | فیلتر ایمنی provider جاری رد کرده (همه امتحان شده‌اند) |
| 4 | `ERR_INTERNAL` | خطای داخلی (استثنای Node، قطع CDP و غیره) |
| 5 | `ERR_RATE_LIMITED` | همه providerها rate limit شده‌اند |
| 9 | `ERR_ALL_EXHAUSTED` | همه providerها پیمایش شدند، همه غیرقابل استفاده |
| 10 | `ERR_TIMEOUT` | timeout کل، بدون پاسخ کامل |
| 64 | `EX_USAGE` | خطای استفاده (prompt خالی / `--from=`،`--only=` مقدار خالی). قبل از v14 سوء استفاده exit 1، با ERR_NO_CDP تداخل داشت، باعث می‌شد orchestration side bug caller را به عنوان "مرورگر خراب شد" تفسیر کند و کل زنجیره را终止 کند. خطای استفاده هم receipt تولید می‌کند |

---

## Architecture

```
index.js
├── main()                    — ورودی CLI，پارامترها را解析 می‌کند
├── tryAllProviders()         — providerها را طبق زنجیره پیمایش می‌کند، اولین موفق را برمی‌گرداند
├── RUNNERS (factory-built)   — 8 runner provider از طریق createProviderRunner()
│   ├── gemini                — config: lib/providers/adapters/gemini.js
│   ├── chatgpt               — config: lib/providers/adapters/chatgpt.js
│   ├── claude                — config: lib/providers/adapters/claude.js
│   ├── qwen                  — config: lib/providers/adapters/qwen.js
│   ├── kimi                  — config: lib/providers/adapters/kimi.js
│   ├── minimax               — config: lib/providers/adapters/minimax.js
│   ├── mimo                  — config: lib/providers/adapters/mimo.js
│   └── deepseek              — config: lib/providers/adapters/deepseek.js
├── helpers/
│   ├── isProviderTabOpen()   — dedup tab (با smokeTest مشترک است)
│   ├── log() / startTimer()  — خروجی ترمینال (lib/terminal.js)
│   └── connectWithRetry()    — اتصال CDP + تلاش مجدد (lib/cdp.js)
└── constants/
    └── PROVIDER_CHAIN        — ترتیب اولویت + URL
```

### تصمیمات طراحی کلیدی

1. **یک صفحه برای هر فراخوانی** — هر فراخوانی tab مستقل ایجاد می‌کند، پیش‌فرض مرورگر را بسته نمی‌کند (`--close` می‌تواند پاک‌سازی خودکار را فعال کند).
2. **tab جدید برای هر provider** — هر تلاش provider از tab مستقل استفاده می‌کند (از طریق `context.newPage()`).
   پس از شکست tab جاری را بسته، tab جدید برای provider بعدی ایجاد می‌کند.
3. **تشخیص quota از طریق DOM** — به کد وضعیت HTTP وابسته نیست، بلکه محتوای DOM صفحه را بررسی می‌کند تا تشخیص دهد rate limit شده است.
4. **بدون context跨 provider** — context بین providerهای مختلف منتقل نمی‌شود. هر بار prompt مستقل است.
5. **Pro Extended الزامی برای Gemini** — Gemini باید Pro Extended را فعال کند تا استفاده شود، در غیر این صورت مستقیم降级 می‌شود.

---

## یادداشت‌های پیاده‌سازی خاص Provider

رفتار خاص هر provider در `lib/providers/adapters/<name>.js` تعریف شده است (config-driven، نه hard-coded)، SKILL.md فقط تفاوت‌های کلیدی را برای مرجع فراخوانی AI نگه می‌دارد:

| Provider | تفاوت کلیدی | ببینید |
|----------|---------|------|
| **Gemini** | اجبار فعال‌سازی Pro Extended، تشخیص خروجی bursty،延长 دکمه توقف 120s، لنگره تکمیل Action Toolbar | `adapters/gemini.js` |
| **ChatGPT** | استراتژی ورودی 3 لایه (clipboard→simulated paste→chunked keyboard)، اعتبارسنجی state دکمه ارسال React | `adapters/chatgpt.js` |
| **Claude** | ویرایشگر ProseMirror، فیلتر placeholder "Thinking"، جداسازی بلوک جستجوی تعبیه شده | `adapters/claude.js` |
| **Qwen** | تاخیر 3s React SPA، حالت detached دکمه توقف، جداسازی پیشوند نام مدل | `adapters/qwen.js` |
| **Kimi** | ایجاد session جدید برای هر فراخوانی، تشخیص disabled send-button-container، پنجره ثبات تطبیقی (5-30s) | `adapters/kimi.js` |
| **MiniMax** | تاخیر 4s挂载 ناهمگام TipTap/ProseMirror، ارسال `<div aria-label="发送消息">` غیر button | `adapters/minimax.js` |
| **MiMo** | تاخیر 4s React SPA، پیمایش DOM برای locating دکمه ارسال (بدون CSS selector قابل اعتماد) | `adapters/mimo.js` |
| **DeepSeek** | pipeline استاندارد، پاسخ ds-markdown | `adapters/deepseek.js` |

---

## افزودن Provider جدید

1. ایجاد `lib/providers/adapters/<name>.js` export کردن شیء config (به adapter موجود مراجعه کنید)
2. افزودن entry در آرایه `PROVIDER_CHAIN`
3. افزودن key در آرایه `PROVIDER_KEYS` (به طور خودکار در RUNNERS ثبت می‌شود)
4. فیلدهای کلیدی Config: `url`, `authDomains`, `editorSelectors`, `sendSelectors`/`sendFallback`, `responseSelectors`
5. تابع `{success: true, response: string}` یا `{success: false, reason: string}` برمی‌گرداند
   - `reason` باید باشد: `"quota"` | `"auth"` | `"error"` | `"timeout"`

---

## موقعیت کد

- `index.js` — ورودی CLI + orchestrator fallback
- `lib/providerFactory.js` — pipeline مبتنی بر config 10 مرحله‌ای (همه 8 provider مشترک)
- `lib/providers/adapters/<name>.js` — پیکربندی تفاوت‌های هر provider
- `lib/providers/chain.js` — ترتیب اولویت (منبع حقیقت واحد مشترک با IndependentTasks)
- `SKILL.md` — راهنمای عملیاتی面向 AI
- `package.json` — metadata npm (playwright-core)

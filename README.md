<div align="center">

[![Telegram](https://img.shields.io/badge/کانال_تلگرام-@avaco__cloud-2CA5E0?style=for-the-badge&logo=telegram&logoColor=white)](https://t.me/avaco_cloud)

<h1>🦕 Avaco Deno</h1>

<p>یک ریلی سبک و رایگان برای پروتکل XHTTP روی Deno Deploy</p>
<p>A lightweight free relay for the XHTTP protocol on Deno Deploy</p>

[![Deno](https://img.shields.io/badge/Deno-≥2.0-000000?style=flat-square&logo=deno&logoColor=white)](https://deno.com)
[![Deno Deploy](https://img.shields.io/badge/Deploy-Deno_Deploy-70FFAF?style=flat-square&logo=deno&logoColor=black)](https://deno.com/deploy)
[![License](https://img.shields.io/badge/License-GPL--3.0-blue?style=flat-square)](LICENSE)
[![Zero Dependencies](https://img.shields.io/badge/Dependencies-Zero-brightgreen?style=flat-square)](#)
[![Free Tier](https://img.shields.io/badge/Free_Tier-10K_req%2Fmonth-orange?style=flat-square)](#محدودیت‌های-رایگان)

---

**🌐 زبان / Language**

**[🇮🇷 فارسی](README.md)** &nbsp;|&nbsp; **[🇬🇧 English](README.en.md)**

<a name="fa"></a>

**[ [معرفی](#-معرفی) • [معماری](#-معماری) • [دیپلوی](#-مراحل-دیپلوی) • [چند اپ](#-دیپلوی-چند-اپ) • [متغیرها](#-متغیرهای-محیطی) • [باگ‌گیری](#-باگ‌گیری-و-عیب‌یابی) • [مدیریت](#-مدیریت) ]**

</div>

---

## 📖 معرفی

یک پروکسی معکوس سبک‌وزن بر پایه TypeScript/Deno خالص (بدون هیچ dependency) که روی پلتفرم **Deno Deploy** (پلن رایگان) اجرا می‌شه و ترافیک XHTTP رو از یه آدرس عمومی به سرور پشتی شما منتقل می‌کنه.

### ✨ ویژگی‌ها

| ویژگی | توضیح |
|-------|-------|
| 🦕 **TypeScript خالص** | بدون هیچ dependency خارجی، فقط Deno استاندارد |
| 🔐 **احراز هویت** | پشتیبانی از کلید امنیتی `x-relay-key` |
| 🚦 **کنترل ترافیک** | محدودیت درخواست‌های همزمان با `MAX_INFLIGHT` |
| ⏱️ **مدیریت تایم‌اوت** | قطع خودکار اتصالات کند با `UPSTREAM_TIMEOUT_MS` |
| 🔍 **دیباگ یکپارچه** | اندپوینت `/__debug` برای بررسی وضعیت لحظه‌ای |
| 🌍 **چند منطقه** | انتخاب لوکیشن از مناطق مختلف جغرافیایی |
| 📦 **چند اپ** | دیپلوی همزمان چند اپ برای چند سرور مختلف |
| 💸 **رایگان** | ۱۰,۰۰۰ درخواست ماهانه رایگان |

---

## 🏗️ معماری

```
┌─────────────────────────────────────────────────────────────┐
│                        کلاینت شما                           │
│                  (v2rayN / Hiddify / ...)                   │
└──────────────────────────┬──────────────────────────────────┘
                           │  HTTPS
                           │  HOST: APP.ORG.deno.net
                           │  Path: /PUBLIC_RELAY_PATH
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                🦕 Deno Deploy (Edge)                        │
│                                                             │
│   ┌─────────────────────────────────────────────────────┐   │
│   │           Avaco Deno Relay (TypeScript)             │   │
│   │                                                     │   │
│   │  ✓ بررسی مسیر (PUBLIC_RELAY_PATH)                  │   │
│   │  ✓ احراز هویت (x-relay-key)                        │   │
│   │  ✓ کنترل ترافیک (MAX_INFLIGHT)                     │   │
│   │  ✓ فوروارد هدرها (بدون لیک IP/Host)               │   │
│   │  ✓ استریم دوطرفه                                   │   │
│   └──────────────────────────┬──────────────────────────┘   │
└──────────────────────────────┼──────────────────────────────┘
                               │  HTTP
                               │  Path: /RELAY_PATH
                               ▼
┌─────────────────────────────────────────────────────────────┐
│              🖥️ سرور شخصی شما (Xray-core)                  │
│              TARGET_DOMAIN:PORT                             │
│              پروتکل: VLESS + XHTTP                         │
└─────────────────────────────────────────────────────────────┘
```

### جریان درخواست

```
کلاینت  ──►  Deno Deploy (PUBLIC_RELAY_PATH)  ──►  سرور (RELAY_PATH)  ──►  پاسخ
         ◄──                                   ◄──                     ◄──
```

---

## 🚀 مراحل دیپلوی

### پیش‌نیاز: نصب Deno

```bash
curl -fsSL https://deno.land/install.sh | sh
export PATH="$HOME/.deno/bin:$PATH"

# تأیید نصب (باید 2.x باشه)
deno --version
```

---

### مرحله ۱ — دریافت Access Token

۱. برو به **[console.deno.com](https://console.deno.com)**
۲. یه **Org** بساز (مثلاً `myorg`)
۳. برو به **Settings → Access Tokens → New Token**
۴. توکن رو کپی کن — با **`ddo_`** شروع می‌شه

> ⚠️ از `deployctl` استفاده نکن — فقط `deno deploy` روی پلتفرم جدید کار می‌کنه

---

### مرحله ۲ — کلون و ورود به پوشه

```bash
git clone https://github.com/avacocloud/avaco-deno-relay.git
cd avaco-deno-relay/deno
```

---

### مرحله ۳ — ساخت App و دیپلوی

```bash
deno deploy create \
  --token=ddo_YOUR_TOKEN \
  --org=YOUR_ORG \
  --app=YOUR_APP_NAME \
  --source=local \
  --entrypoint=main.ts \
  --region=eu \
  --no-wait \
  .
```

> ⚠️ **`--no-wait` ضروریه** — بدونش routing error میده

---

### مرحله ۴ — تنظیم متغیرهای محیطی

```bash
TOKEN=ddo_YOUR_TOKEN
ORG=YOUR_ORG
APP=YOUR_APP_NAME

deno deploy env add --token=$TOKEN --org=$ORG --app=$APP TARGET_DOMAIN      "https://YOUR-SERVER:PORT"
deno deploy env add --token=$TOKEN --org=$ORG --app=$APP PUBLIC_RELAY_PATH  "/api"
deno deploy env add --token=$TOKEN --org=$ORG --app=$APP RELAY_PATH         "/api"
deno deploy env add --token=$TOKEN --org=$ORG --app=$APP RELAY_KEY          "your-secret-key-min-16-chars"
deno deploy env add --token=$TOKEN --org=$ORG --app=$APP UPSTREAM_TIMEOUT_MS "0"
deno deploy env add --token=$TOKEN --org=$ORG --app=$APP MAX_INFLIGHT       "512"
```

---

### مرحله ۵ — تست و تأیید

آدرس اپت:
```
https://YOUR_APP_NAME.YOUR_ORG.deno.net
```

```bash
curl https://YOUR_APP_NAME.YOUR_ORG.deno.net/__debug
```

خروجی موفق:
```json
{
  "TARGET_BASE": "https://your-server:port",
  "PUBLIC_RELAY_PATH": "/api",
  "RELAY_PATH": "/api",
  "RELAY_KEY_SET": true,
  "UPSTREAM_TIMEOUT_MS": 0,
  "MAX_INFLIGHT": 512,
  "inFlight": 0
}
```

---

## 📦 دیپلوی چند اپ

یکی از مزیت‌های Deno Deploy اینه که می‌تونی چند اپ موازی با سرورهای مختلف داشته باشی:

```bash
TOKEN=ddo_YOUR_TOKEN
ORG=YOUR_ORG

APPS=(relay1 relay2 relay3 relay4)
DOMAINS=(
  "https://server1.example.com:443"
  "https://server2.example.com:443"
  "https://server3.example.com:443"
  "https://server4.example.com:443"
)

# ساخت همه اپ‌ها
for i in 0 1 2 3; do
  deno deploy create \
    --token=$TOKEN --org=$ORG \
    --app=${APPS[$i]} \
    --source=local --entrypoint=main.ts \
    --region=eu --no-wait .
done

# ست کردن env برای همه
for i in 0 1 2 3; do
  APP=${APPS[$i]}
  DOMAIN=${DOMAINS[$i]}
  deno deploy env add --token=$TOKEN --org=$ORG --app=$APP TARGET_DOMAIN      "$DOMAIN"
  deno deploy env add --token=$TOKEN --org=$ORG --app=$APP PUBLIC_RELAY_PATH  "/api"
  deno deploy env add --token=$TOKEN --org=$ORG --app=$APP RELAY_PATH         "/api"
  deno deploy env add --token=$TOKEN --org=$ORG --app=$APP UPSTREAM_TIMEOUT_MS "0"
  deno deploy env add --token=$TOKEN --org=$ORG --app=$APP MAX_INFLIGHT       "512"
done
```

> 💡 وقتی به لیمیت رایگان رسیدی، یه org جدید با حساب دیگه بساز و دوباره دیپلوی کن.

---

## ⚙️ متغیرهای محیطی

| متغیر | اجباری | پیش‌فرض | توضیح |
|-------|--------|---------|-------|
| `TARGET_DOMAIN` | ✅ | — | آدرس کامل سرور شما. مثال: `https://1.2.3.4:443` |
| `PUBLIC_RELAY_PATH` | ✅ | `/api` | مسیری که کلاینت می‌زنه. مثال: `/xhttp` |
| `RELAY_PATH` | ✅ | `/api` | مسیری که روی سرور Xray تنظیم کردی |
| `RELAY_KEY` | ❌ | — | کلید امنیتی (حداقل ۱۶ کاراکتر) — توصیه می‌شه |
| `UPSTREAM_TIMEOUT_MS` | ❌ | `0` | تایم‌اوت اتصال به سرور (میلی‌ثانیه). `0` = بدون تایم‌اوت |
| `MAX_INFLIGHT` | ❌ | `512` | حداکثر درخواست همزمان |

### نکات مهم متغیرها

- **`TARGET_DOMAIN`** نباید `/` داشته باشه — مثال: `https://1.2.3.4:443` ✅ نه `https://1.2.3.4:443/` ❌
- **`PUBLIC_RELAY_PATH`** و **`RELAY_PATH`** نمی‌تونن `/` باشن
- اگه `RELAY_KEY` بزاری، کلاینت باید هدر `x-relay-key` رو بفرسته
- **`RELAY_PATH`** هر مقداری می‌تونه باشه ولی **حتماً باید با مسیری که توی کانفیگ Xray روی سرورت تنظیم کردی یکی باشه**

---

## 🌍 انتخاب لوکیشن

برای دیدن لیست کامل لوکیشن‌های موجود:

```bash
deno deploy create --help
```

در دستور `create` لوکیشن رو ست کن:
```bash
--region=YOUR_REGION
```

مثال:
```bash
deno deploy create \
  --token=ddo_YOUR_TOKEN \
  --org=YOUR_ORG \
  --app=YOUR_APP_NAME \
  --source=local \
  --entrypoint=main.ts \
  --region=eu \
  --no-wait \
  .
```

---

## 🔍 باگ‌گیری و عیب‌یابی

### ابزار دیباگ اصلی

```bash
curl https://YOUR_APP.YOUR_ORG.deno.net/__debug
```

این اندپوینت وضعیت کامل ریلی رو نشون می‌ده — اولین جایی که باید چک کنی.

---

### خطاهای رایج و راه‌حل

<details>
<summary>❌ <b>خطای 500 — Misconfigured: TARGET_DOMAIN is not set</b></summary>

**علت:** متغیر `TARGET_DOMAIN` ست نشده.

```bash
deno deploy env add --token=$TOKEN --org=$ORG --app=$APP TARGET_DOMAIN "https://YOUR-SERVER:PORT"
```

</details>

<details>
<summary>❌ <b>خطای 500 — Misconfigured: RELAY_PATH cannot be '/'</b></summary>

**علت:** مسیر رو `/` گذاشتی که مجاز نیست.

```bash
deno deploy env add --token=$TOKEN --org=$ORG --app=$APP RELAY_PATH "/api"
```

</details>

<details>
<summary>❌ <b>خطای 500 — Misconfigured: RELAY_KEY is too short</b></summary>

**علت:** کلید امنیتی کمتر از ۱۶ کاراکتر داره.

```bash
# یه کلید تصادفی بساز:
openssl rand -hex 16

# بعد ست کن:
deno deploy env add --token=$TOKEN --org=$ORG --app=$APP RELAY_KEY "your-generated-key"
```

</details>

<details>
<summary>❌ <b>خطای 403 — Forbidden</b></summary>

**علت:** هدر `x-relay-key` در درخواست وجود نداره یا اشتباهه.

در کانفیگ کلاینتت باید هدر زیر رو اضافه کنی:
```
x-relay-key: your-secret-key
```

</details>

<details>
<summary>❌ <b>خطای 404 — Not Found</b></summary>

**علت:** مسیر درخواست با `PUBLIC_RELAY_PATH` مطابقت نداره.

```bash
# ببین الان چی ست شده:
deno deploy env list --token=$TOKEN --org=$ORG --app=$APP
```

</details>

<details>
<summary>❌ <b>خطای 503 — Server Busy: Too Many Inflight Requests</b></summary>

**علت:** تعداد درخواست‌های همزمان از `MAX_INFLIGHT` بیشتره.

```bash
deno deploy env update-value --token=$TOKEN --org=$ORG --app=$APP MAX_INFLIGHT "1024"
```

</details>

<details>
<summary>❌ <b>خطای 502 — Bad Gateway</b></summary>

**علت:** ریلی نمی‌تونه به `TARGET_DOMAIN` وصل بشه.

**چک‌لیست:**
- [ ] آیا `TARGET_DOMAIN` درسته؟ (IP و پورت)
- [ ] آیا فایروال سرور پورت رو باز کرده؟
- [ ] آیا Xray روی سرور در حال اجراست؟
- [ ] تست دسترسی مستقیم: `curl http://YOUR-SERVER:PORT`

</details>

<details>
<summary>❌ <b>خطای 504 — Gateway Timeout</b></summary>

**علت:** سرور در زمان تنظیم‌شده پاسخ نداد.

```bash
deno deploy env update-value --token=$TOKEN --org=$ORG --app=$APP UPSTREAM_TIMEOUT_MS "0"
```

</details>

<details>
<summary>❌ <b>دیپلوی routing error میده</b></summary>

**علت:** فراموش کردی `--no-wait` رو بزاری.

```bash
# حتماً --no-wait باشه:
deno deploy create ... --no-wait .
```

</details>

<details>
<summary>❌ <b>به لیمیت رایگان رسیدم</b></summary>

```bash
# یه org جدید بساز روی console.deno.com
# توکن جدید بگیر
# دوباره دیپلوی کن با org جدید
deno deploy create --token=NEW_TOKEN --org=NEW_ORG --app=APP_NAME ...
```

</details>

---

### تست سلامت ریلی

```bash
# تست کامل:
curl -v https://YOUR_APP.YOUR_ORG.deno.net/__debug

# تست با RELAY_KEY:
curl -H "x-relay-key: your-secret-key" https://YOUR_APP.YOUR_ORG.deno.net/api

# تست پینگ ساده:
curl -o /dev/null -s -w "%{http_code}\n" https://YOUR_APP.YOUR_ORG.deno.net/__debug
# باید 200 برگردونه
```

---

## 🛠️ مدیریت

```bash
# لیست متغیرها
deno deploy env list --token=$TOKEN --org=$ORG --app=$APP

# آپدیت متغیر
deno deploy env update-value --token=$TOKEN --org=$ORG --app=$APP TARGET_DOMAIN "https://NEW-SERVER:PORT"

# حذف متغیر
deno deploy env delete --token=$TOKEN --org=$ORG --app=$APP RELAY_KEY

# آپدیت کد (ری‌دیپلوی)
deno deploy --token=$TOKEN --org=$ORG --app=$APP --prod --no-wait .

# حذف یک اپ
curl -X DELETE -H "Authorization: Bearer $TOKEN" \
  "https://api.deno.com/v2/apps/APP_NAME"

# حذف چند اپ
for app in relay1 relay2 relay3 relay4; do
  curl -s -X DELETE -H "Authorization: Bearer $TOKEN" \
    "https://api.deno.com/v2/apps/$app"
done
```

---

## 💰 محدودیت‌های پلن رایگان

| متریک | مقدار |
|-------|-------|
| 📡 HTTP Requests | **۱۰,۰۰۰ / ماه** |
| ⚡ CPU Time | 0.2 ساعت / ماه |
| 🧠 Memory | 768 MB |
| 🌐 Outbound Traffic | 1 GiB / ماه |
| 😴 Sleep | ندارد |

> 💡 وقتی به لیمیت رسیدی، یه org جدید با حساب دیگه بساز و دوباره دیپلوی کن.

---

## 📁 ساختار فایل‌ها

```
deno/
├── main.ts        ← کد اصلی ریلی (TypeScript خالص)
├── deno.json      ← تنظیمات Deno
└── README.md      ← این فایل
```

---

## 💖 حمایت مالی

اگه این پروژه بهت کمک کرد و دوست داری حمایت کنی، می‌تونی با ارز دیجیتال دونیت کنی:

[![Donate with crypto](https://nowpayments.io/images/embeds/donation-button-white.svg)](https://nowpayments.io/donation?api_key=53edc3b4-8a65-451a-9ca9-67c30519c7a5)

---

## 🔗 لینک‌های مرتبط

- 🦕 [Deno Deploy Console](https://console.deno.com)
- 🔑 [Deno Access Tokens](https://console.deno.com/account/access-tokens)
- 📦 [Deno Deploy Docs](https://docs.deno.com/deploy/manual)
- 🛠️ [XHTTP-Installer (نصب خودکار سرور)](https://github.com/avacocloud/XHTTP-Installer)

---

## 📜 لایسنس

این پروژه تحت لایسنس **GNU GPL-3.0** منتشر شده.

هر گونه استفاده مجدد، فورک، یا تغییر در کد **مشروط به حفظ** موارد زیر است:

- ✅ کپی‌رایت اصلی: `Copyright (C) 2025 avaco_cloud`
- ✅ لینک مخزن اصلی: [github.com/avacocloud/avaco-deno-relay](https://github.com/avacocloud/avaco-deno-relay)
- ✅ اشاره به سازنده: [@avaco_cloud](https://t.me/avaco_cloud)
- ✅ فایل LICENSE بدون تغییر

> ⚠️ حذف یا جایگزین کردن اطلاعات سازنده نقض این لایسنس است و منجر به **DMCA Takedown** می‌شود.

---

<div align="center">

ساخته شده با ❤️ توسط [@avaco_cloud](https://github.com/avacocloud)

</div>

Bisa. Ini file Markdown yang bisa kamu simpan sebagai README.md.

# Dezz Telegram Bot (Plugin System + Hot Reload + Web Panel)

Bot Telegram berbasis **Telegraf** dengan sistem **plugin universal** (mau plugin bawaan / plugin buatan sendiri tetap bisa), **hot reload** (add/edit/del plugin tanpa restart), **auto install dependency** saat plugin butuh package tambahan, dan **web panel** (public port).

## Struktur Project

├─ index.js ├─ config.js ├─ main.js ├─ package.json ├─ plugins/ │  ├─ example.ping.js │  ├─ example.echo.js │  └─ example.admin.js └─ data/ └─ bans.json

> Folder `data/` otomatis dibuat saat pertama run.

## Instalasi & Menjalankan

### 1) Install dependency
```bash
npm i

2) Isi config (opsional)

Edit config.js.

Minimal agar bot bisa login Telegram:

BOT_TOKEN


Kalau field lain kosong, tidak error, cuma warning di console.

3) Start

node index.js


Config (config.js)

Semua field opsional, kalau kosong hanya warning.

Contoh:

module.exports = {
  BOT_TOKEN: "8556520547:AAHnuI",
  OWNER_ID: "8302651892",
  PORT: 3000
};

Daftar field yang tersedia:

BOT_TOKEN

OWNER_ID, OWNER_USERNAME

BOT_NAME, BOT_USERNAME

Panel/Ptero (opsional): PLTA, PLTC, DOMAIN, EGG, NEST, LOC, KEYPAKASIR, SLUGPAKASIR

Web: PORT, OWNER_WEBSITE

Channel/Group: ID_CHANNEL (array), ID_GROUP, USERNAME_CHANNEL, USERNAME_GROUP



Web Panel (Public Port)

Web server jalan di PORT (default 3000).

Endpoint:

/ : info

/health : status OK

/stats : statistik realtime bot

/plugins : daftar plugin loaded

/config : config disensor (token dimasking)


Contoh:

http://localhost:3000/stats



Logging (Semua Masuk Console)

IN (User -> Bot)

Semua update dicatat:

message teks

command

sticker / gif / video / file / doc / photo / vn / audio

forward

callback button


Format log:

[IN ][message] from=... chat=... type=... text="..."
[IN ][callback_query] ... data="..."

OUT (Bot -> User)

Semua request API Telegram dicatat:

[OUT][API] sendMessage -> chat_id=...


Statistik Bawaan

Bot menyimpan statistik internal:

inMessage : pesan masuk (termasuk callback)

outMessage : pesan keluar (send/edit)

commands : jumlah command masuk

errors : total error (plugin/telegram/api/handler)

noResponse : event tidak ada plugin yang respon

badResponse : disediakan untuk kamu pakai (opsional)

successApiCall : API call sukses

successButtonCall : callback yang berhasil ditangani plugin

spamWarned : jumlah user yang pernah diwarn spam


Cek via:

Command: /stats (plugin contoh)

Web: /stats



Anti-Spam Command / Button

Ada rate limit sederhana:

max 6 aksi dalam 6 detik per user per tipe (cmd/button)

kalau spam, bot akan warn:

⚠️ Jangan spam command ya.

Jangan spam button ya.



> Setelah warning, request tetap diproses (biar plugin tetap bisa decide).




Ban / Unban Bawaan

Sistem ban membuat user di-ignore total (seperti diblokir):

user tidak akan dapat respon apa pun

pesan user dibuang sejak awal handler


Command (dari plugin contoh example.admin.js):

/ban <user_id> atau reply user lalu /ban

/unban <user_id> atau reply user lalu /unban

/banlist


Akses:

hanya OWNER_ID yang boleh

kalau OWNER_ID kosong, admin command otomatis ditolak


Data ban tersimpan di:

data/bans.json



Plugin System

Lokasi plugin

Semua plugin taruh di:

plugins/*.js


Hot Reload Plugin

Add plugin baru -> otomatis load

Edit plugin -> otomatis reload (cache require dibersihin)

Delete plugin -> otomatis unload


Anti double respon:

dispatch plugin dikontrol oleh core, bukan bot.command() per plugin

jadi edit plugin tidak akan bikin handler numpuk


Format Plugin (Universal)

Plugin bisa export object:

module.exports = {
  name: "namaplugin",
  async onCommand(payload) {},
  async onMessage(payload) {},
  async onMedia(payload) {},
  async onCallback(payload) {},
  async onAny(payload) {}
};

Atau export function (akan dianggap onUpdate minimal, tapi yang paling aman tetap object).

Payload yang diterima plugin

Setiap handler plugin menerima payload seperti:

ctx : context Telegraf

bot : instance bot (Telegraf)

config : config object

stats : stats manager

bans : helper ban/unban

utils.safeReply(ctx, text) : reply aman (try/catch)

text : text/caption (untuk message)

mediaLabel : tipe media (photo, video, sticker, dll)

command : hasil parse command (kalau ada)

callbackData : data callback button (kalau callback)

updateType : message/channel_post/edited/callback_query


Return value plugin

Kalau plugin meng-handle event, return true. Kalau tidak handle, return false.

Ini penting untuk:

ngitung noResponse

nentuin apakah event sudah ditangani atau belum



Auto Install Dependency (Plugin Butuh Package Tambahan)

Kalau plugin melakukan:

const axios = require("axios");

Tapi axios belum ada di package.json / node_modules, core akan:

1. Deteksi error: Cannot find module 'axios'


2. Auto jalankan: npm i axios --save


3. Load ulang plugin



> Kalau install gagal, bot tetap hidup, plugin itu aja yang gagal load.




Support Group & Channel

Core listen:

message

edited_message

channel_post

edited_channel_post

callback_query


Jadi command bisa masuk dari:

private chat

group/supergroup

channel post



Contoh Plugin

example.ping.js

/ping -> pong ✅

/stats -> tampil statistik


example.echo.js

echo teks/media (private saja, biar group gak rame)


example.admin.js

/ban, /unban, /banlist (owner-only)



Troubleshooting

BOT_TOKEN kosong

Bot Telegram tidak bisa login (wajar)

Web panel tetap hidup


Plugin error terus

Core akan log error plugin, tapi bot tetap jalan

Cek console untuk detail [PLUGIN][ERR] ...


Plugin tidak ke-load

Pastikan file plugin *.js

Pastikan export module.exports = { name: "...", ... }


Lisensi

Bebas dipakai, modif, dan dikembangkan.

Kalau kamu mau, gue juga bisa buatin versi **“DOCS.md khusus Plugin API”** yang lebih detail + template plugin generator (biar bikin plugin baru tinggal copy-paste).

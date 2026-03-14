# 1. Включи Offline access
В **Dropbox App Console → Permissions**
Поставь галочки:
files.content.write 
files.content.read 
sharing.read
И самое главное:

> ✅ **Allow offline access**

Нажми **Submit**

# 2. Возьми App key и App secret

Вкладка **Settings**

Скопируй:
App key 
App secret

# 3. Получи authorization code

В браузере открой:
https://www.dropbox.com/oauth2/authorize?client_id=APP_KEY&response_type=code&token_access_type=offline

Dropbox спросит доступ → нажми **Allow**

Ты будешь редиректнут на пустую страницу типа:
https://localhost/?code=O4u7sdfkJsd92...
Скопируй значение `code`.

# 4. Обмениваем code → refresh_token

Выполни curl:

`curl https://api.dropboxapi.com/oauth2/token \   -d code=PASTE_CODE_HERE \   -d grant_type=authorization_code \   -d client_id=APP_KEY \   -d client_secret=APP_SECRET`

Ты получишь JSON
{
  "access_token": "sl.ABC...",
  "token_type": "bearer",
  "expires_in": 14400,
  "refresh_token": "slrt.ABCDEFG123456",
  "scope": "files.content.write files.content.read sharing.read",
  "uid": "123456789",
  "account_id": "dbid:AAAA..."
}

Сохраняешь только:
refresh_token
app_key
app_secret

Как использовать в FastAPI

dbx = dropbox.Dropbox(
    oauth2_refresh_token=DROPBOX_REFRESH_TOKEN,
    app_key=DROPBOX_APP_KEY,
    app_secret=DROPBOX_APP_SECRET,
)


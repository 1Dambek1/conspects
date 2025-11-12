### 🧩 1. Установка `uv`

Если у тебя ещё нет `uv`, установи его одной из команд:

#### Для Windows (PowerShell):
```bash
powershell -c "irm https://astral.sh/uv/install.ps1 | iex"
```


Для macOS / Linux:
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

После этого перезапусти терминал и проверь:
```bash
uv --version
```
### ⚙️ 2. Создание проекта с виртуальной средой

#### Вариант 1 — создать новую среду в текущей папке
```bash
uv venv
```

По умолчанию создастся папка `.venv/` с виртуальной средой (аналогично `python -m venv .venv`).

Вариант 2 — создать среду в другом месте
```bash
uv venv .myenv
```
### 🚀 3. Активация виртуальной среды

**Windows (PowerShell):**
```bash
.venv\Scripts\Activate.ps1
```
Windows (cmd):
```bash
.venv\Scripts\activate.bat
```
Linux/macOS:
```bash
source .venv/bin/activate
```

📦 4. Установка зависимостей

```bash
uv pip install fastapi uvicorn
```

🧰 5. Проверка установленных пакетов
```bash
uv pip list
```

### 💡 Дополнительно: создание проекта с зависимостями

`uv` может автоматически создать проект:
```bash
uv init myproject
```
Он создаст структуру проекта и виртуальную среду внутри.

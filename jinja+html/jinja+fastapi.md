# Объяснение: Jinja + FastAPI

## Что такое Jinja?
- **Jinja** — это шаблонизатор для Python, который используется для генерации HTML на основе данных.
- Применяется для создания динамических веб-страниц.
- Преимущества:
  - Простая работа с переменными, условиями и циклами.
  - Возможность переиспользования кода через макросы и наследование шаблонов.

## Как работает Jinja с FastAPI?
- FastAPI интегрируется с Jinja через библиотеку `Jinja2Templates`.
- Это позволяет рендерить HTML-шаблоны и передавать в них данные из Python.

---

## Настройка проекта

### Установка зависимостей
Установите необходимые пакеты:
```bash
pip install fastapi jinja2 uvicorn
```
### Структура проекта

Организация папок для работы с Jinja:
```
src/
├── main.py               # Основной файл приложения
├── templates/            # Папка для шаблонов Jinja
│   ├── index.html
│   ├── about.html
└── static/               # Папка для CSS, JS и изображений

```
## Пример приложения

### Базовый код FastAPI + Jinja

Создайте файл `main.py`:
```python
from fastapi import FastAPI, Request
from fastapi.responses import HTMLResponse
from fastapi.templating import Jinja2Templates
from fastapi.staticfiles import StaticFiles

app = FastAPI()

# Настройка шаблонов и статических файлов
templates = Jinja2Templates(directory="templates")
app.mount("/static", StaticFiles(directory="static"), name="static")

# Маршрут для главной страницы
@app.get("/", response_class=HTMLResponse)
async def read_root(request: Request):
    return templates.TemplateResponse("index.html", {"request": request, "title": "Главная страница"})

# Маршрут для страницы "О нас"
@app.get("/about", response_class=HTMLResponse)
async def about_page(request: Request):
    return templates.TemplateResponse("about.html", {"request": request, "title": "О нас", "description": "Это пример страницы о проекте."})

```

## Шаблоны Jinja

### Пример шаблона `index.html`:
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{{ title }}</title>
    <link rel="stylesheet" href="/static/style.css">
</head>
<body>
    <h1>{{ title }}</h1>
    <p>Добро пожаловать на наш сайт!</p>
    <a href="/about">Узнать больше о нас</a>
</body>
</html>

```
Пример шаблона `about.html`:
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{{ title }}</title>
    <link rel="stylesheet" href="/static/style.css">
</head>
<body>
    <h1>{{ title }}</h1>
    <p>{{ description }}</p>
    <a href="/">Вернуться на главную</a>
</body>
</html>

```

## Возможности Jinja

### Использование переменных

Передача переменной в шаблон:
```python
@app.get("/greet", response_class=HTMLResponse)
async def greet_user(request: Request):
    return templates.TemplateResponse("greet.html", {"request": request, "username": "Алиса"})

```
Шаблон `greet.html`:
```html
<p>Привет, {{ username }}!</p>

```
Условия
```html
{% if user_logged_in %}
    <p>Добро пожаловать, {{ username }}!</p>
{% else %}
    <p>Пожалуйста, войдите в систему.</p>
{% endif %}

```
Циклы
```html
<ul>
    {% for item in items %}
        <li>{{ item }}</li>
    {% else %}
        <li>Список пуст.</li>
    {% endfor %}
</ul>

```

### Наследование шаблонов

Базовый шаблон `base.html`:
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>{% block title %}Мой сайт{% endblock %}</title>
</head>
<body>
    {% block content %}{% endblock %}
</body>
</html>

```

Дочерний шаблон `index.html`:
```html
{% extends "base.html" %}

{% block title %}Главная{% endblock %}

{% block content %}
    <h1>Добро пожаловать!</h1>
{% endblock %}

```


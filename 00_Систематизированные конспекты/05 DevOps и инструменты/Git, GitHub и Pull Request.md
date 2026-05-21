# Git, GitHub и Pull Request

Связанные темы: [[Docker, сервер и окружение]], [[Архитектура FastAPI-проекта|архитектура проекта]].

## Git и GitHub

Git - система контроля версий. GitHub - сервис для хранения репозиториев и совместной работы.

## Базовый цикл

```bash
git init
git status
git add .
git commit -m "Initial commit"
git remote add origin <repo-url>
git push -u origin main
```

## Получить проект

```bash
git clone <repo-url>
```

## Ветки

Ветка позволяет разрабатывать новую функцию отдельно от основной версии.

```bash
git branch
git branch feature/auth
git switch feature/auth
```

Создать и сразу перейти:

```bash
git switch -c feature/auth
```

Слить ветку:

```bash
git switch main
git merge feature/auth
```

Удалить ветку:

```bash
git branch -d feature/auth
```

## Pull Request

Pull Request - запрос на внесение изменений из одной ветки в другую.

Типичный процесс:

1. Создать ветку.
2. Сделать изменения.
3. Закоммитить.
4. Запушить ветку.
5. Открыть Pull Request на GitHub.
6. Дождаться review.
7. Влить изменения.

Команды:

```bash
git switch -c feature/payments
git add .
git commit -m "Add payments"
git push -u origin feature/payments
```

## Если нет доступа к репозиторию

Для open-source проекта:

1. сделать fork;
2. склонировать fork;
3. создать ветку;
4. отправить изменения в свой fork;
5. открыть Pull Request в оригинальный репозиторий.

## Полезные команды

```bash
git log --oneline
git diff
git restore <file>
git pull
git fetch
```

Сначала проверяй `git status`, затем добавляй и коммить только нужные файлы.

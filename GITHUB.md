# Создание репозиториев на GitHub

Репозитории уже созданы у **4sidora**. Для другого аккаунта замените `4sidora` / `YOUR_ORG` на свой префикс.

## Требования

- [Git](https://git-scm.com/)
- [GitHub CLI](https://cli.github.com/) (`gh`) — опционально, удобно для `gh repo create`

Выполните `gh auth login` один раз.

## Уже локально: отдельный `git` в каждом каталоге

В корне каждого из пяти каталогов уже можно выполнить `git init` и первый коммит (см. историю в этом репозитории или выполните команды из раздела «Ручной первый push»).

## Вариант через `gh` (пустой репозиторий + push)

Из каталога **beeplan-docs** (после `git init` и коммита):

```bash
gh repo create 4sidora/beeplan-docs --public --source=. --remote=origin --push
```

Повторите для остальных имён репозитория (`beeplan-api`, `beeplan-web`, `beeplan-edge`, `beeplan-gateway`), меняя вторую часть после `/`.

Под **другим** пользователем или org: `YOUR_ORG/beeplan-docs` и т.д.

## Ручной первый push

1. На GitHub: **New repository** → имя `beeplan-docs` → без README/license (или с .gitignore — тогда придётся `pull --rebase` перед push).
2. Локально:

```bash
cd beeplan-docs
git init
git add .
git commit -m "Initial commit: BeePlan documentation"
git branch -M main
git remote add origin https://github.com/YOUR_ORG/beeplan-docs.git
git push -u origin main
```

Повторите для каждого из пяти репозиториев, подставляя соответствующий каталог и URL.

## Рабочая область с несколькими клонами

Удобно открыть в IDE родительскую папку, внутри которой лежат пять клонов рядом:

```text
work/
  beeplan-docs/
  beeplan-api/
  beeplan-web/
  beeplan-edge/
  beeplan-gateway/
```

Ссылки между репозиториями в документации ведут на GitHub; локальные пути в README каждого репозитория — только внутри этого репозитория.

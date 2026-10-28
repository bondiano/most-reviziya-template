# Ревизия: прокат Квота на PostgreSQL

Босс раздела PY12 школы BatSchool: прокат дронов Квота, уже под тестами и в гексагоне, переезжает на PostgreSQL, и двадцать курьеров разом не могут взять один дрон. Условие заказа в школе, на странице этапа «Заказ: двадцать курьеров и один дрон». В репозитории только обвязка: зависимости, PostgreSQL в Docker, пример окружения и проверка в CI. Код твой: перенеси сюда каталоги `prokat/` и `tests/` из своего решения десятого этапа (их видно в редакторе этапа на странице урока), закоммить и дальше дописывай заказ.

## Как работать локально

Нужны [uv](https://docs.astral.sh/uv/) и Docker.

```sh
uv sync                                   # Python 3.14 и зависимости
docker compose up -d                      # PostgreSQL 17 на :5432
cp .env.example .env
uv run --env-file .env pytest             # твои тесты из tests/
docker compose exec postgres psql -U postgres -c 'SELECT * FROM rentals'
```

## Как проверяется

1. Привяжи свой репозиторий в школе: настройки, блок «Репозитории изделий», строка «Ревизия: легаси Квота под тестами и в гексагоне», поле `владелец/репозиторий`, кнопка «Привязать». Владелец репозитория должен совпасть с твоим аккаунтом GitHub.
2. Каждый пуш запускает `.github/workflows/grade.yml`. Он вызывает общий workflow школы `bondiano/py-graders-ci`, тот поднимает PostgreSQL 17, гоняет скрытые проверки образом `ghcr.io/bondiano/py-grader-reviziya:11` и отправляет результат в школу. Проверки PostgreSQL получают адрес в `DATABASE_URL`, проверки прошлых этапов идут на SQLite во временных файлах. Зелёный прогон засчитывает этап.
3. Не меняй `grade.yml`: школа принимает отчёт только от общего workflow на опубликованном теге.

Тот же прогон можно сделать у себя. Грейдер дописывает свои файлы в каталог проекта, поэтому дай ему копию:

```sh
docker compose up -d
work="$(mktemp -d)" && cp -R . "$work" && rm -rf "$work/.git" && chmod -R a+rwX "$work"
docker run --rm --network host --read-only --tmpfs /tmp \
  -e DATABASE_URL=postgresql://postgres:postgres@localhost:5432/postgres \
  -v "$work:/work" ghcr.io/bondiano/py-grader-reviziya:11
```

Каждая строка вывода это одна проверка: `"passed": true` или `false` и сообщение.

        # redis — Списки и стримы

        Homework-шаблон для урока **l2_lists_and_streams** (Списки и стримы) на платформе Vibe Learn.

        ## Что делать

        Реализуй воркер на go-redis с XREADGROUP / XACK / XCLAIM. На каждое XREADGROUP читает пачку,
обрабатывает (мок: 50мс sleep), делает XACK. Эмулирует падение каждый 10й вызов (panic).
Второй воркер должен через XAUTOCLAIM забрать зависшие. Тесты проверят: at-least-once,
отсутствие потерь, корректный счётчик XLEN.

## Контекст (из transfer-задачи урока)

У тебя email-notifications service. Сейчас на Celery + RabbitMQ. Команда хочет перевести
на Redis-only стек, чтобы убрать ещё один компонент. Ты должен решить: list или stream
для очереди email-задач? Учти: 3 группы воркеров (transactional, marketing, system),
нужен retry при сбоях SMTP, хочется метрику «сколько задач за последний час прошло».

## Recap из урока

- List — простая FIFO/LIFO-очередь. LPUSH/BRPOP, всё O(1). Минусы: одно сообщение одному воркеру, нет групп, нет истории.
- Stream — append-only лог с consumer-группами. Каждое сообщение — map полей с monotonically growing ID. История остаётся.
- XREADGROUP + XACK = at-least-once. Невыданные ACK копятся в PEL. XCLAIM/XAUTOCLAIM забирает зависшие у мёртвых воркеров.
- Reliable queue паттерн: `BLMOVE source dest RIGHT LEFT` (наследник BRPOPLPUSH) атомарно перекладывает в processing-список — переживёт падение воркера.
- Stream — для multi-consumer и event-driven. List — для простых очередей. Не бери Stream «потому что круто» — overhead больше.

        ## Как работать

        1. Платформа Vibe Learn создаёт копию этого репо в твоём GitHub-аккаунте по клику «Начать домашку» на странице урока (через GitHub `/generate`, codecrafters-pattern).
        2. Склонируй копию локально, реализуй TODO в `main.go`, прогони тесты, запушь.
        3. CI (`.github/workflows/ci.yml`) запускает `go vet` + `go test ./...` на каждый push. Платформа слушает результат через webhook от GitHub Actions и обновляет статус домашки на странице урока.

        ## Локальное окружение

        - Go 1.22+
        - Docker + docker-compose — `docker compose up -d` поднимает single-node Redis 7 на `localhost:6379` (с включёнными keyspace-notifications и AOF). Адрес переопределяется через env `REDIS_ADDR`.

        ## Запуск

        ```bash
        # Поднять локальный Redis
        docker compose up -d

        # Прогнать тесты (интеграционный включается через REDIS_INTEGRATION=1)
        go test ./...
        REDIS_INTEGRATION=1 go test ./...

        # Запустить main (печатает marker; замени stub на реализацию)
        go run .
        ```

        ## Заметка автора

        Это baseline-шаблон, сгенерированный платформой. Бизнес-сущность задачи (что конкретно реализовать в `main.go`, какие тесты сделать строгими) расширяется по ходу итераций — параллельно с углублением теории урока.

# Docker + docker-compose (основы)

Вы узнаете:
- Что такое контейнер и зачем он нужен — решение проблемы «у меня работает, у тебя нет»
- Разница между образом (image) и контейнером: образ — шаблон, контейнер — запущенный экземпляр
- Как писать `Dockerfile`: `FROM`, `WORKDIR`, `COPY`, `RUN`, `CMD`, `EXPOSE`
- Многоэтапная сборка (multi-stage build) — как уменьшить финальный образ Go-приложения
- Основные команды: `docker build`, `docker run`, `docker ps`, `docker stop`, `docker logs`, `docker exec`
- Переменные окружения в контейнере: `ENV` в Dockerfile, `-e` при запуске, `.env`-файл
- Что такое docker-compose и зачем он нужен — запуск нескольких сервисов одной командой
- Структура `docker-compose.yml`: services, ports, volumes, environment, depends_on
- Volumes — как сохранять данные между перезапусками контейнера (например, данные PostgreSQL)
- Сети в docker-compose: как контейнеры общаются между собой по имени сервиса

---

**Docker + docker-compose — контейнеры для Go-разработчика**
Видео: https://youtu.be/MNyNxloZR0k?t=27996 (до ~9:49:00)

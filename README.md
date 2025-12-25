Отчёт по практической работе: Дополнительные возможности Docker
Цель работы
Изучение дополнительных возможностей Docker для управления жизненным циклом контейнеров, мониторинга их состояния и оптимизации использования системных ресурсов.

Содержание
Вывод логов контейнера в файл

Проверка docker-stats

Ограничение контейнера по CPU и Memory

Сохранение docker-контейнера в tar

Загрузка контейнера из tar

Структура проекта

Выводы

1. Вывод логов контейнера в файл
Цель
Научиться сохранять логи контейнеров в файлы для последующего анализа.

Выполненные действия
Создан Docker-образ practice-image:1.0

Запущен контейнер practice-container-1 на 30 секунд

Логи сохранены в файл /tmp/container_logs.txt

Проведен анализ полученных логов

Команды
bash
docker build -t practice-image:1.0 .
docker run -d --name practice-container-1 practice-image:1.0 30
docker logs practice-container-1 > /tmp/container_logs.txt
Результаты
Скриншот 1: Содержимое файла container_logs.txt

Скриншот 2: Размер и количество строк в логах

2. Проверка docker-stats
Цель
Освоить мониторинг потребления ресурсов контейнерами в реальном времени.

Выполненные действия
Запущен контейнер practice-container-2 на 45 секунд

Использована команда docker stats для мониторинга

Статистика сохранена в файл /tmp/container_stats.txt

Выполнено несколько снимков статистики с интервалами

Команды
bash
docker run -d --name practice-container-2 practice-image:1.0 45
docker stats --no-stream practice-container-2
docker stats --no-stream practice-container-2 > /tmp/container_stats.txt
Результаты
Скриншот 3: Вывод команды docker stats --no-stream

Скриншот 4: Содержимое файла container_stats.txt

Скриншот 5: Многократная статистика с интервалами

3. Ограничение контейнера по CPU и Memory
Цель
Научиться устанавливать и изменять ограничения ресурсов для контейнеров.

Выполненные действия
Запущен контейнер с ограничениями: 256MB RAM, 0.5 CPU

Проверены установленные ограничения через docker inspect

Лимиты обновлены на лету до 512MB RAM

Отслежено влияние ограничений на статистику контейнера

Команды
bash
docker run -d --name practice-limited --memory=256m --cpus=0.5 practice-image:1.0 45
docker inspect practice-limited --format='{{.HostConfig.Memory}}'
docker update --memory=512m practice-limited
Результаты
Скриншот 6: Конфигурация ограничений через docker inspect

Скриншот 7: Статистика контейнера с ограничениями

Скриншот 8: Обновление ограничений через docker update

4. Сохранение docker-контейнера в tar
Цель
Освоить механизм экспорта контейнеров в архивные файлы.

Выполненные действия
Запущен контейнер practice-export

Выполнен экспорт файловой системы в tar-архив

Проверен размер и содержимое архива

Проанализирована структура экспортированных файлов

Команды
bash
docker run -d --name practice-export practice-image:1.0 20
docker export practice-export > /tmp/container_export.tar
tar -tf /tmp/container_export.tar | head -20
Результаты
Скриншот 9: Размер tar-архива

Скриншот 10: Содержимое архива (первые 20 файлов)

Скриншот 11: Структура каталогов в архиве

5. Загрузка контейнера из tar
Цель
Научиться восстанавливать контейнеры из архивных файлов.

Выполненные действия
Импортирован tar-архив как Docker-образ

Создан исправленный Dockerfile для восстановления метаданных

Протестирован восстановленный образ

Выявлены особенности импорта/экспорта метаданных

Команды
bash
docker import /tmp/container_export.tar restored-practice:1.0
docker build -f Dockerfile.restored -t restored-practice-fixed:1.0 .
docker run --rm restored-practice-fixed:1.0 10
Результаты
Скриншот 12: Список образов после импорта

Скриншот 13: Тестирование восстановленного образа

Скриншот 14: Ошибка при неправильном импорте (опционально)

6. Структура проекта
Файлы
text
docker-practice/
├── Dockerfile                    # Основной Dockerfile
├── Dockerfile.restored          # Dockerfile для восстановленного образа
├── entrypoint.sh                # Скрипт точки входа
└── README.md                    # Данный отчёт
Dockerfile
dockerfile
FROM alpine:3.18
RUN apk add --no-cache bash curl htop procps
RUN addgroup -g 1000 appuser && adduser -D -u 1000 -G appuser appuser
WORKDIR /home/appuser
COPY --chown=appuser:appuser entrypoint.sh /home/appuser/
RUN chmod +x /home/appuser/entrypoint.sh
USER appuser
ENTRYPOINT ["/home/appuser/entrypoint.sh"]
CMD ["60"]
entrypoint.sh
bash
#!/bin/bash
DURATION=${1:-60}
echo "====== Docker Practice Container ======"
# ... логика работы контейнера ...
7. Выводы
Основные результаты
Логирование: Освоен механизм сохранения логов контейнеров в файлы для последующего анализа и архивирования

Мониторинг: Изучены возможности real-time мониторинга потребления ресурсов через docker stats

Ограничение ресурсов: Приобретены навыки установки и динамического изменения ограничений CPU и памяти

Экспорт/импорт: Освоены методы миграции контейнеров через tar-архивы с учётом особенностей метаданных

Технические особенности
Экспорт контейнера сохраняет только файловую систему, без метаданных Docker

Для корректного восстановления требуется явное указание ENTRYPOINT/CMD

Ограничения ресурсов можно изменять без перезапуска контейнера

Команда docker stats предоставляет детальную информацию о потреблении ресурсов

Практическая значимость
Полученные навыки позволяют:

Эффективно мониторить production-окружения

Оптимизировать распределение ресурсов между контейнерами

Создавать резервные копии и мигрировать контейнеры между системами

Проводить анализ проблем через логи контейнеров

Приложение: Полезные команды Docker
Мониторинг и логи
bash
docker logs <container>          # Просмотр логов
docker logs --tail N <container> # Последние N строк
docker logs -f <container>       # Логи в реальном времени
docker stats                     # Статистика всех контейнеров
docker stats <container>         # Статистика конкретного контейнера
Управление ресурсами
bash
docker run --memory=512m --cpus=1.5 <image>
docker update --memory=1g <container>
docker inspect <container>       # Подробная информация
Экспорт/импорт
bash
docker export <container> > backup.tar
docker import backup.tar <image:tag>
docker save <image> > image.tar  # Сохранение образа
docker load < image.tar          # Загрузка образа
Очистка
bash
docker system prune -f          # Очистка неиспользуемых данных
docker rm $(docker ps -aq)      # Удаление всех контейнеров
docker rmi $(docker images -q)  # Удаление всех образов
Отчёт подготовлен в рамках практической работы по изучению дополнительных возможностей Docker.
Дата выполнения: [УКАЖИТЕ ДАТУ]
Выполнил: [ВАШЕ ИМЯ]

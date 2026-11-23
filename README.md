<div align="center">

# 🖥 Kittygram & Taski · деплой на сервер

**Ручной production-деплой двух веб-приложений на один сервер Ubuntu: Gunicorn как systemd-сервис, Nginx и HTTPS от Let's Encrypt**

![Ubuntu](https://img.shields.io/badge/Ubuntu-server-E95420?logo=ubuntu&logoColor=white)
![Nginx](https://img.shields.io/badge/Nginx-reverse_proxy-009639?logo=nginx&logoColor=white)
![Gunicorn](https://img.shields.io/badge/Gunicorn-systemd-499848?logo=gunicorn&logoColor=white)
![Let's Encrypt](https://img.shields.io/badge/Let's_Encrypt-HTTPS-003A70?logo=letsencrypt&logoColor=white)
![Django](https://img.shields.io/badge/Django-DRF-092E20?logo=django&logoColor=white)

</div>

> Следующий шаг этого проекта — контейнеризация и автоматический CI/CD: **[kittygram_final](https://github.com/JuliaDJ1/kittygram_final)**.

## О проекте

Два приложения — **Kittygram** (Django REST API + React) и **Taski** — были развёрнуты на одном облачном сервере без Docker, «руками», как это делается на классическом VPS.

## Что сделано

- Бэкенд запущен через **Gunicorn** (3 воркера) как systemd-сервис с автоперезапуском — [`infra/gunicorn_kittygram.service`](infra/gunicorn_kittygram.service).
- **Nginx** обслуживает оба проекта по разным доменам: проксирует API и админку на Gunicorn, раздаёт собранный React-фронтенд, статику и медиа — [`infra/default`](infra/default).
- Получены сертификаты **Let's Encrypt** (certbot), настроен редирект HTTP → HTTPS для обоих доменов.
- `SECRET_KEY` и параметры окружения Django вынесены в переменные окружения.
- Фронтенд на React собран в production-сборку и отдаётся Nginx как статика.

## Архитектура

```
                     ┌──────────── Ubuntu server ────────────┐
 HTTPS :443  ──►  Nginx ─┬─ kittygram-домен ─► /api/, /admin/ ─► Gunicorn :8080 ─► Django (Kittygram)
                         │                    └─ /, /static/, /media/ ─► /var/www/kittygram
                         └─ taski-домен ─────► Gunicorn :8000 ─► Django (Taski)
 HTTP  :80   ──►  301 → HTTPS
```

## Стек

Ubuntu · Nginx · Gunicorn · systemd · Let's Encrypt / certbot · Python · Django · Django REST Framework · React

## Локальный запуск бэкенда Kittygram

**Требования:** Python 3.9–3.10 (версии зависимостей закреплены под них).

```bash
git clone https://github.com/JuliaDJ1/infra_sprint1.git
cd infra_sprint1
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install -r requirements.txt

cd backend
python manage.py migrate
python manage.py runserver
```

## Деплой на сервер (кратко)

```bash
# на сервере
sudo apt install nginx python3-venv
# бэкенд: venv, зависимости, migrate, collectstatic
sudo cp infra/gunicorn_kittygram.service /etc/systemd/system/
sudo systemctl enable --now gunicorn_kittygram
# фронтенд: npm run build и копирование сборки в /var/www/kittygram
sudo cp infra/default /etc/nginx/sites-enabled/default
sudo nginx -t && sudo systemctl reload nginx
sudo certbot --nginx
```

## Автор

**Вадим Гусейнов** — Python-разработчик (Backend) · [Telegram](https://t.me/eviliing)

Давайте настроим бэкенд на Django, фронтенд на Next.js, а затем интегрируем их с помощью `django-nextjs`.

Вот подробное руководство со всеми командами терминала и результирующей структурой файлов/папок.

**Предварительные требования:**

  * **Python 3:** Убедитесь, что у вас установлен Python 3. Проверить можно командой `python3 --version`.
  * **Node.js и npm/yarn:** Убедитесь, что у вас установлен Node.js и менеджер пакетов (npm или yarn). Проверить можно командами `node -v` и `npm -v` (или `yarn -v`).

-----

## 1\. Настройка проекта: Создание корневой директории

Сначала создадим основную папку проекта, которая будет содержать как бэкенд на Django, так и фронтенд на Next.js.

```bash
# Создаем основную директорию проекта
mkdir my_fullstack_project
cd my_fullstack_project
```

-----

## 2\. Настройка бэкенда Django

Мы создадим папку `backend` и настроим в ней Django.

```bash
# Переходим в директорию backend (сначала создаем ее, если она не существует)
mkdir backend
cd backend

# Создаем виртуальное окружение Python
python3 -m venv venv

# Активируем виртуальное окружение
# Для macOS/Linux:
source venv/bin/activate
# Для Windows:
# venv\Scripts\activate

# Устанавливаем Django
pip install Django

# Создаем новый Django-проект с именем 'myproject' внутри папки 'backend'.
# Точка '.' в конце гарантирует, что manage.py будет находиться непосредственно в 'backend'.
django-admin startproject myproject .

# Выполняем начальные миграции (создает схему базы данных)
python manage.py migrate

# Создаем суперпользователя (необязательно, но полезно для доступа к админке)
# Следуйте инструкциям, чтобы создать имя пользователя, email и пароль
python manage.py createsuperuser

# Деактивируем виртуальное окружение (вы активируете его позже при работе с Django)
deactivate

# Возвращаемся в корень вашего fullstack-проекта
cd ..
```

-----

## 3\. Настройка фронтенда Next.js

Теперь давайте создадим папку `frontend` и настроим в ней Next.js.

```bash
# Переходим в директорию frontend (сначала создаем ее, если она не существует)
mkdir frontend
cd frontend

# Создаем новое Next.js-приложение с помощью create-next-app
# Мы будем использовать TypeScript и ESLint, а также директорию 'src'.
# Откажитесь от App Router для более простой начальной настройки, если вы с ним не знакомы.
# Согласитесь на Tailwind CSS для стандартной практики.
# Согласитесь на 'import alias @/*'
npx create-next-app@latest . --typescript --eslint --src-dir --use-npm --tailwind --no-app

# --- Во время запроса npx create-next-app выберите следующие опции: ---
# Would you like to use TypeScript? Yes
# Would you like to use ESLint? Yes
# Would you like to use Tailwind CSS? Yes
# Would you like to use `src/` directory? Yes
# Would you like to use App Router? No  <-- Важно для простоты этого руководства
# Would you like to customize the default import alias (@/*)? Yes

# Устанавливаем любые дополнительные зависимости (если вы пропустили некоторые при создании, это гарантирует, что все они есть)
npm install

# Вы можете запустить сервер разработки Next.js, чтобы протестировать его (необязательно)
# npm run dev
# Откройте http://localhost:3000 в браузере. Нажмите Ctrl+C, чтобы остановить.

# Возвращаемся в корень вашего fullstack-проекта
cd ..
```

-----

## 4\. Установка `django-nextjs` и интеграция

Теперь мы установим `django-nextjs` в наш Django-проект и настроим его.

```bash
# Возвращаемся в директорию backend
cd backend

# Активируем виртуальное окружение Django
# Для macOS/Linux:
source venv/bin/activate
# Для Windows:
# venv\Scripts\activate

# Устанавливаем пакет django-nextjs
pip install django-nextjs

# --- Теперь мы настроим Django для использования django-nextjs ---

# Откройте 'myproject/settings.py' в вашем любимом текстовом редакторе
# (например, code myproject/settings.py, если вы используете VS Code, или nano myproject/settings.py)
# Добавьте 'django_nextjs' в INSTALLED_APPS

# myproject/settings.py (добавьте в INSTALLED_APPS)
INSTALLED_APPS = [
    # ... другие приложения Django
    'django_nextjs', # Добавьте эту строку
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]

# Добавьте специфические настройки Next.js в конец settings.py
# myproject/settings.py (добавьте эти строки внизу)
NPM_BIN_PATH = 'npm' # Или 'yarn', если вы используете Yarn

# Указываем Django, где находится 'package.json' вашего Next.js-проекта
NEXTJS_ROOT_DIR = os.path.join(BASE_DIR, '../frontend')

# Необязательно: Отключить прокси-сервер разработки Next.js в продакшене, или если вы запускаете Next.js отдельно
# NEXTJS_NO_DEV_SERVER = os.getenv('NEXTJS_NO_DEV_SERVER', '0') == '1'

# Откройте 'myproject/urls.py' в вашем текстовом редакторе
# (например, code myproject/urls.py)
# Добавьте URL-адреса django_nextjs. Обычно они размещаются в самом конце, чтобы перехватывать все несопоставленные URL-адреса.

# myproject/urls.py
from django.contrib import admin
from django.urls import path, include # Убедитесь, что include импортирован

urlpatterns = [
    path('admin/', admin.site.urls),
    # Добавьте здесь свои API или другие специфичные для Django URL-адреса, например:
    # path('api/', include('myapp.urls')),

    # Эта строка ДОЛЖНА быть последней записью в urlpatterns
    path('', include('django_nextjs.urls')), # Это перехватывает все остальные URL-адреса и проксирует их в Next.js
]

# Снова запустите миграции Django (django-nextjs может иметь свои собственные модели, хотя обычно это не требуется)
python manage.py migrate

# Деактивируем виртуальное окружение
deactivate

# Возвращаемся в корень вашего fullstack-проекта
cd ..
```

-----

## 5\. Запуск интегрированного проекта

Чтобы запустить ваш fullstack-проект, вы обычно запускаете сервер разработки Django и сервер разработки Next.js одновременно. `django-nextjs` будет заниматься проксированием.

```bash
# Откройте две отдельные вкладки/окна терминала.

# --- Терминал 1: Запуск бэкенда Django ---
cd backend
source venv/bin/activate # Активируем виртуальное окружение
python manage.py runserver

# --- Терминал 2: Запуск фронтенда Next.js ---
cd frontend
npm run dev # Или 'yarn dev', если вы используете Yarn
```

Теперь откройте ваш браузер и перейдите по адресу `http://localhost:8000`. Django получит запрос, `django-nextjs` определит, что это маршрут фронтенда, и проксирует запрос на сервер разработки Next.js, работающий по адресу `http://localhost:3000`. Вы должны увидеть приветственную страницу Next.js\!

-----

## Результирующая структура файлов и папок

После выполнения всех этих шагов структура вашего проекта будет выглядеть следующим образом:

```
my_fullstack_project/
├── backend/                                # Корень вашего Django-проекта
│   ├── venv/                               # Виртуальное окружение Python
│   │   └── ...                             # (различные файлы для окружения)
│   ├── manage.py                           # Утилита командной строки Django
│   └── myproject/                          # Основной пакет вашего Django-проекта
│       ├── __init__.py
│       ├── asgi.py
│       ├── settings.py                     # Настроены для django_nextjs
│       ├── urls.py                         # Настроены для django_nextjs
│       ├── wsgi.py
│       └── __pycache__/
├── frontend/                               # Корень вашего Next.js-проекта
│   ├── node_modules/                       # Зависимости Node.js
│   ├── public/                             # Статические ресурсы, обслуживаемые напрямую
│   │   └── ...
│   ├── src/                                # Исходная директория для Next.js-приложения
│   │   ├── app/                            # (Если используется App Router - мы сказали "нет")
│   │   ├── pages/                          # Страницы для вашего Next.js-приложения (по умолчанию с Pages Router)
│   │   │   ├── _app.tsx
│   │   │   ├── _document.tsx
│   │   │   └── index.tsx                   # Ваша основная страница Next.js
│   │   ├── styles/                         # Глобальные стили (например, globals.css)
│   │   │   └── globals.css
│   │   └── ...                             # Другие компоненты, библиотеки и т.д.
│   ├── .eslintrc.json                      # Конфигурация ESLint
│   ├── .gitattributes                      # Атрибуты Git (если инициализирован Git)
│   ├── .gitignore                          # Правила игнорирования Git
│   ├── next.config.mjs                     # Конфигурация Next.js
│   ├── package.json                        # Метаданные и зависимости проекта Node.js
│   ├── package-lock.json                   # Файл блокировки npm
│   ├── postcss.config.js                   # Конфигурация PostCSS
│   ├── README.md
│   ├── tailwind.config.ts                  # Конфигурация Tailwind CSS
│   └── tsconfig.json                       # Конфигурация TypeScript
└── .git/                                   # (Необязательно) Если вы здесь инициализировали репозиторий Git
    └── ...
```

Эта настройка обеспечивает чистое разделение вашего бэкенда и фронтенда, в то время как `django-nextjs` действует как мост во время разработки и может помочь при развертывании в продакшене.

Есть ли что-то ещё, о чем вы хотели бы узнать об этой настройке?

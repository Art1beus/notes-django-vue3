# Заметки (Notes App). Руководство программиста.
## Описание проекта
  ### Цель проекта
  Целью проекта было создание полноценного и функционирующего сайта для создания заметок с множественным функционалом.
  Реализован сайт с применением frontend-разработки и backend-разработк.
  ### Целевая аудитория
  Целевой аудиторией являются студенты, представители IT-профессий, люди с творческими хобби и предприниматели.
  Всем указанным группам в процессе этих деятельности понадобится записать и хранить важную информацию как памятку.
  ### Функционал
  Сайт реализовал следующие функции:
  - Регистрация
  - Система пользователей
  - Создание заметок с названием и описанием
  - Редактирование заметок
  - Удаление заметок
## Разработка
  ### Среда разработки
  Сайт и функционал разрабатывался в среде Visual Studio Code.
  Данный редактор выбран как наиболее популярная и удобная среда разработки.
  ### Backend
  Backend-разработка велась на Python, с применением фреймворка Django и сопутствующих библиотеки Django Rest Framework и приложения django-cors-headers.
  
  **Команды:**
  1. `mkdir backend` - создать папку backend'а
  2. `pip install django` - устанавливаем django
  3. `django-admin startproject backend` - создание проекта
  4. `django-admin startapp notes` - создание приложения
  5. `cd backend` - переход в папку проекта
  6. `python manage.py makemigrations` - создание миграций
  7. `python manage.py migrate` - выполнение миграции
  8. `python manage.py runserver` - запуск сервера

  **Код:**

  backend > settings.py > INSTALLED_APPS
  * `rest_framework` - основной каркас приложения
  * `rest_framework_simplejwt` - модуль аутентификации
  * `corsheaders` - инструмент для настройки политики CORS
  * `notes` - основное приложение


  backend > modules.py
  ```
  from django.db import models
  from django.contrib.auth.models import User
  
  class Note(models.Model):
      title = models.CharField('Заголовок', max_length=200)
      content = models.TextField('Содержание', blank=True)
      created_at = models.DateTimeField('Дата создания', auto_now_add=True)
      updated_at = models.DateTimeField('Дата обновления', auto_now=True)
      user = models.ForeignKey(
          User,
          on_delete=models.CASCADE,
          related_name='notes',
          verbose_name='Автор'
      )
      
      class Meta:
          ordering = ['-created_at']
          verbose_name = 'Заметка'
          verbose_name_plural = 'Заметки'
      
      def __str__(self):
          return f"{self.title} ({self.user.username})"
  ```
  Описание:
  
  

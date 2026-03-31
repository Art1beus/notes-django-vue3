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

---

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

  **Описание:**
  
  Модель Note — это ядро приложения, описывающее структуру хранения заметок в базе данных. Она включает заголовок (title), текстовое поле для содержания (content), а также автоматические отметки времени создания и изменения. Связь ForeignKey с моделью User обеспечивает   многопользовательский режим. Каждая заметка жестко привязана к своему автору, и при удалении аккаунта пользователя его данные стираются автоматически (каскадно).
  Внутренний class Meta управляет поведением модели на уровне всей таблицы: он задает человекочитаемые названия для админ-панели («Заметка» / «Заметки») и устанавливает стандартную сортировку, благодаря которой новые записи всегда отображаются вверху списка.
  Метод __str__ определяет текстовое представление объекта, выводя заголовок вместе с именем автора, что упрощает идентификацию конкретной записи при отладке или просмотре списка в панели управления.
  
---
  
  backend > serializers.py
  ```
  from rest_framework import serializers
  from django.contrib.auth.models import User
  from .models import Note

  class UserSerializer(serializers.ModelSerializer):
      class Meta:
          model = User
          fields = ('id', 'username', 'email', 'first_name', 'last_name')

  class NoteSerializer(serializers.ModelSerializer):
      user = UserSerializer(read_only=True)
      
      class Meta:
          model = Note
          fields = ('id', 'title', 'content', 'created_at', 'updated_at', 'user')
          read_only_fields = ('id', 'created_at', 'updated_at', 'user')
  
  class NoteCreateSerializer(serializers.ModelSerializer):
      class Meta:
          model = Note
          fields = ('title', 'content')
      
      def create(self, validated_data):
          user = self.context['request'].user
          return Note.objects.create(user=user, **validated_data)
   ```

**Описание:**



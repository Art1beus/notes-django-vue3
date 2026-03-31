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

  notes > modules.py
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
  
  Модель Note — это ядро приложения, описывающее структуру хранения заметок в базе данных. Она включает заголовок (title), текстовое поле для содержания (content), а также автоматические отметки времени создания и изменения. Связь ForeignKey с моделью User
  обеспечивает   многопользовательский режим. Каждая заметка жестко привязана к своему автору, и при удалении аккаунта пользователя его данные стираются автоматически (каскадно).
  Внутренний class Meta управляет поведением модели на уровне всей таблицы: он задает человекочитаемые названия для админ-панели («Заметка» / «Заметки») и устанавливает стандартную сортировку, благодаря которой новые записи всегда отображаются вверху списка.
  Метод __str__ определяет текстовое представление объекта, выводя заголовок вместе с именем автора, что упрощает идентификацию конкретной записи при отладке или просмотре списка в панели управления.
  
---
  
  notes > serializers.py
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

  Слой сериализаторов обеспечивает преобразование моделей Django в формат JSON для обмена данными через API. 
  UserSerializer отвечает за базовое представление данных автора (ID, логин, email, имя и фамилия), предоставляя необходимый контекст о пользователе. 
  Основной NoteSerializer формирует полную карточку заметки, включая вложенную информацию об авторе (через UserSerializer) и автоматически вычисляемые поля времени, которые доступны только для чтения (read_only).
  Специализированный NoteCreateSerializer оптимизирован для процесса создания новых записей. Он ограничивает ввод пользователя только заголовком и содержанием, гарантируя безопасность остальных данных. Внутренняя логика метода create автоматически извлекает текущего     авторизованного пользователя из контекста запроса (request.user), исключая возможность подмены автора при сохранении заметки в базу данных.

---

notes > views.py
```
from rest_framework import generics, permissions, status
from rest_framework.response import Response
from rest_framework.views import APIView
from django.contrib.auth.models import User
from .models import Note
from .serializers import NoteSerializer, NoteCreateSerializer, UserSerializer

class IsOwner(permissions.BasePermission):
    """Разрешение: только владелец может редактировать/удалять"""
    
    def has_object_permission(self, request, view, obj):
        if request.method in permissions.SAFE_METHODS:
            return True
        return obj.user == request.user

class RegisterView(APIView):
    permission_classes = [permissions.AllowAny]
    
    def post(self, request):
        username = request.data.get('username')
        password = request.data.get('password')
        email = request.data.get('email', '')
        
        if not username or not password:
            return Response(
                {'error': 'Необходимо указать username и password'},
                status=status.HTTP_400_BAD_REQUEST
            )
        
        if User.objects.filter(username=username).exists():
            return Response(
                {'error': 'Пользователь с таким именем уже существует'},
                status=status.HTTP_400_BAD_REQUEST
            )
        
        user = User.objects.create_user(
            username=username,
            password=password,
            email=email
        )
        
        return Response(
            UserSerializer(user).data,
            status=status.HTTP_201_CREATED
        )

class UserProfileView(APIView):
    permission_classes = [permissions.IsAuthenticated]
    
    def get(self, request):
        return Response(UserSerializer(request.user).data)

class NoteListCreateView(generics.ListCreateAPIView):
    permission_classes = [permissions.IsAuthenticated]
    
    def get_queryset(self):
        return Note.objects.filter(user=self.request.user)
    
    def get_serializer_class(self):
        if self.request.method == 'POST':
            return NoteCreateSerializer
        return NoteSerializer

class NoteDetailView(generics.RetrieveUpdateDestroyAPIView):
    permission_classes = [permissions.IsAuthenticated, IsOwner]
    serializer_class = NoteSerializer
    
    def get_queryset(self):
        return Note.objects.filter(user=self.request.user)
```

  **Описание:**
  
  Логика представлений (Views) и прав доступа обеспечивает безопасное управление данными через API. 
  Кастомный класс IsOwner реализует строгую проверку прав, позволяя редактировать или удалять заметки только их непосредственным авторам, при этом разрешая безопасные методы чтения. 
  Система регистрации в RegisterView открыта для всех (AllowAny) и включает валидацию уникальности имени пользователя, а UserProfileView предоставляет авторизованным участникам доступ к данным их собственного профиля.
  Работа с заметками разделена на два ключевых контроллера. NoteListCreateView отвечает за вывод списка записей текущего пользователя и создание новых, динамически переключаясь между сериализаторами в зависимости от типа HTTP-запроса. Для детальных операций (просмотр,   обновление, удаление) используется NoteDetailView, который дополнительно фильтрует выборку по владельцу, гарантируя, что пользователь не сможет взаимодействовать с чужим контентом даже при прямом обращении по ID.



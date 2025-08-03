# documentation

### Руководство по написанию документации к исходному коду веб-приложения

Документация к дипломной работе — это не просто формальность, а **ключ к пониманию вашего кода** преподавателями, рецензентами и потенциальными работодателями. Её цель — показать, что вы осознанно проектировали систему, понимаете каждую деталь реализации и можете объяснить это другим. Вот на чём стоит акцентировать внимание:

#### 1. Структура — основа понятности
Разбейте документацию на логические блоки:
- **Введение**: Кратко опишите назначение приложения и его стэк технологий.
- **Архитектура**: Объясните структуру проекта (папки, ключевые модули).
- **Модели данных**: Как устроена база данных? Какие сущности и связи?
- **Функциональные компоненты**: Пошагово разберите ключевые фичи (регистрация, оплата и т.д.).
- **Запуск**: Как установить и запустить проект? Укажите зависимости.

#### 2. Код ≠ документация
Избегайте очевидных описаний вроде:  
❌ *«Этот метод сохраняет пользователя в БД»*.  
Вместо этого объясняйте:  
✅ *«Метод хеширует пароль через Bcrypt перед сохранением, чтобы избежать утечек в случае компрометации БД»*.

#### 3. Визуализируйте
Используйте:
- **Диаграммы БД** (например, ER-модель от dbdiagram.io)
- **Скриншоты интерфейса** (даже минимального)
- **Блок-схемы** для сложной логики (например, процесс оплаты).

#### 4. Ориентируйтесь на целевую аудиторию
Представьте, что читатель:
- Знает базовые концепции (MVC, REST), но не разбирается в вашем коде.
- Хочет быстро понять архитектуру, не вникая в каждую строку.

#### 5. Честность — ваше преимущество
Не скрывайте **ограничения проекта**:  
✅ *«Из-за сроков диплома платежная система реализована в режиме sandbox»*.  
Это покажет ваше критическое мышление.

---

### Пример документации: Веб-приложение «BookFlow» (магазин электронных книг)

#### 1. Введение
**BookFlow** — минималистичный MVP магазина электронных книг. Пользователи могут регистрироваться, просматривать каталог, добавлять книги в корзину и совершать «покупки» (в учебных целях без реальной оплаты).  

**Стек**:  
- Backend: Python + Flask  
- База данных: SQLite (для упрощения разработки)  
- Безопасность: Flask-Login + Flask-Bcrypt  
- Формы: Flask-WTF  
- ORM: Flask-SQLAlchemy  

#### 2. Структура проекта
```
bookflow/
├── app.py                 # Точка входа
├── config.py              # Конфигурация (секретный ключ, настройки БД)
├── models.py              # Модели данных
├── routes/                # Маршруты Flask
│   ├── auth.py            # Регистрация/авторизация
│   ├── books.py           # Работа с книгами
│   └── cart.py            # Корзина пользователя
├── templates/             # HTML-шаблоны (Jinja2)
├── static/                # CSS/JS
└── requirements.txt       # Зависимости
```

#### 3. Модели данных
```python
# models.py
from flask_sqlalchemy import SQLAlchemy
from flask_bcrypt import Bcrypt

db = SQLAlchemy()
bcrypt = Bcrypt()

class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    email = db.Column(db.String(120), unique=True, nullable=False)
    password_hash = db.Column(db.String(128), nullable=False)
    # Связь: один пользователь → много книг в корзине
    cart_items = db.relationship('CartItem', backref='user', lazy=True)

    def set_password(self, password):
        self.password_hash = bcrypt.generate_password_hash(password).decode('utf-8')

class Book(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(200), nullable=False)
    price = db.Column(db.Float, nullable=False)

class CartItem(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'), nullable=False)
    book_id = db.Column(db.Integer, db.ForeignKey('book.id'), nullable=False)
```
**Пояснения**:  
- `User.password_hash` хранит **хеш пароля** (Flask-Bcrypt), а не сам пароль.  
- `CartItem` — вспомогательная модель для связи пользователей и книг в корзине (many-to-many).

#### 4. Ключевые функции
**А. Регистрация пользователя**  
- Маршрут: `/register` (POST)  
- Логика:  
  1. Валидация формы (Flask-WTF)  
  2. Хеширование пароля  
  3. Сохранение в БД  
```python
# routes/auth.py
@bp.route('/register', methods=['POST'])
def register():
    form = RegistrationForm()
    if form.validate():
        user = User(email=form.email.data)
        user.set_password(form.password.data)  # Хешируем пароль!
        db.session.add(user)
        db.session.commit()
        return redirect('/login')
```

**Б. Добавление книги в корзину**  
- Маршрут: `/cart/add/<book_id>` (POST)  
- Логика:  
  1. Проверка авторизации (Flask-Login)  
  2. Создание связи `CartItem`  
```python
# routes/cart.py
@login_required
def add_to_cart(book_id):
    item = CartItem(user_id=current_user.id, book_id=book_id)
    db.session.add(item)
    db.session.commit()
    return "Книга добавлена в корзину"
```

#### 5. Безопасность
- **Сессии**: Flask-Login управляет аутентификацией через куки.  
- **CSRF-защита**: Включена глобально через Flask-WTF.  
- **Пароли**: Хешируются Bcrypt (алгоритм bcrypt + соль).  

#### 6. Запуск проекта
1. Установите зависимости:  
   ```bash
   pip install -r requirements.txt
   ```
2. Инициализируйте БД:  
   ```python
   from app import db
   db.create_all()
   ```
3. Запустите сервер:  
   ```bash
   flask run --port=5000
   ```

#### 7. Ограничения
- **Без реальной оплаты**: Для диплома реализован "режим симуляции".  
- **Нет восстановления пароля**: Требует интеграции с email-сервисом.  
- **Упрощенный интерфейс**: Bootstrap 5 для базового UI.  

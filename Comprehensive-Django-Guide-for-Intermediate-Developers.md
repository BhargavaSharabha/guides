# Comprehensive Django Guide for Intermediate Developers

## Table of Contents

1. [Introduction](#introduction)
2. [Core Django Concepts and Components](#core-django-concepts-and-components)
   2.1. [Models and Database Management](#models-and-database-management)
   2.2. [Views and URL Routing](#views-and-url-routing)
   2.3. [Templates and Template Inheritance](#templates-and-template-inheritance)
   2.4. [Forms and Form Handling](#forms-and-form-handling)
   2.5. [Authentication and Authorization](#authentication-and-authorization)
   2.6. [Middleware](#middleware)
   2.7. [Static Files and Media Handling](#static-files-and-media-handling)
   2.8. [Django ORM and Querysets](#django-orm-and-querysets)
   2.9. [Admin Interface Customization](#admin-interface-customization)
3. [Advanced Django Features and Topics](#advanced-django-features-and-topics)
   3.1. [Class-based Views](#class-based-views)
   3.2. [Custom Template Tags and Filters](#custom-template-tags-and-filters)
   3.3. [Signals and Receivers](#signals-and-receivers)
   3.4. [Caching Strategies](#caching-strategies)
   3.5. [REST Framework Integration](#rest-framework-integration)
   3.6. [Asynchronous Views and ASGI](#asynchronous-views-and-asgi)
   3.7. [Testing in Django](#testing-in-django)
   3.8. [Django Security Best Practices](#django-security-best-practices)
   3.9. [Performance Optimization Techniques](#performance-optimization-techniques)
4. [Project Tutorials](#project-tutorials)
   4.1. [E-commerce Site](#e-commerce-site)
   4.2. [Social Media Platform](#social-media-platform)
   4.3. [Content Management System](#content-management-system)
   4.4. [API-driven Web App](#api-driven-web-app)
   4.5. [Real-time Chat Application](#real-time-chat-application)
5. [Structuring Large-scale Django Projects](#structuring-large-scale-django-projects)
6. [Django Integration with Other Technologies](#django-integration-with-other-technologies)
7. [Deployment Strategies](#deployment-strategies)
8. [Common Pitfalls and How to Avoid Them](#common-pitfalls-and-how-to-avoid-them)
9. [Resources for Further Learning](#resources-for-further-learning)

## 1. Introduction

Welcome to this comprehensive guide on Django for intermediate developers. This guide covers the latest version of Django (as of 2024) and is designed to take your Django skills to the next level. Whether you're looking to deepen your understanding of core concepts, explore advanced features, or learn best practices for building large-scale applications, this guide has you covered.

Django is a high-level Python web framework that encourages rapid development and clean, pragmatic design. Built by experienced developers, it takes care of much of the hassle of web development, so you can focus on writing your app without needing to reinvent the wheel.

In this guide, we'll dive deep into Django's architecture, explore its powerful features, and learn how to leverage them to build robust, scalable web applications. We'll cover everything from the fundamentals to advanced topics, provide numerous practical examples, and walk through several complete project tutorials.

Let's begin our journey into the world of Django!

## 2. Core Django Concepts and Components

### 2.1. Models and Database Management

Models are at the heart of Django's Object-Relational Mapping (ORM) system. They provide an abstraction layer that allows you to work with your database using Python code instead of SQL.

#### Key Concepts:

1. **Model Definition**: Models are defined as Python classes that inherit from `django.db.models.Model`.
2. **Fields**: Each attribute in the model represents a database field.
3. **Migrations**: Django's way of propagating changes you make to your models into your database schema.
4. **Relationships**: Django supports various types of relationships between models (One-to-One, One-to-Many, Many-to-Many).
5. **Model Inheritance**: Django supports three types of model inheritance: abstract base classes, multi-table inheritance, and proxy models.

Let's look at an example of a model definition:

```python
from django.db import models
from django.contrib.auth.models import User

class Article(models.Model):
    title = models.CharField(max_length=200)
    content = models.TextField()
    pub_date = models.DateTimeField('date published')
    author = models.ForeignKey(User, on_delete=models.CASCADE)
    is_published = models.BooleanField(default=False)
  
    def __str__(self):
        return self.title
  
    class Meta:
        ordering = ['-pub_date']
```

In this example, we define an `Article` model with various fields and a foreign key relationship to the `User` model.

#### Working with Models:

1. **Creating Objects**:

   ```python
   article = Article(title="Django Guide", content="...", pub_date=timezone.now(), author=user)
   article.save()
   ```
2. **Querying Objects**:

   ```python
   # Get all articles
   all_articles = Article.objects.all()

   # Filter articles
   published_articles = Article.objects.filter(is_published=True)

   # Get a single article
   article = Article.objects.get(id=1)
   ```
3. **Updating Objects**:

   ```python
   article = Article.objects.get(id=1)
   article.title = "Updated Title"
   article.save()
   ```
4. **Deleting Objects**:

   ```python
   article = Article.objects.get(id=1)
   article.delete()
   ```

#### Advanced Model Features:

1. **Custom Managers**:

   ```python
   class PublishedManager(models.Manager):
       def get_queryset(self):
           return super().get_queryset().filter(is_published=True)

   class Article(models.Model):
       # ... fields ...
       objects = models.Manager()  # The default manager
       published = PublishedManager()  # Custom manager
   ```
2. **Abstract Base Classes**:

   ```python
   class TimeStampedModel(models.Model):
       created_at = models.DateTimeField(auto_now_add=True)
       updated_at = models.DateTimeField(auto_now=True)

       class Meta:
           abstract = True

   class Article(TimeStampedModel):
       # ... fields ...
   ```
3. **Model Methods**:

   ```python
   class Article(models.Model):
       # ... fields ...

       def get_absolute_url(self):
           return reverse('article_detail', args=[str(self.id)])

       def is_recent(self):
           return self.pub_date >= timezone.now() - datetime.timedelta(days=7)
   ```
4. **Signals**:

   ```python
   from django.db.models.signals import post_save
   from django.dispatch import receiver

   @receiver(post_save, sender=Article)
   def article_post_save(sender, instance, created, **kwargs):
       if created:
           print(f"New article created: {instance.title}")
   ```

#### Best Practices:

1. Use meaningful names for your models and fields.
2. Keep your models small and focused. If a model is getting too complex, consider breaking it into multiple models.
3. Use custom managers to encapsulate common queries.
4. Use model methods to encapsulate business logic.
5. Always run migrations after changing your models.
6. Use Django's built-in field types whenever possible.
7. Be cautious with `ForeignKey` and `ManyToManyField` relationships to avoid circular imports.

Understanding models and database management is crucial for building efficient Django applications. In the next section, we'll explore Views and URL routing, which will help us understand how to handle HTTP requests and responses in Django.

### 2.2. Views and URL Routing

Views and URL routing are fundamental concepts in Django that handle the logic of processing requests and returning responses. They form the 'Controller' part of Django's MTV (Model-Template-View) architecture.

#### Views

A view in Django is a Python function or class that takes a web request and returns a web response. Views can do anything from returning a simple HTML document to processing data and creating, updating, or deleting records in the database.

Let's start with a simple function-based view:

```python
from django.http import HttpResponse
from django.shortcuts import render
from .models import Article

def article_list(request):
    articles = Article.objects.all()
    return render(request, 'articles/list.html', {'articles': articles})

def article_detail(request, article_id):
    article = Article.objects.get(id=article_id)
    return render(request, 'articles/detail.html', {'article': article})
```

In these examples, `article_list` fetches all articles and renders them using a template, while `article_detail` fetches a specific article based on its ID.

#### URL Routing

URL routing in Django is handled by URLconf (URL configuration). It's a mapping between URL patterns and the views that should be called for those URLs. URLconfs are defined in a file typically named `urls.py`.

Here's an example of a `urls.py` file:

```python
from django.urls import path
from . import views

urlpatterns = [
    path('articles/', views.article_list, name='article_list'),
    path('articles/<int:article_id>/', views.article_detail, name='article_detail'),
]
```

In this example, we've defined two URL patterns. The first one maps to the `article_list` view, and the second one maps to the `article_detail` view, passing the `article_id` as a parameter.

#### Class-Based Views

Django also supports class-based views, which offer a way to organize view-related code in a more reusable and extensible manner. Here's an example:

```python
from django.views.generic import ListView, DetailView
from .models import Article

class ArticleListView(ListView):
    model = Article
    template_name = 'articles/list.html'
    context_object_name = 'articles'

class ArticleDetailView(DetailView):
    model = Article
    template_name = 'articles/detail.html'
    context_object_name = 'article'
```

And the corresponding URL configuration:

```python
from django.urls import path
from .views import ArticleListView, ArticleDetailView

urlpatterns = [
    path('articles/', ArticleListView.as_view(), name='article_list'),
    path('articles/<int:pk>/', ArticleDetailView.as_view(), name='article_detail'),
]
```

#### Advanced URL Routing

Django's URL routing system is quite powerful and flexible. Here are some advanced features:

1. **URL Parameters**:

   ```python
   path('articles/<int:year>/<int:month>/', views.month_archive),
   ```
2. **Regular Expressions**:

   ```python
   re_path(r'^articles/(?P<year>[0-9]{4})/$', views.year_archive),
   ```
3. **Naming URL Patterns**:

   ```python
   path('articles/', views.article_list, name='article_list'),
   ```
4. **Including Other URLconfs**:

   ```python
   from django.urls import include, path

   urlpatterns = [
       path('blog/', include('blog.urls')),
   ]
   ```
5. **Reverse URL Resolution**:
   In Python:

   ```python
   from django.urls import reverse
   url = reverse('article_detail', args=[article.id])
   ```

   In templates:

   ```html
   <a href="{% url 'article_detail' article.id %}">{{ article.title }}</a>
   ```

#### Best Practices:

1. Keep your views focused on a single task.
2. Use class-based views for common patterns (list views, detail views, etc.).
3. Use meaningful names for your URL patterns.
4. Group related URLs in separate `urls.py` files and include them in the main URLconf.
5. Use reverse URL resolution instead of hardcoding URLs.
6. Handle exceptions in your views to provide meaningful error messages.

Understanding views and URL routing is crucial for building Django applications. They form the core of how your application handles requests and generates responses. In the next section, we'll explore Templates and Template Inheritance, which will help us understand how to structure and render our HTML content.

### 2.3. Templates and Template Inheritance

Django's template system provides a powerful way to generate dynamic HTML pages. It allows you to define the structure of your output document, with placeholders for data that will be filled in when the template is rendered.

#### Basic Template Usage

Here's a simple example of a Django template:

```html
<!DOCTYPE html>
<html>
<head>
    <title>{{ article.title }}</title>
</head>
<body>
    <h1>{{ article.title }}</h1>
    <p>Published on: {{ article.pub_date|date:"F j, Y" }}</p>
    <div>{{ article.content|safe }}</div>
</body>
</html>
```

In this example, `{{ }}` is used for variable output, and `|` is used for applying template filters.

#### Template Tags

Django provides various built-in template tags that allow you to add logic to your templates:

1. **For loops**:

   ```html
   <ul>
   {% for article in articles %}
       <li>{{ article.title }}</li>
   {% empty %}
       <li>No articles found.</li>
   {% endfor %}
   </ul>
   ```
2. **If statements**:

   ```html
   {% if user.is_authenticated %}
       <p>Welcome, {{ user.username }}!</p>
   {% else %}
       <p>Please log in.</p>
   {% endif %}
   ```
3. **URL tag**:

   ```html
   <a href="{% url 'article_detail' article.id %}">{{ article.title }}</a>
   ```
4. **Static files**:

   ```html
   {% load static %}
   <img src="{% static 'images/logo.png' %}" alt="Logo">
   ```

#### Template Inheritance

Template inheritance is a powerful feature that allows you to build a base "skeleton" template that contains all the common elements of your site and defines blocks that child templates can override.

Here's an example of a base template:

```html
<!-- base.html -->
<!DOCTYPE html>
<html>
<head>
    <title>{% block title %}My Site{% endblock %}</title>
    {% block extra_head %}{% endblock %}
</head>
<body>
    <header>
        {% block header %}
        <h1>My Site</h1>
        <nav>
            <ul>
                <li><a href="{% url 'home' %}">Home</a></li>
                <li><a href="{% url 'about' %}">About</a></li>
                <li><a href="{% url 'contact' %}">Contact</a></li>
            </ul>
        </nav>
        {% endblock %}
    </header>
  
    <main>
        {% block content %}
        {% endblock %}
    </main>
  
    <footer>
        {% block footer %}
        <p>© 2024 My Site</p>
        {% endblock %}
    </footer>
</body>
</html>
```

And here's how a child template would extend and override blocks from the base template:

```html
<!-- article_detail.html -->
{% extends "base.html" %}

{% block title %}{{ article.title }} | My Site{% endblock %}

{% block extra_head %}
<meta name="description" content="{{ article.summary }}">
{% endblock %}

{% block content %}
<article>
    <h1>{{
```

article.title }}`</h1>`
    `<p>`Published on: {{ article.pub_date|date:"F j, Y" }}`</p>`
    `<div>`{{ article.content|safe }}`</div>`

</article>
{% endblock %}
```

In this example, the child template extends the base template and overrides specific blocks to customize the page content.

#### Custom Template Tags and Filters

Django allows you to create custom template tags and filters to extend the template language's functionality. Here's a simple example of a custom template filter:

```python
# In your_app/templatetags/custom_filters.py
from django import template

register = template.Library()

@register.filter(name='cut')
def cut(value, arg):
    """Removes all values of arg from the given string"""
    return value.replace(arg, '')
```

To use this filter in a template:

```html
{% load custom_filters %}
{{ somevariable|cut:"0" }}
```

#### Best Practices for Templates:

1. Use template inheritance to reduce code duplication.
2. Keep your templates DRY (Don't Repeat Yourself).
3. Use meaningful names for your blocks.
4. Minimize logic in templates. Complex operations should be done in views or model methods.
5. Use the `{% csrf_token %}` tag in all forms to prevent Cross-Site Request Forgery.
6. Use the `{% static %}` tag for referencing static files.
7. Use template tags like `{% url %}` instead of hardcoding URLs.

### 2.4. Forms and Form Handling

Django provides a powerful form library that handles rendering forms as HTML, validating user-submitted data, and converting that data to native Python types. Let's explore how to work with forms in Django.

#### Creating a Form

Here's an example of a simple form in Django:

```python
# forms.py
from django import forms
from .models import Article

class ArticleForm(forms.ModelForm):
    class Meta:
        model = Article
        fields = ['title', 'content', 'pub_date']
        widgets = {
            'pub_date': forms.DateInput(attrs={'type': 'date'}),
        }

    def clean_title(self):
        title = self.cleaned_data['title']
        if len(title) < 5:
            raise forms.ValidationError("Title must be at least 5 characters long")
        return title
```

This form is based on the `Article` model and includes custom validation for the title field.

#### Rendering Forms in Templates

To render this form in a template:

```html
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Submit</button>
</form>
```

For more control over form rendering:

```html
<form method="post">
    {% csrf_token %}
    <div>
        {{ form.title.errors }}
        <label for="{{ form.title.id_for_label }}">Title:</label>
        {{ form.title }}
    </div>
    <div>
        {{ form.content.errors }}
        <label for="{{ form.content.id_for_label }}">Content:</label>
        {{ form.content }}
    </div>
    <div>
        {{ form.pub_date.errors }}
        <label for="{{ form.pub_date.id_for_label }}">Publication Date:</label>
        {{ form.pub_date }}
    </div>
    <button type="submit">Submit</button>
</form>
```

#### Handling Forms in Views

Here's how to handle form submission in a view:

```python
from django.shortcuts import render, redirect
from .forms import ArticleForm

def create_article(request):
    if request.method == 'POST':
        form = ArticleForm(request.POST)
        if form.is_valid():
            article = form.save(commit=False)
            article.author = request.user
            article.save()
            return redirect('article_detail', article_id=article.id)
    else:
        form = ArticleForm()
    return render(request, 'create_article.html', {'form': form})
```

#### Form Sets

Django also provides FormSets, which allow you to work with multiple forms on the same page. Here's a simple example:

```python
from django.forms import formset_factory
from .forms import ArticleForm

ArticleFormSet = formset_factory(ArticleForm, extra=3)

def manage_articles(request):
    if request.method == 'POST':
        formset = ArticleFormSet(request.POST)
        if formset.is_valid():
            for form in formset:
                if form.has_changed():
                    article = form.save(commit=False)
                    article.author = request.user
                    article.save()
            return redirect('article_list')
    else:
        formset = ArticleFormSet()
    return render(request, 'manage_articles.html', {'formset': formset})
```

#### Best Practices for Forms:

1. Use ModelForms when working with models to reduce code duplication.
2. Always validate form data on the server side, even if you have client-side validation.
3. Use form cleaning methods for complex validation logic.
4. Use Django's CSRF protection for all forms.
5. Handle form errors gracefully and provide clear feedback to users.
6. Consider using Django Crispy Forms for more advanced form layouts and styling.

### 2.5. Authentication and Authorization

Django comes with a built-in authentication system that handles user accounts, groups, permissions, and cookie-based user sessions. Let's explore how to use these features effectively.

#### User Model

Django provides a default User model in `django.contrib.auth.models.User`. However, it's often beneficial to create a custom user model:

```python
# models.py
from django.contrib.auth.models import AbstractUser
from django.db import models

class CustomUser(AbstractUser):
    bio = models.TextField(blank=True)
    birth_date = models.DateField(null=True, blank=True)

# settings.py
AUTH_USER_MODEL = 'your_app.CustomUser'
```

#### Authentication Views

Django provides several authentication views out of the box. You can include them in your `urls.py`:

```python
from django.contrib.auth import views as auth_views

urlpatterns = [
    path('login/', auth_views.LoginView.as_view(), name='login'),
    path('logout/', auth_views.LogoutView.as_view(), name='logout'),
    path('password_change/', auth_views.PasswordChangeView.as_view(), name='password_change'),
    path('password_change/done/', auth_views.PasswordChangeDoneView.as_view(), name='password_change_done'),
    # ... more authentication views ...
]
```

#### Custom Authentication Views

You can also create custom authentication views:

```python
from django.contrib.auth import login, authenticate
from django.shortcuts import render, redirect
from .forms import SignUpForm

def signup(request):
    if request.method == 'POST':
        form = SignUpForm(request.POST)
        if form.is_valid():
            user = form.save()
            raw_password = form.cleaned_data.get('password1')
            user = authenticate(username=user.username, password=raw_password)
            login(request, user)
            return redirect('home')
    else:
        form = SignUpForm()
    return render(request, 'signup.html', {'form': form})
```

#### Authorization

Django's authorization system allows you to define permissions and groups. You can use decorators or mixins to restrict access to views:

```python
from django.contrib.auth.decorators import login_required, permission_required

@login_required
def profile(request):
    return render(request, 'profile.html')

@permission_required('app.add_article')
def create_article(request):
    # View logic here
```

For class-based views:

```python
from django.contrib.auth.mixins import LoginRequiredMixin, PermissionRequiredMixin

class ProfileView(LoginRequiredMixin, View):
    # View logic here

class CreateArticleView(PermissionRequiredMixin, CreateView):
    permission_required = 'app.add_article'
    # View logic here
```

#### Custom Permissions

You can define custom permissions in your models:

```python
class Article(models.Model):
    # ... fields ...

    class Meta:
        permissions = [
            ("publish_article", "Can publish article"),
        ]
```

#### Best Practices for Authentication and Authorization:

1. Always use HTTPS for authentication pages and authenticated parts of your site.
2. Use Django's built-in password hashing.
3. Implement proper password reset functionality.
4. Use Django's `@login_required` decorator or `LoginRequiredMixin` to protect views that require authentication.
5. Use object-level permissions for fine-grained access control.
6. Be cautious with permission checks in templates to avoid exposing sensitive information.
7. Use Django's messaging framework to provide feedback on authentication actions.

In the next section, we'll dive into Middleware, another crucial component of Django's request/response cycle.

### 2.6. Middleware

Middleware in Django is a framework of hooks into Django's request/response processing. It's a light, low-level "plugin" system for globally altering Django's input or output. Each middleware component is responsible for doing some specific function.

#### How Middleware Works

Middleware is executed in the order it's defined in the `MIDDLEWARE` setting. During the request phase, it's applied from top to bottom. During the response phase, it's applied from bottom to top.

Here's a simple example of a custom middleware:

```python
# middleware.py
import time

class TimingMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):
        start_time = time.time()
    
        response = self.get_response(request)
    
        duration = time.time() - start_time
        response['X-Page-Generation-Duration-ms'] = int(duration * 1000)
        return response
```

To use this middleware, add it to your `MIDDLEWARE` setting:

```python
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
    'your_app.middleware.TimingMiddleware',  # Your custom middleware
]
```

#### Types of Middleware

1. **Request Middleware**: Processes the request before it reaches the view.
2. **View Middleware**: Processes the request just before the view is called.
3. **Response Middleware**: Processes the response after the view is called.
4. **Exception Middleware**: Processes exceptions raised by the view.

#### Built-in Middleware

Django includes several built-in middleware classes:

1. `SecurityMiddleware`: Handles several security improvements for requests/responses.
2. `SessionMiddleware`: Enables session support.
3. `CommonMiddleware`: Handles common operations like setting the Content-Length header.
4. `CsrfViewMiddleware`: Adds protection against Cross Site Request Forgeries.
5. `AuthenticationMiddleware`: Associates users with requests using sessions.
6. `MessageMiddleware`: Enables cookie- and session-based message support.
7. `XFrameOptionsMiddleware`: Protects against clickjacking.

#### Writing Custom Middleware

Here's an example of a more complex middleware that logs user actions:

```python
import time
from django.utils.deprecation import MiddlewareMixin
from django.contrib.auth.models import AnonymousUser

class UserActionMiddleware(MiddlewareMixin):
    def process_request(self, request):
        request.start_time = time.time()

    def process_response(self, request, response):
        if hasattr(request, 'user') and not isinstance(request.user, AnonymousUser):
            duration = time.time() - request.start_time
            UserAction.objects.create(
                user=request.user,
                action=request.method,
                path=request.path,
                duration=duration
            )
        return response
```

#### Best Practices for Middleware:

1. Keep middleware focused on a single responsibility.
2. Be mindful of the order of middleware in the `MIDDLEWARE` setting.
3. Handle exceptions gracefully to avoid breaking the request/response cycle.
4. Use middleware for cross-cutting concerns that apply to many or all views.
5. Be cautious with performance impact, especially in request processing middleware.
6. Use Django's `MiddlewareMixin` for compatibility with both old-style and new-style middleware.

In the next section, we'll explore Static Files and Media Handling in Django, which are crucial for managing assets in your web applications.

### 2.7. Static Files and Media Handling

Django provides a robust system for handling static files (CSS, JavaScript, images) and user-uploaded media files. Let's explore how to effectively manage these assets in your Django projects.

#### Static Files

Static files are the assets that are part of your project and don't change, like CSS stylesheets, JavaScript files, and images used in your site design.

1. **Configuring Static Files**:

   In your `settings.py`:

   ```python
   STATIC_URL = '/static/'
   STATICFILES_DIRS = [BASE_DIR / 'static']
   STATIC_ROOT = BASE_DIR / 'staticfiles'
   ```
2. **Collecting Static Files**:

   Run `python manage.py collectstatic` to collect all static files into the `STATIC_ROOT` directory.
3. **Using Static Files in Templates**:

   ```html
   {% load static %}
   <link rel="stylesheet" href="{% static 'css/style.css' %}">
   <script src="{% static 'js/script.js' %}"></script>
   <img src="{% static 'images/logo.png' %}" alt="Logo">
   ```
4. **Serving Static Files During Development**:

   Django's development server automatically serves static files. For production, you'll need to configure your web server to serve files from `STATIC_ROOT`.

#### Media Files

Media files are user-uploaded files, such as profile pictures or document attachments.

1. **Configuring Media Files**:

   In your `settings.py`:

   ```python
   MEDIA_URL = '/media/'
   MEDIA_ROOT = BASE_DIR / 'media'
   ```
2. **Handling File Uploads in Models**:

   ```python
   from django.db import models

   class Document(models.Model):
       title = models.CharField(max_length=200)
       file = models.FileField(upload_to='documents/')
       uploaded_at = models.DateTimeField(auto_now_add=True)
   ```
3. **Serving Media Files During Development**:

   In your project's `urls.py`:

   ```python
   from django.conf import settings
   from django.conf.urls.static import static

   urlpatterns = [
       # ... your url patterns ...
   ]

   if settings.DEBUG:
       urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
   ```
4. **Accessing Media Files in Templates**:

   ```html
   <img src="{{ document.file.url }}" alt="{{ document.title }}">
   ```

#### File Storage

Django's default file storage system uses the local filesystem, but you can use custom storage backends for services like Amazon S3 or Google Cloud Storage.

Here's an example using django-storages with Amazon S3:

1. Install django-storages and boto3:

   ```
   pip install django-storages boto3
   ```
2. Configure S3 in `settings.py`:

   ```python
   DEFAULT_FILE_STORAGE = 'storages.backends.s3boto3.S3Boto3Storage'
   AWS_ACCESS_KEY_ID = 'your-access-key'
   AWS_SECRET_ACCESS_KEY = 'your-secret-key'
   AWS_STORAGE_BUCKET_NAME = 'your-bucket-name'
   AWS_S3_
   ```

CUSTOM_DOMAIN = 'your-bucket-name.s3.amazonaws.com'
   AWS_S3_FILE_OVERWRITE = False
   AWS_DEFAULT_ACL = None

```

#### Best Practices for Static and Media Files:

1. Use a CDN (Content Delivery Network) for serving static files in production.
2. Implement file type and size validation for user uploads.
3. Use meaningful names for uploaded files, possibly including timestamps or unique identifiers.
4. Regularly backup your media files.
5. Consider using Django's `FileField` and `ImageField` for handling file uploads in models.
6. Use Django's `collectstatic` management command to gather static files in one location.
7. In production, serve static and media files from a separate domain for better performance and security.

### 2.8. Django ORM and Querysets

Django's Object-Relational Mapping (ORM) is a powerful tool that allows you to interact with your database using Python code instead of SQL. It provides an abstraction layer between your models and the database, making it easier to work with your data.

#### Basic Querying

Here are some basic querying operations:

```python
# Get all objects
all_articles = Article.objects.all()

# Filter objects
published_articles = Article.objects.filter(status='published')

# Exclude objects
non_draft_articles = Article.objects.exclude(status='draft')

# Get a single object
article = Article.objects.get(id=1)

# Order objects
latest_articles = Article.objects.order_by('-created_at')

# Limit the number of returned objects
top_5_articles = Article.objects.all()[:5]
```

#### Complex Queries

Django ORM supports complex queries using Q objects and F objects:

```python
from django.db.models import Q, F

# OR condition
Article.objects.filter(Q(status='published') | Q(status='featured'))

# AND condition
Article.objects.filter(Q(status='published') & Q(created_at__year=2024))

# NOT condition
Article.objects.filter(~Q(status='draft'))

# F objects for comparing fields
Article.objects.filter(num_likes__gt=F('num_shares'))
```

#### Aggregation and Annotation

Django ORM provides powerful aggregation and annotation capabilities:

```python
from django.db.models import Count, Avg, Sum

# Count the number of articles per author
author_article_count = Author.objects.annotate(num_articles=Count('article'))

# Get the average number of likes for all articles
avg_likes = Article.objects.aggregate(avg_likes=Avg('num_likes'))

# Annotate each article with the sum of its likes and shares
Article.objects.annotate(total_interactions=Sum(F('num_likes') + F('num_shares')))
```

#### Relationships and Joins

Django makes it easy to work with related models:

```python
# Forward relationship (One-to-Many)
author = Author.objects.get(id=1)
author_articles = author.article_set.all()

# Reverse relationship (Many-to-One)
article = Article.objects.get(id=1)
article_author = article.author

# Many-to-Many relationship
article.tags.add(tag1, tag2)
article.tags.remove(tag1)
article.tags.clear()

# Prefetch related objects to reduce database queries
authors = Author.objects.prefetch_related('article_set').all()
```

#### Custom QuerySets

You can create custom QuerySets to encapsulate common query patterns:

```python
class ArticleQuerySet(models.QuerySet):
    def published(self):
        return self.filter(status='published')

    def popular(self):
        return self.annotate(popularity=F('num_likes') + F('num_shares')).order_by('-popularity')

class Article(models.Model):
    # ... fields ...
    objects = ArticleQuerySet.as_manager()

# Usage
popular_published_articles = Article.objects.published().popular()
```

#### Raw SQL

While Django ORM is powerful, sometimes you might need to write raw SQL:

```python
from django.db import connection

def my_custom_sql():
    with connection.cursor() as cursor:
        cursor.execute("UPDATE bar SET foo = 1 WHERE baz = %s", [self.baz])
        cursor.execute("SELECT foo FROM bar WHERE baz = %s", [self.baz])
        row = cursor.fetchone()
    return row
```

#### Best Practices for Django ORM:

1. Use `select_related()` and `prefetch_related()` to optimize database queries.
2. Create custom model managers or QuerySets for commonly used queries.
3. Use `bulk_create()` and `bulk_update()` for inserting or updating multiple objects efficiently.
4. Be cautious with `defer()` and `only()` - they can sometimes lead to more database queries.
5. Use `values()` or `values_list()` when you don't need full model objects.
6. Always use parameterized queries when using raw SQL to prevent SQL injection.
7. Use database indexes for fields that are frequently used in filters and ordering.

### 2.9. Admin Interface Customization

Django's admin interface is a powerful tool for managing your application's data. However, the default admin can often benefit from customization to better suit your specific needs.

#### Basic Admin Customization

Here's a basic example of customizing the admin for an Article model:

```python
from django.contrib import admin
from .models import Article

@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    list_display = ('title', 'author', 'status', 'created_at')
    list_filter = ('status', 'created_at', 'author')
    search_fields = ('title', 'content')
    prepopulated_fields = {'slug': ('title',)}
    date_hierarchy = 'created_at'
    ordering = ('-created_at',)
```

#### Customizing List Display

You can add custom methods to your admin class to display in the list view:

```python
class ArticleAdmin(admin.ModelAdmin):
    list_display = ('title', 'author', 'status', 'created_at', 'word_count')

    def word_count(self, obj):
        return len(obj.content.split())
    word_count.short_description = 'Word count'
```

#### Inline Editing

You can enable inline editing of related models:

```python
class CommentInline(admin.TabularInline):
    model = Comment
    extra = 1

class ArticleAdmin(admin.ModelAdmin):
    inlines = [CommentInline]
```

#### Customizing Forms

You can customize the forms used in the admin:

```python
from django import forms

class ArticleAdminForm(forms.ModelForm):
    def clean_title(self):
        title = self.cleaned_data['title']
        if len(title) < 5:
            raise forms.ValidationError("Title must be at least 5 characters long")
        return title

class ArticleAdmin(admin.ModelAdmin):
    form = ArticleAdminForm
```

#### Admin Actions

You can add custom actions to the admin:

```python
def make_published(modeladmin, request, queryset):
    queryset.update(status='published')
make_published.short_description = "Mark selected articles as published"

class ArticleAdmin(admin.ModelAdmin):
    actions = [make_published]
```

#### Customizing Admin Templates

You can override the default admin templates by creating templates in your app's `templates/admin` directory. For example, to customize the admin index page, create `templates/admin/index.html`.

#### Admin Site Customization

You can create a custom admin site:

```python
from django.contrib.admin import AdminSite

class MyAdminSite(AdminSite):
    site_header = 'My Custom Admin'
    site_title = 'My Custom Admin Portal'
    index_title = 'Welcome to My Custom Admin Portal'

my_admin_site = MyAdminSite(name='myadmin')
my_admin_site.register(Article, ArticleAdmin)
```

Then in your `urls.py`:

```python
from myapp.admin import my_admin_site

urlpatterns = [
    path('myadmin/', my_admin_site.urls),
]
```

#### Best Practices for Admin Customization:

1. Use `list_display`, `list_filter`, and `search_fields` to make it easier to find and manage objects.
2. Use `readonly_fields` for fields that shouldn't be editable in the admin.
3. Customize the `save_model` method to add custom behavior when saving objects.
4. Use `formfield_overrides` to customize form fields globally.
5. Create custom admin views for complex operations that don't fit into the standard CRUD operations.
6. Use `list_select_related` to reduce database queries in the list view.
7. Customize the admin site for better branding and user experience.

This concludes our section on core Django concepts and components. In the next section, we'll dive into advanced Django features and topics, starting with Class-based Views.

## 3. Advanced Django Features and Topics

### 3.1. Class-based Views

Class-based views (CBVs) in Django provide a way to structure your views and reuse code by harnessing the power of object-oriented programming. They offer a more organized approach to handling HTTP methods and are especially useful for creating generic views.

#### Basic Class-based View

Here's a basic example of a class-based view:

```python
from django.views import View
from django.shortcuts import render

class ArticleListView(View):
    def get(self, request):
        articles = Article.objects.all()
        return render(request, 'articles/list.html', {'articles': articles})

    def post(self, request):
        # Handle POST request
        pass
```

To use this view in your `urls.py`:

```python
from .views import ArticleListView

urlpatterns = [
    path('articles/', ArticleListView.as_view(), name='article_list'),
]
```

#### Generic Class-based Views

Django provides several generic class-based views that you can use and customize:

1. **ListView**: For displaying a list of objects

```python
from django.views.generic import ListView
from .models import Article

class ArticleListView(ListView):
    model = Article
    template_name = 'articles/list.html'
    context_object_name = 'articles'
    paginate_by = 10

    def get_queryset(self):
        return Article.objects.filter(status='published')
```

2. **DetailView**: For displaying a detail page for a single object

```python
from django.views.generic import DetailView
from .models import Article

class ArticleDetailView(DetailView):
    model = Article
    template_name = 'articles/detail.html'
    context_object_name = 'article'
```

3. **CreateView**: For creating a new object

```python
from django.views.generic import CreateView
from django.urls import reverse_lazy
from .models import Article

class ArticleCreateView(CreateView):
    model = Article
    fields = ['title', 'content', 'status']
    template_name = 'articles/create.html'
    success_url = reverse_lazy('article_list')

    def form_valid(self, form):
        form.instance.author = self.request.user
        return super().form_valid(form)
```

4. **UpdateView**: For updating an existing object

```python
from django.views.generic import UpdateView
from django.urls import reverse_lazy
from .models import Article

class ArticleUpdateView(UpdateView):
    model = Article
    fields = ['title', 'content', 'status']
    template_name = 'articles/update.html'

    def get_success_url(self):
        return reverse_lazy('article_detail', kwargs={'pk': self.object.pk})
```

5. **DeleteView**: For deleting an object

```python
from django.views.generic import DeleteView
from django.urls import reverse_lazy
from .models import Article

class ArticleDeleteView(DeleteView):
    model = Article
    template_name = 'articles/delete.html'
    success_url = reverse_lazy('article_list')
```

#### Mixins

Mixins are a form of multiple inheritance for class-based views. They allow you to add extra functionality to your views:

```python
from django.contrib.auth.mixins import LoginRequiredMixin
from django.views.generic import CreateView
from .models import Article

class ArticleCreateView(LoginRequiredMixin, CreateView):
    model = Article
    fields = ['title', 'content', 'status']
    template_name = 'articles/create.html'
    login_url = '/login/'
```

#### Custom Mixins

You can create your own mixins to encapsulate reusable functionality:

```python
from django.contrib import messages

class SuccessMessageMixin:
    success_message = ''

    def form_valid(self, form):
        response = super().form_valid(form)
        messages.success(self.request, self.success_message)
        return response

class ArticleCreateView(SuccessMessageMixin, CreateView):
    model = Article
    fields = ['title', 'content', 'status']
    success_message = 'Article created successfully!'
```

#### Best Practices for Class-based Views:

1. Use generic class-based views when possible to reduce boilerplate code.
2. Create custom mixins for functionality that's shared across multiple views.
3. Override `get_queryset()` and `get_context_data()` methods to customize data retrieval and context.
4. Use `LoginRequiredMixin` and `PermissionRequiredMixin` to handle authentication and authorization.
5. Customize form handling by overriding `form_valid()` and `form_invalid()` methods.
6. Use `reverse_lazy()` instead of `reverse()` when providing a URL to a class attribute.
7. Consider using function-based views for very simple views or highly customized behavior.

In the next section, we'll explore Custom Template Tags and Filters, which allow you to extend Django's template language with your own functionality.

### 3.2. Custom Template Tags and Filters

Django's template language comes with a variety of built-in tags and filters, but sometimes you need to extend its functionality with custom logic. Custom template tags and filters allow you to add your own functionality to templates.

#### Creating Custom Template Tags and Filters

First, create a `templatetags` directory in your app directory. Inside this directory, create a Python file (e.g., `custom_tags.py`):

```
your_app/
    ├── templatetags/
    │   ├── __init__.py
    │   └── custom_tags.py
    ├── models.py
    ├── views.py
    └── ...
```

In `custom_tags.py`, start with:

```python
from django import template

register = template.Library()
```

#### Simple Tag

A simple tag is a Python function that takes some arguments and returns a string:

```python
@register.simple_tag
def multiply(a, b):
    return int(a) * int(b)
```

Usage in template:

```html
{% load custom_tags %}
{% multiply 5 10 %}
```

#### Inclusion Tag

Inclusion tags are more complex. They render a template with a given context:

```python
@register.inclusion_tag('latest_articles.html')
def show_latest_articles(count=5):
    latest_articles = Article.objects.order_by('-created_at')[:count]
    return {'latest_articles': latest_articles}
```

Usage in template:

```html
{% load custom_tags %}
{% show_latest_articles 3 %}
```

And in `latest_articles.html`:

```html
<ul>
{% for article in latest_articles %}
    <li>{{ article.title }}</li>
{% endfor %}
</ul>
```

#### Assignment Tag

Assignment tags allow you to set a variable in the template:

```python
@register.simple_tag(takes_context=True)
def get_current_time(context):
    return datetime.now().strftime("%Y-%m-%d %H:%M:%S")
```

Usage in template:

```html
{% load custom_tags %}
{% get_current_time as current_time %}
<p>The current time is {{ current_time }}</p>
```

#### Custom Filter

Filters are functions that modify the value of a variable:

```python
@register.filter(name='cut')
def cut(value, arg):
    return value.replace(arg, '')
```

Usage in template:

```html
{% load custom_tags %}
{{ "Hello World"|cut:" " }}
```

#### Decorator-based Filters

You can also create filters using decorators:

```python
@register.filter
def lower(value):
    return value.lower()
```

#### Context Processors

Context processors allow you to add variables to the context of all templates:

In `your_app/context_processors.py`:

```python
def global_settings(request):
    return {
        'SITE_NAME': 'My Awesome Site',
        'CURRENT_YEAR': datetime.now().year
    }
```

Add to `settings.py`:

```python
TEMPLATES = [
    {
        ...
        'OPTIONS': {
            'context_processors': [
                ...
                'your_app.context_processors.global_settings',
            ],
        },
    },
]
```

Now `SITE_NAME` and `CURRENT_YEAR` are available in all templates.

#### Best Practices for Custom Template Tags and Filters:

1. Keep your custom tags and filters simple and focused.
2. Use meaningful names that clearly describe what the tag or filter does.
3. Provide default values for optional arguments.
4. Use `stringfilter` decorator for filters that only work on strings.
5. Be mindful of performance, especially for tags that query the database.
6. Document your custom tags and filters clearly.
7. Use `@register.filter(is_safe=True)` for filters that always return safe HTML.

### 3.3. Signals and Receivers

Django's signal dispatcher allows decoupled applications to get notified when actions occur elsewhere in the framework. It's a way of allowing certain senders to notify a set of receivers that some action has taken place.

#### Built-in Signals

Django comes with a set of built-in signals that let you know when certain actions occur:

1. `pre_save` & `post_save`: Sent before or after a model's `save()` method is called.
2. `pre_delete` & `post_delete`: Sent before or after a model's `delete()` method is called.
3. `m2m_changed`: Sent when a ManyToManyField on a model is changed.
4. `request_started` & `request_finished`: Sent when Django starts or finishes an HTTP request.

#### Creating a Signal Receiver

Here's how to create a receiver for the `post_save` signal:

```python
from django.db.models.signals import post_save
from django.dispatch import receiver
from django.contrib.auth.models import User
from .models import Profile

@receiver(post_save, sender=User)
def create_user_profile(sender, instance, created, **kwargs):
    if created:
        Profile.objects.create(user=instance)

@receiver(post_save, sender=User)
def save_user_profile(sender, instance, **kwargs):
    instance.profile.save()
```

This code creates a `Profile` instance every time a `User` is created, and saves the profile whenever the user is saved.

#### Connecting and Disconnecting Signals

You can also connect signals manually:

```python
from django.core.signals import request_finished
from django.dispatch import receiver

def my_callback(sender, **kwargs):
    print("Request finished!")

request_finished.connect(my_callback)
```

And disconnect them:

```python
request_finished.disconnect(my_callback)
```

#### Custom Signals

You can create your own custom signals:

```python
from django.dispatch import Signal

payment_completed = Signal()

# In your view or wherever the payment is processed
def process_payment(request):
    # ... payment processing logic ...
    payment_completed.send(sender=self.__class__, order=order, amount=amount)

# In your receiver
@receiver(payment_completed)
def on_payment_completed(sender, order, amount, **kwargs):
    # ... do something when payment is completed ...
```

#### Best Practices for Signals:

1. Use signals for decoupled and reusable functionality.
2. Be cautious with signals in performance-critical code, as they add overhead.
3. Always include **kwargs in your receiver functions to future-proof against new arguments.
4. Use `dispatch_uid` when connecting signals to ensure the receiver is only connected once.
5. Consider using `post_migrate` signal for initial data creation or schema modifications.
6. Be aware of circular import issues when working with signals.
7. Use signals judiciously - sometimes a direct method call or overriding a model method is more appropriate.

### 3.4. Caching Strategies

Caching is a technique to store the result of an expensive operation so that you can reuse it next time instead of recomputing it. Django provides a robust caching framework that allows you to cache everything from fragments of templates to entire sites.

#### Caching Levels

Django offers different levels of cache granularity:

1. **Low-level cache API**: For caching arbitrary objects.
2. **Template fragment caching**: For caching parts of templates.
3. **Per-view caching**: For caching the output of entire views.
4. **Per-site caching**: For caching your entire site.

#### Setting Up Caching

First, you need to set up your cache in `settings.py`. Here are a few examples:

1. **Local-memory caching** (default):

   ```python
   CACHES = {
       'default': {
           'BACKEND': 'django.core.cache.backends.locmem.LocMemCache',
           'LOCATION': 'unique-snowflake',
       }
   }
   ```
2. **Database caching**:

   ```python
   CACHES = {
       'default': {
           'BACKEND': 'django.core.cache.backends.db.DatabaseCache',
           'LOCATION': 'my_cache_table',
       }
   }
   ```
3. **File-based caching**:

   ```python
   CACHES = {
       'default': {
           'BACKEND': 'django.core.cache.backends.filebased.FileBasedCache',
           'LOCATION': '/var/tmp/django_cache',
       }
   }
   ```
4. **Memcached**:

   ```python
   CACHES = {
       'default': {
           'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
           'LOCATION': '127.0.0.1:11211',
       }
   }
   ```

#### Using the Cache API

Here's how to use the low-level cache API:

```python
from django.core.cache import cache

# Set a value in the cache
cache.set('my_key', 'my_value', timeout=300)  # Cache for 5 minutes

# Get a value from the cache
value = cache.get('my_key')

# Delete a value from the cache
cache.delete('my_key')
```

#### Template Fragment Caching

You can cache parts of a template using the `{% cache %}` tag:

```html
{% load cache %}
{% cache 500 sidebar %}
    .. sidebar content ..
{% endcache %}
```

This caches the sidebar content for 500 seconds.

#### Per-view Caching

You can cache the output of entire views:

```python
from django.views.decorators.cache import cache_page

@cache_page(60 * 15)  # Cache for 15 minutes
def my_view(request):
    ...
```

#### Per-site Caching

To enable per-site caching, add `UpdateCacheMiddleware` and `FetchFromCacheMiddleware` to your `MIDDLEWARE` setting:

```python
MIDDLEWARE = [
    'django.middleware.cache.UpdateCacheMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.cache.FetchFromCacheMiddleware',
]

CACHE_MIDDLEWARE_ALIAS = 'default'
CACHE_MIDDLEWARE_SECONDS = 600
CACHE_MIDDLEWARE_KEY_PREFIX = ''
```

#### Best Practices for Caching:

1. Cache expensive operations, like complex database queries or API calls.
2. Use appropriate cache timeouts - balance between freshness and performance.
3. Use cache versioning to invalidate cached data when models change.
4. Be careful caching user-specific data to avoid leaking information between users.
5. Use Memcached or Redis for production environments instead of local-memory caching.
6. Monitor your cache hit rate and adjust your caching strategy accordingly.
7. Consider using cache warming techniques for critical data.

In the next section, we'll explore integrating Django with REST framework to build powerful APIs.

### 3.5. REST Framework Integration

Django REST framework (DRF) is a powerful and flexible toolkit for building Web APIs. It provides a set of tools that make it easy to build model-backed APIs that have authentication policies and are browsable.

#### Setting Up Django REST Framework

First, install Django REST framework:

```bash
pip install djangorestframework
```

Add `'rest_framework'` to your `INSTALLED_APPS` setting:

```python
INSTALLED_APPS = [
    ...
    'rest_framework',
]
```

#### Serializers

Serializers allow complex data such as querysets and model instances to be converted to native Python datatypes that can then be easily rendered into JSON, XML or other content types. Here's an example:

```python
from rest_framework import serializers
from .models import Article

class ArticleSerializer(serializers.ModelSerializer):
    class Meta:
        model = Article
        fields = ['id', 'title', 'content', 'created_at', 'author']
```

#### Views

DRF provides several view classes to handle different scenarios:

1. **Function-based views with @api_view decorator**:

```python
from rest_framework.decorators import api_view
from rest_framework.response import Response

@api_view(['GET', 'POST'])
def article_list(request):
    if request.method == 'GET':
        articles = Article.objects.all()
        serializer = ArticleSerializer(articles, many=True)
        return Response(serializer.data)
    elif request.method == 'POST':
        serializer = ArticleSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=201)
        return Response(serializer.errors, status=400)
```

2. **Class-based views**:

```python
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import status

class ArticleList(APIView):
    def get(self, request):
        articles = Article.objects.all()
        serializer = ArticleSerializer(articles, many=True)
        return Response(serializer.data)

    def post(self, request):
        serializer = ArticleSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

3. **Generic views**:

```python
from rest_framework import generics

class ArticleList(generics.ListCreateAPIView):
    queryset = Article.objects.all()
    serializer_class = ArticleSerializer

class ArticleDetail(generics.RetrieveUpdateDestroyAPIView):
    queryset = Article.objects.all()
    serializer_class = ArticleSerializer
```

4. **ViewSets**:

```python
from rest_framework import viewsets

class ArticleViewSet(viewsets.ModelViewSet):
    queryset = Article.objects.all()
    serializer_class = ArticleSerializer
```

#### URL Configuration

When using ViewSets, you can automatically generate the URL conf for your API:

```python
from rest_framework.routers import DefaultRouter
from .views import ArticleViewSet

router = DefaultRouter()
router.register(r'articles', ArticleViewSet)

urlpatterns = [
    path('', include(router.urls)),
]
```

#### Authentication & Permissions

DRF provides various authentication classes:

```python
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.BasicAuthentication',
        'rest_framework.authentication.SessionAuthentication',
        'rest_framework.authentication.TokenAuthentication',
    ]
}
```

And permission classes:

```python
from rest_framework.permissions import IsAuthenticated, IsAuthenticatedOrReadOnly

class ArticleList(generics.ListCreateAPIView):
    queryset = Article.objects.all()
    serializer_class = ArticleSerializer
    permission_classes = [IsAuthenticatedOrReadOnly]
```

#### Pagination

DRF makes it easy to add pagination to your APIs:

```python
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 10
}
```

#### Filtering

You can add filtering to your ViewSets:

```python
from django_filters.rest_framework import DjangoFilterBackend

class ArticleViewSet(viewsets.ModelViewSet):
    queryset = Article.objects.all()
    serializer_class = ArticleSerializer
    filter_backends = [DjangoFilterBackend]
    filterset_fields = ['category', 'author']
```

#### Versioning

DRF supports API versioning:

```python
REST_FRAMEWORK = {
    'DEFAULT_VERSIONING_CLASS': 'rest_framework.versioning.NamespaceVersioning'
}
```

#### Best Practices for REST Framework:

1. Use ViewSets and Routers for standard CRUD operations.
2. Implement proper authentication and permissions for your APIs.
3. Use pagination to handle large datasets.
4. Implement filtering, searching, and ordering capabilities.
5. Version your API to maintain backwards compatibility.
6. Use meaningful status codes in your responses.
7. Implement proper exception handling and error responses.
8. Use content negotiation to support multiple formats (JSON, XML, etc.).

### 3.6. Asynchronous Views and ASGI

Django 3.0 introduced support for asynchronous ("async") Python, allowing Django to use asynchronous callable views, middleware, and tests. This is particularly useful for handling long-running operations without blocking the entire application.

#### Asynchronous Views

Here's an example of an asynchronous view:

```python
import asyncio
from django.http import HttpResponse

async def async_view(request):
    await asyncio.sleep(1)
    return HttpResponse("Hello, async world!")
```

#### ASGI (Asynchronous Server Gateway Interface)

ASGI is the spiritual successor to WSGI, designed to provide a standard interface between async-capable Python web servers, frameworks, and applications.

To use ASGI with Django, you need to use an ASGI server like Uvicorn or Daphne. First, install Uvicorn:

```bash
pip install uvicorn
```

Then, in your project's `asgi.py` file:

```python
import os
from django.core.asgi import get_asgi_application

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'myproject.settings')

application = get_asgi_application()
```

To run your project with Uvicorn:

```bash
uvicorn myproject.asgi:application
```

#### Async-Compatible ORM Operations

Django 3.1 introduced async-compatible ORM operations:

```python
from django.http import HttpResponse
from .models import Article

async def async_view(request):
    await Article.objects.afilter(status='published')
    return HttpResponse("Async ORM operation completed!")
```

#### Async Middleware

You can also create asynchronous middleware:

```python
class SimpleMiddleware:
    async def __call__(self, scope, receive, send):
        # ... do something asynchronous
        await self.app(scope, receive, send)
```

#### Best Practices for Asynchronous Views and ASGI:

1. Use async views for I/O-bound operations that can benefit from asynchronous execution.
2. Be aware that not all Django features are async-compatible yet.
3. Use an ASGI server like Uvicorn or Daphne in production for async support.
4. Familiarize yourself with async/await syntax and common pitfalls.
5. Consider using Django Channels for WebSocket support and real-time features.
6. Profile your async views to ensure they're providing a performance benefit.
7. Be cautious mixing sync and async code, as it can lead to unexpected behavior.

In the next section, we'll explore Testing in Django, covering both unit tests and integration tests.

### 3.7. Testing in Django

Testing is a crucial part of software development, and Django provides a robust testing framework that builds on Python's unittest module. Django's testing framework allows you to write and run tests for your models, views, forms, and other components of your application.

#### Types of Tests

1. **Unit Tests**: Test individual components in isolation.
2. **Integration Tests**: Test how different components work together.
3. **Functional Tests**: Test the application from the user's perspective.

#### Setting Up Tests

Django automatically creates a `tests.py` file in each app when you create a new app. You can also create a `tests` directory with multiple test files for better organization.

```python
# tests.py or tests/test_models.py
from django.test import TestCase
from .models import Article

class ArticleModelTest(TestCase):
    def setUp(self):
        Article.objects.create(title="Test Article", content="Test Content")

    def test_article_creation(self):
        article = Article.objects.get(title="Test Article")
        self.assertEqual(article.content, "Test Content")
```

#### Running Tests

To run your tests, use the following command:

```bash
python manage.py test
```

You can also run tests for a specific app or test case:

```bash
python manage.py test myapp
python manage.py test myapp.tests.test_models.ArticleModelTest
```

#### Testing Models

When testing models, you typically want to test:

- Object creation
- Field constraints
- Custom methods
- Relationships between models

```python
class ArticleModelTest(TestCase):
    def setUp(self):
        self.user = User.objects.create_user(username='testuser', password='12345')
        self.article = Article.objects.create(
            title="Test Article",
            content="Test Content",
            author=self.user
        )

    def test_article_str_method(self):
        self.assertEqual(str(self.article), "Test Article")

    def test_article_content(self):
        self.assertEqual(self.article.content, "Test Content")

    def test_article_author(self):
        self.assertEqual(self.article.author, self.user)
```

#### Testing Views

For testing views, Django provides the `Client` class to simulate GET and POST requests:

```python
from django.test import Client, TestCase
from django.urls import reverse
from .models import Article

class ArticleViewsTest(TestCase):
    def setUp(self):
        self.client = Client()
        self.article = Article.objects.create(title="Test Article", content="Test Content")

    def test_article_list_view(self):
        response = self.client.get(reverse('article_list'))
        self.assertEqual(response.status_code, 200)
        self.assertContains(response, "Test Article")

    def test_article_detail_view(self):
        response = self.client.get(reverse('article_detail', args=[self.article.id]))
        self.assertEqual(response.status_code, 200)
        self.assertContains(response, "Test Article")
        self.assertContains(response, "Test Content")
```

#### Testing Forms

When testing forms, you want to test both valid and invalid data:

```python
from .forms import ArticleForm

class ArticleFormTest(TestCase):
    def test_valid_form(self):
        data = {'title': 'Test Article', 'content': 'Test Content'}
        form = ArticleForm(data=data)
        self.assertTrue(form.is_valid())

    def test_invalid_form(self):
        data = {'title': '', 'content': 'Test Content'}
        form = ArticleForm(data=data)
        self.assertFalse(form.is_valid())
```

#### Testing API Endpoints

If you're using Django REST framework, you can use APIClient for testing API endpoints:

```python
from rest_framework.test import APIClient
from rest_framework import status

class ArticleAPITest(TestCase):
    def setUp(self):
        self.client = APIClient()
        self.article_data = {'title': 'Test Article', 'content': 'Test Content'}
        self.response = self.client.post(
            reverse('article-list'),
            self.article_data,
            format="json"
        )

    def test_create_article(self):
        self.assertEqual(self.response.status_code, status.HTTP_201_CREATED)

    def test_get_article(self):
        article = Article.objects.get(id=1)
        response = self.client.get(
            reverse('article-detail', kwargs={'pk': article.id}),
            format="json"
        )
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertEqual(response.data['title'], 'Test Article')
```

#### Using Factories

For creating test data, you can use factories. The `factory_boy` library integrates well with Django:

```python
import factory
from .models import Article, User

class UserFactory(factory.django.DjangoModelFactory):
    class Meta:
        model = User

    username = factory.Sequence(lambda n: f"user{n}")
    email = factory.LazyAttribute(lambda obj: f"{obj.username}@example.com")

class ArticleFactory(factory.django.DjangoModelFactory):
    class Meta:
        model = Article

    title = factory.Faker('sentence')
    content = factory.Faker('paragraph')
    author = factory.SubFactory(UserFactory)
```

#### Mocking

For unit tests, you often want to mock external services or complex operations. The `unittest.mock` module is useful for this:

```python
from unittest.mock import patch
from .services import external_api_call

class ExternalAPITest(TestCase):
    @patch('myapp.services.requests.get')
    def test_external_api_call(self, mock_get):
        mock_get.return_value.json.return_value = {'key': 'value'}
        result = external_api_call()
        self.assertEqual(result, {'key': 'value'})
```

#### Coverage

To measure your test coverage, you can use the `coverage` tool:

```bash
pip install coverage
coverage run --source='.' manage.py test
coverage report
```

#### Best Practices for Testing:

1. Write tests for all new features and bug fixes.
2. Aim for high test coverage, but focus on critical paths.
3. Use meaningful test method names that describe what is being tested.
4. Keep tests isolated and independent of each other.
5. Use setUp and tearDown methods to set up and clean up test data.
6. Use factories to create test data efficiently.
7. Mock external services and APIs in unit tests.
8. Run tests in a separate database to avoid affecting your development data.
9. Use continuous integration to run tests automatically on each commit.
10. Regularly review and update tests as your application evolves.

In the next section, we'll explore Django Security Best Practices to ensure your applications are secure and protected against common vulnerabilities.


### 3.8. Django Security Best Practices

Security is a critical aspect of web development, and Django provides many built-in security features. However, it's important to understand and properly implement these features to ensure your application remains secure.

#### Cross-Site Scripting (XSS) Protection

Django's template system automatically escapes HTML output, providing protection against XSS attacks. However, be cautious when using the `safe` filter or `mark_safe()` function.

```python
# Good - Auto-escaped
{{ user_input }}

# Dangerous - Only use when absolutely necessary and with trusted input
{{ user_input|safe }}
```

#### Cross-Site Request Forgery (CSRF) Protection

Django includes built-in CSRF protection. Always use the `csrf_token` in your forms:

```html
<form method="post">
    {% csrf_token %}
    <!-- form fields -->
</form>
```

For AJAX requests, you need to include the CSRF token in the headers:

```javascript
const csrftoken = document.querySelector('[name=csrfmiddlewaretoken]').value;

fetch(url, {
    method: 'POST',
    headers: {
        'X-CSRFToken': csrftoken
    },
    // ... other options
})
```

#### SQL Injection Protection

Django's ORM provides protection against SQL injection. Always use querysets and avoid raw SQL when possible:

```python
# Good
User.objects.filter(username=username)

# Dangerous - Only use when absolutely necessary and with proper escaping
User.objects.raw("SELECT * FROM auth_user WHERE username = %s", [username])
```

#### Clickjacking Protection

Django provides the `X-Frame-Options` middleware to prevent clickjacking. Ensure it's included in your `MIDDLEWARE` setting:

```python
MIDDLEWARE = [
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
    # ... other middleware
]
```

#### HTTPS/SSL

Always use HTTPS in production. You can enforce this in Django:

```python
SECURE_SSL_REDIRECT = True
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
```

#### Security Headers

Django provides several security-related headers. Enable them in your settings:

```python
SECURE_BROWSER_XSS_FILTER = True
SECURE_CONTENT_TYPE_NOSNIFF = True
```

#### Password Hashing

Django uses PBKDF2 by default for password hashing. You can increase the number of iterations for stronger security:

```python
PASSWORD_HASHERS = [
    'django.contrib.auth.hashers.PBKDF2PasswordHasher',
    'django.contrib.auth.hashers.PBKDF2SHA1PasswordHasher',
]
```

#### User Authentication

Implement strong password policies and consider using two-factor authentication:

```python
AUTH_PASSWORD_VALIDATORS = [
    {
        'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator',
        'OPTIONS': {
            'min_length': 9,
        }
    },
    {
        'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator',
    },
]
```

#### File Uploads

Be cautious when handling file uploads. Validate file types and limit file size:

```python
def validate_file_extension(value):
    import os
    from django.core.exceptions import ValidationError
    ext = os.path.splitext(value.name)[1]
    valid_extensions = ['.pdf', '.doc', '.docx', '.jpg', '.png', '.xlsx', '.xls']
    if not ext.lower() in valid_extensions:
        raise ValidationError('Unsupported file extension.')
```

#### Database Configuration

Protect your database configuration:

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.environ.get('DB_NAME'),
        'USER': os.environ.get('DB_USER'),
        'PASSWORD': os.environ.get('DB_PASSWORD'),
        'HOST': os.environ.get('DB_HOST'),
        'PORT': os.environ.get('DB_PORT'),
    }
}
```

#### Debug Mode

Always turn off debug mode in production:

```python
DEBUG = False
ALLOWED_HOSTS = ['www.yourdomain.com']
```

#### Security Middleware

Ensure all security-related middleware is included and in the correct order:

```python
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]
```

#### Content Security Policy (CSP)

Implement a Content Security Policy to prevent XSS and other injection attacks:

```python
CSP_DEFAULT_SRC = ("'self'",)
CSP_STYLE_SRC = ("'self'", "'unsafe-inline'")
CSP_SCRIPT_SRC = ("'self'",)
CSP_IMG_SRC = ("'self'", "https:", "data:")
```

#### Regular Security Updates

Keep Django and all dependencies up to date:

```bash
pip install --upgrade django
pip install --upgrade -r requirements.txt
```

#### Security Scanning Tools

Use security scanning tools regularly:

1. `bandit` for Python code scanning
2. `safety` for checking dependencies for known vulnerabilities
3. `django-security-check` for Django-specific security checks

#### Best Practices for Django Security:

1. Keep Django and all dependencies up to date.
2. Use HTTPS everywhere, especially for login pages and all authenticated parts of your site.
3. Properly handle user input and never trust it.
4. Use Django's built-in security features and don't disable them unless absolutely necessary.
5. Implement proper authentication and authorization checks.
6. Be cautious with file uploads and always validate them.
7. Use environment variables for sensitive information like secret keys and database credentials.
8. Regularly audit your code for security vulnerabilities.
9. Implement proper logging and monitoring to detect potential security issues.
10. Educate your development team about security best practices.

In the next section, we'll explore Performance Optimization Techniques to ensure your Django applications are fast and efficient.


### 3.9. Performance Optimization Techniques

Optimizing the performance of your Django application is crucial for providing a good user experience and managing server resources efficiently. Here are some key techniques to improve the performance of your Django applications:

#### Database Optimization

1. **Use `select_related()` and `prefetch_related()`**:
   These methods help reduce the number of database queries by fetching related objects in a single query.

   ```python
   # Instead of this
   articles = Article.objects.all()
   for article in articles:
       print(article.author.name)  # This causes a database query for each article

   # Do this
   articles = Article.objects.select_related('author').all()
   for article in articles:
       print(article.author.name)  # No additional queries
   ```
2. **Defer and Only**:
   Use `defer()` to postpone loading of expensive fields and `only()` to retrieve only the fields you need.

   ```python
   # Defer expensive fields
   users = User.objects.defer("last_login", "date_joined").all()

   # Only retrieve specific fields
   users = User.objects.only("username", "email").all()
   ```
3. **Bulk Operations**:
   Use bulk methods for creating, updating, or deleting multiple objects.

   ```python
   # Instead of this
   for item in items:
       Item.objects.create(name=item)

   # Do this
   Item.objects.bulk_create([Item(name=item) for item in items])
   ```
4. **Database Indexing**:
   Add indexes to fields that are frequently used in filters, ordering, or joins.

   ```python
   class Article(models.Model):
       title = models.CharField(max_length=200)
       pub_date = models.DateTimeField(db_index=True)
   ```

#### Caching

1. **Use Django's Cache Framework**:
   Implement caching to store the results of expensive operations.

   ```python
   from django.core.cache import cache

   def get_articles():
       articles = cache.get('all_articles')
       if articles is None:
           articles = list(Article.objects.all())
           cache.set('all_articles', articles, 60 * 15)  # Cache for 15 minutes
       return articles
   ```
2. **Template Fragment Caching**:
   Cache parts of templates that don't change often.

   ```html
   {% load cache %}
   {% cache 500 sidebar %}
       ... expensive to generate sidebar ...
   {% endcache %}
   ```
3. **Use Memcached or Redis**:
   In production, use a memory-based cache like Memcached or Redis for better performance.

   ```python
   CACHES = {
       'default': {
           'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
           'LOCATION': '127.0.0.1:11211',
       }
   }
   ```

#### Query Optimization

1. **Use `exists()` for Checking Existence**:
   When you only need to check if a queryset has any results, use `exists()` instead of `len()` or boolean evaluation.

   ```python
   # Instead of this
   if Article.objects.filter(status='published'):
       # do something

   # Do this
   if Article.objects.filter(status='published').exists():
       # do something
   ```
2. **Use `count()` for Counting**:
   When you only need the count of objects, use `count()` instead of `len()`.

   ```python
   # Instead of this
   article_count = len(Article.objects.all())

   # Do this
   article_count = Article.objects.count()
   ```
3. **Use `values()` or `values_list()` When You Don't Need Model Instances**:
   If you only need specific fields, use `values()` or `values_list()` to reduce memory usage.

   ```python
   # Instead of this
   titles = [article.title for article in Article.objects.all()]

   # Do this
   titles = Article.objects.values_list('title', flat=True)
   ```

#### Template Optimization

1. **Use Template Caching**:
   Cache your templates, especially for parts that don't change often.
2. **Minimize Template Filters and Tags**:
   Complex template filters and tags can slow down rendering. Move complex logic to views or model methods.
3. **Use `{% with %}` for Repeated Expensive Operations**:
   If you're repeating an expensive operation in a template, use `{% with %}` to store the result.

   ```html
   {% with total=business.employees.count %}
       <p>{{ business.name }} has {{ total }} employee{{ total|pluralize }}</p>
   {% endwith %}
   ```

#### Static Files Optimization

1. **Use Django's `ManifestStaticFilesStorage`**:
   This storage backend appends a content hash to filenames, allowing for aggressive caching.

   ```python
   STATICFILES_STORAGE = 'django.contrib.staticfiles.storage.ManifestStaticFilesStorage'
   ```
2. **Minify and Compress Static Files**:
   Use tools like `django-compressor` to minify and compress your CSS and JavaScript files.
3. **Use a CDN**:
   Serve static files from a Content Delivery Network for faster delivery.

#### Profiling and Monitoring

1. **Use Django Debug Toolbar**:
   This tool provides insights into your application's performance during development.
2. **Use Django Silk**:
   A live profiling and inspection tool for Django, useful for finding performance bottlenecks.
3. **Implement Proper Logging**:
   Use Django's logging capabilities to track performance issues in production.

#### Asynchronous Processing

1. **Use Celery for Background Tasks**:
   Offload time-consuming tasks to Celery for asynchronous processing.

   ```python
   from celery import shared_task

   @shared_task
   def process_data(data):
       # Time-consuming operation here
       pass

   # In your view
   def my_view(request):
       data = request.POST['data']
       process_data.delay(data)
       return HttpResponse('Processing started')
   ```
2. **Use Asynchronous Views** (Django 3.1+):
   For I/O bound operations, use async views to improve concurrency.

   ```python
   async def async_view(request):
       await asyncio.sleep(1)
       return HttpResponse('Hello, async world!')
   ```

#### Database Connection Pooling

Use a connection pooler like PgBouncer for PostgreSQL to manage database connections efficiently.

#### Best Practices for Performance Optimization:

1. Always measure performance before and after optimization to ensure your changes are effective.
2. Focus on optimizing the most frequently used parts of your application first.
3. Use tools like Django Debug Toolbar and profilers to identify bottlenecks.
4. Optimize database queries, as they are often the biggest performance bottleneck.
5. Implement caching at various levels: database query, function, view, and template.
6. Use asynchronous processing for time-consuming tasks.
7. Optimize your static files and consider using a CDN.
8. Keep your Django and Python versions up to date, as newer versions often include performance improvements.
9. Consider using specialized tools for specific needs, like full-text search engines for complex search functionality.
10. Monitor your application's performance in production and set up alerts for performance degradation.

Remember, premature optimization can lead to unnecessary complexity. Always profile your application to identify real bottlenecks before optimizing.

In the next section, we'll explore several project tutorials that will help you apply the concepts we've covered in real-world scenarios.


## 4. Project Tutorials

In this section, we'll walk through five different project tutorials, each focusing on a different type of web application. These tutorials will help you apply the concepts we've covered in real-world scenarios.

### 4.1. E-commerce Site

In this tutorial, we'll build a basic e-commerce site. Our site will include product listings, a shopping cart, user authentication, and a checkout process.

#### Step 1: Project Setup

First, let's create a new Django project and app:

```bash
django-admin startproject ecommerce
cd ecommerce
python manage.py startapp store
```

Add 'store' to INSTALLED_APPS in settings.py:

```python
INSTALLED_APPS = [
    # ...
    'store',
]
```

#### Step 2: Define Models

In `store/models.py`, let's define our basic models:

```python
from django.db import models
from django.contrib.auth.models import User

class Category(models.Model):
    name = models.CharField(max_length=200)
    slug = models.SlugField(max_length=200, unique=True)

    class Meta:
        ordering = ['name']
        verbose_name_plural = 'categories'

    def __str__(self):
        return self.name

class Product(models.Model):
    category = models.ForeignKey(Category, related_name='products', on_delete=models.CASCADE)
    name = models.CharField(max_length=200)
    slug = models.SlugField(max_length=200, unique=True)
    image = models.ImageField(upload_to='products/', blank=True)
    description = models.TextField(blank=True)
    price = models.DecimalField(max_digits=10, decimal_places=2)
    available = models.BooleanField(default=True)
    created = models.DateTimeField(auto_now_add=True)
    updated = models.DateTimeField(auto_now=True)

    class Meta:
        ordering = ['name']

    def __str__(self):
        return self.name

class Order(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    created = models.DateTimeField(auto_now_add=True)
    updated = models.DateTimeField(auto_now=True)
    paid = models.BooleanField(default=False)

    class Meta:
        ordering = ['-created']

    def __str__(self):
        return f'Order {self.id}'

class OrderItem(models.Model):
    order = models.ForeignKey(Order, related_name='items', on_delete=models.CASCADE)
    product = models.ForeignKey(Product, related_name='order_items', on_delete=models.CASCADE)
    price = models.DecimalField(max_digits=10, decimal_places=2)
    quantity = models.PositiveIntegerField(default=1)

    def __str__(self):
        return str(self.id)
```

After defining the models, run migrations:

```bash
python manage.py makemigrations
python manage.py migrate
```

#### Step 3: Create Views

In `store/views.py`, let's create some basic views:

```python
from django.shortcuts import render, get_object_or_404
from .models import Category, Product
from django.contrib.auth.decorators import login_required

def product_list(request, category_slug=None):
    category = None
    categories = Category.objects.all()
    products = Product.objects.filter(available=True)
    if category_slug:
        category = get_object_or_404(Category, slug=category_slug)
        products = products.filter(category=category)
    return render(request, 'store/product/list.html',
                  {'category': category,
                   'categories': categories,
                   'products': products})

def product_detail(request, id, slug):
    product = get_object_or_404(Product, id=id, slug=slug, available=True)
    return render(request, 'store/product/detail.html',
                  {'product': product})

@login_required
def order_create(request):
    # We'll implement this later
    pass
```

#### Step 4: Define URLs

In `store/urls.py`:

```python
from django.urls import path
from . import views

app_name = 'store'

urlpatterns = [
    path('', views.product_list, name='product_list'),
    path('<slug:category_slug>/', views.product_list, name='product_list_by_category'),
    path('<int:id>/<slug:slug>/', views.product_detail, name='product_detail'),
    path('order/', views.order_create, name='order_create'),
]
```

And in your project's `urls.py`:

```python
from django.contrib import admin
from django.urls import path, include
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('store.urls', namespace='store')),
]

if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

#### Step 5: Create Templates

Create the following templates:

`store/templates/store/base.html`:

```html
<!DOCTYPE html>
<html>
<head>
    <title>{% block title %}My Shop{% endblock %}</title>
</head>
<body>
    <div id="header">
        <a href="/" class="logo">My Shop</a>
    </div>
    <div id="subheader">
        <div class="cart">
            Your cart is empty.
        </div>
    </div>
    <div id="content">
        {% block content %}
        {% endblock %}
    </div>
</body>
</html>
```

`store/templates/store/product/list.html`:

```html
{% extends "store/base.html" %}
{% block title %}
    {% if category %}{{ category.name }}{% else %}Products{% endif %}
{% endblock %}
{% block content %}
    <h1>{% if category %}{{ category.name }}{% else %}Products{% endif %}</h1>
    {% for product in products %}
        <div class="item">
            <a href="{{ product.get_absolute_url }}">
                <img src="{% if product.image %}{{ product.image.url }}{% else %}{% static "img/no_image.png" %}{% endif %}">
            </a>
            <a href="{{ product.get_absolute_url }}">{{ product.name }}</a>
            <br>
            ${{ product.price }}
        </div>
    {% endfor %}
{% endblock %}
```

`store/templates/store/product/detail.html`:

```html
{% extends "store/base.html" %}
{% block title %}
    {{ product.name }}
{% endblock %}
{% block content %}
    <h1>{{ product.name }}</h1>
    <img src="{% if product.image %}{{ product.image.url }}{% else %}{% static "img/no_image.png" %}{% endif %}">
    <h2><a href="{{ product.category.get_absolute_url }}">{{ product.category }}</a></h2>
    <p class="price">${{ product.price }}</p>
    {{ product.description|linebreaks }}
{% endblock %}
```

#### Step 6: Set Up Admin

In `store/admin.py`:

```python
from django.contrib import admin
from .models import Category, Product, Order, OrderItem

@admin.register(Category)
class CategoryAdmin(admin.ModelAdmin):
    list_display = ['name', 'slug']
    prepopulated_fields = {'slug': ('name',)}

@admin.register(Product)
class ProductAdmin(admin.ModelAdmin):
    list_display = ['name', 'slug', 'price', 'available', 'created', 'updated']
    list_filter = ['available', 'created', 'updated']
    list_editable = ['price', 'available']
    prepopulated_fields = {'slug': ('name',)}

class OrderItemInline(admin.TabularInline):
    model = OrderItem
    raw_id_fields = ['product']

@admin.register(Order)
class OrderAdmin(admin.ModelAdmin):
    list_display = ['id', 'user', 'created', 'updated', 'paid']
    list_filter = ['paid', 'created', 'updated']
    inlines = [OrderItemInline]
```

#### Step 7: Implement Shopping Cart

For the shopping cart, we'll use Django sessions. Create a new file `store/cart.py`:

```python
from decimal import Decimal
from django.conf import settings
from .models import Product

class Cart:
    def __init__(self, request):
        self.session = request.session
        cart = self.session.get(settings.CART_SESSION_ID)
        if not cart:
            cart = self.session[settings.CART_SESSION_ID] = {}
        self.cart = cart

    def add(self, product, quantity=1, override_quantity=False):
        product_id = str(product.id)
        if product_id not in self.cart:
            self.cart[product_id] = {'quantity': 0, 'price': str(product.price)}
        if override_quantity:
            self.cart[product_id]['quantity'] = quantity
        else:
            self.cart[product_id]['quantity'] += quantity
        self.save()

    def save(self):
        self.session.modified = True

    def remove(self, product):
        product_id = str(product.id)
        if product_id in self.cart:
            del self.cart[product_id]
            self.save()

    def __iter__(self):
        product_ids = self.cart.keys()
        products = Product.objects.filter(id__in=product_ids)
        cart = self.cart.copy()
        for product in products:
            cart[str(product.id)]['product'] = product
        for item in cart.values():
            item['price'] = Decimal(item['price'])
            item['total_price'] = item['price'] * item['quantity']
            yield item

    def __len__(self):
        return sum(item['quantity'] for item in self.cart.values())

    def get_total_price(self):
        return sum(Decimal(item['price']) * item['quantity'] for item in self.cart.values())

    def clear(self):
        del self.session[settings.CART_SESSION_ID]
        self.save()
```

Add the following to your `settings.py`:

```python
CART_SESSION_ID = 'cart'
```

#### Step 8: Implement Order Creation

Update the `order_create` view in `views.py`:

```python
from django.shortcuts import render, redirect
from .models import OrderItem
from .forms import OrderCreateForm
from .cart import Cart

@login_required
def order_create(request):
    cart = Cart(request)
    if request.method == 'POST':
        form = OrderCreateForm(request.POST)
        if form.is_valid():
            order = form.save(commit=False)
            order.user = request.user
            order.save()
            for item in cart:
                OrderItem.objects.create(order=order,
                                         product=item['product'],
                                         price=item['price'],
                                         quantity=item['quantity'])
            # clear the cart
            cart.clear()
            return render(request, 'store/order/created.html',
                          {'order': order})
    else:
        form = OrderCreateForm()
    return render(request, 'store/order/create.html',
                  {'cart': cart, 'form': form})
```

Create `store/forms.py`:

```python
from django import forms
from .models import Order

class OrderCreateForm(forms.ModelForm):
    class Meta:
        model = Order
        fields = ['address', 'postal_code', 'city']
```

Create the corresponding templates:

`store/templates/store/order/create.html`:

```html
{% extends "store/base.html" %}

{% block title %}
    Checkout
{% endblock %}

{% block content %}
    <h1>Checkout</h1>
    <div class="order-info">
        <h3>Your order</h3>
        <ul>
            {% for item in cart %}
                <li>
                    {{ item.quantity }}x {{ item.product.name }}
                    <span>${{ item.total_price }}</span>
                </li>
            {% endfor %}
        </ul>
        <p>Total: ${{ cart.get_total_price }}</p>
    </div>
    <form method="post" class="order-form">
        {{ form.as_p }}
        <p><input type="submit" value="Place order"></p>
        {% csrf_token %}
    </form>
{% endblock %}
```

`store/templates/store/order/created.html`:

```html
{% extends "store/base.html" %}

{% block title %}
    Thank you
{% endblock %}

{% block content %}
    <h1>Thank you</h1>
    <p>Your order has been successfully completed. Your order number is
    <strong>{{ order.id }}</strong>.</p>
{% endblock %}
```

This completes a basic e-commerce site with product listings, a shopping cart, and order creation. You can expand on this by adding features like user reviews, payment integration, and more detailed product pages.

In the next tutorial, we'll build a social media platform, which will cover different aspects of Django development.


### 4.2. Social Media Platform

In this tutorial, we'll build a basic social media platform. Our platform will include user profiles, posts, comments, likes, and a news feed.

#### Step 1: Project Setup

First, let's create a new Django project and app:

```bash
django-admin startproject socialmedia
cd socialmedia
python manage.py startapp core
```

Add 'core' to INSTALLED_APPS in settings.py:

```python
INSTALLED_APPS = [
    # ...
    'core',
]
```

#### Step 2: Define Models

In `core/models.py`, let's define our basic models:

```python
from django.db import models
from django.contrib.auth.models import User
from django.urls import reverse

class Profile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    bio = models.TextField(max_length=500, blank=True)
    location = models.CharField(max_length=30, blank=True)
    birth_date = models.DateField(null=True, blank=True)
    avatar = models.ImageField(upload_to='avatars/', null=True, blank=True)

    def __str__(self):
        return self.user.username

class Post(models.Model):
    author = models.ForeignKey(User, on_delete=models.CASCADE, related_name='posts')
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    likes = models.ManyToManyField(User, related_name='liked_posts', blank=True)

    class Meta:
        ordering = ['-created_at']

    def __str__(self):
        return f"{self.author.username}'s post ({self.created_at:%Y-%m-%d %H:%M})"

    def get_absolute_url(self):
        return reverse('post_detail', args=[str(self.id)])

class Comment(models.Model):
    post = models.ForeignKey(Post, on_delete=models.CASCADE, related_name='comments')
    author = models.ForeignKey(User, on_delete=models.CASCADE, related_name='comments')
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        ordering = ['created_at']

    def __str__(self):
        return f'Comment by {self.author.username} on {self.post}'
```

After defining the models, run migrations:

```bash
python manage.py makemigrations
python manage.py migrate
```

#### Step 3: Create Views

In `core/views.py`, let's create some basic views:

```python
from django.shortcuts import render, get_object_or_404, redirect
from django.contrib.auth.decorators import login_required
from django.views.generic import ListView, DetailView
from django.contrib.auth.mixins import LoginRequiredMixin
from .models import Post, Profile
from .forms import PostForm, CommentForm

class NewsFeed(LoginRequiredMixin, ListView):
    model = Post
    template_name = 'core/news_feed.html'
    context_object_name = 'posts'
    paginate_by = 10

class PostDetailView(LoginRequiredMixin, DetailView):
    model = Post
    template_name = 'core/post_detail.html'

@login_required
def create_post(request):
    if request.method == 'POST':
        form = PostForm(request.POST)
        if form.is_valid():
            post = form.save(commit=False)
            post.author = request.user
            post.save()
            return redirect('news_feed')
    else:
        form = PostForm()
    return render(request, 'core/create_post.html', {'form': form})

@login_required
def add_comment(request, post_id):
    post = get_object_or_404(Post, id=post_id)
    if request.method == 'POST':
        form = CommentForm(request.POST)
        if form.is_valid():
            comment = form.save(commit=False)
            comment.post = post
            comment.author = request.user
            comment.save()
            return redirect('post_detail', pk=post.id)
    else:
        form = CommentForm()
    return render(request, 'core/add_comment.html', {'form': form})

@login_required
def like_post(request, post_id):
    post = get_object_or_404(Post, id=post_id)
    if request.user in post.likes.all():
        post.likes.remove(request.user)
    else:
        post.likes.add(request.user)
    return redirect('post_detail', pk=post.id)

class ProfileView(LoginRequiredMixin, DetailView):
    model = Profile
    template_name = 'core/profile.html'
    context_object_name = 'profile'

    def get_object(self):
        return get_object_or_404(Profile, user__username=self.kwargs['username'])
```

#### Step 4: Create Forms

Create a new file `core/forms.py`:

```python
from django import forms
from .models import Post, Comment

class PostForm(forms.ModelForm):
    class Meta:
        model = Post
        fields = ['content']

class CommentForm(forms.ModelForm):
    class Meta:
        model = Comment
        fields = ['content']
```

#### Step 5: Define URLs

In `core/urls.py`:

```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.NewsFeed.as_view(), name='news_feed'),
    path('post/<int:pk>/', views.PostDetailView.as_view(), name='post_detail'),
    path('post/new/', views.create_post, name='create_post'),
    path('post/<int:post_id>/comment/', views.add_comment, name='add_comment'),
    path('post/<int:post_id>/like/', views.like_post, name='like_post'),
    path('profile/<str:username>/', views.ProfileView.as_view(), name='profile'),
]
```

And in your project's `urls.py`:

```python
from django.contrib import admin
from django.urls import path, include
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('core.urls')),
    path('accounts/', include('django.contrib.auth.urls')),
]

if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

#### Step 6: Create Templates

Create the following templates:

`core/templates/core/base.html`:

```html
<!DOCTYPE html>
<html>
<head>
    <title>{% block title %}Social Media{% endblock %}</title>
</head>
<body>
    <header>
        <nav>
            <a href="{% url 'news_feed' %}">Home</a>
            {% if user.is_authenticated %}
                <a href="{% url 'profile' user.username %}">Profile</a>
                <a href="{% url 'logout' %}">Logout</a>
            {% else %}
                <a href="{% url 'login' %}">Login</a>
                <a href="{% url 'signup' %}">Sign Up</a>
            {% endif %}
        </nav>
    </header>
    <main>
        {% block content %}
        {% endblock %}
    </main>
</body>
</html>
```

`core/templates/core/news_feed.html`:

```html
{% extends 'core/base.html' %}

{% block title %}News Feed{% endblock %}

{% block content %}
    <h1>News Feed</h1>
    <a href="{% url 'create_post' %}">Create New Post</a>
    {% for post in posts %}
        <article>
            <h2><a href="{% url 'post_detail' post.id %}">{{ post.author.username }}'s Post</a></h2>
            <p>{{ post.content }}</p>
            <p>Posted on: {{ post.created_at }}</p>
            <p>Likes: {{ post.likes.count }}</p>
        </article>
    {% empty %}
        <p>No posts yet.</p>
    {% endfor %}
    {% if is_paginated %}
        <div class="pagination">
            <span class="step-links">
                {% if page_obj.has_previous %}
                    <a href="?page=1">« first</a>
                    <a href="?page={{ page_obj.previous_page_number }}">previous</a>
                {% endif %}

                <span class="current">
                    Page {{ page_obj.number }} of {{ page_obj.paginator.num_pages }}.
                </span>

                {% if page_obj.has_next %}
                    <a href="?page={{ page_obj.next_page_number }}">next</a>
                    <a href="?page={{ page_obj.paginator.num_pages }}">last »</a>
                {% endif %}
            </span>
        </div>
    {% endif %}
{% endblock %}
```

`core/templates/core/post_detail.html`:

```html
{% extends 'core/base.html' %}

{% block title %}{{ post.author.username }}'s Post{% endblock %}

{% block content %}
    <article>
        <h1>{{ post.author.username }}'s Post</h1>
        <p>{{ post.content }}</p>
        <p>Posted on: {{ post.created_at }}</p>
        <p>Likes: {{ post.likes.count }}</p>
        <form action="{% url 'like_post' post.id %}" method="post">
            {% csrf_token %}
            <button type="submit">{% if user in post.likes.all %}Unlike{% else %}Like{% endif %}</button>
        </form>
    </article>

    <section>
        <h2>Comments</h2>
        {% for comment in post.comments.all %}
            <article>
                <p>{{ comment.content }}</p>
                <p>By {{ comment.author.username }} on {{ comment.created_at }}</p>
            </article>
        {% empty %}
            <p>No comments yet.</p>
        {% endfor %}
        <a href="{% url 'add_comment' post.id %}">Add a comment</a>
    </section>
{% endblock %}
```

`core/templates/core/create_post.html`:

```html
{% extends 'core/base.html' %}

{% block title %}Create Post{% endblock %}

{% block content %}
    <h1>Create a New Post</h1>
    <form method="post">
        {% csrf_token %}
        {{ form.as_p }}
        <button type="submit">Post</button>
    </form>
{% endblock %}
```

`core/templates/core/add_comment.html`:

```html
{% extends 'core/base.html' %}

{% block title %}Add Comment{% endblock %}

{% block content %}
    <h1>Add a Comment</h1>
    <form method="post">
        {% csrf_token %}
        {{ form.as_p }}
        <button type="submit">Comment</button>
    </form>
{% endblock %}
```

`core/templates/core/profile.html`:

```html
{% extends 'core/base.html' %}

{% block title %}{{ profile.user.username }}'s Profile{% endblock %}

{% block content %}
    <h1>{{ profile.user.username }}'s Profile</h1>
    {% if profile.avatar %}
        <img src="{{ profile.avatar.url }}" alt="{{ profile.user.username }}'s avatar">
    {% endif %}
    <p>Bio: {{ profile.bio }}</p>
    <p>Location: {{ profile.location }}</p>
    <p>Birth Date: {{ profile.birth_date }}</p>

    <h2>{{ profile.user.username }}'s Posts</h2>
    {% for post in profile.user.posts.all %}
        <article>
            <h3><a href="{% url 'post_detail' post.id %}">{{ post.content|truncatewords:10 }}</a></h3>
            <p>Posted on: {{ post.created_at }}</p>
            <p>Likes: {{ post.likes.count }}</p>
        </article>
    {% empty %}
        <p>No posts yet.</p>
    {% endfor %}
{% endblock %}
```

#### Step 7: Set Up Authentication

For user authentication, we'll use Django's built-in authentication views. Add the following to your project's `urls.py`:

```python
from django.contrib.auth import views as auth_views

urlpatterns = [
    # ...
    path('accounts/login/', auth_views.LoginView.as_view(template_name='core/login.html'), name='login'),
    path('accounts/logout/', auth_views.LogoutView.as_view(next_page='login'), name='logout'),
    # ...
]
```

Create `core/templates/core/login.html`:

```html
{% extends 'core/base.html' %}

{% block title %}Login{% endblock %}

{% block content %}
    <h1>Login</h1>
    <form method="post">
        {% csrf_token %}
        {{ form.as_p }}
        <button type="submit">Login</button>
    </form>
{% endblock %}
```

#### Step 8: Set Up Admin

In `core/admin.py`:

```python
from django.contrib import admin
from .models import Profile, Post, Comment

@admin.register(Profile)
class ProfileAdmin(admin.ModelAdmin):
    list_display = ['user', 'location', 'birth_date']
    raw_id_fields = ['user']

@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    list_display = ['author', 'created_at', 'updated_at']
    list_filter = ['created_at', 'updated_at']
    raw_id_fields = ['author']

@admin.register(Comment)
class CommentAdmin(admin.ModelAdmin):
    list_display = ['author', 'post', 'created_at']
    list_filter = ['created_at']
    raw_id_fields = ['author', 'post']
```

#### Step 9: Create User Signup

Add a view for user signup in `core/views.py`:

```python
from django.contrib.auth import login
from django.contrib.auth.forms import UserCreationForm

def signup(request):
    if request.method == 'POST':
        form = UserCreationForm(request.POST)
        if form.is_valid():
            user = form.save()
            login(request, user)
            # Create a profile for the new user
            Profile.objects.create(user=user)
            return redirect('news_feed')
    else:
        form = UserCreationForm()
    return render(request, 'core/signup.html', {'form': form})
```

Add the URL in `core/urls.py`:

```python
path('signup/', views.signup, name='signup'),
```

Create `core/templates/core/signup.html`:

```html
{% extends 'core/base.html' %}

{% block title %}Sign Up{% endblock %}

{% block content %}
    <h1>Sign Up</h1>
    <form method="post">
        {% csrf_token %}
        {{ form.as_p }}
        <button type="submit">Sign Up</button>
    </form>
{% endblock %}
```

This completes a basic social media platform with user profiles, posts, comments, an

### 4.3. Content Management System

In this tutorial, we'll build a basic Content Management System (CMS). Our CMS will include features like creating and managing pages, blog posts, categories, and tags. We'll also implement a simple admin interface for content creators.

#### Step 1: Project Setup

First, let's create a new Django project and app:

```bash
django-admin startproject cms_project
cd cms_project
python manage.py startapp cms
```

Add 'cms' to INSTALLED_APPS in settings.py:

```python
INSTALLED_APPS = [
    # ...
    'cms',
]
```

#### Step 2: Define Models

In `cms/models.py`, let's define our basic models:

```python
from django.db import models
from django.contrib.auth.models import User
from django.urls import reverse
from django.utils.text import slugify

class Category(models.Model):
    name = models.CharField(max_length=100, unique=True)
    slug = models.SlugField(max_length=100, unique=True)

    class Meta:
        verbose_name_plural = "categories"

    def __str__(self):
        return self.name

    def save(self, *args, **kwargs):
        self.slug = slugify(self.name)
        super().save(*args, **kwargs)

class Tag(models.Model):
    name = models.CharField(max_length=50, unique=True)
    slug = models.SlugField(max_length=50, unique=True)

    def __str__(self):
        return self.name

    def save(self, *args, **kwargs):
        self.slug = slugify(self.name)
        super().save(*args, **kwargs)

class Page(models.Model):
    title = models.CharField(max_length=200)
    slug = models.SlugField(max_length=200, unique=True)
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    author = models.ForeignKey(User, on_delete=models.CASCADE)
    is_published = models.BooleanField(default=False)

    def __str__(self):
        return self.title

    def get_absolute_url(self):
        return reverse('cms:page_detail', args=[self.slug])

    def save(self, *args, **kwargs):
        if not self.slug:
            self.slug = slugify(self.title)
        super().save(*args, **kwargs)

class Post(models.Model):
    title = models.CharField(max_length=200)
    slug = models.SlugField(max_length=200, unique=True)
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    author = models.ForeignKey(User, on_delete=models.CASCADE)
    category = models.ForeignKey(Category, on_delete=models.CASCADE)
    tags = models.ManyToManyField(Tag)
    is_published = models.BooleanField(default=False)

    def __str__(self):
        return self.title

    def get_absolute_url(self):
        return reverse('cms:post_detail', args=[self.slug])

    def save(self, *args, **kwargs):
        if not self.slug:
            self.slug = slugify(self.title)
        super().save(*args, **kwargs)
```

After defining the models, run migrations:

```bash
python manage.py makemigrations
python manage.py migrate
```

#### Step 3: Create Views

In `cms/views.py`, let's create views for our CMS:

```python
from django.shortcuts import render, get_object_or_404, redirect
from django.contrib.auth.decorators import login_required
from django.views.generic import ListView, DetailView
from django.contrib.auth.mixins import LoginRequiredMixin
from .models import Page, Post, Category, Tag
from .forms import PageForm, PostForm

class HomeView(ListView):
    model = Post
    template_name = 'cms/home.html'
    context_object_name = 'posts'
    queryset = Post.objects.filter(is_published=True).order_by('-created_at')[:5]

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['pages'] = Page.objects.filter(is_published=True).order_by('title')
        return context

class PageDetailView(DetailView):
    model = Page
    template_name = 'cms/page_detail.html'
    context_object_name = 'page'

    def get_queryset(self):
        return Page.objects.filter(is_published=True)

class PostDetailView(DetailView):
    model = Post
    template_name = 'cms/post_detail.html'
    context_object_name = 'post'

    def get_queryset(self):
        return Post.objects.filter(is_published=True)

class CategoryPostsView(ListView):
    model = Post
    template_name = 'cms/category_posts.html'
    context_object_name = 'posts'

    def get_queryset(self):
        self.category = get_object_or_404(Category, slug=self.kwargs['slug'])
        return Post.objects.filter(category=self.category, is_published=True)

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['category'] = self.category
        return context

class TagPostsView(ListView):
    model = Post
    template_name = 'cms/tag_posts.html'
    context_object_name = 'posts'

    def get_queryset(self):
        self.tag = get_object_or_404(Tag, slug=self.kwargs['slug'])
        return Post.objects.filter(tags=self.tag, is_published=True)

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['tag'] = self.tag
        return context

@login_required
def create_page(request):
    if request.method == 'POST':
        form = PageForm(request.POST)
        if form.is_valid():
            page = form.save(commit=False)
            page.author = request.user
            page.save()
            return redirect(page.get_absolute_url())
    else:
        form = PageForm()
    return render(request, 'cms/create_page.html', {'form': form})

@login_required
def create_post(request):
    if request.method == 'POST':
        form = PostForm(request.POST)
        if form.is_valid():
            post = form.save(commit=False)
            post.author = request.user
            post.save()
            form.save_m2m()  # Save many-to-many relationships
            return redirect(post.get_absolute_url())
    else:
        form = PostForm()
    return render(request, 'cms/create_post.html', {'form': form})
```

#### Step 4: Create Forms

Create a new file `cms/forms.py`:

```python
from django import forms
from .models import Page, Post

class PageForm(forms.ModelForm):
    class Meta:
        model = Page
        fields = ['title', 'content', 'is_published']

class PostForm(forms.ModelForm):
    class Meta:
        model = Post
        fields = ['title', 'content', 'category', 'tags', 'is_published']
```

#### Step 5: Define URLs

In `cms/urls.py`:

```python
from django.urls import path
from . import views

app_name = 'cms'

urlpatterns = [
    path('', views.HomeView.as_view(), name='home'),
    path('page/<slug:slug>/', views.PageDetailView.as_view(), name='page_detail'),
    path('post/<slug:slug>/', views.PostDetailView.as_view(), name='post_detail'),
    path('category/<slug:slug>/', views.CategoryPostsView.as_view(), name='category_posts'),
    path('tag/<slug:slug>/', views.TagPostsView.as_view(), name='tag_posts'),
    path('create-page/', views.create_page, name='create_page'),
    path('create-post/', views.create_post, name='create_post'),
]
```

And in your project's `urls.py`:

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('cms.urls')),
]
```

#### Step 6: Create Templates

Create the following templates:

`cms/templates/cms/base.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}My CMS{% endblock %}</title>
</head>
<body>
    <header>
        <nav>
            <ul>
                <li><a href="{% url 'cms:home' %}">Home</a></li>
                {% if user.is_authenticated %}
                    <li><a href="{% url 'cms:create_page' %}">Create Page</a></li>
                    <li><a href="{% url 'cms:create_post' %}">Create Post</a></li>
                    <li><a href="{% url 'logout' %}">Logout</a></li>
                {% else %}
                    <li><a href="{% url 'login' %}">Login</a></li>
                {% endif %}
            </ul>
        </nav>
    </header>
    <main>
        {% block content %}
        {% endblock %}
    </main>
    <footer>
        <p>© 2024 My CMS. All rights reserved.</p>
    </footer>
</body>
</html>
```

`cms/templates/cms/home.html`:

```html
{% extends 'cms/base.html' %}

{% block title %}Home - My CMS{% endblock %}

{% block content %}
    <h1>Welcome to My CMS</h1>
  
    <h2>Pages</h2>
    <ul>
        {% for page in pages %}
            <li><a href="{{ page.get_absolute_url }}">{{ page.title }}</a></li>
        {% empty %}
            <li>No pages available.</li>
        {% endfor %}
    </ul>

    <h2>Recent Posts</h2>
    {% for post in posts %}
        <article>
            <h3><a href="{{ post.get_absolute_url }}">{{ post.title }}</a></h3>
            <p>{{ post.content|truncatewords:30 }}</p>
            <p>Category: <a href="{% url 'cms:category_posts' post.category.slug %}">{{ post.category.name }}</a></p>
            <p>Tags: 
                {% for tag in post.tags.all %}
                    <a href="{% url 'cms:tag_posts' tag.slug %}">{{ tag.name }}</a>{% if not forloop.last %}, {% endif %}
                {% endfor %}
            </p>
            <p>Published on: {{ post.created_at|date:"F d, Y" }}</p>
        </article>
    {% empty %}
        <p>No posts available.</p>
    {% endfor %}
{% endblock %}
```

`cms/templates/cms/page_detail.html`:

```html
{% extends 'cms/base.html' %}

{% block title %}{{ page.title }} - My CMS{% endblock %}

{% block content %}
    <article>
        <h1>{{ page.title }}</h1>
        <div>{{ page.content|safe }}</div>
        <p>Last updated: {{ page.updated_at|date:"F d, Y" }}</p>
    </article>
{% endblock %}
```

`cms/templates/cms/post_detail.html`:

```html
{% extends 'cms/base.html' %}

{% block title %}{{ post.title }} - My CMS{% endblock %}

{% block content %}
    <article>
        <h1>{{ post.title }}</h1>
        <p>Category: <a href="{% url 'cms:category_posts' post.category.slug %}">{{ post.category.name }}</a></p>
        <p>Tags: 
            {% for tag in post.tags.all %}
                <a href="{% url 'cms:tag_posts' tag.slug %}">{{ tag.name }}</a>{% if not forloop.last %}, {% endif %}
            {% endfor %}
        </p>
        <div>{{ post.content|safe }}</div>
        <p>Published on: {{ post.created_at|date:"F d, Y" }}</p>
        <p>Last updated: {{ post.updated_at|date:"F d, Y" }}</p>
    </article>
{% endblock %}
```

`cms/templates/cms/category_posts.html`:

```html
{% extends 'cms/base.html' %}

{% block title %}Posts in {{ category.name }} - My CMS{% endblock %}

{% block content %}
    <h1>Posts in {{ category.name }}</h1>
    {% for post in posts %}
        <article>
            <h2><a href="{{ post.get_absolute_url }}">{{ post.title }}</a></h2>
            <p>{{ post.content|truncatewords:30 }}</p>
            <p>Published on: {{ post.created_at|date:"F d, Y" }}</p>
        </article>
    {% empty %}
        <p>No posts available in this category.</p>
    {% endfor %}
{% endblock %}
```

`cms/templates/cms/tag_posts.html`:

```html
{% extends 'cms/base.html' %}

{% block title %}Posts tagged with {{ tag.name }} - My CMS{% endblock %}

{% block content %}
    <h1>Posts tagged with {{ tag.name }}</h1>
    {% for post in posts %}
        <article>
            <h2><a href="{{ post.get_absolute_url }}">{{ post.title }}</a></h2>
            <p>{{ post.content|truncatewords:30 }}</p>
            <p>Published on: {{ post.created_at|date:"F d, Y" }}</p>
        </article>
    {% empty %}
        <p>No posts available with this tag.</p>
    {% endfor %}
{% endblock %}
```

`cms/templates/cms/create_page.html`:

```html
{% extends 'cms/base.html' %}

{% block title %}Create Page - My CMS{% endblock %}

{% block content %}
    <h1>Create a New Page</h1>
    <form method="post">
        {% csrf_token %}
        {{ form.as_p }}
        <button type="submit">Create Page</button>
    </form>
{% endblock %}
```

`cms/templates/cms/create_post.html`:

```html
{% extends 'cms/base.html' %}

{% block title %}Create Post - My CMS{% endblock %}

{% block content %}
    <h1>Create a New Post</h1>
    <form method="post">
        {% csrf_token %}
        {{ form.as_p }}
        <button type="submit">Create Post</button>
    </form>
{% endblock %}
```

#### Step 7: Set Up Admin

In `cms/admin.py`:

```python
from django.contrib import admin
from .models import Category, Tag, Page, Post

@admin.register(Category)
class CategoryAdmin(admin.ModelAdmin
```


):
    list_display = ['name', 'slug']
    prepopulated_fields = {'slug': ('name',)}

@admin.register(Tag)
class TagAdmin(admin.ModelAdmin):
    list_display = ['name', 'slug']
    prepopulated_fields = {'slug': ('name',)}

@admin.register(Page)
class PageAdmin(admin.ModelAdmin):
    list_display = ['title', 'slug', 'author', 'created_at', 'updated_at', 'is_published']
    list_filter = ['is_published', 'created_at', 'updated_at', 'author']
    search_fields = ['title', 'content']
    prepopulated_fields = {'slug': ('title',)}
    raw_id_fields = ['author']
    date_hierarchy = 'created_at'
    ordering = ['created_at', 'title']

@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    list_display = ['title', 'slug', 'author', 'category', 'created_at', 'updated_at', 'is_published']
    list_filter = ['is_published', 'created_at', 'updated_at', 'author', 'category']
    search_fields = ['title', 'content']
    prepopulated_fields = {'slug': ('title',)}
    raw_id_fields = ['author']
    date_hierarchy = 'created_at'
    ordering = ['created_at', 'title']
    filter_horizontal = ['tags']

```

#### Step 8: Implement Rich Text Editor

To enhance the content editing experience, let's integrate a rich text editor. We'll use django-ckeditor for this purpose.

First, install django-ckeditor:

```bash
pip install django-ckeditor
```

Add 'ckeditor' to INSTALLED_APPS in settings.py:

```python
INSTALLED_APPS = [
    # ...
    'ckeditor',
]
```

Update the Page and Post models in `cms/models.py`:

```python
from ckeditor.fields import RichTextField

class Page(models.Model):
    # ...
    content = RichTextField()
    # ...

class Post(models.Model):
    # ...
    content = RichTextField()
    # ...
```

#### Step 9: Implement Search Functionality

Let's add a search feature to our CMS. First, update `cms/views.py`:

```python
from django.db.models import Q

def search(request):
    query = request.GET.get('q')
    if query:
        pages = Page.objects.filter(Q(title__icontains=query) | Q(content__icontains=query), is_published=True)
        posts = Post.objects.filter(Q(title__icontains=query) | Q(content__icontains=query), is_published=True)
    else:
        pages = Page.objects.none()
        posts = Post.objects.none()
  
    return render(request, 'cms/search_results.html', {
        'query': query,
        'pages': pages,
        'posts': posts
    })
```

Add the search URL in `cms/urls.py`:

```python
path('search/', views.search, name='search'),
```

Create `cms/templates/cms/search_results.html`:

```html
{% extends 'cms/base.html' %}

{% block title %}Search Results - My CMS{% endblock %}

{% block content %}
    <h1>Search Results for "{{ query }}"</h1>
  
    <h2>Pages</h2>
    {% for page in pages %}
        <article>
            <h3><a href="{{ page.get_absolute_url }}">{{ page.title }}</a></h3>
            <p>{{ page.content|striptags|truncatewords:30 }}</p>
        </article>
    {% empty %}
        <p>No pages found.</p>
    {% endfor %}

    <h2>Posts</h2>
    {% for post in posts %}
        <article>
            <h3><a href="{{ post.get_absolute_url }}">{{ post.title }}</a></h3>
            <p>{{ post.content|striptags|truncatewords:30 }}</p>
            <p>Category: <a href="{% url 'cms:category_posts' post.category.slug %}">{{ post.category.name }}</a></p>
        </article>
    {% empty %}
        <p>No posts found.</p>
    {% endfor %}
{% endblock %}
```

Update `cms/templates/cms/base.html` to include a search form:

```html
<header>
    <!-- ... -->
    <form action="{% url 'cms:search' %}" method="get">
        <input type="text" name="q" placeholder="Search...">
        <button type="submit">Search</button>
    </form>
    <!-- ... -->
</header>
```

#### Step 10: Implement Pagination

To handle a large number of posts or pages, let's implement pagination. Update the `HomeView`, `CategoryPostsView`, and `TagPostsView` in `cms/views.py`:

```python
from django.core.paginator import Paginator, EmptyPage, PageNotAnInteger

class HomeView(ListView):
    model = Post
    template_name = 'cms/home.html'
    context_object_name = 'posts'
    paginate_by = 5

    def get_queryset(self):
        return Post.objects.filter(is_published=True).order_by('-created_at')

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['pages'] = Page.objects.filter(is_published=True).order_by('title')
        return context

class CategoryPostsView(ListView):
    model = Post
    template_name = 'cms/category_posts.html'
    context_object_name = 'posts'
    paginate_by = 5

    def get_queryset(self):
        self.category = get_object_or_404(Category, slug=self.kwargs['slug'])
        return Post.objects.filter(category=self.category, is_published=True).order_by('-created_at')

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['category'] = self.category
        return context

class TagPostsView(ListView):
    model = Post
    template_name = 'cms/tag_posts.html'
    context_object_name = 'posts'
    paginate_by = 5

    def get_queryset(self):
        self.tag = get_object_or_404(Tag, slug=self.kwargs['slug'])
        return Post.objects.filter(tags=self.tag, is_published=True).order_by('-created_at')

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['tag'] = self.tag
        return context
```

Update the templates to include pagination. Add the following to `cms/templates/cms/home.html`, `cms/templates/cms/category_posts.html`, and `cms/templates/cms/tag_posts.html`:

```html
{% if is_paginated %}
    <div class="pagination">
        <span class="step-links">
            {% if page_obj.has_previous %}
                <a href="?page=1">« first</a>
                <a href="?page={{ page_obj.previous_page_number }}">previous</a>
            {% endif %}

            <span class="current">
                Page {{ page_obj.number }} of {{ page_obj.paginator.num_pages }}.
            </span>

            {% if page_obj.has_next %}
                <a href="?page={{ page_obj.next_page_number }}">next</a>
                <a href="?page={{ page_obj.paginator.num_pages }}">last »</a>
            {% endif %}
        </span>
    </div>
{% endif %}
```

#### Step 11: Implement Basic Analytics

Let's add a simple view counter for pages and posts. Update the `Page` and `Post` models in `cms/models.py`:

```python
class Page(models.Model):
    # ...
    views = models.PositiveIntegerField(default=0)
    # ...

class Post(models.Model):
    # ...
    views = models.PositiveIntegerField(default=0)
    # ...
```

Update the `PageDetailView` and `PostDetailView` in `cms/views.py`:

```python
from django.db.models import F

class PageDetailView(DetailView):
    # ...
    def get_object(self):
        obj = super().get_object()
        obj.views = F('views') + 1
        obj.save(update_fields=['views'])
        return obj

class PostDetailView(DetailView):
    # ...
    def get_object(self):
        obj = super().get_object()
        obj.views = F('views') + 1
        obj.save(update_fields=['views'])
        return obj
```

Update the templates to display the view count. Add the following to `cms/templates/cms/page_detail.html` and `cms/templates/cms/post_detail.html`:

```html
<p>Views: {{ page.views }}</p>  <!-- For page_detail.html -->
<p>Views: {{ post.views }}</p>  <!-- For post_detail.html -->
```

#### Step 12: Implement RSS Feeds

Finally, let's add RSS feeds for our CMS. Create a new file `cms/feeds.py`:

```python
from django.contrib.syndication.views import Feed
from django.urls import reverse
from .models import Post

class LatestPostsFeed(Feed):
    title = "My CMS Latest Posts"
    link = "/feeds/"
    description = "Latest posts from My CMS."

    def items(self):
        return Post.objects.filter(is_published=True).order_by('-created_at')[:5]

    def item_title(self, item):
        return item.title

    def item_description(self, item):
        return item.content[:200]

    def item_link(self, item):
        return reverse('cms:post_detail', args=[item.slug])
```

Add the feed URL in `cms/urls.py`:

```python
from .feeds import LatestPostsFeed

urlpatterns = [
    # ...
    path('feeds/', LatestPostsFeed(), name='post_feed'),
]
```

Add a link to the RSS feed in `cms/templates/cms/base.html`:

```html
<head>
    <!-- ... -->
    <link rel="alternate" type="application/rss+xml" title="Latest Posts" href="{% url 'cms:post_feed' %}">
</head>
```

This completes our basic Content Management System. It includes features like pages, blog posts, categories, tags, search functionality, pagination, basic analytics, and RSS feeds. You can further expand this CMS by adding features like user comments, social media sharing, or more advanced analytics.

In the next tutorial, we'll build an API-driven web application, which will cover different aspects of Django REST framework and frontend integration.


### 4.4. API-driven Web App

In this tutorial, we'll build a task management application with a Django REST framework backend and a React frontend. This will demonstrate how to create a full-stack application with a decoupled architecture.

#### Step 1: Project Setup

First, let's set up our Django project:

```bash
django-admin startproject taskmanager
cd taskmanager
python manage.py startapp tasks
```

Install necessary packages:

```bash
pip install djangorestframework django-cors-headers
```

Update `settings.py`:

```python
INSTALLED_APPS = [
    # ...
    'rest_framework',
    'corsheaders',
    'tasks',
]

MIDDLEWARE = [
    'corsheaders.middleware.CorsMiddleware',
    # ... other middleware
]

# Allow all origins for development (customize for production)
CORS_ALLOW_ALL_ORIGINS = True

REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.SessionAuthentication',
        'rest_framework.authentication.BasicAuthentication',
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',
    ],
}
```

#### Step 2: Define Models

In `tasks/models.py`:

```python
from django.db import models
from django.contrib.auth.models import User

class Task(models.Model):
    title = models.CharField(max_length=200)
    description = models.TextField(blank=True)
    completed = models.BooleanField(default=False)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    user = models.ForeignKey(User, on_delete=models.CASCADE, related_name='tasks')

    def __str__(self):
        return self.title

    class Meta:
        ordering = ['-created_at']
```

Run migrations:

```bash
python manage.py makemigrations
python manage.py migrate
```

#### Step 3: Create Serializers

Create `tasks/serializers.py`:

```python
from rest_framework import serializers
from .models import Task

class TaskSerializer(serializers.ModelSerializer):
    class Meta:
        model = Task
        fields = ['id', 'title', 'description', 'completed', 'created_at', 'updated_at']
        read_only_fields = ['id', 'created_at', 'updated_at']

    def create(self, validated_data):
        validated_data['user'] = self.context['request'].user
        return super().create(validated_data)
```

#### Step 4: Create Views

In `tasks/views.py`:

```python
from rest_framework import viewsets
from .models import Task
from .serializers import TaskSerializer

class TaskViewSet(viewsets.ModelViewSet):
    serializer_class = TaskSerializer

    def get_queryset(self):
        return Task.objects.filter(user=self.request.user)
```

#### Step 5: Configure URLs

In `taskmanager/urls.py`:

```python
from django.contrib import admin
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from tasks.views import TaskViewSet

router = DefaultRouter()
router.register(r'tasks', TaskViewSet, basename='task')

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include(router.urls)),
    path('api-auth/', include('rest_framework.urls')),
]
```

#### Step 6: Set Up React Frontend

Now, let's set up the React frontend. We'll use Create React App for this example.

```bash
npx create-react-app frontend
cd frontend
npm install axios
```

#### Step 7: Create React Components

Replace the contents of `src/App.js`:

```jsx
import React, { useState, useEffect } from 'react';
import axios from 'axios';

const API_URL = 'http://localhost:8000/api/';

function App() {
  const [tasks, setTasks] = useState([]);
  const [title, setTitle] = useState('');
  const [description, setDescription] = useState('');

  useEffect(() => {
    fetchTasks();
  }, []);

  const fetchTasks = async () => {
    const response = await axios.get(`${API_URL}tasks/`, {
      auth: {
        username: 'your_username',
        password: 'your_password'
      }
    });
    setTasks(response.data);
  };

  const createTask = async (e) => {
    e.preventDefault();
    await axios.post(`${API_URL}tasks/`, { title, description }, {
      auth: {
        username: 'your_username',
        password: 'your_password'
      }
    });
    setTitle('');
    setDescription('');
    fetchTasks();
  };

  const toggleComplete = async (id, completed) => {
    await axios.patch(`${API_URL}tasks/${id}/`, { completed: !completed }, {
      auth: {
        username: 'your_username',
        password: 'your_password'
      }
    });
    fetchTasks();
  };

  const deleteTask = async (id) => {
    await axios.delete(`${API_URL}tasks/${id}/`, {
      auth: {
        username: 'your_username',
        password: 'your_password'
      }
    });
    fetchTasks();
  };

  return (
    <div className="App">
      <h1>Task Manager</h1>
      <form onSubmit={createTask}>
        <input
          type="text"
          value={title}
          onChange={(e) => setTitle(e.target.value)}
          placeholder="Task title"
          required
        />
        <input
          type="text"
          value={description}
          onChange={(e) => setDescription(e.target.value)}
          placeholder="Task description"
        />
        <button type="submit">Add Task</button>
      </form>
      <ul>
        {tasks.map((task) => (
          <li key={task.id}>
            <input
              type="checkbox"
              checked={task.completed}
              onChange={() => toggleComplete(task.id, task.completed)}
            />
            <span style={{ textDecoration: task.completed ? 'line-through' : 'none' }}>
              {task.title}
            </span>
            <button onClick={() => deleteTask(task.id)}>Delete</button>
          </li>
        ))}
      </ul>
    </div>
  );
}

export default App;
```

#### Step 8: Run the Application

1. Start the Django server:

   ```bash
   python manage.py runserver
   ```
2. In a new terminal, start the React development server:

   ```bash
   cd frontend
   npm start
   ```

Now you should have a working task management application with a Django REST framework backend and a React frontend.

#### Step 9: Implement User Authentication

To make our application more secure and user-specific, let's implement token-based authentication.

First, install `djangorestframework-simplejwt`:

```bash
pip install djangorestframework-simplejwt
```

Update `settings.py`:

```python
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ],
}

from datetime import timedelta

SIMPLE_JWT = {
    'ACCESS_TOKEN_LIFETIME': timedelta(minutes=60),
    'REFRESH_TOKEN_LIFETIME': timedelta(days=1),
}
```

Update `taskmanager/urls.py`:

```python
from rest_framework_simplejwt.views import TokenObtainPairView, TokenRefreshView

urlpatterns = [
    # ...
    path('api/token/', TokenObtainPairView.as_view(), name='token_obtain_pair'),
    path('api/token/refresh/', TokenRefreshView.as_view(), name='token_refresh'),
]
```

Now, let's update our React frontend to use token authentication. First, install `jwt-decode`:

```bash
npm install jwt-decode
```

Update `src/App.js`:

```jsx
import React, { useState, useEffect } from 'react';
import axios from 'axios';
import jwt_decode from 'jwt-decode';

const API_URL = 'http://localhost:8000/api/';

function App() {
  const [tasks, setTasks] = useState([]);
  const [title, setTitle] = useState('');
  const [description, setDescription] = useState('');
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');
  const [token, setToken] = useState(localStorage.getItem('token'));

  useEffect(() => {
    if (token) {
      fetchTasks();
    }
  }, [token]);

  const login = async (e) => {
    e.preventDefault();
    try {
      const response = await axios.post(`${API_URL}token/`, { username, password });
      setToken(response.data.access);
      localStorage.setItem('token', response.data.access);
      localStorage.setItem('refresh_token', response.data.refresh);
    } catch (error) {
      console.error('Login failed:', error);
    }
  };

  const logout = () => {
    setToken(null);
    localStorage.removeItem('token');
    localStorage.removeItem('refresh_token');
    setTasks([]);
  };

  const fetchTasks = async () => {
    try {
      const response = await axios.get(`${API_URL}tasks/`, {
        headers: { Authorization: `Bearer ${token}` }
      });
      setTasks(response.data);
    } catch (error) {
      if (error.response && error.response.status === 401) {
        refreshToken();
      } else {
        console.error('Error fetching tasks:', error);
      }
    }
  };

  const refreshToken = async () => {
    try {
      const refresh_token = localStorage.getItem('refresh_token');
      const response = await axios.post(`${API_URL}token/refresh/`, { refresh: refresh_token });
      setToken(response.data.access);
      localStorage.setItem('token', response.data.access);
    } catch (error) {
      console.error('Error refreshing token:', error);
      logout();
    }
  };

  const createTask = async (e) => {
    e.preventDefault();
    try {
      await axios.post(`${API_URL}tasks/`, { title, description }, {
        headers: { Authorization: `Bearer ${token}` }
      });
      setTitle('');
      setDescription('');
      fetchTasks();
    } catch (error) {
      console.error('Error creating task:', error);
    }
  };

  const toggleComplete = async (id, completed) => {
    try {
      await axios.patch(`${API_URL}tasks/${id}/`, { completed: !completed }, {
        headers: { Authorization: `Bearer ${token}` }
      });
      fetchTasks();
    } catch (error) {
      console.error('Error toggling task completion:', error);
    }
  };

  const deleteTask = async (id) => {
    try {
      await axios.delete(`${API_URL}tasks/${id}/`, {
        headers: { Authorization: `Bearer ${token}` }
      });
      fetchTasks();
    } catch (error) {
      console.error('Error deleting task:', error);
    }
  };

  if (!token) {
    return (
      <div className="App">
        <h1>Task Manager</h1>
        <form onSubmit={login}>
          <input
            type="text"
            value={username}
            onChange={(e) => setUsername(e.target.value)}
            placeholder="Username"
            required
          />
          <input
            type="password"
            value={password}
            onChange={(e) => setPassword(e.target.value)}
            placeholder="Password"
            required
          />
          <button type="submit">Login</button>
        </form>
      </div>
    );
  }

  return (
    <div className="App">
      <h1>Task Manager</h1>
      <button onClick={logout}>Logout</button>
      <form onSubmit={createTask}>
        <input
          type="text"
          value={title}
          onChange={(e) => setTitle(e.target.value)}
          placeholder="Task title"
          required
        />
        <input
          type="text"
          value={description}
          onChange={(e) => setDescription(e.target.value)}
          placeholder="Task description"
        />
        <button type="submit">Add Task</button>
      </form>
      <ul>
        {tasks.map((task) => (
          <li key={task.id}>
            <input
              type="checkbox"
              checked={task.completed}
              onChange={() => toggleComplete(task.id, task.completed)}
            />
            <span style={{ textDecoration: task.completed ? 'line-through' : 'none' }}>
              {task.title}
            </span>
            <button onClick={() => deleteTask(task.id)}>Delete</button>
          </li>
        ))}
      </ul>
    </div>
  );
}

export default App;
```

This updated version includes token-based authentication, automatic token refresh, and a login form. Users will need to log in before they can see and manage their tasks.

#### Step 10: Error Handling and User Feedback

To improve the user experience, let's add some error handling and user feedback. Update `src/App.js`:

```jsx
import React, { useState, useEffect } from 'react';
import axios from 'axios';
import jwt_decode from 'jwt-decode';

const API_URL = 'http://localhost:8000/api/';

function App() {
  // ... (previous state variables)
  const [error, setError] = useState(null);
  const [message, setMessage] = useState(null);

  // ... (previous useEffect and functions)

  const showMessage = (msg) => {
    setMessage(msg);
    setTimeout(() => setMessage(null), 3000);
  };

  const login = async (e) => {
    e.preventDefault();
    try {
      const response = await axios.post(`${API_URL}token/`, { username, password });
      setToken(response.data.access);
      localStorage.setItem('token', response.data.access);
      localStorage.setItem('refresh_token', response.data.refresh);
      setError(null);
      showMessage('Logged in successfully');
    } catch (error) {
      console.error('Login failed:', error);
      setError('Invalid username or password');
    }
  };

  const logout = () => {
    setToken(null);
    localStorage.removeItem('token');
    localStorage.removeItem('refresh_token');
    setTasks([]);
    showMessage('Logged out successfully');
  };

  const createTask = async (e) => {
    e.preventDefault();
    try {
      await axios.post(`${API_URL}tasks/`, { title, description }, {
        headers: { Authorization: `Bearer ${token}` }
      });
      setTitle('');
      setDescription('');
      fetchTasks();
      showMessage('Task created successfully');
    } catch (error) {
      console.error('Error creating task:', error);
      setError('Failed to create task');
    }
  };

  const toggleComplete = async (id, completed) => {
    try {
      await axios.patch(`${API_URL}tasks/${id}/`, { completed: !completed }, {
        headers: { Authorization: `Bearer ${token}` }
      });
      fetchTasks();
      showMessage('Task updated successfully');
    } catch (error) {
      console.error('
```


Error toggling task completion:', error);
      setError('Failed to update task');
    }
  };

  const deleteTask = async (id) => {
    try {
      await axios.delete(`${API_URL}tasks/${id}/`, {
        headers: { Authorization: `Bearer ${token}` }
      });
      fetchTasks();
      showMessage('Task deleted successfully');
    } catch (error) {
      console.error('Error deleting task:', error);
      setError('Failed to delete task');
    }
  };

  if (!token) {
    return (
      `<div className="App">`
        `<h1>`Task Manager`</h1>`
        {error && `<div className="error">`{error}`</div>`}
        {message && `<div className="message">`{message}`</div>`}
        `<form onSubmit={login}>`
          <input
            type="text"
            value={username}
            onChange={(e) => setUsername(e.target.value)}
            placeholder="Username"
            required
          />
          <input
            type="password"
            value={password}
            onChange={(e) => setPassword(e.target.value)}
            placeholder="Password"
            required
          />
          `<button type="submit">`Login`</button>`
        `</form>`
      `</div>`
    );
  }

  return (
    `<div className="App">`
      `<h1>`Task Manager`</h1>`
      {error && `<div className="error">`{error}`</div>`}
      {message && `<div className="message">`{message}`</div>`}
      `<button onClick={logout}>`Logout`</button>`
      `<form onSubmit={createTask}>`
        <input
          type="text"
          value={title}
          onChange={(e) => setTitle(e.target.value)}
          placeholder="Task title"
          required
        />
        <input
          type="text"
          value={description}
          onChange={(e) => setDescription(e.target.value)}
          placeholder="Task description"
        />
        `<button type="submit">`Add Task`</button>`
      `</form>`
      `<ul>`
        {tasks.map((task) => (
          `<li key={task.id}>`
            <input
              type="checkbox"
              checked={task.completed}
              onChange={() => toggleComplete(task.id, task.completed)}
            />
            <span style={{ textDecoration: task.completed ? 'line-through' : 'none' }}>
              {task.title}
          
            <button onClick={() => deleteTask(task.id)}>Delete`</button>`
          `</li>`
        ))}
      `</ul>`
    `</div>`
  );
}

export default App;

```

#### Step 11: Styling

Let's add some basic styling to make our app look better. Create a new file `src/App.css` and add the following:

```css
.App {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
  font-family: Arial, sans-serif;
}

h1 {
  text-align: center;
  color: #333;
}

form {
  display: flex;
  margin-bottom: 20px;
}

input[type="text"],
input[type="password"] {
  flex-grow: 1;
  padding: 10px;
  margin-right: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

button {
  padding: 10px 20px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

button:hover {
  background-color: #0056b3;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: flex;
  align-items: center;
  padding: 10px;
  border-bottom: 1px solid #eee;
}

li:last-child {
  border-bottom: none;
}

li input[type="checkbox"] {
  margin-right: 10px;
}

li button {
  margin-left: auto;
  background-color: #dc3545;
}

li button:hover {
  background-color: #c82333;
}

.error {
  background-color: #f8d7da;
  color: #721c24;
  padding: 10px;
  margin-bottom: 20px;
  border: 1px solid #f5c6cb;
  border-radius: 4px;
}

.message {
  background-color: #d4edda;
  color: #155724;
  padding: 10px;
  margin-bottom: 20px;
  border: 1px solid #c3e6cb;
  border-radius: 4px;
}
```

Now, import this CSS file in your `src/App.js`:

```jsx
import React, { useState, useEffect } from 'react';
import axios from 'axios';
import jwt_decode from 'jwt-decode';
import './App.css';

// ... rest of the component code
```

#### Step 12: Add Loading State

To improve user experience, let's add a loading state to show when tasks are being fetched. Update `src/App.js`:

```jsx
function App() {
  // ... other state variables
  const [loading, setLoading] = useState(false);

  const fetchTasks = async () => {
    setLoading(true);
    try {
      const response = await axios.get(`${API_URL}tasks/`, {
        headers: { Authorization: `Bearer ${token}` }
      });
      setTasks(response.data);
    } catch (error) {
      if (error.response && error.response.status === 401) {
        refreshToken();
      } else {
        console.error('Error fetching tasks:', error);
        setError('Failed to fetch tasks');
      }
    } finally {
      setLoading(false);
    }
  };

  // ... in the return statement, add this before the task list
  {loading && <div className="loading">Loading tasks...</div>}
```

Add the following CSS to `src/App.css`:

```css
.loading {
  text-align: center;
  color: #007bff;
  margin: 20px 0;
}
```

#### Step 13: Implement Task Filtering

Let's add the ability to filter tasks based on their completion status. Update `src/App.js`:

```jsx
function App() {
  // ... other state variables
  const [filter, setFilter] = useState('all');

  const filteredTasks = tasks.filter(task => {
    if (filter === 'active') return !task.completed;
    if (filter === 'completed') return task.completed;
    return true;
  });

  // ... in the return statement, add this before the task list
  <div className="filters">
    <button onClick={() => setFilter('all')} className={filter === 'all' ? 'active' : ''}>All</button>
    <button onClick={() => setFilter('active')} className={filter === 'active' ? 'active' : ''}>Active</button>
    <button onClick={() => setFilter('completed')} className={filter === 'completed' ? 'active' : ''}>Completed</button>
  </div>

  // ... replace the task mapping with this
  {filteredTasks.map((task) => (
    // ... task item JSX
  ))}
```

Add the following CSS to `src/App.css`:

```css
.filters {
  display: flex;
  justify-content: center;
  margin-bottom: 20px;
}

.filters button {
  margin: 0 5px;
  padding: 5px 10px;
  background-color: #f8f9fa;
  color: #007bff;
  border: 1px solid #007bff;
}

.filters button.active {
  background-color: #007bff;
  color: white;
}
```

This completes our API-driven web application with a Django REST framework backend and a React frontend. The application now includes:

1. A RESTful API for task management
2. Token-based authentication
3. CRUD operations for tasks
4. Error handling and user feedback
5. Basic styling
6. Loading state
7. Task filtering

This project demonstrates how to build a full-stack application with a decoupled architecture, allowing for scalability and separation of concerns between the frontend and backend.

In the next tutorial, we'll build a real-time chat application, which will introduce websockets and real-time communication in Django.


### 4.5. Real-time Chat Application

In this tutorial, we'll build a real-time chat application using Django Channels, which extends Django to handle WebSockets, allowing for real-time, bi-directional communication between the client and server.

#### Step 1: Project Setup

First, let's set up our Django project:

```bash
django-admin startproject chatproject
cd chatproject
python manage.py startapp chat
```

Install necessary packages:

```bash
pip install django channels
```

Update `chatproject/settings.py`:

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'channels',
    'chat',
]

# Channels
ASGI_APPLICATION = 'chatproject.asgi.application'
CHANNEL_LAYERS = {
    'default': {
        'BACKEND': 'channels.layers.InMemoryChannelLayer'
    }
}
```

#### Step 2: Set up ASGI

Update `chatproject/asgi.py`:

```python
import os
from django.core.asgi import get_asgi_application
from channels.routing import ProtocolTypeRouter, URLRouter
from channels.auth import AuthMiddlewareStack
import chat.routing

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'chatproject.settings')

application = ProtocolTypeRouter({
    "http": get_asgi_application(),
    "websocket": AuthMiddlewareStack(
        URLRouter(
            chat.routing.websocket_urlpatterns
        )
    ),
})
```

#### Step 3: Create the Chat Consumer

Create `chat/consumers.py`:

```python
import json
from channels.generic.websocket import AsyncWebsocketConsumer

class ChatConsumer(AsyncWebsocketConsumer):
    async def connect(self):
        self.room_name = self.scope['url_route']['kwargs']['room_name']
        self.room_group_name = f'chat_{self.room_name}'

        # Join room group
        await self.channel_layer.group_add(
            self.room_group_name,
            self.channel_name
        )

        await self.accept()

    async def disconnect(self, close_code):
        # Leave room group
        await self.channel_layer.group_discard(
            self.room_group_name,
            self.channel_name
        )

    # Receive message from WebSocket
    async def receive(self, text_data):
        text_data_json = json.loads(text_data)
        message = text_data_json['message']

        # Send message to room group
        await self.channel_layer.group_send(
            self.room_group_name,
            {
                'type': 'chat_message',
                'message': message
            }
        )

    # Receive message from room group
    async def chat_message(self, event):
        message = event['message']

        # Send message to WebSocket
        await self.send(text_data=json.dumps({
            'message': message
        }))
```

#### Step 4: Set up Routing

Create `chat/routing.py`:

```python
from django.urls import re_path
from . import consumers

websocket_urlpatterns = [
    re_path(r'ws/chat/(?P<room_name>\w+)/$', consumers.ChatConsumer.as_asgi()),
]
```

#### Step 5: Create Views

Update `chat/views.py`:

```python
from django.shortcuts import render

def index(request):
    return render(request, 'chat/index.html')

def room(request, room_name):
    return render(request, 'chat/room.html', {
        'room_name': room_name
    })
```

#### Step 6: Set up URLs

Update `chatproject/urls.py`:

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('chat/', include('chat.urls')),
]
```

Create `chat/urls.py`:

```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.index, name='index'),
    path('<str:room_name>/', views.room, name='room'),
]
```

#### Step 7: Create Templates

Create `chat/templates/chat/index.html`:

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8"/>
    <title>Chat Rooms</title>
</head>
<body>
    <h1>Chat Rooms</h1>
    <input id="room-name-input" type="text" size="100"><br>
    <input id="room-name-submit" type="button" value="Enter">

    <script>
        document.querySelector('#room-name-input').focus();
        document.querySelector('#room-name-input').onkeyup = function(e) {
            if (e.keyCode === 13) {  // enter, return
                document.querySelector('#room-name-submit').click();
            }
        };

        document.querySelector('#room-name-submit').onclick = function(e) {
            var roomName = document.querySelector('#room-name-input').value;
            window.location.pathname = '/chat/' + roomName + '/';
        };
    </script>
</body>
</html>
```

Create `chat/templates/chat/room.html`:

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8"/>
    <title>Chat Room</title>
</head>
<body>
    <h1>Chat Room: {{ room_name }}</h1>
    <textarea id="chat-log" cols="100" rows="20"></textarea><br>
    <input id="chat-message-input" type="text" size="100"><br>
    <input id="chat-message-submit" type="button" value="Send">
    {{ room_name|json_script:"room-name" }}
    <script>
        const roomName = JSON.parse(document.getElementById('room-name').textContent);

        const chatSocket = new WebSocket(
            'ws://'
            + window.location.host
            + '/ws/chat/'
            + roomName
            + '/'
        );

        chatSocket.onmessage = function(e) {
            const data = JSON.parse(e.data);
            document.querySelector('#chat-log').value += (data.message + '\n');
        };

        chatSocket.onclose = function(e) {
            console.error('Chat socket closed unexpectedly');
        };

        document.querySelector('#chat-message-input').focus();
        document.querySelector('#chat-message-input').onkeyup = function(e) {
            if (e.keyCode === 13) {  // enter, return
                document.querySelector('#chat-message-submit').click();
            }
        };

        document.querySelector('#chat-message-submit').onclick = function(e) {
            const messageInputDom = document.querySelector('#chat-message-input');
            const message = messageInputDom.value;
            chatSocket.send(JSON.stringify({
                'message': message
            }));
            messageInputDom.value = '';
        };
    </script>
</body>
</html>
```

#### Step 8: Run the Application

Run the following commands:

```bash
python manage.py migrate
python manage.py runserver
```

Now, you should be able to access the chat application at `http://localhost:8000/chat/`.

#### Step 9: Enhance the Chat Application

Let's add some features to make our chat application more robust:

1. User authentication
2. Storing chat messages in the database
3. Displaying user names with messages

First, let's create a model for storing messages. Update `chat/models.py`:

```python
from django.db import models
from django.contrib.auth.models import User

class Room(models.Model):
    name = models.CharField(max_length=255, unique=True)

    def __str__(self):
        return self.name

class Message(models.Model):
    room = models.ForeignKey(Room, related_name='messages', on_delete=models.CASCADE)
    user = models.ForeignKey(User, related_name='messages', on_delete=models.CASCADE)
    content = models.TextField()
    timestamp = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return f'{self.user.username}: {self.content} [{self.timestamp}]'
```

Run migrations:

```bash
python manage.py makemigrations
python manage.py migrate
```

Now, update the `ChatConsumer` in `chat/consumers.py`:

```python
import json
from channels.generic.websocket import AsyncWebsocketConsumer
from channels.db import database_sync_to_async
from .models import Room, Message
from django.contrib.auth.models import User

class ChatConsumer(AsyncWebsocketConsumer):
    async def connect(self):
        self.room_name = self.scope['url_route']['kwargs']['room_name']
        self.room_group_name = f'chat_{self.room_name}'
        self.user = self.scope["user"]

        # Join room group
        await self.channel_layer.group_add(
            self.room_group_name,
            self.channel_name
        )

        await self.accept()

        # Send existing messages to the newly connected client
        messages = await self.get_messages()
        for message in messages:
            await self.send(text_data=json.dumps({
                'message': message['content'],
                'username': message['username'],
                'timestamp': message['timestamp']
            }))

    async def disconnect(self, close_code):
        # Leave room group
        await self.channel_layer.group_discard(
            self.room_group_name,
            self.channel_name
        )

    # Receive message from WebSocket
    async def receive(self, text_data):
        text_data_json = json.loads(text_data)
        message = text_data_json['message']

        # Save the message to the database
        await self.save_message(message)

        # Send message to room group
        await self.channel_layer.group_send(
            self.room_group_name,
            {
                'type': 'chat_message',
                'message': message,
                'username': self.user.username,
                'timestamp': str(timezone.now())
            }
        )

    # Receive message from room group
    async def chat_message(self, event):
        message = event['message']
        username = event['username']
        timestamp = event['timestamp']

        # Send message to WebSocket
        await self.send(text_data=json.dumps({
            'message': message,
            'username': username,
            'timestamp': timestamp
        }))

    @database_sync_to_async
    def save_message(self, message):
        room, _ = Room.objects.get_or_create(name=self.room_name)
        Message.objects.create(room=room, user=self.user, content=message)

    @database_sync_to_async
    def get_messages(self):
        room, _ = Room.objects.get_or_create(name=self.room_name)
        messages = Message.objects.filter(room=room).order_by('-timestamp')[:50]
        return [{'content': message.content, 'username': message.user.username, 'timestamp': str(message.timestamp)} for message in messages]
```

Update the `room` view in `chat/views.py` to require login:

```python
from django.shortcuts import render
from django.contrib.auth.decorators import login_required

def index(request):
    return render(request, 'chat/index.html')

@login_required
def room(request, room_name):
    return render(request, 'chat/room.html', {
        'room_name': room_name
    })
```

Update `chat/templates/chat/room.html` to display usernames and timestamps:

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8"/>
    <title>Chat Room</title>
    <style>
        #chat-log {
            height: 300px;
            overflow-y: scroll;
            border: 1px solid #ccc;
            padding: 10px;
        }
    </style>
</head>
<body>
    <h1>Chat Room: {{ room_name }}</h1>
    <div id="chat-log"></div><br>
    <input id="chat-message-input" type="text" size="100"><br>
    <input id="chat-message-submit" type="button" value="Send">
    {{ room_name|json_script:"room-name" }}
    <script>
        const roomName = JSON.parse(document.getElementById('room-name').textContent);

        const chatSocket = new WebSocket(
            'ws://'
            + window.location.host
            + '/ws/chat/'
            + roomName
            + '/'
        );

        chatSocket.onmessage = function(e) {
            const data = JSON.parse(e.data);
            const chatLog = document.querySelector('#chat-log');
            const message = document.createElement('div');
            message.innerHTML = `<strong>${data.username}</strong> [${new Date(data.timestamp).toLocaleString()}]: ${data.message}`;
            chatLog.appendChild(message);
            chatLog.scrollTop = chatLog.scrollHeight;
        };

        chatSocket.onclose = function(e) {
            console.error('Chat socket closed unexpectedly');
        };

        document.querySelector('#chat-message-input').focus();
        document.querySelector('#chat-message-input').onkeyup = function(e) {
            if (e.keyCode === 13) {  // enter, return
                document.querySelector('#chat-message-submit').click();
            }
        };

        document.querySelector('#chat-message-submit').onclick = function(e) {
            const messageInputDom = document.querySelector('#chat-message-input');
            const message = messageInputDom.value;
            chatSocket.send(JSON.stringify({
                'message': message
            }));
            messageInputDom.value = '';
        };
    </script>
</body>
</html>
```

Finally, add login/logout functionality. Update `chatproject/urls.py`:

```python
from django.contrib import admin
from django.urls import path, include
from django.contrib.auth.views import LoginView, LogoutView

urlpatterns = [
    path('admin/', admin.site.urls),
    path('chat/', include('chat.urls')),
    path('accounts/login/', LoginView.as_view(template_name='chat/login.html'), name='login'),
    path('accounts/logout/', LogoutView.as_view(next_page='/chat/'), name='logout'),
]
```

Create a login template at `chat/templates/chat/login.html`:

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8"/>
    <title>Login</title>
</head>
<body>
    <h1>Login</h1>
    <form method="post">
        {% csrf_token %}
        {{ form.as_p }}
        <button type="submit">Login</button>
    </form>
</body>
</html>
```

Update `chat/templates/chat/index.html` to include a logout link:

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8"/>
    <title>Chat Rooms</title>
</head>
<body>
    <h1>Welcome, {{ user.username }}!</h1>
    <a href="{% url 'logout' %}">Logout
```


</a>
    <h2>Chat Rooms</h2>
    <input id="room-name-input" type="text" size="100"><br>
    <input id="room-name-submit" type="button" value="Enter">

    `<script>`
        document.querySelector('#room-name-input').focus();
        document.querySelector('#room-name-input').onkeyup = function(e) {
            if (e.keyCode === 13) {  // enter, return
                document.querySelector('#room-name-submit').click();
            }
        };

    document.querySelector('#room-name-submit').onclick = function(e) {
            var roomName = document.querySelector('#room-name-input').value;
            window.location.pathname = '/chat/' + roomName + '/';
        };`</script>`

</body>
</html>
```

#### Step 10: Add User Online Status

Let's add a feature to show which users are currently online in a chat room. We'll need to modify our consumer and templates.

Update `chat/consumers.py`:

```python
import json
from channels.generic.websocket import AsyncWebsocketConsumer
from channels.db import database_sync_to_async
from .models import Room, Message
from django.contrib.auth.models import User
from django.utils import timezone

class ChatConsumer(AsyncWebsocketConsumer):
    async def connect(self):
        self.room_name = self.scope['url_route']['kwargs']['room_name']
        self.room_group_name = f'chat_{self.room_name}'
        self.user = self.scope["user"]

        # Join room group
        await self.channel_layer.group_add(
            self.room_group_name,
            self.channel_name
        )

        await self.accept()

        # Send existing messages to the newly connected client
        messages = await self.get_messages()
        for message in messages:
            await self.send(text_data=json.dumps({
                'message': message['content'],
                'username': message['username'],
                'timestamp': message['timestamp']
            }))

        # Add user to online users
        await self.add_user_to_online()

        # Send the updated online users list to all clients in the room
        await self.channel_layer.group_send(
            self.room_group_name,
            {
                'type': 'online_users_update',
            }
        )

    async def disconnect(self, close_code):
        # Remove user from online users
        await self.remove_user_from_online()

        # Leave room group
        await self.channel_layer.group_discard(
            self.room_group_name,
            self.channel_name
        )

        # Send the updated online users list to all clients in the room
        await self.channel_layer.group_send(
            self.room_group_name,
            {
                'type': 'online_users_update',
            }
        )

    async def receive(self, text_data):
        text_data_json = json.loads(text_data)
        message = text_data_json['message']

        # Save the message to the database
        await self.save_message(message)

        # Send message to room group
        await self.channel_layer.group_send(
            self.room_group_name,
            {
                'type': 'chat_message',
                'message': message,
                'username': self.user.username,
                'timestamp': str(timezone.now())
            }
        )

    async def chat_message(self, event):
        message = event['message']
        username = event['username']
        timestamp = event['timestamp']

        # Send message to WebSocket
        await self.send(text_data=json.dumps({
            'message': message,
            'username': username,
            'timestamp': timestamp
        }))

    async def online_users_update(self, event):
        online_users = await self.get_online_users()
        await self.send(text_data=json.dumps({
            'type': 'online_users_update',
            'online_users': online_users
        }))

    @database_sync_to_async
    def save_message(self, message):
        room, _ = Room.objects.get_or_create(name=self.room_name)
        Message.objects.create(room=room, user=self.user, content=message)

    @database_sync_to_async
    def get_messages(self):
        room, _ = Room.objects.get_or_create(name=self.room_name)
        messages = Message.objects.filter(room=room).order_by('-timestamp')[:50]
        return [{'content': message.content, 'username': message.user.username, 'timestamp': str(message.timestamp)} for message in messages]

    @database_sync_to_async
    def add_user_to_online(self):
        room, _ = Room.objects.get_or_create(name=self.room_name)
        room.online_users.add(self.user)

    @database_sync_to_async
    def remove_user_from_online(self):
        room, _ = Room.objects.get_or_create(name=self.room_name)
        room.online_users.remove(self.user)

    @database_sync_to_async
    def get_online_users(self):
        room, _ = Room.objects.get_or_create(name=self.room_name)
        return [user.username for user in room.online_users.all()]
```

Update `chat/models.py` to include the `online_users` field:

```python
from django.db import models
from django.contrib.auth.models import User

class Room(models.Model):
    name = models.CharField(max_length=255, unique=True)
    online_users = models.ManyToManyField(User, related_name='online_rooms', blank=True)

    def __str__(self):
        return self.name

class Message(models.Model):
    room = models.ForeignKey(Room, related_name='messages', on_delete=models.CASCADE)
    user = models.ForeignKey(User, related_name='messages', on_delete=models.CASCADE)
    content = models.TextField()
    timestamp = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return f'{self.user.username}: {self.content} [{self.timestamp}]'
```

Don't forget to run migrations after updating the model:

```bash
python manage.py makemigrations
python manage.py migrate
```

Now, update `chat/templates/chat/room.html` to display online users:

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8"/>
    <title>Chat Room</title>
    <style>
        #chat-log {
            height: 300px;
            overflow-y: scroll;
            border: 1px solid #ccc;
            padding: 10px;
        }
        #online-users {
            float: right;
            width: 200px;
            height: 300px;
            overflow-y: scroll;
            border: 1px solid #ccc;
            padding: 10px;
        }
    </style>
</head>
<body>
    <h1>Chat Room: {{ room_name }}</h1>
    <div id="online-users">
        <h3>Online Users</h3>
        <ul id="online-users-list"></ul>
    </div>
    <div id="chat-log"></div><br>
    <input id="chat-message-input" type="text" size="100"><br>
    <input id="chat-message-submit" type="button" value="Send">
    {{ room_name|json_script:"room-name" }}
    <script>
        const roomName = JSON.parse(document.getElementById('room-name').textContent);

        const chatSocket = new WebSocket(
            'ws://'
            + window.location.host
            + '/ws/chat/'
            + roomName
            + '/'
        );

        chatSocket.onmessage = function(e) {
            const data = JSON.parse(e.data);
            if (data.type === 'online_users_update') {
                updateOnlineUsers(data.online_users);
            } else {
                const chatLog = document.querySelector('#chat-log');
                const message = document.createElement('div');
                message.innerHTML = `<strong>${data.username}</strong> [${new Date(data.timestamp).toLocaleString()}]: ${data.message}`;
                chatLog.appendChild(message);
                chatLog.scrollTop = chatLog.scrollHeight;
            }
        };

        chatSocket.onclose = function(e) {
            console.error('Chat socket closed unexpectedly');
        };

        document.querySelector('#chat-message-input').focus();
        document.querySelector('#chat-message-input').onkeyup = function(e) {
            if (e.keyCode === 13) {  // enter, return
                document.querySelector('#chat-message-submit').click();
            }
        };

        document.querySelector('#chat-message-submit').onclick = function(e) {
            const messageInputDom = document.querySelector('#chat-message-input');
            const message = messageInputDom.value;
            chatSocket.send(JSON.stringify({
                'message': message
            }));
            messageInputDom.value = '';
        };

        function updateOnlineUsers(users) {
            const userList = document.querySelector('#online-users-list');
            userList.innerHTML = '';
            users.forEach(user => {
                const li = document.createElement('li');
                li.textContent = user;
                userList.appendChild(li);
            });
        }
    </script>
</body>
</html>
```

#### Step 11: Add Typing Indicator

Let's add a typing indicator to show when a user is typing a message. Update `chat/consumers.py`:

```python
# Add this new method to the ChatConsumer class
async def receive(self, text_data):
    text_data_json = json.loads(text_data)
    if 'typing' in text_data_json:
        # Broadcast that a user is typing
        await self.channel_layer.group_send(
            self.room_group_name,
            {
                'type': 'user_typing',
                'user': self.user.username,
                'typing': text_data_json['typing']
            }
        )
    elif 'message' in text_data_json:
        message = text_data_json['message']
        # Save the message to the database
        await self.save_message(message)
        # Send message to room group
        await self.channel_layer.group_send(
            self.room_group_name,
            {
                'type': 'chat_message',
                'message': message,
                'username': self.user.username,
                'timestamp': str(timezone.now())
            }
        )

# Add this new method to the ChatConsumer class
async def user_typing(self, event):
    # Send typing status to WebSocket
    await self.send(text_data=json.dumps({
        'type': 'typing',
        'user': event['user'],
        'typing': event['typing']
    }))
```

Now, update `chat/templates/chat/room.html` to include the typing indicator:

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8"/>
    <title>Chat Room</title>
    <style>
        #chat-log {
            height: 300px;
            overflow-y: scroll;
            border: 1px solid #ccc;
            padding: 10px;
        }
        #online-users {
            float: right;
            width: 200px;
            height: 300px;
            overflow-y: scroll;
            border: 1px solid #ccc;
            padding: 10px;
        }
        #typing-indicator {
            font-style: italic;
            color: #888;
        }
    </style>
</head>
<body>
    <h1>Chat Room: {{ room_name }}</h1>
    <div id="online-users">
        <h3>Online Users</h3>
        <ul id="online-users-list"></ul>
    </div>
    <div id="chat-log"></div><br>
    <div id="typing-indicator"></div>
    <input id="chat-message-input" type="text" size="100"><br>
    <input id="chat-message-submit" type="button" value="Send">
    {{ room_name|json_script:"room-name" }}
    <script>
        const roomName = JSON.parse(document.getElementById('room-name').textContent);

        const chatSocket = new WebSocket(
            'ws://'
            + window.location.host
            + '/ws/chat/'
            + roomName
            + '/'
        );

        chatSocket.onmessage = function(e) {
            const data = JSON.parse(e.data);
            if (data.type === 'online_users_update') {
                updateOnlineUsers(data.online_users);
            } else if (data.type === 'typing') {
                updateTypingIndicator(data.user, data.typing);
            } else {
                const chatLog = document.querySelector('#chat-log');
                const message = document.createElement('div');
                message.innerHTML = `<strong>${data.username}</strong> [${new Date(data.timestamp).toLocaleString()}]: ${data.message}`;
                chatLog.appendChild(message);
                chatLog.scrollTop = chatLog.scrollHeight;
            }
        };

        chatSocket.onclose = function(e) {
            console.error('Chat socket closed unexpectedly');
        };

        const messageInput = document.querySelector('#chat-message-input');
        let typingTimeout = null;

        messageInput.addEventListener('input', function() {
            chatSocket.send(JSON.stringify({
                'typing': true
            }));

            clearTimeout(typingTimeout);
            typingTimeout = setTimeout(() => {
                chatSocket.send(JSON.stringify({
                    'typing': false
                }));
            }, 3000);
        });

        messageInput.focus();
        messageInput.onkeyup = function(e) {
            if (e.keyCode === 13) {  // enter, return
                document.querySelector('#chat-message-submit').click();
            }
        };

        document.querySelector('#chat-message-submit').onclick = function(e) {
            const message = messageInput.value;
            chatSocket.send(JSON.stringify({
                'message': message
            }));
            messageInput.value = '';
            // Clear typing indicator when message is sent
            chatSocket.send(JSON.stringify({
                'typing': false
            }));
        };

        function updateOnlineUsers(users) {
            const userList = document.querySelector('#online-users-list');
            userList.innerHTML = '';
            users.forEach(user => {
                const li = document.createElement('li');
                li.textContent = user;
                userList.appendChild(li);
            });
        }

        function updateTypingIndicator(user, isTyping) {
            const typingIndicator = document.querySelector('#typing-indicator');
            if (isTyping) {
                typingIndicator.textContent = `${user} is typing...`;
            } else {
                typingIndicator.textContent = '';
            }
        }
    </script>
</body>
</html>
```

This completes our real-time chat application with the following features:

1. Real-time messaging using WebSockets
2. User authentication
3. Persistent chat history
4. Online users list
5. Typing indicator

This project demonstrates how to use Django Channels to implement real-time features in a Django application. It covers important concepts such as WebSocket consumers, channel layers, and integrating real-time functionality with traditional Django views and models.

Some potential improvements and additional features you could consider:


#### Potential Improvements and Additional Features

1. **Message Editing and Deletion**: Allow users to edit or delete their own messages.

   To implement this, you would need to:

   - Add new WebSocket message types for edit and delete operations.
   - Update the consumer to handle these new message types.
   - Modify the frontend to allow users to edit or delete their messages.
   - Ensure that only the message author can perform these actions.
2. **File Sharing**: Enable users to share files in the chat.

   Implementation steps:

   - Create a new model for storing file information.
   - Update the consumer to handle file upload messages.
   - Modify the frontend to allow file selection and display shared files.
   - Implement secure file storage and retrieval.
3. **Emojis and Rich Text**: Add support for emojis and rich text formatting.

   You could:

   - Integrate an emoji picker library in the frontend.
   - Use a rich text editor for the message input.
   - Update the message display to render formatted text and emojis.
4. **Private Messaging**: Allow users to send private messages to each other.

   This would involve:

   - Creating a new consumer for private messages.
   - Implementing a user interface for selecting message recipients.
   - Ensuring that private messages are only sent to the intended recipients.
5. **Message Reactions**: Let users react to messages with emojis.

   Implementation would include:

   - Adding a new model to store message reactions.
   - Updating the consumer to handle reaction messages.
   - Modifying the frontend to display and allow adding reactions.
6. **User Profiles**: Implement more detailed user profiles.

   You could:

   - Create a new model for user profiles with additional fields.
   - Add a profile page where users can view and edit their information.
   - Display user avatars in the chat interface.
7. **Message Search**: Add functionality to search through message history.

   This would involve:

   - Implementing a search view in Django.
   - Creating a search interface in the frontend.
   - Optimizing database queries for efficient searching.
8. **Notifications**: Implement a notification system for new messages and mentions.

   You could:

   - Use browser notifications for real-time alerts.
   - Implement an in-app notification system.
   - Add email notifications for offline users.
9. **Multiple Chat Rooms**: Allow users to create and join multiple chat rooms.

   This would require:

   - Updating the Room model to support user-created rooms.
   - Implementing a room management interface.
   - Modifying the consumer to handle multiple room subscriptions.
10. **Message Threading**: Implement threaded conversations.

    You would need to:

    - Modify the Message model to support parent-child relationships.
    - Update the consumer to handle threaded messages.
    - Adapt the frontend to display and interact with message threads.
11. **User Blocking**: Allow users to block others from sending them messages.

    Implementation steps:

    - Create a new model to store blocking relationships.
    - Update the consumer to check for blocks before delivering messages.
    - Add a user interface for blocking and unblocking users.
12. **Read Receipts**: Implement read receipts for messages.

    This would involve:

    - Adding a new field to the Message model to track read status.
    - Updating the consumer to mark messages as read and broadcast this information.
    - Modifying the frontend to display read status.
13. **Voice and Video Chat**: Integrate WebRTC for voice and video communication.

    This advanced feature would require:

    - Implementing WebRTC signaling using Django Channels.
    - Creating a user interface for initiating and accepting calls.
    - Handling peer-to-peer connections in the frontend.
14. **Chat Bots**: Implement chat bots for automated responses or commands.

    You could:

    - Create a bot framework within your Django application.
    - Implement command parsing in the consumer.
    - Design bot responses and actions.
15. **Message Analytics**: Add analytics to track message trends and user activity.

    This would involve:

    - Implementing data collection in the consumer and models.
    - Creating views to analyze and display the collected data.
    - Potentially integrating with a data visualization library.

To implement these features, you would need to:

1. Modify the existing models to accommodate new data structures.
2. Update the consumer to handle new types of WebSocket messages.
3. Implement new Django views and API endpoints as necessary.
4. Enhance the frontend JavaScript to support new user interactions and display new types of data.
5. Update the HTML templates to include new UI elements.
6. Ensure all new features are secure and perform well at scale.

Remember to thoroughly test each new feature, especially in terms of real-time performance and security. Additionally, consider the impact on the user experience and ensure that new features are intuitive and don't clutter the interface.

By implementing some or all of these features, you can transform the basic chat application into a full-featured communication platform. Each of these enhancements provides an opportunity to deepen your understanding of Django, Django Channels, and real-time web applications.

This concludes our series of project tutorials. These projects have covered a wide range of Django features and concepts, from basic CRUD operations to real-time communication. By working through these tutorials and expanding on them with the suggested improvements, you'll gain a comprehensive understanding of Django and its ecosystem.

In the next sections, we'll discuss best practices for structuring large-scale Django projects, integrating Django with other technologies, deployment strategies, common pitfalls to avoid, and resources for further learning.


## 5. Structuring Large-scale Django Projects

As Django projects grow in size and complexity, it becomes crucial to structure them in a way that promotes maintainability, scalability, and collaboration. Here are some best practices for structuring large-scale Django projects:

### 5.1. Project Layout

A well-organized project layout can significantly improve development efficiency. Here's a recommended structure for a large Django project:

```
project_name/
│
├── config/
│   ├── settings/
│   │   ├── base.py
│   │   ├── local.py
│   │   ├── production.py
│   │   └── test.py
│   ├── urls.py
│   └── wsgi.py
│
├── apps/
│   ├── accounts/
│   ├── blog/
│   └── api/
│
├── static/
│
├── templates/
│
├── media/
│
├── docs/
│
├── requirements/
│   ├── base.txt
│   ├── local.txt
│   └── production.txt
│
├── manage.py
├── .gitignore
├── README.md
└── .env
```

### 5.2. Settings Management

Split your settings into multiple files to handle different environments:

- `base.py`: Common settings shared across all environments.
- `local.py`: Development-specific settings.
- `production.py`: Production-specific settings.
- `test.py`: Settings for running tests.

Use environment variables for sensitive information:

```python
import os
from dotenv import load_dotenv

load_dotenv()

SECRET_KEY = os.getenv('DJANGO_SECRET_KEY')
DATABASE_PASSWORD = os.getenv('DATABASE_PASSWORD')
```

### 5.3. App Structure

Organize your project into smaller, focused apps. Each app should follow the Single Responsibility Principle. A typical app structure might look like this:

```
app_name/
├── migrations/
├── templates/
│   └── app_name/
├── static/
│   └── app_name/
├── templatetags/
├── management/
│   └── commands/
├── __init__.py
├── admin.py
├── apps.py
├── models.py
├── serializers.py
├── urls.py
├── views.py
├── forms.py
├── utils.py
└── tests.py
```

### 5.4. Custom User Model

Always use a custom user model, even if you don't need to customize it immediately:

```python
# accounts/models.py
from django.contrib.auth.models import AbstractUser

class User(AbstractUser):
    # Add custom fields here
    pass

# settings/base.py
AUTH_USER_MODEL = 'accounts.User'
```

### 5.5. Database Optimization

For large-scale projects, database optimization is crucial:

- Use `select_related()` and `prefetch_related()` to reduce database queries.
- Create appropriate indexes on your models.
- Use database-specific features when necessary (e.g., PostgreSQL-specific fields).

### 5.6. Caching

Implement caching to improve performance:

```python
# settings/base.py
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
        'LOCATION': '127.0.0.1:11211',
    }
}

# views.py
from django.views.decorators.cache import cache_page

@cache_page(60 * 15)  # Cache for 15 minutes
def my_view(request):
    # ...
```

### 5.7. Asynchronous Tasks

Use Celery for handling asynchronous tasks:

```python
# celery.py
from celery import Celery

app = Celery('project_name')
app.config_from_object('django.conf:settings', namespace='CELERY')
app.autodiscover_tasks()

# tasks.py
@app.task
def my_task():
    # Task logic here
```

### 5.8. API Versioning

If your project includes an API, implement versioning:

```python
# urls.py
from rest_framework import routers

router = routers.DefaultRouter()
router.register(r'v1/users', UserViewSet, basename='user')

urlpatterns = [
    path('api/', include(router.urls)),
]
```

### 5.9. Documentation

Maintain comprehensive documentation:

- Use docstrings for all functions, classes, and modules.
- Keep a README.md file with setup instructions and project overview.
- Consider using tools like Sphinx for generating documentation.

### 5.10. Testing

Implement thorough testing:

- Write unit tests for models, views, and utility functions.
- Use factories (e.g., Factory Boy) for generating test data.
- Implement integration tests for critical paths in your application.

```python
# tests.py
from django.test import TestCase
from .models import MyModel

class MyModelTestCase(TestCase):
    def setUp(self):
        MyModel.objects.create(name="Test")

    def test_my_model_str(self):
        my_model = MyModel.objects.get(name="Test")
        self.assertEqual(str(my_model), "Test")
```

## 6. Django Integration with Other Technologies

Django can be integrated with various other technologies to enhance its capabilities. Here are some common integrations:

### 6.1. Django REST framework

For building APIs:

```bash
pip install djangorestframework
```

```python
# settings.py
INSTALLED_APPS = [
    # ...
    'rest_framework',
]

# views.py
from rest_framework import viewsets
from .models import MyModel
from .serializers import MyModelSerializer

class MyModelViewSet(viewsets.ModelViewSet):
    queryset = MyModel.objects.all()
    serializer_class = MyModelSerializer
```

### 6.2. Celery

For asynchronous task processing:

```bash
pip install celery
```

```python
# celery.py
from celery import Celery

app = Celery('project_name')
app.config_from_object('django.conf:settings', namespace='CELERY')
app.autodiscover_tasks()

# tasks.py
@app.task
def my_task():
    # Task logic here
```

### 6.3. Redis

As a cache backend or message broker:

```bash
pip install django-redis
```

```python
# settings.py
CACHES = {
    "default": {
        "BACKEND": "django_redis.cache.RedisCache",
        "LOCATION": "redis://127.0.0.1:6379/1",
        "OPTIONS": {
            "CLIENT_CLASS": "django_redis.client.DefaultClient",
        }
    }
}
```

### 6.4. Elasticsearch

For advanced search capabilities:

```bash
pip install elasticsearch-dsl
```

```python
from elasticsearch_dsl import Document, Text
from elasticsearch_dsl.connections import connections

connections.create_connection(hosts=['localhost'])

class ArticleDocument(Document):
    title = Text()
    content = Text()

    class Index:
        name = 'articles'

ArticleDocument.init()
```

### 6.5. Docker

For containerization:

```dockerfile
# Dockerfile
FROM python:3.9

ENV PYTHONUNBUFFERED 1

WORKDIR /app

COPY requirements.txt /app/
RUN pip install -r requirements.txt

COPY . /app/

CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
```

### 6.6. GraphQL

For flexible API queries:

```bash
pip install graphene-django
```

```python
# schema.py
import graphene
from graphene_django import DjangoObjectType
from .models import MyModel

class MyModelType(DjangoObjectType):
    class Meta:
        model = MyModel

class Query(graphene.ObjectType):
    my_models = graphene.List(MyModelType)

    def resolve_my_models(self, info):
        return MyModel.objects.all()

schema = graphene.Schema(query=Query)
```

## 7. Deployment Strategies

Deploying Django applications requires careful planning and consideration of various factors:

### 7.1. Hosting Platforms

- Heroku: Easy deployment with Git integration.
- DigitalOcean: Provides more control over server configuration.
- AWS: Offers a wide range of services for scalable deployments.

### 7.2. Web Servers

- Gunicorn: Lightweight WSGI HTTP Server.
- uWSGI: Fast, self-healing and developer/sysadmin-friendly.

### 7.3. Reverse Proxy

- Nginx: High-performance HTTP server and reverse proxy.

### 7.4. Database

- PostgreSQL: Recommended for production use.

### 7.5. Static Files

- Use Django's `collectstatic` command.
- Serve static files from a CDN for better performance.

### 7.6. Security

- Set `DEBUG = False` in production.
- Use environment variables for sensitive information.
- Implement HTTPS using Let's Encrypt.
- Keep dependencies up-to-date.

### 7.7. Monitoring

- Use tools like New Relic or Sentry for application monitoring.

### 7.8. Continuous Integration/Continuous Deployment (CI/CD)

- Use tools like Jenkins, GitLab CI, or GitHub Actions for automated testing and deployment.

## 8. Common Pitfalls and How to Avoid Them

1. **N+1 Query Problem**: Use `select_related()` and `prefetch_related()` to optimize database queries.
2. **Not Using Database Indexes**: Add indexes to fields that are frequently used in filters and ordering.
3. **Ignoring Database Migrations**: Always run migrations after changing models.
4. **Security Vulnerabilities**: Keep Django and all dependencies up-to-date, use Django's built-in security features.
5. **Poor Performance**: Profile your application, implement caching, and optimize database queries.
6. **Not Writing Tests**: Implement comprehensive unit and integration tests.
7. **Overusing Generic Views**: While convenient, sometimes custom views provide more flexibility.
8. **Tight Coupling**: Use signals and custom managers to keep your code modular.
9. **Ignoring Code Style**: Follow PEP 8 and use tools like flake8 or black for consistent code formatting.
10. **Not Handling Errors Properly**: Implement proper error handling and logging.

## 9. Resources for Further Learning

1. **Official Django Documentation**: https://docs.djangoproject.com/
2. **Django REST framework Documentation**: https://www.django-rest-framework.org/
3. **Two Scoops of Django**: A comprehensive guide to best practices by Daniel and Audrey Roy Greenfeld.
4. **Django for Professionals**: By William S. Vincent, covers advanced Django concepts.
5. **TestDriven.io**: Offers in-depth Django tutorials and courses.
6. **Real Python**: Provides numerous Django tutorials and articles.
7. **DjangoGirls Tutorial**: A beginner-friendly introduction to Django.
8. **Django Channels Documentation**: For learning about WebSockets and asynchronous Django.
9. **Django Weekly**: A weekly newsletter with Django news and articles.
10. **Django Chat Podcast**: Discussions about Django and related technologies.

Remember, the best way to learn is by building projects and contributing to open-source Django projects. Keep coding, stay curious, and don't hesitate to ask questions in Django forums and communities!

This concludes our comprehensive guide to Django for intermediate developers. We've covered a wide range of topics from basic concepts to advanced features and best practices. As you continue your Django journey, remember that there's always more to learn and explore in this powerful framework.


## 10. Advanced Django Topics

### 10.1. Custom Template Tags and Filters

Custom template tags and filters can greatly enhance the functionality of your templates:

```python
# myapp/templatetags/custom_tags.py
from django import template

register = template.Library()

@register.simple_tag
def my_custom_tag(value):
    return f"Processed: {value}"

@register.filter
def add_prefix(value, arg):
    return f"{arg}: {value}"
```

Usage in templates:

```html
{% load custom_tags %}

{% my_custom_tag "Hello" %}
{{ "World" | add_prefix:"Greeting" }}
```

### 10.2. Middleware

Middleware allows you to process requests and responses globally:

```python
# myapp/middleware.py
class SimpleMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):
        # Code to be executed for each request before
        # the view (and later middleware) are called.

        response = self.get_response(request)

        # Code to be executed for each request/response after
        # the view is called.

        return response
```

Add to `MIDDLEWARE` in settings.py:

```python
MIDDLEWARE = [
    # ...
    'myapp.middleware.SimpleMiddleware',
]
```

### 10.3. Custom Management Commands

Create custom management commands for routine tasks:

```python
# myapp/management/commands/my_command.py
from django.core.management.base import BaseCommand

class Command(BaseCommand):
    help = 'Description of my custom command'

    def add_arguments(self, parser):
        parser.add_argument('sample', type=str)

    def handle(self, *args, **options):
        self.stdout.write(self.style.SUCCESS(f"Command executed with {options['sample']}"))
```

Run with:

```bash
python manage.py my_command sample_arg
```

### 10.4. Django Signals

Signals allow certain senders to notify a set of receivers that some action has taken place:

```python
# myapp/signals.py
from django.db.models.signals import post_save
from django.dispatch import receiver
from django.contrib.auth.models import User
from .models import Profile

@receiver(post_save, sender=User)
def create_user_profile(sender, instance, created, **kwargs):
    if created:
        Profile.objects.create(user=instance)

@receiver(post_save, sender=User)
def save_user_profile(sender, instance, **kwargs):
    instance.profile.save()
```

### 10.5. Custom Model Managers

Custom managers can encapsulate common query operations:

```python
# myapp/models.py
from django.db import models

class PublishedManager(models.Manager):
    def get_queryset(self):
        return super().get_queryset().filter(status='published')

class Article(models.Model):
    # ... fields ...
    objects = models.Manager()  # The default manager.
    published = PublishedManager()  # Our custom manager.
```

### 10.6. Database Transactions

Ensure database consistency with transactions:

```python
from django.db import transaction

@transaction.atomic
def viewfunc(request):
    # This code executes inside a transaction.
    do_stuff()
```

### 10.7. Caching Strategies

Implement various caching strategies to improve performance:

```python
from django.core.cache import cache

# View caching
from django.views.decorators.cache import cache_page

@cache_page(60 * 15)
def my_view(request):
    # ...

# Low-level cache API
cache.set('my_key', 'my_value', 300)  # Cache for 5 minutes
value = cache.get('my_key')

# Template fragment caching
{% load cache %}
{% cache 500 sidebar %}
    ... sidebar ...
{% endcache %}
```

### 10.8. Internationalization and Localization

Make your Django project multilingual:

```python
# settings.py
MIDDLEWARE = [
    # ...
    'django.middleware.locale.LocaleMiddleware',
]

LANGUAGE_CODE = 'en-us'
USE_I18N = True
USE_L10N = True

LANGUAGES = [
    ('en', 'English'),
    ('es', 'Spanish'),
]

# views.py
from django.utils.translation import gettext as _

def my_view(request):
    output = _("Welcome to my site.")
    return render(request, 'my_template.html', {'output': output})

# Template
{% load i18n %}
<h1>{% trans "Welcome" %}</h1>
```

### 10.9. Django REST Framework Advanced Features

Explore advanced DRF features like viewsets, routers, and authentication:

```python
from rest_framework import viewsets, permissions
from .models import MyModel
from .serializers import MyModelSerializer

class MyModelViewSet(viewsets.ModelViewSet):
    queryset = MyModel.objects.all()
    serializer_class = MyModelSerializer
    permission_classes = [permissions.IsAuthenticatedOrReadOnly]

# urls.py
from rest_framework.routers import DefaultRouter
from .views import MyModelViewSet

router = DefaultRouter()
router.register(r'mymodels', MyModelViewSet)

urlpatterns = [
    path('api/', include(router.urls)),
]
```

### 10.10. Testing Best Practices

Implement comprehensive testing strategies:

```python
from django.test import TestCase
from django.urls import reverse
from .models import MyModel

class MyModelTestCase(TestCase):
    def setUp(self):
        MyModel.objects.create(name="Test Model")

    def test_model_creation(self):
        test_model = MyModel.objects.get(name="Test Model")
        self.assertEqual(test_model.name, "Test Model")

    def test_view(self):
        response = self.client.get(reverse('my_view'))
        self.assertEqual(response.status_code, 200)
        self.assertContains(response, "Expected Content")
```

## Conclusion

This comprehensive guide has covered a wide range of Django topics, from basic concepts to advanced features and best practices. As you continue your journey with Django, remember these key points:

1. **Keep Learning**: Django is constantly evolving. Stay updated with the latest features and best practices.
2. **Contribute to Open Source**: Contributing to Django or Django-related projects is a great way to improve your skills and give back to the community.
3. **Prioritize Security**: Always follow Django's security best practices and keep your dependencies up-to-date.
4. **Performance Matters**: As your projects grow, pay attention to performance optimization techniques like caching and database query optimization.
5. **Write Clean, Maintainable Code**: Follow Python and Django coding standards, write tests, and document your code.
6. **Leverage the Ecosystem**: Django has a rich ecosystem of third-party packages. Use them wisely to avoid reinventing the wheel.
7. **Scalability Considerations**: Design your applications with scalability in mind from the beginning.
8. **Stay Connected**: Engage with the Django community through forums, conferences, and local meetups.

Remember, becoming proficient in Django is a journey. Each project you build will bring new challenges and opportunities to learn. Embrace the learning process, stay curious, and happy coding!

This concludes our comprehensive Django guide for intermediate developers. We hope this resource helps you in your Django development journey and serves as a reference for your future projects. Good luck with your Django adventures!

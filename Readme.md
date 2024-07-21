# Django View Authorization

- [Django View Authorization](#django-view-authorization)
  - [Summary](#summary)
  - [Building the cloned respository](#building-the-cloned-respository)
  - [Starting from the beginning](#starting-from-the-beginning)
  - [Detecting Logged-In Users and Their Roles in a View](#detecting-logged-in-users-and-their-roles-in-a-view)
  - [Implementing Django View Authorization](#implementing-django-view-authorization)
    - [Restricting Views to Logged-in Users](#restricting-views-to-logged-in-users)
    - [Restricting views to Admin and Staff](#restricting-views-to-admin-and-staff)
  - [Messaging a Logged-In User](#messaging-a-logged-in-user)

## Summary

![](django-view-authorization.gif)

A sample Movie application that uses user authentication and authorization.

Features:

- `HttpRequest` and `HttpRequest.user` objects
- `Authenticate` and `authorize` users
- differentiate among `regular`, `staff`, and `admin` users
- Secure a view with the `@login_required` decorator
- Restrict a view to different roles with the `@user_passes_test` decorator
- Use the Django `messages framework` to notify users

## Building the cloned respository

`git clone https://github.com/sunilale0/django-view-authorization.git`

```shell
cd django-view-authorization
```

set up a virtual environment

```shell
pip install -r requirements.txt
django manage.py migrate
django manage.py createsuperuser
python manage.py runserver
```

visit `localhost:8000/admin/` to log in and create Movie posts and add new users that are 1. staff user and 2. normal user. Skimm through the tutorial below to test various urls for authentication.

## Starting from the beginning

I started by opening my vscode in a new folder. If I run the regular `django-admin startproject 'project_name'`, it will create two level of folders with the same name, i.e `project_name/project_name/` where the second one will be the main project. Instead, I want to build the project on the same directory I am working on. The command that worked:

```shell
django-admin startproject Movie .
```

Some more commands to run on terminal/shell:

```shell
python manage.py startapp core
mkdir templates
python manage.py migrate
python manage.py createsuperuser
```

Add the new app `core` to the `INSTALLED_APPS` in `Movie/settings.py`

```python
INSTALLED_APPS = [
    # ... ,
    "core",
]
```

include the directory of `templates` in `Movie/setting.py`:

```python
TEMPLATES = [
    {
        # ...,
        "DIRS": [os.path.join(BASE_DIR, "templates")],
        # ...,
    },
]
```

Add the following to `core/models.py`:

```python
from django.db import models

class Movie(models.Model):
    title = models.CharField(max_length=50)
    content = models.TextField()
```

Create two views in `core/views.py`

1. List all the Movies
2. View a Movie

```python
from django.http import HttpResponse
from django.shortcuts import render, get_object_or_404
from core.models import Movie

def listing(request):
    data = {
        "Movies": Movie.objects.all(),
    }
    return render(request, "listing.html", data)

def view_Movie(request, Movie_id):
    Movie = get_object_or_404(Movie, id=Movie_id)
    data = {
        "Movie": Movie,
    }
    return render(request, "view_Movie.html", data)

```

3 template files go with the two views `templates/listing.html`

1. base template in `templates/base.html`

```html
<html>
  <body>
    {% block content %} {% endblock content %}
  </body>
</html>
```

2. template to list all Movies `templates/listing.html`

```html
{% extends "base.html" %} {% block content %}
<h1>Movie Listing</h1>
<ul>
  {% for Movie in Movies %}
  <li><a href="{% url 'view_Movie' Movie.id %}">{{ Movie.title }}</a></li>
  {% endfor %}
</ul>
{% endblock content%}
```

3. template to view each of the Movies `templates/view_Movie.html`

```html
{% extends "base.html" %} {% block content %}

<h1>{{ Movie.title }}</h1>
{{ Movie.content | safe }}

<hr />

<a href="{% url 'listing' %}">All Movies</a>
{% endblock content %}
```

`templates/listing.html` and `templates/view_Movie.html` files use the `{% url %}` tag to look up the URLs associated with the `listing()` and `view_Movie()` views. These URLs need to be registered in `Movie/urls.py`. Add the following in `Movie/urls.py`:

```python
# from ...
from django.urls import path, include
from core import views as core_views

urlpatterns=[
    # ... ,
    path("", core_views.listing, name="listing"),
    path("view_Movie/<int:Movie_id>/", core_views.view_Movie, name="view_Movie"),
]

```

So far, we have not created a way to creat content. we can add that feature by registering `Movie` object in `core/admin.py`


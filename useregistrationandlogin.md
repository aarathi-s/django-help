Exercise:- 22                                                 Date:-07/09/2026

Question:  Develop a Django application to implement user registration and login using Django's built-in authentication system.
Aarathi Sankar
Roll No.: 1

Commands for Starting a New Project
mkdir authproject
cd authproject

python -m venv venv venv\Scripts\activate

pip install django
django-admin startproject accounts
OR
Pythom -m Django startproject accounts 
Commands for Starting a New App
python manage.py startapp config 
Commands for Creating Model and Migration
py manage.py makemigrations
py manage.py migrate 

Commands for Creating and Changing the Credentials of Admin
python manage.py createsuperuser
python manage.py changepassword admin

Commands to Run the Application
python manage.py runserver

Project Folder Structure
 


Project – urls.py
from django.contrib import admin from django.urls import path,include

urlpatterns = [
path('admin/', admin.site.urls), path('', include('accounts.urls'))
]
Project – settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    ' inventory',
]

App-<App-accounts> urls.py
from django.contrib.auth import views as auth_views
from django.urls import path

from . import views

urlpatterns = [
    path("", views.home_view, name="home"),
    path("register/", views.register_view, name="register"),
    path("login/", auth_views.LoginView.as_view(template_name="accounts/login.html"),
        name="login",),
    path("logout/", auth_views.LogoutView.as_view(), name="logout"),
]
App – views.py
from django.contrib import messages
from django.contrib.auth import login
from django.contrib.auth.decorators import login_required
from django.shortcuts import redirect, render
from .forms import RegisterForm

def register_view(request):
    if request.user.is_authenticated:
        return redirect("home")

    if request.method == "POST":
        form = RegisterForm(request.POST)
        if form.is_valid():
            user = form.save()
            login(request, user)  # log the user in right after registering
            messages.success(request, f"Welcome, {user.username}! Your account was created successfully.")
            return redirect("home")
    else:
        form = RegisterForm()

    return render(request, "accounts/register.html", {"form": form})

@login_required
def home_view(request):
    return render(request, "accounts/home.html")
.html file
register.html

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Register</title>
</head>
<body style="font-family: Arial, sans-serif; max-width: 400px; margin: 60px auto;">
    <h1>Create an account</h1>

    {% if messages %}
        <ul>
            {% for message in messages %}
                <li>{{ message }}</li>
            {% endfor %}
        </ul>
    {% endif %}

    <form method="post" novalidate>
        {% csrf_token %}
        {{ form.as_p }}
        <button type="submit">Register</button>
    </form>
    <p>Already have an account? <a href="{% url 'login' %}">Login here</a></p>
</body>
</html>

login.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Login</title>
</head>
<body style="font-family: Arial, sans-serif; max-width: 400px; margin: 60px auto;">
    <h1>Login</h1>

    {% if messages %}
        <ul>
            {% for message in messages %}
                <li>{{ message }}</li>
            {% endfor %}
        </ul>
    {% endif %}

    <form method="post" novalidate>
        {% csrf_token %}
        {{ form.as_p }}
        <button type="submit">Login</button>
    </form>
    <p>Don't have an account? <a href="{% url 'register' %}">Register here</a></p>
</body>
</html>

home.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Home</title>
</head>
<body style="font-family: Arial, sans-serif; max-width: 400px; margin: 60px auto;">
    <h1>Welcome, {{ user.username }}!</h1>
    <p>You are logged in. This page is protected by Django's <code>@login_required</code> decorator.</p>
    <p>Your email on file: <strong>{{ user.email|default:"(none provided)" }}</strong></p>

    <form action="{% url 'logout' %}" method="post" style="margin-top: 20px;">

        {% csrf_token %}
        <button type="submit">Logout</button>
    </form>
</body>
</html>
 
 




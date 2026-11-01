Exercise:- 21                                                    Date:-07/09/2026
Question:  Develop a Campus Maintenance Request System using Django to allow students and staff to report maintenance issues.

Implement user registration, login and logout using Django authentication. Create a UserProfile or use Django's built-in User model to store user details.
Create a MaintenanceRequest model with fields such as Title, Description, Category, Location, Date Reported, Priority, Status and User.
Create a form that allows authenticated users to submit a maintenance request and implement CRUD operations for their own requests.
Display all requests submitted by the logged-in user and provide a search/filter option based on category, priority or status.
Create a Staff/Admin page to view all maintenance requests and update their status as Pending, In Progress or Completed.
Design suitable Home, Login, Register, Submit Request, My Requests and Admin/Staff pages using Django templates, template inheritance, URL routing and CSS.
Aarathi Sankar    Roll No.: 1
Commands for Starting a New Project
mkdir campus_maintenance_project
cd campus_maintenance_project

python -m venv venv
venv\Scripts\activate

pip install django

django-admin startproject campus_maintenance .
python manage.py startapp maintenance

mkdir templates
mkdir static
mkdir static\css
Commands for Starting a New App
python manage.py startapp maintenance
Commands for Creating Model and Migration

python manage.py migrate
Commands for Creating and changing the credentials of Admin
python manage.py createsuperuser

python manage.py changepassword admin


Commands to run the application
python manage.py runserver
Project Folder Structure



Project – urls.py

from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('maintenance.urls')),
]
Project- Settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',

    ‘maintenance’,
]
App-<App-profilee> urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('', views.home, name='home'),
    path('register/', views.register_view, name='register'),
    path('login/', views.login_view, name='login'),
    path('logout/', views.logout_view, name='logout'),

    path('submit/', views.submit_request, name='submit_request'),
    path('my-requests/', views.my_requests, name='my_requests'),
    path('request/edit/<int:request_id>/', views.edit_request, name='edit_request'),
    path('request/delete/<int:request_id>/', views.delete_request, name='delete_request'),

    path('staff/requests/', views.staff_requests, name='staff_requests'),
    path('staff/request/<int:request_id>/status/', views.update_status, name='update_status'),
]
App – views.py
from django.shortcuts import render, redirect, get_object_or_404
from django.contrib import messages
from django.contrib.auth import authenticate, login, logout
from django.contrib.auth.decorators import login_required

from .forms import RegisterForm, MaintenanceRequestForm
from .models import MaintenanceRequest


# ---------------------- HOME ----------------------

def home(request):
    return render(request, 'home.html')


# ---------------------- AUTHENTICATION ----------------------

def register_view(request):
    if request.method == 'POST':
        form = RegisterForm(request.POST)
        if form.is_valid():
            form.save()
            messages.success(request, 'Registration successful. Please login.')
            return redirect('login')
    else:
        form = RegisterForm()
    return render(request, 'register.html', {'form': form})


def login_view(request):
    if request.method == 'POST':
        username = request.POST.get('username')
        password = request.POST.get('password')
        user = authenticate(request, username=username, password=password)

        if user is not None:
            login(request, user)
            messages.success(request, 'Welcome, ' + user.username + '!')
            if user.is_staff:
                return redirect('staff_requests')
            return redirect('my_requests')
        else:
            messages.error(request, 'Invalid username or password.')

    return render(request, 'login.html')


def logout_view(request):
    logout(request)
    messages.success(request, 'You have been logged out.')
    return redirect('home')


# ---------------------- USER: CRUD ----------------------

@login_required
def submit_request(request):
    if request.method == 'POST':
        form = MaintenanceRequestForm(request.POST)
        if form.is_valid():
            maintenance_request = form.save(commit=False)
            maintenance_request.user = request.user
            maintenance_request.save()
            messages.success(request, 'Maintenance request submitted successfully.')
            return redirect('my_requests')
    else:
        form = MaintenanceRequestForm()
    return render(request, 'submit_request.html', {'form': form})


@login_required
def my_requests(request):
    maintenance_requests = MaintenanceRequest.objects.filter(user=request.user)

    search = request.GET.get('search', '')
    category = request.GET.get('category', '')
    priority = request.GET.get('priority', '')
    status = request.GET.get('status', '')

    if search:
        maintenance_requests = maintenance_requests.filter(title__icontains=search)
    if category:
        maintenance_requests = maintenance_requests.filter(category=category)
    if priority:
        maintenance_requests = maintenance_requests.filter(priority=priority)
    if status:
        maintenance_requests = maintenance_requests.filter(status=status)

    context = {
        'maintenance_requests': maintenance_requests,
        'category_choices': MaintenanceRequest.CATEGORY_CHOICES,
        'priority_choices': MaintenanceRequest.PRIORITY_CHOICES,
        'status_choices': MaintenanceRequest.STATUS_CHOICES,
        'selected_search': search,
        'selected_category': category,
        'selected_priority': priority,
        'selected_status': status,
    }
    return render(request, 'my_requests.html', context)


@login_required
def edit_request(request, request_id):
    maintenance_request = get_object_or_404(
        MaintenanceRequest, id=request_id, user=request.user
    )

    if request.method == 'POST':
        form = MaintenanceRequestForm(request.POST, instance=maintenance_request)
        if form.is_valid():
            form.save()
            messages.success(request, 'Request updated successfully.')
            return redirect('my_requests')
    else:
        form = MaintenanceRequestForm(instance=maintenance_request)

    return render(request, 'edit_request.html', {
        'form': form,
        'maintenance_request': maintenance_request,
    })


@login_required
def delete_request(request, request_id):
    maintenance_request = get_object_or_404(
        MaintenanceRequest, id=request_id, user=request.user
    )

    if request.method == 'POST':
        maintenance_request.delete()
        messages.success(request, 'Request deleted successfully.')
        return redirect('my_requests')

    return render(request, 'confirm_delete.html', {
        'maintenance_request': maintenance_request,
    })


# ---------------------- STAFF DASHBOARD ----------------------

@login_required
def staff_requests(request):
    if not request.user.is_staff:
        messages.error(request, 'Access denied. Only staff can open the staff dashboard.')
        return redirect('home')

    maintenance_requests = MaintenanceRequest.objects.all().select_related('user')

    category = request.GET.get('category', '')
    priority = request.GET.get('priority', '')
    status = request.GET.get('status', '')

    if category:
        maintenance_requests = maintenance_requests.filter(category=category)
    if priority:
        maintenance_requests = maintenance_requests.filter(priority=priority)
    if status:
        maintenance_requests = maintenance_requests.filter(status=status)

    context = {
        'maintenance_requests': maintenance_requests,
        'category_choices': MaintenanceRequest.CATEGORY_CHOICES,
        'priority_choices': MaintenanceRequest.PRIORITY_CHOICES,
        'status_choices': MaintenanceRequest.STATUS_CHOICES,
        'selected_category': category,
        'selected_priority': priority,
        'selected_status': status,
    }
    return render(request, 'staff_requests.html', context)


@login_required
def update_status(request, request_id):
    if not request.user.is_staff:
        messages.error(request, 'Access denied. Only staff can change the status.')
        return redirect('home')

    maintenance_request = get_object_or_404(MaintenanceRequest,
App-<App-profilee> urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('', views.home, name='home'),
    path('register/', views.register_view, name='register'),
    path('login/', views.login_view, name='login'),
    path('logout/', views.logout_view, name='logout'),

    path('submit/', views.submit_request, name='submit_request'),
    path('my-requests/', views.my_requests, name='my_requests'),
    path('request/edit/<int:request_id>/', views.edit_request, name='edit_request'),
    path('request/delete/<int:request_id>/', views.delete_request, name='delete_request'),

    path('staff/requests/', views.staff_requests, name='staff_requests'),
    path('staff/request/<int:request_id>/status/', views.update_status, name='update_status'),
]

base.html
{% load static %}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}Campus Maintenance Request System{% endblock %}</title>
    <link rel="stylesheet" href="{% static 'css/style.css' %}">
</head>
<body>

    <nav class="navbar">
        <a class="brand" href="{% url 'home' %}">Campus Maintenance</a>
        <div class="nav-links">
            <a href="{% url 'home' %}">Home</a>
            {% if user.is_authenticated %}
                <a href="{% url 'submit_request' %}">Submit Request</a>
                <a href="{% url 'my_requests' %}">My Requests</a>
                {% if user.is_staff %}
                    <a href="{% url 'staff_requests' %}">Staff Dashboard</a>
                {% endif %}
                <span class="nav-user">Hello, {{ user.username }}</span>
                <a href="{% url 'logout' %}">Logout</a>
            {% else %}
                <a href="{% url 'login' %}">Login</a>
                <a href="{% url 'register' %}">Register</a>
            {% endif %}
        </div>
    </nav>

    <main class="container">
        {% if messages %}
            {% for message in messages %}
                <div class="message {{ message.tags }}">{{ message }}</div>
            {% endfor %}
        {% endif %}

        {% block content %}
        {% endblock %}
    </main>

    <footer class="footer">
        <p>&copy; 2026 Campus Maintenance Request System | College Project</p>
    </footer>

</body>
</html>



home.html

{% extends 'base.html' %}

{% block title %}Home - Campus Maintenance{% endblock %}

{% block content %}
<div class="hero">
    <h1>Campus Maintenance Request System</h1>
    <p>Report campus maintenance issues quickly and track their resolution status.</p>

    <div class="hero-buttons">
        {% if user.is_authenticated %}
            <a href="{% url 'submit_request' %}" class="btn btn-primary">Submit Request</a>
            <a href="{% url 'my_requests' %}" class="btn btn-secondary">My Requests</a>
            {% if user.is_staff %}
                <a href="{% url 'staff_requests' %}" class="btn btn-success">Staff Dashboard</a>
            {% endif %}
        {% else %}
            <a href="{% url 'login' %}" class="btn btn-primary">Login</a>
            <a href="{% url 'register' %}" class="btn btn-secondary">Register</a>
        {% endif %}
    </div>
</div>

<div class="card-grid">
    <div class="card">
        <h3>1. Report</h3>
        <p>Submit a request for electrical, plumbing, cleaning, furniture, internet, classroom or laboratory problems.</p>
    </div>
    <div class="card">
        <h3>2. Track</h3>
        <p>See the current status of every request: Pending, In Progress or Completed.</p>
    </div>
    <div class="card">
        <h3>3. Resolve</h3>
        <p>Campus staff review the requests and update the status once work is done.</p>
    </div>
</div>
{% endblock %}

Request_form.html 

{% extends 'base.html' %}
{% block title %}{{ heading }}{% endblock %}
{% block content %}
<div class="card narrow">
  <h2>{{ heading }}</h2>
  <form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button class="btn">Save</button>
    <a class="btn alt" href="{% url 'my_requests' %}">Cancel</a>
  </form>
</div>
{% endblock %}


staff_dashboard.html

{% extends 'base.html' %}
{% block title %}Staff Page{% endblock %}
{% block content %}
<h2>All Maintenance Requests</h2>
<table>
  <tr><th>Title</th><th>Reported By</th><th>Category</th><th>Location</th><th>Priority</th><th>Date</th><th>Status</th></tr>
  {% for r in items %}
  <tr>
    <td>{{ r.title }}<br><small>{{ r.description|truncatechars:60 }}</small></td>
    <td>{{ r.user.username }}</td>
    <td>{{ r.category }}</td>
    <td>{{ r.location }}</td>
    <td><span class="tag {{ r.priority|lower }}">{{ r.priority }}</span></td>
    <td>{{ r.date_reported|date:"d M Y" }}</td>
    <td>
      <form method="post" class="inline">
        {% csrf_token %}
        <input type="hidden" name="id" value="{{ r.pk }}">
        <select name="status">
          {% for v, l in statuses %}<option value="{{ v }}" {% if r.status == v %}selected{% endif %}>{{ l }}</option>{% endfor %}
        </select>
        <button class="btn small">Update</button>
      </form>
    </td>
  </tr>
  {% empty %}
  <tr><td colspan="7">No requests yet.</td></tr>
  {% endfor %}
</table>
{% endblock %}

 


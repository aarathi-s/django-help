CAMPUS MAINTENANCE REQUEST SYSTEM
Django Practical Record – Complete Source Code
Aim: To develop a Campus Maintenance Request System using Django that allows students to register, log in, submit and manage maintenance requests, while staff can view all requests and update their status.
1. Software Requirements
•	Python 3
•	Django
•	Visual Studio Code (or another code editor)
•	SQLite database (Django default)
•	Web browser
2. Project Setup Commands
Create project and application
File type: Terminal commands
django-admin startproject campus_project
cd campus_project
python manage.py startapp maintenance
Add 'maintenance' to INSTALLED_APPS in campus_project/settings.py. Also set LOGIN_URL = '/login/'.
3. Project Folder Structure
Folder structure
File type: Project structure
campus_project/
│
├── campus_project/
│   ├── settings.py
│   └── urls.py
│
├── maintenance/
│   ├── migrations/
│   ├── templates/
│   │   ├── base.html
│   │   ├── index.html
│   │   ├── register.html
│   │   ├── login.html
│   │   ├── submit_request.html
│   │   ├── my_requests.html
│   │   ├── delete_request.html
│   │   ├── staff_requests.html
│   │   └── update_status.html
│   ├── static/
│   │   └── style.css
│   ├── models.py
│   ├── forms.py
│   ├── views.py
│   ├── urls.py
│   └── admin.py
│
└── manage.py
4. Database Model – maintenance/models.py
models.py
File type: Python
# Import Django model classes
from django.db import models
from django.contrib.auth.models import User
 
 
# Maintenance request database table
class MaintenanceRequest(models.Model):
 
    CATEGORY = [
        ('Electrical', 'Electrical'),
        ('Plumbing', 'Plumbing'),
        ('Furniture', 'Furniture'),
        ('Cleaning', 'Cleaning'),
        ('Other', 'Other'),
    ]
 
    PRIORITY = [
        ('Low', 'Low'),
        ('Medium', 'Medium'),
        ('High', 'High'),
    ]
 
    STATUS = [
        ('Pending', 'Pending'),
        ('In Progress', 'In Progress'),
        ('Completed', 'Completed'),
    ]
 
    title = models.CharField(max_length=100)
    description = models.TextField()
    category = models.CharField(max_length=20, choices=CATEGORY)
    location = models.CharField(max_length=100)
    date_reported = models.DateField(auto_now_add=True)
    priority = models.CharField(max_length=10, choices=PRIORITY)
    status = models.CharField(
        max_length=20,
        choices=STATUS,
        default='Pending'
    )
 
    # Connect each request to the user who submitted it
    user = models.ForeignKey(User, on_delete=models.CASCADE)
 
    def __str__(self):
        return self.title
5. Form – maintenance/forms.py
forms.py
File type: Python
# Import Django forms and the model
from django import forms
from .models import MaintenanceRequest
 
 
# Create a form automatically from the model
class MaintenanceRequestForm(forms.ModelForm):
 
    class Meta:
        model = MaintenanceRequest
        fields = [
            'title',
            'description',
            'category',
            'location',
            'priority'
        ]
The form excludes date_reported because it is generated automatically, status because it defaults to Pending, and user because the view assigns the logged-in user.
6. Views – maintenance/views.py
views.py
File type: Python
# ==========================================
# IMPORTS
# ==========================================
from django.shortcuts import render, redirect, get_object_or_404
from django.contrib.auth import login, logout, authenticate
from django.contrib.auth.forms import UserCreationForm
from django.contrib.auth.decorators import login_required
from django.contrib import messages
from django.views.decorators.http import require_POST
 
from .models import MaintenanceRequest
from .forms import MaintenanceRequestForm
 
 
# ==========================================
# HOME PAGE
# ==========================================
def home(request):
    return render(request, 'index.html')
 
 
# ==========================================
# USER REGISTRATION
# ==========================================
def register(request):
    form = UserCreationForm()
 
    if request.method == 'POST':
        form = UserCreationForm(request.POST)
 
        if form.is_valid():
            user = form.save()
            login(request, user)
            return redirect('home')
 
    return render(request, 'register.html', {'form': form})
 
 
# ==========================================
# USER LOGIN
# ==========================================
def user_login(request):
    if request.method == 'POST':
        username = request.POST['username']
        password = request.POST['password']
 
        user = authenticate(
            request,
            username=username,
            password=password
        )
 
        if user is not None:
            login(request, user)
            return redirect('home')
        else:
            messages.error(request, 'Invalid username or password')
 
    return render(request, 'login.html')
 
 
# ==========================================
# USER LOGOUT (POST ONLY)
# ==========================================
@login_required
@require_POST
def user_logout(request):
    logout(request)
    return redirect('home')
 
 
# ==========================================
# SUBMIT MAINTENANCE REQUEST
# ==========================================
@login_required
def submit_request(request):
    form = MaintenanceRequestForm()
 
    if request.method == 'POST':
        form = MaintenanceRequestForm(request.POST)
 
        if form.is_valid():
            obj = form.save(commit=False)
            obj.user = request.user
            obj.save()
            messages.success(request, 'Request submitted successfully!')
            return redirect('my_requests')
 
    return render(request, 'submit_request.html', {'form': form})
 
 
# ==========================================
# VIEW MY REQUESTS AND FILTER
# ==========================================
@login_required
def my_requests(request):
    data = MaintenanceRequest.objects.filter(user=request.user)
 
    category = request.GET.get('category')
    priority = request.GET.get('priority')
    status = request.GET.get('status')
 
    if category:
        data = data.filter(category=category)
 
    if priority:
        data = data.filter(priority=priority)
 
    if status:
        data = data.filter(status=status)
 
    return render(request, 'my_requests.html', {'data': data})
 
 
# ==========================================
# EDIT MY REQUEST
# ==========================================
@login_required
def edit_request(request, id):
    obj = get_object_or_404(
        MaintenanceRequest,
        id=id,
        user=request.user
    )
 
    form = MaintenanceRequestForm(instance=obj)
 
    if request.method == 'POST':
        form = MaintenanceRequestForm(request.POST, instance=obj)
 
        if form.is_valid():
            form.save()
            messages.success(request, 'Request updated successfully!')
            return redirect('my_requests')
 
    return render(request, 'submit_request.html', {'form': form})
 
 
# ==========================================
# DELETE MY REQUEST
# ==========================================
@login_required
def delete_request(request, id):
    obj = get_object_or_404(
        MaintenanceRequest,
        id=id,
        user=request.user
    )
 
    if request.method == 'POST':
        obj.delete()
        messages.success(request, 'Request deleted successfully!')
        return redirect('my_requests')
 
    return render(request, 'delete_request.html', {'obj': obj})
 
 
# ==========================================
# STAFF DASHBOARD - VIEW ALL REQUESTS
# ==========================================
@login_required
def staff_requests(request):
    if not request.user.is_staff:
        return redirect('home')
 
    data = MaintenanceRequest.objects.all()
    return render(request, 'staff_requests.html', {'data': data})
 
 
# ==========================================
# STAFF - UPDATE REQUEST STATUS
# ==========================================
@login_required
def update_status(request, id):
    if not request.user.is_staff:
        return redirect('home')
 
    obj = get_object_or_404(MaintenanceRequest, id=id)
 
    if request.method == 'POST':
        new_status = request.POST.get('status')
        allowed_statuses = [
            choice[0] for choice in MaintenanceRequest.STATUS
        ]
 
        if new_status in allowed_statuses:
            obj.status = new_status
            obj.save()
            messages.success(request, 'Status updated successfully!')
 
        return redirect('staff_requests')
 
    return render(request, 'update_status.html', {'obj': obj})
7. Application URLs – maintenance/urls.py
maintenance/urls.py
File type: Python
# URL patterns for the maintenance application
from django.urls import path
from . import views
 
urlpatterns = [
    # Home page
    path('', views.home, name='home'),
 
    # Authentication
    path('register/', views.register, name='register'),
    path('login/', views.user_login, name='login'),
    path('logout/', views.user_logout, name='logout'),
 
    # Maintenance requests
    path('submit/', views.submit_request, name='submit_request'),
    path('my-requests/', views.my_requests, name='my_requests'),
 
    # CRUD operations
    path('edit/<int:id>/', views.edit_request, name='edit_request'),
    path('delete/<int:id>/', views.delete_request, name='delete_request'),
 
    # Staff operations
    path('staff/', views.staff_requests, name='staff_requests'),
    path('update-status/<int:id>/', views.update_status, name='update_status'),
]
8. Main Project URLs – campus_project/urls.py
campus_project/urls.py
File type: Python
# Main project URL configuration
from django.contrib import admin
from django.urls import path, include
 
urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('maintenance.urls')),
]
9. HTML Templates
Create all the following HTML files inside maintenance/templates/.
base.html
File type: HTML template
<!-- Base template shared by all pages -->
<!DOCTYPE html>
<html>
<head>
    <title>Campus Maintenance System</title>
    {% load static %}
    <link rel="stylesheet" href="{% static 'style.css' %}">
</head>
<body>
 
    <!-- Navigation bar -->
    <nav>
        <h2>Campus Maintenance</h2>
        <a href="{% url 'home' %}">Home</a>
 
        {% if user.is_authenticated %}
            <a href="{% url 'submit_request' %}">Submit Request</a>
            <a href="{% url 'my_requests' %}">My Requests</a>
 
            {% if user.is_staff %}
                <a href="{% url 'staff_requests' %}">Staff Dashboard</a>
            {% endif %}
 
            <form method="POST" action="{% url 'logout' %}" class="inline-form">
                {% csrf_token %}
                <button type="submit">Logout</button>
            </form>
        {% else %}
            <a href="{% url 'login' %}">Login</a>
            <a href="{% url 'register' %}">Register</a>
        {% endif %}
    </nav>
 
    <!-- Display messages -->
    <div class="container">
        {% if messages %}
            {% for message in messages %}
                <p class="message">{{ message }}</p>
            {% endfor %}
        {% endif %}
 
        <!-- Child page content appears here -->
        {% block content %}
        {% endblock %}
    </div>
 
</body>
</html>
index.html
File type: HTML template
<!-- Home page -->
{% extends 'base.html' %}
 
{% block content %}
<h1>Welcome to Campus Maintenance System</h1>
<p>Report and track maintenance issues within your college campus.</p>
 
{% if user.is_authenticated %}
    <a class="button-link" href="{% url 'submit_request' %}">Submit a Request</a>
    <a class="button-link" href="{% url 'my_requests' %}">View My Requests</a>
{% else %}
    <a class="button-link" href="{% url 'login' %}">Login</a>
    <a class="button-link" href="{% url 'register' %}">Register</a>
{% endif %}
{% endblock %}
register.html
File type: HTML template
<!-- User registration page -->
{% extends 'base.html' %}
 
{% block content %}
<h2>User Registration</h2>
<form method="POST">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Register</button>
</form>
<p>Already have an account? <a href="{% url 'login' %}">Login</a></p>
{% endblock %}
login.html
File type: HTML template
<!-- User login page -->
{% extends 'base.html' %}
 
{% block content %}
<h2>User Login</h2>
<form method="POST">
    {% csrf_token %}
 
    <label>Username:</label>
    <input type="text" name="username" required>
 
    <label>Password:</label>
    <input type="password" name="password" required>
 
    <button type="submit">Login</button>
</form>
<p>New user? <a href="{% url 'register' %}">Create Account</a></p>
{% endblock %}
submit_request.html
File type: HTML template
<!-- Submit or edit maintenance request -->
{% extends 'base.html' %}
 
{% block content %}
<h2>Submit Maintenance Request</h2>
<form method="POST">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Save Request</button>
</form>
<a href="{% url 'my_requests' %}">Back to My Requests</a>
{% endblock %}
my_requests.html
File type: HTML template
<!-- Display and filter the logged-in user's requests -->
{% extends 'base.html' %}
 
{% block content %}
<h2>My Maintenance Requests</h2>
 
<!-- Filter form -->
<form method="GET">
    <label>Category:</label>
    <select name="category">
        <option value="">All</option>
        <option value="Electrical">Electrical</option>
        <option value="Plumbing">Plumbing</option>
        <option value="Furniture">Furniture</option>
        <option value="Cleaning">Cleaning</option>
        <option value="Other">Other</option>
    </select>
 
    <label>Priority:</label>
    <select name="priority">
        <option value="">All</option>
        <option value="Low">Low</option>
        <option value="Medium">Medium</option>
        <option value="High">High</option>
    </select>
 
    <label>Status:</label>
    <select name="status">
        <option value="">All</option>
        <option value="Pending">Pending</option>
        <option value="In Progress">In Progress</option>
        <option value="Completed">Completed</option>
    </select>
 
    <button type="submit">Filter</button>
</form>
 
<p><a href="{% url 'submit_request' %}">Submit New Request</a></p>
 
<!-- Requests table -->
<table>
    <tr>
        <th>Title</th>
        <th>Category</th>
        <th>Location</th>
        <th>Priority</th>
        <th>Status</th>
        <th>Date</th>
        <th>Actions</th>
    </tr>
 
    {% for i in data %}
    <tr>
        <td>{{ i.title }}</td>
        <td>{{ i.category }}</td>
        <td>{{ i.location }}</td>
        <td>{{ i.priority }}</td>
        <td>{{ i.status }}</td>
        <td>{{ i.date_reported }}</td>
        <td>
            <a href="{% url 'edit_request' i.id %}">Edit</a>
            <a href="{% url 'delete_request' i.id %}">Delete</a>
        </td>
    </tr>
    {% empty %}
    <tr>
        <td colspan="7">No requests found.</td>
    </tr>
    {% endfor %}
</table>
{% endblock %}
delete_request.html
File type: HTML template
<!-- Delete confirmation page -->
{% extends 'base.html' %}
 
{% block content %}
<h2>Delete Request</h2>
<p>Are you sure you want to delete "{{ obj.title }}"?</p>
 
<form method="POST">
    {% csrf_token %}
    <button type="submit">Yes, Delete</button>
</form>
 
<a href="{% url 'my_requests' %}">Cancel</a>
{% endblock %}
staff_requests.html
File type: HTML template
<!-- Staff dashboard showing all requests -->
{% extends 'base.html' %}
 
{% block content %}
<h2>All Maintenance Requests</h2>
 
<table>
    <tr>
        <th>Title</th>
        <th>Description</th>
        <th>Category</th>
        <th>Location</th>
        <th>User</th>
        <th>Priority</th>
        <th>Status</th>
        <th>Action</th>
    </tr>
 
    {% for i in data %}
    <tr>
        <td>{{ i.title }}</td>
        <td>{{ i.description }}</td>
        <td>{{ i.category }}</td>
        <td>{{ i.location }}</td>
        <td>{{ i.user.username }}</td>
        <td>{{ i.priority }}</td>
        <td>{{ i.status }}</td>
        <td>
            <a href="{% url 'update_status' i.id %}">Update Status</a>
        </td>
    </tr>
    {% empty %}
    <tr>
        <td colspan="8">No maintenance requests available.</td>
    </tr>
    {% endfor %}
</table>
{% endblock %}
update_status.html
File type: HTML template
<!-- Staff status update page -->
{% extends 'base.html' %}
 
{% block content %}
<h2>Update Request Status</h2>
<p>Request: {{ obj.title }}</p>
 
<form method="POST">
    {% csrf_token %}
 
    <label>Select Status:</label>
    <select name="status">
        <option value="Pending" {% if obj.status == 'Pending' %}selected{% endif %}>Pending</option>
        <option value="In Progress" {% if obj.status == 'In Progress' %}selected{% endif %}>In Progress</option>
        <option value="Completed" {% if obj.status == 'Completed' %}selected{% endif %}>Completed</option>
    </select>
 
    <button type="submit">Update</button>
</form>
 
<a href="{% url 'staff_requests' %}">Back</a>
{% endblock %}
10. CSS – maintenance/static/style.css
style.css
File type: CSS
/* General page design */
body {
    font-family: Arial, sans-serif;
    background-color: #f4f4f4;
    margin: 0;
    padding: 0;
}
 
/* Navigation bar */
nav {
    background-color: #23476b;
    color: white;
    padding: 15px;
}
 
nav h2 {
    display: inline-block;
    margin-right: 25px;
}
 
nav a {
    color: white;
    margin: 10px;
    text-decoration: none;
}
 
.inline-form {
    display: inline;
}
 
/* Main content container */
.container {
    background-color: white;
    width: 85%;
    margin: 30px auto;
    padding: 25px;
    border-radius: 8px;
}
 
/* Headings */
h1, h2 {
    color: #23476b;
}
 
/* Buttons and links */
button, .button-link {
    background-color: #23476b;
    color: white;
    padding: 9px 15px;
    border: none;
    text-decoration: none;
    cursor: pointer;
    display: inline-block;
}
 
/* Input fields */
input, select, textarea {
    padding: 8px;
    margin: 6px;
}
 
/* Table */
table {
    width: 100%;
    border-collapse: collapse;
}
 
th {
    background-color: #23476b;
    color: white;
}
 
td, th {
    padding: 10px;
    border: 1px solid #ccc;
    text-align: left;
}
 
/* Messages */
.message {
    background-color: #e5f4e8;
    padding: 10px;
}
11. Optional Admin Registration – maintenance/admin.py
admin.py
File type: Python
# Register the maintenance model in Django admin
from django.contrib import admin
from .models import MaintenanceRequest
 
admin.site.register(MaintenanceRequest)
12. Settings Configuration
Required settings in campus_project/settings.py
File type: Python settings
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'maintenance',
]
 
LOGIN_URL = '/login/'
Keep the default Django TEMPLATES configuration, including APP_DIRS=True and the request context processor. Keep the default STATIC_URL setting.
13. Database Migration and Run Commands
Run these commands from the folder containing manage.py
File type: Terminal commands
python manage.py makemigrations
python manage.py migrate
python manage.py createsuperuser
python manage.py runserver
Open http://127.0.0.1:8000/ in a browser. Use the superuser account to access staff features. The superuser is automatically a staff user.
14. Main Workflow
1.	The user registers using Django's built-in UserCreationForm.
2.	The user logs in using username and password.
3.	The authenticated user submits a maintenance request through MaintenanceRequestForm.
4.	The view assigns request.user to the request and saves it in the database.
5.	The user views, filters, edits or deletes only their own requests.
6.	A staff user opens the staff dashboard to view all requests.
7.	Staff updates a request status to Pending, In Progress or Completed.
15. Important Django Concepts
Concept	Purpose
Model	Defines the database table and its fields.
ModelForm	Creates a form using fields from a model.
View	Contains the logic to process requests.
URL	Connects a browser path to a view.
Template	Displays information using HTML.
Template inheritance	Allows pages to reuse a common base template.
makemigrations	Creates migration instructions from model changes.
migrate	Applies migrations to the database.
ForeignKey	Connects a maintenance request to its user.
login_required	Restricts a view to authenticated users.
16. Conclusion
Thus, a Campus Maintenance Request System was developed using Django. The application supports user registration, login and logout, maintenance request submission, viewing, filtering, editing and deletion. Staff members can view all requests and update their status. Django models, forms, views, URL routing, templates and CSS were used to implement the system.

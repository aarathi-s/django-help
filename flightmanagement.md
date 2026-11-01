Exercise:- 21                                                 Date:-07/08/2026

Experiment Name: Flight Management System

Question: Develop a Django web application named Flight Management System using Django and SQLite to manage flight details and passenger bookings.
1.	Application Design (10 Marks)
Design a simple, attractive, and responsive user interface using HTML and CSS. Use Django template inheritance (base.html) to maintain a consistent layout across all pages.
2.	Database Design (10 Marks)
Design and implement the database models for Flight and Passenger with appropriate fields and relationships using Django ORM.
3.	User Registration and Login (10 Marks)
Implement user registration, login, and logout using Django's authentication system. Allow only authenticated users to access the application.
4.	Flight Management (CRUD) (10 Marks)
Implement CRUD operations to add, view, update, and delete flight details.
5.	Passenger Management (CRUD) (10 Marks)
Implement CRUD operations to add, view, update, and delete passenger details.
6.	Flight Booking Module (10 Marks)
Develop a booking module that allows passengers to book available flights. Prevent duplicate bookings for the same passenger and flight.
7.	Search and Dashboard (10 Marks)
Implement a search feature to find flights by source, destination, or departure date. Display a dashboard showing the total number of flights, passengers, and bookings.


Aarathi Sankar
Roll No.: 1

Commands for Starting a New Project
django-admin startproject FlightManagementSystem
cd FlightManagementSystem

Commands for Starting a New App
python manage.py startapp flights

Commands for Creating Model and Migration
python manage.py makemigrations
python manage.py migrate


Commands for Creating and Changing the Credentials of Admin
python manage.py createsuperuser
python manage.py changepassword admin


Commands to Run the Application
python manage.py runserver


Project Folder Structure

 


Project – urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('flights.urls')),
]
Project – settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'flights',
]

LOGIN_REDIRECT_URL = "dashboard"
LOGOUT_REDIRECT_URL = "login"
LOGIN_URL = "login"
App- <App-flights> urls.py
from django.urls import path
from django.contrib.auth import views as auth_views
from . import views

urlpatterns = [
    path("", views.home, name="home"),
    path("register/", views.register, name="register"),
    path("login/", auth_views.LoginView.as_view(template_name="login.html"), name="login"),
    path("logout/", auth_views.LogoutView.as_view(), name="logout"),
    path("dashboard/", views.dashboard, name="dashboard"),

    path("flights/", views.flight_list, name="flight_list"),
    path("add-flight/", views.add_flight, name="add_flight"),
    path("edit-flight/<int:id>/", views.edit_flight, name="edit_flight"),
    path("delete-flight/<int:id>/", views.delete_flight, name="delete_flight"),

    path("passengers/", views.passenger_list, name="passenger_list"),
    path("add-passenger/", views.add_passenger, name="add_passenger"),
    path("edit-passenger/<int:id>/", views.edit_passenger, name="edit_passenger"),
    path("delete-passenger/<int:id>/", views.delete_passenger, name="delete_passenger"),

    path("bookings/", views.booking_list, name="booking_list"),
    path("add-booking/", views.add_booking, name="add_booking"),

    path("search/", views.search, name="search"),
]
App – views.py
from django.shortcuts import render, redirect
from django.contrib.auth import login
from django.contrib.auth.decorators import login_required
from .forms import RegisterForm, FlightForm, PassengerForm, BookingForm
from .models import Flight, Passenger, Booking

def home(request):
    return render(request, "index.html")

def register(request):
    if request.method == "POST":
        form = RegisterForm(request.POST)
        if form.is_valid():
            user = form.save()
            login(request, user)
            return redirect("dashboard")
        else:
            print(form.errors)   # <-- Add this line
    else:
        form = RegisterForm()
    return render(request, "register.html", {"form": form})

@login_required
def dashboard(request):
    total_flights = Flight.objects.count()
    total_passengers = Passenger.objects.count()
    total_bookings = Booking.objects.count()
    context = {
        "total_flights": total_flights,
        "total_passengers": total_passengers,
        "total_bookings": total_bookings,
    }
    return render(request, "dashboard.html", context)

@login_required
def flight_list(request):
    flights = Flight.objects.all()
    return render(request, "flight_list.html", {"flights": flights})

@login_required
def add_flight(request):
    if request.method == "POST":
        form = FlightForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect("flight_list")
    else:
        form = FlightForm()
    return render(request, "flight_form.html", {"form": form})

@login_required
def search(request):
    flights = Flight.objects.all()
    if request.method == "GET":
        source = request.GET.get("source")
        destination = request.GET.get("destination")
        departure_date = request.GET.get("departure_date")
        if source:
            flights = flights.filter(source__icontains=source)
        if destination:
            flights = flights.filter(destination__icontains=destination)
        if departure_date:
            flights = flights.filter(departure_date=departure_date)
    return render(request, "search.html", {"flights": flights})

@login_required
def edit_flight(request, id):
    flight = Flight.objects.get(id=id)
    if request.method == "POST":
        form = FlightForm(request.POST, instance=flight)
        if form.is_valid():
            form.save()
            return redirect("flight_list")
    else:
        form = FlightForm(instance=flight)
    return render(request, "flight_form.html", {"form": form})

@login_required
def delete_flight(request, id):
    flight = Flight.objects.get(id=id)
    flight.delete()
    return redirect("flight_list")

@login_required
def passenger_list(request):
    passengers = Passenger.objects.all()
    return render(request, "passenger_list.html", {"passengers": passengers})

@login_required
def add_passenger(request):
    if request.method == "POST":
        form = PassengerForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect("passenger_list")
    else:
        form = PassengerForm()
    return render(request, "passenger_form.html", {"form": form})

@login_required
def edit_passenger(request, id):
    passenger = Passenger.objects.get(id=id)
    if request.method == "POST":
        form = PassengerForm(request.POST, instance=passenger)
        if form.is_valid():
            form.save()
            return redirect("passenger_list")
    else:
        form = PassengerForm(instance=passenger)
    return render(request, "passenger_form.html", {"form": form})

@login_required
def delete_passenger(request, id):
    passenger = Passenger.objects.get(id=id)
    passenger.delete()
    return redirect("passenger_list")

@login_required
def booking_list(request):
    bookings = Booking.objects.all()
    return render(request, "booking_list.html", {"bookings": bookings})

@login_required
def add_booking(request):
    if request.method == "POST":
        form = BookingForm(request.POST)
        if form.is_valid():
            flight = form.cleaned_data["flight"]
            passenger = form.cleaned_data["passenger"]
            if Booking.objects.filter(flight=flight, passenger=passenger).exists():
                return render(request, "booking_form.html", {
                    "form": form,
                    "error": "This passenger has already booked this flight."
                })
            form.save()
            return redirect("booking_list")
    else:
        form = BookingForm()
    return render(request, "booking_form.html", {"form": form})

App-models.py
from django.db import models

class Flight(models.Model):
    flight_no = models.CharField(max_length=20)
    airline = models.CharField(max_length=50)
    source = models.CharField(max_length=50)
    destination = models.CharField(max_length=50)
    departure_date = models.DateField()

    def __str__(self):
        return self.flight_no

class Passenger(models.Model):
    name = models.CharField(max_length=100)
    email = models.EmailField()
    phone = models.CharField(max_length=15)

    def __str__(self):
        return self.name

class Booking(models.Model):
    flight = models.ForeignKey(Flight, on_delete=models.CASCADE)
    passenger = models.ForeignKey(Passenger, on_delete=models.CASCADE)

    class Meta:
        unique_together = ('flight', 'passenger')

    def __str__(self):
        return self.passenger.name

App-forms.py
from django import forms
from django.contrib.auth.forms import UserCreationForm
from .models import Flight, Passenger, Booking

class RegisterForm(UserCreationForm):
    pass

class FlightForm(forms.ModelForm):
    class Meta:
        model = Flight
        fields = "__all__"

class PassengerForm(forms.ModelForm):
    class Meta:
        model = Passenger
        fields = "__all__"

class BookingForm(forms.ModelForm):
    class Meta:
        model = Booking
        fields = "__all__"

App-admin.py
from django.contrib import admin
from .models import Flight, Passenger, Booking

admin.site.register(Flight)
admin.site.register(Passenger)
admin.site.register(Booking)

.html file
base.html
{% load static %}
<!DOCTYPE html>
<html>
<head>
    <title>Flight Management System</title>
    <link rel="stylesheet" href="{% static 'style.css' %}">
</head>
<body>
<header><h1>Flight Management System</h1></header>
<nav>
    <a href="{% url 'home' %}">Home</a>
    {% if user.is_authenticated %}
        <a href="{% url 'dashboard' %}">Dashboard</a>
        <a href="{% url 'flight_list' %}">Flights</a>
        <a href="{% url 'passenger_list' %}">Passengers</a>
        <a href="{% url 'booking_list' %}">Bookings</a>
        <a href="{% url 'search' %}">Search</a>
        <form action="{% url 'logout' %}" method="post" style="display:inline;">
            {% csrf_token %}
            <button type="submit">Logout</button>
        </form>
    {% else %}
        <a href="{% url 'login' %}">Login</a>
        <a href="{% url 'register' %}">Register</a>
    {% endif %}
</nav>
<div class="container">
    {% block content %}
    {% endblock %}
</div>
</body>
</html>

Index.html
{% load static %}
<!DOCTYPE html>
<html>
<head>
    <title>Flight Management System</title>
    <link rel="stylesheet" href="{% static 'style.css' %}">
</head>
<body>
<div class="container">
    <h1>Flight Management System</h1>
    <h2>Welcome</h2>
    <p>Online Flight Booking and Management System</p>
    <a href="{% url 'login' %}"><button>Login</button></a>
    <a href="{% url 'register' %}"><button>Register</button></a>
</div>
</body>
</html>

Login.html
{% extends 'base.html' %}
{% block content %}
<h2>Login</h2>
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Login</button>
</form>
{% endblock %}

Register.html
{% extends 'base.html' %}
{% block content %}
{% load static %}
<h2>Register</h2>
<form method="POST">
    {% csrf_token %}
    {{ form.as_p }}
    {% if form.errors %}
        <div style="color:red;">
            {{ form.errors }}
        </div>
    {% endif %}
    <button type="submit">Register</button>
</form>
{% endblock %}
Dashboard.html
{% extends 'base.html' %}
{% block content %}
<h2>Dashboard</h2>
<p><b>Total Flights:</b> {{ total_flights }}</p>
<p><b>Total Passengers:</b> {{ total_passengers }}</p>
<p><b>Total Bookings:</b> {{ total_bookings }}</p>
<hr>
<a href="{% url 'flight_list' %}">Manage Flights</a><br><br>
<a href="{% url 'passenger_list' %}">Manage Passengers</a><br><br>
<a href="{% url 'booking_list' %}">Manage Bookings</a><br><br>
<a href="{% url 'search' %}">Search Flights</a>
{% endblock %}

Booking_form.html
{% extends 'base.html' %}
{% block content %}
<h2>Book Flight</h2>
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    {% if error %}
        <p style="color: red;">{{ error }}</p>
    {% endif %}
    <button type="submit">Book</button>
</form>
{% endblock %}

Booking_list.html
{% extends 'base.html' %}
{% block content %}
<h2>Bookings</h2>
<a href="{% url 'add_booking' %}">New Booking</a><br><br>
<table border="1">
    <tr>
        <th>Passenger</th>
        <th>Flight</th>
    </tr>
    {% for booking in bookings %}
    <tr>
        <td>{{ booking.passenger }}</td>
        <td>{{ booking.flight }}</td>
    </tr>
    {% endfor %}
</table>
{% endblock %}

Flight_form.html
{% extends 'base.html' %}
{% block content %}
<h2>Flight Form</h2>
<form method="POST">
    {% csrf_token %}
    <label>Flight No:</label><br>
    {{ form.flight_no }}<br><br>
    <label>Airline:</label><br>
    {{ form.airline }}<br><br>
    <label>Source:</label><br>
    {{ form.source }}<br><br>
    <label>Destination:</label><br>
    {{ form.destination }}<br><br>
    <label>Departure Date:</label><br>
    {{ form.departure_date }}<br><br>
    <button type="submit">Save</button>
</form>
{% endblock %}

Flight_list.html
{% extends 'base.html' %}
{% block content %}
<h2>Flights</h2>
<a href="{% url 'add_flight' %}">Add Flight</a><br><br>
<table border="1" cellpadding="8">
    <tr>
        <th>Flight No</th>
        <th>Airline</th>
        <th>Source</th>
        <th>Destination</th>
        <th>Date</th>
        <th>Action</th>
    </tr>
    {% for flight in flights %}
    <tr>
        <td>{{ flight.flight_no }}</td>
        <td>{{ flight.airline }}</td>
        <td>{{ flight.source }}</td>
        <td>{{ flight.destination }}</td>
        <td>{{ flight.departure_date }}</td>
        <td>
            <a href="{% url 'edit_flight' flight.id %}">Edit</a>
            |
            <a href="{% url 'delete_flight' flight.id %}">Delete</a>
        </td>
    </tr>
    {% endfor %}
</table>
{% endblock %}

Passenger_form.html
{% extends 'base.html' %}
{% block content %}
<h2>Passenger Form</h2>
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Save</button>
</form>
{% endblock %}
Passenger_list.html
{% extends 'base.html' %}
{% block content %}
<h2>Passengers</h2>
<a href="{% url 'add_passenger' %}">Add Passenger</a><br><br>
<table border="1" cellpadding="8">
    <tr>
        <th>Name</th>
        <th>Email</th>
        <th>Phone</th>
        <th>Action</th>
    </tr>
    {% for passenger in passengers %}
    <tr>
        <td>{{ passenger.name }}</td>
        <td>{{ passenger.email }}</td>
        <td>{{ passenger.phone }}</td>
        <td>
            <a href="{% url 'edit_passenger' passenger.id %}">Edit</a> |
            <a href="{% url 'delete_passenger' passenger.id %}">Delete</a>
        </td>
    </tr>
    {% endfor %}
</table>
{% endblock %}

Search.html
{% extends 'base.html' %}
{% block content %}
<h2>Search Flights</h2>
<form method="get">
    Source:
    <input type="text" name="source">
    Destination:
    <input type="text" name="destination">
    Date:
    <input type="date" name="departure_date">
    <button type="submit">Search</button>
</form>
<br>
<table border="1">
    <tr>
        <th>Flight No</th>
        <th>Airline</th>
        <th>Source</th>
        <th>Destination</th>
        <th>Date</th>
    </tr>
    {% for flight in flights %}
    <tr>
        <td>{{ flight.flight_no }}</td>
        <td>{{ flight.airline }}</td>
        <td>{{ flight.source }}</td>
        <td>{{ flight.destination }}</td>
        <td>{{ flight.departure_date }}</td>
    </tr>
    {% empty %}
    <tr>
        <td colspan="5">No Flights Found</td>
    </tr>
    {% endfor %}
</table>
{% endblock %}

 
 
 
 
 


# -Discussion-Forum
pip install django

Steps to Build the Project – Discussion forum
1. Creating the project and app:
Now we will create a new project, DataFlair_discsnForum and an app inside it named ’Discussion_forum’

django-admin startproject DataFlair_discsnForum
cd DataFlair_discusnForum
django-admin startapp Discussion_Forum
2. Creating the Database (models.py):
Let’s create the databases in models.py which is inside the app folder. The class will be a subclass of model class defined in models module, and we can define as many fields as we want in here and by default all fields are mandatory, to change this default behavior of Django, add to the specific field ‘null=True’ as seen in the below code.

Code:

from django.db import models 
    
#parent model
class forum(models.Model):
    name=models.CharField(max_length=200,default="anonymous" )
    email=models.CharField(max_length=200,null=True)
    topic= models.CharField(max_length=300)
    description = models.CharField(max_length=1000,blank=True)
    link = models.CharField(max_length=100 ,null =True)
    date_created=models.DateTimeField(auto_now_add=True,null=True)
    
    def __str__(self):
        return str(self.topic)
 
#child model
class Discussion(models.Model):
    forum = models.ForeignKey(forum,blank=True,on_delete=models.CASCADE)
    discuss = models.CharField(max_length=1000)
 
    def __str__(self):
        return str(self.forum)

Our models class forum will store:

Customer name
Customer email
The topic of the forum
Reference links
Description
Creation date
Discussion class is a child class of forum that stores views from different users. It has just two fields-

The forum which is a foreign key (Provide a many-to-one relation by adding a column to the local model to hold the remote value). It helps in maintaining a record of which opinion belongs to which forum.
Discuss – It actually stores the opinion
We are using simple model fields for that purpose. The __str__() method will return the string representation of the object. These are simple Django concepts.

We are using Django inbuilt database SQLite so we need not add anything to our settings.py file. If you are using any other database don’t forget to change your database settings in settings.py file.

3. Creating and updating models: forms.py
This is required for creating, updating various models that we have created in models.py (for CRUD functionality). We will be using inbuilt Django forms so we don’t have to code much.

Code:

from django.forms import ModelForm
from .models import *
 
class CreateInForum(ModelForm):
    class Meta:
        model= forum
        fields = "__all__"
 
class CreateInDiscussion(ModelForm):
    class Meta:
        model= Discussion
        fields = "__all__"

It just contains two forms one for creating a new forum and one for adding views to the existing forum, which is clear from the models which we are using in Meta class (a metaclass is used to define an extra option for a model or form so that other classes within the web app know the capabilities of the model)

4. To directly create/update models from django admin site: admin.py
Code:

from django.contrib import admin
from .models import *
 
# Register your models here.
admin.site.register(forum)
admin.site.register(Discussion)
It will register all our models to the admin site so that when we search http://127.0.0.1:8000/admin/ we can easily create/update. But make sure you create a superuser and apply all migrations by :

py manage.py makemigrations
py manage.py migrate
py manage.py createsuperuser
It will ask your username, email, password.

5. Configuring urls.py :
Code:

from django.contrib import admin
from django.urls import path
from Discussion_forum.views import *
 
urlpatterns = [
    path('admin/', admin.site.urls),
    path('',home,name='home'),
    path('addInForum/',addInForum,name='addInForum'),
    path('addInDiscussion/',addInDiscussion,name='addInDiscussion'),
]
We just need to add the last four lines here and the import statements.

6. Let’s finally set our ‘views.py’ :
Code:

from django.shortcuts import render,redirect
from .models import * 
from .forms import * 
# Create your views here.
 
def home(request):
    forums=forum.objects.all()
    count=forums.count()
    discussions=[]
    for i in forums:
        discussions.append(i.discussion_set.all())
 
    context={'forums':forums,
              'count':count,
              'discussions':discussions}
    return render(request,'home.html',context)
 
def addInForum(request):
    form = CreateInForum()
    if request.method == 'POST':
        form = CreateInForum(request.POST)
        if form.is_valid():
            form.save()
            return redirect('/')
    context ={'form':form}
    return render(request,'addInForum.html',context)
 
def addInDiscussion(request):
    form = CreateInDiscussion()
    if request.method == 'POST':
        form = CreateInDiscussion(request.POST)
        if form.is_valid():
            form.save()
            return redirect('/')
    context ={'form':form}
    return render(request,'addInDiscussion.html',context)

Let’s discuss each function one by one:

1. home: This is the home page that takes all forums and discussion objects and passes them to the templates through a dictionary named context. This page links to both the other pages and shows all the required information to the user with the feature of adding more information in any forum.

2. addInForum: This function is used to create a new forum through an instance of CreateInForum() object defined in forms.py and also, it takes the filled data through request.POST and checks if the data is valid to save it in our database and after successfully storing it redirects to the home page otherwise it again asks user to fill the correct information.

3. addInDiscussion: This is very similar to the previous function, the only change is it is used to add opinions to existing forums.

7. Now let’s create the templates:
There are three templates home, addInDiscussion, addInForums and here are the files

Home.html

{% load static %}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>DataFlair discussion forum</title>
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.0/css/bootstrap.min.css" integrity="sha384-9aIt2nRpC12Uk9gS9baDl411NQApFmC26EwAOH8WgZl5MYYxFfc+NcPb1dKGj7Sk" crossorigin="anonymous"0>
    <style>
        .box{
            border: 4px solid black;
            margin: 0 auto;
        }
    </style>
</head>
<body>
        <h2 class="jumbotron">
            Currently active forums: {{count}}
            <form method="POST" action="{% url 'addInForum' %}">
                {% csrf_token %}
            <button class="btn btn-success" style="width:fit-content; padding: 4px; margin:10px;">Add more</button>
            </form>
        </h2>
            <div class="card-columns" style="padding: 10px; margin: 20px;"></div>
            
            
            {%for forum in forums %}
            
                <div class="card box container">
                    <br>
                    <h5 class="card-title">
                        <a href='{{forum.link}}'><h3>{{forum.topic}}</h3></a> 
                        <div class="card-body container">
                                <p>{{forum.description}}</p>
                            </h5>
                            <hr>
                            <p> By: {{forum.name}}</p>
                            email- {{forum.email}}
                            <hr>     
                            <h4>Views from other users</h4>
                            {%for discuss in discussions%}
                            {%for objs in discuss%}  
                            {% if objs.forum == forum %}
                               {{objs.discuss}}
                               <br>
                            {% endif %}
                            {%endfor%}
                            {%endfor%}
                            <form method="POST" action="{% url 'addInDiscussion' %}">
                                {% csrf_token %}
                            <button class="btn btn-success" style="width:fit-content; padding: 4px; margin:10px;">Add more</button>
                            </form>
                        </div>
                </div>
            </div>
            <br>
            
            {%endfor%}
          
    <script src="https://code.jquery.com/jquery-3.5.1.slim.min.js" integrity="sha384-DfXdz2htPH0lsSSs5nCTpuj/zy4C+OGpamoFVy38MVBnE+IbbVYUew+OrCXaRkfj" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/popper.js@1.16.0/dist/umd/popper.min.js" integrity="sha384-Q6E9RHvbIyZFJoft+2mJbHaEWldlvI9IOYy5n3zV9zzTtmI3UksdQRVvoxMfooAo" crossorigin="anonymous"></script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.5.0/js/bootstrap.min.js" integrity="sha384-OgVRvuATP1z7JjHLkuOU7Xw704+h835Lr+6QL9UvYjZE3Ipu6Tp75j7Bh/kR0JKI" crossorigin="anonymous"></script>
</body>
</html>
addInForum.html

<head>
    <style>
        form{
            border:4px solid black;
            margin: 0 auto;
            padding: 40px;
            width: fit-content;
        }
    </style>
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.0/css/bootstrap.min.css" integrity="sha384-9aIt2nRpC12Uk9gS9baDl411NQApFmC26EwAOH8WgZl5MYYxFfc+NcPb1dKGj7Sk" crossorigin="anonymous"0>
    
</head>
<form action="{% url 'addInForum' %}" method="POST">
    {% csrf_token %}
    {{form.as_p}}
    <input type="submit" class="btn btn-success" value="submit">
</form>


addInDiscussion.html

<head>
    <style>
        form{
            border:4px solid black;
            margin: 0 auto;
            padding: 40px;
            width: fit-content;
        }
    </style>
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.0/css/bootstrap.min.css" integrity="sha384-9aIt2nRpC12Uk9gS9baDl411NQApFmC26EwAOH8WgZl5MYYxFfc+NcPb1dKGj7Sk" crossorigin="anonymous"0>
    
</head>
<form action="{% url 'addInDiscussion' %}" method="POST">
    {% csrf_token %}
    {{form.as_p}}
    <input type="submit" class="btn btn-success" value="submit">
</form>
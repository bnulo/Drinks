## Django REST Framework

The API typically allow to view, create new , update and delete data

> python3 -m venv .venv
create virtual environment in a folder called .venv

> . .venv/bin/activate
activate venv

> pip install django
installs django

> pip install djangorestframework

> django-admin
lists commands of django

> django-admin startproject drinks .
creates new project called drinks in current directory

manage.py is a wrapper around the django admin that does a little extra setup for us and makes things easier

> python manage.py runserver
starts server

Migrations represent the data structure for the database and those need to be created. migration describes the change to our data structure. we need to apply that to the database.

> python manage.py migrate
creates tables. applies the migrations.

We need to create a model that represents what a drink look like
Model is a python class and we give it various attributes  

Admin page allows to see tables and see the data inside of them
we can go to this admin in server_address/admin

> python manage.py createsuperuser
gets username password and email and creates user. we use it to login to admin website

in settings.py we add our app 'drinks' to INSTALLED_APPS var which is a list of all apps we are using. we also add django framework to our installed apps.

> INSTALLED_APPS = [
'rest_framework',
'drinks',
'django.contrib.admin',
...
]

in models.py file we create class called Drink which inherits from models.Model and we give it attributes such as name

```python
class Drink(models.Model):
  name = models.CharField(max_length=200)
  description = models.CharField(max_length=500)

  # by this function we can show detail about every record of the table in admin panel automatically
  def __str__(self):
    return self.name + ' ' + self.description
```

Now to create a database table from this information we just need another migration

> python manage.py makemigrations NAME_OF_THE_APP
> python manage.py makemigrations drinks

migrations are available in migrations folder

> python manage.py migrate
we apply any unapplied migrations

in drinks directory in admin.py file we can register different tables we want to show in the admin panel

```python
admin.site.register(Drink)
```

we create a file called serializers.py in drinks folder (in the app folder)
Here is the process of going from python object to json. we use the serializer when we're trying to return our model through API.

```python
class DrinkSerializer(serializers.ModelSerializer):

  # This is meta data describing the model
  class Meta:
    model = Drink    
    fields = ['id', 'name', 'description']  # id is automatically added to model
```
Now we need to create an endpoint in views.py file in drinks folder (in the app folder)  
An endpoint is a certain url that you can access data from

```python
@api_view(['GET', 'POST'])
def drink_list(request):
  # we get all the drinks
  # serialize them
  # return json
  if request.method == 'GET':
    drinks = Drink.objects.all()
    serializer = DrinkSerializer(drinks, many=True)
    return JsonResponse(serializer.data, safe=False) # returns in list format
    return JsonResponse({"drinks": serializer.data }) # returns in dictionary format
    return Response(serializer.data) # returns response, we can view in html view in the admin website
  if request.method == 'POST':
    # take the data sent to us
    # deserialize it
    # create a drink object out of it
    serializer = DrinkSerializer(data=request.data)
    if serializer.is_valid():
      # we save it
      serializer.save()
      return Response(serializer.data, status=status.HTTP_201_CREATED)
```

Now we need to say what url is going to hit this view. and that's all done inside of urls.py by adding new path to urlpatterns variable

```python
urlpatterns = [
  path('admin', admin.site.urls),
  path('drinks/', views.drink_list) # we pass (path, function we want to hit)
]
```

decorator is something you put above your function to describe its functionality

we add new path for a single drink to the urlpatterns

```python
# we have a parameter <int:id> here of type int
path('drinks/<int:id>', views.drink_detail)
```

we add a new function to the views.py

```python
# PUT for update
@api_view(['GET', 'PUT', 'DELETE'])
def drink_detail(request, id, format=None):
  # this checks that this is a valid request
    try:
        drink = Drink.objects.get(pk=id) # we pass id as primary key
    except:
        return Response(status=status.HTTP_404_NOT_FOUND)
    if request.method == 'GET':
        serializer = DrinkSerializer(drink)
        return Response(serializer.data)
    elif request.method == 'PUT':
        serializer = DrinkSerializer(drink, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
    elif request.method == 'DELETE':
        drink.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```

If we want to get normal json data through the browser like .json we need to build that capability in our urls so not only we want these urls but we want another ones with different extensions so we can use a function called format_suffix_patterns(urlpatterns) and reassign return value to urlpatterns.

```python
# under urlpatterns
# urlpatterns = [
# path(...),
# path(...),
# ...
# ]
# we add this variable
urlpatterns = format_suffix_patterns(urlpatterns)
```

now we need to add a new parameter to our views called format like:

```python
def drink_list(request, format=None):
  pass
def drink_detail(request, id, format=None):
  pass
```

#### Consuming an API

add new file in main directory Drinks out of drinks called consume.py  
we install requests with pip wich allows us to make requests to pages on the internet

```python
import requests
response = requests.get('http://127.0.0.1:8000/drinks')
print(response.json())
```

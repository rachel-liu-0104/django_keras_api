# Using Django to Build A Simple Keras Model Api 

## Install Django and Start Django Project
### 1. Create virtual env and install Django
```
conda create --name django_api python=3.7
conda activate django_api
pip install Django==2.0.2
```

### 2. Create Django Project
```
django-admin startproject django_k_api # create project
python manage.py startapp keras_model # create app
```
This will automatically start a project folder like this(README.md was generated by git):


## Develop the application
### step1. add app to django_api/settings.py
```
INSTALLED_APPS = [
    'keras_model',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]

```
This is to register our app

### step2. update django_api/urls.py
```
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('keras_model.urls')),
]
```
add our keras_model urls to project

### step3. write keras_model/form.py (usd default form to better process image)
```
from django import forms

class UploadFileForm(forms.Form):
    title = forms.ImageField()
```
add form to make upload easier

### step4. write keras model prediction function
```
from keras.applications.resnet50 import ResNet50
from keras.preprocessing import image
from keras.applications.resnet50 import preprocess_input,decode_predictions
import numpy as np
import tensorflow as tf

g1 = tf.Graph()
sess1 = tf.Session(graph=g1)

with sess1.as_default():
    with g1.as_default():
        tf.global_variables_initializer().run()
        model = ResNet50(weights='imagenet')

def predict(img_path):
    img = image.load_img(img_path, target_size=(224,224))
    x = image.img_to_array(img)
    x = np.expand_dims(x, axis=0)
    x = preprocess_input(x)

    with sess1.as_default():
        with sess1.graph.as_default():
            preds = model.predict(x)
            predictions = decode_predictions(preds, top=1)[0]
    return(predictions)
```

### step5. write keras_model/views.py
```
from django.shortcuts import render
from .forms import UploadFileForm

from keras_model.predict import predict

def index(request):
    return render(request, "keras_model/index.html")

def save_uploaded_image(f,name):
    with open("keras_model/static/media/"+ "temp.png", "wb+") as destination:
        for chunk in f.chunks():
            destination.write(chunk)

# Create your views here.
def predict_Image(request):
    if request.method == "POST":
        form = UploadFileForm(request.POST, request.FILES)
        if form.is_valid():
            image=request.FILES['image']
            save_uploaded_image(image, image._name)

            result = predict("keras_model/static/media/"+ "temp.png")[0][1]


            resp = {'data':"http://127.0.0.1:8000/static/media/"+ image._name , 'result': result}
            return render(request, "keras_model/result.html", resp)

    else:
        return render(request, "keras_model/index.html")
```

### step6. write keras_model/urls.py
```
from django.urls import path
from . import views


app_name = 'keras_model'
urlpatterns = [
    path('', views.index, name='index'),
    path('image', views.predict_Image, name='image'),
]
```

### step7.write two html
under keras_model/templates/keras_model
i. index.html
```
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>Django_keras_API</title>
</head>

<body>
<h1>Using Resnet50 to Predict images </h1>
<div>Please upload an image</div>
<div class="form">
    <form action= "{% url 'keras_model:image' %}" method="POST" enctype="multipart/form-data">
        {% csrf_token %}
        <input name="image" type="file" placeholder = "Please upload images"><br>
        <button type="submit" style = '...' value="Upload">upload</button>
    </form>
</div>

</body>
</html>
```

ii. result.html
```
<!DOCTYPE html>
{% load static %}
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Keras Model Prediction results</title>
</head>
<body>
<h1>Keras Model Prediction results</h1>
<h2>The prediction results is: {{ result }}</h2>
<img src = {% static "media/temp.png" %}  width="500px">
</body>
</html>
```

### Final Step
```
python manage.py makemigrations
python manage.py migrate
python manage.py runserver
```
the default server will host on 127.0.0.1:8000


## Github link
```
https://github.com/rachel-liu-0104/django_keras_api
```

# Django static v/s media files

This article looks at the different kinds of files usually included in a Django project and how to manage them.

## Objectives

By the end of this article, you should be able to:

1. Know the different kinds of files used in a Django project
1. Using static files (or static assets) in a development environment
1. Using static files (or static assets) in a production environment
1. Uploading files in Django (media files)

Django is a full-stack framework. It comes with many batteries that you can use to build a fully functional web application. Django also lets you work with files with ease. These files could be of three types.

## Three types of files in a Django application

- Source code: The heart of any Django application. The definition for models, views, and templates is found here.
- Static files: Any file part of your application, like an image or a CSS file.
- Media file: Any file that the user uploads is called a media file.

We will be focussing on static and media files in this article. Even though the names are different, both represent regular files. Now you may wonder, why separate them into different categories. It's done for the following reasons.

- You cannot trust the file uploaded by the user. So they have to be treated differently.
- Sometimes, you need to perform processing on files so that they can be better served(For example, you could optimize the user-uploaded images to support different devices)
- You don't want a user uploaded file to replace a static file accidentally.

## Serving Static Files in a development environment


Let's see how we can serve static files in the development mode. The following example serves a `CSS` file and an `image` file.

Django's [staticfiles](https://docs.djangoproject.com/en/3.2/ref/contrib/staticfiles/) app helps us serve static files in development as well as in the production environment.

> django.contrib.staticfiles collects static files from each of your applications (and any other places you specify) into a single location that can easily be served in production. - Django docs

Here are some essential staticfiles parameters that you should take care of:

- [STATIC_URL](https://docs.djangoproject.com/en/3.2/ref/settings/#static-url): URL where the user can access your static files. Default is `/static/`.

- [STATIC_ROOT](https://docs.djangoproject.com/en/3.2/ref/settings/#static-root): Location where [collectstatic](https://docs.djangoproject.com/en/3.2/ref/contrib/staticfiles/#django-admin-collectstatic) will collect all the static files. Collectstatic is a management command that collects static files into the `STATIC_ROOT`.

- [STATICFILES_DIRS](https://docs.djangoproject.com/en/3.2/ref/settings/#staticfiles-dirs): Additional locations to look for static files.

- [STATICFILES_STORAGE](https://docs.djangoproject.com/en/3.2/ref/settings/#staticfiles-storage): Storage for your static files. By default, the files are stored in the file system. You can change the storage to [cloud storage or a CDN](https://docs.djangoproject.com/en/3.2/howto/static-files/deployment/#serving-static-files-from-a-cloud-service-or-cdn).

Now that we have seen the essential parameters to handle static files, let's see them in action.

Check the `settings.py` file to see the settings: https://github.com/testdrivenio/django-static-media-files/blob/21ac2562d1cfcd459629e7c484bcdd2e09defa13/config/settings.py#L123-L126.

Now that we have configured the settings for handling static files, we define the static files in a Django template. 

```html
<!-- templates/_base.html -->

{% load static %}

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <link rel="stylesheet" href="{% static 'base.css' %}">
</head>
<body>
    {% block content %}
    {% endblock content %}
</body>
</html>
```

https://github.com/testdrivenio/django-static-media-files/blob/draft/templates/_base.html

Notice that we have added a line: `{% load static %}` and used a similar tag to point to our `base.css` file. The former allows us to use static templatetags, and the latter will automatically point to the file under the `static` directory. Moreover, the tag will generate a complete URL like `/static/base.css`. This tag comes in handy when deciding to change the `STATIC_URL` in a vast project. In such situations, the URL change is handled by Django automatically.

The same approach can be used wherever we need to access the static file: https://github.com/testdrivenio/django-static-media-files/blob/draft/templates/index.html.

Here's how the static files are seen in action,

![static files on dev](assets/sample.png)

## Serving Static Files in a production environment

Serving static files in a development environment was pretty straightforward. We use the `static` templatetag to load the file into the template. However, we need to do more configuration for production because production would require a production-grade server like [gunicorn](https://gunicorn.org/) that does not serve static files.

We can use [whitenoise](http://whitenoise.evans.io/en/stable/) to serve static files from our production application. However, it is recommended to use whitenoise with the newer, more efficient [brotli](https://en.wikipedia.org/wiki/Brotli) format.

Here's our updated configuration for handling static files in production: https://github.com/testdrivenio/django-static-media-files/blob/21ac2562d1cfcd459629e7c484bcdd2e09defa13/config/settings.py#L123-L128

The `STATIC_ROOT` will host all the collected static files (when using `collectstatic`), and the `STATICFILES_STORAGE` will compress our static files. If you want to cache as well, replace `STATICFILES_STORAGE` with,

```python
STATICFILES_STORAGE = 'whitenoise.storage.CompressedManifestStaticFilesStorage'
```

Running the `collectstatic` command creates the compressed files for the static ones. 

```shell
(.venv) ➜  ls -l staticfiles
total 11912
...
-rw-r--r--  1 admin  staff   103 Dec 12 18:40 base.css
-rw-r--r--  1 admin  staff    63 Dec 12 18:40 base.css.br
-rw-r--r--  1 admin  staff    91 Dec 12 18:40 base.css.gz
...
```

Compare the sizes of the original file (`base.css`) and brotli compressed (`base.css.br`). The compressed file can be served faster via a CDN (due to its small size) and quickly decompressed in the browser. 

## Handling media files in development

Two essential options for handling media files are:

- MEDIA_URL: Similar to static URL, the URL where the users can access media files.

- MEDIA_ROOT: Location where all media files are stored.

Here's the configuration used in the example application. https://github.com/testdrivenio/django-static-media-files/blob/21ac2562d1cfcd459629e7c484bcdd2e09defa13/config/settings.py#L135-L136

```python
# config/settings.py

MEDIA_URL = "/media/"
MEDIA_ROOT = BASE_DIR / "uploads"
```

For the media files example, we have created a new app, `userprofile` with a [tiny model](https://github.com/testdrivenio/django-static-media-files/blob/draft/userprofile/models.py) and [a form](https://github.com/testdrivenio/django-static-media-files/blob/draft/userprofile/forms.py) to handle the file uploads. 

```python
# userprofile/models.py

from django.db import models

class Profile(models.Model):
    image = models.FileField()
```

```python
# userprofile/forms.py
from django import forms

class UploadForm(forms.Form):
    image = forms.FileField(label="Upload a picture")
```

For handling the file uploads, we have a function-based view that will get the uploaded image, then Django will process it, save it in the appropriate directory, and send back the image in the same template. 

```python
# userprofile/views.py

from django.shortcuts import render
from .forms import UploadForm
from .models import Profile
from django.http.request import HttpRequest
from django.http.response import HttpResponse


def profile(request: HttpRequest):
    if request.method == "POST":
        form = UploadForm(request.POST, request.FILES)
        if form.is_valid():
            profile = Profile(image=request.FILES["image"])
            profile.save()
            return render(request, "profile.html", {"profile": profile})
        else:
            return HttpResponse("Image upload failed")
    form = UploadForm()
    return render(request, "profile.html", {"form": form})
```

We won't be using the static tag for media files as done in the case of static files. The uploaded file URL can be obtained directly from the `profile` object created. https://github.com/testdrivenio/django-static-media-files/blob/21ac2562d1cfcd459629e7c484bcdd2e09defa13/templates/profile.html#L14-L16

Django's development server is not capable of serving media files. A workaround is to add the media root as a static path. https://github.com/testdrivenio/django-static-media-files/blob/21ac2562d1cfcd459629e7c484bcdd2e09defa13/config/urls.py#L28-L29

Clone and run the application to see both static and media files in development. Django will take care of the naming (in case multiple uploads have the same name). 

## Handling media files in production

Using external storage with a good CDN is recommended for handling media files in production. [Django-storages](https://django-storages.readthedocs.io/en/latest/) supports S3, Google Cloud Storage, and other cloud storage providers. Django-storages can be used to store static files as well.

Check out these resources to learn how to manage static and media files. The posts also cover the private handling of media files. 

- s3: [https://testdriven.io/blog/storing-django-static-and-media-files-on-amazon-s3/](https://testdriven.io/blog/storing-django-static-and-media-files-on-amazon-s3/)

- DigitalOcean Spaces: [https://testdriven.io/blog/django-digitalocean-spaces/](https://testdriven.io/blog/django-digitalocean-spaces/)

## Conclusion

Static and media files are different and must be treated differently for security purposes.

In this article, you saw examples of how to serve static files in development and production. In addition, the article also covered the different settings for static and media files, how Django handles them with minimal configuration, and how to serve media files in development. However, managing media files in production is out of the scope of this article, but the linked posts take care of the topic excellently. 

You can find the example project on GitHub: https://github.com/testdrivenio/django-static-media-files.

models.py
# -*- coding: utf-8 -*-
from django.db import models
from django.contrib import admin
from django.contrib.contenttypes import generic


class User(models.Model):
    email = models.EmailField(unique=True)
    facebook_user_id = models.CharField(max_length=32, blank=True, null=True, unique=True, db_index=True)
    facebook_token = models.CharField(max_length=128, blank=True, null=True)
    user_name = models.CharField(max_length=100)
    deleted = models.BooleanField(default=False)

    def delete(self):
        self.deleted = True
        self.save() 
 
    def __unicode__(self):
        return self.email


admin.site.register(User)


views.py

# -*- coding: utf-8 -*-
from django.http import HttpResponse, HttpResponseNotFound
from django.core.exceptions import ObjectDoesNotExist
from django.core import serializers
from django.http import HttpResponse
from django.template import Context, loader
from django.http import HttpResponseRedirect, Http404
from django.views.decorators.csrf import csrf_exempt
from tachikoma.apps.api.models import User
import json
from tachikoma.apps.api.forms import UserForm
from tachikoma.libs.json_response import JSONResponse

@csrf_exempt
def register_facebook_token(request):
    if not request.method == 'POST':
        return JSONResponse(False, 'you must be to access by *POST* of http method.').response()
    if request.method == 'POST':
        form = UserForm(request.POST)
        if not form.is_valid():
            return JSONResponse(False,'DATA is not valid.').response()
        elif request.method == 'POST':
            return JSONResponse(True).response()
        else:
            User.objects.create(
                facebook_token = request.POST.get('facebook_token'),
                facebook_user_id = request.POST.get('facebook_user_id'), 
                email = request.POST.get('email'))
            return JSONResponse(True).response()
    
@csrf_exempt
def is_registered(request,facebook_user_id):
    if not request.method == 'GET':
        return JSONResponse(False, 'you must be to access by *GET* of http method.').response()
    else:
        try:
            user = User.objects.get(facebook_user_id__exact=facebook_user_id)
            return JSONResponse(True).response()
        except:
            return JSONResponse(False).response()



forms.py

from django import forms

class UserForm(forms.Form):
    facebook_user_id = forms.CharField()
    facebook_token = forms.CharField(max_length=128)
    email = forms.EmailField()



urls.py

# -*- coding: utf-8 -*-
from django.contrib import admin
from django.conf.urls.defaults import patterns, include, url 

admin.autodiscover()

# See: https://docs.djangoproject.com/en/dev/topics/http/urls/
urlpatterns = patterns('tachikoma.apps.api.views',
    url(r'^api/register_facebook_token$','register_facebook_token'),
    # Admin panel and documentation:
    url(r'^admin/doc/', include('django.contrib.admindocs.urls')),
    url(r'^admin/', include(admin.site.urls)), 
)

from django.contrib import admin
from django.conf.urls.defaults import patterns, include, url 
admin.autodiscover()
urlpatterns += patterns('tachikoma.apps.api.views',
    url(r'^api/facebook_token/(?P<facebook_user_id>\d+)/is_registered$', 'is_registered'),
    url(r'^admin/doc/', include('django.contrib.admindocs.urls')),
    url(r'^admin/', include(admin.site.urls)),
)

from django.contrib import admin
from django.conf.urls.defaults import patterns, include, url 
admin.autodiscover()
urlpatterns += patterns('tachikoma.apps.api.views',
    url(r'^api/facebook_token/(?P<facebook_user_id>\d+)$','delete'),
    url(r'^admin/doc/', include('django.contrib.admindocs.urls')),
    url(r'^admin/', include(admin.site.urls)),
)


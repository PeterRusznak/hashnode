## LDAP Authentication with Django -Part 2

In the  [previous part ](https://fullstackwithpr.hashnode.dev/ldap-authentication-with-django-part-1) we set up an LDAP server on localhost and we also installed and configured Django. Then we started  Django and configured it  in a way that we were able to log in using our LDAP user (i.e. without the need to create a Django-user). So now we have full access to Django admin, and we can take advantage of the full Django functionality, even without creating a Django user, therefore everything  seems to be done and dusted, right? Not quite, I am afraid.

### The Problem

Let's say we have an LDAP user, *cisk* :

![user_cisk.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1641479474787/Kji2u5Lwa.png)

and using this user I would like to log in to Django (which should not be a problem, if you followed the previous part):

![login_cisk.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1641479735379/XVc24yQZg.png)

The password happends to be *test*, to be sure I can even check it using LDAP password checker:

![cisk_pw_check.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1641479894931/0VisvehNB.png)

Once logged in, you can see the user. But this is the place where the problem manifests itself. You can see, that there is a possibility to change the password. Actually there are two possibilities to change the password. One at the top right corner, another one a bit below. Just look for the red marks in the picture below:

![ch_PW.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1641482058409/O6TDh-qHz.png)

That's bad, remember, we logged in using our LDAP user (*cisk*), and this LDAP user has an *LDAP password *(*test*) and that should be enough. If a user decides to change the password here, then it would not effect the original one on the LDAP server,  resulting an unfortunate situation where this user would have an LDAP and a Django password, which means two different passwords, which means confusion.. not good. 


### The Solution

Since we already have an LDAP password, and we can log in using this, the best and simplest way is if we just disable this 'change password' functionality. By disabling, I mean it should not be even visible. If it is not there, it can not cause confusion.

Let's have a look what is at the bottom of ```settings.py```: 

```
AUTHENTICATION_BACKENDS = (
        'django_auth_ldap.backend.LDAPBackend',
        'django.contrib.auth.backends.ModelBackend',          
)
``` 

What we need to subclass is ```LDAPBackend.``` Create a file on the same level where 
```settings.py``` is, and name it ```backend_ldap.py```. The content should be the following:

```
from django_auth_ldap.backend import LDAPBackend
from django.contrib.auth import get_user_model

class CustomLDAPBackend(LDAPBackend):

    def authenticate_ldap_user(self, ldap_user, password):
        print("LDAP_USER: ",ldap_user)      
        print("PW: ",password)       

        user = ldap_user.authenticate(password)     
       
        print("BEFORE set_unusable_password(): ", user.has_usable_password())
        user.set_unusable_password()
        user.save()

        print("AFTER set_unusable_password(): ", user.has_usable_password())

        return user
``` 

As you can see, we we get the user object, set its password unusable, and save it. We also do some logging, but that is irrelevant. Don't forget to use the customized class in settings:

```
AUTHENTICATION_BACKENDS = (       
        'django.contrib.auth.backends.ModelBackend',
        'ldappro.backend_ldap.CustomLDAPBackend',       
)
``` 

If you restart the page, you'll see that at least the one at the top right corner has gone. But the other one is still there...


![Still_there.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1641482620224/CzhVa4DSo.png)

 [This stackoverflow answers](https://stackoverflow.com/questions/62421279/how-to-remove-change-password-link-from-django-admin-site)  says if ``` user.has_usable_password() ``` returns False, then Django will not display change password URL for that user.

``` user.has_usable_password() ```  can be made to return False by calling ``` user.set_unusable_password().``` This works very well for the one in the top right corner, but strangely does not affect the other one.  Let's fix this problem, by overriding the template.

Create a folder called ```templates``` and inside it create another folder called ```admin``` and inside it create a file called ```change_form.html``` Make sure that the ```templates``` folder is at the same level as ```manage.py``` 
![templates.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1641483470876/dxYIaO1-t.png)

The content should be the following:
```
{% extends "admin/change_form.html" %}
{% load i18n %}

{% block field_sets %}
{% for fieldset in adminform %}

  {% if forloop.first and user.has_usable_password %}
    {% include "admin/includes/fieldset.html" %}
  {% endif %}
 
  {% if not forloop.first %}
     {% include "admin/includes/fieldset.html" %}
  {% endif %}

{% endfor %}
{% endblock %}
```
As you can see, we extend the original template and conditionally display its fieldset. Since the "change passsword" fieldset happens to be the first, we display it only if the ```user.has_usable_password```  returns 'True'. In case of an LDAP user, the user object doesn't have usable_password (i.e. returns False), because we overrode ```authenticate_ldap_user()``` in ```CustomLDAPBackend``` that means no password changing for our LDAP user.

We also need to modify our ```settings.py``` in order to make the location of the templates recognizable for Django. Just a slight modification, everything stays the same, apart from the content of 'DIRS'. (Obviously, you also need to import ```os.```, but that's really no big deal.)

```
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, 'templates')],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```

If everything is OK, you can see that there is no 'Change Password' opportunity **for LDAP users**:

![no_pw_ch.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1641485520068/wrbC7FfmS.png)

As for a *normal* Django user, one which has nothing to do with LDAP, the change password functionality is still there, as it should:


![admin.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1641485732038/gM77zyxDE.png)

Congrats, you have reached the end, and now you know how to disable passwords for LDAP backed Django users while keep this functionality accessible to *normal* Django users.

Thanks for your attention.

 




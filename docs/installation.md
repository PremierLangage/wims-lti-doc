# Installation

## Dependencies

* git
* Python 3.6+
* pip3
* Apache2 & mod_wsgi

Note that you need `mod_wsgi` for Python 3 and not Python 2. For instance on Debian / Ubuntu,
you may need to uninstall `libapache2-mod-wsgi` and install `libapache2-mod-wsgi-py3` :

```bash
apt-get remove libapache2-mod-python libapache2-mod-wsgi
apt-get install libapache2-mod-wsgi-py3
```

For more informations about `mod_wsgi`, see the [official documentation](https://modwsgi.readthedocs.io/en/develop/user-guides/quick-installation-guide.html).

## Downloading

You can clone the server from github:

```bash
git clone https://github.com/PremierLangage/wims-lti.git
cd wims-lti
```

## Configuration of WIMS-LTI

You must edit `wims-lti/wimsLTI/config.py` to change some default parameters, 
see [Parameters reference](https://docs.djangoproject.com/en/3.0/ref/settings/).  
You should especially look at:

* [*DEBUG*](https://docs.djangoproject.com/en/3.0/ref/settings/#debug)
* [*SECRET_KEY*](https://docs.djangoproject.com/en/3.0/ref/settings/#secret-key)
* [*ALLOWED_HOSTS*](https://docs.djangoproject.com/en/3.0/ref/settings/#allowed-hosts)
* [*ADMINS*](https://docs.djangoproject.com/fr/2.2/ref/settings/#admins)
* [*EMAIL_HOST*](https://docs.djangoproject.com/en/3.0/ref/settings/#email-host) 
and [*SERVER_EMAIL*](https://docs.djangoproject.com/en/3.0/ref/settings/#server-email)
* [*DATABASES*](https://docs.djangoproject.com/fr/2.2/ref/settings/#databases) - If 
you want to use something else than sqlite (see `wimsLTI/settings.py` for the default value).
* *SEND_GRADE_BACK_CRON_TRIGGER* - If you want to change the default scheduled time every grade
are sent back to their LMS (default to 7h and 19h).
* *CHECK_CLASSES_EXISTS_CRON_TRIGGER* - If you want to change the default scheduled time wims-lti
will check if every registered class still exists on the corresponding WIMS server.
* *WIMSAPI_TIMEOUT* - Time before requests sent to a WIMS server from wims-lti times out. Should be increased
                      if some WIMS server contains a lot of classes / users.

Once the parameters are correctly defined, launch the installation script `install.sh`
from `wims-lti/`. It will ask for a user name, an email address, and a password to create
an administration account.


## Configuration of Apache2

> Note that you can use other webserver such as **Nginx** to server wims-lti.

Now, you only have to configure *Apache*,
see [the documentation](https://docs.djangoproject.com/fr/2.1/howto/deployment/wsgi/modwsgi/).
Here an example of such conf:


```text
<VirtualHost *:80>
    ServerName wims-lti.u-pem.fr

    Alias /static /path/to/wims-lti/static/
    <Directory /path/to/wims-lti/static/>
        Require all granted
    </Directory>

    <Directory /path/to/wims-lti/wimsLTI/>
        <Files wsgi.py>
            Require all granted
        </Files>
    </Directory>

    SetEnv PYTHONIOENCODING utf-8
    WSGIDaemonProcess wims-lti python-path=/path/to/wims-lti/wimsLTI/
    WSGIProcessGroup  wims-lti
    WSGIScriptAlias / /path/to/wims-lti/wimsLTI/wsgi.py
</VirtualHost>
```

Finally, *Apache* need the corresponding rights on these files, I.E. `www-data` need to have access to these files :

```sh
chown www-data: /path/to/wims-lti/ -R
```

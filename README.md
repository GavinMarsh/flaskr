# flaskr
Complete and working tutorial from Flasks official documentation page http://flask.pocoo.org/docs/1.0/tutorial/
Have also included wsgi file details for remote deployment.

flaskr is a flask web-application wich has a model-view-controller layout:

Model       = MongoDB

View        = Jinja2

Controller  = Flask/apache2/wsgi

# local-mac apache2/wsgi-express/flask deployment

### Activating flask server:
    export FLASK_APP=flaskr
    export FLASK_ENV=development
    flask run


### Initialise the database
    flask init-db


### wsgi-express file
       
    import sys

    # add your project directory to the sys.path
    project_home = u'/var/www/reddit/'
    if project_home not in sys.path:
    sys.path = [project_home] + sys.path

    # You can't import a variable that is local to a function, instead call
    # the function inside the application-factory to buid and return the app.
    from __init__ import create_app
   
    application = create_app('config.DevelopmentConfig')
    
    application.secret_key = 'Add your secret key'

To run application, navigate to the source files directory reddit/reddit and enter into the cli: 
     mod_wsgi-express start-server local-mac.wsgi

# Pythonanywhere deployment - build & install
Upload installation wheel file to files folder.
    mkvirtualenv flaskr --python=/usr/bin/python3.7

commands for venv:
    # to activate the venv in pythonanywhere, just type workon and the venv name.
    workon flaskr
    
    # to activate in bash
    source venv/bin/activate
    
    # to deativate the venv      
    deactivate

### load Flask    
    acitvate venv
    
    pip install wheel
    python setup.py bdist_wheel     # not sure why this is not working on Ubuntu 16.04?
    export FLASK_APP=flaskr
    pip3 install flaskr-1.0.0-py3-none-any.whl

Pip will install your project along with its dependencies.

Since this is a different machine, you need to run init-db again to create the database in the instance folder.
    
    export FLASK_APP=flaskr
    flask init-db

When Flask detects that it’s installed (not in editable mode), it uses a different directory for the instance folder. You can find it at venv/var/flaskr-instance instead.

### Configure the Secret Key
In the beginning of the tutorial that you gave a default value for SECRET_KEY. This should be changed to some random bytes in production. Otherwise, attackers could use the public 'dev' key to modify the session cookie, or anything else that uses the secret key.

You can use the following command to output a random secret key:

    python -c 'import os; print(os.urandom(16))'

    b'_5#y2L"F4Q8z\n\xec]/'

Create the config.py file in the instance folder, which the factory will read from if it exists. Copy the generated value into it.

venv/var/flaskr-instance/config.py
    SECRET_KEY = b'_5#y2L"F4Q8z\n\xec]/'
You can also set any other necessary configuration here, although SECRET_KEY is the only one needed for Flaskr.


### create new wsgi file:

    ## wsgi_file.py
    import sys

    # add your project directory to the sys.path
    project_home = u'/home/GMM/flaskr/'
    if project_home not in sys.path:
        sys.path = [project_home] + sys.path

    # You can't import a variable that is local to a function, so we need to call
    # the function inside the application-factory to buid and return the app.

    from flaskr import create_app
    application = create_app()


# VirMach Ubuntu 16.04, apache2/wsgi/flask deployment
/etc/apache2/sites-available/sites.wsgi file
Place each site on it's own port and include it's own WSGI daemon process, this will prevent any data leakage between each site.
    
    Listen 80

    <VirtualHost *:80>
        WSGIDaemonProcess reddit display-name=%{GROUP}
        WSGIProcessGroup reddit

        ServerName 107.172.143.209
        WSGIScriptAlias /  /var/www/reddit/reddit.wsgi
        <Directory /var/www/reddit/reddit/>
                Order allow,deny
                Allow from all
        </Directory>
        Alias /static /var/www/reddit/reddit/static
        <Directory /var/www/reddit/reddit/static/>
                Order allow,deny
                Allow from all
        </Directory>

        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel info
        CustomLog ${APACHE_LOG_DIR}/access.log combined

    </VirtualHost>

    Listen 8080

    <VirtualHost *:8080>
        WSGIDaemonProcess reddit display-name=%{GROUP} 
        WSGIProcessGroup reddit

        ServerName 107.172.143.209
        WSGIScriptAlias /  /var/www/reddit/reddit.wsgi
        <Directory /var/www/reddit/reddit/>
                Order allow,deny
                Allow from all
        </Directory>
        Alias /static /var/www/reddit/reddit/static
        <Directory /var/www/reddit/reddit/static/>
                Order allow,deny
                Allow from all
        </Directory>

        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel info
        CustomLog ${APACHE_LOG_DIR}/access.log combined

    </VirtualHost>

### Configure the Secret Key
In the beginning of the tutorial that you gave a default value for SECRET_KEY. This should be changed to some random bytes in production. Otherwise, attackers could use the public 'dev' key to modify the session cookie, or anything else that uses the secret key.

You can use the following command to output a random secret key:

    python -c 'import os; print(os.urandom(16))'

    b'_5#y2L"F4Q8z\n\xec]/'

Create the config.py file in the instance folder, which the factory will read from if it exists. Copy the generated value into it.

venv/var/flaskr-instance/config.py
    SECRET_KEY = b'_5#y2L"F4Q8z\n\xec]/'
You can also set any other necessary configuration here, although SECRET_KEY is the only one needed for Flaskr.


# TODO
- Work out how to access a new instance of the local sqlite built in python database or work out how to use a MongoDB cluster with the web-application.
- check and see why pytest is not running? may have something to do with the file location of setup.cfg? or may have something to do with  what directory you are in when you run $ pytest from the CLI ? http://flask.pocoo.org/docs/1.0/tutorial/tests/


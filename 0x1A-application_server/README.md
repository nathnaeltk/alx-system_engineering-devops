# :book: 0x1A. Application server
## Application server


## Project setup
```bash
mkdir 0x1A-application_server
touch 0x1A-application_server/README.md
```

# :computer: Tasks
Set up and Server AirBnB clone v2 - Web framework on web-01
## [0. Set up development with Python]()
Process to setting up flask web application in a server.

```bash
# Within the web server.
# AirBnB_clone_v2 clone repo
git clone https://github.com/MosesSoftEng/AirBnB_clone_v2.git AirBnB_clone_v2

# Install flask
sudo pip3 install Flask

# Run flask web app server, not required step
# cd AirBnB_clone_v2
# python3 -m web_flask.0-hello_route

# Error: Port 5000 is in use by another program
# List ports and application using port 5000
sudo netstat -lpdn | grep ":5000"

# Kill the app and processes listening on port 5000
sudo pkill -f agent

# Run flask web app server
python3 -m web_flask.0-hello_route

# Test flask connection in a different terminal by accessing /airbnb-onepage/
#   route in flask.
curl 127.0.0.1:5000/airbnb-onepage/
```

## [1. Set up production with Gunicorn]()
Setup Gunicorn WSGI for flask web application.

```bash
# Install Gunicorn
pip3 install gunicorn

# Confirm installation
gunicorn --version

# Set gunicorn entry point to flask app
cd AirBnB_clone_v2
gunicorn --bind 0.0.0.0:5000 web_flask.0-hello_route:app

# Bind flask app to Gunicorn instance, execute in backgroud terminal
tmux new-session -d 'gunicorn --bind 0.0.0.0:5000 web_flask.0-hello_route:app'

# Confrim binding
ps ax | grep gunicorn

# Test gunicorn in a different terminal
curl 127.0.0.1:5000/airbnb-onepage/
```

## [2. Serve a page with Nginx](2-app_server-nginx_config)
```bash
# Set the configuration in /etc/nginx/sites-enabled/default
sudo vi /etc/nginx/sites-enabled/default

# Server route /airbnb-onepage/ to the public
location /airbnb-onepage/ {
    proxy_pass http://127.0.0.1:5000/airbnb-onepage/;
}

# Confirm congiuration files and Restart nginx.
sudo nginx -t
sudo service nginx restart

# Test in different machine
curl -sI 100.25.17.98/airbnb-onepage/
curl 100.25.17.98/airbnb-onepage/

touch 2-app_server-nginx_config
```

## [3. Add a route with query parameters ](3-app_server-nginx_config)
```bash
# Set the route configuration in /etc/nginx/sites-enabled/default
    location ~ /airbnb-dynamic/number_odd_or_even/([0-9]+)$ {
        proxy_pass http://127.0.0.1:5001/number_odd_or_even/$1;
    }

# Confirm nginx configuration and restart nginx.
sudo nginx -t
sudo service nginx restart

# Bind flask app to Gunicorn instance, execute in backgroud terminal
cd AirBnB_clone_v2
tmux new-session -d 'gunicorn --bind 0.0.0.0:5001 web_flask.6-number_odd_or_even:app'

# Confrim binding
ps ax | grep gunicorn

#Tests
curl 127.0.0.1:5000/airbnb-onepage/
curl 100.25.17.98/airbnb-onepage/

curl -sI 3100.25.17.98/airbnb-onepage/

curl 127.0.0.1:5001/number_odd_or_even/6
curl 100.25.17.98/airbnb-dynamic/number_odd_or_even/6

touch 3-app_server-nginx_config
```

## [4. Let's do this for your API](4-app_server-nginx_config)
```bash
git clone https://github.com/marcrine-geek/AirBnB_clone_v3.git AirBnB_clone_v3

# Bind gunicorn to flask api
cd AirBnB_clone_v3
tmux new-session -d 'gunicorn --bind 0.0.0.0:5002 api.v1.app:app'

# Confrim binding
ps ax | grep gunicorn

# Run flask api
cd AirBnB_clone_v3
HBNB_API_HOST=0.0.0.0 HBNB_API_PORT=5002 python3 -m api.v1.app

# Error
# ModuleNotFoundError: No module named 'sqlalchemy'
# Install SQLAlchemy
sudo pip install SQLAlchemy

# confirm installation
python3
>>> import sqlalchemy
>>> sqlalchemy.__version__ 
'1.4.22'

# Error
# ModuleNotFoundError: No module named 'flask_cors'
# Install Flask-Cors
pip3 install Flask-Cors

# Error
# Port 5000 is in use by another program
# Change flask app ports using environment variables
HBNB_API_HOST=0.0.0.0 HBNB_API_PORT=5002 python3 -m api.v1.app

# Configure nginx to allow request to flask api in /etc/nginx/sites-enabled/default
    location /api {
        proxy_pass http://127.0.0.1:5002/api;
    }

# Confirm congiuration files and Restart nginx.
sudo nginx -t
sudo service nginx restart

# Tests
# On same server
curl 127.0.0.1:5002/api/v1/states
curl 127.0.0.1:5002/api/v1/status/

# From another comp
curl 100.25.17.98/api/v1/states
curl http://100.25.17.98/api/v1/status/

touch 4-app_server-nginx_config
```

## [5. Serve your AirBnB clone ](5-app_server-nginx_config)
```bash
# Clone project to server
git clone https://github.com/MosesSoftEng/AirBnB_clone_v4.git AirBnB_clone_v4

# Install mysql 5.7
https://github.com/MosesSoftEng/alx-system_engineering-devops/tree/master/0x14-mysql 

# Test if web dynamic runs
cd AirBnB_clone_v4
HBNB_MYSQL_USER=hbnb_dev HBNB_MYSQL_PWD=hbnb_dev_pwd HBNB_MYSQL_HOST=localhost HBNB_MYSQL_DB=hbnb_dev_db HBNB_TYPE_STORAGE=db python3 -m web_dynamic.2-hbnb
curl 127.0.0.1:5003/2-hbnb/

# Error
# ModuleNotFoundError: No module named 'MySQLdb'
# Instal MySQLdb
sudo apt-get install python3-dev
sudo apt-get install -y libmysqlclient-dev
sudo apt-get install zlib1g-dev
sudo pip3 install mysqlclient

# Test MySQLdb installation
python3
>>> import MySQLdb
>>>

# Error
# "Access denied for user 'hbnb_dev'@'localhost' (using password: YES)")
# Create user under AirBNB_V2
cat setup_mysql_dev.sql | mysql -hlocalhost -uroot -p

# Error
# Port 5000 is in use by another program.
# Change port in 5000 to 5003 in web_dynamic/2-hbnb.py

# Bind gunicorn to flask dynamic
cd ~/AirBnB_clone_v4
tmux new-session -d 'gunicorn --bind 0.0.0.0:5003 web_dynamic.2-hbnb:app'

# Confrim binding
ps ax | grep gunicorn

# Test
curl 127.0.0.1:5003/2-hbnb/
curl 0.0.0.0:5003/2-hbnb/

# In browser
http://127.0.0.1:5003/2-hbnb/

# Web dynamic set 2-hbnb as root route
    location / {
        proxy_pass http://127.0.0.1:5003/2-hbnb/;
    }

# Confirm congiuration files and Restart nginx.
sudo nginx -t
sudo service nginx restart

# In another machine
curl 100.25.17.98/

# Enable CORS app.py
cors = CORS(app, resources={r"/api/*": {"origins": "*"}})

touch 5-app_server-nginx_config
```

## [6. Deploy it!](gunicorn.service)

```bash
# Create configuration file gunicorn.service at /etc/systemd/system/
sudo touch /etc/systemd/system/gunicorn.service
sudo chmod +x /etc/systemd/system/gunicorn.service

sudo systemctl daemon-reload
sudo systemctl start gunicorn
sudo systemctl enable gunicorn

ps ax | grep gunicorn


curl -s 100.25.17.98/2-hbnb | tail -5
curl -s 100.25.17.98/ | tail -5
```

```bash
touch 4-reload_gunicorn_no_downtime
chmod +x 4-reload_gunicorn_no_downtime
```

# :books: References
1. [How To Serve Flask Applications with Gunicorn and Nginx on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-serve-flask-applications-with-gunicorn-and-nginx-on-ubuntu-16-04)



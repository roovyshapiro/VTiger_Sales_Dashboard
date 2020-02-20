# VTiger Sales Dashboard

[![](https://i.imgur.com/EcnseP9.png)](https://i.imgur.com/EcnseP9.png)

### Overview
VTiger Sales Dashboard is a Django application which shows live data from your VTiger sales team. We gather phone calls and the opportunity stages which have been changed in a given day and assign a point value to each. The goal is for the salesperson to reach 100 points in a day by whatever means necessary. They can reach their goal by making 100 phone calls, or by making 50 phone calls and conducting 5 demos, etc. The various sales stages will change depending on the way you've configured your Opportunities, stages and sales pipelines within VTiger.

### Connecting to VTiger
Make sure to have a file named 'credentials.json' within the main 'sales_dashboard' directory. It should be structured like this:
```python
{"username": "<vtiger_username>", "access_key": "<access_key>", "host": "https://< custom_hostname>vtiger.com/restapi/v1/vtiger/default"}
```

### Celery Beat / Gathering VTiger Data on a Schedule
Celery Beat is used to gather the data from VTiger every ten minutes and save it to the database. It also runs this in the background so it doesn't freeze the webpage and affect the user's experience. 
Here's a great article explaining the basics if you're unfamiliar with celery/celery beat:
https://www.merixstudio.com/blog/django-celery-beat/

Javascript is then used to refresh the page every few minutes if the 'Auto Refresh' button is checked. There is a countdown timer until the page refreshes.

### Starting the Server
It's a good idea to use a virtual environment with Python applications especially with Django. Here's a great article explaining the process:
https://realpython.com/python-virtual-environments-a-primer/

The basic process is:
```python
#Install Virtualenv
$ pip install virtualenv
#Create a virtual environment directory named 'env'
$ python3 -m venv env
#Activate the 'env' environment
$ source /env/bin/activate
#Your shell should look like this:
(env)$
```

Next, make sure to install the dependencies in requirements.txt using,
`pip3 install -r requirements.txt
`

Next, run the django migrations to ensure your database is setup properly.
`python3 manage.py migrate
`
The final step is to start the server. You can use the 'startapps.sh' file which will 
- run the redis-server (broker for celery)
- run the celery worker
- run the celery beat scheduler
- run the built-in django web server.

```bash
#!/bin/bash

redis-server &
celery -A sales_dashboard worker &
celery -A sales_dashboard beat -l INFO --scheduler django_celery_beat.schedulers:DatabaseScheduler &
python3 manage.py runserver 0.0.0.0:8000
```
Afterwards, you should be able to navigate to access it by opening up a web browser and navigating to http://127.0.0.1:8000

### Changing Opportunity Sales Stages
Unfortunately, there are many places where the sales stages are hard coded in. If your sales stages are different, you'll need to edit them in
- sales_dashboard/dashboard/models.py
- sales_dashboard/dashboard/tasks.py
- sales_dashboard/dashboard/templates/dashboard/dashboard.html

This will hopefully be fixed in future iterations so that it doesn't matter which sales stages are set.

### Changing the Points Calculation
Currently, each phone call is worth 1 point, each Demo Scheduled is worth 5 points and each Demo Given is worth 10 points. 
To edit the score calculation, edit the values in  sales_dashboard/dashboard/models.calculate_scores()
The values shown in the score_key in the HTML are static and have no bearing on the calculation.

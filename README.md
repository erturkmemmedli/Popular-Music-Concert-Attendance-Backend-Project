# Project Overview

A website is built for a popular music band. The fans of the band and other users will be able to perform the following actions on the site:
- See pictures from past events
- See popular lyrics of songs
- See a list of upcoming events
- Create an account
- Sign in and register for an event
- Sign in and see past registrations

# Project Implementation Steps

In order to successfully complete the project, the following steps will be taken:
1. Create Get Pictures microservice in Flask  
    - Create CRUD endpoints for picture as a resource.   
    - Create health endpoint for the microservice.
2. Create Get Songs microservice in Flask 
    - Set up MongoDB database.  
    - Implement the service to retrieve song lyrics from the database.  
    - Create health endpoint for the microservice.
3. Create the Main Application in Django  
- Create the concert model.
- Use the built in Django user model. 
- Migrate the model to create tables in the SQLite database. 
- Implement controllers to send data to pre-defined templates.
4. Deploy the services and application  
- Deploy Get Pictures to IBM Code Engine.
- Deploy Get Songs and MongoDB to Redhat OpenShift.  
- Deploy Main Application to IBM Kubernetes Service.

# Backend Architecture

![Architecture](https://d3c33hcgiwev3.cloudfront.net/imageAssetProxy.v1/Vy5tmLkrTAKXylRYbaAO5w_81e966381c9a4d7cb82ecc32b03c23f1_image.png?expiry=1702771200000&hmac=3LXIWxKhBqnlhgrivc0oWS766T3u6KDtHyzwGw72yEk)

1. User visits the Django Website home page.
Anonymous use cases:
2. The song page shows songs and lyrics.
3. The pictures page shows pictures from past concerts.
Admin use cases:
4. Let the admin user change the concert date.
Signed in use cases:
1. The user signs into the application.
2. The user is able to see their concerts.
3. The user is able to book a concert.
4. The user is able to delete their reservation.  

# Procedure

1.  Clone git repo: `git clone https://github.com/erturkmemmedli/Popular-Music-Concert-Attendance-Backend-Project.git`
2.  Get into the directory by `cd Popular-Music-Concert-Attendance-Backend-Project/Main-Django-Service`
3.  Get into Django code `cd djangoserver`
4.  Install requirements `pip install -r requirements.txt`
5.  Run the server `python manage.py runserver`
6.  It will tell you that you have unapplied migrations.

    **Migrations** are Django’s way of propagating changes you make to your models (adding a field, deleting a model, etc.) into your database schema. They’re designed to be mostly automatic, but you’ll need to know when to make migrations, when to run them, and the common problems you might run into. There are several commands which you will use to interact with migrations and Django’s handling of database schema:

    1. **_migrate_**, which is responsible for applying and unapplying migrations.
    2. **_makemigrations_**, which is responsible for creating new migrations based on the changes you have made to your models.
    3. **_sqlmigrate_**, which displays the SQL statements for a migration.
    4. **_showmigrations_**, which lists a project’s migrations and their status.

7.  Create the initial migrations and generate the database schema:

    ```shell
    python manage.py makemigrations
    python manage.py migrate
    ```

8.  Run server successfully this time: `python manage.py runserver`
9.  Launch Application
10. Click on Songs and Photos
11. Click on Concerts, no existing Concert present
12. Let's create admin user `python manage.py createsuperuser`
    1. Username: `admin`
    2. Email address: _leave blank, simply press enter_
    3. Password: Your choice, or simply `qwerty123`
13. Run the server again `python manage.py runserver` and goto admin: `http://localhost:8000/admin/`
14. Enter the admin user details you created in previous step.
15. Now you are in the admin section built by Django.

    One of the most powerful parts of Django is the **automatic admin interface**. It reads metadata from your models to provide a quick, model-centric interface where trusted users can manage content on your site. The admin’s recommended use is limited to an organization’s internal management tool. It’s not intended for building your entire front end around.

16. Add a Concert:
    1. Concert name: `Metallica 2023`
    2. Duration: `72`
    3. City: `Istanbul, Turkey`
    4. Date: `2023-12-21`
17. Click on `View Site` meny at the top
18. Now if you visit `Concerts`, you will see Coachella listed.
19. Our Django application is now running, but Songs and Photos are hard coded.
20. Open `concert\views.py`
21. See `songs` and `photos` definition.

    1. Retrieve `songs` from a REST endpoint by replacing the code with following:

    ```python
    songs = req.get(
        "https://raw.githubusercontent.com/captainfedoraskillup/private-get-songs/main/backend/data/songs.json").json()
    return render(request, "songs.html", {"songs": songs})
    ```

    2. Retrieve `photos` from a REST endpoint by replacing the code with following:

    ```python
    photos = req.get(
        "https://raw.githubusercontent.com/captainfedoraskillup/private-get-pictures/main/backend/data/pictures.json").json()
    return render(request, "photos.html", {"photos": photos})
    ```

22. Verify Songs and Photos changes. Visit the Songs section, you will see a longer list of songs, clicking on each will show its Lyrics in a modal dialog. While going into Photos, you will see more than two.
23. Now back to Concerts, click on the concert Coachella we created. You will see an RSVP page.
24. RSVP Page shows you details of the Concert along with an option to either: Attend, Not Attend or no Option `-`.
25. If you open `concert_detail.html`, you will see an html form:

        ```html
                <form action="{% url 'concert_attendee' %}" method="POST">
                {% csrf_token %}
                <input
                  name="concert_id"
                  type="number"
                  value="{{concert_details.id}}"
                  hidden="hidden"
                />
                <div class="input-group mb-3">
                  <label class="input-group-text" for="attendee_choice">RSVP</label>
                  <select class="form-select" name="attendee_choice" required>
                    {% for attending_choice in attending_choices %}
                      <option {% if attending_choice.0 == status %}selected {% endif %} value="{{ attending_choice.0 }}">{{ attending_choice.1 }}</option>
                    {% endfor %}
                  </select>
                </div>
                <input type="submit" class="btn btn-primary" />
              </form>
        ```

26. On `Submit` of this form, the details are sent to `concert_attendee` in `concert\views.py`.
27. It does two validations:
    `if request.user.is_authenticated:` means whether the user is authenticated. Because anonymous users are not allowed to RSVP.
    Then `if request.method == "POST":` to check whether it is an `HTTP POST` event.
28. From the `body` of the `POST` method, it takes `concert_id` and `attendee_choice`.
29. Then it checks, whether a selection was made previously, if yes, then update it. Otherwise insert new selection the databaes for this user.
30. Finally, redirect the user.

## Cleanup while testing

```shell
find . -path "*/migrations/*.py" -not -name "__init__.py" -delete
find . -path "*/migrations/*.pyc"  -delete
find . -path "*/db.sqlite3"  -delete
python manage.py makemigrations
python manage.py migrate
python manage.py runserver
```

##  To move the data from SQLite to MySQL

Execute:

`python manage.py dumpdata > datadump.json`

Next, change your `settings.py` to the mysql database.

Finally:

`python manage.py loaddata datadump.json`

##  Containerize the application

1. build a docker image
    ```
    docker build . -t concert
    ```
1. tag docker image with the correct registry information
    ```
    docker tag concert captainfedora/concert:v1
    ```
    The above command tags to `captainfedora` repository on dockerhub with the image name of `concert` and label of `v1`

1. Run the docker image to validate everything is working
    ```
    docker run -p 8000:8000 captainfedora/concert:v1
    ```
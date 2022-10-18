# Deploying Your Flask Project to Render

Render is a web application that makes deploying applications easy for a
beginner. (not a good start, our students are not beginners when they reach mod4, and that also suggests should they run in to issues, its easy for just a beginner, how come I am struggling? why am I paying $30k and halfway through the course but I can't do what a beginner should be able to do?) The free tier allows you to create a database instance to store
database schemas and tables for multiple applications, as well as host web
services (such as APIs) and static sites.

Before you begin deploying, **make sure to remove any `print` statements,
`console.log`s, or `debugger`s in any production code**. You can search your 
entire project folder to see if you are using them anywhere by clicking on 
the magnifying glass icon on the top left sidebar of VSCode.

### David note: what happens if I don't remove logs?

In the following phases, you will configure the Postgres database for production
and configure scripts for building and starting the Flask production server.

## Best Deployment Practices

For this project, you can set Render to auto-deploy your project every time you
complete a feature! Essentially, anytime you merge a feature branch into the
development branch.

Merge your development branch into your main branch, and then follow the
deployment instructions below.

## Phase 0: Setting up your Flask + React application

First, you will need to check and possibly adjust some configuration related to
the React build to ensure a smooth deployment.  (add note that if you are using the flask starter these are already set up for you, skip to Phase 1)

### David note: no they're not.

### Part A: Adjust configuration for building static files for React

In the **app/__init.py__** file, you will configure where React will build the
application, and the url for the static resources for server-side rendering.

First, adjust the app variable to include additional arguments:

```py
# app/__init.py__ file

# ... other imports
app = Flask(__name__, static_folder='../react-app/build', static_url_path='/')
# ...
```
(need a note on why we need to do this)


Next, adjust the routes at the bottom of the file to make sure that the static
resources can be accessed at the right locations. The routes should look like
this:

```py
# app/__init.py__ file

# bottom of file
@app.route('/', defaults={'path': ''})
@app.route('/<path:path>')
def react_root(path):
    if path == 'favicon.ico':
        return app.send_from_directory('public', 'favicon.ico')
    return app.send_static_file('index.html')
```

## Part B (Recommended): Adjust build script for React-App

Navigate to the __react-app/package.json__ file, and add `CI=false` to the build
script.

```json
// react-app/package-json file

// ...
    "build": "CI=false && react-scripts build",
// ...
```

This will allow your build to continue even if there are deprecation warnings on
your dependencies. If this were a commercial application in production, you
would probably want to keep the Continuous Integration (CI) enabled and address
all of the warnings to keep your application up-to-date. But on a personal
project, those warnings are fine, and it is best to make sure they are not an
obstacle to deployment.

If you made any changes in this phase, merge your development branch into your
main branch, and then continue.

## Phase 1: Set up Render.com account

_Skip this setep if you already have a Render.com account connected to your
GitHub account._

Navigate to the [Render homepage] and click on "Get Started". On the Sign Up
page, click on the GitHub button. This will allow you to sign in to Render
through your GitHub account, and easily connect your repositories to Render for
deployment. Follow the instructions to complete your registration and verify
your account information.

## Phase 2: Create a Postgres Database Instance

_Skip this step if you have already created your Render Postgres database
instance for another application._

Sign in to Render using your GitHub credentials, and navigate to your
[Dashboard].

Click on the "New +" button in the navigation bar, and click on "PostgreSQL" to
create your Postgres database instance.

In the name field, give your database a descriptive name. Note that all of your
applications will share this database instance, so make it general (for example,
you might name it "App-Academy-Projects"). For the region field, choose the
location nearest to you. The rest of the fields in this section can be left
blank. (should add a note on the postgres version field, even if they dont need to change the version)

Click the "Create Database" button to create the new Postgres database instance.
Within a few minutes, your new database will be ready to use. Scroll down on
the page to see all of the information about your database, including the
hostname, user name and password, and URLs to connect to the database.

You can access this information anytime by going back to your [Dashboard], and
clicking on the database instance.

## Phase 3: Create a New Web Service

From the [Dashboard], click on the "New +" button in the navigation bar, and
click on "Web Service" to create the application that will be deployed.

> _Note: If you set up your Render.com account using your GitHub credentials,
> you should see a list of applications to choose from. If you do not, click on
> "Configure Account" for GitHub in the right sidebar to make the connection
> between your Render and GitHub accounts, then continue._

Look for the name of the application you want to deploy, and click the "Connect"
button to the right of the name.

Now, fill out the form to configure the build and start commands, as well as add
the environment variables to properly deploy the application.

### Part A: Configure the Start and Build Commands

Start by giving your application a name. This is the name that will be included
the URL of the deployed site, so make sure it is clear and simple. The name
should be entered in sword-case. (what is sword case?  I think that means hyphens between words, but google searches do not find sword case, they are coming back with kabob case for this)

Leave the root directory field blank. By default, Render will run commands from
the root directory.

Make sure the Environment field is set set to "Python 3", the Region is set to
the location closest to you, and the Branch is set to "main".

Next, add your Build script. (called "build commands" on render) This is a script that should include everything
that needs to happen _before_ starting the server.

For your Flask project, enter the following script into the Build field, all in
one line: (this field was prepopulated with 'pip install -r requirements.txt' for me)

```shell
# build script - enter all in one line
npm install --prefix react-app &&
npm run build --prefix react-app &&
pip install -r requirements.txt &&
pip install psycopg2 &&
flask db upgrade &&
flask seed all
```

(seeding should not happen on every build.  I know we are limited in what we can do with render, but this is likely going to cause issues with unique constraints or duplicate data if not more.  on my second deploy of my site, cause the first one had issues, get the following error: 

```shell
psycopg2.errors.UniqueViolation: duplicate key value violates unique constraint "users_email_key"
Oct 18 01:37:54 PM  DETAIL:  Key (email)=(demo@aa.io) already exists.
Oct 18 01:37:54 PM  '
```
potential solution is for students to edit the 'build commands' to add or remove the seed all as needed )

This script will install dependencies for the frontend, and run the build
command in the __package.json__ file for the frontend, which builds the React
application. Then, it will install the dependencies needed for the Python
backend, and run the migration and seed files.

Now, add your start script in the Start field: (this field was prepopulated correctly for me)

```shell
# start script
gunicorn app:app
```

### Part B: Add the Environment Variables

Click on the "Advanced" button at the bottom of the form to configure the
environment variables your application needs to access to run properly. In the
development environment, you have been securing these variables in the __.env__
file, which has been removed from source control. In this step, you will need to
input the keys and values for the environment variables you need for production
into the Render GUI.

Click on "Add Environment Variable" to start adding all of the variables you
need for the production environment.

Add the following keys and values in the Render GUI form:

- SECRET_KEY (click "Generate" to generate a secure secret for production)
- FLASK_ENV production
- FLASK_APP app
- REACT_APP_BASE_URL https://this-application-name.onrender.com

(note if the name for you application already exists, it seems render adds a hash afterward, ie: I named my app 'test-deploy', render did not provide any feedback that this was already used, and just named my app 'test-deploy-ajhv' so originally my REACT_APP_BASE_URL was not correct)

In a new tab, navigate to your dashboard and click on your Postgres database
instance.

Add the following keys and values:

- DATABASE_URL (copy value from Internal Database URL field)

_Note: Add any other keys and values that may be present in your local __.env__
file. As you work to further develop your project, you may need to add more
environment variables to your local __.env__ file. Make sure you add these
environment variables to the Render GUI as well for the next deployment._

Next, choose "Yes" for the Auto-Deploy field. This will re-deploy your
application every time you push to main.

Now, you are finally ready to deploy! Click "Create Web Service" to deploy your
project. The deployment process will likely take about 10-15 minutes if
everything works as expected. You can monitor the logs to see your build and
start commands being executed, and see any errors in the build process.

When deployment is complete, open your deployed site and check to see if you
successfully deployed your Flask application to Render! You can find the URL for
your site just below the name of the Web Service at the top of the page
(https://this-application-name.onrender.com).

## Phase 4: Ongoing Maintenance

The main limitation of the free Render Postgres database instance is that it
will be deleted after 90 days. In order to keep your application up and running,
you MUST create a new database instance before the 90 day period ends.

__Set up calendar reminders for yourself to reset your Render Postgres database
instance every 85 days so your application(s) will not experience any
downtime.__

Each time you get your calendar reminder, follow the steps below.

1. Navigate to your Render [Dashboard], click on your database instance, and
   click on either the "Delete Database" or "Suspend Database" button.

2. Next, follow the instructions in Phase #3 above to create a new database
   instance.

3. Finally, you will need to update the environment variables for EVERY
   application that was connected to the original database with the new database
   information. For each application:

  - Click on the application name from your [Dashboard]
  - Click on "Environment" in the left sidebar
  - Replace the value for `DATABASE_URL` with the new value from your new database
    instance, and then click "Save Changes"
  - At the top of the page, click "Manual Deploy", and choose "Clear build cache
    & deploy".

4. After each application is updated with the new database instance and
   re-deployed, manually test each application to make sure everything still
   works and is appropriately seeded.

## Troubleshooting Tips and Tools

Render offers a straightforward deployment process, but you may run into
difficulties. Use the tips and tools below to troubleshoot your deployment.

### Database Issues

Since creating a database is a simple process with the Render GUI, most issues
related to the database are not caused by the database creation phase. Instead,
they may be caused by errors in configuring your application to run in the
production environment (errors in the project repo), or errors in connecting
your application to the Render Postgres database instance (errors setting up the
Web Service in the Render GUI).

Check the following to make sure your project is properly set up to run sqlite
in development, and Postgres in production, and that your database connection is
set up properly:

- Environment variables: Did you include the `FLASK_ENV` variable set to
  "production" and the `DATABASE_URL` key set to the "Internal Database URL"
  value from your Render Postgres database instance?

- Deployment logs: Did each command in your build script and start script run as
  expected?

- Deployment Logs: Did the migration files and seeder files run as expected?

### Checking the Postgres Database (Advanced Troubleshooting)

If you've checked all of the issues described above, you can further
troubleshoot your deployment by examining the contents of your Postgres
database. In order to do this, you must have [PostgreSQL] installed locally on
your computer.

To access your Render Postgres database, copy the PSQL Command value from the
information page of your database. The value should start with "PGPASSWORD=",
and should include information about your database.

Paste this value into your terminal. This will open up Postgres with a
connection to your remote database. At this point, you can use Postgres commands
locally to examine the contents of your database. Try the following:

- `\dn` - lists all of the schemas in the database
  - Does your database show the correct schema for your project?

- `\dt <schema-name>.*` - lists all tables within `<schema-name>` schema
  - Do you see all of your tables within the schema? You should see the `users`
    table, as well as the `alembic_version` tables at this point.

- `SELECT * FROM <schema-name>.users;` - lists all entries in the `users`
  table within `<schema-name>` schema
 - Does the `users` table show the appropriate seed data?

If there are any problems with the way the database or schema is set up, you can
drop the schema for your application and all tables within it, using the
following command:

```sql
DROP SCHEMA <schema-name> CASCADE
```

## Re-deployment

There are two ways to re-deploy your `main` branch changes:

1. If you set up your application for Auto-Deployment, it should automatically
   re-deploy after every push to main.

2. You can manually trigger a re-deployment at any time. Click on the blue
   "Manual Deploy" button, and choose "Clear Build Cache & Deploy". You will be
   able to see the logs and confirm that your re-deployment is successful.

## Wrapping Up

Congratulations, you've created a production-ready, dynamic, full-stack website
that can be securely accessed anywhere in the world! Give yourself a pat on the
back. You're a web developer!

[Render homepage]: https://render.com/
[Dashboard]: https://dashboard.render.com/
[PostgreSQL]: https://www.postgresql.org/

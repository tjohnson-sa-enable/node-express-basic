# Using Codeship Basic to Test and Deploy Node Express Applications
Codeship offers developers a vast array possibilities when creating a Continuous Integration and Deployment pipeline for their applications. I want to focus today on how to build a solid Continuous Integration and Deployment (CI / CD) Pipeline for any kind of Javascript-based application with Codeship.

## Setting Up Your Local Environment
We’ll kick things off by configuring our local application. Once downloaded, we can test out the Docker build process by running:

* A local installation of [Docker Community Edition](https://www.docker.com/community-edition)
* A local [download](https://github.com/hiimtaylorjones/nodejs-express-todoapp) of our sample  Node Express To-Do App
* A Codeship Basic account - [Sign Up](https://signup.heroku.com/)
* A Heroku Account - [Sign Up](https://app.codeship.com/registrations/new?utm_source=NavBar)

With our application downloaded locally, we can test out the build process by running:

```
docker-compose build .
```

This command will tell Docker to pull in all our dependencies and build our container ecosystem.  Once the container build is finished, we’ll now be able to run commands from within the container. More specifically, a successful container build allows us to run our test suite by using the following command:

```
docker-compose -p tests run -p 3000 --rm web npm run test -- --forceExit
```

Our containers will then boot up, run our test suite, and exit. All of the tests should pass with flying colors. With a passing test suite in hand, we’ll now be able to move on to translating our setup for Codeship.

## Setting Up Your Codeship Basic Project
With a local instance of our Dockerized application set up, we now can start building our Codeship build and deployment process.

First, we’ll log into Codeship, navigate to the **Projects** tab, and create a new project. We’ll then select our method of Source Control (Github, Bitbucket, Gitlab) and point Codeship towards our application.

After our application connects, we’ll be presented with the option of making our project a **Codeship Pro** or **Codeship Basic** project. For this example, we’ll be running with Codeship Basic.

Next up is the formulation of  **Setup Commands**. Our setup for Ruby on Rails applications is formulated almost exactly like if were configuring our application locally without Docker.

```
psql -p 5432 -c 'create database todos;'
npm install
npm run migrate
```

With our **Setup Commands** formulated, we’ll move onto creating the **Test Pipeline**. With Codeship Basic, we only have one pipeline at our disposal. Each of these setups will run in sequence, instead of parallel. For our application, this won’t come too much into play since we only have one pipeline command needed:

```
npm test -- --forceExit
```

This command will run our NPM tests for our application on Codeship.

Finally, we’ll navigate over to the **Environment** tab and set two variables for our project.

```
DATABASE_URL=postgres://postgres@localhost/todos
NODE_ENV=test
```

Setting these variables give two important pieces of information to our Codeship application. One is the database url. We need to tell our application information about the database we’ve created. We also need to set the test environment of the application.

With our Codeship CI Pipeline in place, we can now save our setup steps and move on to testing the configuration out! We can trigger a new build, by committing new changes to your local project.

```
git add --all
git commit -m "my first codeship build"
git push origin master
```

This process should trigger our Codeship build pipeline and yield a successful build on Codeship.

##  Mixing In Heroku Deployment
With our passing Codeship build, we’ll now set up the Heroku Deployment step in our pipeline. In our Codeship Project page, we’ll select **Project Settings** and click on the **Deploy** tab.

To start things off, we’ll need to add our Codeship project’s SSH key to the application. We can find our project’s SSH key under the **General** tab in project settings. Create a `.pub` file locally and add it to Heroku via the command line:

```
heroku keys:add your_new_key_file.pub
```

This will allow Codeship to communicate and push to Heroku if our test suite passes.

Next, we’ll configure the deployment pipeline on Codeship’s side. To start this, we need to grab our Heroku API Key:

* Click on your avatar in the upper right, then click Account Settings.
* Near the bottom of the settings page, you will find an API key. Click Reveal.
* Copy the API key.

With the API Key copied, navigate to the **Deploy** tab under project settings. Find the **Heroku** tile and click on it. You’ll then be asked for your application name (as in what its named on Heroku) and your API Key. Fill out those fields and click “Create Deployment”.

To test out our new deployment step, make a quick code change to the `README` of the application and commit the results. This should trigger our pipeline that should follow this patterns:

* Download and configure your project on Codeship
* If configuration is successful, run tests on project
* If tests pass, deploy the application to Heroku


Once this process fully completes without any errors, you then will have a successful Codeship CI / CD Pipeline!



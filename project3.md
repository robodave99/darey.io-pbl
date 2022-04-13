# SIMPLE TO-DO APPLICATION ON MERN WEB STACK
In this project, you are tasked to implement a web solution based on MERN stack in AWS Cloud.

## STEP 1 – BACKEND CONFIGURATION
Update ubuntu
```javascript
sudo apt update
```
Lets get the location of Node.js software from Ubuntu repositories.
```javascript
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
```
Install Node.js on the server
```javascript
sudo apt-get install -y nodejs
```
Note: The command above installs both nodejs and npm. NPM is a package manager for Node like apt for Ubuntu, it is used to install Node modules & packages and to manage dependency conflicts.

Verify the node installation with the command below
```javascript
node -v 
npm -v 
```
Application Code Setup
Create a new directory for your To-Do project:
```javascript
mkdir Todo
```
Run the command below to verify that the Todo directory is created with ls command : ls
Now change your current directory to the newly created one: cd Todo

Next, you will use the command npm init to initialise your project, so that a new file named package.json will be created. This file will normally contain information about your application and the dependencies that it needs to run. Follow the prompts after running the command. You can press Enter several times to accept default values, then accept to write out the package.json file by typing yes.
```javascript
npm init
```
### Screenshot of Backend Configuration
![Screenshot (39)](https://user-images.githubusercontent.com/52970510/163277092-074569ea-c74f-43bd-bd30-04491fe6283b.png)

Run the command ls to confirm that you have package.json file created.

Next, we will Install ExpressJs and create the Routes directory.

## INSTALL EXPRESSJS
Remember that Express is a framework for Node.js, therefore a lot of things developers would have programmed is already taken care of out of the box. Therefore it simplifies development, and abstracts a lot of low level details. For example, Express helps to define routes of your application based on HTTP methods and URLs.

To use express, install it using npm:
```javascript
npm install express
```
### Screenshot of EXPRESSJS installation
![Screenshot (40)](https://user-images.githubusercontent.com/52970510/163280512-6783430d-affc-42f8-9c59-6d59f7e1ce21.png)

Now create a file index.js with the command below
```javascript
touch index.js
```
Run ls to confirm that your index.js file is successfully created

Install the dotenv module
```javascript
npm install dotenv
```
Open the index.js file with the command below
```javascript
vim index.js
```
Type the code below into it and save. Do not get overwhelmed by the code you see. For now, simply paste the code into the file.
```javascript
const express = require('express');
require('dotenv').config();

const app = express();

const port = process.env.PORT || 5000;

app.use((req, res, next) => {
res.header("Access-Control-Allow-Origin", "\*");
res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
next();
});

app.use((req, res, next) => {
res.send('Welcome to Express');
});

app.listen(port, () => {
console.log(`Server running on port ${port}`)
});
```
Notice that we have specified to use port 5000 in the code. This will be required later when we go on the browser.

Use :w to save in vim and use :qa to exit vim

Now it is time to start our server to see if it works. Open your terminal in the same directory as your index.js file and type:
```javascript
node index.js
```
If every thing goes well, you should see Server running on port 5000 in your terminal.
Now we need to open this port in EC2 Security Groups. Refer to Project 1 Step 1 – Installing the Nginx Web Server. There we created an inbound rule to open TCP port 80, you need to do the same for port 5000, like this:

Open up your browser and try to access your server’s Public IP or Public DNS name followed by port 5000:
```javascript
http://<PublicIP-or-PublicDNS>:5000
```

Routes
There are three actions that our To-Do application needs to be able to do:

Create a new task
Display list of all tasks
Delete a completed task
Each task will be associated with some particular endpoint and will use different standard HTTP request methods: POST, GET, DELETE.

For each task, we need to create routes that will define various endpoints that the To-do app will depend on. So let us create a folder routes
```javascript
mkdir routes
```
Change directory to routes folder : cd routes
Now, create a file api.js with the command : touch api.js
Open the file with the command : vim api.js
Copy below code in the file.
```javascript
const express = require ('express');
const router = express.Router();

router.get('/todos', (req, res, next) => {

});

router.post('/todos', (req, res, next) => {

});

router.delete('/todos/:id', (req, res, next) => {

})

module.exports = router;
```
Moving forward let create Models directory.

## MODELS
Now comes the interesting part, since the app is going to make use of Mongodb which is a NoSQL database, we need to create a model.

A model is at the heart of JavaScript based applications, and it is what makes it interactive.

We will also use models to define the database schema . This is important so that we will be able to define the fields stored in each Mongodb document. (Seems like a lot of information, but not to worry, everything will become clear to you over time. I promise!!!)

In essence, the Schema is a blueprint of how the database will be constructed, including other data fields that may not be required to be stored in the database. These are known as virtual properties

To create a Schema and a model, install mongoose which is a Node.js package that makes working with mongodb easier.

Change directory back Todo folder with cd .. and install Mongoose
```javascript
npm install mongoose
```
### Mongoose Installation Screenshot
![Screenshot (41)](https://user-images.githubusercontent.com/52970510/163281225-2e31ec5b-3ca5-486b-b162-2bd96618ecb4.png)
Create a new folder models : mkdir models
Change directory into the newly created ‘models’ folder with : cd models
Inside the models folder, create a file and name it todo.js : touch todo.js

Open the file created with vim todo.js then paste the code below in the file:

### Screenshot showing mongoose schema added
![Screenshot (42)](https://user-images.githubusercontent.com/52970510/163281561-1abf8694-0640-4940-be7a-3161efa1a325.png)

Now we need to update our routes from the file api.js in ‘routes’ directory to make use of the new model.

In Routes directory, open api.js with vim api.js, delete the code inside with :%d command and paste there code below into it then save and exit

### Screenshot showing update on the api.js folder
![Screenshot (43)](https://user-images.githubusercontent.com/52970510/163281787-8d228f52-0b8b-4243-96d1-f86256b94aea.png)

## MONGODB DATABASE
We need a database where we will store our data. For this we will make use of mLab. mLab provides MongoDB database as a service solution (DBaaS), so to make life easy, you will need to sign up for a shared clusters free account, which is ideal for our use case. Sign up here. Follow the sign up process, select AWS as the cloud provider, and choose a region near you.

In the index.js file, we specified process.env to access environment variables, but we have not yet created this file. So we need to do that now.

Create a file in your Todo directory and name it .env.
```javascript
touch .env
vi .env
```
Add the connection string to access the database in it, just as below:
```javascript
DB = 'mongodb+srv://<username>:<password>@<network-address>/<dbname>?retryWrites=true&w=majority'
```
Ensure to update username, password, network-address and database according to your setup

### Screenshot of Database setup
![Screenshot (44)](https://user-images.githubusercontent.com/52970510/163282537-5de7cefd-a865-49de-8209-196b5f0f42dd.png)
Now we need to update the index.js to reflect the use of .env so that Node.js can connect to the database.

Simply delete existing content in the file, and update it with the entire code below.

To do that using vim, follow below steps

Open the file with vim index.js
Press esc
Type :
Type %d
Hit ‘Enter’
The entire content will be deleted, then,

Press i to enter the insert mode in vim
Now, paste the entire code below in the file.
```javascript
const express = require('express');
const bodyParser = require('body-parser');
const mongoose = require('mongoose');
const routes = require('./routes/api');
const path = require('path');
require('dotenv').config();

const app = express();

const port = process.env.PORT || 5000;

//connect to the database
mongoose.connect(process.env.DB, { useNewUrlParser: true, useUnifiedTopology: true })
.then(() => console.log(`Database connected successfully`))
.catch(err => console.log(err));

//since mongoose promise is depreciated, we overide it with node's promise
mongoose.Promise = global.Promise;

app.use((req, res, next) => {
res.header("Access-Control-Allow-Origin", "\*");
res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
next();
});

app.use(bodyParser.json());

app.use('/api', routes);

app.use((err, req, res, next) => {
console.log(err);
next();
});

app.listen(port, () => {
console.log(`Server running on port ${port}`)
});
```
Using environment variables to store information is considered more secure and best practice to separate configuration and secret data from the application, instead of writing connection strings directly inside the index.js application file.

Start your server using the command:
```javascript
node index.js
```
### Screenshot - Configured the database and started the server on port 5000
![Screenshot (45)](https://user-images.githubusercontent.com/52970510/163283002-f6ed8922-6bc8-490f-bf76-12636baa856d.png)
You shall see a message ‘Database connected successfully’, if so – we have our backend configured. Now we are going to test it.

Testing Backend Code without Frontend using RESTful API

So far we have written backend part of our To-Do application, and configured a database, but we do not have a frontend UI yet. We need ReactJS code to achieve that. But during development, we will need a way to test our code using RESTfulL API. Therefore, we will need to make use of some API development client to test our code.

In this project, we will use Postman to test our API.

### Screenshot - Testing the create endpoint with postman
![Screenshot (46)](https://user-images.githubusercontent.com/52970510/163283253-5dbabf82-b284-484e-845a-b4feec8f3e1e.png)
We have successfully created our Backend, now let go create the Frontend.

## STEP 2 – FRONTEND CREATION

In the same root directory as your backend code, which is the Todo directory, run:
```javascript
 npx create-react-app client
```
This will create a new folder in your Todo directory called client, where you will add all the react code.

Running a React App

Before testing the react app, there are some dependencies that need to be installed.

1. Install concurrently. It is used to run more than one command simultaneously from the same terminal window.
```javascript
npm install concurrently --save-dev
```
2. Install nodemon. It is used to run and monitor the server. If there is any change in the server code, nodemon will restart it automatically and load the new changes.
```javascript
npm install nodemon --save-dev
```
### Screenshot of dependency installation
![Screenshot (47)](https://user-images.githubusercontent.com/52970510/163283855-7332e7f7-ae34-4a83-95f2-9052fc33b5cb.png)

3. In Todo folder open the package.json file. Change the highlighted part of the below screenshot and replace with the code below.
```javascript
"scripts": {
"start": "node index.js",
"start-watch": "nodemon index.js",
"dev": "concurrently \"npm run start-watch\" \"cd client && npm start\""
},
```
### Screenshot of package.json file update configuration
![Screenshot (50)](https://user-images.githubusercontent.com/52970510/163284126-0334dad2-af0d-43ff-908e-e30f5f21bf3b.png)

1. Change directory to ‘client’ : cd client
2. Open the package.json file : vi package.json
3. Add the key value pair in the package.json file "proxy": "http://localhost:5000"

Now, ensure you are inside the Todo directory, and simply do:
```javascript
npm run dev
```
Your app should open and start running on localhost:3000

Important note: In order to be able to access the application from the Internet you have to open TCP port 3000 on EC2 by adding a new Security Group rule. You already know how to do it.

Creating your React Components

One of the advantages of react is that it makes use of components, which are reusable and also makes code modular. For our Todo app, there will be two stateful components and one stateless component.

### Screenshot of the backend and frontend application running
![Screenshot (51)](https://user-images.githubusercontent.com/52970510/163284847-a202ec3d-2d7c-4d21-882c-005850b9e506.png)

Created some component updated input.js and installed axios, a library to make HTTP requests.

### Screenshot of the process of creating the front-end file and folders for the react web application
![Screenshot (52)](https://user-images.githubusercontent.com/52970510/163285076-c9dc976c-4247-4c5f-91a2-898daac85142.png)

Created and updated Todo.js, App.js, App.css, ListTodo.js, Input.js

### Screenshot showing the application running in development
![Screenshot (54)](https://user-images.githubusercontent.com/52970510/163285222-e1be7b7c-0041-4969-9ed7-285b8b40a6cb.png)

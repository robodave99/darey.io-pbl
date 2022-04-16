# MEAN STACK DEPLOYMENT TO UBUNTU IN AWS
MEAN Stack is a combination of following components:
1. MongoDB (Document database) – Stores and allows to retrieve data.
2. Express (Back-end application framework) – Makes requests to Database for Reads and Writes.
3. Angular (Front-end application framework) – Handles Client and Server Requests
4. Node.js (JavaScript runtime environment) – Accepts requests and displays results to end user

In this assignment you are going to implement a simple Book Register web form using MEAN stack.
## Step 1: Install NodeJs
Node.js is a JavaScript runtime built on Chrome’s V8 JavaScript engine. Node.js is used in this tutorial to set up the Express routes and AngularJS controllers.

Update ubuntu
```javascript
sudo apt update
```
Upgrade ubuntu
```javascript
sudo apt upgrade
```
Add certificates
```javascript
sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates

curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
```
Install NodeJS
```javascript
sudo apt install -y nodejs
```

### Screenshot of nodejs installation
![Screenshot (59)](https://user-images.githubusercontent.com/52970510/163685391-807f09d1-c3c9-456d-a49d-6e44330dfb3d.png)

## Step 2: Install MongoDB
MongoDB stores data in flexible, JSON-like documents. Fields in a database can vary from document to document and data structure can be changed over time. For our example application, we are adding book records to MongoDB that contain book name, isbn number, author, and number of pages.
mages/WebConsole.gif
```javascript
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6
```
```javascript
echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list
```
Install MongoDB
```javascript
sudo apt install -y mongodb
```
### Screenshot of mongodb installation
![Screenshot (60)](https://user-images.githubusercontent.com/52970510/163685529-a520bebf-800a-4c54-90c7-7c43a1af8d87.png)

Start The server
```javascript
sudo service mongodb start
```
Verify that the service is up and running
```javascript
sudo systemctl status mongodb
```
### Screenshot of mongodb server status
![Screenshot (61)](https://user-images.githubusercontent.com/52970510/163685604-d5ffb327-95b2-4462-ac61-0c94a955c8c5.png)

Install npm – Node package manager
```javascript
sudo apt install -y npm
```
Install body-parser package
We need ‘body-parser’ package to help us process JSON files passed in requests to the server.
```javascript
sudo npm install body-parser
```
### Screenshot of body-parser package
![Screenshot (63)](https://user-images.githubusercontent.com/52970510/163685704-587ac528-81a2-4faa-89a4-5578d74dd730.png)

Create a folder named ‘Books’
```javascript
mkdir Books && cd Books
```
In the Books directory, Initialize npm project
```javascript
npm init
```
Add a file to it named server.js
```javascript
vi server.js
```
Copy and paste the web server code below into the server.js file.
```javascript
var express = require('express');
var bodyParser = require('body-parser');
var app = express();
app.use(express.static(__dirname + '/public'));
app.use(bodyParser.json());
require('./apps/routes')(app);
app.set('port', 3300);
app.listen(app.get('port'), function() {
    console.log('Server up: http://localhost:' + app.get('port'));
});
```
### Screenshot showing updated server.js file
![Screenshot (64)](https://user-images.githubusercontent.com/52970510/163685826-5c9036ba-cd23-4059-a683-9d1f726c736b.png)

## Step 3: Install Express and set up routes to the server
Express is a minimal and flexible Node.js web application framework that provides features for web and mobile applications. We will use Express in to pass book information to and from our MongoDB database.

We also will use Mongoose package which provides a straight-forward, schema-based solution to model your application data. We will use Mongoose to establish a schema for the database to store data of our book register.
```javascript
sudo npm install express mongoose
```
### Screenshot of Express and Mongoose package
![Screenshot (65)](https://user-images.githubusercontent.com/52970510/163686164-31db8bdc-d54d-4ed3-9467-682c9f3d2dbf.png)

In ‘Books’ folder, create a folder named apps
```javascript
mkdir apps && cd apps
```
Create a file named routes.js
```javascript
vi routes.js
```
Copy and paste the code below into routes.js
```javascript
var Book = require('./models/book');
module.exports = function(app) {
  app.get('/book', function(req, res) {
    Book.find({}, function(err, result) {
      if ( err ) throw err;
      res.json(result);
    });
  }); 
  app.post('/book', function(req, res) {
    var book = new Book( {
      name:req.body.name,
      isbn:req.body.isbn,
      author:req.body.author,
      pages:req.body.pages
    });
    book.save(function(err, result) {
      if ( err ) throw err;
      res.json( {
        message:"Successfully added book",
        book:result
      });
    });
  });
  app.delete("/book/:isbn", function(req, res) {
    Book.findOneAndRemove(req.query, function(err, result) {
      if ( err ) throw err;
      res.json( {
        message: "Successfully deleted the book",
        book: result
      });
    });
  });
  var path = require('path');
  app.get('*', function(req, res) {
    res.sendfile(path.join(__dirname + '/public', 'index.html'));
  });
};
```
### Screenshot of routes.js file update
![Screenshot (66)](https://user-images.githubusercontent.com/52970510/163686234-366c6b9e-1f77-4f32-88fe-5bca8350da81.png)

In the ‘apps’ folder, create a folder named models
```javascript
mkdir models && cd models
```
Create a file named book.js
```javascript
vi book.js
```
Copy and paste the code below into ‘book.js’
```javascript
var mongoose = require('mongoose');
var dbHost = 'mongodb://localhost:27017/test';
mongoose.connect(dbHost);
mongoose.connection;
mongoose.set('debug', true);
var bookSchema = mongoose.Schema( {
  name: String,
  isbn: {type: String, index: true},
  author: String,
  pages: Number
});
var Book = mongoose.model('Book', bookSchema);
module.exports = mongoose.model('Book', bookSchema);
```
### Screenshot of book.js file update
![Screenshot (67)](https://user-images.githubusercontent.com/52970510/163686314-30e06e33-5a29-48e4-ae31-bcc9ea72f08d.png)

## Step 4 – Access the routes with AngularJS
AngularJS provides a web framework for creating dynamic views in your web applications. In this tutorial, we use AngularJS to connect our web page with Express and perform actions on our book register.

Change the directory back to ‘Books
```javascript
cd ../..
```
Create a folder named public
```javascript
mkdir public && cd public
```
Add a file named script.js
```javascript
vi script.js
```
Copy and paste the Code below (controller configuration defined) into the script.js file.
```javascript
var app = angular.module('myApp', []);
app.controller('myCtrl', function($scope, $http) {
  $http( {
    method: 'GET',
    url: '/book'
  }).then(function successCallback(response) {
    $scope.books = response.data;
  }, function errorCallback(response) {
    console.log('Error: ' + response);
  });
  $scope.del_book = function(book) {
    $http( {
      method: 'DELETE',
      url: '/book/:isbn',
      params: {'isbn': book.isbn}
    }).then(function successCallback(response) {
      console.log(response);
    }, function errorCallback(response) {
      console.log('Error: ' + response);
    });
  };
  $scope.add_book = function() {
    var body = '{ "name": "' + $scope.Name + 
    '", "isbn": "' + $scope.Isbn +
    '", "author": "' + $scope.Author + 
    '", "pages": "' + $scope.Pages + '" }';
    $http({
      method: 'POST',
      url: '/book',
      data: body
    }).then(function successCallback(response) {
      console.log(response);
    }, function errorCallback(response) {
      console.log('Error: ' + response);
    });
  };
});
```
### Screenshot of updated script.js file
![Screenshot (68)](https://user-images.githubusercontent.com/52970510/163686441-49504883-face-4a7b-864f-f2fab011547a.png)

In public folder, create a file named index.html;
```javascript
vi index.html
```
Copy and paste the code below into index.html file.
```javascript
<!doctype html>
<html ng-app="myApp" ng-controller="myCtrl">
  <head>
    <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.6.4/angular.min.js"></script>
    <script src="script.js"></script>
  </head>
  <body>
    <div>
      <table>
        <tr>
          <td>Name:</td>
          <td><input type="text" ng-model="Name"></td>
        </tr>
        <tr>
          <td>Isbn:</td>
          <td><input type="text" ng-model="Isbn"></td>
        </tr>
        <tr>
          <td>Author:</td>
          <td><input type="text" ng-model="Author"></td>
        </tr>
        <tr>
          <td>Pages:</td>
          <td><input type="number" ng-model="Pages"></td>
        </tr>
      </table>
      <button ng-click="add_book()">Add</button>
    </div>
    <hr>
    <div>
      <table>
        <tr>
          <th>Name</th>
          <th>Isbn</th>
          <th>Author</th>
          <th>Pages</th>

        </tr>
        <tr ng-repeat="book in books">
          <td>{{book.name}}</td>
          <td>{{book.isbn}}</td>
          <td>{{book.author}}</td>
          <td>{{book.pages}}</td>

          <td><input type="button" value="Delete" data-ng-click="del_book(book)"></td>
        </tr>
      </table>
    </div>
  </body>
</html>
```
### Screenshot of updated index.html file
![Screenshot (69)](https://user-images.githubusercontent.com/52970510/163686520-aec96967-1f91-4197-a4e7-2e765bf05c24.png)

Change the directory back up to Books
```javascript
cd ..
```
Start the server by running this command:
```javascript
node server.js
```
### Screenshot showing server status
![Screenshot (70)](https://user-images.githubusercontent.com/52970510/163686574-2da544ca-f627-4a8d-8582-1ddf6bcde515.png)

The server is now up and running, we can connect it via port 3300. You can launch a separate Putty or SSH console to test what curl command returns locally.
```javascript
curl -s http://localhost:3300
```
It shall return an HTML page, it is hardly readable in the CLI, but we can also try and access it from the Internet.

For this – you need to open TCP port 3300 in your AWS Web Console for your EC2 Instance.

You are supposed to know how to do it, if you have forgotten – refer to Project 1 (Step 1 — Installing Apache and Updating the Firewall)
Now you can access our Book Register web application from the Internet with a browser using Public IP address or Public DNS name.

Quick reminder how to get your server’s Public IP and public DNS name:

1. You can find it in your AWS web console in EC2 details
2. Run curl -s http://169.254.169.254/latest/meta-data/public-ipv4 for Public IP address or curl -s http://169.254.169.254/latest/meta-data/public-hostname for Public DNS name.
This is how your Web Book Register Application will look like in browser:

### Screenshot of Web Book Register Application on your browser
![Screenshot (71)](https://user-images.githubusercontent.com/52970510/163686672-f96995e5-f486-44c0-aea6-81f4147a1553.png)



## Express

```js
// server.js
var express = require('express');
var app = express();
var bodyParser = require('body-parser');
var session = require('express-session');
var fs = require('fs');

app.set('views', __dirname + '/views');
app.set('view engine', 'ejs');
app.engine('html', require('ejs').renderFile);

var server = app.listen(3000, function() {
  console.log("Expressserver has started on port 3000");
});

app.use(express.static('public'));

app.use(bodyParser.json());
app.use(bodyParser.urlencoded());
app.use(session({
  secret: '@#$MYSIGN#@$#$',
  resave: false,
  saveUninitialized: true
}));

var router = require('./router/main')(app, fs);
```

```html
<!-- ejs -->
<% JAVASCRIPT %>
<%= VARIABLE %>
```

```js
// router/main.js
module.exports = function(app, fs)
{
  app.get('/', function(req, res) {
    res.render('index', {
      title: 'MY HOMEPAGE',
      length: 5
    });
  });

  app.get('/list', function (req, res) {
    fs.readFile( __dirname + "/../data/" + "user.json", 'utf8', function (err, data) {
      console.log(data);
      res.end(data);
    });
  });

  app.get('/getUser/:username', function (req,res) {
    fs.readFile( __dirname + "/../data/user.json", 'utf8', function (err, data) {
      var users = JSON.parse(data);
      res.json(users[req.params.username]);
    });
  });

  app.post('/addUser/:username', function(req, res) {
    var result = {};
    var username = req.params.username;

    // CHECK REQ VALIDITY
    if(!req.body["password"] || !req.body["name"]) {
      result["success"] = 0;
      result["error"] = "invalid request";
      res.json(result);
      return;
    }

    // LOAD DATA & CHECK DUPLICATION
    fs.readFile( __dirname + "/../data/user.json", 'utf8', function(err, data) {
      var users = JSON.parse(data);
      if(users[username]) {
        // DUPLICATION FOUND
	result["success"] = 0;
	result["error"] = "duplicate";
	res.json(result);
	return;
      }

      // ADD TO DATA
      users[username] = req.body;

      fs.writeFile( __dirname + "/../data/user.json", JSON.stringify(users, null, '\t'), 'utf8', function (err, data) {
        result = {"success": 1};
	res.json(result);
      });
    });
  });

  app.put('/updateUser/:username', function(req, res) {
    var result = {};
    var username = req.params.username;

    // CHECK REQ VALIDITY
    if(!req.body["password"] || !req.body["name"]) {
      result["success"] = 0;
      result["error"] = "invalid request";
      res.json(result);
      return;
    }

    // LOAD DATA
    fs.readFile( __dirname + "/../data/user.json", 'utf8', function (err, data) {
      var users = JSON.parse(data);
      if(!users[username]) {
        // NOT FOUND
        result["success"] = 0;
        result["error"] = "Not found";
        res.json(result);
        return;
      }

      // UPDATE DATA
      users[username] = req.body;

      fs.writeFile( __dirname + "/../data/user.json", JSON.stringify(users, null, '\t'), 'utf8', function (err, data) {
        result = {"success": 1};
	res.json(result);
      });
    });
  });

  app.delete('/deleteUser/:username', function(req, res) {
    var result = {};
    fs.readFile( __dirname + "/../data/user.json", 'utf8', function (err, data) {
      var users = JSON.parse(data);

      // NOT FOUND
      if(!users[req.params.username]) {
        result["success"] = 0;
	result["error"] = "not found";
	res.json(result);
	return;
      }

      delete users[req.params.username]; //동적 해제?
      fs.writeFile( __dirname + "/../data/user.json", JSON.stringify(users, null, '\t'), 'utf8', function (err, data) {
        result["success"] = 1;
	res.json(result);
	return;
      });
    });
  });
}
```

참조 : https://velopert.com

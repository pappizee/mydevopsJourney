# LEMP STACK IMPLEMENTATION

This project covers implementing a web solution that is based on a MERN Stack in AWS cloud, Mern as a term means 
**MongoDB** This is a document based database that is used to store application data in form of documents
**ExpressJS** This is known as a server side web application framework for Node.js 
**ReactJS** This is a framework that is developed by facebook solely for frontend (UI) and it is based on javascript
**Node.js** a JRE (javascript run time environment) which is used to run/compile js on a machine rather than in a browser

# Personal Notes :
 This project has exposed me to few new technologies and understanding what they do, For instance knowing the difference between a relational database management systems(DBMS) and NoSQL, I also have an insight on the kinds of Web App frameworks we have and how a typical client side which is a frontend and a server side a backend frameworks are and how they relate and transfer data to each other

### Configuring my back End
Pre Conditions : Spin up an EC2 instance on AWS and connect to the Linux terminal
 Foremost, I use the `sudo apt update` to update my ubuntu and `sudo apt upgrade` to upgrade , Using curl or wget, I could install nodes on my machine from the ubuntu repositories like 
 ```` curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash - ````
 Now i installed nodes.js on the server using the command `sudo apt-get install -y nodejs` flag y in this instance automatically selects YES when the installation is initiated so you wont be required to press y for installation to proceed. Having installed nodes and npm with the previous action, i need to validate that it has successfully been downloaded on the server by doing `node -v` and `npm -v`

 To kick start the project, I created a new directory where i want to keep the keep all the properties relating to the app by using the below command in the following order :
 `mkdir Todo` then validate directory is created by doing `ls` , For us to start to load up the created directory, I navigated into the directory using the cd command `cd Todo` then i had to initialize the project using the npm init command, This created a package.json file within that project . In a typical package.json file, You have details about the application, if it has all the dependencies and their versions 


### Express JS installation
 After the first step of setting up the project, I need to install the framework my nodes needs to run on which is the expressJS so i used the below commands to complete that action as npm has been installed on the server now its the package manager for nodes and i used it for my installations :
 `npm install express` 
then i created a JS file named index `touch index.js` and as usual i verified a file has been created with the ls command, Next is i installed dotenv module `npm install dotenv` and using the vim or Vi or nano command, I opened the js file `vim index.js` then i pasted the code below then saved the action with the esc+shift+:+wq!:
````
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

````
to validate if server kicks off, Within the todo directory, i did `node index.js` and i see it running on the port set within the pasted and saved code above, I headed back to aws and under the inbound rule of my spinned up server, I opened tCP port 90 to open port range as 5000 then i opened up my browser to try and access my server's public IP `http://<PublicIP-or-PublicDNS>:5000`
**Creating my routes**
Creating a routes folder which defines the various endpoints that the to do app will depend on to complete a request as per the user's input
`mkdir routes` then navigate to routes you just created doing `cd routes` which i then created an api.js file within 
`touch api.js` , Open the created file and paste the below then saved it : `vim api.js`
````const express = require ('express');
const router = express.Router();

router.get('/todos', (req, res, next) => {

});

router.post('/todos', (req, res, next) => {

});

router.delete('/todos/:id', (req, res, next) => {

})

module.exports = router;````

 Now i had to navigate to the root project which is `todo` by doing `cd..`
 now i need to install moongoose using `npm install mongoose` and then create a models folder:
 `mkdir models` now navigate into the models directory `cd models` and add the below in a todo.js file that i created 
 `touch todo.js`

 `mkdir models && cd models && touch todo.js` or to install multiple actions we can use :
 ```` mkdir models && cd models && touch todo.js````
 Open the folder and paste the code :
 ````const mongoose = require('mongoose');
const Schema = mongoose.Schema;

//create schema for todo
const TodoSchema = new Schema({
action: {
type: String,
required: [true, 'The todo text field is required']
}
})

//create model for todo
const Todo = mongoose.model('todo', TodoSchema);

module.exports = Todo;````

Having creating a todo.js file i had to update my routes from the api.js within the routes directory so as to align with the new model, Cd into the route directory and opened api.js `vim api.js` , I had to scrap the code that existed within it using `:%d and pasted the new code in there
````
const express = require ('express');
const router = express.Router();
const Todo = require('../models/todo');

router.get('/todos', (req, res, next) => {

//this will return all the data, exposing only the id and action field to the client
Todo.find({}, 'action')
.then(data => res.json(data))
.catch(next)
});

router.post('/todos', (req, res, next) => {
if(req.body.action){
Todo.create(req.body)
.then(data => res.json(data))
.catch(next)
}else {
res.json({
error: "The input field is empty"
})
}
});

router.delete('/todos/:id', (req, res, next) => {
Todo.findOneAndDelete({"_id": req.params.id})
.then(data => res.json(data))
.catch(next)
})

module.exports = router;
````

###Hooking my project to a MongoDB database

 To store data that is been entered from the frontend, a db would be required so i will be making using of mLab for this piece, mLab as a tool provides MongoDB database as a service solution (DBaaS),and for me to be able to get the full benefit of this feature i had to sign up for a shared clusters free account, which is what is needed for what i need to do. 


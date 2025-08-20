# Getting to know NodeJS and MongoDB

Heres somethings this document will cover:
<ul>
    <li>Creating and using Node.js scripts</li>
    <li>Using Docker</li>
    <li>Using MongoDB</li>
    <li>Using Node.js to access MongoDB</li>
</ul>

### Node.js
This will help us write the backend logic in javascript. Note that Node.js is built on V8 ( a javascript engine used by chrome based browsers ). the envrionment is built on top of the V8 engine. In Node.js, theres modules provided to us to have an interface with the operating system for things like network requests and creating files. The same modules allow us to create backend services using Node.js. Both Node.js and browser level javascript runs on a javascript engine. 

In the ```BasicsofNode``` <a src="nodejsbasics/src/Backend/files.js" >project</a>, we have created a file under the Backend folder to showcase how node js handles writing to files via writeFileSync.

<details>
<summary>using writeFile Sync</summary>

```files.js```
```.js
// using Node.js to handle files on our local machine via: node.fs (filesystem) module.
// we can use this functionality to read and write files  or even use files as a simple database.


// this imports two functions needed for our purpose
import { writeFileSync, readFileSync } from 'node:fs';

// create an array of mock data
const users = [{name: 'Chakane Shegog', email: 'chakanezshegog@gmail.com'}];

// convert mock data to a string
const usersJson = JSON.stringify(users);

// save the JSON string to a file via writeFileSync
// the function takes 2 arguments: (1) the filename, (2) the string to be written in the file
writeFileSync('Backend/users.json', usersJson);

// after writing to the file we will read from it
const readUsersJson = readFileSync('Backend/users.json');
const readUsers = JSON.parse(readUsersJson);

// log the parsed array
console.log(readUsers);
```
</details>

#### Concurrency with JS in the browser and Node.js

Javascript has functions which are mostly asynchronous. This is because it is mostly event based. On the server side things such as networks requests are always asynchronous. 

If* the code was synchronous, it would be directly executed on the call stack, But if it is asynchronous; the operation would be started and thrown into a queue within a callback function. Node.js runtime executes all code left in the stack. then a ```event loop`` check if theres any completed tasks in the queue; if so, the callback function is executed by putting it on the stack. 

For example: when we use the event listener ```onClick```, when the user clicks on the button, the callback will be placed in the task queue. Using this logic, we can also say that in the backend (Node.js) we can add an event listener for net work requests and execute a callback when its completed. 

### Creating a WebServer with Node.js
 This is an example a very simple web server:

<details>
<summary>web server code example</summary>

```.js
import { createServer } from 'node:http';
const server = createServer((req, res) => {
    
    res.statusCode = 200;
    res.setHeader('Content-Type', 'text/plain');
    res.end('Hello HTTP world!');
});

const host = 'localhost';
const port = 3000;

server.listen(port, host, () => {
    console.log(`Server listening on http://${host}:${port}`);
});

```
</details>

A webserver is software which serves web content to a client using HTTP/HTTPS protocols. They differ from regular servers in its focus on web content. Other servers may use other protocols like: SSH, FTP, SMTP, etc and focus on storing large datasets and talking to other servers.

#### Extending the web server to serve a JSON file
Reference webfiles.js.


### Introducing Docker (a platform for containers)
Docker allows us to package, manage, and run applications in "loosely" isolated environments called containers.

Containers are meant to contain all dependencies for an application. With these, we can setup various environments for our applications and services. 

#### The Docker Platform
Docker is mainly 3 parts:
1. Docker Client: runs commands
2. Docker Host: contains Docker dawmon, images, and containers
3. Docker Registry: Hosts and stores docker images, extensions, plugins.

Docker images are templates used to create containers. 
Docker containers are instances of those images. 

Know how to start and stop an instance. 


### Introducing MongoDB
This is a NoSQL document based database meaning that each entry is stored as a document. Documents are pretty much json objects. MongoDB is also based on the javascript engine. ee

#### Setting up a MongoDB server
Because we have Docker installed we can run MongoDB in a Docker container. 

1. Execute ```docker ps``` in terminal to check docket is good. 
2. make sure a container is already started and then hit this command to create a server ```docker run -d --name dbserver -p 27017:27017 --restart unless-stopped mongo:6.0.4```
3. Make sure the MongoDB Shell is installed in the host system
4. Execute this command to connect to the server: ```mongosh mongodb://localhost:27017/ch2```

The last step will open up a shell for the database, we can run javascript code here if we want to. 

#### Running commands on the database
This would be useful for debugging or doing maintenence tasks on the database. 

###### Creating a collection and inserting and listing documents
Collections are equivalent to tables in relational databases. They can be seen as a large json array that contains json objects

Heres an example of some input and output of the shell cmd line for mongodb

```
ch2> db.users.insertOne({ username: 'chakane', fullName: 'chakane shegog', age: 29 })
{
  acknowledged: true,
  insertedId: ObjectId('67883a9e7209d5506a9dafd1')
}
ch2> db.users.find()
[
  {
    _id: ObjectId('67883a9e7209d5506a9dafd1'),
    username: 'chakane',
    fullName: 'chakane shegog',
    age: 29
  }
]
```

Lets add 2 more objects:

```
db.users.insertMany([
    { username: 'jane', fullName: 'Jane Doe', age: 32 },
    { username: 'john', fullName: 'John Doe', age: 22 }
])
```

We can then query for a certain username using ```findOne(arg1)``` which returns the first matching object. ** We can alternatively use ```find(arg1)``` to find all matches. Also we can use a different property aside from username.

```
db.users.findOne({ username: 'jane' })
```

MongoDB provides operators prefixed by $ for certain operations. For example we can find anyone above the age of 30 in our DB by using the ```$gt``` operator.

```
db.users.find({ age: { $gt: 30 } } )
```

We can also sort our results using ```.sort()``` after the .```find()``` method. For example: * The 1 value for age means to have it in ascending order. -1 is for descending.

```
db.users.find().sort({ age: 1 })
```

###### Updating documents
To update an existing entry in our DB we can use something like ```updateOne(arg1, arg2, optionalArg3)``` which updates only one record. 

* For the optional argument ```upsert``` the default value is false. If true, this will cause a new record to be created with the details of the update function. 
* If we want to remove a field we can use ```unset``` or just use the ```replaceOne``` method.

```
db.users.updateOne( {username:'dan'}, {$set:{age:27}}, {upsert: true})
```

###### Deleting documents
To delete a record we can use ```deleteOne(arg1)``` or ```deleteMany(arg1)``` Heres an example of the first:

```
db.users.deleteOne({username:"new"})
```

##### Accessing the MongoDB database via Node.js
Now with all the information above we should be able to create a new web server that returns a list of users from our users collection. 

The first thing we need to do is install the mongodb package in your project
```npm install mongodb@6.3.0```

Then we create a new Backend/mongodbweb.js file and write this code:

```
import { createServer } from 'node:http'
import { MongoClient } from 'mongodb'

// define a connection to the DB
const url = 'mongodb://localhost:27017'
const dbName = 'ch2'
const client = new MongoClient(url)

// connect to the DB
try {
    await client.connect()
    console.log("successfully connected to the client")
} catch (err) {
    console.error("Error connecting to the DB: ", err)
}

// create an HTTP server
const server = createServer(async (requestAnimationFrame, res) => {

    // select the db and extract data
    const db = client.db(dbName)
    const users = db.collection('users')
    const usersList = await users.find().toArray()
    res.statusCode = 200
    res.setHeader('Content-Type', 'application/json')
    res.end(JSON.stringify(usersList))
})

const host = 'localhost';
const port = 3000;

server.listen(port, host, () => {
    console.log(`Server listening on http://${host}:${port}`);
});
```
# Building and Deploying a full stack app with a Rest API

This section will cover 3 chapters
* Implementing a backend using Express, Mongoose ODM, and Jest
* Integrating a frontend using React and TanStack query
* Deploying the application with Docker and CI/CD


## Implementing the backend with: Express, Mongoose ODM, and Jest
With what we have leared about MongoDB and Node.js so far, we should be able to now build a backend service using Express to provide a REST API. Mongoose ```object data modeling``` (ODM) will be an interface for our MongoDB and Jes will be used to test our code.

This section will show us how to: 
* Design a backend service
* Create a DB schema using Mongoose
* Develop and test service functions
* Provide REST API using express


### Designing a backend service
<details>
    <summary>open</summary>



#### Creating the folder structure for our backend service
Within the /src folder have this setup
Folder: db
Folder: services
Folder: routes
Folder: backend

Our first application will be a blog application. The API should do the following:
1. Get a list of posts
2. Get a single post
3. Create a new post
4. Update an existing post
5. Delete an existing post

To provide these function we need to use a db schema to define what a blog post object should look like. Then we need service functions to handle CRUD functionality. Then use our REST API to query, create, update, and delete the blog posts. 


#### Creating db schemas using Mongoose
Mongoose is a library that simplifies MongoDB object modeling by reducing boilerplate code. 

First install the Mongoose library:
```npm install mongoose@8.0.2```

Next, create a new /src/db/init.js file and import mongoose
```import mongoose from 'mongoose'```

Inside that file, define and export a function to initialize a database connection
```export function initDatabase() {}```

Now define the ```DATABASE_URL``` to point to our MongoDB instance running via Docker and specify blog as the database name
```const DATABASE_URL = 'mongodb://localhost:27017/blog'```.

Then, add a listener to the open event on the Mongoose connection so that we can show a log for successful connection. Then use ```mongoose.connect(arg1)``` to actually connect.

```
mongoose.connection.on('open', () => {
    console.info('connection to db successful: ', DATABASE_URL);
})
const connection = mongoose.connect(DATABASE_URL);
return connection;
```

Now create a /src/example.js so we can see our Mongoose connect to our db. Just import the function we created above and execute it.

```
import { initDatabase } from './db/init.js';
initDatabase();
```

Now we have a db for our site that is controlled via Node.js.

#### Defining a model for blog posts
Now we need to define the data structure for blog posts. Blog posts should have:
* Title
* Author
* Contents
* Other tags associated with the post

1. Create a new /src/db/models folder
2. In that new folder create a new /src/db/models/post.js

```
// import mongoose and schema class
import mongoose, { Schema } from 'mongoose';

// define a new schema for posts
const postSchema = new Schema({
    // list all properties of a blog post and their types
    title: { type: String, required: true },
    author: String,
    contents: String,
    tags: [String],
})

// now that we have a schema, we can create a Mongoose model from it by using the mongoose.model() function
// mongoose.model() specifies the name of the collection. 
export const Post = mongoose.model('post', postSchema);
```
Now we can start using the model to create and query posts.

#### Using the blog post model
Lets first access the model inside example.js only because we have no defined any service functions or routes yet. 

Heres an example of how to achieve this (by modifying the example.js file): 

```
import { initDatabase } from "./db/init.js";
import { Post } from './db/models/post.js';

// this function is an async function so we need to await it; without the await we would be trying to access the db before were connected to it
await initDatabase();


// create a new blog post by calling a new ```Post()``` object.
const post = new Post({
    title: 'Hello, world',
    author: 'Chakane Shegog',
    contents: 'This post is stored in a MongoDB database using Mongoose library.',
    tags: ['mongoose', 'mongodb'],
});

// call .save() in the blog post to save it to the database
await post.save();

// now use the .find() function to list all posts and log the result
const posts = await Post.find();
console.log(posts);

```
Now when we run this example.js file, we will see in our docker console that a new object was created and stored in our database. 

By now, we have seen how using Mongoose is just a wrapper for MongoDB. 

#### Defining creation and last update dates in the blog post
We need to utilize the ```{timestamps:true}``` property in each newly created post to add timestamps for our posts. So within the post.js file inside out /db/models/ folder we will need to add that new property to our post schema.

```
const postSchema = new Schema (
    {
        title: String,
        author: String,
        contents: String,
        tags: [String],
    },
    { timestamps: true },
)
```

#### Developing and testing service functions
Now we are at a point to where any code we write, we just test it by executing a node script. Now that were going to be writing service functions, we should use Jest to write actual tests for our code.

Jest is a test runner which will define and execute unit tests. The mongodb-memory-server library will let us create a new instance of the MongoDB database , storing our data only in memory so that we can run our tests on a fresh database instance. 

</details>

### Setting up the test environment
<details>
    <summary>open</summary>

<details>
<summary>how to setup your test environment</summary>
1. Install Jest  and mongo db memory server into your project
```npm install --save-dev jest@29.7.0```
```npm install --save-dev mongodb-memory-server@9.1.1```

1. Create a /src/tests folder to put our tests in
2. In the new tests folder, create a file called globalSetup.js.

```
import { MongoMemoryServer } from 'mongodb-memory-server';

// this function will be used to create a memory server for MongoDB
export default async function globalSetup() {

    const instance = await MongoMemoryServer.create({

        // we have to set the binary version here to the same version that weve installed for our docker container
        binary: {
            version: '6.0.4',
        }
    })

    global.__MONGOINSTANCE = instance;
    process.env.DATABASE_URL = instance.getUri();
}
```
 Then we need a globalTeardown.js to stop the mongoDB instance when were done with our tests

 ```
 export default async function globalTeardown() {
    await global.__MONGOINSTANCE.stop();
}
 ```

 Now we need a setupFileAfterEnv.js file to define a beforeAll and afterAll functions to initialize our db and disconnect respectively. 

 ```
 import mongoose from 'mongoose';
import { beforeAll, afterAll } from '@jest/globals';

import { initDatabase } from '../db/init.js';
beforeAll(async () => {
    await initDatabase();
})

afterAll(async () => {
    await mongoose.disconnect();
})
```

4. Now we will need to create a jest.config.json file in the root of our project

```
{
    "testEnvironment": "node",
    "globalSetup": "<rootDir>//src/tests/globalSetup.js",
    "globalTeardown": "<rootDir>/src/tests/globalTeardown.js",
    "setupFileAfterEnv": ["<rootDir>/src/tests/setupFileAfterEnv.js"]
}
```

* note that we are stating <rootDir>, as this is something jest will resolve on its own. 

Then finally in the package.json script of our project, we will need to add a test property to our script:

```
  "scripts": {
    "start": "react-scripts start",
    "test": "NODE_OPTIONS=--experimental-vm-modules jest",
    "eject": "react-scripts eject"
  },
```

Now we should be able to execute ```npm test``` and the logs will show that there no scripts to test.
</details>

#### Writing our 1st service function: createPost
This function will create a new post for us. We can then write tests for it by verifying the create function makes a new post with the required fields. 

1. Create a new /src/services/posts.js file

```
import { Post } from '../db/models/post.js';

// Using our Post model, we are creating a function that takes the required post fields, creates, and then returns a new Post. 
export async function createPost({ title, author, contents, tags }) {
    const post = new Post({ title, author, contents, tags });
    return await post.save();
};
```

#### Defining test cases for the createPost service function
Lets test our createPost function. 

1. Create a new folder under /src called ```__test__```
2. Then create a new ```posts.test.js``` file in our new folder. 
3. In the new folder write this code:

<details>
<summary>posts.test.js</summary>

```.js
import mongoose from 'mongoose';
import { describe, expect, test } from '@jest/globals';
import { createPost } from '../services/posts.js'
import { Post } from '../db/models/post.js';

// this function creates a new test, we can have multiple tests in here
describe('creating posts', () => {

    // this function is where we define our new test
    test('with all parameters should succeeed', async() => {
        const post = {
            title: 'Hello Mongoose!',
            author: 'Chakane Shegog',
            contents: 'this post is stored in MongoDB db using Mongoose',
            tags: ['mongoose', 'mongodb']
        }
        const createdPost = await createPost(post);
        expect(createdPost._id).toBeInstanceOf(mongoose.Types.ObjectId);
        const foundPost = await Post.findById(createdPost._id);
        expect(foundPost).toEqual(expect.objectContaining(post));
        expect(foundPost.createdAt).toBeInstanceOf(Date);
    });

    test('without title should fail', async () => {
        const post = {
            author: 'Chakane Shegog',
            contents: 'Post with no title',
            tags: ['Empty']
        };

        try {
            await createPost(post);
        } catch (err) {
            expect(err).toBeInstanceOf(mongoose.Error.ValidationError);
            expect(err.message).toContain('`title` is required');
        }
    });

    test('with minimal parameters should succeed', async () => {
        const post = {
            title: 'Only a title',
        };
        const createdPost = await createPost(post);
        expect(createdPost._id).toBeInstanceOf(mongoose.Types.ObjectId);
    });
})
```
</details>

### Defining a function to list posts. 
We will be creating an internal function to list al of our posts called ```listPosts```. This function will be able to query posts and define a sort order. This function will define these related functions: ```listAllPosts``` ```listPostsByAuthor``` ```listPostsByTag```.

1. Edit the /src/services/posts.js file to define a function at the end of the file. 
<details>
<summary>new posts.js</summary>

```.js
import { Post } from '../db/models/post.js';

// Using our Post model, we are creating a function that takes the required post fields, creates, and then returns a new Post. 
export async function createPost({ title, author, contents, tags }) {
    const post = new Post({ title, author, contents, tags });
    return await post.save();
};


// this function accepts a query and options argument (sortBy). 
async function listPosts (query = {}, { sortBy = 'createdAt', sortOrder = 'descending' } = {}) {
    return await Post.find(query).sort({[sortBy]: sortOrder});
};

// now define a function to list all posts
export async function listAllPosts(options) {
    return await listPosts({}, options)
}

// create a function to list all posts by a certain author
export async function listPostsByAuthor(author, options) {
    return await listPosts({author}, options);
}

// define a function to list posts by tag
export async function listPostsByTag(tags, options) {
    return await listPosts({tags}, options)
}
```
</details>
* in MongoDB we can match strings in an array by matching the string as if it was a single value.



### Defining test cases for list posts
We need to create an initial state where we already have some posts in the database to be able to test the list functions. This is done using the ```beforeEach()``` which will execute some code before each test case is executed. 

The function can also be used for a whole test file or execute it within a ```describe()```.

In our case we will use it for the whole file as the sample posts will be needed when we start deleting posts. 

<details>
<summary>updated posts.test.js</summary>

```.js
import mongoose from 'mongoose';
import { describe, expect, test, beforeEach, afterAll} from '@jest/globals';
import { createPost, listAllPosts, listPostsByAuthor, listPostsByTag } from '../services/posts.js'
import { Post } from '../db/models/post.js';

// this function creates a new test, we can have multiple tests in here
describe('creating posts', () => {

    // this function is where we define our new test
    test('with all parameters should succeeed', async() => {
        const post = {
            title: 'Hello Mongoose!',
            author: 'Chakane Shegog',
            contents: 'this post is stored in MongoDB db using Mongoose',
            tags: ['mongoose', 'mongodb']
        }
        const createdPost = await createPost(post);
        expect(createdPost._id).toBeInstanceOf(mongoose.Types.ObjectId);
        const foundPost = await Post.findById(createdPost._id);
        expect(foundPost).toEqual(expect.objectContaining(post));
        expect(foundPost.createdAt).toBeInstanceOf(Date);
    });

    test('without title should fail', async () => {
        const post = {
            author: 'Chakane Shegog',
            contents: 'Post with no title',
            tags: ['Empty']
        };

        try {
            await createPost(post);
        } catch (err) {
            expect(err).toBeInstanceOf(mongoose.Error.ValidationError);
            expect(err.message).toContain('`title` is required');
        }
    });

    test('with minimal parameters should succeed', async () => {
        const post = {
            title: 'Only a title',
        };
        const createdPost = await createPost(post);
        expect(createdPost._id).toBeInstanceOf(mongoose.Types.ObjectId);
    });



    
})

const samplePosts = [
    {title: 'Learning Redux', author: 'Daniel Bugl', tags: ['redux']},
    {title: 'Learn React with Hooks', author: 'Chakane Shegog', tags: ['react']},
    {
        title: 'Full-Stack React Projects',
        author: 'Daniel Bugl',
        tags: ['react', 'nodejs'],
    },
    {title: 'Guide to Typescript'}
]

// const post = require('../db/models/post.js'); // Adjust path to your Post model

beforeAll(async () => {
    await mongoose.connect('mongodb://localhost:27017/testdb', { // Replace with your test DB URI
        useNewUrlParser: true,
        useUnifiedTopology: true,
    });
});

afterAll(async () => {
    await mongoose.connection.close(); // Close connection after all tests
});

let createdSamplePosts = []
beforeEach(async () => {
    console.time("deleteMany");
    await Post.deleteMany({});
    console.timeEnd("deleteMany");
    
    console.time("createPosts");
    createdSamplePosts = await Promise.all(samplePosts.map(post => Post.create(post)));
    console.timeEnd("createPosts");
});

describe('listing posts',  () => {
    test('should return all posts', async () => {
        const posts = await listAllPosts();
        expect(posts.length).toEqual(createdSamplePosts.length);
    });

    test('should take into account provided sorting options', async () => {
        const posts = await listAllPosts({
            sortBy: 'updatedAt',
            sortOrder: 'ascending',
        })
        const sortedSamplePosts = createdSamplePosts.sort(
            (a, b) => a.updatedAt - b.updatedAt,
        )
        expect(posts.map((post) => post.updatedAt)).toEqual(
            sortedSamplePosts.map((post) => post.updatedAt),
        )
    });

    test('should be able to filter posts by author', async () => {
        const posts = await listPostsByAuthor('Daniel Bugl');
        expect(posts.length).toBe(3);
    });
});
```

</details>

### Defining the get single post, update and delete post functions
The service function for getting s single post is similar to list all posts. 

1. Lets start with the posts.js file under /services. and define a function called ```getPostById```.

```.js
export async function getPostsById(postId) {
    return await Post.findById(postId);
}
```


2. Then we will create an ```updatePost``` function
```.js
export async function updatePost(postId, {title, author, contents, tags}) {
    return await Post.findOneAndUpdate(
        {_id: postId},
        {$set:{title, author, contents, tags}},
        {new:true},
    );
};
```
</details>




### Providing a REST API using Express

<details>
    <summary>open</summary>

Now that we have the backend service functions setup with test cases were good to go with creating a REST (representational state transfer) API.This gives us a way to access our server by making your own http based endpoint and pinging it for data. 

This is incredibly useful for all kinds of apps. This gives the developer a safe way to share data to the user in a precise manner. All the user would have to do is make a GET request (could be triggered a button click, or submit form) in the browser and the Server will receive the request and respond back with data. We will be using Express JS to handle the server action to handle requests and respond back. 

The 5 most common requests are:

1. ```GET``` : Used to read resources. Does not change state of database. Typically responds back with a 200
2. ```POST``` : Used to create brand new resources by updating the database. The typical response is 201
3. ```PUT``` : Updates an existing resource (all fields are changed) by changing the database state> Also typically responds with a 201
4. ```PATCH``` : Modified an existing resource via ID (a single field is changed). Typically responds 201
5. ```DELETE``` : Deletes a resource in a database. Typically responds 200

The conventional way to define API routes in your project would be something like: ```/api/v1```. We can use ```/api/v2``` when changing the original ai destination and migrate the changes to v1.

### Defining our API Routes
Lets start defining our routes for the backend. We currently have to api functions written as a service function:

<details>
<summary>All API routes we are going to make</summary>
*  ```GET /api/v1/posts``` 
   *  Get all posts
<br>
*  ```GET /api/v1/posts?sortBy=updatedAt&sortOrder=ascending```
   *  Get a sorted list of all posts
<br>
*  ```GET /api/v1/posts?author=daniel```
   *  Get a lists of posts by name
  <br>
*  ```GET /api/v1/posts/?tag=react```
   *  Get a single post by tag
<br>
*  ```GET /api/v1/posts/:id```
   *  Get an existing post by ID
<br>
*  ```POST /api/v1/posts```
   *  Create a new post
<br>
*  ```PATCH /api/v1/posts/:id```
   *  Update an existing post by ID
<br>
*  ```DELETE /api/v1/posts/:id```
   *  Delete a post by ID
</details>

### Setting up Express
Express is a web framework for Node.js. It provides functions that will help us create our API functionality. 

1. You can install express with this command: ``` npm install express@4.18.2```. 
2. In the app.js file, write this code:

```.js
import express from 'express';

// create a new Express app
const app = express();

// define routes on the Express app. Define a GET route
app.get('/', (req, res) => {
  res.send('Hello from Express');
});

// export the app to use with other files
export { app };
```

3. Then we need to create a webserver in the index.js file

```.js
import { app } from './App.js'

// define a port number and have the Express instance listen to it
const PORT = 3345;
app.listen(PORT);
console.info(`express server running on http://localhost:${PORT}`);
```

4. Edit the package.json a start script to run our server.

```
"scripts": {
    "start": "node src/index.js",
}
```

5. Run the backend server

```npm start```

Now you should be able to access the localhost and wee the message you typed in app.js.

### Using dotenv for setting environment variables

As some point we will need to us environment variables. A good way to load them is in a ```dotenv --> .env```. This is how you can set these up:

1. Install the dotenv dependency: ```npm install dotenv@16.3.1```.
2. Edit the /src/index.js, import dovenv there, and call dotenv.config()

```
import dotenv from 'dotenv';
dotenv.config();
```

3. Start replacing any static variables with environment variables within index.js.

```
const PORT = process.send.PORT;
```

4. Import initDatabase into the index.js file and alter that file to first call the function import. If it fails, then we wont even attempt to start the express app. 

5. Create a ```.env``` file and define 2 variables:

```.env
PORT=3000
DATABASE_URL=mongodb://localhost:27017/blog
```

### Using nodemon for server auto restart

The nodemon tool allows us to run a server with the feature of "auto-restarting" the server on changes to source files.

1. Install the nodemon tool as a dependency

```
npm install -save-dev nodemon@3.0.2
```

2. Create a nodemon.json file in the root of your project and add the following contents to it:

```
{
    "watch": ["./src", ".env", "package-lock.json"]
}

This will ensure all code under /src will utilize the nodemon feature. 
```

3. Edit the package.json and define a new "dev" script that runs nodemon

```
"scripts": {
    "dev": "nodemon src/index.js",
}
```

4. Now run ```npm run dev```
5. Change some stuff around /src to see our code updates in real time.


### Creating API routes with Express
Now to start creating API routes with express, we need to:
1. Create a /src/routes/posts.js and import all our service functions.

<details>
    <summary>posts.js</summary>

```.jsx
import {
    listAllPosts,
    listPostsByAuthor,
    listPostsByTag,
    getPostBtId,
    getPostById,
} from '../services/posts.js';

// this function takes the express app as an argument
export function postsRoutes(app) {
    // define the routes (GET, POST, etc)

    // get all posts
    app.get('/api/v1/posts', async (req, res) => {

        // here, we can use query params to map them to the arguments of our functions. 
        const { sortBy, sortOrder, author, tag } = req.query;
        const options = { sortBy, sortOrder };

        try {

            // check if author or tag was provided
            if(author && tag) {
                return res.status(400).json({error: 'query by either author or tag, not both'});

            } 

            // return respective json
            else if(author) {
                return res.json(await listPostsByAuthor(author, options));
            }
            else if(tag) {
                return res.json(await listPostsByTag(tag, options));
            } else {
                return res.json(await listAllPosts(options));
            }
        } catch(err) {
            console.error('error listing posts', err);
            return res.status(500).end();
        }
    })

    // get single post by id
    app.get('/api/v1/posts/:id', async (req, res) => {
        // use req.params.id to get the :id part of our route and pass it to our service function
        const { id } = req.params;
        try {
            const post = await getPostById();

            // if the result is null return a 404 response, otherwise return json
            if(post === null) return res.status(404).end();
            return res.json(post);
            
        } catch(err) {
            console.error('error getting post', err);
            return res.status(500).end();
        }
    })
}
```

</details>

2. After defining the GET routes, we need to mount them in our app(within ```app.js```)
```import { postsRoutes } from './routes/posts';```

3. Then call the ```postRoutes(app)``` after initializing our express app.

```
const app = express();
postsRoutes(app);
```


</details>
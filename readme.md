# Build and Deploy JavaScript Web APIs to the Cloud 

## Why this Workshop

- Practical examples from start to finish on setting up production-ready APIs
- Learn how to get to the cloud ASAP
- Fill in the knowledge gaps

![An image of a starter owl drawing and the completed drawing. Source](https://pbs.twimg.com/media/C64ktBJWgAE3qxT?format=jpg&name=small)

## What to Expect

- Setup and build a RESTful web API with Node
- Configure CI/CD for your API
- Run API integrated tests in your CI/CD pipeline
- Deploy your API to the Cloud

## How to participate in this workshop

1. Follow along if you can
   - There are **snippets** for VS Code so you DON’T have to type it all
2. Chill and relaxed
    - You can also watch this on your sitting room's television
    - This is good if you don't have the prerequisites
    - Get the **high level knowledge**
        - Maybe take notes
    - But remember to practice what you learn later
        - Youtube (Link coming soon)
        - [Starter repository](https://github.com/christiannwamba/crud-api-node_STARTER)
        - [Completed repository](https://github.com/christiannwamba/crud-api-node) with [exercise commits](https://github.com/search?q=exercise+repo%3Achristiannwamba%2Fcrude-api-w-node&type=Commits&ref=advsearch&l=&l=)
3. Clone [exercise commits](https://github.com/search?q=exercise+repo%3Achristiannwamba%2Fcrude-api-w-node&type=Commits&ref=advsearch&l=&l=) not just the completed or starter repo
    ```bash
    git clone -n https://github.com/christiannwamba/crud-api-node 
    git checkout <COMMIT SHA>
    ```

## Prerequisites

- [Free Azure Account](https://azure.microsoft.com/free/?WT.mc_id=startup-0000-chnwamba)
- [Node.js](https://nodejs.org/en/download/) installed
- [VS Code](https://code.visualstudio.com/?WT.mc_id=startup-0000-chnwamba) (optionally but handy if you want to use the custom snippets)
- **Knowledge requirements**
  - JS fundamentals
  - Node.js basics

## Exercise 1: Setup with the Starter Repo

**Objectives**

- Setup the starter repository
- Walk through the starter code
- Run the starter code test

<details>
	<summary>Task 1: Setup starter repository</summary>

### Task 1: Setup starter repository

Clone the repository:

```bash
git clone https://github.com/christiannwamba/crud-api-node_STARTER crud-api-node
```

Install the dependencies:

```bash
cd crud-api-node
npm install
```

</details>

<details>
	<summary>Task 2. Walk through the starter code</summary>

### Task 2. Walk through the starter code

```bash
# Project tree
.
├── data.json
├── testData.json
├── package.json
├── index.js
├── test
│   ├── articles.skip.js
│   ├── db.skip.js
│   └── home.js
└── utils
    ├── config.js
    ├── flush.js
    ├── httpError.js
    └── seed.js
```

- `data.json` contains test data that we can use to populate our database
- `testData.json` is the same as `data.json` but with fewer data

**+ package.json**

The entry file is specified by the `start` script:

```json
"scripts": {
  "start": "node index.js",
  "dev": "nodemon index.js",
  "seed": "node -e 'require(\"./utils/seed.js\")()'",
  "flush": "node -e 'require(\"./utils/flush.js\")()'",
  "test:dev": "mocha --timeout 100000 --exclude \"./test/**/!(*.skip).js\" -w --recursive",
  "test": "mocha --timeout 100000 \"./test/**/!(*.skip).js\" --exit"
},
```

- The `dev` command starts the entry file with `nodemon`. Nodemon watches for changes and restarts the server so we don’t have to run `start` every time we edit a file.
- `seed` runs an exported function located in `./utils/seed.js` to populate the database with test data
- `flush` behaves like `seed` but instead, it empties the database
- `test:dev` runs the test files in the `test` folder and watches for changes
- `test` does not watch for any change after running the tests

> If a test file ends with `skip.js`, it is skipped when the test is running

**+ index.js**

The entry `index.js` file starts with importing `express` and `body-parser`.

```js
const express = require('express');
const bodyParser = require('body-parser');
```

Express is the HTTP/API/Routing framework or library for Node.js. Body parser formats and attaches request payload on express.

This file then goes one step forward to configure express and body parser:

```js
// Configure express
const app = express();
// Configure body-parser
app.use(bodyParser.urlencoded({ extended: true }));
app.use(bodyParser.json());
```

Next we try to get the port from the environment and if it is not found we use a port 4000:

```js
const port = process.env.PORT || 4000;
```

Then we create our first route:

```js
app.get('/', function (_, res) {
  res.send('Welcome to our API');
});
```

So when you visit the home page you get a greeting.

Lastly the server starts listening for requests:

```js
if (!module.parent) {
  app.listen(port);
  console.log('Magic happens on port ' + port);
}
```

The `!module.parent` check makes sure that this server is not started when the file is imported. Instead it can only be started when we run `index.js` directly.

Lastly we export the app for testing:

```js
module.exports = app;
```

- The test directory contains our tests.
  - `home.js` is for tests that test the `/` path
  - `articles.skip.js` is for tests that test the `/api/articles` path
    - The `skip` flag ensures that we don’t run the tests in this file yet since we haven’t written any articles API code
  - `db.skip.js` is for testing our database connection
- The `utils` folder just like the name contain utility logic:
  - The `config.js` file exports an object that contains our app credentials
    - Note how we use `dotenv` library to check the environment and use different databases for them.
  - The `seed.js` file uses `data.json` or `testData.json` to populate our database
  - The `flush.js` file empties the database
  - `httpError` is a utility function we can use to handle server errors

* You should rename the `.env.example` and `.test.env.example` files to `.env` and `.test.env` respectively. We will paste our database connection strings in these files and not push them to GitHub.

</details>

<details>
	<summary>Task 3. Run starter code</summary>

### Task 3. Run starter code

To check if our setup is ok, run the test script:

```bash
npm run test
```

</details>

<details>
	<summary>Task 4. Test with API Tester</summary>

### Task 4. Test with API Tester

You can use [Paw](https://paw.cloud), [Insomnia](https://insomnia.rest), [Postwoman](https://postwoman.io), or [Postman](https://www.postman.com) to test the endpoint.
</details>

## Exercise 2: Create Cloud Resources

Setting up your cloud should not be an after-thought. Actually, it should be part of your setup.

When you setup deployment early enough, you can use continuous deployment to ship features when they pass their tests

**Objectives**

- Install Azure CLI tool
- Create cloud resources on Azure
- Get credentials for accessing your cloud resources
  - Open a text editor and keep it handy. You will paste your cloud credentials in this editor

</details>

<details>
	<summary>Task 1: Install Azure CLI tool</summary>

### Task 1: Install Azure CLI tool

For Windows:

```powershell
Invoke-WebRequest -Uri https://aka.ms/installazurecliwindows -OutFile .\AzureCLI.msi; Start-Process msiexec.exe -Wait -ArgumentList '/I AzureCLI.msi /quiet'; rm .\AzureCLI.msi
```

For Mac:

```bash
brew update && brew install azure-cli
```

For Linux:

```bash
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

[Learn more…](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest&WT.mc_id=startup-0000-chnwamba)

</details>

<details>
	<summary>Task 2: Login to Azure</summary>

### Task 2: Login to Azure

Login to Azure with your CLI (you need to have created an [Azure Account](https://azure.microsoft.com/free/?WT.mc_id=startup-0000-chnwamba). It’s free):

```bash
az login
```

</details>

<details>
	<summary>Task 3: Set a default Azure Subscription</summary>

### Task 3: Set a default Azure Subscription

Every single time you run an Azure command to manage your resources (eg. Mongo database), Azure would ask for the [subscription](https://faq.rhipe.com/Search/Article/f021634f-b34e-4dbb-8f20-c009beafa170) to use to do that.

Set a default subscription so you don’t have to always provide it when creating or managing resources.

Run the following to see your subscriptions:

```bash
az account list
```

The output will look like:

```
[
  {
    "cloudName": "AzureCloud",
    "id": "<YOUR SUBSCRIPTION ID HERE>",
    "isDefault": true,
    "name": "<YOUR SUBSCRIPTION NAME HERE>",
    "state": "Enabled",
    "tenantId": "...",
    "user": {
      "name": "...",
      "type": "user"
    }
  },
  {
    "cloudName": "AzureCloud",
    "id": "...",
    "isDefault": false,
```

Run the following to set a default subscription:

```bash
az account set --subscription <YOUR SUBSCRIPTION ID>
```

</details>

<details>
	<summary>Task 4: Create cloud resources on Azure</summary>

### Task 4: Create cloud resources on Azure

We need the following resources:

1. A resource group for organizing all the resources
2. A MongoDB-based CosmosDB
   1. Test database
   2. Production database
3. A Service Plan for managing the web app pricing and OS
4. A Web App service for deploying the API

**+ 1. Resource group**

Create a resource group:

```bash
az group create \
  --name crud-api-node  \
  --location southcentralus
```

**+ 2a. CosmosDB for MongoDB test database**

```bash
az cosmosdb create \
  --name crud-api-node-db-test  \
  --resource-group crud-api-node \
  --kind MongoDB # You can setup different kinds of databases with CosmosDB
```

You might get an error that the name is taken. You can add numbers to the name to differ:

```bash
--name crud-api-node-db-test-12345 \
```

Get the primary key for connection to test database:

```bash
# Copy and save the key returned by this command
az cosmosdb keys list \
      --name crud-api-node-db-test \
      --resource-group crud-api-node \
      --query "primaryMasterKey"
```

**+ 2b. CosmosDB for MongoDB production database**

```bash
az cosmosdb create \
  --name crud-api-node-db  \
  --resource-group crud-api-node \
  --kind MongoDB # You can setup different kinds of databases with CosmosDB
```

Get the primary key for connection to test database:

```bash
# Copy and save the key returned by this command
az cosmosdb keys list \
  --name crud-api-node-db \
  --resource-group crud-api-node \
  --query "primaryMasterKey"
```

**+ 3. Service Plan**

```bash
az appservice plan create \
  --name crud-api-node-plan \
  --resource-group crud-api-node \
  --sku P1V2 \
  --is-linux
```

**+ 4. Web App Service for Node**

```bash
az webapp create \
  --name crud-api-node \
  --plan crud-api-node-plan \
  --runtime "node|12.9" \
  --resource-group crud-api-node
```

</details>

<details>
	<summary>Task 5: Set Database  Credentials</summary>

### Task 5: Set Database  Credentials

There few places we need to set credentials:

1. The `.env` file — production connection string
2. The `.test.env` file — test connection string
3. Our deployed web app — production connection string
4. Github Actions Secret — test connection string (we will set this in the [GitHub Actions exercise](#exercise-3-setup-github-actions-for-continuous-delivery))

**+ 1. & 2. Set in** `**.env**` **and** `**.test.env**` **files**

```env
MONGO_DB_CONNECTION_STRING="mongodb://<NAME>:<PRIMARY_KEY>@<NAME>.documents.azure.com:10255/?ssl=true"

MONGO_DB_DATABASE_NAME="blog"
```

> In the connection string, replace **<NAME>** with the database name. Eg: Mine would be `crud-api-node-db` or `crud-api-node-db-test`. Replace **<PRIMARY_KEY>** with the primary key we saved earlier

**+ 3. Set in deployed web app**

```bash
az webapp config appsettings set \
  --resource-group crud-api-node \
  --name crud-api-node \
  --settings MONGO_DB_CONNECTION_STRING="mongodb://<NAME>:<PRIMARY_KEY>@<NAME>.documents.azure.com:10255/?ssl=true" \
        MONGO_DB_DATABASE_NAME="blog"
```
</details>

## Exercise 3: Setup GitHub Actions for Continuous Delivery

<details>
	<summary>Task 1: Create a Github Action Workflow File</summary>

### Task 1: Create a Github Action Workflow File

First create a `.github` folder:

```bash
mkdir .github
```

Next add a `workflows` folder for all your GH actions:

```bash
mkdir .github/workflows
```

Now any `.yml` file you add to this folder will be used to setup a deployment for you by Github. Add a `deploy.yml` file:

```bash
touch .github/workflows/deploy.yml
```

</details>

<details>
	<summary>Task 2: Add Action event and name</summary>

### Task 2: Add Action event and name

In the workflow file, set the name of the action and the event that triggers this action:

```yml
## SNIPPET: ___e3t2.actions.deploy ##

on: [push]

name: Deploy CRUD API to Azure
```

We want to only trigger a deploy when code is pushed to the repo. You can also add pull request to the array.

</details>

<details>
	<summary>Task 3: Action workflow jobs</summary>

### Task 3: Action workflow jobs

Jobs tell GitHub actions what to do:

```yml
## SNIPPET: ___e3t3.actions.deploy ##

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
```

</details>

<details>
	<summary>Task 4: Create a step</summary>

### Task 4: Create a step

A step is a singular task that an Action should run. Eg. npm install, npm build, etc

```yml
## SNIPPET: ___e3t4.actions.deploy ##

steps:
  - name: 'Checkout GitHub Action'
    uses: actions/checkout@master
```

</details>

<details>
	<summary>Task 5: Login with Azure step</summary>

### Task 5: Login with Azure step

```yml
## SNIPPET: ___e3t5.actions.deploy ##

- name: 'Login to Azure'
  uses: azure/login@v1
  with:
    creds: ${{ secrets.AZURE_CREDENTIALS }}
```

</details>

<details>
	<summary>Task 6: Setup Node</summary>

### Task 6: Setup Node

```yml
## SNIPPET: ___e3t6.actions.deploy ##

- name: Setup Node 10.x
  uses: actions/setup-node@v1
  with:
    node-version: '10.x'
```

</details>

<details>
	<summary>Task 7: Run npm commands</summary>

### Task 7: Run npm commands

```yml
## SNIPPET: ___e3t7.actions.deploy ##

- name: 'npm install, build, and test'
  run: |
    npm install
    npm run build --if-present
    npm run test --if-present
```

</details>

<details>
	<summary>Task 8: Deploy to Azure</summary>

### Task 8: Deploy to Azure

```yml
## SNIPPET: ___e3t8.actions.deploy ##

- name: 'Deploy to Azure'
  uses: azure/webapps-deploy@v2
  with:
    app-name: 'crud-node-api'
```

> Remember to supply the correct name. The name should match the name you used when create the web app service with `az webapp create`

</details>

<details>
	<summary>Task 9: Logout of Azure</summary>

### Task 9: Logout of Azure

```yml
## SNIPPET: ___e3t9.actions.deploy ##

- name: logout
  run: |
    az logout
```

</details>

<details>
	<summary>Task 10: Get Azure Credentials</summary>

### Task 10: Get Azure Credentials

We need to give our GitHub action access to our Azure resource. That is why we have the `AZURE_CREDENTIALS` secret in the action file.

To generate the credential, run:

```bash
az ad sp create-for-rbac \
  --name "crud-node-api" \
  --role contributor \
  --scopes /subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.Web/sites/<NAME> \
  --sdk-auth
```

Replace the following:

- **<SUBSCRIPTION ID>**: The subscription ID for the web app. It will be default subscription ID we set at the beginning of the workshop. You can list your subscriptions with `az account list`
- **<RESOURCE GROUP>**: The resource group we created for the resources. Eg: `crud-api-node`
- **<NAME>**: The web app name. Eg: `crud-api-node`. This is different from the `--name` flag. The name flag is the name for the credential that Azure will generate NOT the name of the app

</details>

<details>
	<summary>Task 11: Create a GitHub repository</summary>

### Task 11: Create a GitHub repository

Head to Github and Create a repository

</details>

<details>
	<summary>Task 12: Add a Secret to GitHub Rep</summary>

### Task 12: Add a Secret to GitHub Rep

- Head to the new created repo settings
- Click on Secrets
- Name the secret: `AZURE_CREDENTIALS` to match what we have in the Action file
- Paste the JSON output of the `az ad sp` command from Task 9 as the value

Video:

[![](https://paper-attachments.dropbox.com/s_E4A1CA674E71FF7B0E87416A9281438BCA922F6DF89522392451720B4845D891_1587956518286_image.png)](https://vimeo.com/user96670037/review/412179820/0a2c5a0538)

</details>

<details>
	<summary>Task 13 Deploy with Push</summary>

### Task 13 Deploy with Push

Commit and Push to Github and watch the

```bash
# Remove original git history
rm -rf .git
# Init
git init
# Add
git add .
# Commit
git add commit -m "Initial"
# Remote
git remote add origin <REPO URL>
# Push
git push origin master
```

- Click on the Actions tab and open the action to view the logs
</details>

## Exercise 4: RESTful API Routing with Express

Express has a Router method you can use to create RESTful routes for your API.

**Objectives**

- Setup Express Router
- Create RESTful routes

</details>

<details>
	<summary>Task 1: Setup Router</summary>

### Task 1: Setup Router

In the index file, below the `/` route create a `router`:

```js
/* SNIPPET: ___e4t1.index */

const router = express.Router();
```

</details>

<details>
	<summary>Task 2: Add a route</summary>

### Task 2: Add a route

Use the `router` object to register a route:

```js
/* SNIPPET: ___e4t2.index */

router.get('/', function (req, res) {
  res.json({ message: 'hooray! welcome to our api!' });
});
```

</details>

<details>
	<summary>Task 3: Mount Routes</summary>

### Task 3: Mount Routes

You can take all the routes on a router object and mount it on any path you want. We will mount this `router` object we have on `/api`:

```js
/* SNIPPET: ___e4t3.index */

app.use('/api', router);
```

</details>

<details>
	<summary>Task 4: RESTful routing</summary>

### Task 4: RESTful routing

You can use one URL for say POST and GET:

```js
/* SNIPPET: ___e4t4.index */

router
  .route('/ping')
  .post(function (req, res) {
    res.send('You POST a PING');
  })
  .get(function (req, res) {
    res.send('You GET a PING');
  });

//
app.use('/api', router);
```
</details>

## Exercise 5: Database

**Objectives**

- Create a connection to a database

</details>

<details>
	<summary>Task 1: Create a database connection file</summary>

### Task 1: Create a database connection file

Create a folder `db` at the root of your project and add an `index.js`

</details>

<details>
	<summary>Task 2: Mongo Client and Config</summary>

### Task 2: Mongo Client and Config

Import the MongoDB Node client and the config file where we stored our connection string:

```js
/* SNIPPET: ___e5t2.db */

const MongoClient = require('mongodb').MongoClient;

const config = require('../utils/config');
```

</details>

<details>
	<summary>Task 3: Create a Client</summary>

### Task 3: Create a Client

```js
/* SNIPPET: ___e5t3.db */

function createDatabaseClient(url) {
  return new MongoClient(url, { useUnifiedTopology: true });
}
```

</details>

<details>
	<summary>Task 4: Create a Connection</summary>

### Task 4: Create a Connection

```js
/* SNIPPET: ___e5t4.db */

async function createDatabaseConnection() {
  const client = createDatabaseClient(config.database.connectionString);
  try {
    const clientConnection = await client.connect();
    return clientConnection;
  } catch (error) {
    throw error;
  }
}
```

- The `client.connect` method initiates the connection

</details>

<details>
	<summary>Task 5: Export the Connection</summary>

### Task 5: Export the Connection

```js
/* SNIPPET: ___e5t5.db */

module.exports = createDatabaseConnection;
```

</details>

<details>
	<summary>Task 6: Test Connection</summary>

### Task 6: Test Connection

Rename `test/db.skip.js` to `test/db.js` and run:

```bash
npm run test
```
</details>

## Exercise 6: Database Model

**Objectives**

- Create an database model for interacting with the API

</details>

<details>
	<summary>Task 1: Create database model for articles API</summary>

### Task 1: Create database model for articles API

In the next exercise, we are going to build an articles RESTful API. For now, let’s create a model that the API will use to interact with our database.

```bash
mkdir api
mkdir api/articles
touch api/articles/model.js
```

Import the necessary files for the model:

```js
/* SNIPPET: ___e6t1.articles.model */

const ObjectID = require('mongodb').ObjectID;

const config = require('../../utils/config');
const createDatabaseConnection = require('../../db');
```

`ObjectID` will be used to convert string IDs to Mongo DB Ids.

</details>

<details>
	<summary>Task 2: Insert a new article</summary>

### Task 2: Insert a new article

```js
/* SNIPPET:___e6t2.articles.model */

async function insertDocument(payload) {
  const client = await createDatabaseConnection();
  const db = client.db(config.database.name);
  return await db.collection('articles').insertOne(payload);
}
```

- First, create a client
- Use the client to create and/or get your db
- Use the db to insert a new article to the articles collection

</details>

<details>
	<summary>Task 3: Fetch all documents</summary>

### Task 3: Fetch all documents

```js
/* SNIPPET: ___e6t3.articles.model */

async function fetchAllDocuments() {
  const client = await createDatabaseConnection();
  const db = client.db(config.database.name);
  return await db.collection('articles').find({}).toArray();
}
```

- Same as inserting but instead uses `.find` to find all articles

</details>

<details>
	<summary>Task 4: Update a document</summary>

### Task 4: Update a document

```js
/* SNIPPET: ___e6t4.articles.model */

async function updateDocument(payload, id) {
  const client = await createDatabaseConnection();
  const db = client.db(config.database.name);

  return await db
    .collection('articles')
    .updateOne({ _id: ObjectID(id) }, { $set: payload });
}
```

- Same as inserting but instead uses `.updateOne` to update an article based on the `id` argument

</details>

<details>
	<summary>Task 5: Delete a document</summary>

### Task 5: Delete a document

```js
/* SNIPPET: ___e6t5.articles.model */

async function deleteDocument(id) {
  const client = await createDatabaseConnection();
  const db = client.db(config.database.name);
  return await db.collection('articles').deleteOne({ _id: ObjectID(id) });
}
```

- Same as inserting but instead uses `.deleteOne` to update an article based on the `id` argument

</details>

<details>
	<summary>Task 6: Export model methods</summary>


### Task 6: Export model methods

```js
/* SNIPPET:___e6t6.articles.model */

module.exports = {
  insertDocument,
  fetchAllDocuments,
  updateDocument,
  deleteDocument,
};
```

</details>

## Exercise 7: Build a RESTful Article API

We now have everything setup for us to create a data-backed RESTful API

**Objectives**

- Build a complete RESTful API

<details>
	<summary>Task 1: Restructure for API</summary>


### Task 1: Restructure for API

We don’t want to have all of our API code in just `index.js`. Instead let’s have our CRUD operations inside `api/articles`. In the `api` folder at the root of your project create the following file structure:

```bash
.
├── api
│   ├── articles
│   │   ├── create.js # For Create logic
│   │   ├── delete.js # For Delete logic
│   │   ├── index.js # Assemple all articles route
│   │   ├── model.js # Database model for ex 6
│   │   ├── read.js # For Read logic
│   │   └── update.js # For Update logic
```
</details>

<details>
	<summary>Task 2: C for Create</summary>

### Task 2: C for Create
In the `create.js` import the model and error helper:

```js
/* SNIPPET: ___e7t2.1.articles.api */

const model = require('./model');
const httpError = require('../../utils/httpError');
```

Next export a function that takes a route:

```js
/* SNIPPET: ___e7t2.2.1.articles.api */

module.exports = function (route) {
  //
};
```

Then return a route in the function:

```js
/* SNIPPET: ___e7t2.2.2.articles.api */

module.exports = function (route) {
  return route.post();
};
```

Add a handler for the route:

```js
/* SNIPPET: ___e7t2.2.3.articles.api */

module.exports = function (route) {
  return route.post(async function (req, res) {
    //
  });
};
```

Insert in the database and send a response:

```js
/* SNIPPET: ___e7t2.2.4.articles.api */

module.exports = function (route) {
  return route.post(async function (req, res) {
    try {
      const data = await model.insertDocument(req.body);

      res.json({ data: { insertedId: data.insertedId } });
    } catch (error) {
      httpError(res, error);
    }
  });
};
```

- `req.body` has the request data/payload

</details>

<details>
	<summary>Task 3: R for Read</summary>

### Task 3: R for Read

```js
/* SNIPPET: ___e7t3.articles.api */

const model = require('./model');
const httpError = require('../../utils/httpError');

module.exports = function (route) {
  return route.get(async function (_, res) {
    try {
      const data = await model.fetchAllDocuments();
      res.json({ data });
    } catch (error) {
      httpError(res, error);
    }
  });
};
```

</details>

<details>
	<summary>Task 4: U for Update</summary>

### Task 4: U for Update

```js
/* SNIPPET: ___e7t4.articles.api */

const model = require('./model');
const httpError = require('../../utils/httpError');

module.exports = function (route) {
  return route.put(async function (req, res) {
    try {
      const data = await model.updateDocument(req.body, req.params.id);
      res.json({ data: { modifiedCount: data.modifiedCount } });
    } catch (error) {
      httpError(res, error);
    }
  });
};
```

- `req.parms` is an object of the parameters passed in the URL

</details>

<details>
	<summary>Task 5: D for Delete</summary>

### Task 5: D for Delete

```js
/* SNIPPET: ___e7t5.articles.api */

const model = require('./model');
const httpError = require('../../utils/httpError');

module.exports = function (route) {
  return route.delete(async function (req, res) {
    try {
      const data = await model.deleteDocument(req.params.id);

      res.json({ data: { deletedCount: data.deletedCount } });
    } catch (error) {
      httpError(res, error);
    }
  });
};
```
</details>

<details>
  <summary>Task 6: Assemble with Index</summary>

### Task 6: Assemble with Index

We can use a register function in `articles/index` to setup all the scattered routes we have created.

Import the routes:

```js
/* SNIPPET: ___e7t6.1.articles.api */

const create = require('./create');
const read = require('./read');
const update = require('./update');
const remove = require('./delete'); // Can't name a variable delete cause of the `delete` keyword
```

Create a function that takes `router` (don’t confuse with `route` from previous ex):

```js
/* SNIPPET: ___e7t6.2.articles.api */

module.exports = function registerRoutes(router) {};
```

Create base and params routes:

```js
/* SNIPPET: ___e7t6.3.articles.api */

module.exports = function registerRoutes(router) {
  const baseRoute = router.route('/articles');
  const paramRoute = router.route('/articles/:id');
};
```

- `baseRoute` handles:
  - GET /articles
  - POST /articles
- `paramsRoute` handles
  - PUT /articles/:id
  - DELETE /articles/:id

Mount logics on routes

```js
/* SNIPPET: ___e7t6.4.articles.api */

module.exports = function registerRoutes(router) {
  const baseRoute = router.route('/articles');
  const paramRoute = router.route('/articles/:id');
  create(baseRoute);
  read(baseRoute);
  update(paramRoute);
  remove(paramRoute);
};
```
</details>

<details>
  <summary>Task 7: Run Tests</summary>

### Task 7: Run Tests

Remove the `skip` from `test/articles.skip.js` and try running the test.

```bash
npm run test
```

You should get 404 errors. This is because we have set up the routes but express in the entry point does not know about then

</details>

<details>
  <summary>Mount Routes on Express</summary>

### Task 8: Mount Routes on Express

Import the assemble articles route in the entry point:

```js
/* SNIPPET: ___e7t7.1.index */

// IMPORT ROUTES
const registerArticleRoutes = require('./api/articles');
```

Right below the `/ping` route, call the `registerArticleRoutes` function to register all the article routes:

```js
/* SNIPPET: ___e7t7.2.index */

registerArticleRoutes(router);
```
</details>


## Exercise 8: Deploy Again

</details>

<details>
	<summary>Task 1: Push Changes</summary>

### Task 1: Push Changes

- Push the new changes to Github and monitor the logs
- Watch the deploy fail because we do not have a DB connection string in GH Actions environment

</details>

<details>
	<summary>Task 2: Add Secrets for Connection String and DB Name</summary>

### Task 2: Add Secrets for Connection String and DB Name

```bash
MONGO_DB_CONNECTION_STRING="mongodb://<NAME>:<PRIMARY_KEY>@<NAME>.documents.azure.com:10255/?ssl=true" \
MONGO_DB_DATABASE_NAME="blog"
```

</details>

<details>
	<summary>Task 3: Update the deploy workflow</summary>

### Task 3: Update the deploy workflow

```yml
## SNIPPETS: ___e7t8.3.actions.deploy ##

  - name: 'npm install, build, and test'
      env:
        MONGO_DB_CONNECTION_STRING: ${{ secrets.MONGO_DB_CONNECTION_STRING }}
        MONGO_DB_DATABASE_NAME: ${{ secrets.MONGO_DB_DATABASE_NAME }}
      run: |
        npm install
        npm run build --if-present
        npm run test --if-present
```
</details>

## Appendix 1


**Create a Resource Group**

[![https://vimeo.com/user96670037/review/412175292/394ceff4cb](https://paper-attachments.dropbox.com/s_E4A1CA674E71FF7B0E87416A9281438BCA922F6DF89522392451720B4845D891_1587955029923_image.png)](https://vimeo.com/user96670037/review/412175292/394ceff4cb)

**Create a MongoDB-based Cosmos DB**

[![https://vimeo.com/user96670037/review/412175349/a793715e06](https://paper-attachments.dropbox.com/s_E4A1CA674E71FF7B0E87416A9281438BCA922F6DF89522392451720B4845D891_1587955106125_image.png)](https://vimeo.com/user96670037/review/412175349/a793715e06)

**Get Cosmos DB Primary Key**

1. Go to https://shell.azure.com/?WT.mc_id=startup-0000-chnwamba
2. [Set a default subscription](#task-3-set-a-default-azure-subscription) (on this page)
3. Run:


    az cosmosdb keys list \
      --name <DB NAME> \
      --resource-group <RESOURCE GROUP NAME> \
      --query "primaryMasterKey"

**Create a Web App**

[![https://vimeo.com/user96670037/review/412175465/d1abe504e6](https://paper-attachments.dropbox.com/s_E4A1CA674E71FF7B0E87416A9281438BCA922F6DF89522392451720B4845D891_1587955138574_image.png)](https://vimeo.com/user96670037/review/412175465/d1abe504e6)

**Get Azure Credentials for Web App**

Refer to [Get Azure Credentials](#task-10-get-azure-credentials) (on this page)

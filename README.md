![Serverless GraphQL Architecture Application Boilerplate](/readme_boilerplate_graphql.gif)

# Serverless GraphQL Boilerplate
[![serverless](http://public.serverless.com/badges/v3.svg)](http://www.serverless.com)

This boilerplate application features a serverless architecture that leverages new technologies (Lambda, GraphQL) to reach a very low *total cost of ownership* (i.e., least amount of code, administration, cost).

- [Setup](#setup)
- [FAQs](#FAQs)
- [Team](#team)

---

## Setup

If you haven't yet installed `serverless` on your machine, run:

```
npm install -g serverless
```
then install the GraphQL Boilerplate in the CWD by running:

```
sls project install serverless-boilerplate-graphql
cd serverless-boilerplate-graphql
```

### Back
Add the `authTokenSecret` variable to `_meta/variables/s-variables-STAGE-REGION.json` and give it a strong value. This is the secret used to generate JSON web tokens. Then:

```
cd back/api
npm install
sls function deploy --all
sls endpoint deploy --all
```

### Client
Set `API_URL` in `client/src/app/js/actions/index.js`. Then:

```
cd ../../client/src
npm install
npm start
```

This will run the client locally. You can then deploy the client to an S3 bucket with:

```
npm run build
sls client deploy
```

### Deploying to S3 bucket
Make sure you're still in the `client/src` folder mentioned above, then run:

```
npm run build
sls client deploy
```

### Testing With A Local DynamoDB Instance
- Install [Docker](https://www.docker.com/)
- Run `docker-compose up` to install and run DynamoDB.
- Add the `localDynamoDbEndpoint` variable with the value `http://<DOCKER-MACHINE-IP>:8000` to `_meta/variables/s-variables-common.json`. Example value:  `http://192.168.99.100:8000`.
- Run `sls setup db -s <stage> -r <region>` to create tables in the local DynamoDB instance.
- Run `sls offline start` to start [the offline server](https://github.com/dherault/serverless-offline).

### Testing With GraphiQL
If you're running OSX, you can use the [GraphiQL Electron App](https://github.com/skevy/graphiql-app) to test the GraphQL backend without a client:

- Install [brew cask](https://caskroom.github.io) for easy installation: `brew tap caskroom/cask`
- Install GraphiQL App: `brew cask install graphiql`
- Open GraphiQL application. Just search for `GraphiQL` using OSX Spotlight Search!
- Add your `data` endpoint to the "GraphQL Endpoint" text field, and make sure the "Method" is set to `POST`.
- Try this mutation to create the first user:


```
mutation createUserTest {
  createUser (username: "serverless", name: "Serverless Inc.", email: "hello@serverless.com", password: "secret") {
    id 
    username 
    name 
    email  
  }
}
```

- Now list all users using the following query:


```
query getUsersTest { 
  users {
    id
    username
    name
    email
  } 
}
```

- You should get the user you just created:


```
{
  "data": {
    "users": [
      {
        "id": "aca42ee0-f509-11e5-bc11-0d8b1f79b4b9",
        "username": "serverless",
        "name": "Serverless Inc.",
        "email": "hello@serverless.com"
      }
    ]
  }
}
```

### Team Workflow with Meta Sync Plugin
This boilerplate includes the [Meta Sync Plugin](https://github.com/serverless/serverless-meta-sync). To start using it you need to add the following serverless variables to `_meta/variables/s-variables-common.json`:

```js
"meta_bucket" : "SOME_BUCKET_NAME",
"meta_bucket_region" : "us-east-1" // or any other region
```

---

## FAQs

### Why use GraphQL + Lambda?
Lambda is a revolutionary service that makes it really easy to build and maintain microservices at a fraction of the cost. However, to build a full scale Rest API, you often have to maintain a large number of lambdas and endpoints. This is where GraphQL comes in! GraphQL makes it possible to use only a single `data` lambda with a single endpoint. You can then run queries on this single lambda to get any data shape you want. We believe GraphQL could bring back monolithic architecture from the dead! 

### How to add more data records?
In this boilerplate, we're managing all data records using GraphQL. Currently the boilerplate only has a Users record/collection. But you can easily add any other data collection (i.e.. `Posts`) in the `back/api/lib/graphql/collections` directory. Just follow the same pattern in the `Users` collection.

### How does validation work with GraphQL?
Each data collection has a `validate.js` file. This is where you should keep your validation logic, and call the validation functions on the data received from GraphQL before you resolve them. GraphQL has its own validation implementation, but it's at a very early stage at this point.

### How does authentication work with GraphQL?
In this boilerplate, we're using JSON Web Token for authentication. You can find this logic in the `back/api/lib/auth.js` file. You can simply switch to another authentication mechanism by editing this file.

### Where to put the business logic of my project?

### What plugins are included with this boilerplate?
- [serverless-client-s3](https://github.com/serverless/serverless-client-s3): To deploy front end assets to S3
- [serverless-cors-plugin](https://github.com/joostfarla/serverless-cors-plugin): To enable CORS for your data endpoint and give the client access to your backend.
- [serverless-meta-sync](https://github.com/serverless/serverless-meta-sync): To collaborate with your teammates and sync your `_meta` folder with them securely using an S3 bucket.
- [serverless-offline](https://github.com/dherault/serverless-offline): To test your project locally during development. That also includes a local dynamoDB instance.

### What's the difference between installing vs cloning & initing the boilerplate?
They both achieve the same goal. However, Installing the boilerplate with `sls project install` is simpler because it also handles installing the project dependencies. We recommend using `sls project init` only if you clone the project to contribute/make a PR.

### Why do we have to `npm install` 2-3 times?
Because we're dealing with isolated micro services architecture, we have some separation of concerns around different areas of the project. So dependencies are managed at three levels:
- Project Dependencies: by running `npm install` in the root of the project. This is done for your automatically when you run `sls project install`. This mostly handles installing the plugins.
- Backend Dependencies: by running `npm install` in the root of the `data/api` directory. This makes all the `node_modules` required by the boilerplate available for deployment with your functions.
- Frontend Dependencies: by running `npm install` in the root of the `client/src` directory. This installs all the client side dependencies to make your React application work.

### Why can't we deploy with `sls dash deploy`?
You can deploy with `sls dash deploy`, however the Serverless CORS Plugin requires that you deploy your endpoints with `sls endpoint deploy` so that it can fire the necessary pre hooks that will enable CORS.

### How to connect the client to the Serverless backend?
By setting the `API_URL` variable in `client/src/app/js/actions/index.js` Please keep in mind that this is the **root of your API** not the endpoint url you get from `sls endpoint deploy`

---

## Team
* [@kevinold](https://github.com/kevinold)
* [@pmuens](https://github.com/pmuens)
* [@breandr](https://github.com/breandr)
* [@erikerikson](https://github.com/erikerikson)
* [@ryansb](https://github.com/ryansb)
* [@eahefnawy](https://github.com/eahefnawy)
* [@minibikini](https://github.com/minibikini)
* [@ac360](https://github.com/ac360)

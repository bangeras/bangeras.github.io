---
title: mern-nginx-mfe-serverless-demo
date: 2024-02-20 10:48:30
categories: [MERN, Microfrontends]
tags: [reactjs, microfrontend, nginx, serverless, lambda, nodejs ]     # TAG names should always be lowercase
---

[![Source Code](/assets/img/sourcecode.png "Source code"){: width="30" height="35" }](https://github.com/bangeras/mern-nginx-mfe-serverless-demo){:target="_blank"}{:rel="noopener noreferrer"}

## Overview
This is a template project that can be used to build apps supporting the following base capabilities:
* MERN stack    : MongoDb, Mongoose, Express middleware, ReactJS, Redux, NodeJS
* NGINX         : as a Reverse Proxy, Load Balancer, API Gateway
* Serverless    : Use serverless framework to develop/deploy NodeJS lambdas, exposed via APIGateway on AWS
* Microfrontends: Using Module federation use a container app to host passenger apps built in any Front-end framework(React, Vue, Angular, etc.)
* Passport      : Integrate with Auth providers (Google, Facebook, Twitter, etc.) for OIDC, OAuth integrations
* Redis         : Caching


## Architecture
![](/assets/img/mern-nginx-mfe-serverless-demo/mern.png){: width="500" height="500" }

## NGINX
![NGINX flow](/assets/img/mern-nginx-mfe-serverless-demo/nginx.png){: width="500" height="500" }

## MicroFrontends
MicroFrontends are a design pattern where Front-ends are composed from independent fragments that can be built in a federated model by multiple teams
using different, disparate technology frameworks (React, Vue, Ng2, etc.)

### Module Federation
It is the process of loading separately compiled applications at run time.

```
    new ModuleFederationPlugin(
        {
            name: 'CONTAINER',
            filename: 'remoteEntry.js',
            remotes: {
            "MFE1": "MFE1@http://localhost:3001/remoteEntry.js"
            }, 
            exposes: {
            },
        }
    ),
```
{: file="container app's webpack.config.js"}

```
    new ModuleFederationPlugin(
        {
            name: 'MFE1',
            filename: 'remoteEntry.js',
            exposes: {
            "./ComponentA": "./src/ComponentA",
            "./ComponentB": "./src/ComponentB"
            },
        }
    ),
```
{: file="passenger app's webpack.config.js"}

```
import React, {lazy, Suspense} from 'react'

const ComponentA = lazy(() => import("MFE1/ComponentA"));
const ComponentB = lazy(() => import("MFE1/ComponentB"));

const Dashboard = () => {
  return (
    <div>
      <h1>Dashboard</h1>

      <Suspense fallback={<span>Loading...</span>}>
        <ComponentA /> 
        <ComponentB/>     
      </Suspense>
    </div>
  )
}

export default Dashboard
```
{: file="dashboard.js"}

## EXPRESS Middleware

```
const EXPRESS_PORT = 4000

const app = express()

// ###### Middlewares ######

// Add all essential middlewares
const EXPRESS_PORT = 4000

const app = express()

// ###### Middlewares ######

// Logging middleware:  Log HTTP Requests and Errors
app.use(morgan('tiny'))

// Oauth Passport middleware: Initialize Passport and restore authentication state, if any, from the session
app.use(passport.initialize());
app.use(passport.session());

// Use express.json() middleware to parse JSON requests
app.use(express.json());

// ###### Routes ######
app.use("/auth", authRoutesRouter)
app.use("/api/users", userRoutesRouter);

// connect react to nodejs express server
app.listen(config.get("EXPRESS.PORT"), () => console.log(`Server is running on port ${config.get("EXPRESS.PORT")}!`));
```
{: file="index.js"}


```
// Router for API routes

const {User} = require("../models/user")
const router = require("express").Router();
// const {authCheckMiddleware} = require("../middlewares/authCheckMiddleware")
const {getAllUsers, createUser, getUser, updateUser, deleteUser} = require("../controllers/userController")

router.route("/")
  // GET /api/users
  .get(getAllUsers)
  // POST /api/users
  .post(createUser)

```
{: file="api-routes.js"}
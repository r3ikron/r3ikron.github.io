---
layout: post
title:  "Setting up Node.js and Angular development"
date:   2022-12-28 06:44:00 +0200
categories: jekyll update
---

## Introduction

This guide walks you through setting up a Node.js and Angular development environment. We'll also touch upon database setup, configuration, and other essential tasks.

## NPM Configuration

Before starting, it's a good idea to configure npm with your personal information. This will be used to populate the package.json file in your future projects.

```
npm config set init-author-name "username" -g
npm config set init-author-url "https://github.com/username" -g
npm config set init-license "MIT" -g
```

## Git Configuration

You'll also want to set up Git with your information and preferred settings.

```bash
git config --global init.defaultBranch main
git config --global user.email "email"
git config --global user.name "username"
```

Once your local project is ready, you can link it with a remote repository as follows:

```bash
git remote add origin https://github.com/username/REPOSITORY.git
git branch -M main
git push -u origin main
```

Angular CLI Installation

Next, we'll install the Angular CLI, which provides a powerful set of tools for Angular development.

```bash
npm install -g @angular/cli
```

You can then create a new Angular project:

```bash
ng new todo-app --routing --style=scss --skip-tests
cd todo-app
```

## Project Directory Structure

We'll organize our project by creating several directories for different types of files:

```bash
mkdir data
mkdir data/db
touch data/db/.gitkeep
mkdir data/migrations
touch data/migrations/.gitkeep
mkdir data/seeds
touch data/seeds/.gitkeep
mkdir data/files
touch data/files/.gitkeep
mkdir doc
touch doc/.gitkeep
mkdir api
mkdir test
touch test/.gitkeep
```

## Git Ignore File

To ignore some generated files or folders, create a `.gitignore` file with the following entries:

```bash
/data/db
/data/files
```

## Proxy Configuration

To set up a proxy for your API calls during development, create a proxy.conf.json file:

_proxy.conf.json_

```json
{
  "/api": {
    "target": "http://localhost:3000",
    "secure": false,
    "pathRewrite": {
      "^/api": ""
    }
  }
}
```

And update your Angular JSON file to use this proxy during development:

```json
  "architect": {
    "serve": {
      "builder": "@angular-devkit/build-angular:dev-server",
      "options": {
        "proxyConfig": "src/proxy.conf.json"
      },
```

## Database Configuration with Knex

Knex is a powerful query builder for Postgres, MySQL, and other databases. We will be using it to manage our database operations.

Create a `knexfile.js` in your project root:

```javascript
const path = require("path");

module.exports = {

  development: {
    client: "sqlite3",
    connection: {
      filename: path.join(__dirname, "/data/db/development.db")
    },
    useNullAsDefault: true,
    migrations: {
      directory: path.join(__dirname, "/data/migrations"),
    },
    seeds: {
      directory: path.join(__dirname, "/data/seeds"),
    }
  },

  test: {
    client: "sqlite3",
    connection: {
      filename: ":memory:"
    },
    useNullAsDefault: true,

    migrations: {
      directory: path.join(__dirname, "/data/migrations"),
    },
    seeds: {
      directory: path.join(__dirname, "/data/seeds"),
    }
  },

  production: {
    client: "pg",
    connection: process.env['DATABASE_URL'],

    migrations: {
      directory: path.join(__dirname, "/data/migrations"),
    },
    seeds: {
      directory: path.join(__dirname, "/data/seeds"),
    }
  }
}
```

And a `db.js` file inside your api folder:

```javascript
const Knex = require("knex");

const config = require("../knexfile.js");
const env = process.env['NODE_ENV'] || "development";

/*config[env].log = {
  warn(message) {
  },
  error(message) {
  },
  deprecate(message) {
  },
  debug(message) {
  },
};*/

module.exports = Knex(config[env]);
```

## Back-end Setup with Express

We'll set up an Express server to handle our backend logic. Install the required packages first:

```bash
npm install --save express body-parser cookie-parser express-session connect-session-knex pg bootstrap validator
npm install --save-dev sqlite3 nodemon concurrently
```

Create an api/main.js file to initiate your Express server:

```javascript
const express = require("express");
const path = require("path");
const bodyParser = require("body-parser");
const cookieParser = require("cookie-parser");
const session = require("express-session");
const knex = require("./db");
const SessionStore = require("connect-session-knex")(session);
const sessionStore = new SessionStore({ knex });

const routes = require("./routes");

const DIST = "../dist/todo-app";
const ENV = process.env['NODE_ENV'] || "production";
const PORT = parseInt(process.env['PORT'] || "3000");

const app = express();
app.set("env", ENV);
app.set("port", PORT);
app.disable("x-powered-by");
app.use(bodyParser.urlencoded({ extended: false }));
app.use(bodyParser.json());
app.use(express.static(path.join(__dirname, DIST)));
app.use((req, res, next) => {
    if (req.url.startsWith("/api/")) {
        next();
    } else {
        return res.status(200).sendFile("/", { root: path.join(__dirname, DIST) });
    }
});
app.use(cookieParser());
app.use(session({ name: "sid", secret: "conduit", cookie: { maxAge: 60000 }, resave: false, saveUninitialized: false, store: sessionStore }));

/*app.use(async (req, res, next) => {
  if (req.session.userId) {
    let rows = await knex.select("id", "email").from("users").where({ id: req.session.userId });
    res.locals['user'] = rows[0];
  }
  next();
});*/

app.use("/api/", routes);

app.listen(PORT, () => console.log(`http://localhost:${PORT} in ${app.get("env")} mode`));
```

Create an api/routes.js file for your api calls:

```javascript
const { Router } = require("express");
const router = module.exports = Router();
```

## Front-end Setup with Angular

First, add the Angular HTTP client module to make HTTP requests.

```typescript
import { HttpClientModule } from '@angular/common/http';

@NgModule({
  imports: [
    HttpClientModule,
  ]
})
```

For styles, you can import Bootstrap in your `styles.scss`:
_styles.scss_

```scss
@import '~bootstrap/dist/css/bootstrap.min.css';
```

Adding external javaScript libraries to angular

```json
"scripts": [
  "node_modules/bootstrap/dist/js/bootstrap.bundle.min.js",
  "node_modules/validator/validator.min.js"
]
```

In an Angular application, the `app.component.ts` file is central. Let's consider this minimalist setup:

```typescript
template: `<router-outlet></router-outlet>`,
styles: []
```

Nodemon & Concurrency: `nodemon.json`

```
{
  "watch": ["api"],
  "ext": "js,json",
  "exec": "concurrently --kill-others \"node ./api/main.js\" \"ng serve --proxy-config proxy.conf.json\""
}
```

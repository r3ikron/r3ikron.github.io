---
layout: post
title:  "Setting up Node.js and Angular development"
date:   2022-12-28 06:44:00 +0200
categories: jekyll update
---

_npm_
```
npm config set init-author-name "username" -g
npm config set init-author-url "https://github.com/username" -g
npm config set init-license "MIT" -g
```

_git_
```
git config --global init.defaultBranch main
git config --global user.email "email"
git config --global user.name "username"
```

```
git remote add origin https://github.com/username/REPOSITORY.git
git branch -M main
git push -u origin main
```

```
npm install -g @angular/cli
```

```
ng new todo-app --routing --style=scss --skip-tests
cd todo-app
```

```
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

_.gitignore_

```
/data/db
/data/files
```
_proxy.conf.json_

```
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

_angular.json_
```
  "architect": {
    "serve": {
      "builder": "@angular-devkit/build-angular:dev-server",
      "options": {
        "browserTarget": "your-application-name:build",
        "proxyConfig": "src/proxy.conf.json"
      },
```

_knexfile.js_

```
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

_api/db.js_

```
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

```
npm install --save express body-parser cookie-parser express-session connect-session-knex pg bootstrap validator
npm install --save-dev sqlite3 nodemon concurrently
```

```
const { Router } = require("express");
const router = module.exports = Router();
```

 _api/main.js_

```
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

_styles.scss_

```
@import '~bootstrap/dist/css/bootstrap.min.css';
```

_angular.json_

```
"scripts": [
  "node_modules/bootstrap/dist/js/bootstrap.bundle.min.js",
  "node_modules/validator/validator.min.js"
]
```

_src/app/app.component.ts_

```
template: `<router-outlet></router-outlet>`,
styles: []
```

_src/app/app.module.ts_

```
import { HttpClientModule } from '@angular/common/http';

@NgModule({
  imports: [
    HttpClientModule,
  ]
})
```

_nodemon.json_
```
{
  "watch": ["api"],
  "ext": "js,json",
  "exec": "concurrently --kill-others \"node ./api/main.js\" \"ng serve --proxy-config proxy.conf.json\""
}
```

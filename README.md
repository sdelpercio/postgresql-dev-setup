# Setting Up a Local Development PostgreSQL Database with pgAdmin4, Node.js, Express, and Knex #

### 1. Initialize Repo
* Create a Remote repository on Github, initialize with a .gitignore and readme.md
* Clone repo to local computer, open with code editor
* In terminal, cd into root folder
* Run these commands in terminal:
  * `$ npm init -y`
  * `$ npm i express knex pg dotenv` and any other production dependencies you may need (helmet, bcryptjs, jsonwebtoken, etc)
  * `$ npx knex init`
  
## 2. Setup Server
* Create a 'server.js' file inside an 'API' folder
```javascript
const server = express();

server.use(express.json());

server.use("/api/auth", authRouter);

server.get("/", (req, res) => {
  res.status(200).json({ api: "up" });
});

module.exports = server;
```
* Create an 'index.js' file at root
```javascript
// we'll create the .env later
require("dotenv").config();

const server = require("./API/server.js");

const port = process.env.PORT || 5000;
server.listen(port, () => console.log(`\n** server up on port ${port} **\n`));
```
* Create your routes, such as 'auth-router.js' (this will depend on what information you need from your database)
```javascript
const router = require("express").Router();
const bcrypt = require("bcryptjs");
const jwt = require("jsonwebtoken");
const { jwtSecret } = require("../config/secrets.js");

const Users = require("./users-model.js");

// for endpoints beginning with /api/auth
router.post("/register", validateUserContent, (req, res) => {
  let user = req.body;
  const hash = bcrypt.hashSync(user.password, 10); // 2 ^ n
  user.password = hash;

  Users.add(user)
    .then((saved) => {
      res.status(201).json({
        saved,
      });
    })
    .catch((error) => {
      res.status(500).json(error);
    });
});

// ...

module.exports = router;
```
* Create your database model functions, such as 'users-model.js'
```javascript
const db = require("../data/dbConfig.js");

module.exports = {
  add
};

async function add(user) {
  const [id] = await db("users").insert(user, "id");

  return findById(id);
}
```

## 3. Setup Knex and Migrations/Seeds
* Create a folder named 'data' at root, and inside create two more folders named 'migrations' and 'seeds'
* To create migrations execute `$ npx knex migrate:make migration_name` in terminal
* To create seeds execute `$ npx knex seed:make seed_name` in terminal
* In your 'knexfile.js' file copy and paste this:
```javascript
require("dotenv").config();

module.exports = {
  development: {
    client: "pg",
    connection: process.env.DATABASE_URL,
    migrations: {
      directory: "./data/migrations",
    },
    seeds: {
      directory: "./data/seeds",
    },
  },
  production: {
    client: "pg",
    connection: process.env.DATABASE_URL,
    migrations: {
      directory: "./data/migrations",
    },
    seeds: {
      directory: "./data/seeds",
    },
  },
  testing: {
    client: "pg",
    connection: process.env.DATABASE_URL,
    migrations: {
      directory: "./data/migrations",
    },
    seeds: {
      directory: "./data/seeds",
    },
  },
};
```

## 4. Setup PostgreSQL Database in pgAdmin4
* Start pgAdmin
* Log into your account upon opening
* Click on server at left, login again
* Create new database under PostgreSQL 12
<img src="https://i.ibb.co/P64Fg3n/pgadmin1.png" alt="pgadmin database creation" border="0"/></br>
* Name your database and click 'save' at bottom right

## 5. Make Connection with Database
* Inside your 'data' folder, create a file named 'dbConfig.js'
```javascript
const knex = require("knex");
const config = require("../knexfile.js");

const environment = process.env.DB_ENV || "development";

module.exports = knex(config[environment]);
```
* At root, create a new file named '.env', the URL will let knex connect with our locally hosted database
* Replace 'username' and 'password' with the login credentials used with your PostgreSQL 12 server
```
PORT=5500
DATABASE_URL=postgres://username:password@localhost/database-name
DB_ENV=development
```

## 6. Run Migrations/Seeds
* Start your server by executing `$ npm run server` in terminal
* Open new terminal window, then execute these commands:
  * `$ npx knex migrate:rollback`
  * `$ npx knex migrate:latest`
  * `$ npx knex seed:run`
* Refresh your database in pgAdmin, then check your schema to make sure the migrations/seeds worked

## Congrats!! Your local PostgreSQL database is ready to be used with your Node/Express API!

# Steps to change from session to token

This repository contains all the steps needed to switch from session to token after installation with `npx ironlauncher <project name> --auth -json`.
You can test the code in combination with the following repository, which contains the code for all steps for authentication in React / client-side:
https://github.com/Ironborn-Ironhack-March-2022/jwt-auth-client

## Initial Setup:

```javascript
npx ironlauncher <project name> --auth --json
npm i jsonwebtoken express-jwt
```

```javascript
npm uninstall express-session connect-mongo
```

- necessary for this code, but deppends on your code in auth.routes:

```javascript
npm install bcryptjs
npm uninstall bcrypt
```

## Steps

## 0. Delete all code for session, for example in config/index.js

- DELETE:

```javascript
config / index.js;
Delete; // ℹ️ Session middleware for authentication
// https://www.npmjs.com/package/express-session
const session = require("express-session");

// ℹ️ MongoStore in order to save the user session in the database
// https://www.npmjs.com/package/connect-mongo
const MongoStore = require("connect-mongo");

// ℹ️ Middleware that adds a "req.session" information and later to check that you are who you say you are 😅
app.use(
  session({
    secret: process.env.SESSION_SECRET || "super hyper secret key",
    resave: false,
    saveUninitialized: false,
    store: MongoStore.create({
      mongoUrl: MONGO_URI,
    }),
    cookie: {
      maxAge: 1000 * 60 * 60 * 24 * 365,
      sameSite: "none",
      secure: process.env.NODE_ENV === "production",
    },
  })
);

app.use((req, res, next) => {
  req.user = req.session.user || null;
  next();
});
```

## 1. Modify auth routes

## in auth.routes.js

### POST /signup

- "can" be modified (we don't need to store info in session + can automatically log in the user)

### POST /login

- we do not store info in session
- we generate & sign a token + we send it in the response

- `const jwt = require("jsonwebtoken")`

- generate token after you check that password is correct.

  In router.post("/login" add below:

```javascript
  // Compare the provided password with the one saved in the database
      const passwordCorrect = bcrypt.compareSync(password, foundUser.password);

      if (passwordCorrect) { // login was successful

        // Deconstruct the user object to omit the password
        const { _id, email } = foundUser;

        // Create an object that will be set as the token payload
        const payload = { _id, email };

  const authToken = jwt.sign(payload, process.env.TOKEN_SECRET, {
    algorithm: "HS256",
    expiresIn: "6h",
  });

  return res.json({ authToken: authToken });
});
```

### in .env file

- `TOKEN_SECRET=<yoursecret>`

### example of current .env file

```javascript
PORT = 5005;
TOKEN_SECRET = "keyboard cat";
ORIGIN = "http://localhost:3000";
```

## 2. Create middleware

- in middleware folder create jwt.middleware.js file and delete isLoggedIn + isLoggedOut

- delete middleware from auth.routes.js, for example here delete isLoggedOut:

`router.post("/login", isLoggedOut, (req, res, next) => {`

### in jwt.middleware.js

```javascript
const { expressjwt } = require("express-jwt");

// Instantiate the JWT token validation middleware
const isAuthenticated = expressjwt({
  secret: process.env.TOKEN_SECRET,
  algorithms: ["HS256"],
  requestProperty: "payload",
  getToken: getTokenFromHeaders,
});

// Function used to extracts the JWT token from the request's 'Authorization' Headers
function getTokenFromHeaders(req) {
  // Check if the token is available on the request Headers
  if (
    req.headers.authorization &&
    req.headers.authorization.split(" ")[0] === "Bearer"
  ) {
    // Get the encoded token string and return it
    const token = req.headers.authorization.split(" ")[1];
    return token;
  }

  return null;
}

// Export the middleware so that we can use it to create a protected routes
module.exports = {
  isAuthenticated,
};
```

## 3. Require this middleware and include it in routes

### in auth.routes.js

```javascript
const { isAuthenticated } = require("./middleware/jwt.middleware");
```

### in index.routes.js

```javascript
const { isAuthenticated } = require("../middleware/jwt.middleware");
```

## 4. Create new route in auth.routes

### GET /verify

```javascript
router.get("/verify", isAuthenticated, (req, res, next) => {
  console.log(`req.payload`, req.payload);

  res.json(req.payload);
});
```

- Remove route /logout if you have it

## 5. Create 401 error in error-handling/index.js

### add code to send a 401

```javascript
if (err.name === "UnauthorizedError") {
  res.status(401).json({ message: "invalid token..." });
}
```

### in app.js

- if you want to protect all your routes, you can do it this way:

```javascript
router.use("/projects", isAuthenticated, projectRoutes);
router.use("/tasks", isAuthenticated, taskRoutes);
```

in this example our routes are called /projects and /tasks (we have projects.routes.js & tasks.routes.js). In your case you would be using the names of YOUR routes

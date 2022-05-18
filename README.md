## Initial Setup:

```javascript
npx ironlauncher <project name> --auth --json
npm i jsonwebtoken express-jwt
```

```javascript
npm uninstall express-session connect-mongo
```

## Steps

### 1.

### (in auth.routes.js)

- `javascript const jwt = require("jsonwebtoken")`

- generate token after you check that password is correct.

  In router.post("/login" add below:

```javascript
bcrypt.compare(password, user.password).then((isSamePassword) => {
  if (!isSamePassword) {
    return res.status(400).json({ errorMessage: "Wrong credentials." });
  }

  const payload = {
    _id: user._id,
    username: user.username,
  };

  const authToken = jwt.sign(payload, process.env.TOKEN_SECRET, {
    algorithm: "HS256",
    expiresIn: "6h",
  });

  return res.json({ authToken: authToken });
});
```

### (in .env file)

- `TOKEN_SECRET=<yoursecret>`

### 2.

- in middleware folder create jwt.middleware.js file and delete isLoggedIn + isLoggedOut

- delete middleware from auth.routes.js, for example here delete isLoggedOut:

`router.post("/login", isLoggedOut, (req, res, next) => {`

### (in jwt.middleware.js)

- create JWT token validation middleware

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

### 3.

- require this middleware and include it in routes

### (in auth.routes.js )

```javascript
const { isAuthenticated } = require("./middleware/jwt.middleware");
```

### (in index.routes.js)

```javascript
const { isAuthenticated } = require("../middleware/jwt.middleware");
```

- if you want to protect all your routes, you can do it this way:

```javascript
router.use("/projects", isAuthenticated, projectRoutes);
router.use("/tasks", isAuthenticated, taskRoutes);
```

in this example our routes are called /projects and /tasks (we have projects.routes.js & tasks.routes.js). In your case you would be using the names of YOUR routes

# Create Vite-React-AntDesign Project
> Instrucations for setting up Node.js server, MongoDB instance, Vite + React project, installing Ant Design, and starting the server. This will create the base for and MERN stack app Vite and Ant Design.
### Set Up Folder and Git
###### terminal
```
$ mkdir project-name
$ cd project-name
$ git init
```
### Add .gitignore file
- create new file > .gitignore
- add the following content:
###### .gitignore
```
# standard files to ignore in git project folders
.gitignore
**/.github
*.md
LICENSE
.env

# See https://help.github.com/articles/ignoring-files/ for more about ignoring files.

# dependencies
/node_modules
/.pnp
.pnp.js

# testing
/coverage

# production
/build

# misc
.DS_Store
.env.local
.env.development.local
.env.test.local
.env.production.local

npm-debug.log*
yarn-debug.log*
yarn-error.log*

# configs
*.config.js

# Logs
logs
*.log
npm-debug.log*
yarn-debug.log*
yarn-error.log*
pnpm-debug.log*
lerna-debug.log*

node_modules
dist
dist-ssr
*.local

# Editor directories and files
.vscode/*
!.vscode/extensions.json
.idea
.DS_Store
*.suo
*.ntvs*
*.njsproj
*.sln
*.sw?
```
### Add README and Environment files
- create new file > README.md
- add readme content:
###### README.md
```
# Project Title
Project description

### [Josh Schmitz](https://github.com/JoshSchmitz)

### MERN Application
Created with Vite, React, and Ant Design with ExpressJS API and MongoDB data center.
```
### Add Dependencies File
- create and add the following to package.json file
###### package.json
```
{
  "name": "project-name",
  "version": "1.0.0",
  "description": "Project description.",
  "main": "main.jsx",
  "scripts": {
    "start": "node api/server.js",
    "server": "nodemon api/server.js",
    "client": "npm run dev --prefix client",
    "dev": "concurrently \"npm run server\" \"npm run client\""
  },
  "author": "Josh Schmitz",
  "license": "ISC",
  "type": "module",
  "dependencies": {},
  "devDependencies": {}
}
```
### Install Server and API Packages
###### terminal
```
$ npm install express
$ npm install express-async-handler
$ npm install dotenv
$ npm install mongoose
$ npm install bcrtyptjs
$ npm install jsonwebtoken
$ npm install cookie-parser
$ npm install -D nodemon
$ npm install -D concurrently
```
### Set up MongoDB database
- log in to MongoDB
- create a new database `project-name` with collection `users`

###### .env
```
NODE_ENV=development
PORT=6000
MONGO_URI=mongodb+srv://joshschmitz:IF1ZcBITUFE7tEu7@cluster0.hj3wny9.mongodb.net/[project-name]?retryWrites=true&w=majority
JWT_SECRET=z7uWXaJAwDCCy3&eswR#k#PGjSEMbzt6Kfbc3ghAk76oi2u9hK
```
### Start API folder
###### terminal
```
$ mkdir api
```
- create the following server files
###### server.js
```
import express from 'express';
import dotenv from 'dotenv';
import cookieParser from 'cookie-parser';
import connectDB from './config/db.js';
import userRoutes from './routes/user.js';
import { notFound, errorHandler } from './middleware/errors.js';

// config
dotenv.config();
const app = express();
const port = process.env.PORT || 6000;
connectDB();

// middleware
app.use(express.json());
app.use(express.urlencoded({ extended: true }));
app.use(cookieParser());

// routes
app.use('/api/user', userRoutes);

// error handlers
app.use(notFound);
app.use(errorHandler);

// start server
app.listen(port, () => console.log(`Server started on port ${port}`));
```
###### middleware/errors.js
```
const notFound = (req, res, next) => {
  const error = new Error(`Not Found - ${req.originalUrl}`);
};

const errorHandler = (err, req, res, next) => {
  let statusCode = res.statusCode === 200 ? 500 : res.statusCode;
  let message = err.message;

  if (err.name === 'CastError' && err.kind === 'ObjectId') {
    statusCode = 404;
    message = 'Resource not found';
  }

  res.status(statusCode).json({
    message,
    stack: process.env.NODE_ENV === 'production' ? null : err.stack,
  });
};

export { notFound, errorHandler };
```
###### middleware/authenticate.js
```
import jwt from 'jsonwebtoken';
import asyncHandler from 'express-async-handler';
import User from '../models/user.js';

const protect = asyncHandler(async (req, res, next) => {
  let token;
  token = req.cookies.jwt;

  if (token) {
    try {
      const decoded = jwt.verify(token, process.env.JWT_SECRET);
      req.user = await User.findById(decoded.userId).select('-password');
      next();
    } catch (err) {
      res.status(401);
      throw new Error('Not authorized, invalid token');
    }
  } else {
    res.status(401);
    throw new Error('Not authorized, no token');
  }
});

export { protect };
```
###### config/db.js
```
import mongoose from 'mongoose';

const connectDB = async () => {
  try {
    const conn = await mongoose.connect(process.env.MONGO_URI);
    console.log(
      `MongoDB Connected: ${conn.connection.host} - ${conn.connection.name}`
    );
  } catch (err) {
    console.error(`Error: ${err.message}`);
    process.exit(1);
  }
};

export default connectDB;
```
###### utils/generate-token.js
```
import jwt from 'jsonwebtoken';

const generateToken = (res, userId) => {
  const token = jwt.sign({ userId }, process.env.JWT_SECRET, {
    expiresIn: '30d',
  });

  res.cookie('jwt', token, {
    httpOnly: true,
    secure: process.env.NODE_ENV !== 'development',
    sameSite: 'strict',
    maxAge: 30 * 24 * 60 * 60 * 1000,
  });
};

export default generateToken;
```
###### models/user.js
```
import mongoose from 'mongoose';
import bcrypt from 'bcryptjs';

const userSchema = mongoose.Schema(
  {
    name: {
      type: String,
      required: true,
    },
    email: {
      type: String,
      unique: true,
      required: true,
    },
    password: {
      type: String,
      required: true,
    },
    admin: { type: Boolean, default: false },
    cart: {
      status: { type: String, default: 'active' },
      products: [
        {
          _id: {
            type: mongoose.Schema.Types.ObjectId,
            required: true,
            ref: 'Product',
          },
          name: { type: String, required: true },
          price: { type: Number, required: true },
          quantity: { type: Number, required: true },
        },
      ],
      quantity: { type: Number, default: 0 },
      total: { type: Number, default: 0 },
    },
  },
  { timestamps: true }
);

userSchema.pre('save', async function (next) {
  if (!this.isModified('password')) {
    next();
  }
  const salt = await bcrypt.genSalt(10);
  this.password = await bcrypt.hash(this.password, salt);
});

userSchema.methods.matchPassword = async function (sentPassword) {
  return await bcrypt.compare(sentPassword, this.password);
};

const User = mongoose.model('User', userSchema);
export default User;
```
###### controllers/user.js
```
import asyncHandler from 'express-async-handler';
import generateToken from '../utils/generate-token.js';
import User from '../models/user.js';

/* 
    @desc: Authenticate user
    @route: POST /api/users/auth
    @access: public
*/
const authUser = asyncHandler(async (req, res) => {
  const { email, password } = req.body;
  const user = await User.findOne({ email });

  if (user && (await user.matchPassword(password))) {
    generateToken(res, user._id);
    res.status(200).json({
      _id: user._id,
      name: user.name,
      email: user.email,
      admin: user.admin,
    });
  } else {
    res.status(401);
    throw new Error('Invalid email or password');
  }
});

/* 
    @desc: Register a new user
    @route: POST /api/users
    @access: public
*/
const registerUser = asyncHandler(async (req, res) => {
  const { name, email, password } = req.body;
  const exists = await User.findOne({ email });

  if (exists) {
    res.status(400);
    throw new Error('User already exists');
  }

  const user = await User.create({
    name,
    email,
    password,
  });

  if (user) {
    generateToken(res, user._id);
    res.status(201).json({
      _id: user._id,
      name: user.name,
      email: user.email,
    });
  } else {
    res.status(400);
    throw new Error('Invalid user data');
  }
});

/* 
    @desc: Logout user
    @route: POST /api/users/logout
    @access: public
*/
const logoutUser = asyncHandler(async (req, res) => {
  res.cookie('jwt', '', { httpOnly: true, expires: new Date(0) });
  res.status(200).json({ message: 'User logged out' });
});

/* 
    @desc: Get user profile
    @route: GET /api/users/profile
    @access: private
*/
const getUserProfile = asyncHandler(async (req, res) => {
  const user = await User.findById(req.user._id);
  if (user) {
    res.status(200).json(user);
  } else {
    res.status(404);
    throw new Error('User not found');
  }
});

/* 
    @desc: Update user profile
    @route: PUT /api/users/profile
    @access: private
*/
const updateUserProfile = asyncHandler(async (req, res) => {
  const user = await User.findById(req.user._id);

  if (user) {
    user.name = req.body.name || user.name;
    user.email = req.body.email || user.email;
    if (req.body.password) {
      user.password = req.body.password;
    }
    const updatedUser = await user.save();

    res.status(200).json({
      _id: updatedUser._id,
      name: updatedUser.name,
      email: updatedUser.email,
    });
  } else {
    res.status(404);
    throw new Error('User not found');
  }
});

export {
  authUser,
  registerUser,
  logoutUser,
  getUserProfile,
  updateUserProfile,
};
```
###### routes/userjs
```
import express from 'express';
import {
  authUser,
  registerUser,
  logoutUser,
  getUserProfile,
  updateUserProfile,
} from '../controllers/user.js';
import { protect } from '../middleware/authenticate.js';

// instantiate router
const router = express.Router();

// routes
router.post('/', registerUser);
router.post('/auth', authUser);
router.post('/logout', logoutUser);
router
  .route('/profile')
  .get(protect, getUserProfile)
  .put(protect, updateUserProfile);

export default router;
```
### Start Vite Application
###### terminal
```
$ npm create vite client
> React
> Javascript
$ cd client
$ npm install
$ npm audit fix --force
```
### Clean Up Folder Structure
###### vscode
```
_ project-folder
|_ >api
|_ >client
 |_ >node_modules
 |_ >src
  |_ >assets
  |_ >components
   |_ App.jsx
  |_ main.jsx
 |_ .gitignore
 |_ eslint.config.js
 |_ index.html
 |_ package-lock.json
 |_ package.json
 |_ vite.config.js
```
> Make sure to update file imports, dependencies, title, and metadata in App.jsx, main.jsx, and index.html.
### Add API Proxy to Vite Config
Update Vite Config to include the following code to add a server proxy to the application so the api server and client server can run simultaneously.
###### vite.config.js
```
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [react()],
  server: {
    port: 7001,
    proxy: {
      '/api': {
        target: 'http://localhost:6000',
        changeOrigin: true,
      },
    },
  },
});
```
### install Ant Design
###### terminal
```
$ npm install antd
```
### Start the servers
###### terminal
```
$ cd ..
$ npm run dev
```
> ### Start Coding!

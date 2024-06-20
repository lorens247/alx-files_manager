# 0x04. Files Manager

## Overview

This project demonstrates how to create a files management API using Express, authenticate users, store data in MongoDB, manage temporary data in Redis, and set up a background worker.

## Table of Contents

1. [Setup](#setup)
2. [Creating an API with Express](#creating-an-api-with-express)
3. [User Authentication](#user-authentication)
4. [Storing Data in MongoDB](#storing-data-in-mongodb)
5. [Managing Temporary Data in Redis](#managing-temporary-data-in-redis)
6. [Setting Up and Using a Background Worker](#setting-up-and-using-a-background-worker)

## Setup

To get started, clone the repository and install the necessary dependencies:

```bash
git clone https://github.com/lorens247/alx-files-manager.git
cd alx-files-manager
npm install
```

## Creating an API with Express

Express is a web framework for Node.js. To create an API with Express:

1. **Initialize the Project**: 
    ```bash
    npm init -y
    npm install express
    ```

2. **Create the Server**:
    ```javascript
    const express = require('express');
    const app = express();
    const port = 3000;

    app.use(express.json());

    app.get('/', (req, res) => {
      res.send('Welcome to the Files Manager API');
    });

    app.listen(port, () => {
      console.log(`Server running at http://localhost:${port}/`);
    });
    ```

3. **Define Routes**: 
    Create routes for managing files, e.g., upload, download, delete, etc.

## User Authentication

To authenticate users, you can use JWT (JSON Web Tokens):

1. **Install Dependencies**:
    ```bash
    npm install jsonwebtoken bcryptjs
    ```

2. **Set Up Authentication**:
    ```javascript
    const jwt = require('jsonwebtoken');
    const bcrypt = require('bcryptjs');

    // Sign up
    app.post('/signup', async (req, res) => {
      const { username, password } = req.body;
      const hashedPassword = await bcrypt.hash(password, 10);
      // Store user in database
      // ...
      res.status(201).send('User created');
    });

    // Log in
    app.post('/login', async (req, res) => {
      const { username, password } = req.body;
      // Fetch user from database
      // ...
      const isValidPassword = await bcrypt.compare(password, user.password);
      if (isValidPassword) {
        const token = jwt.sign({ userId: user._id }, 'your_jwt_secret');
        res.json({ token });
      } else {
        res.status(401).send('Invalid credentials');
      }
    });

    // Middleware to protect routes
    const authenticate = (req, res, next) => {
      const token = req.header('Authorization').replace('Bearer ', '');
      try {
        const decoded = jwt.verify(token, 'your_jwt_secret');
        req.user = decoded;
        next();
      } catch (err) {
        res.status(401).send('Please authenticate');
      }
    };
    ```

## Storing Data in MongoDB

1. **Install Mongoose**:
    ```bash
    npm install mongoose
    ```

2. **Connect to MongoDB**:
    ```javascript
    const mongoose = require('mongoose');

    mongoose.connect('mongodb://localhost:27017/files_manager', {
      useNewUrlParser: true,
      useUnifiedTopology: true,
    });

    const db = mongoose.connection;
    db.on('error', console.error.bind(console, 'connection error:'));
    db.once('open', () => {
      console.log('Connected to MongoDB');
    });
    ```

3. **Define Schemas and Models**:
    ```javascript
    const fileSchema = new mongoose.Schema({
      filename: String,
      path: String,
      userId: mongoose.Schema.Types.ObjectId,
    });

    const File = mongoose.model('File', fileSchema);
    ```

## Managing Temporary Data in Redis

1. **Install Redis**:
    ```bash
    npm install redis
    ```

2. **Connect to Redis**:
    ```javascript
    const redis = require('redis');
    const client = redis.createClient();

    client.on('error', (err) => {
      console.log('Redis error: ', err);
    });

    client.on('connect', () => {
      console.log('Connected to Redis');
    });
    ```

3. **Use Redis to Store Temporary Data**:
    ```javascript
    // Set a key
    client.set('key', 'value', redis.print);

    // Get a key
    client.get('key', (err, reply) => {
      console.log(reply);
    });
    ```

## Setting Up and Using a Background Worker

1. **Install Bull**:
    ```bash
    npm install bull
    ```

2. **Create a Background Worker**:
    ```javascript
    const Bull = require('bull');
    const fileQueue = new Bull('file-processing', 'redis://127.0.0.1:6379');

    // Producer: Add jobs to the queue
    app.post('/upload', (req, res) => {
      // Logic to handle file upload
      // ...
      fileQueue.add({ fileId: file._id });
      res.status(200).send('File uploaded and job added to queue');
    });

    // Consumer: Process jobs
    fileQueue.process(async (job) => {
      const { fileId } = job.data;
      // Logic to process the file
      // ...
    });
    ```

3. **Monitor Jobs**:
    You can use a UI like [Bull Board](https://github.com/vcapretz/bull-board) to monitor your Bull jobs.

## Conclusion

This project demonstrates the basic setup for a files manager API using Express, JWT for authentication, MongoDB for data storage, Redis for temporary data management, and Bull for background job processing. You can extend these concepts to build a robust and scalable files management system.

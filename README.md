# Docker & MERN Experimentation

This guide will walk you through creating a simple MERN stack application using Docker, with local JSON data instead of MongoDB.

## Project Structure

```plaintext
mern-docker-app/
├── backend/
│   ├── data/
│   │   └── items.json
│   ├── Dockerfile
│   ├── package.json
│   └── server.js
├── frontend/
│   ├── public/
│   ├── src/
│   │   ├── App.js
│   │   └── index.js
│   ├── Dockerfile
│   └── package.json
├── docker-compose.yml
└── .dockerignore
```

## Step 1: Set Up the Backend

1. Create the backend directory and navigate into it:

   ```bash
   mkdir -p mern-docker-app/backend
   cd mern-docker-app/backend
   ```

2. Initialize a new Node.js project and install dependencies:

   ```bash
   npm init -y
   npm install express cors
   ```

3. Create a `data` directory and an `items.json` file inside it:

   ```bash
   mkdir data
   echo '[
     {"id": 1, "name": "Item 1"},
     {"id": 2, "name": "Item 2"},
     {"id": 3, "name": "Item 3"}
   ]' > data/items.json
   ```

4. Create `server.js`:

   ```javascript
   const express = require("express");
   const cors = require("cors");
   const fs = require("fs");
   const path = require("path");

   const app = express();
   const PORT = process.env.PORT || 5000;

   app.use(cors());
   app.use(express.json());

   app.get("/api/items", (req, res) => {
     const items = JSON.parse(
       fs.readFileSync(path.join(__dirname, "data", "items.json"), "utf8")
     );
     res.json(items);
   });

   app.listen(PORT, () => {
     console.log(`Server running on port ${PORT}`);
   });
   ```

5. Create a `Dockerfile` for the backend:

   ```dockerfile
   FROM node:14
   WORKDIR /usr/src/app
   COPY package*.json ./
   RUN npm install
   COPY . .
   EXPOSE 5000
   CMD ["node", "server.js"]
   ```

## Step 2: Set Up the Frontend

1. Create the frontend directory and navigate into it:

   ```bash
   cd ..
   npx create-react-app frontend
   cd frontend
   ```

2. Replace the content of `src/App.js`:

   ```jsx
   import React, { useState, useEffect } from "react";

   function App() {
     const [items, setItems] = useState([]);

     useEffect(() => {
       fetch("http://localhost:5000/api/items")
         .then((response) => response.json())
         .then((data) => setItems(data));
     }, []);

     return (
       <div>
         <h1>Items</h1>
         <ul>
           {items.map((item) => (
             <li key={item.id}>{item.name}</li>
           ))}
         </ul>
       </div>
     );
   }

   export default App;
   ```

3. Create a `Dockerfile` for the frontend:

   ```dockerfile
   FROM node:14
   WORKDIR /usr/src/app
   COPY package*.json ./
   RUN npm install
   COPY . .
   EXPOSE 3000
   CMD ["npm", "start"]
   ```

## Step 3: Set Up Docker Compose

1. In the root directory, create a `docker-compose.yml` file:

   ```yaml
   version: "3"
   services:
     backend:
       build: ./backend
       ports:
         - "5000:5000"
       volumes:
         - ./backend:/usr/src/app
         - /usr/src/app/node_modules
     frontend:
       build: ./frontend
       ports:
         - "3000:3000"
       volumes:
         - ./frontend:/usr/src/app
         - /usr/src/app/node_modules
       stdin_open: true
       depends_on:
         - backend
   ```

2. Create a `.dockerignore` file in the root directory:

   ```plaintext
   node_modules
   npm-debug.log
   ```

## Step 4: Build and Run the Application

1. In the root directory, build and start the containers:

   ```bash
   docker-compose up --build
   ```

2. Access the application:
   - Frontend: <http://localhost:3000>
   - Backend API: <http://localhost:5000/api/items>

## Step 5: Development Workflow

- To make changes, edit the files in your local directory. The changes will be reflected in the running containers due to the volume mounts.
- If you add new dependencies, you'll need to rebuild the containers:

  ```bash
  docker-compose down
  docker-compose up --build
  ```

## Cleanup

To stop and remove the containers:

```bash
docker-compose down
```

This setup provides a simple MERN stack application using Docker, with local JSON data instead of MongoDB. The backend serves the JSON data, and the frontend displays it.

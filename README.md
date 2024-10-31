This is a learning module for docker and CI/CD pipelining with Docker and Git

### Step 1: Project Setup
1. **Choose a Tech Stack**:
   - For this example, we’ll use **Node.js** for the backend and **Express** for the web framework, along with a simple front-end using **HTML/CSS/JavaScript**.

2. **Initialize the Project**:
   - Create a new directory for your project:
     ```bash
     mkdir devsecops-project
     cd devsecops-project
     ```
   - Initialize a Git repository:
     ```bash
     git init
     ```

3. **Create the Web Application**:
   - Set up a basic Node.js application:
     ```bash
     npm init -y
     npm install express
     ```
   - Create the following file structure:
     ```
     devsecops-project/
     ├── server.js
     ├── public/
     │   ├── index.html
     │   └── styles.css
     └── package.json
     ```

4. **Create `server.js`**:
   ```javascript
   const express = require('express');
   const path = require('path');

   const app = express();
   const PORT = process.env.PORT || 3000;

   // Serve static files
   app.use(express.static('public'));

   app.get('/', (req, res) => {
       res.sendFile(path.join(__dirname, 'public', 'index.html'));
   });

   app.listen(PORT, () => {
       console.log(`Server is running on http://localhost:${PORT}`);
   });
   ```

5. **Create the Frontend**:
   - In `public/index.html`, create a basic HTML structure:
   ```html
   <!DOCTYPE html>
   <html lang="en">
   <head>
       <meta charset="UTF-8">
       <meta name="viewport" content="width=device-width, initial-scale=1.0">
       <link rel="stylesheet" href="styles.css">
       <title>DevSecOps Project</title>
   </head>
   <body>
       <h1>Welcome to DevSecOps Project</h1>
       <p>This is a basic web application.</p>
   </body>
   </html>
   ```

6. **Commit Your Changes**:
   ```bash
   git add .
   git commit -m "Initial commit: Set up basic web application"
   ```

### Step 2: Dockerization

1. **Create a Dockerfile**:
   - In the project root, create a `Dockerfile`:
   ```Dockerfile
   # Use the official Node.js image
   FROM node:16

   # Set the working directory
   WORKDIR /usr/src/app

   # Copy package.json and install dependencies
   COPY package*.json ./
   RUN npm install

   # Copy the rest of the application code
   COPY . .

   # Expose the port the app runs on
   EXPOSE 3000

   # Command to run the application
   CMD ["node", "server.js"]
   ```

2. **Create a Docker Compose File**:
   - In the project root, create a `docker-compose.yml`:
   ```yaml
   version: '3.8'

   services:
     web:
       build: .
       ports:
         - "3000:3000"
       volumes:
         - .:/usr/src/app
       environment:
         - NODE_ENV=development
   ```

3. **Build and Run the Docker Container**:
   ```bash
   docker-compose up --build
   ```
   - Access your application at `http://localhost:3000`.

### Step 3: CI/CD Configuration

1. **Create GitHub Repository**:
   - Create a new repository on GitHub and push your local repository to GitHub:
   ```bash
   git remote add origin <your-github-repo-url>
   git push -u origin master
   ```

2. **Set Up GitHub Actions**:
   - In the root of your project, create a `.github/workflows/ci.yml` file:
   ```yaml
   name: CI

   on:
     push:
       branches:
         - master

   jobs:
     build:
       runs-on: ubuntu-latest

       steps:
       - name: Checkout code
         uses: actions/checkout@v2

       - name: Set up Node.js
         uses: actions/setup-node@v2
         with:
           node-version: '16'

       - name: Install dependencies
         run: npm install

       - name: Run tests (if any)
         run: npm test || echo "No tests to run"

       - name: Build Docker image
         run: docker build . -t devsecops-project

       - name: Run OWASP ZAP security scan
         uses: zaproxy/action-baseline@v0.0.12
         with:
           target: 'http://localhost:3000'
           docker-image: 'owasp/zap2docker-stable'
   ```

3. **Configure Secrets**:
   - If you need secrets (like API keys), add them in your GitHub repository settings under "Secrets".

### Step 4: Integrate Security Tools

1. **Static Code Analysis with SonarQube**:
   - Set up a SonarQube server (you can run it in a Docker container).
   - Add a SonarQube configuration to your GitHub Actions workflow:
   ```yaml
   - name: SonarQube Scan
     uses: SonarSource/sonarcloud-github-action@master
     with:
       args: >
         sonar-scanner
         -Dsonar.projectKey=<your_project_key>
         -Dsonar.organization=<your_organization>
         -Dsonar.host.url=<your_sonarqube_server>
         -Dsonar.login=${{ secrets.SONAR_TOKEN }}
   ```

2. **Dynamic Application Security Testing (DAST) with OWASP ZAP**:
   - OWASP ZAP will automatically run as part of your GitHub Actions workflow. Ensure you have the target URL correctly set.

### Step 5: Monitoring Setup

1. **Set Up Prometheus**:
   - Create a `prometheus.yml` configuration file:
   ```yaml
   global:
     scrape_interval: 15s

   scrape_configs:
     - job_name: 'node_app'
       static_configs:
         - targets: ['localhost:3000']
   ```

2. **Run Prometheus**:
   - You can run Prometheus in a Docker container:
   ```bash
   docker run -d -p 9090:9090 -v $(pwd)/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus
   ```

3. **Set Up Grafana**:
   - Run Grafana in a Docker container and configure it to use Prometheus as a data source.
   ```bash
   docker run -d -p 3001:3000 grafana/grafana
   ```
   - Access Grafana at `http://localhost:3001`, and add Prometheus as a data source.

### Step 6: Testing and Iteration

1. **Continuous Testing**:
   - Ensure you have automated tests in place (using frameworks like Jest for Node.js).
   - Add test scripts in your `package.json`:
   ```json
   "scripts": {
     "test": "jest"
   }
   ```

2. **Iterate Based on Feedback**:
   - After each deployment, monitor logs and metrics, and adjust your application and security settings as necessary.

### Step 7: Deployment

1. **Deploy to Production**:
   - You can deploy your Docker container to a cloud provider (AWS ECS, GCP GKE, or DigitalOcean).
   - Ensure that the production environment matches your local setup in terms of configuration and security practices.

2. **Monitor in Production**:
   - Use Prometheus and Grafana to monitor your production application, checking for performance issues or security vulnerabilities.

3. **Documentation**:
   - Update your README file to include setup instructions, deployment procedures, and security practices you implemented.

### Final Thoughts

This project covers the essential components of a DevSecOps pipeline, from code development to security testing and deployment. You can expand on it by adding features like database integration, advanced security measures (e.g., SSO), or more complex CI/CD configurations. If you need any further details on specific sections or tools, feel free to ask!

# 🎬 Netflix Movie Project — DevOps CI/CD Deployment

This project documents my hands-on deployment of a full-stack Netflix Movie Application using **GitHub, GitHub Actions, Docker, AWS ECR, AWS EC2, MongoDB Atlas, and Portainer**.

The objective was to containerize the frontend and backend applications, store the Docker images in Amazon ECR, deploy them on an AWS EC2 instance, connect the backend to MongoDB Atlas, manage the containers using Portainer, and automate deployment with GitHub Actions.

---

## 🏗️ Project Architecture

```text
Developer
    │
    ▼
GitHub Repository
    │
    ▼
GitHub Actions
    │
    ├── Build Docker Image
    │
    ▼
Amazon ECR
    │
    ▼
AWS EC2
    │
    ├──────────────┐
    ▼              ▼
Frontend         Backend
Port 3000        Port 8080
                   │
                   ▼
              MongoDB Atlas

Portainer → Docker Container Management
```

---

# 1. Preparing the Backend Project

I started by cloning the provided Netflix backend source code to my local computer. Because the project originally belonged to another Git repository, I removed the existing `.git` directory and initialized a new Git repository.

This allowed me to manage the project under my own GitHub account.

Some of the Git commands used during this stage included:

```bash
rm -rf .git
git init
git branch -M main
git status
```

**What I achieved:** I successfully converted the cloned backend project into my own Git repository.

<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/bfd42fc8-86ed-4c1d-afe8-3a08498f3392" />


---

# 2. Adding and Committing the Backend Files

After initializing Git, I added the backend project files to the staging area and created my first commit.

```bash
git add .
git commit -m "Initial commit"
```

I then connected the local project to my own GitHub repository and pushed the `main` branch.

**What I achieved:** The Netflix backend source code was successfully uploaded to my GitHub account.

<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/a6810b92-55ba-40f8-953c-93366499926a" />

---

# 3. Preparing the Frontend Repository

I repeated the Git setup process for the Netflix frontend application.

I removed the original Git history, initialized a new repository, and connected the project to my frontend GitHub repository.

During this process, I also encountered a Git push rejection because the remote repository already contained changes. I synchronized the local and remote history before pushing again.

```bash
git pull --rebase origin main
git push origin main
```

**What I learned:** A rejected Git push does not always mean the project is damaged. Git may simply require the local repository to be synchronized with changes already present on GitHub.

<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/a09d18eb-ec59-40c3-946e-b52531d54777" />

---

# 4. Creating the MongoDB Atlas Database

The backend application required a MongoDB database. I configured **MongoDB Atlas** and created the `movies` database.

The backend application was configured to obtain the MongoDB connection string from an environment variable:

```properties
spring.data.mongodb.database=movies
spring.data.mongodb.uri=${MONGODB_URI}
```

Using an environment variable prevented the MongoDB connection credentials from being hardcoded directly into the application source code.

**What I achieved:** The database environment required by the Spring Boot backend was successfully prepared.

<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/3e89a358-85a5-4d7e-ab14-7eb182744c25" />


---

# 5. Verifying the Movie Data

Using MongoDB Atlas Data Explorer, I verified that the `movies` database contained the movie collection required by the application.

**What I achieved:** I confirmed that the database contained data that could later be retrieved through the Spring Boot API.

<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/6c51a05c-41f6-4868-95ac-d36ef1492b36" />


---

# 6. Creating Amazon ECR Repositories

I used **Amazon Elastic Container Registry (ECR)** to store the Docker images generated from the frontend and backend applications.

Two ECR repositories were used:

```text
movie-backend
movie-frontend
```

This separated the frontend and backend images and made it easier to manage different application versions.

**What I achieved:** I created a private AWS container registry for the application.

<img width="1280" height="718" alt="image" src="https://github.com/user-attachments/assets/079a814d-20c2-4ea6-8217-c364d425c12c" />


---

# 7. Configuring GitHub Actions Secrets

GitHub Actions needed permission to communicate with AWS.

I configured the required credentials as GitHub Actions repository secrets instead of writing sensitive values directly inside the workflow.

The workflow referenced secrets such as:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_REGION
```

**Security note:** Secret values must never be displayed inside the README.

**What I achieved:** GitHub Actions could authenticate securely with AWS.

<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/5eb9eb96-d00b-4706-a3c9-45b1573577c5" />
<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/a2dc35a9-2adb-4a97-b5f2-916c696d8641" />

---

# 8. Building the Backend CI Pipeline

I created a GitHub Actions workflow for the backend.

The workflow performed the following operations:

```text
GitHub Push
     ↓
Checkout Source Code
     ↓
Configure AWS Credentials
     ↓
Login to Amazon ECR
     ↓
Build Backend Docker Image
     ↓
Tag Docker Image
     ↓
Push Image to ECR
```

During this process, I encountered errors involving AWS region configuration, credentials, Docker tags, and ECR authentication.

I corrected the workflow until the `build-and-push` job completed successfully.

**What I learned:** CI/CD troubleshooting requires reading workflow logs carefully because a small configuration error can stop the entire pipeline.

<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/66a6d95f-2c76-4454-81ae-c9b2281c0e6d" />

---

# 9. Verifying the Backend Docker Image in ECR

After GitHub Actions completed successfully, I opened the `movie-backend` repository in Amazon ECR.

The generated backend image appeared with a version tag.

**What I achieved:** This confirmed that GitHub Actions successfully built the Docker image and transferred it to AWS ECR.

<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/9bef2fd0-6466-4b87-b8a1-95fce0892eba" />

---

# 10. Building the Frontend CI Pipeline

I configured a similar GitHub Actions workflow for the frontend application.

The frontend workflow:

```text
Checkout
   ↓
AWS Authentication
   ↓
ECR Login
   ↓
Docker Build
   ↓
Docker Tag
   ↓
Docker Push
```

After troubleshooting the workflow, the frontend image was successfully built and pushed.

<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/59d0ebc5-ecd9-46bb-b499-d5b664d4e4a4" />
<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/470770ea-6dbe-4e02-b053-4d73b86b0e83" />

---

# 11. Verifying the Frontend Image in ECR

I checked the `movie-frontend` ECR repository after the successful workflow.

The Docker image generated by GitHub Actions was available in the repository.

**What I achieved:** Both the frontend and backend application images were now stored in Amazon ECR.

<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/42ceb604-368c-45bd-873c-09203e1c3966" />

---

# 12. Preparing the AWS EC2 Server

I used an Ubuntu EC2 instance as the Docker host for the application.

Docker was installed on the server, and I connected to the instance through SSH.

Commands such as the following helped me verify the Docker environment:

```bash
docker --version
docker images
docker ps
```

**What I achieved:** My EC2 instance was ready to host Dockerized applications.

<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/9039690b-a633-462f-ae25-42cd62636780" />
<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/b059f687-f3cf-431b-934d-9e65cc9071c5" />

---

# 13. Configuring the EC2 IAM Role

Instead of storing permanent AWS credentials on EC2, I configured an IAM role that allowed the instance to retrieve images from ECR.

The role used for this purpose was attached directly to the EC2 instance.

**What I achieved:** EC2 received permission to access the private ECR repositories using AWS IAM.


<img width="1280" height="721" alt="image" src="https://github.com/user-attachments/assets/1cd89e40-777c-457d-93da-7516ae4c6d48" />
<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/075a761f-8920-42b3-963a-701db88e3dda" />

---

# 14. Installing AWS CLI on EC2

I installed AWS CLI on the Ubuntu EC2 server so that it could communicate with AWS services such as Amazon ECR.

<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/961597dc-5b19-4387-adb6-2bb83ec0bebb" />
<img width="1280" height="710" alt="image" src="https://github.com/user-attachments/assets/1e8fc459-a904-4267-abdb-aed968ab1f43" />

**What this screenshot proves:** It shows the AWS CLI installation process being performed from the Ubuntu EC2 terminal.

---

# 15. Verifying AWS CLI Installation

After installation, I verified that AWS CLI was available on the server.

This was important because the deployment process required AWS CLI to obtain an ECR authentication token.

<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/de988ebd-b667-45d5-abc2-83dba7e849b4" />


**What this screenshot proves:** AWS CLI was successfully installed and accessible from the EC2 environment.

---

# 16. Authenticating Docker with Amazon ECR

I authenticated Docker with Amazon ECR using AWS CLI.

The process followed this pattern:

```bash
aws ecr get-login-password --region us-east-2 | \
docker login --username AWS --password-stdin <ECR-REGISTRY>
```

After successful authentication, I pulled the frontend and backend images from ECR.

<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/b4936e49-deff-4593-93b8-c8e88cc749c3" />


**What this screenshot proves:** Docker successfully authenticated with Amazon ECR and the application images could be pulled onto EC2.

---

# 17. Starting the Backend and Connecting to MongoDB

I started the backend Docker container and inspected the application logs.

The Spring Boot logs confirmed that the backend started on port `8080` and connected to MongoDB Atlas.

Important log messages included:

```text
Tomcat started on port(s): 8080
Started MovieistApplication
```

<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/c8fdb730-2c61-4b4a-8714-70204a3baec0" />


**What this screenshot proves:** The Dockerized backend was running and communicating with MongoDB.

---

# 18. Testing Backend Server Availability

I tested the Spring Boot backend from the browser.

When I accessed the root `/` URL, Spring Boot displayed a **Whitelabel Error Page with 404**.

This did **not** mean the backend had failed. The application simply did not have a controller mapped to `/`.

The movie API was available at:

```text
/api/v1/movies
```
<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/65f7f4e0-515f-43cc-9e3e-7c09b325f0ab" />

**What this screenshot proves:** The Spring Boot server was reachable through port `8080`.

---

# 19. Deploying the Frontend Docker Container

After confirming that the backend was running, I deployed the frontend Docker container.

The frontend was exposed through port `3000`.

```text
Frontend → Port 3000
Backend  → Port 8080
```

<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/6789f2a5-98d1-447b-8436-fd3ea57700da" />

**What this screenshot proves:** The frontend container was deployed and running on the EC2 instance.

---

# 20. Testing the Complete Netflix Application

I opened the frontend application in the browser.

During testing, I discovered that the frontend was pointing to an old backend IP address.

I updated the API configuration to use the correct backend:

```javascript
import axios from 'axios';

export default axios.create({
    baseURL: 'http://18.223.186.155:8080',
    headers: {
        'Content-Type': 'application/json',
    },
});
```

I then rebuilt and redeployed the frontend image.

The movie information appeared successfully.

<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/2cc4011d-47fe-4a25-a0aa-cee409add15a" />

**What this screenshot proves:** The deployed frontend was successfully retrieving and displaying movie information.

---

# 21. Testing Movie Trailer Functionality

I also tested the trailer functionality from the deployed Netflix application.

<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/bd3acfea-853f-4d24-8084-2bd5152e9b8b" />

**What this screenshot proves:** The deployed application could successfully open and play a movie trailer.

---

# 22. Preparing Portainer

After successfully deploying the application, I introduced **Portainer Community Edition** to manage the Docker environment through a graphical interface.

<img width="1280" height="718" alt="image" src="https://github.com/user-attachments/assets/02b860ff-8a64-44cb-b5ec-471462f90e6f" />


**What this screenshot proves:** This was the preparation stage for installing Portainer CE.

---

# 23. Installing Portainer on EC2

I installed Portainer as another Docker container on the same EC2 server.

Portainer used persistent storage and exposed its web interface through port `9443`.

<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/2f47446a-028c-4c2f-8fe9-885c805f2cb2" />


**What this screenshot proves:** Portainer CE was installed and started on the EC2 Docker host.

---

# 24. Accessing the Portainer Web Interface

I accessed Portainer through HTTPS on port `9443`.

The browser initially displayed a certificate warning because Portainer was using its default/self-signed certificate.

<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/b5c034b0-fc44-4046-9794-738b63b107a6" />


**What this screenshot proves:** The Portainer HTTPS web interface was reachable from the browser.

---

# 25. Connecting Portainer to the Local Docker Environment

After opening Portainer, I connected it to the local Docker environment running on the EC2 instance.

<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/176b4cbb-1cc4-41e2-9d67-78a32cb227fb" />


**What this screenshot proves:** Portainer recognized the local Docker environment and was ready to manage it.

---

# 26. Managing the Docker Containers with Portainer

I opened the container section in Portainer.

The environment showed the main running containers:

```text
movie-backend
movie-frontend
portainer
```

<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/4bd8a937-00a6-4496-8dcc-6e7198abb88e" />


**What this screenshot proves:** The frontend, backend and Portainer containers were visible and manageable through the Portainer GUI.

---

# 27. Inspecting the Frontend Container

I selected the `movie-frontend` container to inspect its configuration and status.

Portainer provided options for actions such as:

```text
Logs
Inspect
Stats
Console
Restart
Stop
```

<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/d81b7904-e08e-4d1d-91b9-6da41e90c7f3" />


**What this screenshot proves:** Portainer could inspect and manage the running frontend container.

---

# 28. Monitoring Frontend Logs

I used Portainer's Logs feature to monitor requests being handled by the frontend application.

<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/35cb6179-5e70-4ab3-9c2f-6852f0ceb42c" />

**What this screenshot proves:** The frontend container was actively serving HTTP requests.

---

# 29. Inspecting the Backend Container

I also selected the `movie-backend` container in Portainer.

This allowed me to inspect its status, networking and container configuration from the graphical interface.

<img width="1280" height="717" alt="image" src="https://github.com/user-attachments/assets/4cfc511f-e2f0-493c-af6c-f346d12250fc" />


**What this screenshot proves:** The backend Docker container was running and could be managed through Portainer.

---

# 30. Monitoring Backend Logs

I opened the backend container logs through Portainer.

This demonstrated how Portainer can be used for troubleshooting and monitoring applications without relying entirely on terminal commands.

<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/9e78b00b-35b6-46b1-954c-7d0baea06910" />


**What this screenshot proves:** Portainer provided access to the Spring Boot backend container logs.

---

# 31. Moving from CI to CI/CD

At the beginning of the project, my GitHub Actions workflow stopped after building and pushing the Docker image:

```text
GitHub
   ↓
GitHub Actions
   ↓
Build
   ↓
Amazon ECR
```

This was **Continuous Integration (CI)**, but deployment to EC2 still required manual commands.

I therefore added a `deploy` job that depended on the successful completion of `build-and-push`.

The pipeline became:

```text
Code Push
    ↓
GitHub Actions
    ↓
Build Docker Image
    ↓
Push to Amazon ECR
    ↓
Deploy Job
    ↓
SSH into EC2
    ↓
Pull Latest Image
    ↓
Replace Existing Container
    ↓
Updated Application
```

<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/e55f4602-cf7a-472e-80d7-2954fe0ce115" />


**What this screenshot proves:** A deployment stage was added after `build-and-push`, converting the workflow into a CI/CD pipeline.

---

# 32. Successful Automated Deployment to EC2

Finally, I triggered the updated GitHub Actions workflow.

Both stages completed successfully:

```text
build-and-push  ✅
       ↓
deploy          ✅
```

The deployment job connected to EC2, pulled the newly generated image from ECR and replaced the existing application container.

<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/ab4f3dda-ecc1-400d-9259-090372768d70" />


**What this screenshot proves:** The frontend application was successfully deployed automatically from GitHub Actions to EC2.

---

# 🔧 Challenges and Troubleshooting

This project involved several real troubleshooting situations. Some of the major issues I encountered included GitHub permission errors, Git push conflicts, missing GitHub Actions AWS secrets, incorrect Docker tagging, ECR authentication problems, YAML configuration errors, AWS CLI PATH issues, MongoDB environment configuration, and the frontend pointing to an outdated backend IP address.

Working through these problems helped me understand how the different parts of a DevOps deployment pipeline depend on one another.

---

# 🔐 Security Considerations

Sensitive credentials were not intentionally stored directly in the source code. AWS credentials were handled using GitHub Actions Secrets, MongoDB used the `MONGODB_URI` environment variable, and the SSH private key required for automated deployment was stored as a secret.

For a production deployment, I would further improve the environment by restricting security-group ports, limiting MongoDB Atlas network access, using an Elastic IP or DNS name instead of a changing EC2 public IP, and serving the application over HTTPS through ports `80/443`.

---

# 🛠️ Technologies Used

| Technology     | Purpose                      |
| -------------- | ---------------------------- |
| Git            | Source-code version control  |
| GitHub         | Repository hosting           |
| GitHub Actions | CI/CD automation             |
| Docker         | Application containerization |
| Amazon ECR     | Docker image registry        |
| AWS EC2        | Application hosting          |
| AWS IAM        | Access control               |
| AWS CLI        | AWS command-line management  |
| MongoDB Atlas  | Cloud database               |
| Spring Boot    | Backend application          |
| React          | Frontend application         |
| Portainer      | Docker GUI management        |
| Ubuntu/Linux   | Server operating system      |

---

# 🎯 What I Learned

Through this project, I gained practical experience in Git and GitHub, Docker image and container management, GitHub Actions, AWS IAM, EC2, Amazon ECR, MongoDB Atlas, environment variables, Linux server administration, application logs, Portainer and CI/CD troubleshooting.

Most importantly, I learned the difference between simply creating a Docker image and implementing an actual deployment pipeline.

My initial workflow was:

```text
Build → Push to ECR
```

My final workflow was:

```text
Code Push → Build → ECR → Automated EC2 Deployment
```

---

# ✅ Final Result

The final application architecture successfully connected all the major components:

```text
GitHub
   ↓
GitHub Actions
   ↓
Amazon ECR
   ↓
AWS EC2
   ├── movie-frontend :3000
   ├── movie-backend  :8080
   └── Portainer      :9443
            │
            ▼
      MongoDB Atlas
```

The frontend successfully communicated with the backend, the backend connected to MongoDB Atlas, Docker containers were managed through Portainer, and GitHub Actions automated the build, push and deployment process.

---

## 👩🏽‍💻 Author

**Udemobi Chinecherem Blessing**

*Netflix Movie Project — Cloud & DevOps Practical Project*


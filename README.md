Python-App with AWS CI/CD 🚀


This is a simple Python (Flask) application (Made with Chagpt, I am not a pro in Pythonthon) that I deployed on AWS EC2 using Docker, Amazon ECR, and a GitHub Actions CI/CD pipeline.

The goal of this project is to learn how to:

Containerize a Python app with Docker

Push the image to Amazon ECR

Automate deployments using GitHub Actions

Run the app on AWS EC2

📌 Tech Stack

Python + Flask (web framework)

Docker (containerization)

Amazon ECR (container registry)

AWS EC2 (deployment server)

GitHub Actions (CI/CD pipeline)

⚙️ Project Setup
1️⃣ Clone the repository
git clone (https://github.com/abirall/Python-App.git)

2️⃣ Run locally with Docker
docker build -t python-app.
docker run -p 5000:5000 python-app


Then open: http://localhost:5000

🚀 Deployment Workflow

Push code to GitHub → triggers GitHub Actions

CI/CD builds Docker image → uploads to Amazon ECR

EC2 pulls the latest image → runs the container

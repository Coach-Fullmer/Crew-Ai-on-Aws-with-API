# Crew-Ai-on-Aws-with-API
### Repository Setup for CrewAI on AWS with Secure Secrets Management

#### Repository Name
`crewai-aws-api`

#### Repository Structure
```plaintext
crewai-aws-api/
├── .github/
│   └── workflows/
│       └── deploy.yml
├── .gitignore
├── README.md
├── requirements.txt
├── app.py
├── crewai_config.py
└── scripts/
    ├── setup_ec2.sh
    ├── deploy_flask.sh
    └── configure_aws_secrets.sh
```

### Explanation of Repository Structure

- **.github/workflows/deploy.yml**: GitHub Actions workflow file for automating deployment tasks.
- **.gitignore**: Specifies files and directories to be ignored by Git.
- **README.md**: Documentation for the project.
- **requirements.txt**: Lists the Python dependencies required for the project.
- **app.py**: The main Flask application file.
- **crewai_config.py**: Configuration file for CrewAI setup.
- **scripts/**: Contains shell scripts for setting up EC2, deploying Flask, and configuring AWS secrets.

### Step-by-Step Guide

#### 1. Create Repository

1. Go to GitHub and create a new repository named `crewai-aws-api`.
2. Initialize the repository with a `.gitignore` for Python.

#### 2. Add `.gitignore`
Create a `.gitignore` file with the following content:

```plaintext
# Ignore environment variables
.env

# Byte-compiled / optimized / DLL files
__pycache__/
*.py[cod]
*$py.class

# C extensions
*.so

# Distribution / packaging
.Python
build/
develop-eggs/
dist/
downloads/
eggs/
.eggs/
lib/
lib64/
parts/
sdist/
var/
wheels/
*.egg-info/
.installed.cfg
*.egg
MANIFEST

# Installer logs
pip-log.txt
pip-delete-this-directory.txt

# Unit test / coverage reports
htmlcov/
.tox/
.nox/
.coverage
.coverage.*
.cache
nosetests.xml
coverage.xml
*.cover
*.py,cover
.hypothesis/
.pytest_cache/

# Jupyter Notebook
.ipynb_checkpoints

# pyenv
.python-version

# Environments
.env
.venv
env/
venv/
ENV/
env.bak/
venv.bak/

# Spyder project settings
.spyderproject
.spyproject

# Rope project settings
.ropeproject

# mypy
.mypy_cache/
.dmypy.json
dmypy.json

# Pyre
.pyre/
```

#### 3. Create `requirements.txt`
List the necessary dependencies:
```plaintext
Flask
crewAI
boto3
python-dotenv
```

#### 4. Create `app.py`
Main Flask application file:
```python
from flask import Flask, request, jsonify
import os
from crewai import Agent, Task, Crew
import boto3
import json

app = Flask(__name__)

# Load AWS Secrets
def get_secret(secret_name):
    client = boto3.client('secretsmanager')
    response = client.get_secret_value(SecretId=secret_name)
    return json.loads(response['SecretString'])

secrets = get_secret('crewai-secrets')
api_key = secrets['API_KEY']

@app.route('/')
def home():
    return "Welcome to the CrewAI API!"

@app.route('/run_crew', methods=['POST'])
def run_crew():
    data = request.get_json()

    researcher = Agent(
        role='Senior Research Analyst',
        goal='Uncover cutting-edge developments in AI and data science',
        backstory="You work at a leading tech think tank."
    )

    writer = Agent(
        role='Tech Content Strategist',
        goal='Craft compelling content on tech advancements',
        backstory="You are a renowned Content Strategist."
    )

    task1 = Task(
        description=data['task1_description'],
        expected_output="Full analysis report",
        agent=researcher
    )

    task2 = Task(
        description=data['task2_description'],
        expected_output="Full blog post",
        agent=writer
    )

    crew = Crew(agents=[researcher, writer], tasks=[task1, task2], verbose=2)
    result = crew.kickoff()

    return jsonify(result)

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

#### 5. Create `crewai_config.py`
Configuration file for CrewAI:
```python
import os

class Config:
    SECRET_NAME = os.getenv('AWS_SECRET_NAME', 'crewai-secrets')

config = Config()
```

#### 6. Create Shell Scripts in `scripts/`
- **setup_ec2.sh**: Script to set up the EC2 instance.
- **deploy_flask.sh**: Script to deploy the Flask application.
- **configure_aws_secrets.sh**: Script to configure AWS Secrets Manager.

**setup_ec2.sh**:
```bash
#!/bin/bash
# Update and install necessary packages
sudo apt-get update
sudo apt-get upgrade -y
sudo apt-get install python3 python3-pip -y

# Install Docker
sudo apt-get install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker $USER
```

**deploy_flask.sh**:
```bash
#!/bin/bash
# Deploy Flask app
cd /path/to/your/app
pip3 install -r requirements.txt
python3 app.py
```

**configure_aws_secrets.sh**:
```bash
#!/bin/bash
# Configure AWS Secrets Manager
aws secretsmanager create-secret --name crewai-secrets --secret-string '{"API_KEY":"your_api_key_here"}'
```

#### 7. Add GitHub Actions Workflow in `.github/workflows/deploy.yml`
Automate deployment tasks using GitHub Actions:

```yaml
name: Deploy to EC2

on: [push]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Set up Python
      uses: actions/setup-python@v2
      with:
        python-version: '3.x'

    - name: Install dependencies
      run: |
        pip install boto3

    - name: Deploy to EC2
      run: |
        ssh -i "your-key-pair.pem" ec2-user@your-ec2-public-ip 'bash -s' < scripts/deploy_flask.sh
```

### Summary
1. **Repository Structure**: Organized to separate configuration, application logic, and deployment scripts.
2. **Secrets Management**: Using AWS Secrets Manager to securely manage secrets.
3. **Automated Deployment**: GitHub Actions workflow to automate deployment tasks.

By following this structure, you can maintain a secure, organized, and automated workflow for deploying and managing your CrewAI application on AWS.

Coach B Fullmer  
coachfullmer.com  
#Coach #CoachF

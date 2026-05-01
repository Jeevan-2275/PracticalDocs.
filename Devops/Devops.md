# DevOps Practical Exam Questions and Solutions

## Q1. Design Thinking – Digital Complaint System
**Scenario:** A college wants to improve its student complaint and grievance system.

**Solution / Implementation:**

Applying the five stages of Design Thinking:
1. **Empathise (Understanding Student Pain Points):**
   * **Action:** Conduct interviews, anonymous surveys, and focus groups with students who have previously filed complaints or felt frustrated by the current system.
   * **Goal:** Uncover real frustrations—such as long waiting times, lack of status updates, fear of retaliation (lack of anonymity), or confusing interfaces.
2. **Define (Frame the Exact Problem):**
   * **Problem Statement:** "Students need a secure, transparent, and easy-to-use digital platform to track their grievances in real-time because the current manual system is slow, lacks feedback, and causes a lack of institutional trust."
3. **Ideate (Generate Solutions):**
   * **Solution 1:** An anonymous chatbot integrated into the student portal for instant complaint lodging.
   * **Solution 2:** A mobile application with a ticketing system (like Jira or Zendesk) tailored for students.
   * **Solution 3:** A web dashboard showing real-time status of the complaint (e.g., Pending, In Progress, Resolved).
   * **Solution 4:** Automated SMS/Email alerts to keep students updated every time the status of their grievance changes.
   * **Solution 5:** A public (anonymized) dashboard showing average complaint resolution times by department.
4. **Prototype (Design a System/Interface):**
   * **Action:** Create wireframes and a clickable prototype (e.g., using Figma) of the mobile app or web portal. 
   * **Features:** It should include a simple "Lodge Complaint" button, an option to upload photo evidence, a clear tracking timeline, and a prominent "Anonymous Mode" toggle.
5. **Test (Validate Effectiveness):**
   * **Action:** Release a beta version to a small group (e.g., student council members). Track how intuitively they navigate the system. 
   * **Metrics:** Gather feedback on the UI/UX and validate if the tracking feature actively reduces their anxiety about the complaint status.

**How this improves transparency and institutional trust:**
By providing real-time tracking (similar to tracking an e-commerce package), students know their voices are being heard and acted upon. Anonymous options protect students, and automated updates prevent the feeling of complaints falling into a "black hole." This clear, accountable workflow fosters a culture of reliability and trust between the students and the administration.

---

## Q2. Agile – Bug Fixing & Maintenance Scenario
**Scenario:** You are working on a live application where users frequently report bugs and small feature requests.

**Solution / Implementation:**

**1. Agile Methodology for Handling Continuous Updates:**
For a live maintenance environment, **Kanban** is often the most effective methodology.
* **Workflow:** We will use a Kanban board with distinct columns: `Backlog`, `Selected for Development`, `In Progress`, `Code Review`, `Testing`, and `Done`.
* **WIP Limits:** We will set Work-In-Progress limits on the `In Progress` and `Testing` columns to ensure the team finishes current bug fixes before pulling new ones, preventing bottlenecks.

**2. Prioritizing Bugs vs. New Features:**
* Use a priority matrix based on **Severity/Urgency** for bugs and **Business Value** for features.
* **Critical/Blocker bugs** in production take immediate priority as hotfixes, bypassing standard sprint planning.
* Minor bugs and new features are groomed in the backlog. A standard capacity rule (e.g., 70% developer capacity for features, 30% for bugs/technical debt) can be applied during weekly planning sessions.

**3. Applying Test-Driven Development (TDD):**
* **Bug Scenario:** Users report that the application crashes if the "email address" field on the login page contains leading or trailing spaces.
* **TDD Test Case Idea (Before Solving):** Write a unit test that passes an email string with leading spaces (e.g., `"   user@example.com"`) to the authentication service. The test will assert that the function should successfully strip the spaces, authenticate the user, and return a success response without throwing an exception.
* **Execution:** Initially, this test will **fail** (Red). Then, the developer modifies the code to add `.trim()` to the input (Green). Finally, the code is optimized (Refactor).

**4. Applying Extreme Programming (XP) Practices:**
* **Continuous Integration (CI):** Every bug fix pushed by a developer triggers an automated pipeline that builds the app and runs all unit tests to ensure the fix didn't break existing functionality.
* **Quick Releases:** Utilize CI/CD pipelines to push small, incremental fixes to production rapidly (daily or multiple times a week) instead of waiting for large, risky monthly releases.
* **Feedback Loops:** Gather user feedback immediately after a release via crash analytics monitoring (e.g., Sentry) or quick user surveys to ensure the fix actually resolved the user's issue.

**5. Metrics to Track:**
* **Lead Time:** The time from when a bug or feature is reported by the user to when it is deployed in production.
* **Cycle Time:** The time from when a developer actively starts writing code for the ticket to when it is resolved and deployed.
* **Defect Resolution Rate:** The number of bugs successfully fixed versus the number of new bugs reported in a given timeframe.

---

## Q3. Jenkins CI/CD Pipeline with AWS Deployment
**Scenario:** You have a Spring Boot CRUD application hosted on GitHub. Deploy it to AWS EC2 using a Jenkins pipeline.

**Solution / Implementation:**

Below is a declarative `Jenkinsfile` that automates pulling the code, building via Maven, running tests, generating an artifact, and deploying it to an AWS EC2 instance.

```groovy
pipeline {
    agent any

    tools {
        // Ensure Maven and JDK are configured in Jenkins "Global Tool Configuration"
        maven 'Maven3'
        jdk 'JDK17'
    }

    environment {
        // Defined in Jenkins Credentials for SSH access to the EC2 instance
        EC2_CREDENTIALS_ID = 'aws-ec2-ssh-key' 
        EC2_USER = 'ec2-user'
        EC2_IP = '192.168.1.100' // Replace with actual EC2 Public/Private IP
        APP_NAME = 'springboot-crud-app.jar'
        DEPLOY_DIR = '/home/ec2-user/application'
    }

    stages {
        stage('Pull Code from Repository') {
            steps {
                echo 'Checking out source code from GitHub...'
                git branch: 'main', url: 'https://github.com/your-username/springboot-crud.git'
            }
        }

        stage('Build & Run Test Cases') {
            steps {
                echo 'Building using Maven and running unit tests...'
                // This command compiles the code, runs tests, and packages the artifact
                sh 'mvn clean package'
            }
            post {
                always {
                    // Archive JUnit test results
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Generate Build Artifact') {
            steps {
                echo 'Archiving the generated JAR artifact...'
                // Archive the generated JAR file from the target directory
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }

        stage('Deploy Application on AWS (EC2)') {
            steps {
                echo 'Deploying application to AWS EC2 via SSH...'
                sshagent([EC2_CREDENTIALS_ID]) {
                    // 1. Create deployment directory on EC2 if it doesn't exist
                    sh "ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_IP} 'mkdir -p ${DEPLOY_DIR}'"
                    
                    // 2. Copy the generated JAR artifact to the EC2 instance
                    sh "scp -o StrictHostKeyChecking=no target/*.jar ${EC2_USER}@${EC2_IP}:${DEPLOY_DIR}/${APP_NAME}"
                    
                    // 3. Stop the old application and start the new one
                    sh """
                    ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_IP} '
                        cd ${DEPLOY_DIR}
                        
                        # Find and kill the previously running Spring Boot application
                        pid=\$(pgrep -f ${APP_NAME}) || true
                        if [ -n "\$pid" ]; then kill -9 \$pid; fi
                        
                        # Start the new JAR application in the background
                        nohup java -jar ${APP_NAME} > application.log 2>&1 &
                    '
                    """
                }
            }
        }
    }
    
    post {
        success {
            echo "Pipeline executed successfully! Spring Boot App deployed to AWS EC2."
        }
        failure {
            echo "Pipeline failed. Please inspect the Jenkins console logs."
        }
    }
}
```

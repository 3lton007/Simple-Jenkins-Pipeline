# Simple-Jenkins-Pipeline
Elevate Labs DevOps Internship Task 2

## Implementation

- Used Docker hub to pull jenkins image and create jenkins locally.
- Fetched Jenkin Admin Access through gitbash using "docker exec -it <container>"
- Initiated Simple route based flask application with pytest cases.
- Pushed the application to github repository and configured the jenkins pipeline. 
- Issued Jenkinsfile to add stages like test, build and deploy. 
- As Jenkins, was implemented locally, the webhook for Github Hook trigger, had to be done through ngrok (Webhook Gateway) as Github Webhooks require public URL
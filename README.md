# Linux Server Configuration
A baseline installation of a Linux distribution on a virtual machine and prepare it to host the web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

## Project Description
- Public IP address: 54.173.124.133
- Acessible SSH port: 2200
- Application URL: http://54.173.124.133.xip.io/ 
- Linux version: Ubuntu 16.04.4 LTS
- virtual private server: [Amazon Lightsail](https://lightsail.aws.amazon.com)
- Deployed web application: [Item Catalog](https://github.com/blueblackw/item-catalog)
- Database server: [PostgreSQL](https://www.postgresql.org)

## Configuration Steps
### Step 1: Create an instance with Amazon Lightsail
1. Log into [Amazon Lightsail]((https://lightsail.aws.amazon.com)) with an Amazon Web Service account. If you don't already have an Amazon Web Services account, you'll be prompted to create one.
2. Follow the 'Create an instance' link.
3. Under "Create an instance", select platform as " Linux/Unix" and select a blueprint as "OS Only". Then under the options "OS Only", click to select "Ubuntu" as the operating system. 
4. Choose the payment plan. Name the instance in the textbox on the bottom of the page. Click "Create".
5. Wait for the instance to start up.
#### Reference 
- [ServerPilot](https://serverpilot.io/community/articles/how-to-create-a-server-on-amazon-lightsail.html)
   



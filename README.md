# jenkins-pipeline-aws
Deploy to AWS using Jenkins 

## Summary of Task ##

Set up one click deploy and provisioning of an environment with all neccessary elements of a DevOps Toolchain.

**Description**

Use Puppet / Chef / Ansible for the following setup

1. Apache tomcat server :white_check_mark: 
2. Mysql Database, with configuration controlled through the tool
3. Apache Http Webserver :white_check_mark: 
4. Web loadbalancer on Apache Server for Tomcat. 
5. Jenkins for CI :white_check_mark: 
 
With the setup in place:

1. Make a continuous delivery (CD) pipeline using Jenkins, it should include CI Builds and other jobs neccessary for the software delivery lifecycle :white_check_mark: 
2. Create a DevOps Toolchain to completely automate the pipeline :white_check_mark: 
3. Push a built WAR using Jenkins build pipeline into the VM :white_check_mark: 
4. Also make sure that the location of tomcat and apache HTTPD should be flexible and controlled by Puppet/Chef/Ansible, in case no specific value is provided it should fall back to defaults :white_check_mark: 
 
**NOTES:**
 
1. You can make any assumptions and be as innovative and creative as possible in your usage of tools for DevOps tool-chain
2. You are expected to implement a CD pipeline with no use of shell scripts
3. Check-in the complete project (cookbooks, manifests, Jenkins build definitions etc.) into a GitHub account and send us the repository location
4. Use the spring application https://github.com/spring-projects/spring-petclinic/ as source for the CI And CD implementations
5. Feel free to use AWS and share the working installation URL also.
6. Recommended tool for AWS : Vagrant

<a name="workingdeployment"/>
#### Working Deployment

**Notes**

1. Due to time constraints I was unable to complete the following
  1. Web load balancer  --working


The flow is as follows:

1. The job is set to poll the GitHub repository once every day.
2. Upon detecting changes it executes the jenkins pipeline.
3. The JUnit test results get parsed as a post-build step.
4. Upon successful build, the WAR is deployed to Tomcat.

Here is where some key Ansible-related files are located:

```
ansible-tomcat-mysql
├── AddingKeys
│   ├── group_vars
│   │   └── all
│   ├── site.yml
│   └── users
│       ├── public_keys
│       │   └── k.pub
│       └── tasks
│           └── main.yml
├── aws-private.pem
├── CreatingEC2
│   ├── create
│   │   └── tasks
│   │       └── main.yml
│   ├── group_vars
│   │   └── all
│   ├── mysql
│   │   ├── files
│   │   │   ├── sakila-data.sql
│   │   │   └── sakila-schema.sql
│   │   ├── handlers
│   │   │   └── main.yml
│   │   └── tasks
│   │       └── main.yml
│   ├── site.retry
│   ├── site.yml
│   └── tomcat
│       ├── handlers
│       │   └── main.yml
│       ├── tasks
│       │   └── main.yml
│       ├── templates
│       │   ├── index.html
│       │   └── tomcat-users.xml
│       └── vars
│           └── main.yml
├── ELB
│   ├── group_vars
│   │   └── all
│   ├── site.yml
│   └── web
│       ├── handlers
│       │   └── main.yml
│       ├── static_files
│       │   └── index.html
│       └── tasks
│           └── main.yml
├── hosts
└── README.md

```


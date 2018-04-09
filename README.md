Deploy to AWS using Jenkins pipeline

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
jenkins-pipeline-aws
├── ec2.ini
├── ec2-lauch.yml
├── ec2.py
├── README.md
├── roles
│   ├── common
│   │   └── tasks
│   │       └── main.yml
│   ├── ec2
│   │   ├── tasks
│   │   │   └── main.yml
│   │   └── vars
│   │       ├── haproxy.yml
│   │       ├── main.yml
│   │       └── tomcat.yml
│   ├── haproxy
│   │   ├── files
│   │   │   └── haproxy_default
│   │   ├── handlers
│   │   │   └── main.yml
│   │   ├── tasks
│   │   │   └── main.yml
│   │   ├── templates
│   │   │   └── haproxy.cfg.j2
│   │   └── vars
│   │       └── main.yml
│   ├── nagios
│   │   ├── files
│   │   │   └── nagios.cfg
│   │   ├── handlers
│   │   │   └── main.yml
│   │   ├── tasks
│   │   │   └── main.yml
│   │   ├── templates
│   │   │   └── nrpe.cfg.j2
│   │   └── vars
│   │       └── main.yml
│   ├── nagios-slave
│   │   ├── handlers
│   │   │   └── main.yml
│   │   ├── tasks
│   │   │   └── main.yml
│   │   └── templates
│   │       └── nrpe.cfg.j2
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
└── site.yml


```

# Ansible Scalable and Load Balanced (SLB) Tomcat

An ansible playbook to deploy on Amazon AWS a scalable set of instances of Apache Tomcat 7 container proxied via a “load-balancing purpose” instance driven by HAProxy, as a bonus I  Nagios3 on all the instance, so even an example of infrastructure monitoring is given.

So the ansible playbook will launch:

* 2 EC2 micro instances tagged as *tomcat*
    * Security Group with port 8080 and 22
    * Tomcat 7 installed via apt

* 1 EC2 micro instances tagged as *harpoxy*
    * Security Group with port 80, 8888, 1936 and 22
    * HAProxt and Socat installed via apt
    * the instance will proxy and balance (using Round-Robin strategy) from port 8888 to the Tomcat instances
    * an HAproxy amdin interface is available on port 1936

## Prerequisites
* [Ansible](http://docs.ansible.com/intro_installation.html)
* an account on [Amazon AWS](http://aws.amazon.com/) and a corresponding IAM role

## Configuration

###1. AWS Credentials
Export system env variables for Amazon AWS IAM role credentials like the following:

        export ANSIBLE_HOST_KEY_CHECKING=False
        export AWS_ACCESS_KEY_ID='AK123'
        export AWS_SECRET_ACCESS_KEY='abc123'

or put the previous commands in a file (e.g. secret_iam.env) and launch form terminal:

        source secret_iam.env

###2. AWS regions, vpc, subnets

Fill the configuration file under ec2/var/main.yml

    keypair: <<your keipair name>>
    region: us-east-1
    instance_type: t2.micro
    image: ami-9eaa1cf6 # Ubuntu Server 14.04 LTS (HVM), SSD Volume Type
    tag_ec2_current: launched
    subnets:
    - subnet-fbe306d0
    vpc_id: vpc-9ed752fb

All the ec2 instances will be configured starting from this file.

## Running

Now you are able to launch the playbook

    ansible-playbook -i ec2.py --private-key=<path_to_your_aws-pem_key ec2-lauch.yml  -v

## Notes

* Templates and playbooks are massively compiled from variables yaml files. Remeber to check them before installing your inventory.

## Known Issues

* Launching the playbook for the first time may fail resulting in *AnsibleUndefinedVariable* for some values of ec2 tags (*ec2_tag_Name*). Running the same for a new attempt will succeed silently.


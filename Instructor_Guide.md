Ansible Workshop Instructor's Guide
=========================================

In addition to having some core Ansible and Ansible Tower skills, hosting a successful Ansible workshop will require a number of prerequisites be met to build and host the workshop infrastructure and student guides.  For the experienced, this may only take a few hours but for the newly initiated, it may take a bit longer.  This guide is designed to help an Instructor create all of the tools/access required to host a workshop.  These instructions can be broken down into several categories, each of which will be expanded upon in this guide.

* Access to an AWS account, keys, and some working knowledge of EC2, S3, and Route53
* Cloning and updating student documentation and slides
* Hosting student documentation and slides
* Building AWS instances for students

## Access to an AWS account

1. Gain access to the redhatgov.io https://us-east-1.signin.aws.amazon.com Amazon AWS account -  Shawn Wells is the owner/maintainer of this account.
2. An existing administrator (Shawn or other admin) will need to add you as a user to IAM with the following:
        - RHTGOV_ADMINS group
3. Once account is added to this group, you will be able to login and configure your account further.
        - Using the IAM service, configure an "Assigned MFA Device" (your phone probably)
        - Create an "Access Key"
            - This key will contain an "access key" and a "secret key" giving you the option to download a .csv with the same information.  This is the only time you'll be able to retain that key, i.e. don't lose it.  You'll be using this key later on your laptop during the aws_lab_setup playbook.
 4. Navigate to the EC2 service
        - Generate your own keypair and place the <keypair>.pem file into your ~username/.ssh/ directory (This is the ssh key that will enable you to perform post-provisioning ansible tasks on EC2 instances from your laptop to the redhatgov.io AWS account, as well as login to the instances you've created)

## Hosting student documentation and slides

* A general student guide and instructor slides are already hosted at http://ansible-workshop.redhatgov.io
* Here you will find complete student instructions for each exercise as well as the presentation decks under the "Resources" drop down.
* During the workshop, it is recommended that you have a second screen or printed copy of both.  Previous workshops have demonstrated that unless you've memorized all of it, you'll likely need to refer to these, but your laptop will most likely be projecting.  Some students will fall behind and you'll need to refer back to other exercises/slides without having to change the projection for the entire class.

### NOTE:  If you need to modify existing documentation

 1. Email bhirsch@redhat.com to either have a new branch created or be added as contributor to https://github.com/bhirsch70/Ansible_Tower_Workshop
 2. Spend a little time with the README to understand this project's structure and asciibinder
 3. git clone -b <your_branch>  https://github.com/bhirsch70/Ansible_Tower_Workshop
 4. Use Atom or another asciidoctor-aware IDE to edit your workshop content
    * You should primarily be working with Ansible_Tower_Workshop/[workshop/*|_topic_map.yml]
    * Once editing is complete, commit, push, and render
    
        `git commit workshop -m "<insert change log comment>"`
        
        `git push`
        
        - install asciibinder - http://www.asciibinder.org/
        - From your local project copy of Ansible_Tower_Workshop...
        - asciibinder clean
        - asciibinder
        - Your rendered html will be your local copy of Ansible_Tower_Workshop/preview/
    - Navigate to the S3 service
        - Looking at existing S3 buckets in AWS, copy the ansible-tower.redhatgov.io bucket
        - Edit Permissions-->Bucket Policy for the ansible-tower.redhatgov.io bucket and copy the content
        - Cancel the Editing process for that bucket
        - Select your new S3 bucket and Edit Permission-->Add Bucket Policy and paste previously copied content
        - Edit the "Resource:" with the new bucket name and Save
        - Edit Properties-->Static Website Hosting and select "Use this bucket to host a website" and enter "index.html" under index document.  Save
        - Because you made a copy of the ansible-tower-redhatgov.io bucket, you should have the following directories in your bucket
            - _images
            - decks
            - standard
            - workshop-files
            - index.html
        - Delete the folder called "standard"
        - Click the "upload" button, and upload the contents from your Ansible_Tower_Workshop/_preview/red-hat-public-sector/* folder.  This is your recently edited, html content.
        - Back in your local git clone, modify the Ansible_Tower_Workshop/index.html to point to your new bucket and do the git stuff to it (commit, push)
        - In your S3 bucket, delete the existing index.html file and upload a copy of the new one
    - Navigate to the AWS route53 service
        - Select Hosted Zone
        - Select redhatgov.io
        - Create a record set
            - Name = your bucket name minus the .redhatgov.io
            - Alias = Yes
            - Alias Target = being typing "s3-website" and it should prepopulate, but the result should be s3-website-us-east-1.amazonaws.com.
            - Save your record set
        - Now, you should be able to navigate to http://<whatever you named your bucket> and see your student guides.  If not, hmm... Most likely are A) Route53 screwup, B) index.html screwup, C) Bucket Policy screwup or you forgot to do step - enable "Use this bucket to host a website"

Building AWS instances for students

- If you don't already have it, get yourself setup with git on your laptop. https://git-scm.com/book/en/v2/Getting-Started-Installing-Git
- git clone lightbulb https://github.com/ansible/lightbulb
- Read EVERYTHING.  The goal here is not to rewrite instructions, so it's important to read about Lightbulb, it's philosophy, and intent before attempting to build and run a workshop.
- Follow lightbulb instructions for the aws_lab_setup playbook.
    - Modifications to aws_lab_setup need to be made for this specific workshop and are as follows:  These changes were submitted to the lightbulb project by gdykeman, but to my knowledge, they still have not been merged.
        - Create an "Instances" directory under the lightbulb/aws_lab_setup/ subdirectory.  This is more organizational than anything else as the playbook will otherwise dump inventories for every student directly in the aws_lab_setup directory and that can get messy.
            - Once you create this directory, you must modify the lightbulb/tools/aws_lab_setup/roles/manage_ec2_instances/tasks/create.yml file, to incorporate that directory 'generate student inventory' and 'generate instructor inventory' tasks for the 'dest:' parameter.

        - Create a third inventory type that is  appropriate for email.  Currently, lightbulb emails the student's inventory to them after the ec2 instances are built, but it includes the instances username and password, which is a security concern.  This has also been submitted to the project as an issue.
            - modify the lightbulb/tools/aws_lab_setup/roles/manage_ec2_instances/tasks/create.yml file, and duplicate the 'generate student inventory' task.  Rename the duplicate task 'generate student inventories for email' and modify the 'src:' with 'instances-nopass.txt.j2'
            - copy the 'lightbulb/aws_lab_setup/roles/manage_ec2_instances/templates/instance.txt.j2'  to 'instances-nopass.txt.j2' in the same directory.
            - Remove the second line 'ansible_ssh_pass={{ admin_password }}' and save that new file
            - Modify the 'lightbulb/aws_lab_setup/roles/email/tasks/tasks.yml' file in the following ways
                - Remove the reference to the "password" in the body to read something like "... and the password will be given during the workshop."
                - Modify the attachment configuration to attach the 'instances-nopass.txt.j2'
- Follow the remaining lightbulb instructions, i.e. extra_vars.yml and users.yml
- Once you launch 'ansible-playbook provision_lab.yml -e @extra_vars.yml -e @users.yml' pay close attention.  We've had failures before.  In fact, I recommend adding 'email: no' into you extra_vars.yml file initially so that your students are not sent an email.  Then, once you've successfully deployed all instances and they are fully configured, change 'email: yes' and rerun the playbook.  The reasoning is that during one of our workshop preps, some of the hosts were provisioned, but had to be destroyed and new ones created.  This resulted in some student receiving multiple emails.

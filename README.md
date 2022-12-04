# ANSIBLE-REFACTORING
Jenkins CI/CD on a 3-tier application && Ansible Configuration Management Dev and UAT servers using Static Assignments

Ansible Refactoring and Static Assignments (IMPORTS AND ROLES)

In the previous CI-CD Ansibile Jenkins project, I implemented CI/CD and Configuration Managment solution on the Development Servers using Ansible 

In this project, I will be extending the functionality of this architecture and introducing configurations for UAT environment.

![jenkinsansiblearc](https://user-images.githubusercontent.com/61475969/205504382-e09a5aa0-be28-477a-8075-b0d438678a05.png)

STEP 1 - Jenkins Job Enhancement/ Refactoring

Download Copy Artifacts plugin on jenkins. This will help us copy all our artifacts to a specific directory in our Jenkins server
Go to Jenkins web console -> Manage Jenkins -> Manage Plugins -> on Available tab search for Copy Artifact and install this plugin without restarting Jenkins

Make a directory inside the root directory.

<img width="1398" alt="Screenshot 2022-12-04 at 17 05 14" src="https://user-images.githubusercontent.com/61475969/205504921-cb2b4a33-dc1a-4190-8d3a-86f3ddacf88a.png">

<img width="1398" alt="Screenshot 2022-12-04 at 17 07 38" src="https://user-images.githubusercontent.com/61475969/205504968-2055e81d-494a-4ecc-b184-aa184cd433cb.png">

On the Jenkins-Ansible server, create a new directory called ````ansible-config-artifact```` by running the command below;

```sudo mkdir /home/ubuntu/ansible-config-artifact```

Change permission of the directory

```sudo chmod -R 0777 /home/ubuntu/ansible-config-artifact```

<img width="569" alt="Screenshot 2022-12-04 at 17 15 05" src="https://user-images.githubusercontent.com/61475969/205505350-f91c2658-423c-43c3-bb62-915c7dae074f.png">

Create a new Freestyle project and name it save_artifacts.

This project will be triggered by completion of your existing ansible project. Configure it accordingly:

   - Configure the directory where you want to copy your artifacts.
   - create a Build step and choose Copy artifacts from other project
   - Specify ansible as a source project and /home/ubuntu/ansible-config-artifact as a target directory.

<img width="719" alt="Screenshot 2022-12-04 at 17 30 19" src="https://user-images.githubusercontent.com/61475969/205506127-3109f46b-68c8-4bb0-88b2-59dcbb87cd9f.png">

<img width="719" alt="Screenshot 2022-12-04 at 17 30 44" src="https://user-images.githubusercontent.com/61475969/205506142-3506742b-b96b-4880-8aa4-4e59ef49ce5f.png">

We configured the number of build to 2. This is useful because whenever the jenkins pipeline runs, it creates a directory for the artifacts and it takes alot of space. By specifying the number of build, we can choose to keep only 2 of the latest builds and discard the rest.

Test your set up by making some change in README.MD file inside your ansible-config-mgt repository (right inside master/main branch).

If both Jenkins jobs have completed one after another – you shall see your files inside /home/ubuntu/ansible-config-artifact directory and it will be updated with every commit to your master branch.

Now your Jenkins pipeline is more neat and clean.




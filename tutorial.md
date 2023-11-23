# Deploying a Sandbox Django Application

[Django](https://www.djangoproject.com/) is a high-level Python web framework that encourages rapid development and clean, pragmatic design. Built by experienced developers, it takes care of much of the hassle of web development, so you can focus on writing your app without needing to reinvent the wheel. It’s free and open source.

[Acorn](http://www.acorn.io) is a cloud computing platform with a big free sandbox that anyone can use by registering with a GitHub account. It is designed to simplify running modern  cloud-native apps on the public cloud. You use familiar development and deployment workflows based on mainstream container tools without having to deal with provisioning or configuring any underlying cloud resources. Basically it provides all the power of Kubernetes and Terraform, without any of the complexity.

To deploy an application on Acorn we need to define our application as an Acornfile, which will produce the Acorn Image that we can deploy on the platform.  In this tutorial, we will explore how to provision a sample Django Application on Acorn.


If you’re the kind of person who likes to skip to the end, you can [deploy the sample application in your sandbox now](https://acorn.io/run/ghcr.io/infracloudio/django-acorn:v4.%23.%23-%23?ref=slayer321&name=django) and just start poking around in it.  Sandbox deployments in Acorn are restricted by size, and run for two hours, so it should provide plenty of time for you to evaluate and test anything. You can start them over as often as you like, or you can upgrade to a paid Pro account if you want to run something in production. 

If you want to follow along, I’ll walk through the steps to deploy the sample Vote  App using Acorn, described in the Django [official documentation](https://docs.djangoproject.com/en/4.2/intro/tutorial01/).

_Note: Everything shown in this tutorial can be found in [this repository](https://github.com/infracloudio/django-acorn.git)_.


## Pre-requisites

- [Acorn CLI](https://docs.acorn.io/installation/installing)
- Github account to sign up for the Acorn Platform.

## Acorn Login
Login to the [Acorn Platform](http://beta.acorn.io) using the Github Sign-In option with your Github user.
![](./assests/acorn-login-page.png)

After the installation of Acorn CLI for your OS, you can login to the Acorn platform.
```
$ acorn login beta.acorn.io
```

## Create the Django Application
In this post we will create a simple Django Vote Poll app from the [official documentation](https://docs.djangoproject.com/en/4.2/intro/tutorial01/). It is a simple Vote Poll application that provides standard CRUD features that let people view polls and vote in them. An admin site that lets you add, change, and delete polls.

In the Acorn platform, there are two ways you can try this sample application.
1. Using Acorn platform dashboard.
2. Using CLI


The First way is the easiest one where, in just a few clicks you can deploy the Vote Poll application on the platform and start using it. However, if you want to customize the application or want to understand how you can run your own RoR applications using Acorn, use the second option.


## Running the application using Dashboard

In this option you use the published Acorn application image to deploy the Vote Poll sample application in just a few clicks. It allows you to deploy your applications faster without any additional configurations. Let us see below how you can deploy the articles app to the Acorn platform dashboard.

1. Login to the [Acorn Platform](https://acorn.io/auth/login)  using the Github Sign-In option with your Github user.
2. Select the “Create Acorn” option.
3. Choose the source for deploying your Acorns
   3.1. Select “From Acorn Image” to deploy the sample Application.
![](./assests/select-from-acorn-image.png)

   3.2. Provide a name "Django Sample Acorn”, use the default Region and provide the URL for the Acorn image and click Create.
```
ghcr.io/infracloudio/django-acorn:v4.#.#-#
```
![](./assests/django-app-deployment-preview.png)

_Note: The App will be deployed in the Acorn Sandbox Environment. As the App is provisioned on AcornPlatform in the sandbox environment it will only be available for 2 hrs and after that it will be shutdown. Upgrade to a pro account to keep it running longer_.

4. Once the Acorn is running, you can access it by clicking the Endpoint or the redirect link.
   4.1. Running Application on Acorn
   ![](./assests/django-running-on-platform.png)
   4.2. Running Vote app
   ![](./assests/django-vote-app.png)


## Running the Application using acorn CLI
As mentioned previously, running the acorn application using CLI lets you understand the Acornfile. With the CLI option, you can customize the sample app to your requirement or use your Acorn knowledge to run your own Django application.

To run the application using CLI you first need to clone the source code repository on your machine.


```
$ git clone https://github.com/infracloudio/django-acorn.git
```
Once cloned here’s how the directory structure will look.

![](./assests/django-root-dir.png)

### Understanding the Acornfile

We have the sample Django Application ready. Now to run the application we need an Acornfile which describes the whole application without all of the boilerplate of Kubernetes YAML files. The Acorn CLI is used to build, deploy, and operate Acorn on the Acorn cloud platform.  It also can work on any Kubernetes cluster running the open source Acorn Runtime. 


Below is the Acornfile for deploying the Vote Poll app that we created earlier:

![](./assests/django-acorn-file.png)

There are 2 requirements for running Ruby on Rails Application
- Application Itself
- DB

The above Acornfile has the following elements:

- **Args**: Which is used to take the user args.
- **Services**: Here we're using the [MariaDB](https://github.com/acorn-io/mariadb) service that is built into Acorn as an [Acorn Service](https://docs.acorn.io/reference/services).
- **Containers**: We define a single container named web and define the following configurations:
  - **build**: details required to build the Django Application
  - **ports**: port number the application is listening on and we are also specifying [publish field](https://docs.acorn.io/reference/acornfile#ports) which will give the url to access the application.
  - **env**: In the env section we are providing all the env variables which the application will be using . And this env variable will override all the values that you have specified inside the settings.py file.


### Running the Application
We have already logged in using Acorn CLI now you can directly deploy applications on your sandbox on the Acorn platform. Run the following command from the root of the directory.

```
$ acorn run -n django-app -i
```

Below is what the output looks like.

![](./assests/django-local-run.png)

## The Vote Poll Application

As it is a simple Vote Poll application it provides CRUD features. It’ll consist of two parts:
- A public site that lets people view polls and vote in them.
- An admin site that lets you add, change, and delete polls.

On the homepage you will get the admin credentails which can be used to login as admin and you can add question for your vote app. 

![](./assests/django-vote-app.png)

## Running the app in dev mode

If you are developing your application and don't want to start and stop the application everytime you make the changes you can use the incredibly helpful Acorn Dev Mode.    Running in dev mode allows you to do continous development of the application, and have your running instance reflect the changes as soon as you save your changes. Run the following command.

```
$ acorn dev -n django .
```

## Push an artifact to a registry



Using the Dev mode you can easily modify the application as per your requirement and once the application is working as expected and is ready to be built and packaged, push it to a registry. You can push it to any OCI registry. We will use Github Container Registry in this example. Once published, you can use the acorn image to deploy it directly in the Acorn platform using the dashboard or CLI, as described previously.

### Log in to the registry
Log in to the registry with the command below and follow the prompts.

```
$ acorn login ghcr.io
```
### Build and push the image
Build and push the image with the below command.

```
$ acorn build --push -t ghcr.io/infracloudio/django-acorn:v4.2.6-1
```
Once the application is built and pushed you can use those images to run your application on Acorn Platform.

## What's Next?

1. The App is provisioned on Acorn Platform and is available for two hours. Upgrade to Pro account for anything you want to keep running longer.
2. After deploying you can edit the Acorn Application or remove it if no longer needed. Click the Edit option to edit your Acorn's Image. Toggle the Advanced Options switch for additional edit options.
3. Remove the Acorn by selecting the Remove option from your Acorn dashboard.

## Conclusion
In this tutorial we show how we can use the Acornfile and get our Django application up and running and it’s very easy to make changes to the Application file when you are developing it without the need of restarting your application. And If you are looking to run the application directly you can run it on Acorn Platform by providing the image name. 

# Deploying a Sandbox Django Application

[Django](https://www.djangoproject.com/) is a high-level Python web framework known for its rapid development and clean design. Meanwhile,  [Acorn](http://www.acorn.io) offers a cloud computing platform, providing a free sandbox accessible with a GitHub account. Acorn simplifies running cloud-native apps on the public cloud, utilizing familiar development workflows without the need for provisioning or configuring underlying cloud resources. With Django's hassle-free web development and Acorn's streamlined cloud deployment, developers can focus on building robust applications without reinventing the wheel.


[Acorn](http://www.acorn.io) is a lightweight deployment framework that makes developers productive and efficient by encapsulating both the application and its associated dependencies, including cloud services, within a singular [Acornfile](https://docs.acorn.io/reference/acornfile). In this tutorial, we will deploy a sample Django application from the [official documentation](https://docs.djangoproject.com/en/4.2/intro/tutorial01/) on the Acorn cloud platform.This Django app is sample vote application.


If you want to skip to the end, just click [Run in Acorn](https://acorn.io/run/ghcr.io/infracloudio/django-acorn:v4.%23.%23-%23?ref=slayer321&name=django) to launch the app immediately in a free sandbox environment. All you need to join is a GitHub ID to create an account.

_Note: Everything shown in this tutorial can be found in [this repository](https://github.com/infracloudio/django-acorn.git)_.


## Pre-requisites

- Acorn CLI: The CLI allows you to interact with the Acorn Runtime as well as Acorn to deploy and manage your applications. Refer to the [Installation documentation](https://docs.acorn.io/installation/installing) to install Acorn CLI for your environment.
- A GitHub account is required to sign up and use the Acorn Platform.

## Acorn Login
Log in to the [Acorn Platform](http://beta.acorn.io) using the GitHub Sign-In option with your GitHub user.
![](./assests/acorn-login-page.png)

After the installation of Acorn CLI for your OS, you can login to the Acorn platform.
```
$ acorn login beta.acorn.io
```

## Create the Django Application
In this tutorial we will create a simple Django Vote Poll app from the [official documentation](https://docs.djangoproject.com/en/4.2/intro/tutorial01/). It is a simple Vote Poll application that provides standard CRUD features that let people view polls and vote in them. An admin site that lets you add, change, and delete polls.

In the Acorn platform, there are two ways you can try this sample application.
1. Using Acorn platform dashboard.
2. Using CLI


The first way is the easiest one where, in just a few clicks you can deploy the Vote Poll application on the platform and start using it. However, if you want to customize the application or want to understand how you can run your own Django applications using Acorn, use the second option.


## Deploying Using Acorn Dashboard

In this option you use the published Acorn application image to deploy the Vote Poll sample application in just a few clicks. It allows you to deploy your applications faster without any additional configurations. Let us see below how you can deploy the vote app to the Acorn platform dashboard.

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


## Deploy Using Acorn CLI
As mentioned previously, running the acorn application using CLI lets you understand the Acornfile. With the CLI option, you can customize the sample app to your requirement or use your Acorn knowledge to run your own Django application.

To run the application using CLI you first need to clone the source code repository on your machine.


```
$ git clone https://github.com/infracloudio/django-acorn.git
```
Once cloned here’s how the directory structure will look.

```
.
├── Acornfile
├── django.svg
├── LICENSE
├── mysite
│   ├── db-script.sh
|   |.....
│   └── requirements.txt
└── README.md
```

### Understanding the Acornfile

We have the sample Django Application ready. Now to run the application we need an Acornfile which describes the whole application without all of the boilerplate of Kubernetes YAML files. The Acorn CLI is used to build, deploy, and operate Acorn on the Acorn cloud platform.  It also can work on any Kubernetes cluster running the open source Acorn Runtime. 


Below is the Acornfile for deploying the Vote Poll app that we created earlier:

```
args: {
  djangodbname: "djangodb"
}

services: db: {
  image: "ghcr.io/acorn-io/mariadb:v10.11.5-1"
    serviceArgs: {
      dbName: args.djangodbname
  }
}
containers: web: {
  build: {
    context:    "./mysite"
    dockerfile: "./mysite/Dockerfile"
	}
  ports: publish: "8000:8000/http"
	if args.dev {dirs: "/app": "./mysite"}
	env: {
    MARIADB_USER:          "@{service.db.secrets.admin.username}"
    MARIADB_ROOT_PASSWORD: "@{service.db.secrets.admin.password}"
    MARIADB_HOST:          "@{service.db.address}"
    MARIADB_PORT:          "@{service.db.port.3306}"
    MARIADB_DATABASE:      "@{service.db.data.dbName}"
	}
}
```

There are 2 requirements for running Django Application
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
In this tutorial we saw how we can use the Acornfile and get our Django application up and running and it’s very easy to make changes to the Application file when you are developing it without the need of restarting your application. And If you are looking to run the application directly you can run it on Acorn Platform by providing the image name. 

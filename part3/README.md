*Quick links :*
[Home](/README.md) - [Part 1](../part1/README.md) - [Part 2](../part2/README.md) - [**Part 3**](../part3/README.md) - [Part 4](../part4/README.md) - [Part 5](../part5/README.md)
***

# Part 3: Deploy you application to the IBM Cloud

**Goal:** *Get your application running in the IBM Cloud*

## Introduction

This section adds some additional resources to your Node-RED project so that it
can be deployed to the IBM Cloud.

This involves:

 - adding some extra properties to the project's `package.json` file,
 - adding a `settings.js` file
 - adding a `manifest.yml` file

## Steps

 - [1 - Update package.json](#31---update-packagejson)
 - [2 - Add a settings.js file](#32---add-a-settingsjs-file)
 - [3 - Add a manifest.yml file](#33---add-a-manifestyml-file)
 - [4 - Push the application to the cloud](#34---push-the-application-to-the-cloud)

## 3.1 - Update package.json

The project already has a default `package.json` file that needs some updates:

The project's `package.json` file can be found at `~/.node-red/projects/<name-of-project>/package.json`. Open that file in a text editor and make the following changes:


 1. add `node-red` in the dependencies section - this will ensure Node-RED gets installed when the application is deployed. Note you should already have `node-red-contrib-cloudantplus` listed.

         "dependencies": {
             "node-red": "1.x",
             "node-red-contrib-cloudantplus": "0.2.0"
         },


 2. Add a `scripts` section to define a `start` command - this is how IBM Cloud will run the application.

         "scripts": {
             "start": "node --max-old-space-size=160 ./node_modules/node-red/red.js --userDir . --settings ./settings.js flow.json"
         }

    Lets take a closer look at the start command:

        node
             --max-old-space-size=160         (1)
             ./node_modules/node-red/red.js   (2)
             --userDir .                      (3)
             --settings ./settings.js         (4)
             flow.json                        (5)


    1. As we’re running with a fixed memory limit, this argument is used to tell node when it should start garbage collecting.
    2. With node-red listed as an npm dependency of the project, we know exactly where it will get installed and where the red.js main entry point it.
    3. We want Node-RED to use the current directory as its user directory
    4. Just to be sure, we point at the settings file it should use - something we’ll add in the next step
    5. Finally we specify the flow file to use.


 3. Add an `engines` section to define what version of Node.js we want to run with. You could leave this out and just get whatever the current Node.js buildpack defaults to, but it is better to be explicit.

         "engines": {
             "node": "10.x"
         },


Your `package.json` file should now look like this:

```
{
    "name": "ProductionWorkshop",
    "description": "A Node-RED Project",
    "version": "0.0.1",
    "dependencies": {
        "node-red": "1.x",
        "node-red-contrib-cloudantplus": "0.2.0"
    },
    "node-red": {
        "settings": {
            "flowFile": "flow.json",
            "credentialsFile": "flow_cred.json"
        }
    },
    "engines": {
        "node": "10.x"
    },
    "scripts": {
        "start": "node --max-old-space-size=160 ./node_modules/node-red/red.js --userDir . --settings ./settings.js flow.json"
    }
}
```


At this point you should restart Node-RED.


## 3.2 - Add a settings.js file

You are already familiar with the Node-RED `settings.js` file you had to edit
in the earlier part of the workshop.

Before you can deploy this application to the cloud you need to give it its own
settings file to use.

Create the file `~/.node-red/projects/<name-of-project>/settings.js` with the following
contents:

```
const path = require("path");

module.exports = {
    uiPort: process.env.PORT,
    credentialSecret: process.env.NODE_RED_CREDENTIAL_SECRET,
    httpAdminRoot: false,
    httpStatic: path.join(__dirname,"public")
}

```

 - Setting `httpAdminRoot` to `false` will disable the Node-RED editor entirely - we
   do not want the flows to be edited directly in our production environment.
 - As you did with the main settings file, `httpStatic` is pointed at the folder
   containing the static resources of the application. `__dirname` is a special
   Node.js variable that contains the full path of the directory containing the
   current file. We can use that to get the full path to the `public` folder without
   having to know the specific file-system layout of the cloud environment.
 - `credentialSecret` is how you can provide the key for decrypting your credentials
   file. Rather than hardcode the key in the file - which is a bad idea - we set
   it using the environment variable `NODE_RED_CREDENTIAL_SECRET`. We will come
   back to setting that environment variable later.


## 3.3 - Add a manifest.yml file

When deploying to a Cloud Foundry environment, the `manifest.yml` file is used
to configure the application.

Create the file `~/.node-red/projects/<name-of-project>/manifest.yml` with the following
contents:

```
applications:
- name: give-your-application-a-unique-name
  memory: 256MB
  instances: 1
```

**Note:** you *must* give your application a unique name as that will form part
of the URL you use to access it.



## 3.4 - Push the application to the cloud

At this point you should have:

 -  a local Node-RED project with the extra files and settings needed to run as a cloud app
 - an IBM Cloud account (see [part 2](../part2/README.md))
 - the `ibmcloud` command installed and logged in (see [part 2](../part2/README.md))

To push your application to the cloud, open a terminal and change to the project
directory - `~/.node-red/projects/<name-of-project>`. Then run the command:

    ibmcloud cf push

This will push your application, using the manifest file to set the application name.
It will take a minute or two to complete.

Whilst that is running, it is useful to run the command `ibmcloud cf logs <name-of-app>`
in a second terminal window. This will show the logs of your application once it
starts.

The deploy should complete and you'll see output similar to the following:

```
name:              my-node-red-app
requested state:   started
routes:            my-node-red-app.eu-gb.mybluemix.net
last uploaded:     Wed 06 Nov 13:54:28 GMT 2019
stack:             cflinuxfs3
buildpacks:        sdk-for-nodejs

type:            web
instances:       1/1
memory usage:    256M
start command:   npm start
     state     since                  cpu    memory          disk           details
#0   running   2019-11-06T13:54:44Z   0.1%   65.4M of 256M   132.6M of 1G
```

Note the `routes` line - that is the URL you will be able to access your application
on.

**However**, some further configuration is needed before it will run properly:

1. you need to tell it the key for decrypting the credentials file using the `NODE_RED_CREDENTIAL_SECRET` environment variable
2. it is still configured to use CouchDB on localhost - the application needs to be
   updated to use a cloud-hosted CouchDB.


To address the first item, run the following command to set the environment variable
for your application to the value of the encryption key you set in step 1.5.

    ibmcloud cf set-env <name-of-app> NODE_RED_CREDENTIAL_SECRET <your-encryption-key>

It will offer to restage your application at this point, to which you should say
'yes'.

The second item to address is which CouchDB instance it should be using. That
will be done in the next part of the workshop.

## Summary

In this section of the workshop you have:

 - created the additional files needed to deploy it as a cloud application
 - pushed it to the IBM Cloud

## Next Steps

The application needs to be updated to use a cloud-hosted database. The next
part sets that up.

Take a moment, reflect on how far you've come. Then make a start on [Part 4](../part4/README.md).


***
*Quick links :*
[Home](/README.md) - [Part 1](../part1/README.md) - [Part 2](../part2/README.md) - [**Part 3**](../part3/README.md) - [Part 4](../part4/README.md) - [Part 5](../part5/README.md)

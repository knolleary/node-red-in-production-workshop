*Quick links :*
[Home](/README.md) - [Part 1](../part1/README.md) - [Part 2](../part2/README.md) - [Part 3](../part3/README.md) - [**Part 4**](../part4/README.md) - [Part 5](../part5/README.md)
***

# Part 4: Getting Started with Node-RED

**Goal:** *Update application to use cloud-hosted Couchdb*

## Introduction

This section of the workshop gets you started with an existing Node-RED application
that provides a simple To-Do list. It is based on the [TodoMVC](http://todomvc.com/)
and [Todo-Backend](https://www.todobackend.com/) projects.

The Node-RED flows implement the backend REST API used by the Todo webapp and
store the application data in a CouchDB database.

Rather than build the application from scratch, an example Node-RED project is
provided as the starting point.

## Steps

 - [1 - Install Node-RED](#1---install-node-red)


## 4.1 - Create a Cloudant Lite plan instance
    - Allow basic+iam access
    - Create two databases `todos` and `todos_dev` - both "non-partitioned"
    - Leave tab open

## 4.2 - Bind the cloudant service to your application

## 4.3 - Download the vcap services file for the application

## 4.4 - Edit NR settings
    - set VCAP SERVICES env var from file
    - set TABLE_NAME (tbd) env var to `todos_dev`

## 2. Restart NR

## 3. Edit every couchdb node
    - to use service instance
    - set database name to `${TABLE_NAME}` (tbd)

## 4. Go back to ToDo app and verify it works.
    - Switch back to cloudant dashboard and verify documents going into dev database


## 5. Run `cf push` again and verify the IBM Cloud hosted version works



## Summary

In this section of the workshop you have:

 - XXXXXXXXXX
 - XXXXXXXXXX
 - XXXXXXXXXX

## Next Steps

ZZZZZZZZZ

***
*Quick links :*
[Home](/README.md) - [Part 1](../part1/README.md) - [Part 2](../part2/README.md) - [Part 3](../part3/README.md) - [**Part 4**](../part4/README.md) - [Part 5](../part5/README.md)

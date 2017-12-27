# Docker-microservice-multicontainer

In this repository contains **main application** that communicate with **microservice** that was exposed via RESTful api.

Main application was developed using single simple page that develop using **PHP language**, meanwhile the Microservice was developed using **Python language** running on top of **Flask Microservice Framework** and **flask-restful** package as API representation.

Both applications live in its own `docker-container`.

## Requirement
 - Docker installed on your machine(Docker version 17.09.1-ce, build 19e2cf6 for mac).
 
## Final Folder Structure

```
|- ~/projects/docker-microservice-container
   |- product #-----> Microservice
     |- api.py
     |- Dockerfile
     |- requirements.txt
   |- website #-----> Main application
     |- index.php
   |- docker-compose.yml  
```

## Installation

Create one root project folder with name `docker-microservice-container` that holds our projects(anywhere) like above structure. These folder contains:
   
   **1. products folder**
      
   This is where our API resource lived in. It just a simple microservice that was built using python language running on top of Flask Microservice Framework and finally exposed via http protocol.
      
   **2. website folder**
      
   Our main application lived in here with a simple HTML markup mixed with PHP language.
      
   **3. docker-composer.yml file**
      
   Contains set of instruction to instruct `docker` to create docker container,images, etc. We're using this file instead of using `docker client` command like `docker run, docker build,etc..` which is more easy to maintain. 
      
## 1.1 Product Folder

As mentioned above, this folder contains our API resource and having 3 files to get started:

   - api.py file
   
     This file contain our python code to expose REST api to the main container.
     
     ```python
     from flask import Flask
     from flask_restful import Resource, Api
     
     app = Flask(__name__)
     api = Api(app)
     
     class Product(Resource):
        def get(self):
           return {
              'products': [
                 'proton', 'perodua', 'toyota'
              ]
           }
     
     api.add_resource(Product, '/')
     
     if __name__ == '__main__':
        app.run(host='0.0.0.0', port=80, debug=True)
     ```
     
   - Dockerfile file
     
     Sets of instruction to pull image from `docker hub` and copy our source code into target destination.
     
     ```
      FROM python:3-onbuild      # pull image from docker-hub    
      COPY . /usr/src/app        # copy source code into target destination   
      CMD ["python", "api.py"]   # command to run microservice
     ```
     
   - requirements.txt file
     
     Contains name of the pyhton packages to install along with the docker image.
     
     ```
      Flask==0.12
      flask-restful==0.3.5
     ```
     
## 2.1 Website Folder
     
This folder only contain simple php code to pull/fetch data from microservice that reside on another container.

**index.php**

```php
<html>
    <head>
        <title>Main Website</title>
    </head>
    <h1>List of products</h1>
    <ul>
        <?php
            $products = file_get_contents('http://product-service');
            // product-service is a name of services for another container.
            $json = json_decode($products);

            $products = $json->products;

            foreach ($products as $product) {
                echo "<li>$product</li>";
            }
        ?>
    </ul>
</html>
```

## 3.1 docker-composer file
     
This file is the main entry to begin with which contains set of instruction to instruct docker to build an image, container, etc.
     
```yaml
version: '3'

services:
    product-service: # microservice container
        build: ./product # folder to looks for dockerfile
        volumes: # This will make our code reflect to the changes that was made inside text editor.
            - ./product:/usr/src/app
        ports: # exposed 5001 port for host machine and 80 for container.
            - 5001:80

    website:
        image: php:apache # Get image from docker-hub
        volumes:
            - ./website:/var/www/html
        ports:
            - 5000:80
        depends_on: # This container depends on product-service container
            - product-service
```     


## Run

Open terminal and navigate to root project directory and run `docker-compose -d up` . 

`-d` will make the process running on background.

If everything goes well, you should be able to run main application inside web browser using `http://localhost:5000`.

## Extras

1. List of running docker-containers

   `docker ps`. Run `docker ps -a` to list containers(include not running containers)
   
2. List of docker-images   
 
   `docker images`
   
3. Remove image
   
   `docker rmi <image-id> <image-id> ....`
   
4. Remove container

   `docker rm <container-id> <container-id>`
   

*p/s: Self noted references :)*
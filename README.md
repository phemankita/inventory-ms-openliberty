# inventory-ms-openliberty: Openliberty Microservice with MySQL

## Table of Contents

* [Pre-requisites](#pre-requisites)
* [Run Locally](#run-locally)

## Pre-requisites:
* [Appsody](https://appsody.dev/)
    + [Installing on MacOS](https://appsody.dev/docs/installing/macos)
    + [Installing on Windows](https://appsody.dev/docs/installing/windows)
    + [Installing on RHEL](https://appsody.dev/docs/installing/rhel)
    + [Installing on Ubuntu](https://appsody.dev/docs/installing/ubuntu)
For more details on installation, check [this](https://appsody.dev/docs/installing/installing-appsody/) out.
* Docker Desktop
    + [Docker for Mac](https://docs.docker.com/docker-for-mac/)
    + [Docker for Windows](https://docs.docker.com/docker-for-windows/)


## Run Locally
To run this microservice locally run the following commands
1. Clone repository:
      ```
      git clone https://github.com/ibm-garage-ref-storefront/inventory-ms-openliberty
      cd inventory-ms-openliberty
      ```
2. Set up MySQL container by running the following command:
      ```
      # Start a MySQL Container with a database user, a password, and create a new database
      docker run --name inventorymysql \
          -e MYSQL_ROOT_PASSWORD=admin123 \
          -e MYSQL_USER=dbuser \
          -e MYSQL_PASSWORD=password \
          -e MYSQL_DATABASE=inventorydb \
          -p 3306:3306 \
          -d mysql:5.7.14
      ```

If it is successfully deployed, you will see something like below.

      ```
      $ docker ps
      CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
      c082aa55767c        mysql:5.7.14        "docker-entrypoint.s…"   9 seconds ago       Up 9 seconds        0.0.0.0:3306->3306/tcp   inventorymysql
      ```

* Populate the MySQL Database

Now let us populate the MySQL with data.

- Firstly ssh into the MySQL container.

      ```
      docker exec -it inventorymysql bash
      ```

* Now, run the below command for table creation.

      ```
      mysql -udbuser -ppassword
      ```

* This will take you to something like below.

      ```
      root@d88a6e5973de:/# mysql -udbuser -ppassword
      mysql: [Warning] Using a password on the command line interface can be insecure.
      Welcome to the MySQL monitor.  Commands end with ; or \g.
      Your MySQL connection id is 2
      Server version: 5.7.14 MySQL Community Server (GPL)

      Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

      Oracle is a registered trademark of Oracle Corporation and/or its
      affiliates. Other names may be trademarks of their respective
      owners.

      Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

      mysql>
      ```

* Go to `scripts > mysql_data.sql`. Copy the contents from [mysql_data.sql](https://github.com/ibm-garage-ref-storefront/inventory-ms-openliberty/blob/master/mysql/scripts/load-data.sql) and paste the contents in the console.

* You can exit from the console using `exit`.

      ```
      mysql> exit
      Bye
      ```

* To come out of the container, enter `exit`.

      ```
      root@d88a6e5973de:/# exit
      ```

3. Set up environment variables
    ```
    export MYSQL_HOST=host.docker.internal
    export MYSQL_PORT=3306
    export MYSQL_DATABASE=inventorydb
    export SSL_ENABLED=true
    export MYSQL_USER=dbuser
    export MYSQL_PASSWORD=password
    export jdbcURL=jdbc:mysql://${MYSQL_HOST}:${MYSQL_PORT}/${MYSQL_DATABASE}?useSSL=${SSL_ENABLED}
    export dbuser=${MYSQL_USER}
    export dbpassword=${MYSQL_PASSWORD}
    ```

4. Set up Jaegar for opentracing. This is optional. If you want to enable distributed tracing your application, run this step.
    ```
     docker run -d --name jaeger   -e COLLECTOR_ZIPKIN_HTTP_PORT=9411   -p 5775:5775/udp   -p 6831:6831/udp   -p 6832:6832/udp   -p 5778:5778   -p 16686:16686   -p 14268:14268   -p 9411:9411   jaegertracing/all-in-one:1.11
    ```

   You should have the following output:
    ```
    $ docker ps
    CONTAINER ID        IMAGE                                                 COMMAND                  CREATED             STATUS              PORTS                                                                                                                                                                     NAMES
    d0600b983133        jaegertracing/all-in-one:1.11                         "/go/bin/all-in-one-…"   2 hours ago         Up 2 hours          0.0.0.0:5775->5775/udp, 0.0.0.0:5778->5778/tcp, 0.0.0.0:9411->9411/tcp, 0.0.0.0:14268->14268/tcp, 0.0.0.0:6831-6832->6831-6832/udp, 0.0.0.0:16686->16686/tcp, 14250/tcp   jaeger
    ```
5. Run the application
   ```
   appsody run --docker-options "-e jdbcURL=jdbc:mysql://<docker_host>:3306/inventorydb?useSSL=true -e dbuser=dbuser -e dbpassword=password -e JAEGER_SERVICE_NAME=inventory-ms-openliberty -e JAEGER_AGENT_HOST=docker.for.mac.localhost -e JAEGER_AGENT_PORT=6831 -e JAEGER_REPORTER_LOG_SPANS=true -e JAEGER_REPORTER_FLUSH_INTERVAL=2000 -e JAEGER_SAMPLER_TYPE=const -e JAEGER_SAMPLER_PARAM=1"
   ```

   For instance <docker_host>, if it is `docker-for-mac` it will be `host.docker.internal`.

   If not running Jaegar, run the below command.  
    ```
     appsody run --docker-options "-e jdbcURL=jdbc:mysql://<docker_host>:3306/inventorydb?useSSL=true -e dbuser=dbuser -e dbpassword=password"
    ```

   For instance <docker_host>, if it is `docker-for-mac` it will be `host.docker.internal`.

   If this runs successfully, you will be able to see the below messages.

    ```
    [Container] [INFO] Connection gotten: com.mysql.jdbc.JDBC4Connection@68236731.
    [Container] [INFO] Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 1.205 s - in it.dev.appsody.starter.HealthEndpointTest
    [Container] [INFO]
    [Container] [INFO] Results:
    [Container] [INFO]
    [Container] [INFO] Tests run: 3, Failures: 0, Errors: 0, Skipped: 0
    [Container] [INFO]
    [Container] [INFO] Integration tests finished.
    [Container] [INFO] Running unit tests...
    [Container] [INFO] Unit tests finished.
    [Container] [INFO] Running integration tests...
    ```
5. Validate the REST API by visiting the following links:
    - http://localhost:9080/openapi/ui/
    - http://localhost:9080/index.html
    - http://localhost:9080/health
    - http://localhost:9080/openapi
    - http://localhost:9443/ibm/api/

Visit http://localhost:9080/openapi/ui/ and test the rest end point `micro/inventory`
as shown below:
![Openapi UI](./images/openapi-ui.png)

If enabled Jaegar, access it at http://localhost:16686/ and point the service to `inventory-ms-openliberty` to access the traces.

![Jaegar UI](./images/jaegar_ui_inventory.png)

![Jaegar traces](./images/jaegar_traces.png)

# How to use docker and Related commands to create container and image of container?

## Link docker to the project

1. Create `Dockerfile` in root `dir` of project to connect with docker and create `container image`.
1. after creating the `Dockerfile` name file add this the script on this file to create container

FROM node:12-alpine
# Adding build tools to make yarn install work on Apple silicon / arm64 machines

        RUN apk add --no-cache python2 g++ make
        WORKDIR /app
        COPY . .
        RUN yarn install --production
        CMD ["node", "src/index.js"]

>> Note: Dockerfile name file without any extention like .txt or .anything

1. Now create container image using this command

        docker build -t getting-started .

>>This command used the Dockerfile to build a new container image. You might have noticed that a lot of "layers" were downloaded. This is because we instructed the builder that we wanted to start from the node:12-alpine image. But, since we didn't have that on our machine, that image needed to be downloaded.

>>After the image was downloaded, we copied in our application and used yarn to install our application's dependencies. The CMD directive specifies the default command to run when starting a container from this image.

>>Finally, the -t flag tags our image. Think of this simply as a human-readable name for the final image. Since we named the image getting-started, we can refer to that image when we run a container.

>>The . at the end of the docker build command tells that Docker should look for the Dockerfile in the current directory.

## Starting an App Container

1. Start your container using the docker run command and specify the name of the image we just created:

        docker run -dp 3000:3000 getting-started

>>Remember the -d and -p flags? We're running the new container in "detached" mode (in the background) and creating a mapping between the host's port 3000 to the container's port 3000. Without the port mapping, we wouldn't be able to access the application.

        
1. After a few seconds, open your web browser to http://localhost:3000. You should see our app!

### Now, let's make a few changes and learn about managing our containers.

>>If you take a quick look at the Docker Dashboard, you should see your two containers running now (this tutorial and your freshly launched app container)!

## Updating our App

1. Go to `src/static/js/app.js` file and change line number 56. It is message shwoing on below of todo box. After chane you notice that when you refresh page will not reflect new changes.
1. For showing new changes need to remove previous container image and update with new one. For that 1st need to stop and remove current runing image by following steps:
    
    ## Following steps to remove image and create new one
    1. Run this command to get `docker container images` id

            docker ps

    1. first stop the conatiner using ip
            docker stop ip

    1. know get the id of current runing image. use to remove it

            docker rm  ip
            
    >> Direct command to stop and remove the container at once using this command 'docker rm -f id'
    1. Also have other way to stop and delete the current runing conatiner using `docker dashboard` from where manully stop and delete the container
    
    ## Starting our updated app container
    
    1. Now, start your updated app.
            
            docker run -dp 3000:3000 getting-started
    1. Refresh your browser on http://localhost:3000 and you should see your updated help text!

### It's not good practice to remove current image every time when update anything. How to prevent that?

## How to Sharing our App?

Now that we've built an image, let's share it! To share Docker images, you have to use a Docker registry. The default registry is Docker Hub and is where all of the images we've used have come from.

1. Create a Repo
    >>To push an image, we first need to create a repo on Docker Hub.

    1. Go to Docker Hub and log in if you need to.

    1. Click the Create Repository button.

    1. For the repo name, use getting-started. Make sure the Visibility is Public.

    1. Click the Create button!    
1. Pushing our Image
    1. In the command line, try running the push command you see on Docker Hub. Note that your command will be using your namespace, not "docker".

            docker push docker/getting-started
        Why did it fail? The push command was looking for an image named docker/getting-started, but didn't find one. If you run docker image ls, you won't see one either.
        To fix this, we need to "tag" our existing image we've built to give it another name.
    1. Login to the Docker Hub using the command docker login -u YOUR-USER-NAME.
    1. Use the docker tag command to give the getting-started image a new name. Be sure to swap out YOUR-USER-NAME with your Docker ID. 

            docker tag getting-started YOUR-USER-NAME/getting-started
    1.  Now try your push command again. If you're copying the value from Docker Hub, you can drop the tagname portion, as we didn't add a tag to the image name. If you don't specify a tag, Docker will use a tag called latest.
            
            docker push YOUR-USER-NAME/getting-started
1. Running our Image on a New Instance

    Now that our image has been built and pushed into a registry, let's try running our app on a brand new instance that has never seen this container image! To do this, we will use Play with Docker.
    1. Open your browser to Play with Docker https://labs.play-with-docker.com/
    1. Log in with your Docker Hub account.
    1. Once you're logged in, click on the "+ ADD NEW INSTANCE" link in the left side bar. (If you don't see it, make your browser a little wider.) After a few seconds, a terminal window will be opened in your browser.
    1. In the terminal, start your freshly pushed app.

            docker run -dp 3000:3000 YOUR-USER-NAME/getting-started
        
        You should see the image get pulled down and eventually start up!
    1. Click on the 3000 badge when it comes up and you should see the app with your modifications! Hooray! If the 3000 badge doesn't show up, you can click on the "Open Port" button and type in 3000.

## Persisting our DB / Create SqlLite Db
1. Create a volume by using the docker volume create command.

        docker volume create todo-db
1. Know again remove currnet image by geting `ip` and run the command

        docker rm -f ip
1. Start the todo app container, but add the -v flag to specify a volume mount. We will use the named volume and mount it to /etc/todos, which will capture all files created at the path.

        docker run -dp 3000:3000 -v todo-db:/etc/todos getting-started
1. Once the container starts up, open the app and add a few items to your todo list.
1. Remove the container for the todo app. Use the Dashboard or docker ps to get the ID and then `docker rm -f <container-id>` to remove it.
1. Start a new container using the same command from above.
1. Open the app. You should see your items still in your list!

### Diving into our Volume / Or check existing todo db
1. A lot of people frequently ask "Where is Docker actually storing my data when I use a named volume?" If you want to know, you can use the docker volume inspect command.
1. To check current sqlite db run this command

        docker volume inspect todo-db
1. Know you will se the local systm path where the todo.db is exist. Something like this:

        [
            {
                "CreatedAt": "2019-09-26T02:18:36Z",
                "Driver": "local",
                "Labels": {},
                "Mountpoint": "/var/lib/docker/volumes/todo-db/_data",
                "Name": "todo-db",
                "Options": {},
                "Scope": "local"
            }
        ]
1. The `Mountpoint` is the actual location on the disk where the data is stored. Note that on most machines, you will need to have root access to access this directory from the host. But, that's where it is!

# How to install MySql and connect with existing app?
1. for that development langaue is seperate and databse is seperate so in our normal localhost case we will download xampp for php coding to setup local server and in which we get mysql in the form of `phpmyadmin`. same like that we need here connection b/w both for that we need to create a newtowrk connection using this command

        docker network create todo-app
1. Start a MySQL container and attach it to the network. We're also going to define a few environment variables that the database will use to initialize the database (see the "Environment Variables" section in the MySQL Docker Hub listing).
        docker run -d --network todo-app --network-alias mysql -v todo-mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=secret -e MYSQL_DATABASE=todos mysql:5.7
1. To confirm we have the database up and running, connect to the database and verify it connects.

        docker exec -it <mysql-container-id> mysql -p
1. When the password prompt comes up, type in secret. In the MySQL shell, list the databases and verify you see the todos database.

        mysql> SHOW DATABASES;
1. You should see output that looks like this:

        +--------------------+
        | Database           |
        +--------------------+
        | information_schema |
        | mysql              |
        | performance_schema |
        | sys                |
        | todos              |
        +--------------------+
        5 rows in set (0.00 sec)

    Hooray! We have our todos database and it's ready for us to use!

    To exit the sql terminal type exit in the terminal.

## Connecting to MySQL?
1. Now that we know MySQL is up and running, let's use it! But, the question is... how? If we run another container on the same network, how do we find the container (remember each container has its own IP address)?
1. To figure it out, we're going to make use of the nicolaka/netshoot container, which ships with a lot of tools that are useful for troubleshooting or debugging networking issues.
1. Start a new container using the nicolaka/netshoot image. Make sure to connect it to the same network.

        docker run -it --network todo-app nicolaka/netshoot
1. Inside the container, we're going to use the dig command, which is a useful DNS tool. We're going to look up the IP address for the hostname mysql.

        dig mysql

    And you'll get an output like this...

        ; <<>> DiG 9.14.1 <<>> mysql
        ;; global options: +cmd
        ;; Got answer:
        ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 32162
        ;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

        ;; QUESTION SECTION:
        ;mysql.             IN  A

        ;; ANSWER SECTION:
        mysql.          600 IN  A   172.23.0.2

        ;; Query time: 0 msec
        ;; SERVER: 127.0.0.11#53(127.0.0.11)
        ;; WHEN: Tue Oct 01 23:47:24 UTC 2019
        ;; MSG SIZE  rcvd: 44

    In the "ANSWER SECTION", you will see an A record for mysql that resolves to 172.23.0.2 (your IP address will most likely have a different value). While mysql isn't normally a valid hostname, Docker was able to resolve it to the IP address of the container that had that network alias (remember the --network-alias flag we used earlier?).

    What this means is... our app only simply needs to connect to a host named mysql and it'll talk to the database! It doesn't get much simpler than that!

## Running our App with MySQL

The todo app supports the setting of a few environment variables to specify MySQL connection settings. They are:

        MYSQL_HOST - the hostname for the running MySQL server
        MYSQL_USER - the username to use for the connection
        MYSQL_PASSWORD - the password to use for the connection
        MYSQL_DB - the database to use once connected
1. With all of that explained, let's start our dev-ready container!
1. We'll specify each of the environment variables above, as well as connect the container to our app network.

        docker run -dp 3000:3000 -w /app -v "$(pwd):/app" --network todo-app -e MYSQL_HOST=mysql -e MYSQL_USER=root -e MYSQL_PASSWORD=secret -e MYSQL_DB=todos node:12-alpine sh -c "yarn install && yarn run dev"
1. 
1. 
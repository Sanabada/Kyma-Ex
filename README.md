# Kyma-Ex
Kyma is (related kubernetes)  is a platform for extending applications with serverless functions and microservices. It provides a selection of cloud-native projects glued together to simplify the creation and management of extensions..

Microservice Deployment in Kyma:
Here are few steps to deploy microservice to Kyma environment. We will take a small example for reference.
Prerequisites:

•	Only some basic knowledge about Kyma
            https://kyma-project.io/docs/components/serverless#tutorials-create-a-function-from-                  git-repository-sources

•	Basic knowledge about Node.js
Note: if you don’t want to install Node.js, you can skip it.
Really, because we’ll create a docker container which will contain a Node.js installation

Application:

Here we are creating an application with node.js  which meant to  be accessible for free. In this case, we create a basic server app which exposes 2 REST endpoints

The next application example will be a secured one.

Create node app
We create a working directory called
– C:\tmp_kyma
Inside, we create 2 files:
– package.json
– server.js
It looks like this:
 
package.json
{
    "dependencies": {
        "express": "^4.16.3"
    }
}


server.js
const express = require('express')
const app = express()

app.get('/free', (req, res)=>{
    res.send('Response for Kyma')
})

app.get('/protected', (req, res)=>{

    res.send(`Successfully passed security control. Running on '${req.headers.host}${req.path}'. Auth: '${req.headers.authorization}'`)
})

app.listen(1111, ()=>{
    console.log('Node app running on port 1111')
})
Afterwards, we need to open command prompt, jump into our working directory and execute the following command
Npm install
This will download and install all required dependencies
Let’s have a quick look at the app:
We create a server
We define an endpoints, listening to GET requests
That is called /free and returns a  response
The server is started and listens on port 1111
In our example, 1111 stands for initial port number, defined in our application code
Run app locally
To run our app, we go to our command prompt and from the working directory we execute the command node server.js
Now that the server is running, we open a browser and call our 2 endpoints:
http://localhost:1111/free
and
http://localhost:1111/protected
Docker

Create image
We create a file called Dockerfile in the working directory

Our current project folder:
 
The content of the Dockerfile can be found as below:
Dockerfile
FROM node:12
COPY . .
RUN npm install
EXPOSE 1111
CMD ["node", "server.js"]
You’ll easily find better dockerfiles to be used with node.js applications.
What it says:
We just copy everything (the first dot) to the container (the second dot)
We also execute the 2 commands which we just executed locally (install dependencies and run the app)
After that  I recommend to check again:
Exposed 1111 correctly?
We go to command prompt, jump into our working dir and execute this command:
docker build -t yourDockerID/safeapp .
Note: Make sure to replace the dockerID with yours (remember, you need docker ID in order to upload to public docker repo)

Upload image
We want to upload our image to the public docker hub, such that it can later be easily used by Kyma
The command: docker push <yourDockerID>/safeapp
Until now, we’ve created a little node.js application, containerized it and uploaded to public docker repo
Next, we want to deploy to Kyma
Kyma
To deploy our application to Kyma, we need to define a deployment descriptor. we create a file called deploy_app.yaml in our working directory and copy the following content:
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-app-nodesafe
spec:
  template:
    metadata:
      labels:
        apptemplate: template-nodesafeapp
    spec:
      containers:
        - image: <yourDockerID>/safeapp
          name: container-safeapp
          ports:
            - name: http
              containerPort: 1111
  selector:
    matchLabels:
      apptemplate: template-nodesafeapp

Note:
Make sure to replace the dockerID with yours

Now open the Kyma Console UI In Kyma dashboard, we navigate into our namespace (default), then press the button “deploy new resource” and deploy our deploy_app.yaml file
After deployment, we can check the new resources (deployment and pod) in our Kyma dashboard
To check the log, we can use the context menu of the pod
We can see the log outpout of our server.js
We can check our node.js app has been deployed and has been started.
To check the endpoint now, we need to define service.
To create a Service, we have to deploy a resource file
In our working directory, we create another file, called deploy_service.yaml
The content:
apiVersion: v1
kind: Service
metadata:
  name: service-nodesafeapp
  labels:
    servicelabel: nodesafeappservice
spec:
  ports:
    - port: 3333
      targetPort: 1111
  selector:
    apptemplate: template-nodesafeapp
We deploy this resource file as us usual: Overview->Deploy new resource
To view the newly created service, we go to Operation-> Services then click on the service entry
 
we still can’t call our server-endpoint from browser
We need to expose it to the outside world, so we create Api Rule resources.
Create API Rule
Creating an API Rule can be done in the dashboard
In the “Service” details screen, there’s an “Expose Service” button in the “API Rules” section
This opens the creation dialog
Note:
Alternatively, we can navigate to Configuration -> API Rules, then select the desired service
 
In the Create API Rule screen, we enter an arbitrary name for the API rule, e.g. apirule-nodesafe
Then enter a short name for the host, which will end up in the URL of our API, e.g. safeapp
We make sure that the service, which we created above, is selected
we enter the relative path to our free endpoint. We leave the defaults, which means that no restricting rule will be applied to our “free” endpoint, Finally, we press create. We can see the specified host is concatenated with the Kyma domain.
Run app in Kyma: 
 After saving the new API Rule, we can click on the “Host” URL
After pressing the Host-URL, we only need to append our free endpoint
In my example:  https://safeapp.c2de3ec.kyma.shoot.live.k8s-hana.ondemand.com/free
We can check about the security feature in the next example. Till then enjoy 😊
                                                              

                                                                                                                       By Abhinita Sanabada


# AMAZON ECS Deployment Example

## Pre-requisite bugfix for backend

In our application code, we need to tweak this code block in the app.js

```
mongoose.connect(
  `mongodb://${process.env.MONGODB_USERNAME}:${process.env.MONGODB_PASSWORD}@${process.env.MONGODB_URL}:27017/course-goals?authSource=admin`,
  {
    useNewUrlParser: true,
    useUnifiedTopology: true,
	family: 4
  },
  (err) => {
    if (err) {
      console.error('FAILED TO CONNECT TO MONGODB');
      console.error(err);
    } else {
      console.log('CONNECTED TO MONGODB!!');
      app.listen(80);
    }
  }
);
```

_This is a bugfix for mongodb connectivity on AWS, possibly related to some incorrect vpc setting, I added the `family: 4` configuration option. Please ensure you made this change, built, and pushed your updated image to docker hub._

### Creating the cluster

1. Open your AWS dashboard and search for **"ECS"**
   ![ECS Dashboard](./images/ECS%20Dashboard.png)
1. Select the **Create Cluster** button and give you cluster a name, all other options should be default, and click the **Create** button
   ![ECS Cluster Create](./images//ECS%20Cluster%20Create.png)

### Creating Task Definitions

1. Expand the sidebar navigation and select **Task Definitions**
   ![ECS Task Definition Dashboard](./images/ECS%20Task%20Definitions%20Dashboard.png)
1. Click the **Create new task definition** button and give "Task definition family" a unique name
   ![ECS Task Defintion name](./images/ECS%20Task%20Definition__name.png)

    1. scroll down to the container definitions and lets start by defining the mongodb container

        - give it a container name
        - define the image, in this case its just **mongo**
        - in the _Port Mappings_ we need to tell it what the container port is where it is listening (**the default port for mongod is 27017**) and give the port a name
        - ⚠️ Extremely Important ⚠️ <br/> Scroll down to **HealthCheck** and expand the section, and enter the following command in the "Command box"
            > CMD-SHELL,mongosh --eval 'db.runCommand("ping").ok' --quiet
        - ⚠️ Extremely Important ⚠️ <br/> Scroll down to the _Environment Variables_ section and click the button to **Add Environment Variable**
            - Add _**BOTH**_ of these
                > MONGO_INITDB_ROOT_USERNAME
                > "max" <br/>
                > MONGO_INITDB_ROOT_PASSWORD
                > "secret"

    1. all other options can remain default

    ![ECS TD Container 1](./images/ECS%20Task%20Definition__container1.png)

    1. Click the **+ Add container** button to add a new container entry.

        - give it a unique name, this time we're defining our backend container (I'm calling mine "node-goals")
        - enter the image address, this will be your docker hub id, a slash, and then the image name
            > _e.g._ unsivilaudio/node-goals
        - we can leave **Essential Container** as "No" in this case
        - select **Add port mapping** button so we can expose a port container, it will default to 80 (you still need to type it in) which is correct for our container, and you can once again give it a name
        - select **App protocol** to "HTTP"
        - ⚠️ Extremely Important ⚠️ <br/> Scroll down to the _Environment Variables_ section and click the button to **Add Environment Variable**

            - Add _**ALL**_ of these
                > MONGODB_USERNAME
                > "max" <br/>
                > MONGODB_PASSWORD
                > "secret" <br/>
                > MONGODB_URL
                > "localhost"

        - ⚠️ Extremely Important ⚠️ <br/> Scroll down to **Startup dependency ordering**, and expand the section, select your mongodb container name, and set condition to "Healthy"
        - all other settings can remain default, however it is probably a good idea to optionally enable the logs for this container as well

    ![ECS TD Container 2](./images/ECS%20Task%20Definition__container2.png)

    1. Scroll to the bottom and click the **Create** button

### (Deploying) Running the task definition

1. Back at the dashboard, select your newly created task definition, and then open the **Deploy** dropdown and select "Create service"
   ![ECS Task Definition run](./images/ECS%20Task%20Definition__run.png)

1. Make sure your cluster you just created is selected and scroll the page down and find the **Deployment Configuration** section, we need to give the "Service name" a unique identifier

1. Scroll down to the **Networking** section and expand it. We need VPC (which creates our own private little network on AWS) but you might notice we have none to select from.

    1. In the searchbox at the top of the page, search for "VPC", right-click and open this in a new tab

        1. on the VPC dashboard click the **Create VPC** button
            - **optionally** in the "Name tag auto-generation", you can give it a more specific name
        1. all other settings can remain default
        1. scroll to the bottom and click **Create VPC** button
        1. **⚠️ KEEP THIS PAGE OPEN ⚠️**

    1. go back to your service deployment tab and reload the page (we might have to redo step 2), we'll now have a VPC we can select, you will notice it automatically adds selected subnets to this (leave them added)

    1. in the "Security group" section, make sure there is a selected security group

1. leave all other options on this page unchanged and click the **Create** button

    ![ECS Task Definition deploy](./images/ECS%20Task%20Definition__deploy.png)

1. You will now be at your **Services** dashboard, click on your newly created service name (under the "Services" tab) to go see the details of your deployment

1. Select the "Tasks" tab and do the same, clicking on your task id to see the details of your currently creating tasks

    ![ECS Tasks overview](./images/ECS%20Service%20Tasks__overview.png)

1. After some time you should see both your containers are running, and we're almost done, but we need some additional configuration before we can access our backend from the outside

    ![ECS Tasks details](./images/ECS%20Service%20Tasks__details.png)

    > Note: we did not add a HealthCheck on the backend container, so it will always show "unknown"

### (VPC) Security Groups configuration

1. Now we need to head back over to the VPC page I told you to keep open. From the sidebar under the **Security** heading, select "Security groups"

1. This will show you a list of the different security groups in associated with your AWS account, you likely will only have one.
   ![ECS VPC SG Dashboard](./images/ECS%20VPC%20Security__dashboard.png)

1. Click on the "Security group ID" for the group you associated with your created Service, this will bring us to the details of this group.

    1. Select the "Inbound rules" tab and find the **Edit Inbound Rules** button, click it
    1. In the configuration screen we need to add a new rule, so click the **Add rule** button
    1. For our new rule we need to select "HTTP" as the type, and in the box to the right "Custom" dropdown, click inside and select `0.0.0.0/0` from the popout menu options
        - **optionally** enter something into the description box
    1. Click **Save rules** button finish the configuration
       ![ECS VPC SG Inbound](./images/ECS%20VPC%20Security__inbound_rules.png)

1. Now we have access to our backend _from our public ip_ associated with our deployment and we can test this out in postman 🎂

> \_send a request to "http://\<public-ip\>/goals" to test

> Note: you can retrieve the publically accessible ip from your task details view (which you can see in the above screenshot)

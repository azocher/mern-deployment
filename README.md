### 💻 MERN App Deployment to Heroku
----
### Deployment Details:
We will be deploying our MERN stack apps to Heroku. 

There are three primary ways to organize and deploy a MERN stack app:

1. Single File App
	* React and Express files are kept in the same folder
	* React files all nested in `client` folder
	* Use this system if your Express server is acting as your API router
2. Fully Decoupled Apps
	* Express and React apps kept in completely seperate files
	* Good if you are not making complex API calls to Express OR if you want to focus on threading for delivery. 
3. API Behind a Proxy
	* Uses a proxy to interact similar to Single File App setup, but on seperate machiens ala Fully Decoupled

	
Our deployment for SEI MERN apps is going to be the same as **Single File App** deployment. 

## Steps to Achieve:

### Setting up Git and Heroku Connection
1. Navigate on the terminal to your project folder. Confirm that git master is up to date; commit if not. 
2. *Confirm that your .gitignore is up to date with current app organization*. 
3. Login to Heroku CLI by typing `heroku login` on terminal and following given directions. 
4. Create a new app @ Heroku named whatever you would like. 
5. Set your Heroku git remote (`heroku git:remote -a name-of-your-app`)

**Stop at this step.** We will not be deploying our app to Heroku yet (we will eventually using the `git push heroku master` command). 

First, we need to re-organzie our app into a Single File App organizational structure. 

Second, we need to update our app details to reflect these changes. 

Third, we need to update config variables and add a proxy so that our app will reflect the new location of variables on Heroku (this step will include setting .env variables on Heroku). 

Fourth, we will add details to our `package.json` files to instruct Heroku on how to build and run our app. 

Finally, we have to setup our MongoDB remote on Heroku. 

### Re-Organize App to Single File App Setup

1. Navigate to the parent folder which currently contains two folders: `client` and `server`. 
2. Take all files from your `server` folder and move them one level up - out of the `server` folder and into our parent folder. Now your `server` folder should be empty (you can go ahead and delete it), and the `client` folder should be a sub folder of this parent folder. 


**Example New File Structure:**

	
```
client/  ** THIS IS OUR REACT APP **
	|__ node_modules/
		|__ tons of stuff...
    |__ public/
    |__ src/
    .env
models/
   |__ user.js
   |__ todo.js
   |__ etc.
node_modules/
   |__ stuff...
controllers/
   |__ userRoutes.js
   |__ todoRoutes.js
   |__ etc.
.gitignore
package.json
server.js
.env

```


### Update App Details to Reflect Change

1. Now that our files are in slightly different locations, please update your `.gitignore` to reflect these changes. Example file structure and .gitignore below. 
2. Make a git commit to reflect new app organizational structure with updated `.gitignore`.

**Example `.gitignore`:**

```
.env
node_modules
client/.env
client/node_modules
```

### Add Proxy, Update API Calls, and Set ENV Variables

#### A. Add Proxy
In your `package.json` in the `client` folder, add the following line (anywhere on the `package.json` list is fine):
`"proxy": "http://localhost:8000`

This proxy will act as a "secret" for our app to know what server address to call when making calls to our Express Server. In this case we are going to set the port of our Express Server to be 8000 on Heroku. 

#### B. Update API Calls

Now that we have a proxy, we need to update our API/Express server calls. 

Currently your app has calls that look like the following: 
`fetch("${process.env.REACT_APP_SERVER_URL}/auth/login", {`

We no longer will need a direct server URL for our React side calls - the proxy will automatically take care of this for us. Now, we can remove any url/localhost address and just put the exact path after. 

For example the call above becomes:
`fetch("/auth/login", {`

Our app knows that when `/auth/login` is called, it is supposed to attach the `proxy` we have added to the front of the call. Our app will run this call as `http://localhost:8000/auth/login`. 

**Go through your React App and update every call to your Express server to reflect this new format. Once you are done make a git commit. **

#### C. Set ENV Variables

To use `.env` stored variables on Heroku, we have to set them remotely on our Heroku app instance. 

Using the Heroku web client, navigate to the `Settings` tab for your app. Add your .env variables for both client and server to `Config Vars` section one by one. 


*Example of ENV Vars to Add:*

* JWT secret message
* External API Keys
* the PORT for your Express app (**in this case make sure to set to 8000**). 

Next, go through your local code and make sure that each call to an ENV variable uses `process.env` and not a static value (i.e. calling `port=8000` instead of `port=process.env.PORT`). 

### Instruct App on How to Build and Run
Heroku allows two ways for us to instruct it on setup: the first is using a `Procfile` in which we write specific instructions on how to build and run. In the absence of a `Procfile` our Heroku app will turn to our `package.json` and look for instructions on how to run. 
 
The first thing we have to do is tell Heroku which Node version we want our app to run on, and what to do with our React/`client` files to compile and build our React app at start. 

Add the following lines to your `package.json` in your server/project folder. 

```
 "scripts": {
    "start": "node index.js", // user server.js if using that file name
    "heroku-postbuild": "cd client && npm install && npm run build"
  },
  "engines": {
    "node": "10.15.0"
  }
```

#### Using Path Package

We now want to tell our Express app how to run our React build, and how to find our correct React files. 

1. Add the following to your main Express server app at the top, with our other `require` calls.
```
let path = require('path')
```

2. Run `npm install path` to install the path module.

3. In our `app.use()` call list, add the following line. This is our way of telling our app what files to use on build. `app.use(express.static(path.join(__dirname, 'client/build')));`

4. In our general route (`app.get('*', `), add the following for `res.send()`. You likely currenlty have a 404 status call. This is going to redirect back to our index.
`res.sendFile(path.join(__dirname, '/client/build/index.html'));
});`

5. Make a git commit to master to reflect these new changes. 


### Create and Link Remote MongoDB Server
Heroku has a built in library tht allows us to create a MongoDB add on and automatically setup a `MONGODB_URI` ENV variable. 

Navigate to your project folder and run the following line of code: 
`heroku addons:create mongolab -a <name-of-heroku-app>`

If your Heroku account asks for you to add a payment method to include this addon, ping instructors and we will help set you up with a ghost card. 

### Push and Test

Once you have finished all these steps, go ahead and make an initial push to Heroku and see what errors come up. We will then work through each indiviudal error log as they come up for each group. 

To push and build: `git push heroku master`









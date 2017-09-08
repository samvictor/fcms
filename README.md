# FCMS
A CMS solution that can be run in a static environment. Uses and requires Firebase.

## Let's Begin
### 1. Create a Firebase Account and Project
In order to use Flicker CMS (FCMS), you need to create a [Firebase](https://console.firebase.google.com/) account 
(same as your google account) and create a new project.

By the way, if you want to add FCMS to an existing Firebase project, that's fine too. 
These instructions will cover both cases.

### 2. Setup your Firebase Folder
Go to [firebase.google.com/docs/hosting/deploying](https://firebase.google.com/docs/hosting/deploying) 
to install the Firebase cli and setup a Firebase directory for the project you just created.
If you already have a Firebase project, 
you probably already have the CLI installed and a directory setup.

In the Firebase project directory, find the hosting directory. (Named "*public*" by default).
Clone this Git Repository into that folder.

When you've done this, the file structure should look like this:

```
firebase_project/
    - firebase.json
    - public/
        - index.html
        - 404.html
        - fcms/
            - index.html
            - init_settings.js
            - template_1.html
            - template_2.html
            - fcms_admin.html
            - fcms_serve/
            - ...
```

If you would like FCMS to be installed at the root of your domain (*example.com/*),
delete *public/index.html* and copy the contents of *public/fcms/* into *public/*. 
If you would like FCMS to be installed into a subdirectory (*example.com/sub/*),
change the name of the *public/fcms/* folder to something else (*/public/sub/*).

I'll continue these instructions assuming you didn't change the *public/fcms/* folder.

Finally, find and rename */public/fcms/init_settings.js* to */public/fcms/settings.js*

### 3. Get your Firebase Config Info
Once you've created your Firebase account, go to the *Overview* tab in your Firebase console.
*console.firebase.google.com/project/<your project name>/overview*

Click on the button that says "Add Firebase to your web app".
If you've already connected an app to this project, then click on "Add another app" first, then click "Web".

Now go back into the FCMS 

Copy the piece of code that sets the `config` variable.

Open the */public/fcms/settings.js* file again paste the code into that settings file. 
When you're done, you should have a */public/fcms/settings.js* file that looks like this:

```
settings = {
	"ready": "no", 
	"email": "", 
	"password": "", 
	"root": ""
};

// Initialize Firebase
var config = {
    apiKey: "<your-api-key>",
    authDomain: "<your.auth.domain>",
    databaseURL: "<your.database.url>",
    projectId: "<your-project-id>",
    storageBucket: "<your-storage-bucket>",
    messagingSenderId: "<your messageing sender Id>"
};
```





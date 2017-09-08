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

### 4. Setup an Admin in Firebase Auth
Now you need to create an admin account for your website.
This will be the only account that can edit pages and essential details about your website,
besides user posts/comments. 

Go into the *Authentication* section of your Firebase Console 
(https://console.firebase.google.com/project/<project-name>/authentication/users).

Enable "*Google*" as a sign-in method from the *Sign-In Method* tab.  

From the *Users* tab, create your Admin user.

Copy that user's *User UID*.

Go to the *Database* section of your Firebase Console.

In the *Rules* tab, add the following rules:

```

"fcms_data": {
      ".read": false,
    	".write": "auth.uid === '<YOUR ADMIN AUTH UID HERE>'",
        
    	"gen_site_data": {
        ".read": true
      },
      "pages": {
        ".read": true
      },
        
      "users": {
        ".read": false,
        ".indexOn": ["username", "full_name", "last_login", "privacy", "hidden"],
        "$uid": {
        	".read": "$uid === auth.uid",
        	".write": "$uid === auth.uid",
            
          "pub_data": {
            ".read": true,
          },
          "friend_data": {
            "$connection": {
              ".read": "$uid === auth.uid || 
              					data.parent().parent().child('connection')
                          .child($connection).child(auth.uid).exists()",
            }
          },
          "priv_data": {
            ".read": "$uid === auth.uid"
          },
            
          "connection": {
            ".read": "$uid === auth.uid ||
            					data.child('friends').child(auth.uid).exists()",
          },
            
          "posts_made": {
            ".indexOn": ["date_made", "target_user", "target_post", "type", "privacy", "hidden"],
            "$pid": {
              ".read": "$uid === auth.uid ||
              					(data.child('privacy').val() === 'public' && 
                         	data.child('type').val() != 'message') ||
              					data.child('list_targets').child(auth.uid).exists()",
            },
          },
          "posts_received": {
        		".read": "$uid === auth.uid",
            ".indexOn": ["date_recieved", "target_post", "source", "type", "privacy", "hidden"]
          },
            
          "wall": {
        		".read": true,
            ".indexOn": ["date_made", "target_post", "source", "type", "hidden"]
          },
          "connected_wall": {
            "$connection": {
              ".read": "$uid === auth.uid ||
                        data.parent().parent().child('connection')
                        	.child($connection).child(auth.uid).exists()",
              ".indexOn": ["date_made", "target_post", "source", "type", "privacy", "hidden"]
            }
          },
          "priv_wall": {
        		".read": "$uid === auth.uid",
            ".indexOn": ["date_made", "target_post", "source", "type", "hidden"]
          }
        }
      },
      "public_user_list": {
        ".read": "auth != null",
      },
      "nexus": {
        ".read": true,
        ".write": "auth != null",
        ".indexOn": ["date", "target_user", "source", "type"],
        "$nid": {
          ".write": "!data.exists() ||
          						data.child('target_user').val() === auth.uid"
        } 
      }
    }

```

Put these rules in the open curly braces of the existing `rules` variable. 
You also need to set the existing global `.read` and `.write` rules to `false`
In context, it should look like this.

```
{
  "rules": {
    ".read": false,
    ".write": false,
    
    "fcms_data": {
        ".read": false,
    	".write": "auth.uid === '<YOUR ADMIN AUTH UID HERE>'",
        ... other fcms rules ...
    }
    
    ... other database rules ...
}

```

Make sure to replace **<YOUR ADMIN AUTH UID HERE>**, on the 3rd line,
with the uid that you copied from the Firebase console.

If you would like to add other Admin users, 
you need to update this database rule in the following pattern.

```
".write": "auth.uid === '<ADMIN AUTH UID 1>' ||
                auth.uid === '<ADMIN AUTH UID 2>' ||
                auth.uid === '<ADMIN AUTH UID 3>'",
```


**Important**:
If you have *Database* enabled in the Firebase Folder on your local machine, 
you will need to add these rules to your *database.rules.json* file also. 
Otherwise, the next time you *deploy*, your database rules will be overwritten.
If you do not have a *database.rules.json* file in your Firebase directory, 
Then you probably do not have *database* enabled in that directory and you can ignore this notice.






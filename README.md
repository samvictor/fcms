# FCMS
A CMS solution that can be run in a static environment. Uses Firebase and Bootstrap.

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
delete */public/index.html* and copy the contents of */public/fcms/* into */public/*. 
*/public/* is now your FCMS directory and *example.com/* is your FCMS url.

If you would like FCMS to be installed into a subdirectory (*example.com/sub/*),
change the name of the *public/fcms/* folder to something else (*/public/sub/*).
In this case, */public/sub/* is your FCMS directory and *example.com/sub/* is your FCMS url.

I'll continue these instructions assuming you didn't change anything and */public/fcms/* 
is your FCMS directory.

Find and rename */public/fcms/init_settings.js* to */public/fcms/settings.js*

Finally edit your */firebase.json* file to look like this:

```
{
  "hosting": {
    "public": "public",    
    "rewrites": [
      {
        "source": "/fcms/**/settings.js", "destination": "/fcms/settings.js"
      },
      {
        "source": "/fcms/**", "destination": "/fcms/index.html"
      }
    ]
  }
}

```

You can replace "/fcms/" with your own root. 
Also, because earlier "rewrites" entries have higher precedence, 
the order is important here.

### 3. Get your Firebase Config Info
Once you've created your Firebase account, go to the *Overview* tab in your Firebase console.
`console.firebase.google.com/project/<your project name>/overview`

Click on the button that says "Add Firebase to your web app".
If you've already connected an app to this project, then click on "Add another app" first, then click "Web".

Now go back into the FCMS 

Copy the piece of code that sets the `config` variable.

Open the */public/fcms/settings.js* file again paste the code into that settings file. 

While you're here, also set the *"tab_title"* variable. 
This will be the title that shows up in browser tabs.

When you're done, you should have a */public/fcms/settings.js* file that looks like this:

```
settings = {
    'tab_title': 'My Site'
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
`https://console.firebase.google.com/project/<project-name>/authentication/users`.

Enable "*Email/Password*" as a sign-in method from the *Sign-In Method* tab.  

From the *Users* tab, create your Admin user with the *Add User* button.

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


### 5. Hit the Site
Make sure you deploy your changes with `firebase deploy --only hosting`.
The `--only hosting` part is not necessary, but it should make deploying faster.

Now try hiting your FCMS url.

If you've already forgotten what I mean by FCMS url and FCMS directory,
I talked about it in "*step 1*" but I'll explain it again.

Your FCMS_url will depend on where you "installed" or pasted FCMS. 
If your FCMS index.html and template files are located at */public/sub/index.html* 
and */public/sub/template_1.html* etc., then */public/sub/* is your FCMS directory 
and *example.com/sub/* is your FCMS url.

When you hit your FCMS url, you should see a loading screen that never ends. 
This is expected.

If you haven't yet connected a custom domain to your project, 
you can use the one provided automatically by Firebase, 
but please remember to change it in your fcms_admin settings when you do connect a custom domain.
If you don't know what fcms_admin is yet, don't worry, I'll explain it later.
 
You can find your Firebase-provided domain in the console print-out after you deploy. 
It will be named "*Hosting URL:*". 

If your hosting url is `https://<proj-id>.firebaseapp.com`, 
and your FCMS directory is "*/public/fcms/*",
then your FCMS url is `https://<proj-id>.firebaseapp.com/fcms/`


### 6. Serving Static Assets & Files
Any static assets or files that you need for your website (such as images, videos, PDFs, etc.)
should be placed in the folder named "*/public/fcms/fcms_serve/*".
You can then use those same assets using the following url format:

```
FCMS_url/fcms_serve/<file_name> 
or in this case
*example.com/fcms/fcms_serve/<file_name>*
```

Remember that after adding any file to this or any other directory,
you must deploy again before you see the change on your website.

**FYI:** You can change the name of the */fcms_serve/* directory, and thus its corresponding url,
 at any time if you feel like you need to 
(e.g. */public/fcms/someother_serve/* and *example.com/fcms/someother_serve/<file_name>*).
Everything will probably be fine but, it is not recommended.

### 7. Next Steps
Your back-end setup is now complete. Mostly.

Go to *FCMS_url/fcms_admin/*, or *example.com/fcms/fcms_admin/* in this case,
to finish setting up and start designing your website. 

#### FCMS URL Setting
On this page, the most important setting is the "*FCMS url*" setting.

Make sure you including the full url with the trailing slash (e.g. *https://example.com/fcms/*).
Even if FCMS is installed at the root of your domain (e.g. *https://example.com/*).

Make sure you also include the "https://" at the beginning. Notice that you don't need the "www." 
if your domain is a "naked domain", however you can include it if you want to.

Again, you can use your Firebase-provided "*Hosting URL*" 
if you have not yet connected a custom domain to your Firebase project,
but remember to change it later when you have connected a custom domain.

If you mess this up, your website **will be buggy or break** until you fix it.

**FYI:** Because of the way Firebase hosting works, 
all of the files in your FCMS directory are available publicly on the web, including this one.
So, if you go to *FCMS_url/README.md* or *example.com/fcms/README.md* in this case,
you will find this file. If that bothers you, just delete this README file, you're done with it anyway.
For that matter, you can delete every file in this directory except for the following:

```
index.html
settings.js
fcms_admin/
fcms_serve/
```

As long as you have those four files and folders, your website will run just fine. 
However, you might want to keep the other templates in case you decide to change your template later on.
FCMS is designed to allow you to easily change your template at any time.
You will learn how to use templates on the *fcms_admin* page. 

**Another FYI:** You can also change your FCMS directory location, and thus your FCMS url, 
at any time if you feel like you need to
(e.g. "*/public/somewhereelse/*" and "example.com/somewhereelse/").
Again, chances are everything will be fine, but this is not recommended. 
If you do this, make sure you update the "*FCMS url*" in your "*fcms_admin*" settings,
otherwise your website **will break**. 
 

Thanks for your time and have fun making your website.



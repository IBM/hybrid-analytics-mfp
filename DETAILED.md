## Steps
1. [Use Ionic-MFP-App as a starting point for this project](#step-1-use-ionic-mfp-app-as-a-starting-point-for-this-project)
2. [Enable monitoring using MobileFoundation Analytics](#step-2-enable-monitoring-using-mobilefoundation-analytics)



## Step 1. Use Ionic-MFP-App as a starting point for this project

This project builds on top of the app built in https://github.com/IBM/Ionic-MFP-App. In this code pattern, we will enhance the 
app with following user authentication mechanisms:

* Enterprise login by connecting to on-premise LDAP server using Secure Gateway.

* Social login: Google login and Facebook login.

Copy Ionic Mobile app and Mobile Foundation adapters from parent repo as per instructions in 
http://bit-traveler.blogspot.in/2012/08/git-copy-file-or-directory-from-one.html as shown below.

* Create your repo on [Github.com](https://github.com) and add `README.md` file. Clone your new repo.

```
$ git clone https://github.com/<your-username>/<your-new-repo-name>.git
```

* Make a git format-patch for the entire history of the sub-directories that we want as shown below.

<pre><code>
$ mkdir gitpatches
$ git clone https://github.com/IBM/Ionic-MFP-App.git
$ cd Ionic-MFP-App
$ git format-patch -o ../gitpatches/ --root IonicMobileApp/ MobileFoundationAdapters/
</code></pre>

* Import the patches into your new repository as shown below.

<pre><code>
$ cd ../<your-new-repo-name>
$ git am ../gitpatches/*
$ git push
</code></pre>

## Step 2. Enable monitoring using MobileFoundation Analytics


1.Install the cordova plugin `cordova-plugin-mfp-analytics` by using the command below

<pre><code>
$ cd IonicMobileApp
$ ionic cordova plugin add  cordova-plugin-mfp-analytics
</code></pre>

2.Add the following code in login.ts file under the `setLoginSuccessCallback` method

<pre><code>
this.authHandler.setLoginSuccessCallback(() => {        
        WL.Analytics.log({'fromPage':'LoginPage','toPage':'HomePage'});
        let view = this.navCtrl.getActive();
        if (!(view.instance instanceof HomePage)) {
          this.navCtrl.setRoot(HomePage);
        }
      }); 
</code></pre>

Add the following code in the `ionViewDidLoad` method

<pre><code>
WL.Analytics.send();
WL.Logger.send();
</code></pre>

3.Add the following code in home.ts file under the `ionViewDidLoad` method
<pre><code>
WL.Analytics.send();
WL.Logger.send();

</code></pre>

To capture the custom data on page Transition details modify the `itemClick` method in home.ts file, as given below
<pre><code>
itemClick(grievance) {
    WL.Analytics.log({'fromPage':'HomePage','toPage':'ProblemDetailPage'});
    this.navCtrl.push(ProblemDetailPage, { grievance: grievance, baseUrl: this.objectStorageAccess.baseUrl });
  }
</code></pre>

and modify the `reportNewProblem` method as below
<pre><code>
reportNewProblem(){
    WL.Analytics.log({'fromPage':'HomePage','toPage':'NewProblemPage'});
    this.navCtrl.push(ReportNewPage);
  }
</code></pre>

4.To capture the `username` information for the custom chart, add the following code in the success callback of `WLAuthorizationManager.login` in the auth-handler.ts file

<pre><code>
 WLAuthorizationManager.login(this.securityCheckName, {'username':username, 'password':password})
      .then(
        (success) => {
          console.log('--> AuthHandler: login success');
          WL.Analytics.log({'username':  username});           
          console.log("username:"+username);
        
        },
        (failure) => {
          console.log('--> AuthHandler: login failure: ' + JSON.stringify(failure));
          self.loginFailureCallback(failure.errorMsg);
        }
      );
</code</pre>
5.Add the following code to report-new.ts file `initAuthChallengeHandler` method, to capture the custom data on page Transition details 

<pre><code>
initAuthChallengeHandler() {
    this.authHandler.setHandleChallengeCallback(() => {
      WL.Analytics.log({'fromPage':'NewProblemPage','toPage':'LoginPage'});
      this.navCtrl.push(LoginPage, { isPushed: true, fixedUsername: this.authHandler.username });
    });
    this.authHandler.setLoginSuccessCallback(() => {
      let view = this.navCtrl.getActive();
      if (view.instance instanceof LoginPage) {
        this.navCtrl.pop().then(() =>{
          this.loader = this.loadingCtrl.create({
            content: 'Uploading data to server. Please wait ...'
          });
          this.loader.present();
        });
      }
    });
  }
</code></pre>

In the same file, include the Analytics send API call in the `ionViewDidLoad` method
<pre><code>
ionViewDidLoad() {
    console.log('--> ReportNewPage ionViewDidLoad() called');
    this.createMap();
    this.initAuthChallengeHandler();
    WL.Analytics.send();
    WL.Logger.send();
  }
</code></pre>

6.To add the in app feedback feature to the app, first add a button in the home.html file, next to the add button.

<pre><code>
&lt;ion-buttons end&gt;
      &lt;button ion-button icon-only (click)="reportNewProblem()"&gt;
        &lt;ion-icon name="add"&gt;&lt;/ion-icon&gt;
&lt;/button&gt;
&lt;button ion-button icon-only (click)="feedback()"&gt;
        &lt;ion-icon name="list-box"&gt;&lt;/ion-icon&gt;
&lt;/button&gt;
</code></pre>

Now add the `feedback` method implementation to the home.ts file
<pre><code>
feedback(){
    WL.Analytics.triggerFeedbackMode(); 
  }
</code></pre>


Yii Twitter Oauth

Installation 

Download and extract extension files to the directory protected/extensions
In your protected/config/main.php, add the following:
'components'=>array(
    //----
      'twitter' => array(
                'class' => 'ext.yiitwitteroauth.YiiTwitter',
                'consumer_key' => 'YOUR TWITEER KEY',
                'consumer_secret' => 'YOUR TWITTER SECRET',
                'callback' => 'YOUR CALLBACK URL',
            ),
        //-----
        ),
Usage 

public function actionIndex()
     {
     $twitter = Yii::app()->twitter->getTwitter();  
     $request_token = $twitter->getRequestToken();
 
         //set some session info
         Yii::app()->session['oauth_token'] = $token =           $request_token['oauth_token'];
         Yii::app()->session['oauth_token_secret'] = $request_token['oauth_token_secret'];
 
        if($twitter->http_code == 200){
            //get twitter connect url
            $url = $twitter->getAuthorizeURL($token);
            //send them
            $this->redirect($url);
        }else{
            //error here
            $this->redirect(Yii::app()->homeUrl);
        }
 
 
    }
This action will redirect you to the twitter app authentcation url. If the user authenticate your application,you will redirect back to your callback url

In your callback url action

public function actionTwitterCallBack() {   
        /* If the oauth_token is old redirect to the connect page. */
        if (isset($_REQUEST['oauth_token']) && Yii::app()->session['oauth_token'] !== $_REQUEST['oauth_token']) {
            Yii::app()->session['oauth_status'] = 'oldtoken';
        }
 
        /* Create TwitteroAuth object with app key/secret and token key/secret from default phase */
        $twitter = Yii::app()->twitter->getTwitterTokened(Yii::app()->session['oauth_token'], Yii::app()->session['oauth_token_secret']);   
 
        /* Request access tokens from twitter */
        $access_token = $twitter->getAccessToken($_REQUEST['oauth_verifier']);
 
        /* Save the access tokens. Normally these would be saved in a database for future use. */
        Yii::app()->session['access_token'] = $access_token;
 
        /* Remove no longer needed request tokens */
        unset(Yii::app()->session['oauth_token']);
        unset(Yii::app()->session['oauth_token_secret']);
 
        if (200 == $twitter->http_code) {
            /* The user has been verified and the access tokens can be saved for future use */
            Yii::app()->session['status'] = 'verified';
 
            //get an access twitter object
            $twitter = Yii::app()->twitter->getTwitterTokened($access_token['oauth_token'],$access_token['oauth_token_secret']);
 
            //get user details
            $twuser= $twitter->get("account/verify_credentials");
            //get friends ids
            $friends= $twitter->get("friends/ids");
                        //get followers ids
                $followers= $twitter->get("followers/ids");
            //tweet
                        $result=$twitter->post('statuses/update', array('status' => "Tweet message"));
 
        } else {
            /* Save HTTP status for error dialog on connnect page.*/
            //header('Location: /clearsessions.php');
            $this->redirect(Yii::app()->homeUrl);
        }
    }

# Oauth-Working

## How OAuth works when the user clicks on the login with facebook or google?

* first in registration.jsp in the last few lines you can see
```
script(src='https://cdn.auth0.com/js/auth0/9.3.1/auth0.min.js')
var auth0 = new auth0.WebAuth({
    // The Auth0 domain - this is your tenant-specific domain in Auth0
    domain: 'filerightapp.auth0.com',
    
    // The application's unique identifier in Auth0
    clientID: 'rRJodW5dQ7E9Pwwb24anrhl601VvAzfp',
    
    // The type of response we want from Auth0 - 'code' means we're using the authorization code flow
    responseType: 'code',
    
    // Where Auth0 should redirect after authentication is complete
    redirectUri: window.location.protocol + "//" + window.location.hostname + "/registration/callback.do",
    
    // The permissions we're requesting
    scope: 'openid profile email',
    
    // State parameter to prevent CSRF attacks
    state: $.urlParameter('next')
});
```
* This is where the authentication based imports and and the type, token and other things are set.
* And also you can see there is a redirect url (i will come to that in just a bit)
* You can see the connection and the url cleaning in `login.js` and `signup.js`
* Login.js
```login.js
$(document).ready(function(){
$('#login-google').click(function(e) {
    e.preventDefault();
    auth0.authorize({
      connection: 'google-oauth2',
    });
});


$('#login-facebook').click(function(e) {
      e.preventDefault();
      auth0.authorize({
        connection: 'facebook',
      });
});

if (window.location.hash == "#_=_")
{
  window.location.hash = "";
}
});
```
* Signup.js
```signup.js
$(document).ready(function(){
		$('#login-google').click(function(e) {
		    e.preventDefault();
		    auth0.authorize({
		 	   connection: 'google-oauth2',
			});
		});

		$('#login-facebook').click(function(e) {
		    e.preventDefault();
		    auth0.authorize({
		    connection: 'facebook',
		  });
		});

		if (window.location.hash == "#_=_")
		{
			window.location.hash = "";
		}

	});
```

* So as you can see in the above code it prevents the default behaviour, and the connection is being set.
* And also you can see in `signup.js` the user should accept the privacy policy to continue their signup with (facebook or google)
```signup.js
	$.validator.addMethod("validSmsConsent", function (value, element) { 
		var isPrivacyAgree = $("input[name='privacyagree']").prop('checked');
		var isSmsAgree = $("input[name='smsAgree']").prop('checked');
		return isPrivacyAgree || isSmsAgree ;
	}, smsConsentAgreeError);
```
* And now once the auth0 does its process it will get redirected to `callback.do` in this line you can see it `redirectUri: window.location.protocol + "//" + window.location.hostname + "/registration/callback.do",`
* And now we go to registration-webapp repo
* Inside that repo we can see inside `src/main/java/com/formsdirectinc/ui/controllers/auth0/CallbackController.java`
* And the above controller uses the service package from `src/main/java/com/formsdirectinc/services/AuthService.java`
* Do note the backend controller always gets hits (even if its a login request too or signup request)


### And inside the controller 

* If the id_token is null or missing, redirect the user to "logincheck.do?next=%s" (the login page).

* Otherwise, decode the id_token.

* Then, try to retrieve the existing user profile (i.e., customerSignup).

* If customerSignup == null, this means it is the user’s first time (user does not exist yet), so create a new CustomerSignup instance and register the user.

* If customerSignup != null, the user already exists and you proceed as normal (no registration needed).if the `id_token` is null we redirect the user to `"logincheck.do?next=%s` else it tries to decode the token_id and if its (customerSignup == null) there its not the user's first time else its the users first time and Creates a new CustomerSignup instance.

* the getTokens is declared in the `AuthService` which is being used by the `CallbackController.java`

| Condition                     | Meaning                     | Action                       |
| ----------------------------- | --------------------------- | ---------------------------- |
| `id_token` is null or missing | No valid token from OAuth   | Redirect to login page       |
| `customerSignup == null`      | User not found (first time) | Create new user & register   |
| `customerSignup != null`      | User exists                 | Proceed without registration |

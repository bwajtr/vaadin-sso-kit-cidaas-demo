# Vaadin SSO Kit Cidaas demo

This project showcases a minimal setup of [Cidaas](https://www.cidaas.com/), so you can use it together with Vaadin SSO Kit as an SSO identity manager in the Vaadin
application. Please take into account that the tutorial was created in January 2023, and the involved technologies may have changed since then. Especially the
screenshots do not have to be 100% accurate anymore.
 
* SSO Kit documentation: https://vaadin.com/docs/latest/tools/sso
* Cidaas documentation: https://docs.cidaas.com/

The demo consists of two views:

* Public view, which is accessible without login and which is mapped to http://localhost:8080/  
  [<img height="100px" src="tutorial/public.png?raw=true"/>](tutorial/public.png?raw=true)
* Private view, which is protected by @PermitAll and requires you to log in. This view is accessible at http://localhost:8080/private  
  [<img height="100px" src=".\tutorial\private.png"/>](.\tutorial\private.png)

You should be redirected to the configured Oauth Provider Login page when:
* you either attempt to enter the Private view, 
* or when you want to explicitly log in using the button in the lower-left corner of the screen

You should be able to log out using the user dropdown button in the lower-left corner of the screen.

## Cidaas setup

To run the demo, you have to configure Cidaas first. In this tutorial, we will do the following:

- create a Cidaas account and Cidaas IAM instance in it
- create at least one user in this new IAM instance
- create an OIDC client (an "App" in Cidaas terms) configuration so our application can use the IAM instance for login

1. Go to https://www.cidaas.com/ → Pricing → Choose "cidaas IAM" product → Choose Free Tier → Create an account
2. After registering, create a new company in Cidaas administration
3. After the company has been created, choose "Individual / Free" plan  
   [<img height="100px" src=".\tutorial\plan.png"/>](.\tutorial\plan.png)
4. That will bring you to the instance creation screen  
   [<img height="100px" src=".\tutorial\instance.png"/>](.\tutorial\instance.png)
5. Wait until the instance is created and then go to the Admin portal (e.g. https://xxxxxxx-prod.cidaas.eu/admin-ui - just replace xxxxxxxx with the name of your instance)  
   [<img height="100px" src=".\tutorial\instance.png"/>](.\tutorial\dashboard.png)
6. Go to Users → "Create user"
7. Fill in user details as you like, for example:  
       [<img height="100px" src=".\tutorial\user.png"/>](.\tutorial\user.png)
8. Go to Apps → App Settings → Click on the "Create New App" button
9. Fill in the details as you like. We've selected the "Single page" app type, but I believe other values would work as well.  
   [<img height="100px" src=".\tutorial\appcreation.png"/>](.\tutorial\appcreation.png)
10. Click next, and on the following App settings screen, fill in the following fields:
    1. Scope → openid, profile, email, roles
    2. Redirect URLs: http://localhost:8080/login/oauth2/code/cidaas
    3. Allow logout URLs: http://localhost:8080  
       [<img height="100px" src=".\tutorial\appsettings.png"/>](.\tutorial\appsettings.png)
11. Click next, and on the last page of the app creation wizard, fill in any details about your company:  
    [<img height="100px" src=".\tutorial\appcompany.png"/>](.\tutorial\appcompany.png)
12. Go to Apps → App Settings and lookup your newly created app
13. Click on the edit button to open the details of the app
14. Copy app "Client id" and "Client Secret" and use them in application.properties in this project (see Vaadin application setup below)
15. Note that at the time of this tutorial creation, there was a problem in Cidaas app processing, where the signing key of your newly created app was not immediately added to https://xxxxxxx-prod.cidaas.eu/.well-known/jwks.json. This JSON is used as a source of known singing keys by the SSO Kit, and a missing key caused SSO Kit not to work. Therefore, it was necessary to wait some time (24 hours) before the key was added to this JSON. This problem was reported to Cidaas and is likely already fixed by now.
    * You can verify that all is ok by verifying that the app signing key ID is present in https://xxxxxxx-prod.cidaas.eu/.well-known/jwks.json. If it's not there, then you have to wait. The key id can be found in app settings under Advanced Settings -> Certificates:  
    [<img height="100px" src=".\tutorial\kid.png"/>](.\tutorial\kid.png)  
16. Go to Advanced settings of the app   
    [<img height="100px" src=".\tutorial\advanced.png"/>](.\tutorial\advanced.png)
17. In the OAuth2/OIDC Settings, set the following values:
    * Response Types: only "code"
    * Grant Types: only "authorization_code"
    * Backchannel logout URI: http://localhost:8080/logout/back-channel/cidaas
    * Backchannel logout session: set to enabled  
    [<img height="100px" src=".\tutorial\oauthoidc.png"/>](.\tutorial\oauthoidc.png)
18. No other changes in the Advanced Settings of the app were necessary, and all other options were set to their default values
19. That's it. Your Cidaas provider is now ready 

## Vaadin application setup

You must modify the application.properties and fill in the Cidaas-specific values to the oauth2 configuration for this application to work.
You can find all Cidaas-specific values in the configuration of your Cidaas instance (https://xxxxxxx-prod.cidaas.eu/admin-ui ). Go to App Settings and open the detail of your app there:   
```properties
spring.security.oauth2.client.provider.cidaas.issuer-uri=[put issuer URI here, e.g. https://xxxxxxx-prod.cidaas.eu]
spring.security.oauth2.client.registration.cidaas.client-id=[client id can be found in cidaas app details]
spring.security.oauth2.client.registration.cidaas.client-secret=[client secret can be found in cidaas app details]
```

Keep the other two configuration settings as they are:
```properties
spring.security.oauth2.client.registration.cidaas.scope=profile,openid,email,roles
vaadin.sso.login-route=/oauth2/authorization/cidaas
```

## Running the application

The project is a standard Maven project. To run it from the command line,
type `mvnw` (Windows), or `./mvnw` (Mac & Linux), then open
http://localhost:8080 in your browser.

You can also import the project to your IDE of choice as you would with any
Maven project. Read more on [how to import Vaadin projects to different
IDEs](https://vaadin.com/docs/latest/guide/step-by-step/importing) (Eclipse, IntelliJ IDEA, NetBeans, and VS Code).


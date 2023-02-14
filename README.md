# Vaadin SSO Kit cidaas demo

This project showcases a minimal setup of [cidaas](https://www.cidaas.com/), so you can use it together with Vaadin SSO Kit as an SSO identity manager in the Vaadin
application. Please take into account that the tutorial was created in January 2023, and the involved technologies may have changed since then. Especially the
screenshots do not have to be 100% accurate anymore.
 
* SSO Kit documentation: https://vaadin.com/docs/latest/tools/sso
* cidaas documentation: https://docs.cidaas.com/

The demo consists of two views:

* Public view, which is accessible without login and which is mapped to http://localhost:8080/  
  [<img height="100px" src="tutorial/public.png?raw=true"/>](tutorial/public.png?raw=true)
* Private view, which is protected by @PermitAll and requires you to log in. This view is accessible at http://localhost:8080/private  
  [<img height="100px" src="tutorial/private.png?raw=true"/>](tutorial/private.png?raw=true)

You should be redirected to the configured Oauth Provider Login page when:
* you either attempt to enter the Private view, 
* or when you want to explicitly log in using the button in the lower-left corner of the screen

You should be able to log out using the user dropdown button in the lower-left corner of the screen.

## cidaas setup

To run the demo, you have to configure cidaas first. In this tutorial, we will do the following:

- create a cidaas account and cidaas IAM instance in it
- create at least one user in this new IAM instance
- create an OIDC client (an "App" in cidaas terms) configuration so our application can use the IAM instance for login

1. Go to https://www.cidaas.com/ → Pricing → Choose "cidaas IAM" product → Choose Free Tier → Create an account
2. After registering, create a new company in cidaas administration
3. After the company has been created, choose "Individual / Free" plan  
   [<img height="100px" src="tutorial/plan.png?raw=true"/>](tutorial/plan.png?raw=true)
4. That will bring you to the instance creation screen  
   [<img height="100px" src="tutorial/instance.png?raw=true"/>](tutorial/instance.png?raw=true)
5. Wait until the instance is created and then go to the Admin portal (e.g. https://xxxxxxx-prod.cidaas.eu/admin-ui - just replace xxxxxxxx with the name of your instance)  
   [<img height="100px" src="tutorial/instance.png?raw=true"/>](tutorial/dashboard.png?raw=true)
6. Go to Users → "Create user"
7. Here we will create a user, which we will use to test the demo application login. Fill in user details as you like, for example:  
       [<img height="100px" src="tutorial/user.png?raw=true"/>](tutorial/user.png?raw=true)
8. Go to Apps → App Settings → Click on the "Create New App" button
9. Here we will create an instance of an OIDC client to which our demo application will connect to. Fill in the details as you like. We've selected the "Single page" app type, but I believe other values would also work.  
   [<img height="100px" src="tutorial/appcreation.png?raw=true"/>](tutorial/appcreation.png?raw=true)
10. Click next, and on the following App settings screen, fill in the following fields:
    1. Scope → openid, profile, email, roles
    2. Redirect URLs: http://localhost:8080/login/oauth2/code/cidaas
    3. Allow logout URLs: http://localhost:8080  
       [<img height="100px" src="tutorial/appsettings.png?raw=true"/>](tutorial/appsettings.png?raw=true)
11. Click next, and on the last page of the app creation wizard, fill in any details about your company:  
    [<img height="100px" src="tutorial/appcompany.png?raw=true"/>](tutorial/appcompany.png?raw=true)
12. Go to Apps → App Settings and lookup your newly created app
13. Click on the edit button to open the details of the app
14. Copy app "Client id" and "Client Secret" and use them in application.properties in this project (see Vaadin application setup below)
15. Note that at the time of this tutorial creation, there was a problem in cidaas app processing, where the signing key of your newly created app was not immediately added to https://xxxxxxx-prod.cidaas.eu/.well-known/jwks.json. This JSON is used as a source of known singing keys by the SSO Kit, and a missing key caused SSO Kit not to work. Therefore, it was necessary to wait some time (24 hours) before the key was added to this JSON. This problem was reported to cidaas and is likely already fixed by now.
    * You can verify that all is ok by checking that the app signing key ID is present in https://xxxxxxx-prod.cidaas.eu/.well-known/jwks.json. If it's not there, then you have to wait. The key id can be found in app settings under Advanced Settings -> Certificates:  
    [<img height="100px" src="tutorial/kid.png?raw=true"/>](tutorial/kid.png?raw=true)  
16. Go to Advanced settings of the app   
    [<img height="100px" src="tutorial/advanced.png?raw=true"/>](tutorial/advanced.png?raw=true)
17. In the OAuth2/OIDC Settings, set the following values:
    * Response Types: only "code"
    * Grant Types: only "authorization_code"
    * Backchannel logout URI: http://localhost:8080/logout/back-channel/cidaas
    * Backchannel logout session: set to enabled  
    [<img height="100px" src="tutorial/oauthoidc.png?raw=true"/>](tutorial/oauthoidc.png?raw=true)
18. No other changes in the Advanced Settings of the app were necessary, and all other options were set to their default values
19. That's it. Your cidaas provider is now ready 

## Vaadin application setup

First of all, clone this repository to your local computer. This repository contains a preconfigured Vaadin application, which already contains dependencies to SSO Kit and has also authentication set up. Please see top of this page for overview of how the demo application looks like and how it behaves.

You must modify the application.properties and fill in the cidaas-specific values to the oauth2 configuration for this application to work.
You can find all cidaas-specific values in the configuration of your cidaas instance (https://xxxxxxx-prod.cidaas.eu/admin-ui ) → go App Settings → find app which you created in previous steps → click on edit button -> "Client id" and "Client Secret" can be found there. Please modify the following properties in application.properties:   

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


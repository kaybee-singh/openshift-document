
# Entra ID and Openshift configuration

A brief description of what this project does and who it's for

1. Sign in to Azure Portal

2. Click on `Azure Entra ID`

3. If you are using a free tier azure account then you may already have a `Default Directory` created which you can utitlize. If you want to create a new directory you can also do it, however, as far as I know we can not create new directory while using a free acount.

4. Ensure you have some users, if you do not have then create few users.

5. Create some groups as well which we will use for groupclaim later.

6. Add required users to these groups.

7. Register the OCP cluster as service 

Navigate to Azure Portal >> Azure Entra ID >> Your Directory >> Manage >> App registration >> New Registration >> Provide a name >> Tick on "Accounts in this organizational directory only (Default Directory only - Single tenant)" >> In Redirect URL select web platform and provide the https://oauth-url/oauth2callback/azure as redirect URL. Here azure at end of URL is the oauth which you have created in the OCP cluster.

8. 

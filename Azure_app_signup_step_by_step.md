```python
# Program: Accessing OneDrive via Graph API
# Author: Pranab Das (Twitter: @Pranab_Das)
# Version: 20191104 
```

# Azure App Signup step by step

1. Go to https://portal.azure.com 

![01.PNG](attachment:01.PNG)

2. Navigate to Azure Active Directory

![02.PNG](attachment:02.PNG)


3. Select App Registration

![03.PNG](attachment:03.PNG)

4. Click New Registration

![04.PNG](attachment:04.PNG)

5. Give a name to your app, set the redirect URL, and hit Registration button. 

![05.PNG](attachment:05.PNG)

6. Note down the client ID and go to API permissions. 

![06.PNG](attachment:06.PNG)

7. Click Add permissions, select Microsoft Graph. 

![07.PNG](attachment:07.PNG)

8. Choose Delegated permission. 

![08.PNG](attachment:08.PNG)

9. We will add Files.ReadWrite.All for our purpose. 

![09.PNG](attachment:09.PNG)

10. Now go to Authentication tab, and enable Access token. Click the save button, and now we are all set to go.

![10.PNG](attachment:10.PNG)


```python

```

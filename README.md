# Access OneDrive via Graph API (Python code)
Upload, download, rename your files and many more to your OneDrive both personal
and business accounts using Microsoft Graph API (Python code).

First you need to [register your app](Azure_app_signup_step_by_step.md) in the
Azure portal.

```python
# Program: Accessing OneDrive via Graph API
# Author: Pranab Das (GitHub: @pranabdas)
# Version: 20210820
```

```python
# requirements
import requests
import json
import urllib
import os
from getpass import getpass
import time
from datetime import datetime
```


## Get Access token

### Token flow authentication

```python
URL = 'https://login.microsoftonline.com/common/oauth2/v2.0/authorize'
client_id = "362422eb-d9d6-4245-9eca-2be5cf256450"
permissions = ['files.readwrite']
response_type = 'token'
redirect_uri = 'http://localhost:8080/'
scope = ''
for items in range(len(permissions)):
    scope = scope + permissions[items]
    if items < len(permissions)-1:
        scope = scope + '+'

print('Click over this link ' +URL + '?client_id=' + client_id + '&scope=' + scope + '&response_type=' + response_type+\
     '&redirect_uri=' + urllib.parse.quote(redirect_uri))
print('Sign in to your account, copy the whole redirected URL.')
code = input("Paste the URL here :")
token = code[(code.find('access_token') + len('access_token') + 1) : (code.find('&token_type'))]
URL = 'https://graph.microsoft.com/v1.0/'
HEADERS = {'Authorization': 'Bearer ' + token}
response = requests.get(URL + 'me/drive/', headers = HEADERS)
if (response.status_code == 200):
    response = json.loads(response.text)
    print('Connected to the OneDrive of', response['owner']['user']['displayName']+' (',response['driveType']+' ).', \
         '\nConnection valid for one hour. Reauthenticate if required.')
elif (response.status_code == 401):
    response = json.loads(response.text)
    print('API Error! : ', response['error']['code'],\
         '\nSee response for more details.')
else:
    response = json.loads(response.text)
    print('Unknown error! See response for more details.')
```

    Click over this link https://login.microsoftonline.com/common/oauth2/v2.0/authorize?client_id=362422eb-d9d6-4245-9eca-2be5cf256450&scope=files.readwrite&response_type=token&redirect_uri=http%3A//localhost%3A8080/
    Sign in to your account, copy the whole redirected URL.
    Paste the URL here :http://localhost:8080/#access_token=EwBYA8l6BAAUO9chh8cJscQLmU%2bLSWpbnr0vmwwAAXbgH8Q919pMC8ErHXfcrM/uuPvmmsIyKar8nmAp1mvv/0QwrjAkSBM8Y6sJqpPEPrGKBrDHairoIVrQK7FhGCtYLGEy3P88wnaKGr4NYygckbi2g6P4S5KPt7d3m3/7XuAhLips6jwD3X8g89a72SajQaa1xbPFw2TfUed/UG6kqUxzlVUy4gkPCBMm%2bizQ3mP7lXRbmeXqCY5omTSQz6djvkCcjXf9TqC1WfVpRLHGc7yLUPcg15nGmdMfwRxWDxYi8rlD34Y0cVYt4KYw3B2VkdxyMvCWqARgauWApLYTFopGZIUQ8M0Fggb89PncdhHInKehD8Rp7rkBJIhkfIIDZgAACBnA%2btK5eKnhKAJmVnI6%2b2MwF54q9NR04O9xTn0Py/uOJPpyGeAtMRBgHTSI6Eh/Bwr/ybQh2TMbfNBbqpOEjPYx0KDhDhrcS1LldJKKoYj2EOREEkwKZNKYfmTdO1jWQ/MohoOFawGB29gdSyJxkqgRHrC2RedL3wFYMOxE78ehVvfCl1/UqBR4Z4ypPMZ%2bsFlyCOCQ6E2fiLyJt0AF5wZencLGoAhXdlh/gIDVZuSZBVQXZuEP19d07IGqLmwDoVnhecniQMjy3cLVQ5v0vlT15b/GpuESNhtgrdQwGT307F9gHPVO6U9UMzfT1iEx%2bjqOBR5paJz8OiIZOG3SZmqZFB4c606Vycio3BVnkXyNlf6kBZfJMNVLB4IubmXSbM%2byFjadP1Cq3pc2dsQRx%2bMqhYCDYS%2bYm4yBqHW0r/XfLrs/QmiIgVtAneHyw99TYVFEO2sqM3MLPZS1W8Wm0cFvwvfxuDI4cDllhjkX5jPy0wSD35c9rDZ8gwWdpR1x6Xc/XaTsn1eQEb9CcsZxyyIeJ6SA9t2kysZ1udqbu4xqIuMt3QtIdYA3tDDIg9IPJtF7tuC48G5tjm7BlfOANxhfsg8USYtjovd3KC4Jl4w87OBeiGrRQgaoI4pEfZVgYPa4TuOYe6ZuYCEyNW8GYumvetzmkRkrMBwicAqJ5KXVco5Lird6gCbQSWFjBTfxtdzXFKCiEgcQSDAm88xTwN2LvBKsrbV17QkEyNYqcVzo1CdsAg%3d%3d&token_type=bearer&expires_in=3600&scope=Files.ReadWrite%20Files.ReadWrite.All
    Connected to the OneDrive of  ( personal ).
    Connection valid for one hour. Reauthenticate if required.


Looks all right. We have got the access token, and included in the HEADERS. You
can print response to see more. Go ahead with OneDrive operations.

### Code flow authentication

Code flow returns both `access_token` and `refresh_token` which can be used to
request new `access_token` and `refresh_token` for persistent session. If you
are using organization account, you might require consent of organization
administrator.

```python
# Get code
URL = 'https://login.microsoftonline.com/common/oauth2/v2.0/authorize'
client_id = "362422eb-d9d6-4245-9eca-2be5cf256450"
permissions = ['offline_access', 'files.readwrite', 'User.Read']
response_type = 'code'
redirect_uri = 'http://localhost:8080/'
scope = ''
for items in range(len(permissions)):
    scope = scope + permissions[items]
    if items < len(permissions)-1:
        scope = scope + '+'

print('Click over this link ' +URL + '?client_id=' + client_id + '&scope=' + scope + '&response_type=' + response_type+\
     '&redirect_uri=' + urllib.parse.quote(redirect_uri))
print('Sign in to your account, copy the whole redirected URL.')
code = getpass("Paste the URL here :")
code = code[(code.find('?code') + len('?code') + 1) :]

URL = 'https://login.microsoftonline.com/common/oauth2/v2.0/token'

response = requests.post(URL + '?client_id=' + client_id + '&scope=' + scope + '&grant_type=authorization_code' +\
     '&redirect_uri=' + urllib.parse.quote(redirect_uri)+ '&code=' + code)
```

```python
# Get token
data = {
    "client_id": client_id,
    "scope": permissions,
    "code": code,
    "redirect_uri": redirect_uri,
    "grant_type": 'authorization_code',
    "client_secret": client_secret
}

response = requests.post(URL, data=data)

token = json.loads(response.text)["access_token"]
refresh_token = json.loads(response.text)["refresh_token"]
```

```python
# Refresh token
def get_refresh_token():
    data = {
        "client_id": client_id,
        "scope": permissions,
        "refresh_token": refresh_token,
        "redirect_uri": redirect_uri,
        "grant_type": 'refresh_token',
        "client_secret": 'xxxx-yyyy-zzzz',
    }

    response = requests.post(URL, data=data)

    token = json.loads(response.text)["access_token"]
    refresh_token = json.loads(response.text)["refresh_token"]
    last_updated = time.mktime(datetime.today().timetuple())

    return token, refresh_token, last_updated
```

```python
token, refresh_token, last_updated = get_refresh_token()
```

If you have a large data to upload, you may use below mock code inside your
upload loop:
```python
elapsed_time = time.mktime(datetime.today().timetuple()) - last_updated

if (elapsed_time < 45*60*60):
    do_something()
else if (elapsed_time < 59*60*60):
    token, refresh_token, last_updated = get_refresh_token()
else:
    go_to_code_flow()
```

## OneDrive operations

```python
URL = 'https://graph.microsoft.com/v1.0/'

HEADERS = {'Authorization': 'Bearer ' + token}

response = requests.get(URL + 'me/drive/', headers = HEADERS)
if (response.status_code == 200):
    response = json.loads(response.text)
    print('Connected to the OneDrive of', response['owner']['user']['displayName']+' (',response['driveType']+' ).', \
         '\nConnection valid for one hour. Refresh token if required.')
elif (response.status_code == 401):
    response = json.loads(response.text)
    print('API Error! : ', response['error']['code'],\
         '\nSee response for more details.')
else:
    response = json.loads(response.text)
    print('Unknown error! See response for more details.')
```

### List folders under root directory

We will print both directory `name` and `item-d`
```python
items = json.loads(requests.get(URL + 'me/drive/root/children', headers=HEADERS).text)
items = items['value']
for entries in range(len(items)):
    print(items[entries]['name'], '| item-id >', items[entries]['id'])
```

    Documents | item-id > C1465DBECD7188C9!103
    Pictures | item-id > C1465DBECD7188C9!104
    Getting started with OneDrive.pdf | item-id > C1465DBECD7188C9!102


### Create new folder (in the root directory)

```python
url = URL + 'me/drive/root/children/'
body = {
    "name": "New_Folder",
    "folder": {},
    "@microsoft.graph.conflictBehavior": "rename"
}
response = json.loads(requests.post(url, headers=HEADERS, json=body).text)
```

Now lets list the directory again:

```python
items = json.loads(requests.get(URL + 'me/drive/root/children', headers=HEADERS).text)
items = items['value']
for entries in range(len(items)):
    print(items[entries]['name'], '| item-id >', items[entries]['id'])
```

    Documents | item-id > C1465DBECD7188C9!103
    New_Folder | item-id > C1465DBECD7188C9!106
    Pictures | item-id > C1465DBECD7188C9!104
    Getting started with OneDrive.pdf | item-id > C1465DBECD7188C9!102


Here we go, we have successfully created the folder New_Folder.


### List folders under a sub-folder (need to use item-id)
Note that if you need to create or list sub-folders, you need to use the
`item-id`. The `path/folder` notation does not work everywhere.


```python
url = URL + 'me/drive/items/C1465DBECD7188C9!106/children'
items = json.loads(requests.get(url, headers=HEADERS).text)
items = items['value']
for entries in range(len(items)):
    print(items[entries]['name'], '| item-id >', items[entries]['id'])
```

Well there are no files or folders under the `New_Folder`. Ok let's create one.

```python
url = URL + 'me/drive/items/C1465DBECD7188C9!106/children/'
data = {
    "name": "sub_folder",
    "folder": {},
    "@microsoft.graph.conflictBehavior": "rename"
}

response = json.loads(requests.post(url, headers=HEADERS, json = data).text)
```

Now let's print the list again.

```python
url = URL + 'me/drive/items/C1465DBECD7188C9!106/children'
items = json.loads(requests.get(url, headers=HEADERS).text)
items = items['value']
for entries in range(len(items)):
    print(items[entries]['name'], '| item-id >', items[entries]['id'])
```

    sub_folder | item-id > C1465DBECD7188C9!107


### Rename an item

```python
url = URL + 'me/drive/items/C1465DBECD7188C9!106'
body = {
    "name": "New_folder_2",
}
response = json.loads(requests.patch(url, headers=HEADERS, json = body).text)
```

```python
url = URL + 'me/drive/items/root/children'
items = json.loads(requests.get(url, headers=HEADERS).text)
items = items['value']
for entries in range(len(items)):
    print(items[entries]['name'], '| item-id >', items[entries]['id'])
```

    Documents | item-id > C1465DBECD7188C9!103
    New_folder_2 | item-id > C1465DBECD7188C9!106
    Pictures | item-id > C1465DBECD7188C9!104
    Getting started with OneDrive.pdf | item-id > C1465DBECD7188C9!102


### Move item
```python
# url = URL + 'me/drive/items/{item-id-of-item-to-be-moved}'
# provide item-id-of-destination-directory under parentReference in the body
url = URL + 'me/drive/items/C1465DBECD7188C9!106'
body = {
  "parentReference": {
    "id": "C1465DBECD7188C9!103"
  },
}
response = json.loads(requests.patch(url, headers=HEADERS, json=body).text)
```


### Delete item

```python
url = '/me/drive/items/C1465DBECD7188C9!106'
url = URL + url
confirmation = input('Are you sure to delete the Item? (Y/n):')
if (confirmation.lower()=='y'):
    response = requests.delete(url, headers=HEADERS)
    if (response.status_code == 204):
        print('Item gone! If need to recover, please check OneDrive Recycle Bin.')
else:
    print("Item not deleted.")
```

    Are you sure to delete the Item? (Y/n):y
    Item gone! If need to recover, please check OneDrive Recycle Bin.

```python
url = URL + 'me/drive/items/root/children'
items = json.loads(requests.get(url, headers=HEADERS).text)
items = items['value']
for entries in range(len(items)):
    print(items[entries]['name'], '| item-id >', items[entries]['id'])
```

    Documents | item-id > C1465DBECD7188C9!103
    Pictures | item-id > C1465DBECD7188C9!104
    Getting started with OneDrive.pdf | item-id > C1465DBECD7188C9!102


### Find item-id by item name

```python
items = json.loads(requests.get(URL + 'me/drive/items/root/children', headers=HEADERS).text)
look_for_item = 'Documents'
item_id = ''
items = items['value']
for entries in range(len(items)):
    if(items[entries]['name'] == look_for_item):
        item_id = items[entries]['id']
        print('Item-id of', look_for_item, ':', item_id)
        break
if(item_id==''):
            print(look_for_item, 'not found in the directory.')
```

    Item-id of Documents : C1465DBECD7188C9!103


### Upload file

```python
url = 'me/drive/root:/example_spectrum.txt:/content'
url = URL + url
content = open('example_spectrum.txt', 'rb')
response = json.loads(requests.put(url, headers=HEADERS, data = content).text)
```


```python
url = URL + 'me/drive/items/root/children'
items = json.loads(requests.get(url, headers=HEADERS).text)
items = items['value']
for entries in range(len(items)):
    print(items[entries]['name'], '| item-id >', items[entries]['id'])
```

    Documents | item-id > C1465DBECD7188C9!103
    Pictures | item-id > C1465DBECD7188C9!104
    example_spectrum.txt | item-id > C1465DBECD7188C9!108
    Getting started with OneDrive.pdf | item-id > C1465DBECD7188C9!102


### Access/Download data

```python
url = 'me/drive/root:/example_spectrum.txt:/content'
url = URL + url
data = requests.get(url, headers=HEADERS).text
```

You may like to save the data in a file in your local drive.


### Upload large files (Can be used to upload small files as well)
If you have files (probably larger than 4 MB), you need to create upload
sessions.


```python
url = 'me/drive/items/C1465DBECD7188C9!103:/large_file.dat:/createUploadSession'
url = URL + url
url = json.loads(requests.post(url, headers=HEADERS).text)
url = url['uploadUrl']
file_path = '/local/file/path/large_file.dat'
file_size = os.path.getsize(file_path)
chunk_size = 320*1024*10 # Has to be multiple of 320 kb
no_of_uploads = file_size//chunk_size
content_range_start = 0
if file_size < chunk_size :
    content_range_end = file_size
else :
    content_range_end = chunk_size - 1

data = open(file_path, 'rb')
while data.tell() < file_size:
    if ((file_size - data.tell()) <= chunk_size):
        content_range_end = file_size -1
        headers = {'Content-Range' : 'bytes '+ str(content_range_start)+ '-' +str(content_range_end)+'/'+str(file_size)}
        content = data.read(chunk_size)
        response = json.loads(requests.put(url, headers=headers, data = content).text)
    else:
        headers = {'Content-Range' : 'bytes '+ str(content_range_start)+ '-' +str(content_range_end)+'/'+str(file_size)}
        content = data.read(chunk_size)
        response = json.loads(requests.put(url, headers=headers, data = content).text)
        content_range_start = data.tell()
        content_range_end = data.tell() + chunk_size - 1
data.close()
response2 = requests.delete(url)
```


### OneDrive storage usage

```python
response = json.loads(requests.get(URL + 'me/drive/', headers = HEADERS).text)
used = round(response['quota']['used']/(1024*1024*1024), 2)
total = round(response['quota']['total']/(1024*1024*1024), 2)
print('Using', used, 'GB (', round(used*100/total, 2),'%) of total', total, 'GB.')
```

    Using 0.48 GB ( 9.6 %) of total 5.0 GB.

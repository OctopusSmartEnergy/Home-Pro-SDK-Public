# Octopus Home Pro Screen/LED API

## Introduction

The blue LEDs on the front of the Home Pro form a 24\*12 (x\*y) display matrix.
This display has some unique features which can be leveraged for showing visual
expressions.

The display architecture introduces the concept of screens/cards where a special
screen is allotted to the system. This is the same screen/card that shows you the
bluetooth/WiFi/cloud etc animated icons when you are setting up your Home Pro device.
The SDK has its own unique screen as well and a user can navigate between these
(system/SDK) screens using the touch button on the front of the Home Pro device.

The application/SDK screen cannot directly be accessed from applications/SDK and
hence the Octave OS implements RESTful APIs on the main system for client applications.

## The Screen API Endpoint

In order to access any RESTful API, the most basic part of the connection
mechanism is knowing the endpoint that hosts the API or say responds to it. For
the screen API to be accessed from the SDK the main system exposes an environment
variable **SCREEN_API_HOST** into the SDK environment.

For any python based applications, it can simply be retrieved as follows

```
import os

host = os.getenv('SCREEN_API_HOST')
```

The Home Pro SDK has its own special screen and access to this screen is managed
i.e. the APIs require authentication. For the authentication mechanism to work
correctly additional environment variables are exposed inside the SDK. The main
application boiler plate then looks like this

```
import os

# Create the base header
headers = {
    "Authorization": f"Basic {os.environ['AUTH_TOKEN']}"
}

# Get basic info from environment
host = os.getenv('SCREEN_API_HOST')
app_name = os.getenv('APPLICATION_NAME')
```

Notice how **APPLICATION_NAME** and **AUTH_TOKEN** are taken from the base environment.

## APIs

This section discusses the RESTful APIs that are available over the screen API
endpoint, how these are invoked and explains the output.

### Screen Info

The screen info API helps in getting the application screen ID, data/bitmap currently
being displayed and other information related to the application screen. Here's how
the screen ID is fetched using this API

```
import os, requests

# Create the base header
headers = {
    "Authorization": f"Basic {os.environ['AUTH_TOKEN']}"
}

# Get basic info from environment
host = os.getenv('SCREEN_API_HOST')
app_name = os.getenv('APPLICATION_NAME')
base_url = f"{host}/api/v1/screen"

# Get screen info
screen_info_url = f"{base_url}/application/{app_name}"
rsp = requests.get(screen_info_url, headers=headers)
if not rsp.ok:
    print("Failed to get screen info for application... exiting!!!")
    exit(1)

# Get our screen id from the response
first_screen = rsp.json()[0]
screen_id = first_screen['_id']
```

### Screen Update

The screen update API is used to modify the contents currently being shown on the
application screen. Note that each application has a unique screen and hence a unique
screen ID which was explained earlier.

Once the screen ID is collected the SDK/application screen can be updated as follows

```
import os, requests

# Create the base header
headers = {
    "Authorization": f"Basic {os.environ['AUTH_TOKEN']}"
}

# Get basic info from environment
host = os.getenv('SCREEN_API_HOST')
app_name = os.getenv('APPLICATION_NAME')
base_url = f"{host}/api/v1/screen"

# Get screen info
screen_info_url = f"{base_url}/application/{app_name}"
rsp = requests.get(screen_info_url, headers=headers)
if not rsp.ok:
    print("Failed to get screen info for application... exiting!!!")
    exit(1)

# Get our screen id from the response
first_screen = rsp.json()[0]
screen_id = first_screen['_id']

# Build up the request json
new_msg = "Hello!"
payload = {
    "value": f"{new_msg}",
    "animationType": "scroll",
    "type": "text",
    "brightness": 200,
    "animationInterval": 100
}

# Try to update
screen_update_url = f"{base_url}/{screen_id}"
rsp = requests.patch(screen_update_url, headers=headers, json=payload)
if not rsp.ok:
    print("Failed updating screen for application")
    exit(1)
```

In this case, the SDK/application screen will start showing a scrolling 'Hello'
on the screen/card assigned to it.
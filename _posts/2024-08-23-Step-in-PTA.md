---
title: "Steph's in the PTA, which means I'm in the PTA"
layout: post
date: 2024-08-23
---

When my wife joined the "birthday box" committee at our local PTA, I didn't really think I would be involved too (or maybe that was wishful thinking?). I watched her and other committee members copy-paste data from one website into another multiple times a day, and of course I saw an opportunity to put my tech skills to some use. 

## The Birthday Box Tradition

For those unfamiliar, a birthday box is a simple yet thoughtful way our school celebrates students' birthdays. Parents can order pre-approved snacks or goodies for their child's birthday month through a platform called Givebacks. The PTA committee then assembles these boxes, and ensures these treats are delivered safely and on time.

However, the process of managing these orders was far from efficient. Committee members were manually copying orders from Givebacks into a Google Sheet, which was time-consuming and prone to errors. That’s when I decided to step in.

## Understanding the Workflow

The first step was to understand the existing workflow:

1. Order Submission: Parents place orders through Givebacks.
2. Manual Data Entry: PTA members copy the order details into a Google Sheet.
3. Order Fulfillment: After the order is fulfilled, it is marked as such in the Google Sheet and on Givebacks.

This of course made it an ideal candidate for automation, so I decided to use Zapier to streamline the process.

## Building the Automation with Zapier and Python

I began by setting up a Zapier workflow to automate the task. Zapier has the ability to run Python directly in a "Zaps", so that made things great since there is no existing method here.

- OAuth Authentication: The first challenge was logging into Givebacks. Thankfully, Zapier's Python integration allowed me to handle OAuth authentication seamlessly, although it is far from a secure login, as the Email / Password does require hardcoding credentials.

```
import requests
import json

# Define the URL and headers
url = 'https://api.givebacks.com/services/core/users/sign_in'
headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:129.0) Gecko/20100101 Firefox/129.0',
    'Accept': 'application/json, text/plain, */*',
    'Accept-Language': 'en-US,en;q=0.5',
    'Accept-Encoding': 'gzip, deflate, br, zstd',
    'Content-Type': 'application/json',
    'Origin': 'https://somewhere.givebacks.com',
    'Connection': 'keep-alive',
    'Referer': 'https://somewhere.givebacks.com/',
    'Sec-Fetch-Dest': 'empty',
    'Sec-Fetch-Mode': 'cors',
    'Sec-Fetch-Site': 'same-site',
    'DNT': '1',
    'Sec-GPC': '1',
    'Priority': 'u=0',
    'TE': 'trailers'
}

# Define the payload
payload = {
    'user': {
        'email': 'someEmail@address.com',
        'password': 'A Super Secure Password'
    }
}

# Make the POST request
response = requests.post(url, headers=headers, data=json.dumps(payload))

output = {}
output['token'] = response.json()['user']['session']['token']
output['secret'] = response.json()['user']['session']['secret']
```
This `output` dict is how data is exchange between steps in the Zap pipeline. 

- Navigating Givebacks Orders: Once logged in, my Python code iterated through the orders on Givebacks. I had to reverse engineer the REST calls that the Single Page Application (SPA) made, as the public Givebacks API wasn’t in sync with the actual data being displayed. This involved a bit of sleuthing—inspecting network traffic and deciphering API endpoints that weren't officially documented.

```
import requests

# Define the URL with query parameters
url = 'https://api.givebacks.com/services/store/carts'
params = {
    'cause_id': 'somecauseGUID',
    'offset': '0',
    'limit': '50',
    'search[status][value]': 'purchased',
    'search[cart_items.fulfilled][value]': 'false',
    'join': 'AND'
}

# Define the headers
headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:129.0) Gecko/20100101 Firefox/129.0',
    'Accept': 'application/json, text/plain, */*',
    'Accept-Language': 'en-US,en;q=0.5',
    'Accept-Encoding': 'gzip, deflate, br, zstd',
    'Authentication-Session-Secret': input['secret'],
    'Authentication-Session-Token': input['token'],
    'Origin': 'https://somewhere.givebacks.com',
    'Connection': 'keep-alive',
    'Referer': 'https://somewhere.givebacks.com/',
    'Sec-Fetch-Dest': 'empty',
    'Sec-Fetch-Mode': 'cors',
    'Sec-Fetch-Site': 'same-site',
    'DNT': '1',
    'Sec-GPC': '1',
    'If-None-Match': 'W/"asdfasdf23r2f23f23"',
    'Priority': 'u=0',
    'TE': 'trailers'
}

# Make the GET request
response = requests.get(url, headers=headers, params=params)

# Print the response status code and content
print(response.status_code)
print(response.json())
```
Note the `output` dict from the prior step is mapped directly to the `input` dict on input. Pretty intuitive. 

- Identifying Birthday Box Orders: The next step was to identify orders specifically related to the birthday box. I added logic to filter these orders, ensuring only relevant data was passed along. Using their foreach methods you can iterate though these orders, then the items on those orders.
    - For each instance of a non-captured Birthday Box order, **Updating Google Sheets:** Using Zapier’s built-in Google Sheets connection, I automated the insertion of new rows for each birthday box order. This saved the committee from having to manually enter data multiple times a day.
- Marking Orders as Fulfilled: Finally, after adding the order to the Google Sheet, my script made a call to the Givebacks API (again, through reverse-engineered endpoints) to mark the item as fulfilled, ensuring there was no duplication of effort.

## The Impact

This automation set out to reduce administrative and repetitive tasks, allowing the PTA members to focus on what really matters. Eliminating this manual data entry will also remove chances of errors, and ensuring orders are fulfilled correctly. This application is limited to the Birthday Box team first, but that's just where I've started; just wait until I find more oppurtunities for automation ;)

## Lessons Learned

- Reverse Engineering REST Calls: Not all APIs are created equal, and sometimes the official documentation doesn’t provide everything you need. Reverse engineering the SPA’s REST calls, I was able to access the necessary endpoints.
- Zapier being able to execute custom Python code opened  up a world of possibilities for automations. It was my first time using it here, but I will be sure to keep it in mind in the future. 
- Community Contribution: Even though this all started to help my wife, it’s been fun to contribute to the community.

## Looking Forward

This project was a small yet satisfying venture into the world of PTA tech (or lack thereof). It’s more a reminder for me that everyone doesn't know the same things I do - and vice-versa; so if I can use my skills to help out others, I'd happily do so. 
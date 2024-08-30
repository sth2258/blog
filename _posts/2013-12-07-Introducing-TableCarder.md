---
title: "Crafting the Ultimate Wedding Seating Experience with TableCarder"
layout: post
date: 2013-12-07
---

Our wedding needed to be epic, but with a mix of traditional. We didn't want traditional escord cards, but rather someting innovative. Our solution? TableCarder. A custom-built, touch-screen interface that guided our guests to their tables with a bit of flair. Here's the story of how I brought it to life. 

## The Vision

We wanted an interactive experience where guests could approach a sleek HP touchscreen monitor, tap on their name, and the be guided directly to their table. This had to be seamless process (caugh grandparents caugh), so we set out to develop TableCarder with this in mind.

## Backend: C# for Data Handling and Integration

The backend of TableCarder was built using C#, handling everything from fetching guest information to integrating with external services like the Facebook API and AWS SNS. Here's a simple example of how we used C# to retrieve guest information and send an SMS notification with AWS SNS:

### Retrieving Guest Information
```
public class Guest
{
    public string Name { get; set; }
    public string TableNumber { get; set; }
    public string PhoneNumber { get; set; }
}

public class GuestService
{
    private List<Guest> guests;

    public GuestService()
    {
        // Populate guest list
        guests = new List<Guest>
        {
            new Guest { Name = "John Doe", TableNumber = "5", PhoneNumber = "1234567890" },
            new Guest { Name = "Jane Smith", TableNumber = "10", PhoneNumber = "0987654321" }
            // Add more guests as needed
        };
    }

    public Guest GetGuestByName(string name)
    {
        return guests.FirstOrDefault(g => g.Name.Equals(name, StringComparison.OrdinalIgnoreCase));
    }
}
```

### Sending SMS with AWS SNS
```
using Amazon.SimpleNotificationService;
using Amazon.SimpleNotificationService.Model;

public class NotificationService
{
    private readonly AmazonSimpleNotificationServiceClient _snsClient;

    public NotificationService()
    {
        _snsClient = new AmazonSimpleNotificationServiceClient();
    }

    public async Task SendTableNumberSms(string phoneNumber, string tableNumber)
    {
        var message = $"Hi! You are seated at table {tableNumber}. See you there!";
        var request = new PublishRequest
        {
            Message = message,
            PhoneNumber = phoneNumber
        };
        
        await _snsClient.PublishAsync(request);
    }
}
```

## Frontend: jQuery for Interactivity
The frontend was built using HTML, CSS, and JavaScript, with jQuery adding interactivity. Here’s how we handled the display of guest information and table flashing:

### Displaying Guest Info and Flashing the Table
```
<div id="guestInfo">
    <input type="text" id="guestNameInput" placeholder="Enter your name...">
    <button id="findTableBtn">Find My Table</button>
</div>

<div id="tableDisplay">
    <!-- SVG of the seating plan -->
</div>
```

```
$(document).ready(function() {
    $('#findTableBtn').click(function() {
        var guestName = $('#guestNameInput').val();
        
        $.ajax({
            url: '/api/guest',  // Your C# API endpoint
            method: 'GET',
            data: { name: guestName },
            success: function(data) {
                if (data) {
                    alert('Your table is ' + data.tableNumber);
                    flashTable(data.tableNumber);
                } else {
                    alert('Guest not found!');
                }
            }
        });
    });

    function flashTable(tableNumber) {
        var tableElement = $('#table-' + tableNumber); // Assuming each table has an ID like 'table-1', 'table-2', etc.
        tableElement.fadeOut(500).fadeIn(500).fadeOut(500).fadeIn(500);
    }
});
```

## Integrating Guest Photos with the Facebook API
One of the standout features of TableCarder was the ability to display each guest's profile picture as they searched for their name. To achieve this, we tapped into the Facebook API to download profile pictures:
### Fetching Profile Pictures
```
using System.Net.Http;
using Newtonsoft.Json.Linq;

public async Task<string> GetFacebookProfilePictureUrl(string userId, string accessToken)
{
    var url = $"https://graph.facebook.com/{userId}/picture?type=large&access_token={accessToken}";
    
    using (var client = new HttpClient())
    {
        var response = await client.GetAsync(url);
        response.EnsureSuccessStatusCode();
        
        return response.RequestMessage.RequestUri.ToString();
    }
}

```
### Displaying Profile Pictures with jQuery
```
function displayProfilePicture(userId, accessToken) {
    $.ajax({
        url: '/api/facebook-profile-picture',  // Your C# API endpoint
        method: 'GET',
        data: { userId: userId, accessToken: accessToken },
        success: function(url) {
            $('#profilePicture').attr('src', url);
        }
    });
}

// Example usage after guest is found
displayProfilePicture(guestFacebookId, facebookAccessToken);
```

## Operating the Setup: A Family Affair
On the big day, the TableCarder setup was operated by my wife’s cousins, Jake and John. These two tech-savvy cousins ensured everything ran smoothly, from the touchscreen interactions to the SMS notifications. Their involvement added a personal touch to the experience, making it not just a tech demo but a family-driven success.

## The Result

TableCarder was a big hit. It was the mix of my tech saviness and Steph's aesthetics, and lots of guests seemed to have never seen such a solution before. They loved having something interactive, and the combination of visual (flashing tables) and the text-to-speech made finding their tables a breeze (though we did have some human backups). This blend of technology with a user-friendly interface added a unique element to our wedding, making it a memorable experience for everyone involved.

At the end, TableCarder was our way to enhance the standard wedding experience and we were vert happy with how it all came together.
---
title: 
author: norie
description: Landing page overview for Winter Starfall, a PlayFab demo game.
ms.author: natashaorie
ms.date: 09/26/2024
ms.topic: article
ms.service: azure-playfab
keywords: playfab, gaming, sample, demo
ms.localizationpriority: medium
---

# Demo game: Winter Starfall

Winter Starfall is a game developed by PlayFab to showcase our different features and provide a way to explore how they are implemented in a live game.

The game is available to play [here](link). If you already have a PlayFab developer account you can also access it through the developer portal on the Studios & Titles overview.

![studios overview demo games]

The game is built as a web app and uses APIs from the PlayFab SDK [link] along with PlayFab’s Azure Functions integration to implement custom functionality. It currently makes use of the following features:

- Economy V2 (catalog, currency, bundles, stores)
- Title data, player data, and other core PlayFab functions
- Title news for communicating with players
- CloudScript with Azure Functions

## Additional demo features

Winter Starfall comes with some features that make it easier to see what is going on ‘under the hood’ with PlayFab. The game is available for anyone to play, but to get the full use of these features you’ll need to sign up for a [free developer account](link). 

Next to the profile icon in the upper right there is a PlayFab icon that will open the PlayFab activity sidebar when clicked.

![winter-starfall-profile](media/winter-starfall-activity.jpeg)

1. The activity sidebar shows a list of API calls that updates as you play the game. 
1. You can select one of the API calls in the list to view the full API request and response.
1. Selecting **View title F8941** will open the title in Game Manager, the developer portal, where you can see how the different features are configured. This is where you'll be prompted to sign in with a PlayFab developer account.
1. **Clear** will remove all the current notifications from the activity bar. 

You’ll also notice throughout the game these various callouts. Each of these indicates where a specific PlayFab feature is used to power a certain aspect of the game and hovering over will show more information and take you to the related pages in Game Manager.

[screenshot - callouts]

> [!NOTE] 
> You can play the demo game at any time, but to access Game Manager you’ll need to sign up for a [free developer account](link]).

## Included PlayFab features

The following section breaks down in more detail the different PlayFab features that are used in the game.

### Authentication

The first thing that happens in any PlayFab game is logging in a player, which returns an authentication token that is required for all subsequent API calls. Winter Starfall supports multiple forms of recoverable login with email, Google, and Facebook. The [source code and scenarios tutorial](source-code-and-best-practices.md) gives an in depth walkthrough of the login flow and best practices for player authentication.

Learn more about PlayFab Authentication [here](../features/authentication/login/index.md).

### Economy

As a fantasy RPG style game, Winter Starfall includes an economy system for the player to visit stores, purchase items, earn currency, and more. PlayFab’s newest economy service handles everything related to inventory and commerce in the game. The [source code and scenarios tutorial](source-code-and-best-practices.md) gives an in depth walkthrough of the purchase flow that occurs in the game.

Learn more about Economy V2 [here](../features/economy-v2/overview.md).

### Player data 

Players in PlayFab have associated data that is stored in the service by different features. Player data as a feature allows you to store player-related information in key/value pairs or objects and files that can be shared across multiple games and devices.

In Winter Starfall, Player data is used to store the game state and information like which characters have joined the player's party. When the player moves through the story, their position is recorded in their player profile through a call to the UpdateUserData API[xref api docs]. This data is then accessed with GetUserData when they log in, to load the player into the right point in the story with all their past progress.

![winter-starfall-player-data](media/winterstarfall-player-data.jpeg)

For example, these are the API request and response body from the call to `GetUserData` during the login flow. 

API Request - GetUserData
```json
{
  "Keys": [
    "stats",
    "location",
    "party",
    "notifications",
    "cinematicProgression",
    "enemyGroupProgression"
  ]
}
```
API Response - GetUserData
```json
{
  "code": 200,
  "status": "OK",
  "data": {
    "Data": {
      "location": {
        "Value": "{\"id\":\"intro\",\"map\":[\"intro\",\"intro-village\"]}",
        "LastUpdated": "2024-09-20T18:16:13.68Z",
        "Permission": "Private"
      },
      "party": {
        "Value": "{\"characters\":[{\"id\":1,\"armor\":\"dfae3ef9-92a0-493b-9bb8-f88a09718d26\",\"weapon\":\"32e1f684-43b0-4fe6-982e-b0829ff0d6f3\",\"spells\":null}],\"guests\":[]}",
        "LastUpdated": "2024-09-20T18:16:13.68Z",
        "Permission": "Private"
      }
    },
    "DataVersion": 20
  },
  "CallBackTimeMS": 97
}
```

Learn more about the Player data feature [here](../features/playerdata/index.md). 

### Title data

Title data is similar to Player data, just being data pertaining to a game title instead of a specific player. Winter Starfall uses title data in conjuction with the Economy system to calculate the price when selling an item by storing the value `multipliers` with a value of `sell, 0.5`.

![winterstarfall-title-data](winterstarfall-title-data.jpeg)

Learn more about Title data [here](../features/titledata/index.md).

### CloudScript with Azure Functions

CloudScript is another key part of making this game work, because its a very flexible feature that allows you to enables serverless compute on demand 

request execution of any custom server-side functionality you can implement, and it can be used in conjunction with virtually anything.

Vanguard Outrider runs CloudScript functions in C#, but y can use any language supported by Azure Functions [
Supported languages in Azure Functions | Microsoft Learn ] 

CloudScript functions are used to 
For example, in this case the CloudScript function ‘ProgressCheckpoint’ is executed after the player wins a battle, updating the character’s stats, location, items gained/used, etc.

[use code snippet to demonstrate how this azure function updates player data] vanguard-outrider-2/azure-functions/ProgressCheckpoint.cs at main · PlayFab/vanguard-outrider-2 (github.com)
ProgressCheckpoint makes calls to the economy APIs

In the activity stack, 
[screenshot of API call]
[copy of API request and response]

Learn more about CloudScript with Azure Functions [here](../features/automation/cloudscript-af/index.md).

### Title news

Title news is used to communicate with all players scoped to a title. Winter Starfall implements it as a notification system to display gameplay tips and notices. Additional capabilities such as using player segments to send messages to specific subsets of players, and Experimentation to A/B test with different messaging

![winter-starfall-title-news](media/winterstarfall-title-news.jpeg)

In addition to title news, PlayFab offers other communication features like templating for email and push notifications. Learn more about title communication methods [here](../features/engagement/overview.md).

## Demo limitations

Because Winter Starfall is powered by real player data, some features are limited in scope in the Game Manager view. This section will give an overview of what the limited features would look like in Game Manager. To explore these pages in more detail, you can [download the source code](link) and run a local instance of the game, or create your own new title [link to get started/logging in a player tutorial]. 

### Players

[screenshot - Sample of player overview (inventory, information, etc)]

### Data & Analytics

[screenshot of sample query and reports]

Also see the Data & Analytics documentation for more information on these features.

## Next steps

After trying the demo, we recommend starting with these topics to learn more about how PlayFab works:
- Learn more about [Game Manager](link)
- Learn about Entity model
- Other demos – recipes and samples
- Learn about SDK

## See also

- [Play Winter Starfall](here)
- [Tutorial: Download source code and example flows](source-code-and-best-practices.md)
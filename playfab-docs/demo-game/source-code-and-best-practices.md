---
title: 
author: norie
description: Tutorial for downloading Winter Starfall source code and best practices for login and purchase scenarios.
ms.author: natashaorie
ms.date: 09/26/2024
ms.topic: article
ms.service: azure-playfab
keywords: playfab, gaming, sample, demo
ms.localizationpriority: medium
---

# Source code and scenarios in Winter Starfall

This tutorial contains instructions for downloading and running the source code for Winter Starfall, and a walkthrough of the code samples demonstrating best practices for login and purchase scenarios.

## Source code download

### Prerequisites

- Install Node JS
- Install Visual Studio Code

### Setup for local development

1. Download the source code from [GitHub](https://github.com/PlayFab/vanguard-outrider-2).
1. In VS Code, open the `/website` folder and opt to install all the extensions it recommends.
1. In the `/website` directory, run npm install to install all dependencies.
1. Use `npm run dev` to start the site. Click on the link it offers to view the site.

### Azure credentials

To update the AZURE_CREDENTIALS secrets, run this command in the Azure Portal shell:

`az ad sp create-for-rbac --name "VanguardOutrider2" --role contributor --scopes /subscriptions/76634aed-e403-4f92-b02d-9ebf155f2382/resourceGroups/vanguard-outrider-2-resources --json-auth`

## Login flow

The player authentication flow in Winter Starfall has two steps: the login call and post-login functions that are used get the player's information from the service and load them into the correct location in the story.

### Player authentication

For the login portion, the file [use-login.tsx](https://github.com/PlayFab/vanguard-outrider-2/blob/main/website/src/hooks/use-login.ts) defines a TypeScript wrapper function that handles the multiple authentication methods available to players. Lines 144 -169 defines the callback function for login in with an email address, and calls the post-login functions if login suceeds.

```typescript
	const onLogin = useCallback(() => {
		setIsLoading(true);
		setLoginMethodInProgress("email");

		dispatch(siteSlice.actions.loginSteps(loginEventCount));

		ClientLoginWithEmailAddress({ Email: data.email, Password: data.password })
			.then(result => {
				dispatch(siteSlice.actions.login(result));
				dispatch(siteSlice.actions.loginStepsAdvance());
				// Track logins from returning users
				trackEvent({ name: "Returning User", properties: {} });
			})
			.then(() => {
				return postLoginFunctions();
			})
			.then(() => {
				setIsLoading(false);
				navigate(routes.Explore());
			})
			.catch(problem => {
				dispatch(siteSlice.actions.loginStepsReset());
				onError(problem);
			})
			.finally(() => dispatch(siteSlice.actions.loginStepsReset()));
	}, [ClientLoginWithEmailAddress, data.email, data.password, dispatch, navigate, onError, postLoginFunctions]);
```

The game offers 3 recoverable methods for player authentication: email, Google, and Facebook, so that player accounts will never be lost. For more information, see [Login best practices](../features/authentication/login/login-basics-best-practices.md).

First part - login
Login > progress.tsx, runner.tsx, sign-up-register.tsx reference use-login
OnLogin, OnloginWithGoogle, OnLoginWithFacebook

### Post login: Getting player data

Once the security token has been retrieved, it is used to run the post login functions to get the saved game state.
Lines 340 - 378 defines **postLoginFunctions**, which calls various Economy and PlayFab Services APIs to load the correct story location, inventory items, currency, etc based on the player.

```typescript
return new Promise<void>((resolve, reject) => {
			ClientGetTitleData({ Keys: TITLE_DATA_KEYS_ALL })
				.then(result => {
					dispatch(siteSlice.actions.titleData(result));
					dispatch(siteSlice.actions.loginStepsAdvance());
					loadScripts();
				})
				.then(() =>
					EconomySearchItems({
						Count: SEARCH_ITEMS_MAX_COUNT,
						Filter: "type eq 'currency' or type eq 'catalogItem'",
					})
				)
				.then(result => {
					dispatch(siteSlice.actions.catalog(result.Items));
					dispatch(siteSlice.actions.loginStepsAdvance());
				})
				.then(() => EconomyGetInventoryItems({ Count: SEARCH_ITEMS_MAX_COUNT }))
				.then(result => {
					dispatch(siteSlice.actions.inventory(result.Items));
					dispatch(siteSlice.actions.loginStepsAdvance());
				})
				.then(() => ClientGetUserData({ Keys: USER_DATA_KEYS_PLAYER_ALL }))
				.then(result => {
					dispatch(siteSlice.actions.userDataPlayer(result));
					dispatch(siteSlice.actions.loginStepsAdvance());
				})
				.then(() => ClientGetUserReadOnlyData({ Keys: USER_DATA_KEYS_READONLY_ALL }))
				.then(result => {
					dispatch(siteSlice.actions.userDataReadOnly(result));
					dispatch(siteSlice.actions.loginStepsAdvance());
				})
				.then(() => {
					resolve();
				})
				.catch(problem => {
					dispatch(siteSlice.actions.loginStepsReset());
					reject(problem);
				});
```

- First `ClientGetTitleData`
- Then `EconomySearchItems` searches the catalog for currency and item types
- Which is used as input to `EconomyGetInventoryItems` to return the player's inventory items
- Then `ClientGetUserData` retrieves the value that stores the story location as part of a player's data.

## Purchase flow

At certain points in the game, the player has the option to purchase and sell inventory items from the store. The purchase flow is implemented in another wrapper function.

In the file [use-store.tx](https://github.com/PlayFab/vanguard-outrider-2/blob/main/website/src/hooks/use-store.ts)

### Selling items

The selling flow is an example of how to use Azure Functions driven CloudScript to extend the functionality of Economy system.

[SellItem.cs](https://github.com/PlayFab/vanguard-outrider-2/blob/main/azure-functions/SellItem.cs) defines the CloudScript function that executes when selling an item.

Line 49 - 61 is where PlayFab's `GetTitleData API` is used to return the store multiplier and calculate the sale price.

```csharp
 // Get your sell multiplier
                var titleData = await PlayFabFunctions.GetTitleDataAsync(player, new List<string> { TitleDataKeys.Multipliers }, log);
                var sellMultiplier = JsonConvert.DeserializeObject<Multipliers>(titleData[TitleDataKeys.Multipliers])?.sell ?? 1;
```
The CloudScript function then gets called in lines 161- 187 of [use-store.ts](https://github.com/PlayFab/vanguard-outrider-2/blob/main/website/src/hooks/use-store.ts).

```typescript
export function useEconomyStoreSell(): IEconomyStoreSellItemResults {
	const { isLoading, error, setError, CloudScriptExecuteFunction, EconomyGetInventoryItems } = usePlayFab();
	const dispatch = useDispatch();

	const onSell = useCallback(
		(itemId: string, amount: number) => {
			return new Promise<void>((resolve, reject) => {
				CloudScriptExecuteFunction({
					FunctionName: "SellItem",
					FunctionParameter: {
						ItemId: itemId,
						Amount: amount,
					},
				})
					.then(() => EconomyGetInventoryItems({ Count: SEARCH_ITEMS_MAX_COUNT }))
					.then(data => {
						dispatch(siteSlice.actions.inventory(data.Items));
						resolve();
					})
					.catch(issue => {
						setError(issue);
						reject(issue);
					});
			});
		},
		[CloudScriptExecuteFunction, EconomyGetInventoryItems, dispatch, setError]
	);
```

> [!NOTE]
> To run Winter Starfall locally you'll need an Azure account to support the CloudScript functions. You can sign up for a free Azure account [here](https://azure.microsoft.com/en-us/pricing/purchase-options/azure-account?msockid=1dec68fa155462cb2baa7ca6147963d7) and then follow the instructions above to reset the Azure credentials correctly.

Reach a checkpoint in the story where the player visits the store for the first time
trackEvent({ name: "Checkpoint reached" }, { checkpoint: request.FunctionParameter.checkpoint});

For more detailed information on Economy practices, see the Economy V2 documentation:

- Overview
- Stores
- Inventory

As a next step, Economy crafting game.
EconomySearchItems – search catalog for inventory items
Economy crafting game

## See also

- Player login documentation
- Economy V2 documentation
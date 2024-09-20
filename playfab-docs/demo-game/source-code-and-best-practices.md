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

# Sample code and scenarios in Winter Starfall

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

For the login portion, the file [use-login.tsx](https://github.com/PlayFab/vanguard-outrider-2/blob/main/website/src/hooks/use-login.ts) defines a wrapper function that handles the multiple authentication methods available to players. Lines 144 -169 defines the callback function for login in with an email address, and calls the post-login functions if login suceeds.

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

### Post login: Getting player data
Involves player auth and title data APIs to login player and load correct game state.
First part - login
Login > progress.tsx, runner.tsx, sign-up-register.tsx reference use-login
OnLogin, OnloginWithGoogle, OnLoginWithFacebook

Returns security key that is used for all other API calls

Second part - Post login

Lines 309 - 344 defines **postLoginFunctions**, which calls various Economy and PlayFab Services APIs to load the correct story location, inventory items, currency, etc based on the player. 

loads scripts and uses player data to load correct location and story

```typescript
export function usePlayFabPostLoginData() {
	const {
		ClientGetUserData,
		ClientGetUserReadOnlyData,
		ClientGetTitleData,
		EconomySearchItems,
		EconomyGetInventoryItems,
	} = usePlayFab();
	const dispatch = useDispatch();

	const postLoginFunctions = useCallback(() => {
		const scripts = import.meta.glob("../data/*.json", { query: "raw" });

		const loadScripts = () => {
			for (const path in scripts) {
				scripts[path]().then(mod => {
					if (path.indexOf("locations.json") !== -1) {
						dispatch(siteSlice.actions.locations(JSON.parse((mod as any).default) as ILocalDataLocations));
					}
					if (path.indexOf("enemies.json") !== -1) {
						dispatch(siteSlice.actions.enemies(JSON.parse((mod as any).default) as ILocalDataEnemies));
					}
					if (path.indexOf("cinematics.json") !== -1) {
						dispatch(
							siteSlice.actions.cinematicData(JSON.parse((mod as any).default) as ILocalDataCinematics)
						);
					}
				});
			}
		};
```

- `ClientGetUserData` retrieves the value that stores the story location as part of a player's data.
- `ClientGetTitleData`
- `EconomySearchItems`
-  `EconomyGetInventoryItems`

Lines 340 - 378 

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

## Purchase flow

At certain points in the game, the player has the option to purchase and sell inventory items from the store.

In the file [use-store.tx](https://github.com/PlayFab/vanguard-outrider-2/blob/main/website/src/hooks/use-store.ts)


Purchase flow
This demonstrates best practices – how/why?
Typescript wrapper function
In file use-store.ts 
vanguard-outrider-2/website/src/hooks/use-store.ts at main · PlayFab/vanguard-outrider-2 (github.com)
Reach a checkpoint in the story where the player visits the store for the first time
trackEvent({ name: "Checkpoint reached" }, { checkpoint: request.FunctionParameter.checkpoint});

For more detailed information on Economy practices, see the Economy V2 documentation:

- Overview
- Stores
- Inventory

As a next step, Economy crafting game.

## See also

- Player login / authenticaiton 

In the file use-login.ts at lines 340 – 378
vanguard-outrider-2/website/src/hooks/use-login.ts at main · PlayFab/vanguard-outrider-2 (github.com)
-	Get title data (API call)
-	EconomySearchItems – search catalog for inventory items
- Economy crafting game


Is there anything that won’t run without additional setup? ie Azure subscription 

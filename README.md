# tdialogs

[![sampctl](https://img.shields.io/badge/sampctl-tdialogs-2f2f2f.svg?style=for-the-badge)](https://github.com/TommyB123/tdialogs)

tdialogs is a library that provides various functions to handle dialog responses in open.mp servers asynchronously through the power of PawnPlus' task system.

# Dependencies
This library requires the following dependencies.
* [PawnPlus](https://github.com/IS4Code/PawnPlus)
* [sscanf](https://github.com/Y-Less/sscanf)

## Installation

Simply install to your project:

```bash
sampctl install TommyB123/tdialogs
```

Include in your code and begin using the library:

```pawn
#include <tdialogs>
```

# Introduction
As stated above, tdialogs is a PawnPlus-powered library that can be used to handle dialog responses asynchronously in a single PAWN function. Here's a quick example.
```pawn
MyCoolFunction()
{
	yield 1;

	new response[DIALOG_RESPONSE];
	await_arr(response) ShowAsyncDialog(playerid, DIALOG_STYLE_LIST, "Cool Dialog", "Entry 1\nEntry 2\nEntry 3", "Neat!", "Go back");
	if(response[DIALOG_RESPONSE_RESPONSE])
	{
		SendClientMessage(playerid, -1, "you clicked row %i and got the message %s",
			response[DIALOG_RESPONSE_LISTITEM], response[DIALOG_RESPONSE_INPUTTEXT]); //values could be 1 and Entry 2 respectively
	}
	else
	{
		SendClientMessage(playerid, -1, "you closed the dialog");
	}
}
```

If you're familiar with all of the dialog handler libraries available, you might be wondering sets this library apart from them. The simple answer is I've included numerous functions to quickly fetch specific/single values from dialog responses. Do you simply need to show quick yes/no confirmation dialog? No problem. Use `ShowAsyncConfirmationDialog`. What about buying a user-submitted quantity from an in-game shop? `ShowAsyncNumberInputDialog` has you covered. Examples usages can be found [here](https://github.com/TommyB123/tdialogs?tab=readme-ov-file#examples).

As of version 1.1.0, automatically paginated dialogs are also included and can be created with `AddPaginatedDialogRow` and `ShowAsyncPaginatedDialog`. Full function arguments and example below.

## Dialog Data
tdialogs also has a small but handy feature for tracking IDs or other data throughout the entire dialog flow. Each player gets a dynamic [PawnPlus list](https://github.com/IS4Code/PawnPlus/wiki/Lists) allocated for them on login. This list can be used to store things like house indexes, vehicle IDs or other data matched with rows of text inside your dialogs. This is done for the purpose of retaining identifiers and avoiding needless loop lookups or other somewhat costly methods to fetch data you've already found.

Alongside this list, there's also a special dialog function made with its combined use in mind. When you've formatted a dialog and pushed accompanying entity IDs, such as a house, to the `DialogData` list, `ShowAsyncEntityIndexDialog` can be used to fetch the house ID a player clicked on with a single line of code. You can see it in action in the examples section below.

It's important to note that the list **MUST** be cleaned out by using `list_clear` before you populate it with IDs when showing a new dialog. Otherwise, data from previously shown dialogs will bleed into subsequent dialog showings.

# Constants

| Constant | Default Value | Description  |
| ------------- | ------------- | ------------- |
| `TDIALOG_DIALOG_ID_BEGIN`  | `1234`  | The library reserves 9 dialog IDs for each dialog type (not to be confused with dialog styles). `TDIALOG_DIALOG_ID_BEGIN` is the first ID that will be used, followed by the 8 subsequent numbers after. Note any existing dialog IDs you have and adjust accordingly. |
| `PAGINATED_NEXT_TEXT`  | `"--> Next Page"`  | The string that will show when a paginated dialog prompts you to view the next page.  |
| `PAGINATED_PREVIOUS_TEXT`  | `"<-- Previous Page"`  | The string that wil show when you can go back to a previous page in a paginated dialog.  |

# Prerequisites
It's advised that you add `#define PP_SYNTAX_AWAIT` before including PawnPlus. This allows you to use the proper await methods this library was written in mind with.

It's also advised to add `#define PP_SYNTAX_YIELD` as well. This is a quick alias for `task_yield`, which is used to yield temporary values to functions when a function waits for a task. For instance, if you don't `yield 1;` inside of a command, it's likely you will receive an erroneous unknown command error in chat.

# Functions
All functions below have PawnPlus string variants. `ShowAsyncDialog` is `ShowAsyncDialog_s` and so on.

```pawn
ShowPlayerDialog_s(playerid, dialogid, DIALOG_STYLE:style, ConstAmxString:title, ConstAmxString:body, const button1[], const button2[])
```
Simple PawnPlus string wrapper for `ShowPlayerDialog`

```pawn
ShowAsyncDialog(playerid, DIALOG_STYLE:style, const title[], const body[], const button1[], const button2[] = "")
```
Shows an asynchronous dialog with the full dialog response array.

Used with `await_arr(response) ShowAsyncDialog(...)`

```pawn
ShowAsyncNumberInputDialog(playerid, const title[], const body[], const button1[], const button2[])
```
Shows an asynchronous dialog specifically to parse numeric input.

If an integer is not received, the dialog will simply be shown again.

If the player closes the dialog/presses ESC, `cellmin` is returned.

Used with `new number = await ShowAsyncNumberInputDialog(...)`

```pawn
ShowAsyncFloatInputDialog(playerid, const title[], const body[], const button1[], const button2[])
```
Shows an asynchronous dialog specifically to parse numeric input.

If a float is not received, the dialog will simply be shown again.

If the player closes the dialog/presses ESC, `FLOAT_NAN` is returned.

Used with `new Float:number = Float:await ShowAsyncFloatInputDialog(...)`

```pawn
ShowAsyncStringInputDialog(playerid, const title[], const body[], const button1[], const button2[])
```
Shows an asynchronous dialog specifically to parse string input.

If an empty string is received, the dialog will simply be shown again.

If the player closes the dialog/presses ESC, a null string is returned. Use `isnull` to check.

Used with `await_str(string) ShowAsyncStringInputDialog(...)`

```pawn
ShowAsyncPasswordDialog(playerid, const title[], const body[], const button1[], const button2[])
```
Identical to `ShowAsyncStringInputDialog` but with the password dialog style.

```pawn
ShowAsyncListitemTextDialog(playerid, DIALOG_STYLE:style, const title[], const body[], const button1[], const button2[])
```
Shows an asynchronous dialog that only returns the text passed through a listitem.

If the player closes the dialog/presses ESC, a null string is returned. Use `isnull` to check.

Used with `await_str(string) ShowAsyncListitemTextDialog(...)`

```pawn
ShowAsyncListitemIndexDialog(playerid, DIALOG_STYLE:style, const title[], const body[], const button1[], const button2[])
```
Shows an asynchronous dialog that only returns the index of the listitem that was clicked.

If the player closes the dialog/presses ESC, `-1` is returned.

Used with `new index = await ShowAsyncListitemIndexDialog(...)`

```pawn
ShowAsyncConfirmationDialog(playerid, const title[], const body[], const button1[], const button2[] = "")
```
Shows an asynchronous dialog that only returns the response status as a boolean.

Used with `new bool:confirm = bool:await ShowAsyncConfirmationDialog(...)`

```pawn
ShowAsyncEntityIndexDialog(playerid, DIALOG_STYLE:style, const title[], const body[], const button1[], const button2[])
```
Shows an asynchronous dialog that returns the value of the player's `DialogData` [list](https://github.com/IS4Code/PawnPlus/wiki/Lists) found at `listitem`. Read more [here](https://github.com/TommyB123/tdialogs/edit/main/README.md#dialog-data) or in the example below.

If the player closes the dialog/presses ESC, `-1` is returned.

Used with `new id = await ShowAsyncEntityIndexDialog(...)`

```pawn
ShowAsyncPaginatedDialog(playerid, DIALOG_STYLE:style, rows_per_page, const title[], List:rows, const button1[], const button2[], const tablist_header_text[] = "")
```
Shows an asynchronous dialog that paginates supplied text based on the supplied rows per page. Returns a full dialog response.

This function is only compatible with `DIALOG_STYLE_TABLIST_HEADERS`, `DIALOG_STYLE_TABLIST` and `DIALOG_STYLE_LIST`.

The `tablist_header_text` argument is only to be used with the `DIALOG_STYLE_TABLIST_HEADERS` and, as the name implies, is the header text for the dialog.

Used with `await_arr(response) ShowAsyncPaginatedDialog`

```pawn
AddPaginatedDialogRow(List:list, const text[], extraid = 0)
```
Adds a row to a paginated dialog. extraid argument can be used to pass additional IDs relevant to the line of text you're appending.

Required when building a paginated dialog.

# Examples
Here's some slightly more in-depth examples.

Quickly confirming whether or not a user wants to sell their home.
```pawn
CMD:sellmyhouse(playerid)
{
	if(PlayerOwnsHouse(playerid))
	{
		task_yield(1);
		new bool:confirm = bool:await ShowAsyncConfirmationDialog(playerid, "Sell Your House?", "Are you sure you wish to sell your house?", "Yes", "No");
		if(confirm)
		{
			SellPlayerHouse(playerid);
		}
	}
}
```

Enter how much fish bait you wish to purchase from a store.
```pawn
new baitamount = await ShowAsyncNumberInputDialog(playerid, "Bait Quantity", "Enter how much fishing bait you'd like to purchase below", "Buy", "Cancel");
if(baitamount == cellmin) return false; //cellmin is returned when a player cancels a dialog
if(baitamount <= 0 || baitamount > 10000)
{
	SendClientMessage(playerid, -1, "Invalid bait amount.");
	return false;
}

GivePlayerFishBait(playerd, baitamount);
return true;
```

Resolve a house ID from a dialog by using the `DialogData` list.
```pawn
ShowHouseTeleportDialog(playerid)
{
	new string[256], substring[64];
	list_clear(DialogData[playerid]); //empty the persistent DialogData list so it matches the amount of rows in the dialog we're about to show
	for(new i = 0; i < MAX_HOUSES; ++i)
	{
		if(HouseData[i][HouseOwner] == playerid)
		{
			//append the house name to the string and add the house's index to the DialogData list.
			//what this does is keep the house name row consistent with the arbitrary house ID.
			//there may be 500 houses, but only a few (or even one) owned by the player. storing the 
			//home's index this way lets us quickly fetch the proper ID without doing additional needless lookups

			format(substring, sizeof(substring), "%s\n", HouseData[i][HouseAddress]);
			strcat(string, substring);
			list_add(DialogData[playerid], i);
		}
	}

	new houseid = await ShowAsyncEntityIndexDialog(playerid, DIALOG_STYLE_LIST, "Teleport to your homes", string, "Teleport!", "Never mind");
	if(houseid != -1) //returns -1 if the player cancels the dialog
	{
		SetPlayerPos(playerid, HouseData[houseid][HouseX], HouseData[houseid][HouseY], HouseData[houseid][HouseZ]);
	}
}
```

Showing a paginated dialog with 10 rows per page.
```pawn
ShowOwnedVehiclesDialog(playerid)
{
	new string[128];
	list_clear(DialogData[playerid]);
	for(new i = 1; i <= MAX_VEHICLES; ++i)
	{
		if(PlayerOwnsVehicle(playerid, i))
		{
			format(string, sizeof(string), "%s\t%s", ReturnVehicleName(i), ReturnVehicleLocation(i));
			AddPaginatedDialogRow(DialogData[playerid], string, i);
		}
	}

	new response[DIALOG_RESPONSE];
	await_arr(response) ShowAsyncPaginatedDialog(playerid, DIALOG_STYLE_TABLIST_HEADERS, 10, "Your Vehicles", DialogData[playerid], "Teleport", "Cancel", "Vehicle\tLocation");
	if(!response[DIALOG_RESPONSE_RESPONSE]) return false;

	new vehicleid = response[DIALOG_RESPONSE_EXTRAID], Float:x, Float:y, Float:z;
	GetVehiclePos(vehicleid, x, y, z);
	SetPlayerPos(playerid, x, y, z);
	SendClientMessage(playerid, -1, "You have teleported to your %s", ReturnVehicleName(vehicleid));
}
```

# Credits
me

Special thanks to [Graber](https://github.com/AGraber). I've used his original [async dialogs](https://github.com/AGraber/samp-async-dialogs) library for years before being inspired to make my own. If you don't need the extra features my library provides, consider using his instead.

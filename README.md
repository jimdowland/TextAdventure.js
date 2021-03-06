# TextAdventure.js

TextAdventure.js is a text adventure engine that runs on [Node.js](http://nodejs.org/) and makes use of [Express](http://expressjs.com/). The project has four main components: a simple server, a retro command line inspired web interface known as the Terminal, the text adventure engine itself colloquially called the Console and finally the cartridges (games) which are made up of two JavaScript objects. Each of these components is further explained its respective section below.

## Server

The server is an extremely simple Node.js file. It fires up an instance of an Express server on port 3000 to which it serves the Terminal. All AJAX requests made to it are quickly passed to the Console to be executed. Responses are then dispatched back to the Terminal. To get TextAdventure up and running on port 3000 simply run the following command from the project's folder:

```
node server.js
```

## Terminal

The Terminal consists of a single HTML file, a single CSS file and two JavaScript files (one of which is jQuery in case you need to run TextAdventure.js without an Internet connection). The Terminal's main job is to send the user's input to the Server and then display the Server's response to the user. The terminal has a few other creature comforts built in. First, it sends a "dummy" AJAX call to the server when the page loads to get the list of cartridges without requiring the user to input anything. Secondly, it keeps a record of the user's input that can be navigated via the UP and DOWN arrow keys. The terminal's CSS can easily be tweaked to your liking.

## Console

The Console takes the user input passed to it by the Server, acts on it and then returns a string that is ultimately displayed to the user. Upon receiving the user input the first thing the Console does is run it through the Parser to convert it to a more easy to use JavaScript object; consisting of an action, a subject and an object. The Console then runs the appropriate functionality based on the following hierarchy:

1. Does the action have a corresponding function in the cartridge
2. Does the action have a corresponding function in the console
3. Does the subject have a corresponding interaction

Following the executions of the appropriate functionality, the updateLocation() function of the player's current location is run. If that function returns a string that string will be passed to the user in lieu of the string generated by the appropriate functionality. Finally, the console checks to see if the cartridge `gameOver` attribute has been set to true and if so will return the cartridge's `outroText`.

The console implements the following player commands: `die`, `drop`, `go`, `inventory`, `load`, `look`, `take` and `use`. Each console command returns an object with a two attributes. The first, `message`, is the text that will be displayed to the user. The second, `success`, is a boolean indicating whether the command was able to be executed given the user inputed subject. More detail is provided for each of the console commands below.

### `die` Command

`die` deletes the user's current game and returns a message to the user.

### `drop` Command

`drop` checks to see if the `player` has an item in their `inventory` that matches the command's subject. If so the item is removed from the `player`'s `inventory` and is added to the player's current location's items. A confirmation message or a failure message is passed back to the user.

### `go` Command

`go` checks to see if the command's subject matches one of the room's exits. If it does the player's current location's `teardown()` function is run followed by the destination location's `setup()` function. The player's current location is then updated to the destination location. The player's new current location's description is returned if this is the first time the player has visited this location, otherwise the location's room is returned. A failure message is displayed if the user fails to provide a subject that matches an exit.

### `inventory` Command

`inventory` returns an enumerated list of all the items contained in the player's inventory.

### `load` Command

`load` will only run if the player is not currently playing a game. Similarly, `load` is the only command a player can execute if they are not currently playing a game. `load` copies the content of the cartridge file that matches the subject into the list of active games maintained by the Console and assigns it to the user.

### `look` Command

`look` will return the player's current room's description if the subject is not passed to it. If a subject is passed to it `look` instead attempts to return the description of the item in the player's inventory matching the description. Failing that, `look` attempts to return the description of the matching item in the player's current room. Finally, if that fails it attempts to return the look interaction of the matching interactable in the player's current room. 

### `take` Command

`take` checks to see if the player's current location has an item that matches the command's subject. If so the item is removed from the player's current location's list of items and is added to the player's inventory. A confirmation message or a failure message is then passed back to the user.

### `use` Command

The `use` command will run the `use` function of the item in the player's inventory that matches the command's subject.

## Cartridges

Cartridges are loaded into the Console and are then playable by the user. The Console does most of the heavy lifting while the Cartridge adds all the flavor. A Cartridge consists of two very important objects, `gameData` and `gameActions`, as well as any number of helper functions.

### `gameData` Object

The `gameData` object has four required components; `commandCounter`, `gameOver`, `introText`, `outroText`, a `player` object and a `map` object. It can also contain any number of other fields, objects and functions as needed by the cartridge. Below is bare-bones architecture of the `gameData` object.

```javaScript
exports.gameData = {
	commandCounter : 0,
	introText : 'Lorem Ipsum',
	player : { },
	map : {	}
};
```

#### `commandCounter`

The `commandCounter` is an int that keeps track of the number of inputs the user has submitted to the console for evaluation on this particular cartridge. The `commandCounter` can be initialized to any integer depending on the cartridges needs. The console will increment the number by one each time it receives a command.

#### `gameOver`

`gameOver` is a required boolean. After finishing all executing all code related to any given command the console checks `gameOver`. If it is true the `outroText` is added to the end of the string sent to the console and the game is deleted. If it is false nothing happens.

#### `introText`

The console will display the `introText` to the user following a successful `load` command.

#### `outroText`

The `outroText` string is added to the string sent to the terminal by the console if after finishing executing a command the `gameOver` boolean is set to true.

#### `player` Object

The `player` object has two required elements, `currentLocation` and `inventory`. It can also contain any number of other fields, objects and functions as needed by the cartridge. Below is bare-bones architecture of the `player` object.

```javaScript
player : {
	currentLocation : 'MineEntrance',
	inventory : {}
}
```

##### `currentLocation`

`currentLocation` is a string that points to the location that `player` is currently interacting with. It must be populated with a string that points to a matching location within the `map` object. That location will be the player's starting location. The console will keep `currentLocation` updated as the player moves from location to location within the game.

##### `inventory` Object

A player's `inventory` is a collection of `items` that the `player` takes with them from location to location as they move around within the game. The inventory can be empty or can have any number of `item`s that player will begin the game already possessing. The `item` object is explained in greater later in this document.

### `map` Object

The `map` is a collection of `location` objects. It can also contain any number of other fields, objects and functions as needed by the cartridge.

#### `location` Objects

Each `location` has a name/key and the follwoing attributes: `firstVisit`, `displayName`, `description`, an `interactables` object, and `items` object, and an `exits` object. It has three opitional functions that can be implemented if needed: `setup()`, `teardown()` and `updateLocation()`. It can also contain any number of other fields, objects and functions as needed by the cartridge.

##### `firstVisit`

`firstVisit` is a boolean that when true will cause the Console to display the `description` to the user upon entering this location via the `move` command. It it is instead false then the user will be presented with the `displayName` instead. The console will set `firstVisit` to false when the user uses the `move` command to leave a `location`.

##### `displayName`

The user is presented with a location's `displayName` if `firstVisit` is set to false when the user makes use of the `go` command to enter this location.

##### `description`

The user is presented with a location's `description` if `firstVisit` is set to true when the user makes use of the `go` command to enter this location. Additionaly, the user is presented with the 'description' if they issue the 'look' command without a subject.

##### `interactables` Object

A room may optionally have an `interactables` object. The `interatables` is a collection of interactions that the player can take while in this location. Each intractable object is it's own object within `interactables` which contains any number of key value pairs that map to its different interactions. An interaction can be either a string or a function that after executing any amount of arbitrary code returns a string. Below is an example of a barebones `interactables` object.

```javaScript
interactables : {
	interactable1 : { interaction : 'returnString' },
	interactable2 : {
		interaction1 : unshownHelperFunction(),
		interaction2 : function(){
			return 'returnString';
		}
	}
}
```

##### `items` Object

The `items` objects is a collection of items contained within this location. An item is also a JavaScript object and its structure is detailed below.

###### `item` Object

`item` objects can exist in both a player's inventory or they can reside in the `items` object of a location. Users can use the `take` command to move an `item` from a location to the player's inventory. The `drop` command does the reverse. Items have some mandatory properties; namely, `description`, `displayName`, `hidden`, and `quantity`. Optionally, it can contain a `use` function and/or an `interactions` Object. An `item` can also contain any number of additional properties or functions as dictated by the design of the game.

**`description`**

The 'description' of an `item` is the string returned when the user submits the 'look' command lists the item's name or `displayName` as the subject.

**`displayName`**

The `displayName` of an item is the string used by the console in all text related to the item that is displayed to the user.

**`hidden`**

`hidden` is a boolean that if set to true will prevent the console from listing the item as in the location when constructing the location's description. Alternatively if `hidden` is set to true the item will be listed in the location's description.

**`quantity`**

An item's `quantityy` must be a non-negative int. The `quantity` keeps track of how many of that specific item are contained within any given location of the player's inventory. The 'take' command causes the `quantity` to be decremented for the named item with the location's items object and incremented with the player's inventory. The `drop` command does the revers. If an item's quantity ever reaches zero it is deleted.

**`use()` Function**

The `use()` function will run then the user issues the 'use' command and names the item as the item as the subject. The use function an execute any arbitrary code but must return a string that will be displayed to the user.

**`interactions` Object**

The `interactions` Object of an item is identical to an `interactables` object of a location except that it is tied to the item and will move with said item.

##### `exits` Object

The `exits` object contains any number of `exit` objects that detail how this location is connected to other locations within the `map`.

###### `exit` Object

An `exit` object has two required keys; `displayName` and `destination`.

**`displayName`**

The `display` name is the string that is displayed to the user in all text related to the exit. It is listed in the location's description.

**`destination`**

The `displayName` is a string that matches the key to another `location` within the `map`. When the user issues the `go` command and names this exit as the subject the room listed in `destination` is the location the player's `currentLocation` will be changed to.

##### `setup()` Function

The `setup()` is an optional function that is run after the player's `currentLocation` is changed to this location by the `go` command but before the locations description is generated and returned to the user. It is not required to return anything and anything that is returned will be ignored by the Console.

##### `teardown()` Function

The `teardown()` is an optional function that is run before the player's `currentLocation` is changed from this location by the `go` command. It is not required to return anything and anything that is returned will be ignored by the Console.

##### `updateLocation()` Function

The `updateLocation()` function of the player's `currentLocation` is run following the execution of any successfully interpreted input submitted by the user. The function is passed the `command` object as generated by the parser, consisting of an `action`, a `subject` and an `object`. If this function returns anything that return value will be displayed to the user in lieu of the string generated by the Console.

### `gameActions` Object

The `gameActions` object holds functions that user can access as inputed actions. Functions within the object can feature new actions not implemented by the console or they can override the actions implemented in the console as explained in the Console section of this document. All function within the object are passed three parameters: a `game` object, a `command` object and a `consoleInterface` function. Each of the three parameters is explained below. All action functions must return a string that will be displayed to the user.

#### `game` Object

The `game` object is an item that contains both the `gameData` object and the `gameActions` object. It is a handy way to access or alter the `gameData` object.

#### `command` Object

The `command` object is the command that the user inputed after it has been parsed. It has three attributes: `action`, `subject` and `object`. The `action` will match the current action function. The `subject` is the what the action is to be performed on. The 'object' is the target of the action in conjunction with the `subject`.

#### `consoleInterface` Function

The `consoleInterface` function is a way for cartridge actions to call the console implemented actions. It has the following signature `consoleInterface(game, command)`. Below is an example of cartridge action, `take` that simply call it's cartridge counterpart:

```JavaScript
exports.gameActions = {
	take : function(game, command, consoleInterface){
		return consoleInterface(game, command);
	}
}
```

### Helper Functions

A cartridge can contain any number of additional functions as its design dictates.

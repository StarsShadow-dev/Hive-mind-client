# Hive-mind-client

### Work in progress example client for my game

- I do not have a non-localhost server set up right now

- player.png is a placeholder until I make a player image.

To run on macOS type `python3 -m http.server 8000` in to the terminal.

Go to `http://localhost:8000/game.html` in your web browser.

Please tell me if some thing in here requires clarification.

# Client server API

Start by creating a web socket
```js
const socket = new WebSocket(
    "wss://",
    // include UUID, pass and version in the protocols object
    [UUID, pass, "1.0.1"]
);
```
If UUID, pass or version are invalid the web socket will be **terminated immediately**.

## Receiving updates

To receive information from the web socket use:
```js
socket.addEventListener('message', (event) => {
    // Your amazing code goes here
});
```

While playing the game you'll receive arrays (in string form) of updates from the server.
These updates describe the state of the game to your client.

All updates contain a

* `type` a string describing the type of update

and a

* `data` an object describing that update

An example update:
```js
update = {
    type: "drone",
    data: {
        ownerUUID: "a UUID",
        UUID: "a UUID",
        x: 20,
        y: 30,
    },
}
```

List of update types:

* `drone` a drone exists
    * usually contains:
    * `UUID` UUID of the drone
    * `x` drones x coordinate
    * `y` drones y coordinate
    * also contains when drone is created:
    * `ownerUUID` UUID of the drones owner
* `noDrone` a drone no longer exists 
    * `UUID` UUID of the drone that is gone
    * TODO add information describing why the drone is gone
* `pro` a projectile exists
    * `id` id of the projectile
    * `x` projectiles x coordinate
    * `y` projectiles y coordinate
* `noPro` a projectile no longer exists
    * `id` id of the projectile
* `chat` a player has typed a chat message
    * `senderUUID` UUID of the sender
    * `name` name of the sender
    * `msg` message (as a string) from the sender
    * ### msg Is unsterilized and may contain malicious code from the sender ***never* inject into the HTML tree**

#### A basic implementation is:

```js
const socket = new WebSocket(
    "wss://",
    ["my UUID", "1234", "1.0.1"]
);
socket.addEventListener('message', (event) => {
    const updates = JSON.parse(event.data)
    // uncomment the line below to log all updates from the server
    // console.log('updates from server ', updates);

    updates.forEach(update => {
        const data = update.data
        switch (update.type) {
            case "drone":
                // create the drone if it does not exist
                // move the drone if it does exist
                break;
            case "noDrone":
                // remove the drone
                break;
            case "pro":
                // create the projectile if it does not exist
                // move the projectile if it does exist
                break;
            case "noPro":
                // remove the projectile
            case "chat":
                // display a chat message
                break;
        
            default:
                console.error("unknown update.type", update.type)
                break;
        }
    })
});
```

## Sending actions

Now that you're keeping your [client up-to-date with the server](#receiving-updates).

You can start sending actions to the server!

--- 

You can send one array (in string form) of actions every 100 milliseconds each array has a max length of 10 objects (0 to 9)

Sending too many objects will cause the web socket to be **terminated immediately**

The easiest way to follow this API is to make a socket stream. (example implementation below)

```js
var socket_stream = []
function loop() {
    if (socket.readyState == 1 && socket_stream.length != 0) {
        let actions = socket_stream.slice(0, 10)
        socket_stream.splice(0, 10)
        // uncomment the line below to log all actions from the client
        // console.log("actions sent to server", actions)
        socket.send(JSON.stringify(actions))
    }
    setTimeout(() => { loop() }, 100); // continue the loop
}
loop()
```

All actions contain a

* `type` a string describing the type of action

and a

* `data` an object containing additional information about the action

Now to send an action to the server:

```js
socket_stream.push({
    type: "move",
    data: {
        DUUID: "", // drone universally unique identifier
        x: 0,
        y: 0,
    },
})
```

List of action types:

* `makeDrone` Create a new drone
* `move` move an existing drone
    * `DUUID` drone UUID (the drone must be owned by you)
    * `x` x in world coordinates
    * `y` y in world coordinates
* `shoot`
    * `DUUID` drone UUID (the drone must be owned by you)
    * `rad` angle in radians that you would like to shoot in
* `scan`
    * `DUUID` drone UUID (the drone must be owned by you)
    * `UUID` universally unique identifier of the drone you want to scan (does not have to be owned by you)
* `chat`
    * `msg` a string containing a message you would like to broadcast to other players
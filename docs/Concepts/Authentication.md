# Authentication

When you have a multiplayer game, often you need to store information about your player for later games, keep game stats or communicate with your friends. For all these use cases, you often need a way to uniquely identify a user. Being able to tell users apart is called authentication. There are several methods available, some examples include:

-   Ask the user for username and password

-   Use a third party oath or OpenID identity provider, such as Facebook, Twitter, Google

-   Use a third party service such as PlayFab, GameLift or Steam

-   Use the device id, very popular method in mobile

-   Use Google Play in Android

-   Use Game Center in IOS

-   Use a web service in your website

Trying to write a comprehensive authentication framework that cover all these is very complex. There is no one size fit all, and we would quickly end up with bloated code.

Instead, **Mirror does not perform authentication**, but we provide hooks you can use to implement any of these.

Here is an example of how to implement simple username/password authentication:

1.  Select your `NetworkManager` game object in the unity editor.

2.  In the inspector, under `Spawn Info`, disable `Auto Create Player`

3.  Call `AddPlayer` in your client to pass the credentials.

4.  Override the `OnServerAddPlayer` method and validate the user's credential.

For example this would be part of your `NetworkManager` class:

```cs
class MyGameNetworkManager : NetworkManager {

    class CredentialsMessage : MessageBase
    {
        // use whatever credentials make sense for your game
        // for example, you might want to pass the accessToken if using oauth
        public string username;
        public string password;
    }

    // this gets called on the client after it has connected to the server
    public override void OnClientConnect(NetworkConnection conn)
    {
        base.OnClientConnect(conn);

        CredentialsMessage msg = new CredentialsMessage()
        {
            // perhaps get the username and password from textboxes instead
            username = "Joe",
            password = "Gaba Gaba"
        };

        ClientScene.AddPlayer(conn, MessagePacker.Pack(msg));
    }

    // this gets called in your server when the client requests to add a player.
    public override void OnServerAddPlayer(NetworkConnection conn, AddPlayerMessage extraMessage)
    {
        CredentialsMessage msg = MessagePacker.Unpack<CredentialsMessage>(extraMessage.value);

        // check the credentials by calling your web server, database table, playfab api, or any method appropriate.
        if (msg.username == "Joe" && msg.password == "Gaba Gaba")
        {
            // proceed to regular add player
            base.OnServerAddPlayer(conn, extraMessage);
        }
        else
        {
            conn.Disconnect();
        }
    }
}
```

## Warning

By default Mirror uses Telepathy, which is not encrypted. The above code sample works, but if you want to do authentication through Mirror, we highly recommend you use a transport that supports encryption.

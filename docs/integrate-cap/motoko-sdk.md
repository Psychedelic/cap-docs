---
date: "1"
---
# ⚡ Integrating CAP - Motoko-based Projects
## ⚡ Integrating CAP - Motoko-based Projects

To integrate CAP, a Token or NFT must use the **CAP Motoko SDK/Library**. A development kit we created to facilitate the integration of **new or existing projects written in Motoko**.

- Visit [CAP Motoko Library](https://github.com/Psychedelic/cap-motoko-library)
- CAP's Mainnet canister: `lj532-6iaaa-aaaah-qcc7a-cai`

-----

The following guide will take you through creating a sample project (NFT/Token) that uses **CAP for its event history**. It will create a simple token that registers with CAP and then submits and retrieves events. It is recommended that you read the CAP specifications to understand how CAP works. 

- **That can be found here **https://github.com/Psychedelic/cap/tree/main/spec

----

## 🦖 CAP Motoko Library Integration Example

This example guides you towards deploying a Motoko-based canister that integrates CAP to have a transaction/event history, using the CAP Motoko Library to both instantiate a history in CAP for your canister, and adding the appropriate "insert" requests to send event records to CAP.

With this guide you will:

1. Deploy a project in a local replica environment with CAP.
2. Install CAP on the canister to instantiate and connect its history.
3. Deploy your project on mainnet, with a live integration on mainnet with CAP.

Use the documentation here to understand how to run the separate services which are required in your local development environment.

Deploying to Mainnet, shouldn't be any different, although the version of [CAP](https://github/com/psychedelic/cap) might be diferent from the version you run locally, so keep track of releases.

## 📓 The Example Repository 

We have created a simple Motoko canister example on our GitHub:

- [View the repository here.](https://github.com/Psychedelic/cap-motoko-library/tree/main/examples/insert)

This example displays how one would integrate CAP to its project, and you can view the main [Motoko file](https://github.com/Psychedelic/cap-motoko-library/blob/main/examples/insert/src/main.mo) that the process looks like:

1. Connect with the proper CAP instance.
2. Instantiate a new root bucket/history in CAP for your canister.
3. Wrap your canister's methods/calls with CAP's insert method to insert events to your canister's history in CAP when the method is called.

In the case of an NFT or Token canister, you would replicate this structure with your own canister's methods.


## 🤔 How to run the Example Locally?

**TLDR;** Deploy the example to mainnet and use an actor to interact with it, either via [DFX CLI](https://sdk.dfinity.org/docs/developers-guide/cli-reference.html) or your [Agentjs](https://github.com/dfinity/agent-js).

If planning to run the examples in your local environment and not the mainnet network, then the main [CAP repo](https://github.com/Psychedelic/cap) should be cloned and deployed to your local replica!

Alternatively, the CAP Service handling can be borrowed from the [CAP Explorer](https://github.com/Psychedelic/cap-explorer), which is documented and is easy to grasp...

Once the `CAP router` is running in your local, copy the Router id; for our reading we'll name it `<Router ID>` to keep it easy to follow!

### 1. Set Up the Example Repository

When ready, open the directory for [one of our examples](https://github.com/Psychedelic/cap-motoko-library/blob/main/examples/insert/src/main.mo) (e.g. the `/cap-motoko-library/examples/insert`) and deploy the example to your local replica network, as follows:

```sh
dfx deploy cap-motoko-example --argument "(opt \"<Router ID>\")"
```

Obs: Notice that the `<Router ID>` is the Canister Id of the deployed local replica `ic-history-router` of [CAP Service](https://github.com/psychedelic/cap). We pass the `<Router ID>` to override the default Mainnet Router Canister id.

Make sure you execute the command in the correct directory, where a `dfx.json` exists describing the canister `cap-motoko-example`, otherwise it'll fail.

💡 When deploying, the `cap-motoko-example` is pulling a particular version of the [CAP Motoko Library](https://github.com/Psychedelic/cap-motoko-library) via the [Vessel Package Manager](https://github.com/dfinity/vessel/releases) which is described in the main README of the [CAP Motoko Library](https://github.com/Psychedelic/cap-motoko-library) repository. For example, you'll find the field `version` in the additions setup in the `package-set.dhall`, you can have another tag or a commit hash.

Now that we have deployed the `cap-motoko-example` we can find the Canister id in the output. So, from now on, we'll use `<Application Token Contract ID>` to refer to the `cap-motoko-example` to keep it easy to follow!

Here's an example of how the output should look like:

```sh
Deploying: cap-motoko-example
All canisters have already been created.
Building canisters...
Installing canisters...
Creating UI canister on the local network.
The UI canister on the "local" network is "<xxxxx>"
Installing code for canister cap-motoko-example, with canister_id "<Application Token Contract ID>"
Deployed canisters.
```

Copy the `<Application Token Contract ID>` because you are going to use it to send requests via the DFX CLI!

### 2. Deploy a Root History Canister in CAP

Now, we need to push our example source code to CAP! For that we have a `handshake` process that creates a new Root canister for us, with the right controller.

For our example, we're going to use the [DFX CLI]() to call a method in our example application actor, called `init`

```sh
dfx canister call <Application Token Contract ID> init "()"
```

It should take a bit, and once completed you'll find the output it similar to:

```sh
()
```

Where `()` is the returned value, if we did NOT get any errors during the process handling!

### 3. Test the History by Making a Call to Insert an Event

From then on we can simple use the remaining methods available, such as `insert`. This means that we do the initialisation only once and NOT everytime we need to make a CAP call.

To complete, we execute the `insert` to push some data to our Root bucket for our `<Application Token Contract ID>` example application.

```sh
dfx canister call <Application Token Contract ID> insert "()"
```

Here's how it looks:

```sh
(variant { ok = 0 : nat64 })
```

The `(variant { ok = 0 : nat64 })` is a wrapped response of the expected returned value, the transaction id as `nat64` (starts at zero), as we can verify by looking at the [Candid](https://github.com/Psychedelic/cap/blob/main/candid/root.did#L57) for CAP Root. It's wrapped by our example `insert` method.

👋 That's it! You can now use the CAP Motoko Library in your local replica and the same knowledge can be applied to deploy to the [mainnet](#-deploying-the-examples-to-mainnet)!

## 🚀 Deploying the Examples to Mainnet

We start by deploying to the Mainnet by using the flag `--network ic`, and omit the constructor argument which is used locally to override the Mainnet Router ID. By setting `null`, we cause the constructor to use the Router ID Mainnet.

```sh
dfx deploy --network ic --argument "(null)"
```

The output should show:

```sh
Creating canister "cap-motoko-example"...
Installing code for canister cap-motoko-example, with canister_id <Application Token Contract Id>
Deployed canisters.
```

Copy the `<Application Token Contract Id>`, as it's needed for initialisation.

### 2. Initialize CAP in Your Mainnet Canister


The initialisation is the process that calls the CAP `handshake` endpoint to create root/history canister in CAP for your project. Your events will be stored in this canister exclusively.

```sh
dfx canister --network ic call <Application Token Contract Id> init "()"
```

It should take a bit, and once completed you'll find the output it similar to:

```sh
()
```

### 3. Test Your Mainnet Canister, Call a Method and Insert Data

From then on we can simple use the remaining methods available, such as `insert`. The same principals we found in the local replica apply here, so we only need to call the initialisation once.

To complete, we execute the `insert` to push some data to our Root bucket for our `<Application Token Contract ID>` example application.

```sh
dfx canister --network ic call <Application Token Contract ID> insert "()"
```

Here's how the output looks (e.g. if you request a new `insert`, then the ok number will increase, as that's the wrapped transaction id):

```sh
(variant { ok = 0 : nat64 })
```

👋 Well done!
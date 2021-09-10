# Fluence Character Counter

This is a tutorial on how to create a character count application using Fluence. [Fluence](https://fluence.network/) is an easy to use platform for creating fast peer-to-peer applications.

The deployed contract from this tutorial is accessible at [Character Count Service](https://dash.fluence.dev/blueprint/5c187205fef8d3306dcf6ffd73efd623a8774d3adc698e6244bc11e81d704ba6).

### About this tutorial

It is recommended that you first follow the section "Quick Start" in the excellent getting started tutorial from the Fluence documentation.

[Quick Start](https://doc.fluence.dev/docs/quick-start)

The tutorial presented here builds on the Quick Start tutorial by extending the basic hello world app to add message sending and character count functionality.

Along the way we link to source files within this repository so that you open these follow along. You will find that the code required to deploy a peer-to-peer application with Fluence is very simple and concise.

Unless otherwise indicated, the applicable license is [Apache 2.0](https://github.com/fluencelabs/fluence/blob/master/LICENSE).

### The following tools are used within this tutorial

```bash
    $ marine           – build wasm from Rust  
    $ aqua-cli         – compile Aqua to AIR + Typescript  
    $ fldist           – deploy & query services  
```

### Character Count Extension

The task is to extend the simple hello world example to add a character count to sent messages.

As you will have learned from the [Quick Start](https://doc.fluence.dev/docs/quick-start) tutorial, Fluence provides a Docker container pre-configured with example applications. We will be extending code in the examples/quickstart folder. We first prepare the version control for the examples code to create our own repository.

```bash
cd /workspaces/devcontainer/examples
rm -r ./.git
git init
git remote add origin git@github.com:ben-razor/Fluence-Service.git
git add .
git commit -m "Initial commit"
git branch -M main
git push -u origin main
```

### Extending The Service

The rust code and configuration files for the service that will be deployed resides in the [quickstart/2-hosted-services](https://github.com/ben-razor/Fluence-Service/tree/main/quickstart/2-hosted-services) folder. We modify the code in [src/main.rs](https://github.com/ben-razor/Fluence-Service/tree/main/quickstart/2-hosted-services) to add a character count to the message reply.

```rust
#[marine]
pub struct CharCount {
    pub msg: String,
    pub reply: String,
}

#[marine]
pub fn char_count(message: String) -> CharCount {
    let num_chars = message.chars().count();
    let _msg;
    let _reply;

    if num_chars < 1 {
        _msg = format!("Your message was empty");
        _reply = format!("Your message has 0 characters");
    } else {
        _msg = format!("Message: {}", message);
        _reply = format!("Your message {} has {} character(s)", message, num_chars);
    }

    CharCount {
        msg: _msg,
        reply: _reply
    }
}
```

Before building we must also refactor all referencies to hello_world, hello and HelloWorld in the configs directory and Cargo.toml file to use the new char_count module, char_count function name and CharCount struct.

Once this is done we build the service. This will create a WebAssembly file char_count.wasm in ./artifacts that will be deployed later.

```bash
cd quickstart
cd 2-hosted-services
./scripts/build.sh
```

We update the tests in [src/main.rs](https://github.com/ben-razor/Fluence-Service/tree/main/quickstart/2-hosted-services) to work with the new service. We enter a special character in the string to ensure that special characters are being counted correctly. 

Using the following command we test that our service is working as expected before deployment:

```bash
cargo +nightly test --release
```

### Deployment To Fluence

Once the service is built and the tests pass, we are ready to deploy the service to Fluence.

Services are deployed to specific peers that will handle our requests. We can gather a list of test peers using the command:

```bash
fldist env
```

We choose the top entry from the list for the deployment and use the node id with the fldist deployment command:

```bash
fldist --node-id 12D3KooWSD5PToNiLQwKDXsu8JSysCwUt8BVUJEqCHcDe7P5h45e \
       new_service \
       --ms artifacts/char_count.wasm:configs/char_count_cfg.json \
       --name char-count-br
```

Running this command returned the service id **9454c078-1b68-4ae1-bb30-b82690d5fec0** which we can use later to interact with the service. (Your deployed service id will be different, or you can use this one if you are only performing front end experiments).

### Fluence Developer Hub

You can ensure your service has been deployed by viewing the [Fluence Developer Hub](https://dash.fluence.dev/)

The deployed contract used in this tutorial can be viewed at:
[Character Count Service](https://dash.fluence.dev/blueprint/5c187205fef8d3306dcf6ffd73efd623a8774d3adc698e6244bc11e81d704ba6)

### Updating The Aqua Code 

[Aqua](https://doc.fluence.dev/aqua-book/) is a simplified language for defining peer-to-peer applications with Fluence.

With the character count contract deployed, we move to the front end code. This is located in the [quickstart/3-browser-to-service](https://github.com/ben-razor/Fluence-Service/tree/main/quickstart/3-browser-to-service).

Our changes focus on the [getting-started.aqua](https://github.com/ben-razor/Fluence-Service/tree/main/quickstart/3-browser-to-service/aqua/getting-started.aqua) file.

Firstly we change the service id and peer id to match those returned when we deployed the contract.

Secondly, we refactor all references related to hello world to char count versions.

The most important change is that we will now need to pass through our message (messageToSend) and need to update the interface to support this:

```TypeScript
func countChars(messageToSend: string, targetPeerId: PeerId, targetRelayPeerId: PeerId) -> string:
    -- execute computation on a Peer in the network
    on charCountNodePeerId:
        CharCount charCountServiceId
        comp <- CharCount.char_count(messageToSend)

    -- send the result to target browser in the background
    co on targetPeerId via targetRelayPeerId:
        res <- CharCountPeer.char_count(messageToSend)

    -- send the result to the initiator
    <- comp.reply
```

We now run the aqua compiler which processes the Aqua code and generates a typescript file in src/_aqua/getting-started.ts that we can use in our web app.

```bash
aqua --input ./aqua/getting-started.aqua --output ./src/_aqua/
```

### Updating The Front End Code

Finally we need to update the front end code to interact with our new character count service via the interfaces defined in the Aqua file.

The front end code is found in the [src/App.tsx](https://github.com/ben-razor/Fluence-Service/tree/main/quickstart/3-browser-to-servicesrc/src/App.tsx) file. For this simplified application this mostly requires changing hello and hello world variable names to their character count alternatives that we used in the getting-started.aqua file.

We also add a message box so that we can send our custom messages so that we can say more than a simple hello.

### Running The Updated Application

The application is started by running:

```
npm start
```

1. We are presented with a list of relays (you can choose any). You will then be presented with the main interface for the application:

![Main Interface](https://github.com/ben-razor/Fluence-Service/blob/main/img/2-peer-selected.png)

2. Once this is selected we fire up (or get a friend to fire up) another browser window and do the same.

![Other Client](https://github.com/ben-razor/Fluence-Service/blob/main/img/3-second-peer-selected.png)

3. Next we fire off our amazing message to the other peer id:

![Send Message](https://github.com/ben-razor/Fluence-Service/blob/main/img/4-pre-message-send.png)

4. We get back out message with the characters counted!

![Character Count Displayed](https://github.com/ben-razor/Fluence-Service/blob/main/img/5-character-count-displayed.png)

And as a bonus our message gets sent through to the other client like magic!

![Message sent to other client](https://github.com/ben-razor/Fluence-Service/blob/main/img/6-message-send-to-other-client.png)

### Wrapping Up

And that's it! I hope this tutorial will help you to make some amazing applications using fluence.

---
title: Using Fluid with NodeJS
menuPosition: 3
author: sdeshpande3
aliases:
  - "/start/nodejs-tutorial/"
---

In this tutorial, you'll learn about using the Fluid Framework by building a simple application in NodeJS that enables connected clients to generate random numbers and display the result of any changes to the shared state.  You'll also learn how to connect the Fluid data layer in [Node](https://nodejs.org/).

To jump ahead into the finished demo, check out the [Node demo in our FluidExamples repo](https://github.com/microsoft/FluidExamples/tree/main/node-demo).

The following image shows the random number generated open in four terminals after every one second.

![Four terminals with the Random Number app open in them.](https://user-images.githubusercontent.com/46719950/140808244-abb4a046-6170-48e2-8766-c497e6ce977d.png)

{{< callout note >}}

This tutorial assumes that you are familiar with the [Fluid Framework Overview]({{< relref "/docs/_index.md" >}}) and that you have completed the [QuickStart]({{< relref "quick-start.md" >}}). You should also be familiar with the basics of [Node](https://nodejs.org/en/) and [creating Node projects](https://nodejs.org/en/about/).

{{< /callout >}}

## Create the project

1. Open a Command Prompt and navigate to the parent folder where you want to create the project; e.g., `c:\My Fluid Projects\fluid-node-tutorial`. The project is created in a subfolder named `fluid-node-tutorial`.

1. Run the following command at the prompt.

   ```dotnetcli
   npm init
   ```

1. The project uses two Fluid libraries:

    |Library |Description |
    |---|---|
    | `fluid-framework`    |Contains the SharedMap [distributed data structure]({{< relref "dds.md" >}}) that synchronizes data across clients. *This object will hold the most recent timestamp update made by any client.*|
    | `@fluidframework/tinylicious-client`   |Defines the connection to a Fluid service server and defines the starting schema for the [Fluid container][].|
    {.table}

    Run the following command to install the libraries.

    ```dotnetcli
    npm install @fluidframework/tinylicious-client fluid-framework readline-async
    ```

## Code the project

1. Create `\src\index.js` file in your code editor and add the imports. The file should look like the following:

   ```js
   import { TinyliciousClient } from "@fluidframework/tinylicious-client";
   import { SharedMap } from "fluid-framework";
   import readlineAsync from "readline-async";
   ```

### Move Fluid data to the terminal

1. The Fluid runtime will bring changes made to the value from any client to the current client. Add the following code below the import statements.

    ```js

    // TODO 1: Configure the container

    async function readInput() {
      // TODO 2: Read the input container id
    }

    function loadCli(map) {
       // TODO 3: Set the value that will appear on the terminal

       // TODO 4: Register handlers
   }

    async function createContainer() {
      // TODO 5: Create the container
    }

    async function loadContainer(id) {
      // TODO 6: Get the container from the Fluid service
    }

    async function start() {
        // TODO 7: Read container id from terminal and create/load container
    }

   start().catch(console.error());
    ```

1. Replace `TODO 1` with the following code. Note that there is only one object in the container: a SharedMap holding the random number. Note also that `sharedRandomNumber` is the ID of the `SharedMap` object and it must be unique within the container.

    ```js
      const client = new TinyliciousClient();
      const containerSchema = {
          initialObjects: { sharedRandomNumber: SharedMap }
      };
    ```

1. Replace `TODO 2` with the following code. Note that `containerId` is taken as an input, and if there is no `containerId` we create a new container instead.

    ```js
    let containerId = "";
    console.log("Type a Container ID or press Enter to continue: ");
    await readlineAsync().then( line => {
        console.log("You entered: " + line);
        containerId = line;
    });
    return containerId;
    ```

1. To ensure that both local and remote changes to the random number are reflected in the terminal, we will use the `newRandomNumber()` function to store the local value and `updateConsole()` function to ensure that the console is updated whenever any client changes the value. This helps in keeping the client terminals synchronized with the Fluid data.

   Replace `TODO 3` with the following code. It will set a timer to update the random number every one second.

   ```js
   const newRandomNumber = () => {
     map.set("random-Number-Key", Math.floor(Math.random() * 100) + 1);
   };
   setInterval(newRandomNumber, 1000);
   ```

  Replace `TODO 4` with the following code. It will listen for updates and print changes to the random number.

   ```js
   const updateConsole = () => {
     console.log("Value: ", map.get("random-Number-Key"));
   }
   updateConsole();
   map.on("valueChanged", updateConsole);
   ```

1. Replace `TODO 5` with the following code.

    ```js
    const { container } = await client.createContainer(containerSchema);
    container.initialObjects.map.set("random-Number-Key", 1);
    const id = await container.attach();
    console.log("Initializing Node Client----------", id);
    loadCli(container.initialObjects.map);
    return id;
    ```

1. Replace `TODO 6` with the following code.

   ```js
   const { container } = await client.getContainer(id, containerSchema);
   console.log("Loading Existing Node Client----------", id);
   loadCli(container.initialObjects.map);
   ```

2. Replace `TODO 7` with the following code. Note that, this code will first take the container id as the input. To create a new Fluid container, press Enter or type `undefined`. A new container will be initialized and the container id will be printed in the terminal. You can copy the container id, launch a new terminal window, and type/paste the initial container id to have multiple collaborative NodeJS clients.

   ```js
   const containerId = await readInput();

   if(containerId.length === 0 || containerId === 'undefined' || containerId === 'null') {
     await createContainer();
   } else {
     await loadContainer(containerId);
   }
   ```

## Start the Fluid server and run the application

In the Command Prompt, run the following command to start the Fluid service. Note that `tinylicious` is the name of the Fluid service that runs on localhost.

   ```dotnetcli
   npx tinylicious@latest
   ```

Open a new Command Prompt and navigate to the root of the project; for example, `C:/My Fluid Projects/fluid-node-tutorial`. Start the application server with the following command. The application opens in your terminal.

   ```dotnetcli
   npm run start:client
   ```

To create a new Fluid container press Enter. The container id will be printed in the terminal. Copy the container id, launch a new terminal window, and type/paste the initial container id to have multiple collaborative NodeJS clients.

## Next steps

- You can find the completed code for this example in our Fluid Examples GitHub repository [here](https://github.com/microsoft/FluidExamples/tree/main/node-demo).
- Try extending the demo with more key/value pairs and a more complex framework such as Express.
- Try changing the container schema to use a different shared data object type or specify multiple objects in `initialObjects`.

{{< callout tip >}}

When you make changes to the code the project will automatically rebuild and the application server will reload. However, if you make changes to the container schema, they will only take effect if you close and restart the application server. To do this, give focus to the Command Prompt and press Ctrl-C twice. Then run `npm run start:client` again.

{{< /callout >}}

<!-- AUTO-GENERATED-CONTENT:START (INCLUDE:path=docs/_includes/links.md) -->
<!-- Links -->

<!-- Concepts -->

[Fluid container]: {{< relref "containers.md" >}}

<!-- Classes and interfaces -->

[FluidContainer]: {{< relref "fluidcontainer.md" >}}
[IFluidContainer]: {{< relref "ifluidcontainer.md" >}}
[SharedCounter]: {{< relref "/docs/data-structures/counter.md" >}}
[SharedMap]: {{< relref "/docs/data-structures/map.md" >}}
[SharedNumberSequence]: {{< relref "sequences.md#sharedobjectsequence-and-sharednumbersequence" >}}
[SharedObjectSequence]: {{< relref "sequences.md#sharedobjectsequence-and-sharednumbersequence" >}}
[SharedSequence]: {{< relref "sequences.md" >}}
[SharedString]: {{< relref "string.md" >}}

<!-- AUTO-GENERATED-CONTENT:END -->

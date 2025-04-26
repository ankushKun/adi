# EN Technical Documentation
​
Generated: 2025-03-17T15:10:13.422Z
​
This file contains technical reference documentation and release notes.
​
# API AND FUNCTION REFERENCES
​
## ao Module
​
Source: https://cookbook_ao.arweave.net/references/ao.html
​
version: 0.0.3
​
`ao` process communication is handled by messages, each process receives messages in the form of [ANS-104 DataItems](https://specs.arweave.net/view/xwOgX-MmqN5_-Ny_zNu2A8o-PnTGsoRb_3FrtiMAkuw), and needs to be able to do the following common operations.
​
- [ao.send(msg)](#ao-send-msg-message) - send message to another process
- [ao.spawn(module, msg)](#ao-spawn-module-string-spawn-spawn) - spawn a process
- [ao.isTrusted(msg)](#ao-istrusted-msg-message) - check to see if this message trusted?
​
The goal of this library is to provide this core functionality in the box of the `ao` developer toolkit. As a developer you have the option to leverage this library or not, but it integrated by default.
​
## Properties
​
| Name               | Description                                                                                                  | Type     |
| ------------------ | ------------------------------------------------------------------------------------------------------------ | -------- |
| id                 | Process Identifier (TxID)                                                                                    | string   |
| \_module           | Module Identifier (TxID)                                                                                     | string   |
| authorities        | Set of Trusted TXs                                                                                           | string   |
| Authority          | Identifiers that the process is able to accept transactions from that are not the owner or the process (0-n) | string   |
| \_version          | The version of the library                                                                                   | string   |
| reference          | Reference number of the process                                                                              | number   |
| env                | Evaluation Environment                                                                                       | object   |
| outbox             | Holds Messages and Spawns for response                                                                       | object   |
| assignables        | List of assignables of the process                                                                           | list     |
| nonExtractableTags | List of non-extractable tags of the process                                                                  | list     |
| nonForwardableTags | List of non-forwardable tags of the process                                                                  | list     |
| init               | Initializes the AO environment                                                                               | function |
| send               | Sends a message to a target process                                                                          | function |
| assign             | Assigns a message to the process                                                                             | function |
| spawn              | Spawns a process                                                                                             | function |
| result             | Returns the result of a message                                                                              | function |
| isTrusted          | Checks if a message is trusted                                                                               | function |
| isAssignment       | Checks if a message is an assignment                                                                         | function |
| isAssignable       | Checks if a message is assignable                                                                            | function |
| addAssignable      | Adds an assignable to the assignables list                                                                   | function |
| removeAssignable   | Removes an assignable from the assignables list                                                              | function |
| clearOutbox        | Clears the outbox                                                                                            | function |
| normalize          | Normalizes a message by extracting tags                                                                      | function |
| sanitize           | Sanitizes a message by removing non-forwardable tags                                                         | function |
| clone              | Clones a table recursively                                                                                   | function |
​
### Environment Schema
​
The `ao.env` variable contains information about the initializing message of the process. It follows this schema:
​
```lua
ao.env = {
  Process = {
    Id = string,      -- Process ID
    Owner = string,   -- Process owner
    TagArray = {      -- Array of name-value pairs
      { name = string, value = string }
    },
    Tags = {          -- Tags as key-value pairs
      [string] = string
    }
  }
}
```
​
#### Example
​
```lua
{
  Process = {
    Id = "A1b2C3d4E5f6G7h8I9j0K1L2M3N4O5P6Q7R8S9T0",
    Owner = "Xy9PqW3vR5sT8uB1nM6dK0gF2hL4jC7iE9rV3wX5",
    TagArray = {
      { name = "App-Name", value = "aos" }
    },
    Tags = {
      ["App-Name"] = "aos"
    }
  }
}
```
​
## Methods
​
### `ao.send(msg: Message)`
​
Takes a [Message](#message) as input. The function adds `ao`-specific tags and stores the message in `ao.outbox.Messages`.
​
#### Example
​
```lua
local message = ao.send({
    Target = msg.From,
    Data = "ping",
    Tags = {
        ["Content-Type"] = "text/plain",
        ["Action"] = "Ping"
    }
})
​
-- or
​
local message = ao.send({
    Target = msg.From,
    Data = "ping",
    Action = "Ping",               -- will be converted to Tags
    ["Content-Type"] = "text/plain"  -- will be converted to Tags
})
```
​
### `ao.spawn(module: string, spawn: Spawn)`
​
Takes a module ID string and [Spawn](#spawn) as input. Returns a Spawn table with a generated `Ref_` tag.
​
#### Example
​
```lua
local process = ao.spawn("processId", {
    Data = { initial = "state" },
    Tags = {
        ["Process-Type"] = "calculator"
    }
})
```
​
### `ao.isTrusted(msg: Message)`
​
Takes a [Message](#message) as input. Returns `true` if the message is from a trusted source.
​
#### Example
​
```lua
if ao.isTrusted(msg) then
    -- Process trusted message
else
    -- Handle untrusted message
end
```
​
### `ao.assign(assignment: Assignment)`
​
Takes an [Assignment](#assignment) as input. Adds the assignment to `ao.outbox.Assignments`.
​
#### Example
​
```lua
ao.assign({
    Processes = {"process-1", "process-2"},
    Message = "sample-message-id"
})
```
​
### `ao.result(result: Result)`
​
Takes a [Result](#result) as input. Returns the final process execution result.
​
#### Example
​
```lua
local process_result = ao.result({
    Output = "Process completed successfully",
    Messages = {
        { Target = "ProcessY", Data = "Result data", Tags = { ["Status"] = "Success" } }
    },
    Spawns = { "spawned-process-1" },
    Assignments = { {Processes = { "process-1" }, Message = "assignment-message-id"} }
})
```
​
### `ao.isAssignable(msg: Message)`
​
Takes a [Message](#message) as input. Returns `true` if the message matches a pattern in `ao.assignables`.
​
#### Example
​
```lua
local can_be_assigned = ao.isAssignable({
    Target = "ProcessA",
    Data = "Some content",
    Tags = {
         ["Category"] = "Info"
    }
})
```
​
### `ao.isAssignment(msg: Message)`
​
Takes a [Message](#message) as input. Returns `true` if the message is assigned to a different process.
​
#### Example
​
```lua
local is_assigned_elsewhere = ao.isAssignment({
    Target = "AnotherProcess"
})
```
​
## Custom `ao` Table Structures
​
### Tags
​
Used by: `ao.send()`, `ao.spawn()`, `ao.normalize()`, `ao.sanitize()`
​
All of the below syntaxes are valid, but each syntax gets converted to `{ name = string, value = string }` tables behind the scenes. We use **alternative 1** throughout the documentation for brevity and consistency.
​
```lua
-- Default: Array of name-value pair tables
Tags = {
    { name = "Content-Type", value = "text/plain" },
    { name = "Action", value = "Ping" }
}
​
-- Alternative 1: Direct key-value pairs in Tags table using string keys
Tags = {
    ["Content-Type"] = "text/plain",
    ["Action"] = "Ping"
}
​
-- Alternative 2: Direct key-value pairs in Tags table using dot notation
Tags = {
    Category = "Info",
    Action = "Ping"
}
```
​
::: info Root-level Tag Conversion
Any keys in the root message object that are not one of: `Target`, `Data`, `Anchor`, `Tags`, or `From` will automatically be converted into Tags using the key as the tag name and its value as the tag value.
​
```lua
-- These root-level keys will be automatically converted to Tags
{
    Target = "process-id",
    Data = "Hello",
    ["Content-Type"] = "text/plain",  -- Will become a Tag
    Action = "Ping"                   -- Will become a Tag
}
```
​
:::
​
### Message
​
Used by: `ao.send()`, `ao.isTrusted()`, `ao.isAssignment()`, `ao.isAssignable()`, `ao.normalize()`, `ao.sanitize()`
​
```lua
-- Message structure
{
    Target = string,     -- Required: Process/wallet address
    Data = any,          -- Required: Message payload
    Tags = Tag<table>
}
```
​
### Spawn
​
Used by: `ao.spawn()`
​
```lua
-- Spawn structure
{
    Data = any,          -- Required: Initial process state
    Tags = Tag<table>    -- Required: Process tags
}
```
​
### Assignment
​
Used by: `ao.assign()`, `ao.result()`
​
```lua
-- Assignment configuration table structure
{
    Processes = { string }, -- Required: List of target process ID strings
    Message = string       -- Required: Message to assign
}
```
​
### Result
​
Used by: `ao.result()`
​
```lua
-- Process result structure
{
    Output = string,           -- Optional: Process output
    Messages = Message<table>,   -- Optional: Generated messages
    Spawns = Spawn<table>,        -- Optional: Spawned processes
    Assignments = Assignment<table>,    -- Optional: Process assignments
    Error = string         -- Optional: Error information
}
```
​
## BetterIDEa
​
Source: https://cookbook_ao.arweave.net/references/betteridea/index.html
​
[BetterIDEa](https://ide.betteridea.dev) is a custom web based IDE for developing on ao.
​
It offers a built in Lua language server with ao definitions, so you don't need to install anything. Just open the IDE and start coding!
​
Features include:
​
- Code completion
- Cell based notebook ui for rapid development
- Easy process management
- Markdown and Latex cell support
- Share projects with anyone through ao processes
- Tight integration with [ao package manager](https://apm.betteridea.dev)
​
Read detailed information about the various features and integrations of the IDE in the [documentation](https://docs.betteridea.dev).
​
## Community Resources
​
Source: https://cookbook_ao.arweave.net/references/community.html
​
This page provides a comprehensive list of community resources, tools, guides, and links for the AO ecosystem.
​
## Core Resources
​
[Autonomous Finance](https://www.autonomous.finance/)
​
- Autonomous Finance is a dedicated research and technology entity, focusing on the intricacies of financial infrastructure within the ao network.
​
[BetterIdea](https://betteridea.dev/)
​
- Build faster, smarter, and more efficiently with BetterIDEa, the ultimate native web IDE for AO development
​
[0rbit](https://www.0rbit.co/)
​
- 0rbit provides any data from the web to an ao process
  by utilizing the power of ao, and 0rbit nodes.
  The user sends a message to the 0rbit ao, 0rbit nodes fetches the data and the user process receives the data.
​
[ArweaveHub](https://arweavehub.com/)
​
- A community platform for the Arweave ecosystem featuring events, developer resources, and discovery tools.
​
[AR.IO](https://ar.io/)
​
- The first permanent cloud network built on Arweave, providing infrastructure for the permaweb with no 404s, no lost dependencies, and reliable access to applications and data through gateways, domains, and deployment tools.
​
## Developer Tools
​
- [AO Package Manager](https://apm_betteridea.arweave.net)
​
## Contributing
​
> Not seeing an AO Community Member or resource? Create an issue or submit a pull request to add it to this page: https://github.com/permaweb/ao-cookbook
​
## Cron Messages
​
Source: https://cookbook_ao.arweave.net/references/cron.html
​
ao has the ability to generate messages on a specified interval, this interval could be seconds, minutes, hours, or blocks. These messages automatically get evaluated by a monitoring process to inform the Process to evaluate these messages over time. The result is a real-time Process that can communicate with the full ao network or oracles in the outside network.
​
## Setting up cron in a process
​
The easiest way to create these cron messages is by spawning a new process in the aos console and defining the time interval.
​
```sh
aos [myProcess] --cron 5-minutes
```
​
When spawning a new process, you can pass a cron argument in your command line followed by the interval you would like the cron to tick. By default, cron messages are lazily evaluated, meaning they will not be evaluated until the next scheduled message. To initiate these scheduled cron messages, call `.monitor` in aos - this kicks off a worker process on the `mu` that triggers the cron messages from the `cu`. Your Process will then receive cron messages every `x-interval`.
​
```lua
.monitor
```
​
If you wish to stop triggering the cron messages simply call `.unmonitor` and this will stop the triggering process, but the next time you send a message, the generated cron messages will still get created and processed.
​
## Handling cron messages
​
Every cron message has an `Action` tag with the value `Cron`. [Handlers](handlers.md) can be defined to perform specific tasks autonomously, each time a cron message is received.
​
```lua
Handlers.add(
  "CronTick", -- Handler name
  Handlers.utils.hasMatchingTag("Action", "Cron"), -- Handler pattern to identify cron message
  function () -- Handler task to execute on cron message
    -- Do something
  end
)
```
​
Cron messages are a powerful utility that can be used to create "autonomous agents" with expansive capabilities.
​
## Accessing Data from Arweave with ao
​
Source: https://cookbook_ao.arweave.net/references/data.html
​
There may be times in your ao development workflow that you want to access data from Arweave. With ao, your process can send an assignment instructing the network to provide that data to your Process.
​
To request data from Arweave, you simply call `Assign` with a list of every `Process` you would like to assign the data to, along with a `Message`, which is the TxID of a Message.
​
```lua
Assign({
  Processes = { ao.id },
  Message = 'message-id'
})
​
```
​
You can also call `Send` with a table of process IDs in the `Assignments` parameter. This will tell the network to generate the Message and then assign it to all of the process IDs in the `Assignments` list.
​
```lua
Send({
  Target = ao.id,
  Data = 'Hello World',
  Assignments = { 'process-id-1', 'process-id-2' }
})
```
​
## Why data from Arweave?
​
Your Process may need to access data from a message to make a decision about something, or you may want to add features to your Process via the data load feature. Alternatively, you may want to access a message from a process without replicating the entire message.
​
## Editor setup
​
Source: https://cookbook_ao.arweave.net/references/editor-setup.html
​
Remembering all the built in ao functions and utilities can sometimes be hard. To enhance your developer experience, it is recommended to install the [Lua Language Server](https://luals.github.io) extension into your favorite text editor and add the [ao addon](https://github.com/martonlederer/ao-definitions). It supports all built in aos [modules](../guides/aos/modules/index.md) and [globals](../guides/aos/intro.md#globals).
​
## VS Code
​
Install the [sumneko.lua](https://marketplace.visualstudio.com/items?itemName=sumneko.lua) extension:
​
1. Search for "Lua" by sumneko in the extension marketplace
2. Download and install the extension
3. Open the VS Code command palette with `Shift + Command + P` (Mac) / `Ctrl + Shift + P` (Windows/Linux) and run the following command:
​
```
> Lua: Open Addon Manager
```
​
4. In the Addon Manager, search for "ao", it should be the first result. Click "Enable" and enjoy autocomplete!
​
## Other editors
​
1. Verify that your editor supports the [language server protocol](https://microsoft.github.io/language-server-protocol/implementors/tools/)
2. Install Lua Language Server by following the instructions at [luals.github.io](https://luals.github.io/#install)
3. Install the "ao" addon to the language server
​
## BetterIDEa
​
[BetterIDEa](https://ide.betteridea.dev) is a custom web based IDE for developing on ao.
​
It offers a built in Lua language server with ao definitions, so you don't need to install anything. Just open the IDE and start coding!
​
Features include:
​
- Code completion
- Cell based notebook ui for rapid development
- Easy process management
- Markdown and Latex cell support
- Share projects with anyone through ao processes
- Tight integration with [ao package manager](https://apm.betteridea.dev)
​
Read detailed information about the various features and integrations of the IDE in the [documentation](https://docs.betteridea.dev).
​
## glossary
​
Source: https://cookbook_ao.arweave.net/references/glossary.html
​
<style>
  .glossary-iframe {
    height: 400px;
    width: 100%;
    border: none;
  }
  
  html:not(.dark) .dark-mode-iframe {
    display: none;
  }
  
  html.dark .light-mode-iframe {
    display: none;
  }
</style>
​
<iframe class="glossary-iframe light-mode-iframe" src="https://glossary.permagate.io/"></iframe>
<iframe class="glossary-iframe dark-mode-iframe" src="https://glossary.permagate.io/?link-color=%2334d399&bg-color=%231b1b1f&text-color=%23e0e0e0&border-color=%23444444&hover-bg=%23222222&heading-color=%23ffffff&button-bg=%23444444&button-text=%23ffffff&section-bg=%23333333&section-color=%23ffffff&category-bg=%23333333&category-text=%23ffffff&tag-bg=%233a3a3a&tag-text=%23e0e0e0&secondary-text=%23a0a0a0&result-bg=%231e1e1e&result-hover=%23333333"></iframe>
​
## Handlers (Version 0.0.5)
​
Source: https://cookbook_ao.arweave.net/references/handlers.html
​
## Overview
​
The Handlers library provides a flexible way to manage and execute a series of process functions based on pattern matching. An AO process responds based on receiving Messages, these messages are defined using the Arweave DataItem specification which consists of Tags, and Data. Using the Handlers library, you can define a pipeline of process evaluation based on the attributes of the AO Message. Each Handler is instantiated with a name, a pattern matching function, and a function to execute on the incoming message. This library is suitable for scenarios where different actions need to be taken based on varying input criteria.
​
## Concepts
​
### Handler Arguments Overview
​
When adding a handler using `Handlers.add()`, you provide three main arguments:
​
1. `name` (string): The identifier for your handler
2. `pattern` (table or function): Defines how to match incoming messages
3. `handler` (function or resolver table): Defines what to do with matched messages
​
### Pattern Matching Tables
​
Pattern Matching Tables provide a declarative way to match incoming messages based on their attributes. This is used as the second argument in `Handlers.add()` to specify which messages your handler should process.
​
#### Basic Pattern Matching Rules
​
1. **Simple Tag Matching**
​
   ```lua
   { "Action" = "Do-Something" }  -- Match messages that have an exact Action tag value
   ```
​
2. **Wildcard Matching**
​
   ```lua
   { "Recipient" = '_' }  -- Match messages with any Recipient tag value
   ```
​
3. **Pattern Matching**
​
   ```lua
   { "Quantity" = "%d+" }  -- Match using Lua string patterns (similar to regex)
   ```
​
4. **Function-based Matching**
   ```lua
   { "Quantity" = function(v) return tonumber(v) ~= nil end }  -- Custom validation function
   ```
​
#### Common Pattern Examples
​
1. **Balance Action Handler**
​
   ```lua
   { Action = "Balance" }  -- Match messages with Action = "Balance"
   ```
​
2. **Numeric Quantity Handler**
   ```lua
   { Quantity = "%d+" }  -- Match messages where Quantity is a number
   ```
​
### Default Action Handlers (AOS 2.0+)
​
AOS 2.0 introduces simplified syntax for Action-based handlers. Instead of writing explicit pattern functions, you can use these shorthand forms:
​
```lua
-- Traditional syntax
Handlers.add("Get-Balance", function (msg) return msg.Action == "Balance", doBalance)
​
-- Simplified syntax options:
Handlers.add("Balance", "Balance", doBalance)  -- Explicit action matching
Handlers.add("Balance", doBalance)             -- Implicit action matching
```
​
### Resolvers
​
Resolvers are special tables that can be used as the third argument in `Handlers.add()` to enable conditional execution of functions based on additional pattern matching. Each key in a resolver table is a pattern matching table, and its corresponding value is a function that executes when that pattern matches.
​
```lua
Handlers.add("Update",
  {
    [{ Status = "Ready" }] = function (msg) print("Ready") end,
    [{ Status = "Pending" }] = function (msg) print("Pending") end,
    [{ Status = "Failed" }] = function (msg) print("Failed") end
  }
)
```
​
This structure allows developers to create switch/case-like statements where different functions are triggered based on which pattern matches the incoming message. Resolvers are particularly useful when you need to handle a group of related messages differently based on additional criteria.
​
## Module Structure
​
- `Handlers._version`: String representing the version of the Handlers library.
- `Handlers.list`: Table storing the list of registered handlers.
​
## Common Handler Function Parameters
​
| Parameter            | Type                             | Description                                                                                                                                                                                                                                                                                                                                                                                                             |
| -------------------- | -------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `name`               | `string`                         | The identifier of the handler item in the handlers list.                                                                                                                                                                                                                                                                                                                                                                |
| `pattern`            | `table` or `function`            | Specifies how to match messages. As a table, defines required message tags with string values (e.g. `{ Action = "Balance", Recipient = "_" }` requires an "Action" tag with string value "Balance" and any string "Recipient" tag value). As a function, takes a message DataItem and returns: "true" (invoke handler and exit pipeline), "false" (skip handler), or "continue" (invoke handler and continue pipeline). |
| `handler`            | (Resolver) `table` or `function` | Either a resolver table containing pattern-function pairs for conditional execution, or a single function that processes the message. When using a resolver table, each key is a pattern matching table and its value is the function to execute when that pattern matches. When using a function, it takes the message DataItem as an argument and executes business logic.                                            |
| `maxRuns` (optional) | number                           | As of 0.0.5, each handler function takes an optional function to define the amount of times the handler should match before it is removed. The default is infinity.                                                                                                                                                                                                                                                     |
​
## Functions
​
### `Handlers.add(name, pattern, handler)`
​
Adds a new handler or updates an existing handler by name
​
### `Handlers.append(name, pattern, handler)`
​
Appends a new handler to the end of the handlers list.
​
### `Handlers.once(name, pattern, handler)`
​
Only runs once when the pattern is matched. Equivalent to setting `maxRuns = 1`.
​
### `Handlers.prepend(name, pattern, handler)`
​
Prepends a new handler to the beginning of the handlers list.
​
### `Handlers.before(handleName)`
​
Returns an object that allows adding a new handler before a specified handler.
​
### `Handlers.after(handleName)`
​
Returns an object that allows adding a new handler after a specified handler.
​
### `Handlers.remove(name)`
​
Removes a handler from the handlers list by name.
​
## Handler Execution Notes
​
### Execution Order
​
- Handlers are executed in the order they appear in `Handlers.list`.
- When a message arrives, each handler's pattern function is called sequentially to determine if it should process the message.
​
### Pattern Function Return Values
​
Pattern functions determine the message handling flow based on their return values:
​
1. **Skip Handler (No Match)**
​
   - **Return**: `0`, `false`, or any string except "continue" or "break"
   - **Effect**: Skips current handler and proceeds to the next one in the list
​
2. **Handle and Continue**
​
   - **Return**: `1` or `"continue"`
   - **Effect**: Processes the message and continues checking subsequent handlers
   - **Use Case**: Ideal for handlers that should always execute (e.g., logging)
​
3. **Handle and Stop**
   - **Return**: `-1`, `true`, or `"break"`
   - **Effect**: Processes the message and stops checking further handlers
   - **Use Case**: Most common scenario where a handler exclusively handles its matched message
​
### Practical Examples
​
- **Logging Handler**: Place at the start of the list and return `"continue"` to log all messages while allowing other handlers to process them.
- **Specific Message Handler**: Return `"break"` to handle matched messages exclusively and prevent further processing by other handlers.
​
## Handlers.utils
​
The `Handlers.utils` module provides two functions that are common matching patterns and one function that is a common handle function.
​
- `hasMatchingData(data: string)`
- `hasMatchingTag(name: string, value: string)`
- `reply(text: string)`
​
### `Handlers.utils.hasMatchingData(data: string)`
​
- This helper function returns a pattern matching function that takes a message as input. The returned function checks if the message's `Data` field contains the specified string. You can use this helper directly as the pattern argument when adding a new handler.
​
  ```lua
  Handlers.add("ping",
      Handlers.utils.hasMatchingData("ping"),
      ...
  )
  ```
​
### `Handlers.utils.hasMatchingTag(name: string, value: string)`
​
- This helper function returns a pattern matching function that takes a message as input. The returned function checks if the message has a tag with the specified `name` and `value`. If they match exactly, the pattern returns true and the handler function will be invoked. This helper can be used directly as the pattern argument when adding a new handler.
​
  ```lua
  Handlers.add("ping",
      Handlers.utils.hasMatchingTag("Action", "Ping"),
      ...
  )
  ```
​
### `Handlers.utils.reply(text: string)`
​
- This helper is a simple handle function, it basically places the text value in to the `Data` property of the outbound message.
​
  ```lua
  Handlers.add("ping",
      Handlers.utils.hasMatchingData("ping"),
      Handlers.utils.reply("pong")
  )
  ```
​
## Example Handlers
​
### Pattern Matching Table
​
```lua
Handlers.add("Ping",    -- Name of the handler
  { Action = "Ping" },  -- Matches messages with Action = "Ping" tag
  function(msg)         -- Business logic to execute on Message
    print("ping")
    msg.reply({ Data = "pong" })
  end
)
```
​
### Resolver Table Handler
​
```lua
Handlers.add("Foobarbaz",  -- Name of the handler
  { Action = "Speak" },    -- Matches messages with Action = "Speak" tag
  {
    -- Resolver with pattern-function pairs
    [{ Status = "foo" }] = function(msg) print("foo") end,
    [{ Status = "bar" }] = function(msg) print("bar") end,
    [{ Status = "baz" }] = function(msg) print("baz") end
  }
)
```
​
### Function-Based Pattern Matching & Handler
​
```lua
Handlers.add("Example",    -- Name of the handler
  function(msg)           -- Pattern function matches messages with Action = "Speak" tag
    return msg.Action == "Speak"
  end,
  function(msg)           -- Handler function that executes business logic
    print(msg.Status)
  end
)
```
​
## References
​
Source: https://cookbook_ao.arweave.net/references/index.html
​
This section provides detailed technical references for AO components, languages, and tools. Use these resources to find specific information when implementing your AO projects.
​
## Programming Languages
​
Resources for the programming languages used in AO:
​
- [Lua](./lua) - Reference for the Lua programming language, the primary language used in AO
- [WebAssembly (WASM)](./wasm) - Information about using WebAssembly modules in AO
- [Lua Optimization](./lua-optimization) - Techniques and best practices for optimizing Lua code in AO
​
## AO API Reference
​
Documentation for AO's core APIs and functionality:
​
- [AO Core](./ao) - Core `ao` module and API reference
- [Messaging](./messaging) - Comprehensive guide to the AO messaging system patterns
- [Handlers](./handlers) - Reference for event handlers and message processing
- [Token](./token) - Information about token creation and management
- [Arweave Data](./data) - Guide to data handling and storage in AO
- [Cron](./cron) - Documentation for scheduling and managing timed events
​
## Development Environment
​
Tools and setup for AO development:
​
- [Editor Setup](./editor-setup) - Guide to setting up your development environment for AO
- [BetterIDEa](./betteridea/index) - The ultimate native web IDE for AO development
​
## Community Resources
​
Connect with the AO community:
​
- [Community Resources](./community) - Information about AO community resources and support
​
## Navigation
​
Use the sidebar to navigate between reference topics. References are organized by category to help you find the information you need quickly.
​
## Lua Optimization Guide for AO Platform
​
Source: https://cookbook_ao.arweave.net/references/lua-optimization.html
​
This guide provides practical tips for writing efficient, fast, and performant Lua code for on-chain programs on the AO platform.
​
## Table Operations
​
### Appending Elements
​
```lua
-- ❌ Inefficient: Up to 7x slower in tight loops
table.insert(t, v)
​
-- ✅ Efficient: Direct indexing is ~2x faster
t[#t + 1] = v
```
​
### Removing Elements
​
```lua
-- ❌ Inefficient: Shifts all elements left
table.remove(t, 1)
​
-- ✅ Efficient: Remove from end
local x = t[#t]
t[#t] = nil
```
​
## Variable Access
​
### Local Variables
​
```lua
-- ❌ Inefficient: Global lookup each time
for i = 1, 1000 do
  math.sin(i)
end
​
-- ✅ Efficient: Cache the function
local sin = math.sin
for i = 1, 1000 do
  sin(i)  -- ~30% faster in loops
end
```
​
### Upvalues
​
```lua
-- ❌ Inefficient: Config lookup on each call
Handlers.add("ValidateGameToken",
  function(msg)
    local config = ao.config
    validateToken(msg, config)
  end
)
​
-- ✅ Efficient: Cache config as upvalue
local config = ao.config
Handlers.add("ValidateGameToken",
  function(msg)
    validateToken(msg, config)
  end
)
```
​
## String Operations
​
### String Concatenation
​
```lua
-- ❌ Inefficient: Creates many intermediate strings
local str = ""
for i = 1, N do
  str = str .. "line" .. i
end
​
-- ✅ Efficient: Single concatenation at end
local lines = {}
for i = 1, N do
  lines[i] = "line" .. i
end
local str = table.concat(lines)
```
​
### Pattern Matching
​
```lua
-- ❌ Inefficient: Recompiles pattern on each iteration
for line in io.lines() do
  if line:match("^%s*(%w+)%s*=%s*(%w+)") then
    -- Process match
  end
end
​
-- ✅ Efficient: Compile pattern once
local pattern = "^%s*(%w+)%s*=%s*(%w+)"
for line in io.lines() do
  if line:match(pattern) then
    -- Process match
  end
end
```
​
## Memory Management
​
### Table Reuse
​
```lua
-- ❌ Inefficient: Creates new table on each call
Handlers.add("ComputeGameResults",
  function(msg)
    local results = {}
    -- Fill results
    return results
  end
)
​
-- ✅ Efficient: Reuse and clear table
local results = {}
Handlers.add("ComputeGameResults",
  function(msg)
    for k in pairs(results) do results[k] = nil end
    -- Fill results
    return results
  end
)
```
​
### Minimize Garbage Creation
​
```lua
-- ❌ Inefficient: Creates new response table on every transfer
local function createTransferResponse(sender, recipient, amount)
  return {
    from = sender,
    to = recipient,
    quantity = amount,
    success = true,
    newBalance = Balances[sender],
    tags = {
      Action = "Transfer-Complete",
      Type = "Token"
    }
  }
end
​
-- ✅ Efficient: Reuse template table
local transferResponse = {
  from = nil,
  to = nil,
  quantity = 0,
  success = false,
  newBalance = 0,
  tags = {
    Action = "Transfer-Complete",
    Type = "Token"
  }
}
​
local function createTransferResponse(sender, recipient, amount)
  transferResponse.from = sender
  transferResponse.to = recipient
  transferResponse.quantity = amount
  transferResponse.success = true
  transferResponse.newBalance = Balances[sender]
  return transferResponse
end
```
​
## Blockchain-Specific Optimizations
​
### State Management
​
```lua
-- ❌ Inefficient: Multiple separate state updates
for _, item in ipairs(items) do
  ao.send({ Target = "processID", Action = "Update", Data = item })
end
​
-- ✅ Efficient: Batch updates into single message
local updates = {}
for _, item in ipairs(items) do
  table.insert(updates, item)
end
ao.send({ Target = "processID", Action = "BatchUpdate", Data = updates })
```
​
## Additional Resources
​
- [Lua Performance Guide](https://www.lua.org/gems/sample.pdf)
- Special thanks to [@allquantor](https://x.com/allquantor/status/1887370546259644728?s=12) for sharing optimization tips
​
## Meet Lua
​
Source: https://cookbook_ao.arweave.net/references/lua.html
​
## Understanding Lua
​
- **Background**: Lua is a lightweight, high-level, multi-paradigm programming language designed primarily for embedded systems and clients. It's known for its efficiency, simplicity, and flexibility.
- **Key Features**: Lua offers powerful data description constructs, dynamic typing, efficient memory management, and good support for object-oriented programming.
​
## Setting Up
​
1. **Installation**: Visit [Lua's official website](http://www.lua.org/download.html) to download and install Lua.
2. **Environment**: You can use a simple text editor and command line, or an IDE like ZeroBrane Studio or Eclipse with a Lua plugin.
​
## Basic Syntax and Concepts (in aos)
​
- **Hello World**:
  ```lua
  "Hello, World!"
  ```
- **Variables and Types**: Lua is dynamically typed. Basic types include `nil`, `boolean`, `number`, `string`, `function`, `userdata`, `thread`, and `table`.
- **Control Structures**: Includes `if`, `while`, `repeat...until`, and `for`.
- **Functions**: First-class citizens in Lua, supporting closures and higher-order functions.
- **Tables**: The only data structuring mechanism in Lua, which can be used to represent arrays, sets, records, etc.
​
## Hands-On Practice

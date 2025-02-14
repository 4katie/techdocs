---
id: ergo-cli
title: Ergo CLI
---

Install the `@accordproject/ergo-cli` npm package to access the Ergo command line interface (CLI). After installation you can use the ergo command and its sub-commands as described below.

To install the Ergo CLI:
```
npm install -g @accordproject/ergo-cli
```

This will install `ergo`, to compile and run contracts locally on your machine, and `ergotop`, which is a _read-eval-print-loop_ utility to write Ergo interactively.

> In ergo-cli `0.20` release, `ergoc`, the Ergo compiler, and `ergorun`, used to run contracts locally on your machine, which were previously part of the `ergo-cli` npm package, have been merged into `ergo` commands.
>
> For more information about the changes that were made for the `0.20` release, please refer to our [Migrating from 0.13.\*](0.13to0.20.html) guide.


## Ergo

### Usage

```md
ergo <command>

Commands:
  ergo draft       create a contract text from data
  ergo trigger     send a request to the contract
  ergo invoke      invoke a clause of the contract
  ergo initialize  initialize the state for a contract
  ergo compile     compile a contract

Options:
  --help         Show help                                             [boolean]
  --version      Show version number                                   [boolean]
  --verbose, -v                                                 [default: false]
```

### ergo draft

`ergo draft` allows you to create a contract text from data. Note that `--data` is a required field.

```md
## ergo draft
Usage: ergo draft --data [file] [ctos] [ergos]

Options:
  --help           Show help                                           [boolean]
  --version        Show version number                                 [boolean]
  --verbose, -v                                                 [default: false]
  --data           path to the contract data                          [required]
  --currentTime    the current time
                                 [string] [default: "2019-10-30T12:03:24+00:00"]
  --wrapVariables  wrap variables in curly braces     [boolean] [default: false]
  --template       path to the template directory       [string] [default: null]
  --warnings       print warnings                     [boolean] [default: false]
```

#### Example

For example, using the `draft` command for the [Volume Discount example](https://github.com/accordproject/ergo/tree/master/examples/volumediscount) in the [Ergo Directory](https://github.com/accordproject/ergo):

```md
ergo draft --template ./examples/volumediscount --data ./examples/volumediscount/data.json
```

returns:

```md
info: Volume-Based Card Acceptance Agreement [Abbreviated]

This Agreement is by and between Card, Inc., a New York corporation, and you, the Merchant.
...
b) Discount. The Discount is determined according to the following table:

| Annual Dollar Volume      | Discount             |
| Less than $1.0 million      | 3.0%                |
| $1.0 million to $10.0 million | 2.9%                |
| Greater than $10.0 million  | 2.8%                |
```

The variables specified in the `data.json` file (such as `firstVolume`: 1, `firstRate`: 3) are incorporated into the text of the contract (which can be found in the `./examples/volumediscount` template directory).

> In general, the `data.json` files aren’t part of the template archive from the [Cicero Template Library](https://github.com/accordproject/cicero-template-library/tree/js-release-0.20/src). It is possible to generate one with [cicero parse](cicero-cli#cicero-parse).


## ergo trigger
`ergo trigger` allows you to send a request to the contract.

```md
Usage: ergo trigger --data [file] --state [file] --request [file] [cto files]
[ergo files]

Options:
  --help         Show help                                             [boolean]
  --version      Show version number                                   [boolean]
  --verbose, -v                                                 [default: false]
  --data         path to the contract data                            [required]
  --state        path to the state data                 [string] [default: null]
  --currentTime  the current time[string] [default: "2019-10-30T20:18:28+00:00"]
  --request      path to the request data                     [array] [required]
  --template     path to the template directory         [string] [default: null]
  --warnings     print warnings                       [boolean] [default: false]
```

### Example

For example, using the `trigger` command for the [Volume Discount example](https://github.com/accordproject/ergo/tree/master/examples/volumediscount) in the [Ergo Directory](https://github.com/accordproject/ergo):

```md
ergo trigger --template ./examples/volumediscount --data ./examples/volumediscount/data.json --request ./examples/volumediscount/request.json --state ./examples/volumediscount/state.json
```

returns:

```json
{
  "clause": "orgXaccordprojectXvolumediscountXVolumeDiscount",
  "request": {
    "$class": "org.accordproject.volumediscount.VolumeDiscountRequest",
    "netAnnualChargeVolume": 10.4
  },
  "response": {
    "$class": "org.accordproject.volumediscount.VolumeDiscountResponse",
    "discountRate": 2.8,
    "transactionId": "3024ea58-ad82-4c83-87b1-d8fea8436d49",
    "timestamp": "2019-10-31T11:17:36.038Z"
  },
  "state": {
    "$class": "org.accordproject.cicero.contract.AccordContractState",
    "stateId": "1"
  },
  "emit": []
}
```

As the `request` was sent for an annual charge volume of 10.4, which falls into the third discount rate category (as specified in the `data.json` file), the `response` returns with a discount rate of 2.8%.


## ergo invoke
`ergo invoke` allows you to invoke a specific clause of the contract. The main difference between `ergo invoke` and `ergo trigger` is that `ergo invoke` sends data to a specific clause, whereas `ergo trigger` lets the contract choose which clause to invoke. This is why `--clauseName` (the name of the contract you want to execute) is a required field for `ergo invoke`.

You need to pass the CTO and Ergo files (`--template`), the name of the contract that you want to execute (`--clauseName`), and JSON files for: the contract data (`--data`), the contract parameters (`--params`), the current state of the contract (`--state`), and the request.

If contract invocation is successful, `ergorun` will print out the response, the new contract state and any emitted events.

```md
Usage: ergo invoke --data [file] --state [file] --params [file] [cto files]
[ergo files]

Options:
  --help         Show help                                             [boolean]
  --version      Show version number                                   [boolean]
  --verbose, -v                                                 [default: false]
  --clauseName   the name of the clause to invoke                     [required]
  --data         path to the contract data                            [required]
  --state        path to the state data                      [string] [required]
  --currentTime  the current time[string] [default: "2019-10-30T20:18:57+00:00"]
  --params       path to the parameters        [string] [required] [default: {}]
  --template     path to the template directory         [string] [default: null]
  --warnings     print warnings                       [boolean] [default: false]
```

### Example

For example, using the `invoke` command for the [Volume Discount example](https://github.com/accordproject/ergo/tree/master/examples/volumediscount) in the [Ergo Directory](https://github.com/accordproject/ergo):

```md
ergo invoke --template ./examples/volumediscount --clauseName volumediscount --data ./examples/volumediscount/data.json --params ./examples/volumediscount/params.json --state ./examples/volumediscount/state.json
```

returns:

```json
info:
{
  "clause": "orgXaccordprojectXvolumediscountXVolumeDiscount",
  "params": {
    "request": {
      "$class": "org.accordproject.volumediscount.VolumeDiscountRequest",
      "netAnnualChargeVolume": 10.4
    }
  },
  "response": {
    "$class": "org.accordproject.volumediscount.VolumeDiscountResponse",
    "discountRate": 2.8,
    "transactionId": "2a979363-56bc-48ff-a6b4-49994a848a0c",
    "timestamp": "2019-10-31T11:22:37.368Z"
  },
  "state": {
    "$class": "org.accordproject.cicero.contract.AccordContractState",
    "stateId": "1"
  },
  "emit": []
}
```

Although this looks very similar to what `ergo trigger` returns, it is important to note that `--clauseName volumediscount` was specifically invoked.

## ergo initialize
`ergo initialize` allows you to obtain the initial state of the contract. This is the state of the contract without requests or responses.

```md
Usage: ergo intialize --data [file] --params [file] [cto files] [ergo files]

Options:
  --help         Show help                                             [boolean]
  --version      Show version number                                   [boolean]
  --verbose, -v                                                 [default: false]
  --data         path to the contract data                            [required]
  --currentTime  the current time[string] [default: "2019-10-30T20:19:24+00:00"]
  --params       path to the parameters                 [string] [default: null]
  --template     path to the template directory         [string] [default: null]
  --warnings     print warnings                       [boolean] [default: false]

```

### Example

For example, using the `initialize` command for the [Volume Discount example](https://github.com/accordproject/ergo/tree/master/examples/volumediscount) in the [Ergo Directory](https://github.com/accordproject/ergo):

```md
ergo initialize --template ./examples/volumediscount --data ./examples/volumediscount/data.json
```

returns:

```json
info:
{
  "clause": "orgXaccordprojectXvolumediscountXVolumeDiscount",
  "params": {
  },
  "response": null,
  "state": {
    "$class": "org.accordproject.cicero.contract.AccordContractState",
    "stateId": "org.accordproject.cicero.contract.AccordContractState#1"
  },
  "emit": []
}
```

## ergo compile
`ergo compile` takes your input models (`.cto` files) and input contracts (`.ergo` files) and allows you to compile a contract into a target platform. By default, Ergo compiles to JavaScript (ES6 compliant) for execution.


```md
Usage: ergo compile --target [lang] --link --monitor --warnings [cto files]
[ergo files]

Options:
  --help         Show help                                             [boolean]
  --version      Show version number                                   [boolean]
  --verbose, -v                                                 [default: false]
  --target       Target platform (available: es5,es6,cicero,java)
                                                       [string] [default: "es6"]
  --link         Link the Ergo runtime with the target code (es5,es6,cicero
                 only)                                [boolean] [default: false]
  --monitor      Produce compilation time information [boolean] [default: false]
  --warnings     print warnings                       [boolean] [default: false]
```

### Example
For example, using the `compile` command on the [Volume Discount example](https://github.com/accordproject/ergo/tree/master/examples/volumediscount) in the [Ergo Directory](https://github.com/accordproject/ergo):

```md
ergo compile ./examples/volumediscount/model/model.cto ./examples/volumediscount/logic/logic.ergo
```

returns:

```md
Compiling Ergo './examples/volumediscount/logic/logic.ergo' --  './examples/volumediscount/logic/logic.js'
```

Which means a new `logic.js` file is located in the `./examples/volumediscount/logic` directory.

To compile the contract to Javascript and **link the Ergo runtime for execution**:
```md
ergo compile ./examples/volumediscount/model/model.cto ./examples/volumediscount/logic/logic.ergo --link
```

returns:

```md
Compiling Ergo './examples/volumediscount/logic/logic.ergo' --  './examples/volumediscount/logic/logic.js'
```



## ergotop (REPL)

### Starting the REPL

`ergotop` is a convenient tool to try-out Ergo contracts in an interactive way. You can write commands, or expressions and see the result. It is often called the Ergo REPL, for _read-eval-print-loop_, since it literally: reads your input Ergo from the command-line, evaluates it, prints the result and loops back to read your next input.

To start the REPL:

```
$ ergotop
Welcome to ERGOTOP version 0.20.0
ergo$
```

It should print the prompt `ergo$` which indicates it is ready to read your command. For instance:

```ergo
    ergo$ return 42
    Response. 42 : Integer
```

`ergotop` prints back both the resulting value and its type. You can then keep typing commands:

```ergo
    ergo$ return "hello " ++ "world!"
    Response. "hello world!" : String
    ergo$ define constant pi = 3.14
    ergo$ return pi ^ 2.0
    Response. 9.8596 : Double
```

If your expression is not valid, or not well-typed, it will return an error:

```ergo
    ergo$ return if true else "hello"
    Parse error (at line 1 col 15).
    return if true else "hello"
                   ^^^^        
    ergo$ return if "hello" then 1 else 2
    Type error (at line 1 col 10). 'if' condition not boolean.
    return if "hello" then 1 else 2
              ^^^^^^^
```

If what you are trying to write is too long to fit on one line, you can use `\` to go to a new line:

```ergo
    ergo$ define function squares(l:Double[]) : Double[] { \
      ...   return \
      ...     foreach x in l return x * x \
      ... }
    ergo$ return squares([2.3,4.5,6.7])
    Response. [5.29, 20.25, 44.89] : Double[]
```

### Loading files

You can load CTO and Ergo files to use in your REPL session. Once the REPL is launched you will have to import the corresponding namespace. For instance, if you want to use the `compoundInterestMultiple` function defined in the `./examples/promissory-note/money.ergo` file, you can do it as follows:

```ergo
$ ergotop ./examples/promissory-note/logic/money.ergo
Welcome to ERGOTOP version 0.20.0

ergo$ import org.accordproject.ergo.money.*
ergo$ return compoundInterestMultiple(0.035, 100)
Response. 1.00946960405 : Double
```

### Calling contracts

To call a contract, you first needs to _instantiate_ it, which means setting its parameters and initializing its state. You can do this by using the `set contract` and `call init` commands respectively. Here is an example using the `volumediscount` template:

```ergo
$ ergotop ./examples/volumediscount/model/model.cto ./examples/volumediscount/logic/logic.ergo
ergo$ import org.accordproject.cicero.contract.*
ergo$ import org.accordproject.cicero.runtime.*
ergo$ import org.accordproject.volumediscount.*
ergo$ set contract VolumeDiscount over VolumeDiscountContract {firstVolume: 1.0, secondVolume: 10.0, firstRate: 3.0, secondRate: 2.9, thirdRate: 2.8, contractId: "0", parties: none }
ergo$ call init()
Response. unit : Unit
State. AccordContractState{stateId: "org.accordproject.cicero.contract.AccordContractState#1"} : AccordContractState
```

You can then invoke clauses of the contract:

```ergo
ergo$ call volumediscount(VolumeDiscountRequest{ netAnnualChargeVolume : 0.1 })
Response. VolumeDiscountResponse{discountRate: 3.0} : VolumeDiscountResponse
ergo$ call volumediscount(VolumeDiscountRequest{ netAnnualChargeVolume : 10.5 })
Response. VolumeDiscountResponse{discountRate: 2.8} : VolumeDiscountResponse
```

You can also invoke the contract without explicitly naming the clause by sending a request. The Ergo engine dispatches that request to the first clause which can handle it:
```ergo
ergo$ send VolumeDiscountRequest{ netAnnualChargeVolume : 0.1 }
Response. VolumeDiscountResponse{discountRate: 3.0} : VolumeDiscountResponse
ergo$ send VolumeDiscountRequest{ netAnnualChargeVolume : 10.5 }
Response. VolumeDiscountResponse{discountRate: 2.8} : VolumeDiscountResponse
```

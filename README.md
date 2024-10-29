# Functional web app development with F# and Fable

[![License: CC BY 4.0](https://licensebuttons.net/l/by/4.0/80x15.png)](https://creativecommons.org/licenses/by/4.0/) Copyright (C) 2024 by Alceste Scalas

_Presented on 28 October 2024 at the
[Mødegruppe for F#unctionelle Københavnere](https://www.meetup.com/moedegruppefunktionellekoebenhavnere)._

This tutorial is a gentle introduction to web application development using
Fable.

[Fable](https://fable.io/) compiles F# code into JavaScript, and has a rich
ecosystem including various libraries and DSLs for the type-safe creation of
HTML and reactive user interfaces. This tutorial focuses on
[Elmish](https://elmish.github.io/) and
[Feliz](https://zaid-ajaj.github.io/Feliz/), and demonstrates how they enable
the development of purely-functional web applications with reactive UIs. This
tutorial also touches upon the creation of F# projects targeting both .NET and
JavaScript.

## System requirements

- .NET ≥ 8.0
- VS Code with Ionide extension
- npm ≥ 9.2.0
- Node.js ≥ 22.10

## Basic setup

### Creating a command-line F# application

Create new F# project targeting the console.

```none
$ dotnet new console -lang F# -n HelloApp -o app
```

Move into the `app/` directory.

```none
$ cd app/
```

Open and modify the file `Program.fs` if you like.

Compile and run `Program.fs` using .NET.

```none
$ dotnet run
```

### Installing Fable and compiling the app into JavaScript

Install the Fable compiler.

```none
$ dotnet new tool-manifest
$ dotnet tool install fable
```

Use Fable to compile everything in the current directory --- in particular,
compile `Program.fs` into `Program.fs.js`.

```none
$ dotnet fable
```

Run `Program.fs.js` using Node.js.

```none
$ node Program.fs.js
```

### Adding more files to the project

Add the file `Core.fs` with the core logic of the application. **IMPORTANT:**
add the file via VS Code + Ionide --- otherwise, you will need to manually edit
`HelloApp.fsproj`, too.

```fsharp
module Core

let increment (n: int): int =
    n + 1

let decrement (n: int): int =
    n - 1
```

Edit `Program.fs` to use `increment` and `decrement`, e.g:

```fsharp
open Core

printfn $"increment (decrement 1) = %d{increment (decrement 1)}"
```

Recompile and re-run with both .NET and Fable + Node.js. Observe that now Fable
has also compiled `Core.fs` into `Core.fs.js`.

## Adding a test project

Go back to the parent of the `app/` directory.

Create a new F# testing project.

```none
$ dotnet new mstest -lang F# -n HelloTest -o tests
```

Move into the `tests/` directory.

```none
$ cd tests/
```

Add Expecto with FsCheck support, and its .NET TestSdk adapter.

```none
$ dotnet add package Expecto.FsCheck
$ dotnet add package YoloDev.Expecto.TestSdk
```

Add a reference to `../app/HelloApp.fsproj` into `HelloTest.fsproj`.

```none
$ dotnet add reference ../app/HelloApp.fsproj
```

Edit `Tests.fs` adding some test cases.

```fsharp
module HelloTest

open Expecto

[<Tests>]
let tests = testList "Property-based tests" [
    testProperty "Increment increases value" <| fun (n: int) ->
        Core.increment n > n
    testProperty "Decrement decreases value" <| fun (n: int) ->
        Core.decrement n < n
    testProperty "Inverse" <| fun (n: int) ->
        Core.increment (Core.decrement n) = n
]
```

## Putting together the app project and the testing project

Go back to the parent of both directories `app/` and `tests/`.

Create a solution file `HelloWorld.sln`, and add both sub-projects to it.

```none
$ dotnet new sln -n HelloWorld
$ dotnet sln add app/HelloApp.fsproj
$ dotnet sln add tests/HelloTest.fsproj
```

Then, open VS Code in the folder containing `HelloWorld.sln`, and observe that
both sub-projects are loaded.

## Running Fable-generated JavaScript in a web page

Go into the `app/` directory.

```none
$ cd app/
```

Create the file `index.html`.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Hello, World!</title>
  </head>
  <body>
    <div id="root"></div>
    <!-- Make sure to point to the correct location -->
    <script type="module" src="Program.fs.js"></script>
  </body>
</html>
```

Add the `Fable.Browser.Dom` package, to manipulate the DOM.

```none
$ dotnet add package Fable.Browser.Dom
```

Edit `Program.fs` and replace its content with some DOM manipulation.

```fsharp
open Core

Browser.Dom.document.getElementById("root").innerHTML <-
    $"<p>increment (decrement 1) = <strong>%d{increment (decrement 1)}</strong></p>"
```

Recompile all files using Fable.

```none
$ dotnet fable
```

To execute the resulting web page, we need to launch a web server (otherwise the
JavaScript file will be rejected with a
"[CORS request not HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS/Errors/CORSRequestNotHttp)" error). For instance, in the
folder containing the file `index.html`, we can try:

```fsharp
$ python3 -m http.server
```

## Integrating the Vite dev server & build tool

Create a minimal `package.json` file.

```json
{
  "name": "hello-app",
  "version": "0.0.1"
}
```

Install [Vite](https://vite.dev/) using `npm`, saving it in the project's
development dependencies (`devDependencies`).

```none
$ npm install --save-dev vite
```

Create a configuration file `vite.config.ts` to set up which files should be
watched.

```typescript
import { defineConfig } from 'vite'

// Documentation: https://vitejs.dev/config/
export default defineConfig({
    clearScreen: false,
    server: {
        watch: {
            ignored: [
                "*.fs" // Don't watch F# files
            ]
        }
    }
})
```

Start Fable in watch mode: recompile the `.fs` files when they are changed, and
then let Vite reload the `.js` files.

```none
$ dotnet fable watch --verbose --run npx vite
```

Open the Vite URL in a browser, edit `Program.fs`, and notice how the page is
automatically updated whenever a change is saved.

To build a production version of the web app, execute:

```none
$ vite build
```

## Integrating Femto

[Femto](https://github.com/Zaid-Ajaj/Femto) is a tool that simplifies the
management of `npm` packages used by Fable bindings. It will be handy in the
next section.

```none
$ dotnet tool install femto
```

## Integrating Feliz

[Feliz](https://zaid-ajaj.github.io/Feliz) provides a DSL for building
[React](https://react.dev/) applications in F# and Fable. We can install Feliz
together with its `npm` dependencies by running:

```none
$ dotnet femto install Feliz
```

Edit `Program.fs` and create a minimal web application with one React component.

```fsharp
open Core

open Feliz

[<ReactComponent>]
let reactComponent () =
    let (count, setCount) = React.useState(0)
    Html.div [
        Html.h1 count
        Html.button [
            prop.text "Increment"
            prop.onClick (fun _ -> setCount(increment count))
        ]
        Html.button [
            prop.text "Decrement"
            prop.onClick (fun _ -> setCount(decrement count))
        ]
    ]

let root = ReactDOM.createRoot(Browser.Dom.document.getElementById "root")
root.render(reactComponent())
```

## Integrating Elmish

[Elmish](https://elmish.github.io/) is a library for building F# applications
with [Elm](https://elm-lang.org/)-style "model view update" architecture.
[Elmish.React](https://elmish.github.io/react/) wires up the rendering of React
components to Elmish.

Install Elmish.React and its dependencies by executing:

```none
$ dotnet add package Fable.Elmish.React
```

Then, modify `Program.fs` to create a minimal Elmish web app, with React
components based on Feliz.

```fsharp
open Core

open Elmish
open Elmish.React
open Feliz

type Model = {
    Value: int
}

type Msg =
    | Increment
    | Decrement

let init (): Model =
    { Value = 0 }

let update (msg: Msg) (model: Model): Model =
    match msg with
    | Increment ->
        { model with Value = increment model.Value }
    | Decrement ->
        { model with Value = decrement model.Value }

let view (model: Model) (dispatch: Msg -> unit) =
    Html.div [
        Html.h1 model.Value
        Html.button [
            prop.text "Increment"
            prop.onClick (fun _ -> dispatch Increment)
        ]
        Html.button [
            prop.text "Decrement"
            prop.onClick (fun _ -> dispatch Decrement)
        ]
    ]

Program.mkSimple init update view
|> Program.withReactSynchronous "root"
|> Program.run
```

## JavaScript interoperability

The `Fable.Core.JsInterop` module is one of the
[Fable facilities for interoperating with JavaScript](https://fable.io/docs/javascript/features.html).
For instance, `jsOptions` creates a JavaScript object of a desired type and
allows for setting its fields in a type-safe way. It is used in the example
below, which extends the previous version of `Program.fs` with a long table and
a button to scroll to a desired row.

```fsharp
open Core

open Elmish
open Elmish.React
open Feliz

type Model = {
    Value: int
    Row: int
}

type Msg =
    | Increment
    | Decrement
    | SetRow of row: int

let init (): Model =
    { Value = 0
      Row = 0 }

let update (msg: Msg) (model: Model): Model =
    match msg with
    | Increment ->
        { model with Value = increment model.Value }
    | Decrement ->
        { model with Value = decrement model.Value }
    | SetRow row ->
        { model with Row = row }

let scrollToRow (row: int): unit =
    // https://developer.mozilla.org/en-US/docs/Web/API/Element/scrollIntoView
    let opt = Fable.Core.JsInterop.jsOptions<Browser.Types.ScrollIntoViewOptions>(fun o ->
        o.behavior <- Browser.Types.ScrollBehavior.Smooth
        o.block <- Browser.Types.ScrollAlignment.Nearest
        o.``inline`` <- Browser.Types.ScrollAlignment.Nearest
    )
    Browser.Dom.document.getElementById($"row-%d{row}").scrollIntoView opt

let view (model: Model) (dispatch: Msg -> unit) =
    Html.div [
        Html.h1 model.Value
        Html.button [
            prop.text "Increment"
            prop.onClick (fun _ -> dispatch Increment)
        ]
        Html.button [
            prop.text "Decrement"
            prop.onClick (fun _ -> dispatch Decrement)
        ]
        Html.input [
            prop.type' "text"
            prop.onChange (fun (value: string) ->
                                let isInt, v = System.Int32.TryParse value
                                if isInt then dispatch (SetRow v)
                                         else dispatch (SetRow 0))
            prop.value model.Row
        ]
        Html.button [
            prop.text "<- Scroll to this row"
            prop.onClick (fun _ -> if model.Row <= 500 then scrollToRow model.Row)
        ]
        Html.table [
            Html.tbody (
                List.map (fun (i: int) ->
                    Html.tr [
                        prop.id $"row-%d{i}"
                        prop.children [
                            Html.td [
                                prop.text $"Row %d{i}"
                            ]
                        ]
                    ]
                ) [0 .. 500]
            )
        ]
    ]

Program.mkSimple init update view
|> Program.withReactSynchronous "root"
|> Program.run
```
---
title: Subsecond Documentation

language_tabs: # must be one of https://git.io/vQNgJ
  - ts

toc_footers:
  - <a href='https://playground.subsecond.app'>Try it out with Subsecond Playground</a>
  - <a href='https://github.com/slatedocs/slate'>Documentation Powered by Slate</a>

includes:

search: true

code_clipboard: true

meta:
  - name: description
    content: Documentation for Subsecond, a structured code modification library.
---

# Getting Started

## 1. Start a new project

> A new directory

```bash
> mkdir subsecond-example
> cd subsecond-example
```

> Initialize an npm project

```bash
> npm init
```

## 2. Install the library

> npm

```bash
> npm install subsecond
```

> yarn

```bash
> yarn add subsecond
```

## 3. Start writing code

> index.js

```ts
import S from "subsecond";

S.load({ "example.js": "console.log('Hello World!');" });

S("Identifier.log").text("error");

console.log(S.print());
```

> Running this with `node index.js` results in:

```json
{
  "example.js": "console.error('Hello World');"
}
```

All Subsecond scripts follow the same basic structure:

1. `S.load()` all of the code we want to modify.
2. `S()` selectors and modification functions like `.text()` and `.before()`.
3. `S.print()` at the end to output our finished product.

## A full example

```ts
import S from "subsecond";

const registerCarJS = `
registerCar("Honda", "CR-V", "silver", 2004);
`;

S.load({ "registerCar.js": registerCarJS });

S("CallExpression.registerCar").each((registerCar) => {
  const carParams = registerCar.children().map((child) => child.text());

  // carParams[0] is the function name itself "registerCar", so skip it.
  registerCar.text(`
    registerCar({
      make: ${carParams[1]},
      model: ${carParams[2]},
      color: ${carParams[3]},
      year: ${carParams[4]},
    })
  `);
});

console.log(S.print()["registerCar.js"]);
```

> Running this subsecond script results in the following output:

```ts
registerCar({
  make: "Honda",
  model: "CR-V",
  color: "silver",
  year: 2004,
});
```

Let's say you have a function `registerCar` that takes 4 parameters:

- Make
- Model
- Color
- Year

It's been decided to switch this function to take a single parameter which is an object with each of the four previous parameters as keys.

`registerCar` could potentially be in your codebase hundreds of times. Manually would be annoying, regex get way too complicated.

This is how Subsecond can be used to fix this problem.

# Selectors

Currently support for selectors is limited, but it will grow soon.

Use https://astexplorer.net/ set to `@typescript-eslint/parser` to figure out what everything is named.

## The empty selector

```ts
S();
```

Selects the root nodes of all files currently loaded.

## Just ESTree selector

```ts
S("FunctionDeclaration");
```

Selects all nodes of this type across all files

## ESTree with name

```ts
S("FunctionDeclaration.testFunction");
```

Selects all nodes with both the type and name

## Comma Separator

```ts
S("FunctionDeclaration, CallExpression");
```

Select all `FunctionDeclaration` nodes and `CallExpression` nodes.

## Space Separator

```ts
S("ExpressionStatement Literal");
```

Select only `Literal` nodes that have some ancestor `ExpressionStatement` node.

# Utilities

## S.load(files: Object): void

> Load in a single line of code as `index.ts`

```ts
S.load({ "index.ts": "console.log('Hello World!');" });
```

> More commonly, use node's `fs` library to load in code files.

```ts
import { readdirSync, readFileSync } from "fs";

const files = Object.fromEntries(
  readdirSync("../src")
    .filter((fileName) => fileName.endsWith(".ts") || fileName.endsWith(".tsx"))
    .map((fileName) => [fileName, readFileSync(fileName, "utf8")])
);

S.load(files);
```

Load typescript files into the Subsecond object.

Multiple calls to `S.load()` will add additional files without deleting any already added files. Files the the same name be overwritten with subsequent calls to `S.load()`.

| Parameter | Description                                                          |
| --------- | -------------------------------------------------------------------- |
| `files`   | `Object` with file names as keys and code as a string as the values. |
| Output    | `void`                                                               |

## S.print(): Object

```ts
Subsecond.print();
```

```json
{ "index.ts": "console.log('Hello World!')" }
```

returns modified files in the same

| Parameter | Description                                                            |
| --------- | ---------------------------------------------------------------------- |
| N\A       | This function takes no parameters.                                     |
| Output    | An Object with keys and values arranged in the same way as `S.load()`. |

# Subsecond Functions

## S(selector, context): Subsecond

Select a list of nodes using the selector syntax seen above in the Selector section.

| Parameter  | Description                                                                                     |
| ---------- | ----------------------------------------------------------------------------------------------- |
| `selector` | `string` using Subsecond's selector syntax                                                      |
| `context`  | Optional. A Subsecond object to use instead of the default `files` to apply the selector to.    |
| Output     | A Subsecond object containing all of the elements selected by by the selector and context pair. |

## type(): string

Returns the [estree spec](https://github.com/estree/estree) type of currently selected node.

| Parameter | Description                                     |
| --------- | ----------------------------------------------- |
| N\A       | This function takes no parameters.              |
| Output    | The estree type as a string. (ex. "Identifier") |

## text(): string

```ts
S.load({ "index.ts": 'console.log("Hello World!");' });

console.log(S("CallExpression").text());
```

> Running this would produce the following result:

```ts
console.log("Hello World!");
```

Get the full text of the selected elements as a string.

If multiple elements are selected, they are all concatenated together and returned as a string.

| Parameter | Description                                |
| --------- | ------------------------------------------ |
| N\A       | This function takes no parameters.         |
| Output    | A string of code of the selected elements. |

## text(newText: string): Subsecond

```ts
S("CallExpression").text("console.log('something else!')");
```

Replace each of the currently selected elements with `newText`.

| Parameter | Description                                                           |
| --------- | --------------------------------------------------------------------- |
| `newText` | The `string` to replace existing elements with.                       |
| Output    | A Subsecond `Object` that contains all of the newly created elements. |

## text(callback: (oldText: string) => string): Subsecond

| Parameter  | Description                                                           |
| ---------- | --------------------------------------------------------------------- |
| `callback` | This function takes no parameters.                                    |
| `oldText`  | The existing string of code that will be replaced by the callback.    |
| Output     | A Subsecond `Object` that contains all of the newly created elements. |

## name(): string

```ts
S("CallExpression").name();
// console.log
```

return the names of all currently selected nodes.

| Parameter | Description                                |
| --------- | ------------------------------------------ |
| N\A       | This function takes no parameters.         |
| Output    | A string of code of the selected elements. |

## name(newName: string): Subsecond

```ts
S("CallExpression").name((oldName) => oldName.split("").reverse().join(""));
// gol.elosnoc('Hello World!');
```

Change the names of all currently selected nodes.

| Parameter | Description                                                           |
| --------- | --------------------------------------------------------------------- |
| `newName` | The `string` to replace existing elements with.                       |
| Output    | A Subsecond `Object` that contains all of the newly created elements. |

## name((oldName: string) => string)): Subsecond

| Parameter  | Description                                                           |
| ---------- | --------------------------------------------------------------------- |
| `callback` | This function takes no parameters.                                    |
| `oldName`  | The existing string of code that will be replaced by the callback.    |
| Output     | A Subsecond `Object` that contains all of the newly created elements. |

## attr(name: string): any

> Check if an object property is using [method definition syntax](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Method_definitions)

```ts
S("Property").attr("method");
```

Get any attribute exposed from `@typescript-eslint/typescript-estree` on a selected node.

If multiple elements are selected this function will only return the first elements response.

| Parameter | Description                                                                     |
| --------- | ------------------------------------------------------------------------------- |
| `name`    | The name of the attribute to return.                                            |
| Output    | Returns the type of the attribute from estree or undefined if it doesn't exist. |

## before(newElement: string): Subsecond

> Insert an import at the top of the page

```ts
S("Program").children().eq(0).before("import S from 'subsecond';\n");
```

Insert an element directly before each currently selected element.

| Parameter    | Description                                                |
| ------------ | ---------------------------------------------------------- |
| `newElement` | A string of code.                                          |
| Output       | A Subecond `Object` containing the newly created elements. |

## after(newElement: string): Subsecond

> Console log every variable after it is defined

```ts
S("VariableDeclarator").each((declarator) => {
  declarator.parent().after(`console.log(${declarator.name()});`);
});
```

Insert an element directly after each currently selected element.

| Parameter    | Description                                                |
| ------------ | ---------------------------------------------------------- |
| `newElement` | A string of code.                                          |
| Output       | A Subecond `Object` containing the newly created elements. |

## lines(): number

> Get an array of lines of code for every function block.

```ts
S("FunctionDeclaration Block").map((block) => block.lines());
```

The total number of lines taken up by selected elements. Elements with overlapping lines are only counted once.

| Parameter | Description                                                   |
| --------- | ------------------------------------------------------------- |
| N\A       | This function takes no parameters.                            |
| Output    | The number of lines of code spanned by the selected elements. |

## eq(index: number): Subsecond

> Get the first parameter for each time function fn is called

```ts
S("CallExpression.fn").map((expression) => expression.children().eq(1));
```

Select a specific numbered index.

| Parameter | Description                                               |
| --------- | --------------------------------------------------------- |
| `index`   | Number. The specific index to select.                     |
| Output    | A Subsecond `Object` containing the one selected element. |

## length: number

> Return how many times console log is called.

```ts
S("CallExpression.console.log").length;
```

The amount elements selected by the current Subsecond object.

Important note this is a property and not a function.

## each(callback: (element: Subsecond, i: number, original: Subsecond) => void): Subsecond

```ts
S("Identifier").each((id) => console.log(id.text()));
```

Define a callback function to be executed on each selected element individually.

| Parameter  | Description                                         |
| ---------- | --------------------------------------------------- |
| `callback` | A function to executed for each selected element.   |
| `element`  | The individual element in it's own Subsecond object |
| `i`        | The numbered index of the current `element`.        |
| `original` | A copy of the unmodified Subsecond object.          |
| Output     | Returns the unmodified Subsecond object.            |

## map\<T\>(callback: (element: Subsecond, i: number, original: Subsecond) => T): T[]

> Return an array with each Identifier string in the code

```ts
S("Identifier").map((id) => id.text());
```

Transform each selected element and return all results in an array.

| Parameter  | Description                                                     |
| ---------- | --------------------------------------------------------------- |
| `callback` | A function to executed for each selected element.               |
| `element`  | The individual element in it's own Subsecond object             |
| `i`        | The numbered index of the current `element`.                    |
| `original` | A copy of the unmodified Subsecond object.                      |
| Output     | An array containing everything that was returned by `callback`. |

## filter(callback: (element: Subsecond, i: number, original: Subsecond) => boolean): Subsecond

> Filter all functions that are longer than 10 lines long

```ts
S("FunctionDefinition").filter((fn) => fn.lines() > 10);
```

Define a callback function that returns true or false for if the element should be in the resulting array.

| Parameter  | Description                                                                                 |
| ---------- | ------------------------------------------------------------------------------------------- |
| `callback` | A function to executed for each selected element.                                           |
| `element`  | The individual element in it's own Subsecond object                                         |
| `i`        | The numbered index of the current `element`.                                                |
| `original` | A copy of the unmodified Subsecond object.                                                  |
| Output     | Returns a Subsecond object containing each element who's callback function returned `true`. |

## find(selector: string): Subsecond

> All three of these methods are identical

```ts
S("VariableDeclaration").find("Property");

S("Property", S("VariableDeclaration"));

S("VariableDeclaration Property");
```

Query selector on an array of previously selected elements.

| Parameter  | Description                                            |
| ---------- | ------------------------------------------------------ |
| `selector` | See above selector syntax.                             |
| Output     | A Subsecond `Object` containing the selected elements. |

## parent(): Subsecond

> Go back one generation

```ts
S("Property").parent();
```

Select the parent element of each of the currently selected elements.

| Parameter | Description                                                               |
| --------- | ------------------------------------------------------------------------- |
| N\A       | This function takes no parameters.                                        |
| Output    | A Subsecond `Object` containing parent elements of the original elements. |

## parent(generations: number): Subsecond

> Go back 3 generations

```ts
S("Property").parent(3);
```

Go back a specific number of generations for each of the currently selected elements.

| Parameter     | Description                                                                 |
| ------------- | --------------------------------------------------------------------------- |
| `generations` | The number of generations to go back.                                       |
| Output        | A Subsecond `Object` containing ancestor elements of the original elements. |

## parent(selector: string): Subsecond

> Find the nearest VariableDeclaration parent for each Property element.

```ts
S("Property").parent("VariableDeclaration");
```

Go back generations until a selector is satisfied.

If an element doesn't have an ancestor that fufills the selector, nothing is returned for that element.

| Parameter  | Description                                                               |
| ---------- | ------------------------------------------------------------------------- |
| `selector` | String selector, see above for selector syntax.                           |
| Output     | A Subsecond `Object` containing parent elements of the original elements. |

## children(): Subsecond

> Select the function name and parameters for function `test()`

```ts
S("CallExpression.test").children();
```

Selects only the immidate next generation children for each currently selected node.

| Parameter | Description                                                              |
| --------- | ------------------------------------------------------------------------ |
| N\A       | This function takes no parameters.                                       |
| Output    | A Subsecond `Object` containing child elements of the original elements. |

## children(selector: string): Subsecond

> Select the Literal function parameters for the function `test()`

```ts
S("CallExpression.test").children("Literal");
```

Selects only the immidate next generation children for each currently selected node. Filters children nodes with selector syntax.

| Parameter  | Description                                                                       |
| ---------- | --------------------------------------------------------------------------------- |
| `selector` | Selector string. See above for more information about selector syntax.            |
| Output     | A Subsecond `Object` containing selected child elements of the original elements. |

## fileName(): string

> Get the file names of every file loaded

```ts
S().map((root) => root.fileName());
```

Get the fileName associated with the currently selected element.

If multiple elements are selected, this function only returns the file name of the first element.

| Parameter | Description                                                 |
| --------- | ----------------------------------------------------------- |
| N\A       | This function takes no parameters.                          |
| Output    | String. the filename as set by `S.load()` or `toNewFile()`. |

## esNodes(): es.TSESTree.Node[]

Eject out of Subsecond to the current underlying array of nodes from `@typescript-eslint/typescript-estree`.

Only do this if you know what you are doing.
Elements are passed by reference by default.
It is very easy to break stuff when dealing with these nodes directly.
If you find yourself commonly using this function, please file an issue with a feature request for a new use case.

| Parameter | Description                                                                           |
| --------- | ------------------------------------------------------------------------------------- |
| N\A       | This function takes no parameters.                                                    |
| Output    | An array of ESTree node objects. The objects have varied structure depending on type. |

## toNewFile(fileName: string): Subsecond

> Move all function declarations longer than 10 lines to their own files

```ts
S("FunctionDeclaration")
  .filter((fn) => fn.lines() > 10)
  .each((fn) => fn.toNewFile(`${fn.name()}.ts`));
```

Move currently selected nodes to a new file.

| Parameter  | Description                                                         |
| ---------- | ------------------------------------------------------------------- |
| `fileName` | String name of new file to create.                                  |
| Output     | A Subsecond `Object` containing the root of the newly created file. |

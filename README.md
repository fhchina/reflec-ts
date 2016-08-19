# reflec-ts

A enhanced (unofficial) version of the [official TypeScript compiler](https://github.com/Microsoft/TypeScript) that provides Reflection capabilities.
**Warning:** I'm improving the runtime API: some details will change until the first stable version. Keep your npm registry updated and feel free to submit issues/suggestions.

## Installing
Releases follow the official ones, so I try to keep the same scheme. For the latest stable version:


```
npm install -g reflec-ts
```

For the latest available version:

```
npm install -g reflec-ts@next
```

I try to keep reflec-ts master aligned with the official one, at least weekly.

## How it works

Every language that offers *real* reflection capabilities has to provide two things, both at coding time and runtime:
- Classes, interfaces, for building type hierarchies
- Meta-classes for accessing reflection data (for an example in Java, see [Class](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html)).

Actually, if picked individually, these two features are not enough to achieve reflection, but they should be linked in some way. For example, we generally expect to retrieve class metadata from its identifier (for example `String.class` in Java), or we want to instantiate a new object from class metadata (`Class.newInstance()` in Java).

Here, this aspect has been the hardest one to implement, since the mere type serialization was already achieved some time ago by many people (LanguageService API is enough).

In order to link metadata with classes (constructors in JavaScript) reflec-ts injects some synthetic instructions in the AST, just after the parsing phase. These instructions do not contain all type information, but just link the metadata object with constructors. In a second phase, just before the usual JS code emitting, reflec-ts builds a special JS file (Reflection.js) that contains metadata for all types.

Since we modify the AST, types metadata are immediately visible from the IDE (LanguageService uses the same parser), so we can achieve something like the following:

![reflec-ts demo](./doc/images/reflec-ts-demo.gif?raw=true "reflec-ts: working with Atom IDE")

As you can see, the compiler added a new *magic* local object `$reflection` that is automatically exported to be visible to the other modules.

## Usage

It works exactly like the official TypeScript compiler. You can use `tsc`:

```shell
node built/local/tsc.js -p /path/to/project/
```

or you can reference `typescriptServices.js` in your IDE.

Include `runtime-types/reflection.d.ts` in your project.

### How to enable reflection

Open `tsconfig.json` and add `"reflectionEnabled": true`, like the following snippet:

```
{
    "compilerOptions": {
        "noImplicitAny": true,
        "removeComments": false
    },
    "files": [
        "src/main.ts"
    ],
    "reflectionEnabled": true
}
```

### And now show me the code!

In one of your typescript files, create an interface and a class that implements it like the following:

```TypeScript

interface MyInterface {
    doSomething(what: string): number;
}

class MyClass implements MyInterface {
    counter = 0;

    doSomething(what: string): number {
        console.log('Doing ' + what);
        return this.counter++;
    }
}

```

now let's print some useful things about MyClass...

```TypeScript
for (let int of $reflection.MyClass.implements) {
    console.log('Implemented interface: ' + int.name)
}

for (let member of $reflection.MyClass.members) {
    console.log("Member name: " + member.name + " - member kind: " + member.type.kind);
}
```

compile with reflec-ts, launch it and, voilà!

```shell
$ node main.js
Implemented interface: MyInterface
Member name: counter - member kind: number
Member name: doSomething - member kind: function
```

now let's build an instance of MyClass from its metadata...

```TypeScript
let ctor = $reflection.MyClass.getConstructor<MyClass>();
let myObj = new ctor();
console.log('myObj instanceof MyClass: ' + (myObj instanceof MyClass));
myObj.doSomething('nothing :)');
```

launch it, and... yeah!!

```shell
$ node main.js
myObj instanceof MyClass: true
Doing nothing :)
```

## Current limitations

There are a few limitations, some of them are by design, others may be overcome in the future.
- reflec-ts needs `tsconfig.json`, it cannot be enabled with command line parameters.
- Legacy `--outFile` is not supported.
- If you need types metadata from an external library, you must include its .d.ts type definition file. Only referred types will be included in type serialization.

## Roadmap

I'm defining the roadmap right now, the project will evolve to support full reflection capabilities and modularity. At the moment, there are some things that have to be completed soon:
- Integration with the official test infrastructure: we want to create another `RunnerBase` subclass in order to write reflection tests.
- Support for class expressions.
- Support for other types (Union, Intersection, Literals, Tuples and so on)
- Sample projects

## Disclaimer

This project is derived from the TypeScript project, but **it is not officially supported by Microsoft**. See LICENSE for details.

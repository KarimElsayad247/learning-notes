# Import maps

## Goal

ship many small JavaScript files instead of one big JavaScript file. Thanks to HTTP/2 that no longer carries a material performance penalty during the initial transport, and in fact offers substantial benefits over the long run due to better caching dynamics. Whereas before any change to any JavaScript file included in your big bundle would invalidate the cache for the whole bundle, now only the cache for that single file is invalidated.

## How to use

In order to be ESM compatible, you must provide one of the following specifiers when loading Javascript code

- Absolute path: `import React from "/Users/Odin/projects/TOP/node_modules/react"`

- Relative path: `import React from "./node_modules/react"`

- HTTP path: `import React from "https://ga.jspm.io/npm:react@17.0.1/index.js"`

## Pinning

You can use the ./bin/importmap command that’s added as part of the install to pin, unpin, or update npm packages in your import map. This command resolves the package dependencies and adds the pins to your config/importmap.rb.

```sh
./bin/importmap pin react react-dom
```

Download vendor files to avoid going to cdns

```sh
./bin/importmap pin react --download
```

Delete downloaded pins

```sh
./bin/importmap unpin react --download
```

## Preloading

`<link rel="preload">` and its HTTP header equivalent provide a simple, declarative way of letting the browser know straight away about critical files that will be needed as part of the current navigation. When the browser sees the preload, it starts a high priority download for the resource, so that by the time it's actually needed it's either already fetched or partly there.

## Importmaps don’t at this stage manage dependencies and so are best used when your reliance on third party packages is minimal.

Import map considerations:

- Dependancy management (errors due to conflicting dependancies)
- Dependancy versioning and updates (security)
- Much less control over asset bundling 
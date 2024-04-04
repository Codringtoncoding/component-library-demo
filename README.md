# React Component Library Demo

This Repo contains the necessary steps in the readme to create your own component library.

## Prerequisites

- NPM
- Node
- Github account
- Good terminal setup
- Vscode
- Basic knowledge of React + Typescript. (We will only be creating 1 component)

I currently use node v20. It's best to have that installed locally. You can use node version manager (nvm) to install and use multiple node versions (see [nvm docs](https://github.com/nvm-sh/nvm) for installation instructions).

## Setup

1. `npm init` press enter and take all the defaults for now at each step.
2. `npm install react typescript @types/react --save-dev`

## Folder structure

It is super important we keep this folder structure as we need to export our components at each level so they can be consumed later on in the correct way.

.
├── src
│ ├── components
| │ ├── Button
| | │ ├── Button.tsx
| | │ └── index.ts
| │ └── index.ts
│ └── index.ts
├── package.json
└── package-lock.json

### Add typescript

`npx tsc --init`

add this config to your file:

```
{
  "compilerOptions": {
    "target": "es5",
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "strict": true,
    "skipLibCheck": true,
    "jsx": "react",
    "module": "ESNext",
    "declaration": true,
    "declarationDir": "types",
    "sourceMap": true,
    "outDir": "dist",
    "moduleResolution": "node",
    "allowSyntheticDefaultImports": true,
    "emitDeclarationOnly": true,
  }
}
```

### components

Now lets add out button component.

```
import React from "react";

export interface ButtonProps {
  label: string;
}

const Button = (props: ButtonProps) => {
  return <button>{props.label}</button>;
};

export default Button;
```

1. we need to the export this button from the first level our our directory.

`src/components/Button/index.ts`
`export { default } from "./Button";`

2. Then we export the button from the components directory.

`src/components/index.ts`
`export { default as Button } from "./Button";`

3. Finally we export all our components from the base src directory.

`src/index.ts` - this is our entry point to the application
`export * from './components';`

This is really important otherwise rollup won't be able to bundle our components properly and we won't be able to use our Typesafe components in our consuming app!

## Rollup

`npm install rollup @rollup/plugin-node-resolve @rollup/plugin-typescript @rollup/plugin-commonjs rollup-plugin-dts --save-dev`

1. create a `rollup.config.mjs` file in your root.

```
import resolve from '@rollup/plugin-node-resolve';
import commonjs from '@rollup/plugin-commonjs';
import typescript from '@rollup/plugin-typescript';
import dts from 'rollup-plugin-dts';
import packageJson from './package.json' assert { type: 'json' };

export default [
  {
    input: 'src/index.ts',
    output: [
      {
        file: packageJson.main,
        format: 'cjs',
        sourcemap: true,
      },
      {
        file: packageJson.module,
        format: 'esm',
        sourcemap: true,
      },
    ],
    plugins: [
      resolve(),
      commonjs(),
      typescript({ tsconfig: './tsconfig.json' }),
    ],
  },
  {
    input: 'dist/esm/types/index.d.ts',
    output: [{ file: 'dist/index.d.ts', format: 'esm' }],
    plugins: [dts()],
  },
];
```

## Our package JSON.

Here we need to amend the following properties.

1. main
2. module
3. files
4. types
5. scripts

Here is an example of mine

```
{
"name": "@Codringtoncoding/design-system-demo",
"publishConfig": {
"registry": "https://npm.pkg.github.com/Codringtoncoding"
},
"version": "1.0.0",
"description": "Design-system-demo",
"scripts": {
"test": "test",
"rollup": "rollup -c"
},
"repository": {
"type": "git",
"url": "design-system-demo"
},
"keywords": [
"design",
"system"
],
"author": "Humphrey Codrington",
"license": "ISC",
"devDependencies": {
"@rollup/plugin-commonjs": "^25.0.7",
"@rollup/plugin-node-resolve": "^15.2.3",
"@rollup/plugin-typescript": "^11.1.6",
"@types/react": "^18.2.74",
"react": "^18.2.0",
"rollup": "^4.14.0",
"rollup-plugin-dts": "^6.1.0",
"typescript": "^5.4.3"
},
//were are interested in these
"main": "dist/cjs/index.js",
"module": "dist/esm/index.js",
"files": [
"dist"
],
"types": "dist/index.d.ts"
}
```

## Building the Library!

.
├── src
│ ├── components
| │ ├── Button
| | │ ├── Button.tsx
| | │ └── index.ts
| │ └── index.ts
│ └── index.ts
├── package.json
├── package-lock.json
├── tsconfig.json
└── rollup.config.js

now for the magic run:

`npm run rollup`

If you have been successful you will see this appear in your project!

![File structure](https://file%2B.vscode-resource.vscode-cdn.net/var/folders/0p/9d9q90vd3nj469llpvg4rpcm0000gn/T/TemporaryItems/NSIRD_screencaptureui_PZhz5m/Screenshot%202024-04-04%20at%2017.05.05.png?version%3D1712246711252)

## Publishing the Library!

1. create a `.gitignore` and add

```
dist
mode_modules
```

2. update the package.json with your cred similar to what I have done above

`package.json`

```
    {
    "name": "@YOUR_GITHUB_USERNAME/YOUR_REPOSITORY_NAME",
    "publishConfig": {
    "registry": "https://npm.pkg.github.com/YOUR_GITHUB_USERNAME"
    },
}
```

3. We need to configure our local install of NPM to publish to your github account. If you already have one you need to open and add the following:

- open your terminal at the root and type `open -e ~/.npmrc` This should open your file so you can copy in the details below and save it.
- if you do not have one you need to create one by running `touch .npmrc` and run the command above to open and paste in the below info with your creds.
- We need to generate a new personal access token with `write:packages` which can be done [here](https://github.com/settings/tokens)

```
    registry=https://registry.npmjs.org/
    @YOUR_GITHUB_USERNAME:registry=https://npm.pkg.github.com/
    //npm.pkg.github.com/:_authToken=YOUR_AUTH_TOKEN
```

4. Now run `npm publish`. Boom you have published your first package which you can check out in your github page.

## Using our library!

1. lets create a new create react app
   `npx create-react-app my-app --template typescript`

2. Run the app `npm run start`

3. Try and install your library - don't forget the @s

`npm install @YOUR_GITHUB_USERNAME/YOUR_REPOSITORY_NAME`

4. Import into the application and see if it works

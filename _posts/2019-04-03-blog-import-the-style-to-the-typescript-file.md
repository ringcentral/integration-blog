---
layout: post
title:  "Researching the solution for import style file in TypeScript"
date:   2019-04-03 14:00:00 +0800
categories: blog
author: Jeff Wu
---

![](/integration-blog/assets/2019-04-03-blog-import-the-style-to-the-typescript-file/babel-ts-scss.jpg)

### TypeScript show the error when import style file like `scss` or `css`

After the success of migrating to TypeScript development from JavaScript,  we can get more confident in building an incredible product. But the challenge will not stop. When I import a style file to TypeScript file, the editor shows an error said that could not find the module. To overcome the issue about the style file importing is the next challenge we need to face.

I am researching the solution to avoid the problem that the error message showed due to the TS compile do not know about that when importing the style file. Currently, I found some solution and need to clarify which one is the better choice.



### Using the "typings-for-css-modules-loader"
It's a Webpack loader make our style file like SCSS can have intellisense by generating a TypeScript definition (d.ts). If you have using the "css-loader" you can easily set up the Webpack configuration by the following:

```javascript
{
  loader: 'typings-for-css-modules-loader',
    options: {
      modules: true,
      localIdentName: `${hashPrefix}_[path]_[name]_[local]_[hash:base64:5]`,
      namedExport: true,
      camelCase: true,
			banner: '// *** Generated File - Do not Edit ***'
    }
}
```
However, it has some flaw. The process of the definition file generator need always start a watch by a command like `yarn start`. It also makes our size of the repository become a bigger one although the intellisense is convenient for us.



### Using the "require" syntax instead of using import.

Using the different style or syntax for import may make our project become a mess, we should keep a consistent form of the import, so maybe there were have a better solution that I not found yet.



### TypeScript server plugin - [typescript -plugin-css-modules](https://github.com/mrmckeb/typescript-plugin-css-modules)
This plugin is a TypeScript plugin was inspired by this [create-react-app issue](https://github.com/facebook/create-react-app/issues/5677) and based on css-module-types. It's easy to set up by installing the node module and add plugin to the `tsconfig.json` like the following:

```json
{
  "compilerOptions": {
    "plugins": [{ "name": "typescript-plugin-css-modules" }]
  }
}
```

The match pattern default is `\\.module\\.(sa|sc|c)ss$`. We can custom set other pattern by the property.

```json
{
  "compilerOptions": {
    "plugins": [
      {
        "name": "typescript-plugin-css-modules",
        "options": {
          "customMatcher": "\\.m\\.css$",
          "camelCase": "dashes"
        }
      }
    ]
  }
}
```
After the setting, we also have to change the version of TypeScript. By default, VSCode will use its own version of TypeScript. To make it work correctly with this plugin, there have two options:

1. Add this plugin to "typescript.tsserver.pluginPaths" in settings.
```json
{
  "typescript.tsserver.pluginPaths": ["typescript-plugin-css-modules"]
}
```
But this option is only setable for users setting. It's not convinient for coporation. The next option is more useful for us.

2. Use our workspace's version of TypeScript
  The workspace's version which will load the plugins from our tsconfig.json` file. We can add the config to the workspace setting.

```json
{
  "typescript.tsdk": "./node_modules/typescript/lib"
}
```

Besides, we found the issue about import not only style file but also other files like SVG in TypeScript. It will show the error like: "Cannot find module '../../assets/images/logo.svg'.ts(2307)". To resolve the issue that unable to import the SVG file in TypeScript, we can create a definition file named "custom.d.ts" and add the following code and add the file path to the includes field of the `tsconfig.json`:
```typescript
declare module "*.svg" {
  const content: any;
  export default content;
}
```

Using the TypeScript server plugin is a good way for the solution by import the style files that TypeScript can understand. It also not generates the definition file to the project and keeping the repository clean. Furthermore, we can enjoy the benefit of intellisense for CSS selector.



### References:

- [Github: Typescript Plugin CSS Modules](https://github.com/mrmckeb/typescript-plugin-css-modules)
- [Github: Typings For CSS Modules Loader](https://github.com/Jimdo/typings-for-css-modules-loader)
- [CSS Modules for React and Typescript](https://medium.com/@tommybernaciak/css-modules-for-react-and-typescript-2dd8f0fd7cdd)


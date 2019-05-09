---
layout: post
title:  "Researching the solution for import style file in TypeScript"
date:   2019-04-03 14:00:00 +0800
categories: blog
author: Jeff Wu
---

![](/integration-blog/assets/2019-04-03-blog-import-the-style-to-the-typescript-file/babel-ts-scss.jpg)

### TypeScript show the error when import style file like scss or css

After the success of migrating to TypeScript development from JavaScript,  we can get more confident in building an incredible product. But the challenge will not stop. When I import a style file to TypeScript file, the editor shows an error saying that could not find the module. To overcome the issue of importing style files is the next challenge we need to face.

I was researching for the solution to avoid the problem that TS compiler would show errors when importing style files because it does not know what types to expect from those files. I found some solutions but need to evaluate which one is the better choice.



### Using the "typings-for-css-modules-loader"
This is a Webpack loader that make our style files like SCSS to support intellisense by generating a TypeScript definition file (d.ts). If you have been using the "css-loader" already, you can easily set up the Webpack configuration like the following:

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
However, this method has some flaws. The process of generating the definition files need a `watch` process to be started by command line (by scripting it to `yarn start`, for example). It would also make the size of the repository bigger, even though the intellisense support is convenient.



### Using the "require" syntax instead of using import.

Using different syntax for importing modules may make our project messy, we should keep the syntax consistent. So maybe a different solution should be considered.



### TypeScript server plugin - [typescript -plugin-css-modules](https://github.com/mrmckeb/typescript-plugin-css-modules)
This plugin is a TypeScript plugin was inspired by this [create-react-app issue](https://github.com/facebook/create-react-app/issues/5677) and based on css-module-types. It's easy to set up by installing the node module and add plugin to the `tsconfig.json` like the following:

```json
{
  "compilerOptions": {
    "plugins": [{ "name": "typescript-plugin-css-modules" }]
  }
}
```

The match pattern default is `\\.module\\.(sa|sc|c)ss$`. We can customize the pattern like the following:

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
After configurations, we also have to change the version of TypeScript used by VSCode. By default, VSCode comes with its own version of TypeScript. However, we can specify the TS version by two different ways:

1. Add this plugin to "typescript.tsserver.pluginPaths" in settings.
```json
{
  "typescript.tsserver.pluginPaths": ["typescript-plugin-css-modules"]
}
```
But this option is only available for user settings, and is not convenient for collaborative work. The next option is more suitable for us.

2. Use the version of TypeScript specified in our workspace
    We can add the following configuration to the workspace settings to force VSCode to use the TypeScript engine specified in our work space, which will load the plugins from our tsconfig.json.

```json
{
  "typescript.tsdk": "./node_modules/typescript/lib"
}
```

Aside from style files, we also run into the same issue for other files like SVG. It will show errors like: "Cannot find module '../../assets/images/logo.svg'.ts(2307)". To resolve this issue with loading SVG files, we can create a definition file named "custom.d.ts" with the code shown below, and add the file path to the "includes" field in the `tsconfig.json` file.
```typescript
declare module "*.svg" {
  const content: any;
  export default content;
}
```



### Conclusion

Using TypeScript server plugin is a good solution for importing style files and assets in TypeScript. It does not generate definition files into the project so that the repository is kept clean. Furthermore, we can also enjoy the benefits of intellisense for CSS selectors.



### References:

- [Github: Typescript Plugin CSS Modules](https://github.com/mrmckeb/typescript-plugin-css-modules)
- [Github: Typings For CSS Modules Loader](https://github.com/Jimdo/typings-for-css-modules-loader)
- [CSS Modules for React and Typescript](https://medium.com/@tommybernaciak/css-modules-for-react-and-typescript-2dd8f0fd7cdd)


# Vue Cli Build Production Minimize Behaviour
For deployment, we usually use the `npm run build` command to build the project into dist folder. At first, I used to seperate development and production environment variables, and add the following in `package.json`:
```
"scripts": {
  "serve": "vue-cli-service serve --mode dev",
  "build": "vue-cli-service build --mode prod",
  "lint": "vue-cli-service lint"
},
```
As well as environment variables files `.env.dev` and `.env.prod`.

But soon we find that the javascript files build are not minimized, which means it takes a very long time to load the javascript files when users first landed on the website.

Soon we try to solve this problem by avoiding `Vue.use(<package>)` in main.js to reduce the size, try to chunk and split those route components, etc.

Finally when we dig into the javascript files built, we find that it is not minimized. At last we change `package.json` to this:

```
"scripts": {
  "serve": "vue-cli-service serve --mode dev",
  "build": "vue-cli-service build --mode production",
  "lint": "vue-cli-service lint"
},
```

As mode production is a mode that build minimized js files, as decripted in vue cli documents.

Then the `main.js` of our project reduces its size from more than 20MB to only 400+KB!

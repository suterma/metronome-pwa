# metronome-pwa

A simple metronome, as progressive web app

## Project setup

```
npm install
```

### Compiles and hot-reloads for development

```
npm run serve
```

### Compiles and minifies for production

```
npm run build
```

### Lints and fixes files

```
npm run lint
```

### Customize configuration

See [Configuration Reference](https://cli.vuejs.org/config/).

# Notes

## How to use Bulma in a Vue3 project (not using Buefy)

For a Vue3 project created with the Vue CLI (With the CSS Pre-processors not selected), mainly follow the guide at https://bulma.io/documentation/customize/with-node-sass/ with a few changes:

-   Omit the first step, since we are using the already available project structure with it's package.json
-   As indicated install the packages for development, as we are only customizing bulma during development time

    npm install node-sass --save-dev
    npm install bulma --save-dev

This will add the packages to the `devDependencies` in `package.json`

Then follow the steps from Number 3: https://bulma.io/documentation/customize/with-node-sass/#3-create-a-sass-file with the following adaptions

-   Use an appropriate style name
-   Put the css in `public/css`

To run both the SASS autocompilation and npm's serve script, use two terminals. You can do that also from within VSCode.

## Aligning the Buttons with controls

https://css-tricks.com/snippets/css/a-guide-to-flexbox/

## Resources

-   [Getting Started with Web Audio API](https://www.html5rocks.com/en/tutorials/webaudio/intro/) is a very good intro into the Audio API

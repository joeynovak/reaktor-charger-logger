# reaktor-charger-logging

> A Reaktor Charger Logging App

This electron app lets you view charging details when using a Turnigy Reaktor charger (It probably works with LOTS of other chargers as well)

##Usage
Click on "refresh port list", then click "open port".

Note: The charger doesn't send anything via the USB port until it is doing something (charging, etc...)

##Installing
Currently there aren't any binary packages, you'll have to clone this repo and then follow the instructions below. (You'll need node and npm to be installed)

#### Build Setup

``` bash
# install dependencies
npm install

# serve with hot reload at localhost:9080
npm run dev

# build electron application for production
npm run build

# run unit & end-to-end tests
npm test


# lint all JS/Vue component files in `src/`
npm run lint

```

---

This project was generated with [electron-vue](https://github.com/SimulatedGREG/electron-vue) using [vue-cli](https://github.com/vuejs/vue-cli). Documentation about the original structure can be found [here](https://simulatedgreg.gitbooks.io/electron-vue/content/index.html).

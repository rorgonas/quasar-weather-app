# Quasar App (quasar-weather-app)

A Quasar Framework app

## Install the dependencies
```bash
yarn
```

### Install plugins from /src-cordova
```
cordova plugin add cordova-plugin-geolocation
```

### List all available cordova ios devices from /src-cordova
```
cordova run ios --list
```

### Start the app in development mode
```bash
quasar dev -m android
quasar dev -m ios
quasar dev -m ios -e "iPhone-8, 14.4"
```

### Build the app for production
```bash
quasar build -m android
quasar build -m ios
```

### Customize the configuration
See [Configuring quasar.conf.js](https://quasar.dev/quasar-cli/quasar-conf-js).

In this tutorial we are going to build a famous todo App using React Native with Redux Toolkit.

> _The `Redux Toolkit` package is intended to be the standard way to write Redux logic. It was originally created to help address three common concerns about Redux:_

* Configuring a Redux store is too complicated
* I have to add a lot of packages to get Redux to do anything useful
* Redux requires too much boilerplate code

### Prerequisites
* Expo CLI
* IOS or Android Device with Expo Client installed / Xcode / Android Studio - __N.B__ I am using Xcode for this tutorial.
* Knowledge of React Native

### Quick Start

Let's start off by installing an Expo CLI if it's not already installed:

```bash
yarn add expo-cli --global
```

Initiliaze the project:

```bash
expo init reactNativeReduxToolkitExample
cd reactNativeReduxToolkitExample
```

We also need to add packages for navigation and redux toolkit:

```bash
yarn add @react-navigation/native @react-navigation/stack react-redux @reduxjs/toolkit
```
And we finally add these dependencies with Expo since our project is Expo managed:

```bash
expo install react-native-gesture-handler react-native-reanimated react-native-screens react-native-safe-area-context @react-native-community/masked-view
```

### State management with Redux Toolkit
Let's add a state management functionality by creating redux a store, action and reducer. Run these commands in the root directory of your project:

```bash
mkdir src && cd src
mkdir store && cd store
touch index.js
```

#### store/index.js

```javascript
import { configureStore } from '@reduxjs/toolkit'
import { combineReducers } from 'redux'

const reducer = combineReducers({
})

const store = configureStore({
    reducer,
})

export default store;
```

### Stack Navigator

Let's add screens and a navigation functionality so that we can easily navigate between screens:

```bash
mkdir navigation
mkdir screens
```

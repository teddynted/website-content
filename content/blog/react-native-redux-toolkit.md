In this tutorial we are going to build a famous todo App using React Native with Redux Toolkit.

> _The `Redux Toolkit` package is intended to be the standard way to write Redux logic. It was originally created to help address three common concerns about Redux:_

* Configuring a Redux store is too complicated
* I have to add a lot of packages to get Redux to do anything useful
* Redux requires too much boilerplate code

### Prerequisites
* Expo CLI
* IOS or Android Device with Expo Client installed / Xcode / Android Studio - __N.B__ I am using Xcode for this tutorial.

### Quick Start

Let's start off installing expo-cli if it's not already installed, we will also add packages for navigation and redux.

```bash
yarn add expo-cli --global
expo init reactNativeReduxToolkitExample
yarn add @react-navigation/native @react-navigation/stack react-redux @reduxjs/toolkit
expo install react-native-gesture-handler react-native-reanimated react-native-screens react-native-safe-area-context @react-native-community/masked-view
```

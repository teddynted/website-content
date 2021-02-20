
This tutorial walks you through the steps of setting up a Continuous Integration/Continuous Delivery workflow using Github actions. So we are going to build a simple Hello world app using React Native. We will then deploy that app to Expo through the workflow.

> _Continuous integration and Continuous Delivery are the processes in which development team delivers software in short cycles in the main branch while ensuring that it does not impact any changes made by other developers working on the same codebase._

`Expo` is a open source framework and a platform for react native applications to help developers build native IOS and Android mobile apps. Expo gives you will be able to develop and publish updates for your apps faster. When an update is ready for production you can easily publish it. There will be no need to update the app through the App Store manually thus saving you a lot time when it comes to developing and publishing updates of your solution. So in these tutorial we are going to automate the process of publishing to Expo using Github actions, this will happen whenever you create a PR against your master branch.

### Prerequisites

* Expo and Github account
* Expo installed on either your Andriod or IOS device
* Expo CLI
* And of course a basic knowledge of React Native

### Let's get it cracking

Let's begin by installing Expo CLI and creating our app, ensure that node is installed on machine since we need it for expo CLI.

```bash
npm install -g expo-cli
expo init react-native-github-pages
cd react-native-github-pages
```

We are not going to do much on the code, just change this wording `Open up App.js to start working on your app` to `Hello World!`, as shown below in the `App.js` file:

```javascript
import { StatusBar } from 'expo-status-bar';
import React from 'react';
import { StyleSheet, Text, View } from 'react-native';

export default function App() {
  return (
    <View style={styles.container}>
      <Text>Hello World!</Text>
      <StatusBar style="auto" />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
    alignItems: 'center',
    justifyContent: 'center',
  },
});
```

The next step is to add our Github workflow yaml file in the root directory of our project:

```bash
mkdir .github && cd .github && mkdir workflows && cd workflows && touch actions.yml
```


This tutorial walks you through the steps of setting up a Continuous Integration/Continuous Delivery workflow using Github actions. So we are going to build a simple Hello world app using React Native. We will then deploy that app to Expo through the workflow.

> _Continuous integration and Continuous Delivery are the processes in which development team delivers software in short cycles in the main branch while ensuring that it does not impact any changes made by other developers working on the same codebase._

`Expo` is a open source framework and a platform for react native applications to help developers build native IOS and Android mobile apps. Expo gives you will be able to develop and publish updates for your apps faster. When an update is ready for production you can easily publish it. There will be no need to update the app through the App Store manually thus saving you a lot time when it comes to developing and publishing updates of your solution. So in these tutorial we are going to automate the process of publishing to Expo using Github actions, this will happen whenever you create a PR against your master branch.

### Prerequisites

* Expo and Github account
* Expo installed on either your Andriod or IOS device
* Expo CLI
* And of course a basic knowledge of React Native

### Let's get it cracking

Let's begin by install Expo CLI and creating our app, ensure that node is installed on machine since we need it to expo CLI

```bash
npm install -g expo-cli
expo init react-native-github-pages
```


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
#### Store

Let's configure our store `store/index.js`:

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

#### Slice

Add a slice - `todos` - that will accepts a reducer function - `addEditDeleteTodoSuccess` and an initial state value - `null`. A slice will automatically generates a slice reducer with corresponding action creators and action types:

```bash
touch todos.js
```

`store/todos.js`:

```javascript
import { createSlice } from '@reduxjs/toolkit'

// Slice
const slice = createSlice({
    name: 'todos',
    initialState: {
        todos: null,
    },
    reducers: {
        addEditDeleteTodoSuccess: (state, action) => {
            state.todos = action.payload;
        }
    },
});
export default slice.reducer

// Action
const { addEditDeleteTodoSuccess } = slice.actions
export const addEditDeleteTodo = (todos) => async dispatch => {
    try {
        dispatch(addEditDeleteTodoSuccess(todos));
    } catch (e) {
        return console.error(e.message);
    }
}
```

Update `store/index.js` by importing a slice and passing it to `combineReducers`:

```javascript
...
import todos from './todos'

const reducer = combineReducers({
    todos
})
...
```

### Stack Navigator

Change directories to the root, and add screens and a navigation functionality so that we can easily navigate between screens:

```bash
mkdir navigation
mkdir screens && cd screens
touch AddTodo.js EditTodo.js ViewTodos.js
```

`AddTodo.js`:

```javascript
import React, { useState } from 'react'
import { 
    StyleSheet,
    View, 
    Text, 
    TouchableOpacity, 
    TextInput, 
    Dimensions 
} from 'react-native'
import { AntDesign } from '@expo/vector-icons'

const { width } = Dimensions.get('window')

export default function AddTodo({navigation}) {
    const [value, onChangeText] = useState('')
    const addTodoItem = value => {
    }
    return (
        <View style={styles.container}>
            <Text style={styles.text}>React Native Todo App</Text>
            <View style={styles.innerContainer}>
                <TextInput
                    style={styles.textInput}
                    onChangeText={text => onChangeText(text)}
                    value={value}
                    placeholder="Add a task to do"
                />
                <TouchableOpacity
                    style={styles.buttonContainer}
                    onPress={() => addTodoItem(value)}
                >
                    <AntDesign name="plus" size={24} color="white" />
                </TouchableOpacity>
            </View>
        </View>
    )
}

let styles = StyleSheet.create({
    container: {
        flex: 1,
        justifyContent: 'center',
        alignItems: 'center',
        backgroundColor: 'white',
        width: width
    },
    innerContainer: {
        justifyContent: 'center',
        alignItems: 'center',
        flexDirection: 'row',
        width: '90%',
        marginTop: 30,
        marginBottom: 20,
    }, 
    text: {
        color: '#101010',
        fontSize: 20,
        fontWeight: 'bold'
    },
    textInput: {
        height: 50, 
        width: '70%',
        borderColor: '#ccc', 
        borderWidth: 1,
        borderBottomLeftRadius: 8,
        borderTopLeftRadius: 8,
        paddingLeft: 10,
        fontSize: 17,
    },
    buttonContainer: {
        height: 50,
        backgroundColor: '#222',
        borderBottomRightRadius: 8,
        borderTopRightRadius: 8,
        padding: 10,
        margin: 0,
        justifyContent: 'center',
        alignItems: 'center',
    }
})
```

Let's connect Redux to our `AddTodo.js` screen and add the following code: 

```javascript
...
import { useDispatch, useSelector } from 'react-redux'
import { addEditDeleteTodo } from '../store/todos'
...
const dispatch = useDispatch()
    const [value, onChangeText] = useState('')
    const { todos } = useSelector(state => state.todos)
    const addTodoItem = value => {
        if( value ) {
            let todoEntry;
            if(!todos) {
                todoEntry = [{ 'index': todos ? todos.length + 1 : 1, 'isDone': false, 'task': value}];
            } else {
                todoEntry = [...todos, { 'index': todos ? todos.length + 1 : 1, 'isDone': false, 'task': value}];
            }
            dispatch(addEditDeleteTodo(todoEntry));
            onChangeText('')
            navigation.navigate('ViewTodos', { item: '' })
        }
    }
...
    return (
         ...
            {todos && <TouchableOpacity
                    style={{
                        backgroundColor: 'white'
                    }}
                    onPress={() => navigation.navigate('ViewTodos')}
                >
                    <Text style={{
                        color: '#dc3545',
                        fontWeight: 'bold'
                    }}>View Your Todo List</Text>
                </TouchableOpacity>
            }
          ...
    )
...
```

We will integrate Redux with the screen using `useDispatch` and `useSelector`. And lastly the add the functionality to update the todos against Redux store in `addTodoItem`.

![alt text](https://nextjs-portfolio.s3.amazonaws.com/AddTodo.png "Add Todo Screen")

And lastly we just need to wrap StackNavigator with Provider inside `App.js` with the below content:

```javascript
import React from 'react'
import { Provider } from 'react-redux'
import store from './src/store'
import Navigation from './src/navigation'

export default function App() {
  return (
    <Provider store={store}>
        <Navigation />
    </Provider>
  )
}
```

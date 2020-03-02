# react-hooks-kata
Base repository for the kata I create to practice React Hooks

## 1 - Display todo list

* Go to components/TodoList.js 
* Create a function name TodoList() that will return a list of todo from state.todos
* Use pre-defined CSS that was commented to create a better looks and feels
* The result should looks like 

```javascript
import React, {useContext} from 'react'
import TodosContext from '../context'


export default function TodoList() {
    const {state, dispatch} = useContext(TodosContext)
    const title = state.todos.length > 0 ? `${state.todos.length} Todos` 
      : "Nothing to do!"

    return (
        <div className="container mx-auto max-w-md text-center font-mono">
            <h1 className="text-bold">{title}</h1>
            <ul className="list-reset text-white p-0">
                {state.todos.map(todo => (
                    <li key={todo.id} className="flex items-center 
                      bg-orange-500 border-black border-dashed border-2 my-2 py-4">
                        <span
                            className={`flex-1 ml-12 cursor-pointer`}>{todo.text}</span>
                        <button>
                            <img src="https://icon.now.sh/edit/0050c5" 
                              alt="Edit Icon" className="h-6"/>
                        </button>
                        <button>
                            <img src="https://icon.now.sh/delete/8b0000" 
                              alt="Delete Icon" className="h-6"/>
                        </button>
                    </li>
                ))}
            </ul>
        </div>
    )
}
```

## 2 - Create a new todo
#### Go to components/TodoForm.js
* Add local state todo via useState
* Add useContext from TodosContext to extract the dispatch function 
* Add form with className="flex justify-center p-5"
* Add input field 
```javascript 
<input type="text" className="border-black border-solid border-2" 
    onChange={event => setTodo(event.target.value} />
```
* Add onSubmit function to the form
```javascript  
const handleSubmit = event => {
  event.preventDefault()
  dispatch({type: "ADD_TODO", payload: todo})
  setTodo("")
}
``` 
#### Go reducer.js to add handling for ADD_TODO
```javascript
case "ADD_TODO": 
  if (!action.payload) {
      return state;
  }

  if (state.todos.findIndex(t => t.text === action.payload) > -1) {
      return state;
  }

  const newTodo = {
      id: uuidv4(),
      text: action.payload,
      complete: false
  }

  const addedTodos = [...state.todos, newTodo]

  return {
      ...state,
      todos: addedTodos
  }
        
```

## 3 - Toogle a todo
#### Go to components/TodoList.js, under li > span, 
* Add handling for doubleClick 
```javascript 
onDoubleClick={() => dispatch({type: "TOOGLE_TODO", payload: todo})} ...
```
* add different styling for complete item
```javascript 
className={ ... ${todo.complete && "line-through text-gray-700"}}
```

#### Go to reducer.js
* Add handling for TOOGLE_TODO
```javascript
case "TOOGLE_TODO": 
  const toggledTodos = state.todos.map(t => 
      t.id === action.payload.id 
      ? {...action.payload, complete: !action.payload.complete}: t) 
  return {
      ...state,
      todos: toggledTodos
  }
```

## 4 - Edit a todo
#### Go to components/TodoList.js
* On edit button, add handling for onClick as a dispatch function 
```javascript
onClick = {() => dispatch({type: "SETCURRENT_TODO", payload: todo})}
```

#### Go to components/TodoForm.js
* deconstruct currentTodo from state
```javascript
const {state: {currentTodo = {}}, dispatch} = useContext(TodosContext);
```

* add handling for current item changes that set the value to the todo text box via useEffect on currentTodo.id
```javascript
useEffect(() => {
    if (currentTodo.text) {
        setTodo(currentTodo.text)
    }
    else {
        setTodo("")
    }
}, [currentTodo.id])
```

* on handleSubmit add handling for UPDATE_TODO if there the currentTodo has any text. 
```javascript
...
if (currentTodo.text) {
  dispatch({type: "UPDATE_TODO", payload: todo})
}
else {
  dispatch({type: "ADD_TODO", payload: todo})
}
...
```

#### Go do reducer.js
* Add handling for SETCURRENT_TODO
```javascript
case "SETCURRENT_TODO": 
  return {
      ...state,
      currentTodo: action.payload
  }
```

* Add handling for UPDATE_TODO
```javascript
case "UPDATE_TODO": 
  if (!action.payload) {
      return state;
  }

  if (state.todos.findIndex(t => t.text === action.payload) > -1) {
      return state;
  }

  const updatedTodo = {...state.currentTodo, text: action.payload}
  const updatedTodoIndex = state.todos.findIndex(t => t.id === state.currentTodo.id)

  const updatedTodos = [
      ...state.todos.slice(0, updatedTodoIndex),
      updatedTodo,
      ...state.todos.slice(updatedTodoIndex + 1)
  ]
  return {
      ...state,
      todos: updatedTodos,
      currentTodo: {}
  }
```
## 5 - Delete a todo

#### Go to components/TodoList.js
* On delete button, add onClick dispatch activity 
```javascript
onClick={() => dispatch({type: "REMOVE_TODO", payload: todo})}
```

#### Go do reducer.js
* Add handling for REMOVE_TODO
```javascript
case "REMOVE_TODO": 
  const remainingTodos = state.todos.filter(t => t.id !== action.payload.id);
  const isRemovedTodo = state.currentTodo.id === action.payload.id ? {} : state.currentTodo;
  return {
      ...state,
      todos: remainingTodos,
      currentTodo: isRemovedTodo
  }
```

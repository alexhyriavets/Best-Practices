## Creating custom error class
### The problem we solve
Build-in JavaScript error class allows us to pass only message to its constructor. But we want also pass some additional info, for example, error code  generated by backend.
### The idea
We will create our custom error class, which will extend built-in error class and will populate it with additional fields or/and methods.
### Implementation
```javascript
// ApiError.js
export default class ApiError extends Error {
  // static getters to incapsulate specific error codes
  static get NO_PERMISSIONS () { return 'np24a' }
  static get WRONG_LOCALE () { return 'wl56b' }

  constructor (message, code = '') {
    // calls the contructor of build-it Error and pass error message to it
    super(message)
    
    this.code = code
  }
}
```
### Usage
We are in store and we make ajax call. We know **fetchUsers()** can throw an **ApiError**
```javascript
async fetch ({ commit }) {
  try {
    const users = await fetchUsers()
    
    commit(types.SET_USERS, users)
  } catch (err) {
    // check if specific error code
    if (err.code === ApiError.NO_PERMISSIONS) {
      // some logic
    }
    
    // throw error forward
    throw err
  }
}
```

### References
- [Custom errors, extending Error](https://javascript.info/custom-errors)
- [Custom JavaScript Errors in ES6](https://medium.com/@xjamundx/custom-javascript-errors-in-es6-aa891b173f87)

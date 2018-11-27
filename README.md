## Vue error handling practical guide
In this guide i going to cover error handling related to **ajax calls**.

In the most cases every api request goes through three main stages:
1. Api Module
2. Store Module
3. View Layer

### Api Module
Perform api call, validate, serialize/deserialize the result and return it. This is the place where we should specify error: write error message and add error code to it.
```javascript
try {
	// perform api call, validation, serialization/deserialization 
	// return the result in case of success
} catch (err) {
	const { errorCode } = err.response.data
	// catch the error, populate it with error code and throw forward
	throw new ApiError('Error during fetching users', errorCode)
}
```
**ApiError** is the custom error which extend standart **Error**. Implementation of this class described <a href="#">here</a>.

### Store Module
Calls Api Module, commits mutations, implements other logic. At this level we should log error to sentry or dev console (depends on current environment) and throws it forward.
```javascript
try {
	// some actions
} catch (err) {
	// here 'err' is instance of ApiError, so it contain error code in addition
	// to standart Error properties. Here some logic based on error code
	// can be implemented.
	if (err.code === ApiError.NO_PERMISSIONS) {
		// do something
	}
	// log via $logger
	this.$logger(
		// detailed message about what exactly happened
		new Error('Something unexpected happened'),
		// extra data (in most cases should contain error object
		// and additional data to easy debug an exception)
		{ err },
		// tags (in most cases should specify the module where error happened)
		{ module: 'exampleModule' }
	)
	// throw it forward
	throw err
}
```
Implementation of **$logger** and its setup described in details <a href="#s">here</a>
### View Layer
Calls store action, uses the result of it. This is the final stage of error handling. At first, we must understand in which way we want to handle an error. Here can be a lot of cases, but most often you want to show some notification to user.
```javascript
async created () {
	try {
		await this.fetchUsers()
	} catch (err) {
		this.$notify({
			type: 'error',
			message: 'Users can not be fetched :('
		})
	}
}
```
Implementation of notification in details described <a href="#">here</a>

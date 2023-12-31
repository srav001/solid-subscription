# solid-subscription

This Vue package provides a simple way to create reactive subscriptions that can be used to observe changes to a value and execute a list of subscribers when the value changes. It also includes methods to mutate the value and trigger subscribers manually.

The `useSubscription` function takes an initial value and returns an object with a reactive value of the initial value passed in, and a subscriber can be added to be executed when the value is changed.

---

## Installation and Import

To use this package, you can install it via npm:

```sh
// In your console
npm install @solid-subscription
```

```typescript
// In your file
import { useSubscription } from '@solid-subscription';
const $mySubscription = useSubscription('hello'); // Type will be string
```

---

## API

### $value / $get()

This property/method returns the current value of the subscription.

```typescript
const value = $mySubscription.$value;
const value = $mySubscription.$get();
```

### $value = val / $set(val)

This property/method sets the current value of the subscription.

```typescript
$mySubscription.$value = 42;
$mySubscription.$set(42);
```

The $set method can also accept a mutator function that takes the current value as an argument and returns the new value:

```typescript
$mySubscription.$set(value => value + 1);
```

### $read

This is a read-only version of the subscription value. It wraps the subscription in a readonly ref.

```typescript
const readonlySubscription = $mySubscription.$read;
console.log(readonlySubscription.value);
```

### $addSub

This method adds a subscriber to the subscription. A subscriber is a function that takes the new value as an argument and is executed whenever the value changes. The subscriber can be `async`

```typescript
function logValue(value) {
	console.log(`New value: ${value}`);
}

$mySubscription.$addSub(logValue);
$mySubscription.$deleteSub(subscriber);
```

### $deleteSub

This method removes a subscriber from the subscription.

```typescript
subscription.$deleteSub(logValue);
```

### $triggerSubs

This method manually triggers all subscribers to the subscription.

```typescript
subscription.$triggerSubs();
```

### $mutate

This method mutates the subscription value with a mutator function. The mutator function takes the current value as an argument and returns the new value.

```typescript
subscription.$mutate(value => {
	value.name = 'John';
	return value;
});
```

---

## Usage

### Basic Example

```typescript
const $mySubscription = useSubscription('hello'); // Type will be string

// Get the current value
console.log($mySubscription.$value); // 'hello'

// Subscribers can `async`
async function mySubscriber(value: string) {
	return new Promise((resolve, reject) => {
		setTimeout(() => {
			console.log(`The value is now: ${value}`);
		}, 1);
	});
}

// Add a subscriber
$mySubscription.$addSub(mySubscriber);
// Manually trigger the subscribers if needed(rarely)
$mySubscription.$triggerSubs(); // 'The value is now: hello'

// Set the value
$mySubscription.$value = 'world';

// Subscriber runs here -  'The value is now: world'

// Remove a subscriber (can be used in Unmount, beforeRouteLeave etc)
$mySubscription.$deleteSub(mySubscriber);

// Use the readonly version of the value
const myReadonlyValue = $mySubscription.$read;
console.log(myReadonlyValue.value); // 'world'
```

### Complex state

Example uses a complex objects which won't be tracked deeply by default. Unless the subscriber is used in templates, watch, watchEffect and computed you don't need to add the deep flag.

```typescript
const $mySubscription = useSubscription(
	{
		user: {
			name: 'John',
			isActive: false
		}
	},
	// You can pass `true` as the deep flag to make the subscription deeply reactive if used in templates
	true
);
// Add a subscriber
$mySubscription.$addSub(data => {
	console.log(`The data is now: ${JSON.stringify(data)}`);
});

function myMutator(data: typeof $mySubscription.$value) {
	data.user.isActive = true;
	return data;
}

// Trigger the subscribers
$mySubscription.$triggerSubs(); // 'The data is now: { user: { name: 'John', isActive: false }}'

function tester() {
	// Mutate the value (only works if the value is an object)
	$mySubscription.$mutate(myMutator);
	// Subscriber runs here -  'The data is now: { user: { name: 'John', isActive: true }}'
}
tester();
```

### Destructured

You can also destructure the properties to have a seperate getter and setter.

```typescript
const { $get, $set, $read, $addSub } = useSubscription('hello');

// Get the current value
console.log($get()); // 'hello'

function mySubscriber(value: string) {
	console.log(`The value is now: ${value}`);
}

// Add a subscriber
$addSub(mySubscriber);

// Set the value
$set('world');

// Subscriber runs here -  'The value is now: world'

$set(val => `Hello ${val}`);
// Subscriber runs here -  'The value is now: Hello world'

// Use the readonly version of the value
console.log($read.value); // 'Hello world'
```

---

## Type definition

### Function Signature

```typescript
function useSubscription<T>(
	value: T,
	deep?: boolean
): {
	$value: T;
	$get: () => T;
	$set: (value: T | ((value: T) => T)) => void;
	$read: Readonly<Ref<T>>;
	$addSub: (subscriber: (value: T) => Promise<void> | void) => void;
	$deleteSub: (subscriber: (value: T) => Promise<void> | void) => void;
	$triggerSubs: () => void;
	$mutate: (mutator: (value: T) => T) => void;
};
```

### Arguments

value - The initial value of the subscription.
deep (optional) - Whether to create a shallow or deep reactive subscription. Defaults to false.
Return Value
An object with the following properties:

- $value - The current value of the subscription.
- $get() - A function that returns the current value of the subscription.
- $set(value: T | ((value: T) => T)) - A function that sets the value of the subscription. If a function is passed, it will receive the current value of the subscription as its argument and should return the new value.
- $read - A readonly reactive reference to the current value of the subscription.
- $addSub(subscriber: (value: T) => Promise<void> | void)) - A method for adding a subscriber to the subscription. It can be `async`. The subscriber is a function that will be executed whenever the value of the subscription changes. It can take the new value of the subscription as its argument.
- $deleteSub(subscriber: (value: T) => Promise<void> | void)) - A method for removing a subscriber from the subscription.
- $triggerSubs() - A method for manually triggering all subscribers. This should rarely be necessary.
- $mutate(mutator: (value: T) => T) - A method for updating the value of the subscription with a function that takes the current value as its argument and returns the new value. This should only be used for updating complex objects.

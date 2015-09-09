# Motivating Examples

## Simple Array of Models

```javascript
// @type Person = { name: String, age: Number }

// @assume fetchPeople : () => Promise(Array(Person), HttpError)
function fetchPeople () {
  return fetch('http://api.example.com/people');
}

// In this example fetchPeople is the anchor function, making
// type annotations unnecessary for the rest of the program!
fetchPeople()
  .then(function(people) {
    var names = people.map(p => p.name);
    var ageSum = people.map(p => p.age).reduce(add);

    // Won't pass: Person has no `hobby` property
    // var hobbies = people.map(p => p.hobby);

    // Creating an array in this fashion indicates a Tuple type.
    return Array(names.length, ageSum);
  })
  .then(function([nameCount, ageSum]) {
    console.log("# of names:", nameCount);
    console.log("Sum of ages:", ageSum);

    // Won't pass: number is not a function
    // ageSum()
  });

function add (x, y) { return x + y }
```


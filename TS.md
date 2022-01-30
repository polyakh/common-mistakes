####  [Using type assertions instead of type declarations](https://blog.softwaremill.com/typescript-mistakes-to-avoid-d3ab240c90eb#55e2)
```typescript
const dog: Dog = { breed: 'beagle' };   // first example we are telling Typescript to check if our variable complies with the specified type
const dog = { breed: 'beagle' } as Dog. // second one we tell the type checker that we know better and it can skip the check
```
It is then similar to any in a way it offers us an escape hatch from the type system. It is, however, a bit less dangerous because we will get an error if we try to assert something unreasonable, i.e. writing undefined as string

#### Using any instead of unknown
Best cases => external input or a call to a backend API. And these are the places where unknown type fits best.
```typescript
const isSomeType = (value: unknown): value is SomeType =>
  value.hasOwnProperty('a');
if (isSomeType(a)) {
  someFn(a); // this is fine
}
const isBook =(val: unknown): val is Book => typeof(val) === 'object' && val !== null &&
'name' in val && 'author' in val;

const processValue = (val: unknown) => {
    if (isBook(val)) {
        val; // Type is Book
    }
}
```
The unknown type is a type-safe alternative to any. Use it when you know you have a value but do not know what its type is.

#### [Unnecessarily iterating over object properties](https://blog.softwaremill.com/typescript-mistakes-to-avoid-d3ab240c90eb#940f)
When we use Object.entries we are getting the same: string keys and any values. This means that iterating over object keys should be a tool that we don’t overuse.
```typescript
interface Dog {
  name: string;
  breed: string;
  age: number;
}
const run = (dog: Dog) => { 
    let k: keyof Dog;
    for(k in dog) { const val = dog[k];  /* string | number */ } 
};
```

#### Not specifying function return types
We always need to define types for function parameters.(any)
```typescript
const compareLength = (a: string, b: string) => {
  return a.length > b.length;
}
```
If we are dealing with more complex function we will be able to just have a look at it and have a pretty good idea what it does without diving into its body

#### [Redeclaring interfaces](https://betterprogramming.pub/7-typescript-common-mistakes-to-avoid-581c30e514d6#7a5a)
```typescript
interface Book {
  author?: string;
  numPages: number;
  price: number;
}
// ✅ Article is a Book without a Page
type Article = Omit<Book, 'numPages'>;
// ✅ We might need a readonly verison of the Book Type
type ReadonlyBook = Readonly<Book>;
// ✅ A Book that must have an author
type NonAnonymousBook = Omit<Book, 'author'> & Required<Pick<Book, 'author'>>;
```
Mapped Types can be also be applied to unions, as shown below:
```typescript
type animals = 'bird' | 'cat' | 'crocodile';
type mamals = Exclude<animals, 'crocodile'>;
// 'bird' | 'cat'
```

#### Not Relying on Type Inference
```typescript
const addNumber = (a: number, b: number) => a + b;
// ✅ result will be whatever the return type of addNumber function
// no need for us to redefine it
const processResult = (result: ReturnType<typeof addNumber>) => console.log(result);
processResult(addNumber(1, 1));
```
!! // ❌ Sometimes for one of objects it is not necessary to define interfaces for it
```typescript
interface Book {name: string,}
const book: Book = { author: 'Hemingway'}
const printBook = (bookInstance: Book) => console.log(bookInstance); // Book interface is used in a particular scenario and there’s no even a need to create an interface.
```
// ✅ For simple scenarios we can rely on type inference
```typescript
const book = { author: 'Hemingway' }
const printBook = (bookInstance: typeof book) => console.log(bookInstance); // By relying on TypeScript’s inference, the code becomes less cluttered with types and easier to read
```


#### Using the Function Type (it accepts any number and type of parameters. & the return type is always any.)
```typescript
// ❌ Avoid, parameters types and length are unknown. Return type is any
const onSubmit = (callback: Function) => callback(1, 2, 3);
// ✅ Preferred, the arguments and return type of callback is now clear
const onSubmit = (callback: () => Promise<unknown>) => callback();
```

#### Relying on a Third Party for Immutability
When using the functional programming paradigm, TypeScript can be of great help. 
It provides all the necessary tooling to make sure we don’t mutate our objects.
```typescript
// ✅ declare properties as readonly
interface Person { readonly name: string; }
// ✅ implicitely declaring a readonly arrays
const x = [1,2,3,4,5] as const;
// ✅ explicitely declaring a readonly array
const y: ReadonlyArray<{ x: number, y: number}> = [ {x: 1, y: 1}]
interface Address { street: string; }

// ✅ converting all the type properties to readonly
type ReadonlyAddress = Readonly<Address>;
```
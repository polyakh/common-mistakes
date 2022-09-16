[#### **Better validations with `includes` and selector function](https://obaranovskyi.medium.com/top-5-techniques-in-typescript-to-bring-your-code-to-the-next-level-6f20be543b39#7cec)
We have a function that compares the same object property with multiple values of the same type and returns true in case
if at least one statement is positive.

```typescript
// ❌ Problem:
enum UserStatus {
    Administrator = 1,
    Editor = 4,
}

interface User {
    firstName: string;
    lastName: string;
    status: UserStatus;
}

const isEditActionAvailable = (user: User): boolean => (user.status === UserStatus.Administrator ||
    user.status === UserStatus.Editor);

// ✅ Solution:
enum TeamStatus {
    Lead = 1,
    Manager = 2,
}

interface TeamUser extends User {
    teamStatus: TeamStatus;
}

function roleCheck<D, T>(selector: (data: D) => T, roles: T[]): (value: D) => boolean {
    return (value: D) => roles.includes(selector(value));
}

const MANAGER_OR_LEAD = [
    TeamStatus.Lead,
    TeamStatus.Manager
]

const isManagerOrLead = roleCheck((user: User) => user.teamStatus, MANAGER_OR_LEAD);
```

[#### Use callbacks to encapsulate code that changes](https://obaranovskyi.medium.com/top-5-techniques-in-typescript-to-bring-your-code-to-the-next-level-6f20be543b39#f5d8)
We have multiple functions that are pretty similar, with a minor difference.

```typescript
async function makeUserAction(fn: Function): Promise<void> {
    LoadingService.startLoading();
    await fn();
    LoadingService.stopLoading();
    UserGrid.reloadData();
}

async function createUser2(user: User): Promise<void> {
    makeUserAction(() => userHttpClient.createUser(user));
}

async function updateUser2(user: User): Promise<void> {
    makeUserAction(() => userHttpClient.updateUser(user));
}
```

[#### Consider using predicate combinators(combinator predicates with factories)](https://obaranovskyi.medium.com/top-5-techniques-in-typescript-to-bring-your-code-to-the-next-level-6f20be543b39#7cec)

```typescript
const isEditor = (user: User) => user.role === UserRole.Editor;
const isGreaterThan17 = (user: User) => user.age > 17;
const isRole = (role: UserRole) =>
    (user: User) => user.role === role;
const isWriter = isRole(UserRole.Writer)
const greaterThan17AndWriterOrEditor = users.filter(
    and(isGreaterThan(17), or(isWriter, isRole(UserRole.Editor)))
);

```

[#### Types vs. interfaces in TypeScript](https://blog.logrocket.com/types-vs-interfaces-in-typescript/)
Interface work better with objects and method objects, and types are better to work with functions, complex types, etc.
Type:

```typescript
type Person = {
    name: string,
};
type ReturnPerson = (
    person: Person
) => Person;

const returnPerson: ReturnPerson = (person) => {
    return person;
};
```

Extends and implements, Declaration merging:

```typescript
interface Song {
    artistName: string;
};

interface Song {
    songName: string;
};

const song: Song = {
    artistName: "Freddie",
    songName: "The Chain"
};

class Car {
    printCar = () => {
        console.log("this is my car")
    }
};

interface NewCar extends Car {
    name: string;
};

class NewestCar implements NewCar {
    name: "Car";

    constructor(engine: string) {
        this.name = name
    }

    printCar = () => {
        console.log("this is my car")
    }
};
```

#### [Using type assertions instead of type declarations](https://blog.softwaremill.com/typescript-mistakes-to-avoid-d3ab240c90eb#55e2)

```typescript
const dog: Dog = {breed: 'beagle'};   // first example we are telling Typescript to check if our variable complies with the specified type
const dog = {breed: 'beagle'} as Dog; // second one we tell the type checker that we know better and it can skip the check
```

It is then similar to any in a way it offers us an escape hatch from the type system. It is, however, a bit less
dangerous because we will get an error if we try to assert something unreasonable, i.e. writing undefined as string

#### Using any instead of unknown

Best cases => external input or a call to a backend API. And these are the places where unknown type fits best.

```typescript
const isSomeType = (value: unknown): value is SomeType =>
    value.hasOwnProperty('a');
if (isSomeType(a)) {
    someFn(a); // this is fine
}
const isBook = (val: unknown): val is Book => typeof (val) === 'object' && val !== null &&
    'name' in val && 'author' in val;

const processValue = (val: unknown) => {
    if (isBook(val)) {
        val; // Type is Book
    }
}
```

The unknown type is a type-safe alternative to any. Use it when you know you have a value but do not know what its type
is.

#### [Unnecessarily iterating over object properties](https://blog.softwaremill.com/typescript-mistakes-to-avoid-d3ab240c90eb#940f)

When we use Object.entries we are getting the same: string keys and any values. This means that iterating over object
keys should be a tool that we don’t overuse.

```typescript
interface Dog {
    name: string;
    breed: string;
    age: number;
}

const run = (dog: Dog) => {
    let k: keyof Dog;
    for (k in dog) {
        const val = dog[k];  /* string | number */
    }
};
```

#### Not specifying function return types

We always need to define types for function parameters.(any)

```typescript
const compareLength = (a: string, b: string) => {
    return a.length > b.length;
}
```

If we are dealing with more complex function we will be able to just have a look at it and have a pretty good idea what
it does without diving into its body

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

!! ❌ Sometimes for one of objects it is not necessary to define interfaces for it

```typescript
interface Book {
    name: string,
}

const book: Book = {author: 'Hemingway'}
const printBook = (bookInstance: Book) => console.log(bookInstance); // Book interface is used in a particular scenario and there’s no even a need to create an interface.
```

✅ For simple scenarios we can rely on type inference

```typescript
const book = {author: 'Hemingway'}
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

When using the functional programming paradigm, TypeScript can be of great help. It provides all the necessary tooling
to make sure we don’t mutate our objects.

```typescript
// ✅ declare properties as readonly
interface Person {
    readonly name: string;
}

// ✅ implicitely declaring a readonly arrays
const x = [1, 2, 3, 4, 5] as const;
// ✅ explicitely declaring a readonly array
const y: ReadonlyArray<{ x: number, y: number }> = [{x: 1, y: 1}]

interface Address {
    street: string;
}

// ✅ converting all the type properties to readonly
type ReadonlyAddress = Readonly<Address>;
```

[#### Use type inference for defining a component state or DefaultProps](https://dev.to/alexomeyer/10-must-know-patterns-for-writing-clean-code-with-react-and-typescript-1m0g)

```typescript
const defaultProps = Object.freeze({name: "John Doe"}); // In the code above, by freezing the DefaultProps or initialState the TypeScript type system can now infer them as readonly types.
type GreetOwnProps = { someProps: string } & typeof defaultProps;
const Greet = (props: GreetOwnProps) => {
    // etc
};
Greet.defaultProps = defaultProps;
```

[#### Consuming Props of a Component with defaultProps](https://react-typescript-cheatsheet.netlify.app/docs/basic/getting-started/default_props/#consuming-props-of-a-component-with-defaultprops)

```tsx
type ComponentProps<T> = T extends | React.ComponentType<infer P>
    | React.Component<infer P>
    ? JSX.LibraryManagedAttributes<T, P>
    : never;

const TestComponent = (props: ComponentProps<typeof GreetComponent>) => {
    return <h1 / >;
};

// No error
const el = <TestComponent name = "foo" / >;
```

[#### Don’t use method declaration within interface/type alias](https://dev.to/alexomeyer/10-must-know-patterns-for-writing-clean-code-with-react-and-typescript-1m0g#6-dont-use-method-declaration-within-interfacetype-alias)

```typescript
// Do
interface Counter {
    start: (count: number) => string
    reset: () => string
} // This ensures pattern consistency in our code as all members of type/inference are declared in the same way.
```

[#### Don’t use FunctionComponent](https://dev.to/alexomeyer/10-must-know-patterns-for-writing-clean-code-with-react-and-typescript-1m0g#7-dont-use-functioncomponent)
Using FC provides some advantages such as type-checking and autocomplete for static properties like displayName,
propTypes, and defaultProps. But it has a known issue of breaking defaultProps and other props: propTypes, contextTypes,
displayName.

FC also provides an implicit type for children prop which also have known issues. Also, as discussed earlier a component
API should be explicit so an implicit type for children prop is not the best.


[#### Type Refinement and Disjoint Unions](https://onesignal.com/blog/effective-typescript-for-react-applications/#type-refinement-and-disjoint-unions)
Support multiple variants of a shared interface.
Although this component compiles, the type definition of props doesn’t inform the TypeScript compiler when specialPrimaryMethod is permitted.
```typescript jsx
// ❌ Problem:
type ButtonKind = "primary" | "secondary";
interface Props extends React.ComponentPropsWithoutRef<"button"> {
  kind: ButtonKind;
  specialPrimaryMethod?: () => void;
}
// Correct use-case
<Button kind="primary" specialPrimaryMethod={doSpecial}>...

// Invalid use-case: specialPrimaryMethod shouldn't be optional
<Button kind="primary">...

// Invalid use-case: secondary shouldn't support specialPrimaryMethod
<Button kind="secondary" specialPrimaryMethod={doSpecial}>...
```

This is where the disjoint union comes in handy. By splitting apart the interfaces for the “primary” variant and the “secondary” variant, you can achieve better compile-time type-checking.
// ✅ Solution:
```typescript jsx
interface PrimaryButton {
  kind: "primary";
  specialPrimaryMethod: () => void;
}

interface SecondaryButton {
  kind: "secondary";
}
// Create a disjoint union
type Button = PrimaryButton | SecondaryButton;

// Add built-in HTML props to the disjoin union
type Props = React.ComponentPropsWithoutRef<"button"> & Button;
```

Links:

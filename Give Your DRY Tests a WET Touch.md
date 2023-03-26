For quite some time, the commonly accepted practice has been to write DRY code, but there are developers who suggest using WET code. The purpose of this article is to demonstrate how to combine DRY and WET code to write better test cases.

The tests are written in Jest for React, but the chose of test framework doesn't really matter. You can give your DRY tests a WET approach in any language or framework. The approach can even be used for unit tests as well as integration and e2e tests.

## DRY vs WET

First of all, let's shortly describe what DRY and WET code is for those who doesn't now about it. DRY (Don't Repeat Yourself) and WET (Write Everything Twice) are two terms used to describe different programming styles.

DRY code is more efficient and easier to maintain because it eliminates duplicate code. Changes made in one place affect all related code, making it easier to update and modify. DRY code is easier to understand and to reason about, which can improve code quality and reduce errors.

On the other side, WET code can be faster to write initially because it relies on copying and pasting existing code. WET code can be easier for less experienced programmers to understand because it is more explicit and detailed. In some cases, WET code may be more suitable when the code is simple and unlikely to change in the future.

## WET Unit Tests

This example shows three simple example test cases for updating a user object. This code is WET, since the initialization for the test data is copied into each an every test. In this short example, the only test setup data is the user object, but in more complex test cases it can consist of much more data than that.

```js
describe('Users', () => {
    test('increments Alice\'s age by 1 when she has a birthday', () => {
        const user = {
            name: 'Alice',
            age: 26,
            friends: ['Britney', 'Chili', 'Dennis']
        }

        haveABirthday(user)
        expect(user.age).toBe(27)
    })

    test('adds a new friend to the friend list', () => {
        const user = {
            name: 'Alice',
            age: 26,
            friends: ['Britney', 'Chili', 'Dennis']
        }

        makeANewFriend(user, 'Elliot')
        expect(user.friends).toContain('Elliot')
        expect(user.friends).toHaveLength(4)
    })

    test('removes a friend from the list when being stupid', () => {
        const user = {
            name: 'Alice',
            age: 26,
            friends: ['Britney', 'Chili', 'Dennis']
        }

        playRussianRouletteWithFriends(user)
        expect(user.friends).toHaveLength(2)
    })
})
```

## DRY Unit Tests

Although there wasn't anything wrong with the WET tests above, the test file can be much briefer if we lift out the initialization in a beforeEach function, which will run before each test case.

For a file with tens of tests, that change will save a lot of lines of code and make the whole test file easier to read. And even each test itself can get a lot less bloated with the DRY approach. In this case, each test can be written in 4-5 lines instead of 10-11 lines as in the WET case.

```js
describe('Users', () => {
    let user

    beforeEach(() => {
        user = {
            name: 'Alice',
            age: 26,
            friends: ['Britney', 'Chili', 'Dennis']
        }
    })

    test('increments Alice\'s age by 1 when she has a birthday', () => {
        haveABirthday(user)
        expect(user.age).toBe(27)
    })

    test('adds a new friend to the friend list', () => {
        makeANewFriend(user, 'Elliot')
        expect(user.friends).toContain('Elliot')
        expect(user.friends).toHaveLength(4)
    })

    test('removes a friend from the list when being stupid', () => {
        playRussianRouletteWithFriends(user)
        expect(user.friends).toHaveLength(2)
    })
})
```

## The Problem With the DRY Tests

Many test files appear very similar to the previous DRY test file. However, programmers who prefer WET programming have a valid argument against writing tests in that way. The reason being that it's common for people to forget the data used to initialize their tests.

For instance, the first of our tests is the age test.

```js
test('increments Alice\'s age by 1 when she has a birthday', () => {
    haveABirthday(user)
    expect(user.age).toBe(27)
})
```

Now, without scrolling up to look at the test data, answer how old Alice were before having a birthday?

![age-meme](https://res.cloudinary.com/practicaldev/image/fetch/s--wjdN9F0L--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://www.perssondennis.com/images/articles/give-your-dry-tests-a-wet-touch/i-am-not-old.webp)

*I promise, it's not a bug!*

Well, if you know math better than meth, you can probably calculate that she must have been 26 before her birthday. But you cannot be sure that the haveABirthday function works, that's the very reason you are writing that test.

To be able to see how old she really were before the birthday function was invoked, you would have to scroll up to the definition of the user variable in the beforeEach at the top of the file. Doing that over and over again can be quite annoying, and if you have written many DRY tests, you most probably get what I mean.

Furthermore, if you would change the age in the beforeEach, you have no idea how many tests you would to fail.

Now tell me, how can we circumvent issues like those? One way to do that is to add a WET touch to your DRY tests!

## DRY Tests With a WET Touch

With DRY tests with a WET touch, we get rid of the unexpressive beforeEach function and instead write some initialization functions. By naming the functions appropriately and specifically using them in each test case, we can achieve both readable and brief tests in which we have all the information we need.

```js
const createTwentySixYearsOldUser = () => {
    return {
        name: 'Alice',
        age: 26,
    }
}

const createUserWithThreeFriends = () => {
    return {
        name: 'Alice',
        friends: ['Britney', 'Chili', 'Dennis']
    }
}

describe('Users', () => {
    test('increments Alice\'s age by 1 when she has a birthday', () => {
        const user = createTwentySixYearsOldUser()
        haveABirthday(user)
        expect(user.age).toBe(27)
    })

    test('adds a new friend to the friend list', () => {
        const user = createUserWithThreeFriends()
        makeANewFriend(user, 'Elliot')
        expect(user.friends).toContain('Elliot')
        expect(user.friends).toHaveLength(4)
    })

    test('removes a friend from the list when being stupid', () => {
        const user = createUserWithThreeFriends()
        playRussianRouletteWithFriends(user)
        expect(user.friends).toHaveLength(2)
    })
})
```

## DRY Tests With a WET Touch Using Build Design Pattern

The approach to write DRY tests with a WET touch should be enough for most fairly small test cases. But if you are one of those who like structure, or if you are using the same data to initialize plenty of tests, you could go the bit more cumbersome way and initialize your data with the build design pattern.

In that case, your can write a class for a basic user, then add builder functions to add attributes to it. This one is written as a JavaScript class, but you can do the same thing using regular functions.

```js
class User {
  constructor(name) {
    this.name = name
    this.age = null
    this.friends = []
  }

  setAge(age) {
    this.age = age
    return this
  }

  addFriends(friends) {
    this.friends = this.friends.concat(friends)
    return this
  }
}
```

When you have a class like that, you can write test like this. And hopefully, you can even reuse it for multiple files.

```js
describe('Users', () => {
    test('increments Alice\'s age by 1 when she has a birthday', () => {
        const user = new User('Alice').setAge(26)
        haveABirthday(user)
        expect(user.age).toBe(27)
    })

    test('adds a new friend to the friend list', () => {
        const friends = ['Britney', 'Chili', 'Dennis']
        const user = new User('Alice').addFriends(friends)
        makeANewFriend(user, 'Elliot')
        expect(user.friends).toContain('Elliot')
        expect(user.friends).toHaveLength(4)
    })

    test('removes a friend from the list when being stupid', () => {
        const friends = ['Britney', 'Chili', 'Dennis']
        const user = new User('Alice').addFriends(friends)
        playRussianRouletteWithFriends(user)
        expect(user.friends).toHaveLength(2)
    })
})
```

With this new improved test file, each test is only 5-7 lines, almost as thin as the 4-5 DRY lines and much thinner than the 10-11 WET lines. We do have all necessary information we need to see in each and every test, but we have cut out all the extra unnecessary information we had in the verbose WET tests.

Other developers at your company can look at these new brief tests and quickly understand what they do and if they work as intended. If they need to change any of the tests, they can simply do that right within the test. With the DRY tests, they would need to alter the data initialized in the beforeEach function, which potentially could destroy some of the other tests in the file, or in any other file if the test data is shared between several files.

## Conclusion

WET tests quickly gets very bloated. By making them DRY, they get more comprehensible, but we risk to lose information in the test, making it necessary to scroll to definitions or open definitions in new files to see the data we are working with in the test. Altering that common data also have the potential to ruin plenty of other test cases.

By combining the advantages of DRY and WET tests, we can write short readable tests where we can see all necessary information directly in the test without having to scroll or look up test data.

The key to doing that, is to write functions for the data initialization and use those functions in each test. For even more structure, when dealing with bigger amount of data and number of tests, one can use the builder pattern to customize the data on detail level for each test.

For non-trivial test examples, it will make a big difference in readability. For whole projects with much data, it will make a huge improvement.
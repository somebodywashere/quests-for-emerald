# Chapter 3
## Day 1

#### 1. In words, list 3 reasons why structs are different from resources.

Structs are just containers of data.
Resources are extremely secure containers of data that:
- Can't be copied or overwritten
- Can't be created outside of the contract
- Hard to lose and easy to track because of how we handle them (primarily move)

#### 2. Describe a situation where a resource might be better to use than a struct.

The good example is creating an NFT. Being it a resource rather then struct creates additional layer of security and clarity.

#### 3. What is the keyword to make a new resource?

'create'

#### 4. Can a resource be created in a script or transaction (assuming there isn't a public function to create one)?

No, it can't. Look at the reason №2 on first topic :)

#### 5. What is the type of the resource below?

'@Jacob'

#### 6. Let's play the "I Spy" game from when we were kids. I Spy 4 things wrong with this code. Please fix them.

##### Original code:

```cadence
pub contract Test {

    // Hint: There's nothing wrong here ;)
    pub resource Jacob {
        pub let rocks: Bool
        init() {
            self.rocks = true
        }
    }

    pub fun createJacob(): Jacob { // there is 1 here
        let myJacob = Jacob() // there are 2 here
        return myJacob // there is 1 here
    }
}
```

##### Fixed code:

```cadence
pub contract Test {

    pub resource Jacob {
        pub let rocks: Bool
        init() {
            self.rocks = true
        }
    }

    pub fun createJacob(): @Jacob { 
        let myJacob <- create Jacob() 
        return <- myJacob 
    }
}
```
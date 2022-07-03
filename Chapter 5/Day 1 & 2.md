# Chapter 5
## Day 1

#### 1. Describe what an event is, and why it might be useful to a client.

Event is a way to point out to everything outside of the contract that something happened inside the contract. It might be useful because we can code something (for example website) that will get use of it.

#### 2. Deploy a contract with an event in it, and emit the event somewhere else in the contract indicating that it happened.

#### 3. Using the contract in step 2), add some pre conditions and post conditions to your contract to get used to writing them out.

```cadence
pub contract Playerz {

    pub var totalSupply: UInt64
    pub let maxSupply: UInt64
    pub var isMintable: Bool

    pub event Minted(id: UInt64)
    pub event Deposited(id: UInt64)
    pub event Withdrawn(id: UInt64)
    pub event Burned(id: UInt64)

    pub resource NFT {
        pub let id: UInt64

        init() {
            self.id = self.uuid
            emit Minted(id: self.id)
        }

        destroy() {
            emit Burned(id: self.id)
            Playerz.totalSupply = Playerz.totalSupply - 1
        }
    }

    pub fun mint(): @NFT {
        pre {
            self.totalSupply < self.maxSupply: "Reached maximum supply"
            self.isMintable: "Mint halted"
        }
        post {
            before(self.totalSupply) == self.totalSupply - 1: "Error"
        }
        self.totalSupply = self.totalSupply + 1
        return <- create NFT()
    }

    pub resource interface CollectionPublic {
        pub fun deposit(token: @NFT)
        pub fun getIDs(): [UInt64]
    }

    pub resource Collection: CollectionPublic {
        pub var ownedNFTs: @{UInt64: NFT}

        pub fun deposit(token: @NFT) {
            emit Deposited(id: token.id)
            self.ownedNFTs[token.id] <-! token
        }

        pub fun withdraw(withdrawID: UInt64): @NFT {
            let nft <- self.ownedNFTs.remove(key: withdrawID) 
                ?? panic("This NFT does not exist in this Collection.")
            emit Withdrawn(id: nft.id)
            return <- nft
        }

        pub fun getIDs(): [UInt64] {
            return self.ownedNFTs.keys
        }

        init() {
            self.ownedNFTs <- {}
        }

        destroy() {
            destroy self.ownedNFTs
        }
    }

    pub fun createEmptyCollection(): @Collection {
        return <- create Collection()
    }

    init() {
        self.totalSupply = 0
	    self.maxSupply = 1000
        self.isMintable = true
    }
}
```

#### 4. For each of the functions below (numberOne, numberTwo, numberThree), follow the instructions.

```cadence
pub contract Test {

  // TODO
  // Tell me whether or not this function will log the name.
  // name: 'Jacob'
  pub fun numberOne(name: String) {
    pre {
      name.length == 5: "This name is not cool enough."
    }
    log(name)
  }
  // Yes it will log because length of Jacob is 5 so we met pre condition.

  // TODO
  // Tell me whether or not this function will return a value.
  // name: 'Jacob'
  pub fun numberTwo(name: String): String {
    pre {
      name.length >= 0: "You must input a valid name."
    }
    post {
      result == "Jacob Tucker"
    }
    return name.concat(" Tucker")
  }
  // Yes it will return a value because both pre and post conditions are met.

  pub resource TestResource {
    pub var number: Int

    // TODO
    // Tell me whether or not this function will log the updated number.
    // Also, tell me the value of `self.number` after it's run.
    pub fun numberThree(): Int {
      post {
        before(self.number) == result + 1 //** 0 == 1 + 1 - that's incorrect
      }
      self.number = self.number + 1
      return self.number
    }
	// It won't log updated number because we don't have log function. And it won't return anything, because we don't met post condition (marked with ** above). So self.number will remain unchanged '0'.

    init() {
      self.number = 0
    }

  }

}
```

## Day 2

#### 1. Explain why standards can be beneficial to the Flow ecosystem.

Standards can be beneficial because it can be used as "rules" for some type of contracts. If we implement some contract of fungible token, we want it to to use flow standard 'FungibleToken', so we can easily add it to DEx.

#### 2. What is YOUR favourite food?

Ehm, burgers? Yeah, I know, that's not very healthy.

#### 3. Please fix this code (Hint: There are two things wrong):

##### The contract interface:

```cadence

pub contract interface ITest {
  pub var number: Int
  
  pub fun updateNumber(newNumber: Int) {
    pre {
      newNumber >= 0: "We don't like negative numbers for some reason. We're mean."
    }
    post {
      self.number == newNumber: "Didn't update the number to be the new number."
    }
  }

  pub resource interface IStuff {
    pub var favouriteActivity: String
  }

  pub resource Stuff: IStuff {
    pub var favouriteActivity: String
  }
}

```

##### The implementing contract:

```cadence
import ITest from 0x01 			// Added import
pub contract Test: ITest { 		// We forgot to add contract interface ITest
  pub var number: Int
  
  pub fun updateNumber(newNumber: Int) {
    self.number = newNumber // We should input '5' as a newNumber, otherwise post condition of the interface contract won't never meet
  }

	//removed unnecessary implementation of IStuff

  pub resource Stuff: ITest.IStuff { // Added resource interface from the ITest
    pub var favouriteActivity: String

    init() {
      self.favouriteActivity = "Playing League of Legends."
    }
  }

  init() {
    self.number = 0
  }
}

```
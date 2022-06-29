# Chapter 4
## Day 3

#### 1. Why did we add a Collection to this contract? List the two main reasons.

First, because if we wanted to have multiple NFTs even from the same collection we would needed to store every NFT in a new path. So to solve this we added a Collection, so we can have every NFT in the same paths with their unique id.
Secondly, because only owner of the account (myself) could have minted an NFT. But what if I want to be gifted with NFT? So we created a Collection for that reason aswell.

#### 2. What do you have to do if you have resources "nested" inside of another resource? ("Nested resources")

We need to add 'destroy()' function where we will destroy our nested resource on destroy of the "main" resource.

#### 3. Brainstorm some extra things we may want to add to this contract. Think about what might be problematic with this contract and how we could fix it.

##### - Idea #1: Do we really want everyone to be able to mint an NFT? 🤔.

Probably not. We can implenemt some admin role or "whitelist" if needed and we should limit our NFT contract. Let's add this limit.

```cadence

pub contract CryptoPoops {

  pub var totalSupply: UInt64
  pub let maxSupply: UInt64
  pub var isMintable: Bool

  pub resource NFT {
    pub let id: UInt64

    init() {
      self.id = self.uuid
    }

    destroy() {
    CryptoPoops.totalSupply = CryptoPoops.totalSupply - 1}
  }

  pub fun createNFT(): @NFT {
    if (self.totalSupply < self.maxSupply) && self.isMintable {
      self.totalSupply = self.totalSupply + 1
      return <- create NFT()
    } 
    else {
      return panic("Reached maximum supply or mint halted")
    }
    
  }

  // Only exposes `deposit` and `getIDs`
  pub resource interface CollectionPublic {
    pub fun deposit(token: @NFT)
    pub fun getIDs(): [UInt64]
  }

  // `Collection` implements `CollectionPublic` now
  pub resource Collection: CollectionPublic {
    pub var ownedNFTs: @{UInt64: NFT}

    pub fun deposit(token: @NFT) {
      self.ownedNFTs[token.id] <-! token
    }

    pub fun withdraw(withdrawID: UInt64): @NFT {
      let nft <- self.ownedNFTs.remove(key: withdrawID) 
              ?? panic("This NFT does not exist in this Collection.")
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
    self.totalSupply = 4998 //4998 was used for testing, should be 0
	self.maxSupply = 5000
    self.isMintable = true
  }
}

```

##### - Idea #2: If we want to read information about our NFTs inside our Collection, right now we have to take it out of the Collection to do so. Is this good?

No, we should create function inside Collection to get reference of the NFT inside a Collection :)


## Day 4

#### Take our NFT contract so far and add comments to every single resource or function explaining what it's doing in your own words:

```cadence

pub contract CryptoPoops {
  pub var totalSupply: UInt64

  // This is an NFT resource that contains an id, name,
  // favouriteFood, and luckyNumber
  pub resource NFT {
    pub let id: UInt64

    pub let name: String
    pub let favouriteFood: String
    pub let luckyNumber: Int

    init(_name: String, _favouriteFood: String, _luckyNumber: Int) {
      self.id = self.uuid

      self.name = _name
      self.favouriteFood = _favouriteFood
      self.luckyNumber = _luckyNumber
    }
  }

  // This is a resource interface that allows us to deposit, get IDs and get a reference of the NFTs inside collection. We take reference to look our NFT's fields (name, favouriteFood and luckyNumber)
  pub resource interface CollectionPublic {
    pub fun deposit(token: @NFT)
    pub fun getIDs(): [UInt64]
    pub fun borrowNFT(id: UInt64): &NFT
  }

  // This is a Collection resource that contains dictionary of NFT's and their IDs and some functions.
  pub resource Collection: CollectionPublic {
    pub var ownedNFTs: @{UInt64: NFT}
	
	// Deposit function to the dictionary of all owned NFTs inside collection.
    pub fun deposit(token: @NFT) {
      self.ownedNFTs[token.id] <-! token
    }
	
	// Withdraw function from dictionary
    pub fun withdraw(withdrawID: UInt64): @NFT {
      let nft <- self.ownedNFTs.remove(key: withdrawID) 
              ?? panic("This NFT does not exist in this Collection.")
      return <- nft
    }
	
	// Return all IDs of owned NFTs in collection
    pub fun getIDs(): [UInt64] {
      return self.ownedNFTs.keys
    }
	
	// Return reference of NFT at some id
    pub fun borrowNFT(id: UInt64): &NFT {
      return (&self.ownedNFTs[id] as &NFT?)!
    }

    init() {
      self.ownedNFTs <- {}
    }

    destroy() {
      destroy self.ownedNFTs
    }
  }
  
  // Function to create empty collection and return it
  pub fun createEmptyCollection(): @Collection {
    return <- create Collection()
  }
  
  // Minter resource that needed to be able to mint new NFTs.
  pub resource Minter {

    pub fun createNFT(name: String, favouriteFood: String, luckyNumber: Int): @NFT {
      return <- create NFT(_name: name, _favouriteFood: favouriteFood, _luckyNumber: luckyNumber)
    }

    pub fun createMinter(): @Minter {
      return <- create Minter()
    }

  }

  init() {
    self.totalSupply = 0
    self.account.save(<- create Minter(), to: /storage/Minter)
  }
}


```
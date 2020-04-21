# pychain
Making a simple blockchain using python.

The first step is to decide the block structure. To keep things a little bit simple I use: index, timestamp, transactions, hash and previous hash.

We define a function to compute the hash of a block as it needs to be hashed to keep integrity of the data.
```Python
def compute_hash(self):
        """
        A function that return the hash of the block contents.
        """
        block_string = json.dumps(self.__dict__, sort_keys=True)
        return sha256(block_string.encode()).hexdigest()
```
BOTH of these are inside class Block.

Then we define another class Blockcahin
We create a function to  generate a block. We must know the hash of the previous block and create the rest of the required content (= index, hash, transactions, timestamp).
```Python
 def add_block(self, block):

        block.index=len(self.chain) + 1
        block.timestamp=time()
        block.transactions=self.current_transactions
        block.previous_hash= previous_hash or self.hash(self.chain[-1])
        
        self.chain.append(block)
        return True
```
the function stores block by appending it to the genesis block which we create by defining a function as:
```Python
def create_genesis_block(self):
        """
        A function to generate genesis block and appends it to
        the chain. The block has index 0, previous_hash as 0, and
        a valid hash.
        """
        genesis_block = Block(0, [], 0, "0")
        genesis_block.hash = genesis_block.compute_hash()
        self.chain.append(genesis_block)
```
Now at any given time we must be able to validate if a block or a chain of blocks are valid . This is true especially when we receive new blocks from other nodes or peers and must decide whether to accept them or not.As we are not using PoW or PoS, we create a simple of our own to check this.
```Python
    def check_chain_validity(add_block, last_block):
    
        if (last_block.index + 1 != add_block.index) :
            print('invalid index')
            return False
        elif (last_block.hash != add_block.previousHash): 
            print('invalid previoushash')
            return False
        elif (Block.compute_hash(add_block) != add_block.hash): 
            print('invalid hash: ' + Block.compute_hash(add_block) + ' ' + add_block.hash)
            return False
        else:
            return True
```


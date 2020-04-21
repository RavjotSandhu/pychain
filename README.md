# pychain
A very simple implementaion of blockchain using python.

The first step is to decide the block structure. To keep things simple I use: index, timestamp, data, hash and previous hash.

We define a function to compute the hash of a block
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

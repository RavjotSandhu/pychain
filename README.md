# pychain
Making a simple blockchain using python, and following functionality to it.
    1.List all blocks
    2.Create a new block with a content given by the user
    3.List or add peers

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

We use flask, a micro web framework, for communicating with other nodes
We do routing for different tasks and in addition we pass second argument to decorator which are some of accepted HTTP route methods.

For transctions after additions of new blocks or peers a  function is defined as follows: 
```Python       
        @app.route('/new_transaction', methods=['POST'])
def new_transaction():
    tx_data = request.get_json()
    required_fields = ["author", "content"]

    for field in required_fields:
        if not tx_data.get(field):
            return "Invalid transaction data", 404

    tx_data["timestamp"] = time.time()

    blockchain.add_new_transaction(tx_data)

    return "Success", 201
```
To add more peers or nodes we do this:
```Python
@app.route('/register_node', methods=['POST'])
def register_new_peers():
    node_address = request.get_json()["node_address"]
    if not node_address:
        return "Invalid data", 400

    # Add the node to the peer list
    peers.add(node_address)

    # Return the consensus blockchain to the newly registered node
    # so that he can sync
    return get_chain()
   ``` 
   We the route our app to register them and regenerate blockchain:
   ```Python
   
@app.route('/register_with', methods=['POST'])
def register_with_existing_node():
    """
    Internally calls the `register_node` endpoint to
    register current node with the node specified in the
    request, and sync the blockchain as well as peer data.
    """
    node_address = request.get_json()["node_address"]
    if not node_address:
        return "Invalid data", 400

    data = {"node_address": request.host_url}
    headers = {'Content-Type': "application/json"}

    # Make a request to register with remote node and obtain information
    response = requests.post(node_address + "/register_node",
                             data=json.dumps(data), headers=headers)

    if response.status_code == 200:
        global blockchain
        global peers
        # update chain and the peers
        chain_dump = response.json()['chain']
        blockchain = create_chain_from_dump(chain_dump)
        peers.update(response.json()['peers'])
        return "Registration successful", 200
    else:
        # if something goes wrong, pass it on to the API response
        return response.content, response.status_code


def create_chain_from_dump(chain_dump):
    generated_blockchain = Blockchain()
    generated_blockchain.create_genesis_block()
    for idx, block_data in enumerate(chain_dump):
        if idx == 0:
            continue  # skip genesis block
        block = Block(block_data["index"],
                      block_data["transactions"],
                      block_data["timestamp"],
                      block_data["previous_hash"],
                     )
        
        added = generated_blockchain.add_block(block)
        if not added:
            raise Exception("The chain dump is tampered!!")
    return generated_blockchain
```
Finally a function to make consensus. There should be only one explicit set of blocks in a chain at a given time.So in case of conflicts we choose longest chain.
```Python
@app.route('/nodes/resolve', methods=['GET'])
def consensus():
    """
    Our naive consnsus algorithm. If a longer valid chain is
    found, our chain is replaced with it.
    """
    global blockchain

    longest_chain = None
    current_len = len(blockchain.chain)

    for node in peers:
        response = requests.get('{}chain'.format(node))
        length = response.json()['length']
        chain = response.json()['chain']
        if length > current_len and blockchain.check_chain_validity(chain):
            current_len = length
            longest_chain = chain

    if longest_chain:
        blockchain = longest_chain
        return True

    return False
    ```

# Blockchain-starter
Starter app in Flask for basic blockchain functions

## Prerequisites
- Python 3.0+
- Flask and Requests 
```
# for install
pip install Flask requests
```
### Major steps
1. Implementing a basic proof of work
2. Making our blockchain an API endpoint
3. Creating a miner for your blockchain
4. Interacting with the blockchain


### Step 1 - Implementing a basic proof of work
Understanding Proof of Work

A Proof of Work algorithm (PoW) is how new Blocks are created or mined on the blockchain.

The goal of PoW is to discover a number which solves a problem. The number must be difficult to find but easy to verify for anyone on the network. The number is made of cryptographic signatures, meaning that if the wrong number is sent somewhere it will be denied access. Thus the PoW algorithm means you can now send money without having to trust anybody or any organisation, as the blockchain only cares about the cryptographic signatures.


### Step 2 - Making our blockchain an API endpoint
This will allow us to use postman/insomnia to send/receive api requests in our blockchain.

```
class Blockchain(object):

# Instantiate our Node
app = Flask(__name__)

# Generate a globally unique address for this node
node_identifier = str(uuid4()).replace('-', '')

# Instantiate the Blockchain
blockchain = Blockchain()


@app.route('/mine', methods=['GET'])
def mine():
    return "We'll mine a new Block"

@app.route('/transactions/new', methods=['POST'])
def new_transaction():
    return "We'll add a new transaction"

@app.route('/chain', methods=['GET'])
def full_chain():
    response = {
        'chain': blockchain.chain,
        'length': len(blockchain.chain),
    }
    return jsonify(response), 200

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)


@app.route('/transactions/new', methods=['POST'])
def new_transaction():
    values = request.get_json()

    # Check that the required fields are in the POST'ed data
    required = ['sender', 'recipient', 'amount']
    if not all(k in values for k in required):
        return 'Missing values', 400

    # Create a new Transaction
    index = blockchain.new_transaction(values['sender'], values['recipient'], values['amount'])

    response = {'message': f'Transaction will be added to Block {index}'}
    return jsonify(response), 201
```

### Step 3 - Creating a miner for your blockchain
This will mine and create a new transaction to be placed on a block in the blockchain.

```
@app.route('/mine', methods=['GET'])
def mine():
    # We run the proof of work algorithm to get the next proof...
    last_block = blockchain.last_block
    last_proof = last_block['proof']
    proof = blockchain.proof_of_work(last_proof)

    # We must receive a reward for finding the proof.
    # The sender is "0" to signify that this node has mined a new coin.
    blockchain.new_transaction(
        sender="0",
        recipient=node_identifier,
        amount=1,
    )

    # Forge the new Block by adding it to the chain
    block = blockchain.new_block(proof=proof, previous_hash=0)

    response = {
        'message': "New Block Created",
        'index': block['index'],
        'transactions': block['transactions'],
        'proof': block['proof'],
        'previous_hash': block['previous_hash'],
    }
    return jsonify(response), 200
```

### Step 4 - Interacting with the blockchain
Start up your server.

Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
Make a get request on postman/insomnia

Then get request on http://localhost:5000/mine

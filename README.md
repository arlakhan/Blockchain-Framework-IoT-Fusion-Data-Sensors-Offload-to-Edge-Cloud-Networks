The entitled Secure IoT Sensors Fusion Enabled Offloading in Edge Cloud Networks framework. 
This simulator generates synthetic IoT data at the local devices, and offload to edge nodes for blockchain operations. All the sythetic sensors data are executed on the cloud computing. 
The data security is very high and processed based on blockchain Technology. 

![image](https://github.com/user-attachments/assets/f152ac84-f4fc-44fd-816d-4f581b967b48)

![image](https://github.com/user-attachments/assets/d7ea6db2-a6a6-47dc-9a4f-cdaedb337fd3)




from flask import Flask, request, jsonify, render_template_string
from flask_ngrok import run_with_ngrok
import hashlib
import time
import random

app = Flask(__name__)
run_with_ngrok(app)

# Blockchain implementation
class Blockchain:
    def __init__(self):
        self.chain = []
        self.difficulty = 2
        self.create_genesis_block()

    def create_genesis_block(self):
        genesis_block = Block(0, "0", time.time(), "Genesis Block", "0")
        genesis_block.hash = genesis_block.calculate_hash()
        self.chain.append(genesis_block)

    def add_block(self, data):
        previous_block = self.chain[-1]
        new_block = Block(len(self.chain), previous_block.hash, time.time(), data, "")
        self.mine_block(new_block)
        self.chain.append(new_block)

    def mine_block(self, block):
        while block.hash[:self.difficulty] != "0" * self.difficulty:
            block.nonce += 1
            block.hash = block.calculate_hash()

    def get_chain(self):
        return [block.to_dict() for block in self.chain]


class Block:
    def __init__(self, index, previous_hash, timestamp, data, hash):
        self.index = index
        self.previous_hash = previous_hash
        self.timestamp = timestamp
        self.data = data
        self.hash = hash
        self.nonce = 0

    def calculate_hash(self):
        block_string = f"{self.index}{self.previous_hash}{self.timestamp}{self.data}{self.nonce}"
        return hashlib.sha256(block_string.encode("utf-8")).hexdigest()

    def to_dict(self):
        return {
            "index": self.index,
            "previous_hash": self.previous_hash,
            "timestamp": self.timestamp,
            "data": self.data,
            "hash": self.hash,
            "nonce": self.nonce,
        }


# IoT Sensors, Edge, and Cloud Nodes
iot_data = []  # Simulates incoming sensor data
edge_blockchain = Blockchain()  # Edge node blockchain
cloud_blockchain = Blockchain()  # Cloud node blockchain

# HTML Templates
HOME_TEMPLATE = """
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Abdullah Lakhan Research Laboratory:IoT-Edge-Cloud Blockchain</title>
</head>
<body>
    <h1>IoT-Edge-Cloud Blockchain System</h1>
    <h2>IoT Sensors</h2>
    <form action="/generate_100_sensors" method="post">
        <button type="submit">Generate 100 IoT Sensor Data</button>
    </form>
    <h2>Edge Node</h2>
    <form action="/process_edge" method="post">
        <button type="submit">Process IoT Data at Edge</button>
    </form>
    <h2>Cloud Node</h2>
    <form action="/offload_to_cloud" method="post">
        <button type="submit">Offload Data to Cloud</button>
    </form>
    <hr>
    <h2>View Blockchains</h2>
    <form action="/view_blockchain" method="get">
        <button name="node" value="edge">View Edge Blockchain</button>
        <button name="node" value="cloud">View Cloud Blockchain</button>
    </form>
</body>
</html>
"""

BLOCKCHAIN_TEMPLATE = """
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{{ node.capitalize() }} Blockchain</title>
</head>
<body>
    <h1>{{ node.capitalize() }} Blockchain Data</h1>
    <ul>
        {% for block in blockchain %}
        <li>
            <strong>Block #{{ block.index }}</strong><br>
            Previous Hash: {{ block.previous_hash }}<br>
            Timestamp: {{ block.timestamp }}<br>
            Data: {{ block.data }}<br>
            Hash: {{ block.hash }}<br>
            Nonce: {{ block.nonce }}
        </li>
        <hr>
        {% endfor %}
    </ul>
    <form action="/" method="get">
        <button type="submit">Back to Home</button>
    </form>
</body>
</html>
"""


@app.route("/")
def home():
    """Render the home page."""
    return render_template_string(HOME_TEMPLATE)


@app.route("/generate_100_sensors", methods=["POST"])
def generate_100_sensors():
    """Generate 100 IoT sensor data."""
    global iot_data
    iot_data = []  # Clear old data
    for i in range(100):
        sensor_reading = {
            "sensor_id": f"sensor_{i + 1}",
            "temperature": round(random.uniform(15.0, 35.0), 2),
            "humidity": round(random.uniform(40.0, 90.0), 2),
            "timestamp": time.time(),
        }
        iot_data.append(sensor_reading)
    return f"<h1>Generated 100 IoT Sensor Data</h1><form action='/'><button type='submit'>Back to Home</button></form>"


@app.route("/process_edge", methods=["POST"])
def process_edge():
    """Process IoT data at the edge node."""
    global iot_data
    if not iot_data:
        return "<h1>No IoT Data to Process!</h1><form action='/'><button type='submit'>Back to Home</button></form>"

    for data in iot_data:
        edge_blockchain.add_block(data)
    iot_data = []  # Clear IoT data after processing
    return "<h1>IoT Data Processed and Added to Edge Blockchain!</h1><form action='/'><button type='submit'>Back to Home</button></form>"


@app.route("/offload_to_cloud", methods=["POST"])
def offload_to_cloud():
    """Offload edge blockchain data to the cloud."""
    for block in edge_blockchain.chain:
        if block.index > 0:  # Skip genesis block
            cloud_blockchain.add_block(block.data)
    return "<h1>Edge Data Offloaded to Cloud Blockchain!</h1><form action='/'><button type='submit'>Back to Home</button></form>"


@app.route("/view_blockchain", methods=["GET"])
def view_blockchain():
    """View the blockchain for the specified node."""
    node = request.args.get("node")
    blockchain_data = (
        edge_blockchain.get_chain() if node == "edge" else cloud_blockchain.get_chain()
    )
    return render_template_string(
        BLOCKCHAIN_TEMPLATE, node=node, blockchain=blockchain_data
    )


if __name__ == "__main__":
    app.run()



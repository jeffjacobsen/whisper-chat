# whisper-chat

This demonstrates how to use Ethereum Whisper v6. This dApp is tested using geth v1.9.14 on Windows 10 and Ubuntu 18.04. 


If you don't know how to test and run this example, [this wiki article](https://github.com/CrabAss/dCollab/wiki/How-to-test-and-run-dCollab) will help you establish a private Ethereum network with two nodes whose Whisper feature is enabled.

## Running the example

This example assumes that there is a running Whisper v6 node exposing a WebSocket RPC interface at URL `ws://localhost:8546`. 

Clone this repository and download the dependencies:

    $ npm install

Finally, start the example with:

    $ npm run dev

The example should be started and the application will be available at `http://localhost:8080`.

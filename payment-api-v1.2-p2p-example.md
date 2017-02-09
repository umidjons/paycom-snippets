# Payment API v1.2 + P2P example

```sh
npm init --yes
npm i -S ws debug
```

Following script will transfer from card 5555-5555-5555-5555 5000 soms to card 6666-6666-6666-6666.

File `index.js`:
```js
'use strict';

const WebSocket = require('ws');
const crypto = require('crypto');
const debug = require('debug')('ws:app');
const debugWs = require('debug')('ws:ws');

debug.log = console.log.bind(console);
debugWs.log = console.log.bind(console);

const ID = '181ef5955c75f5025ad381fa';
const KEY = '98T5GZWZm?6nfIPodUZ4X8riJFBhyUgYy876%USh0@eSdoMEdXfZmJAQO9Y37bsWDdhmvKgL';
const URL = 'ws://gateway.test.paycom.uz/';

class SuperAPI {

    constructor() {
        this.ws = null;
        this.time = null;
        this.hash = null;

        this.generateNewHash();
        this.initSocket();
    }

    initSocket() {
        this.ws = new WebSocket(URL, {headers: {Service: this.getServiceHeader()}});

        this.ws.on('open', () => {
            debugWs('Connected');
            this.p2pInfo('5555555555555555');
        });

        this.ws.on('close', () => {
            debugWs('Closed');
        });

        this.ws.on('error', (err) => {
            debugWs('Error>', err);
        });

        this.ws.on('message', (message) => {
            debugWs('Received>', message);

            message = JSON.parse(message);

            if (message.result) { // handle success response

                if (message.result.limits) {
                    // received p2p info

                    // todo: check, does a receiver have enough limit to accept transfer?

                    // create cheque to transfer
                    this.receiptCreate('6666666666666666', 500000);
                }

                if (message.result.receipt && message.result.receipt.state === 0) {

                    // cheque created
                    debugWs('Cheque created> ID=', message.result.receipt._id);

                    let senderCard = {
                        number: '6666666666666666',
                        expire: '1704'
                    };

                    // pay just created cheque
                    this.receiptPay(message.result.receipt._id, senderCard);
                }

                if (message.result.receipt && message.result.receipt.state === 4) {

                    // cheque payed
                    debugWs('Cheque payed> ID=', message.result.receipt._id);

                    // todo: show notification, that the transfer success

                    this.ws.close();
                }
            } else {
                debugWs('Something Wrong>', message);
                this.ws.close();
            }
        });
    }

    generateNewHash() {
        this.time = Date.now();
        this.hash = crypto.createHash('sha1').update(this.time + KEY).digest('hex');
        debug('generateHash() hash=', this.hash);
        return this.hash;
    }

    getServiceHeader() {
        return `${ID}; ${this.hash}; ${this.time}`;
    }

    p2pInfo(receiverCardNumber) {
        let data = {
            method: 'cards.get_p2p_info',
            params: {number: receiverCardNumber}
        };
        this._send(data);
    }

    receiptCreate(senderCardNumber, amount, description, detail) {
        let data = {
            method: 'receipt.p2p',
            params: {
                card: senderCardNumber,
                amount: amount
            }
        };

        if (description)
            data.params.description = description;

        if (detail)
            data.params.detail = detail;

        this._send(data);
    }

    receiptPay(chequeId, senderCard, payer) {
        let data = {
            method: 'receipt.pay_by_card',
            params: {
                id: chequeId,
                card: senderCard
            }
        };

        if (payer)
            data.params.payer = payer;

        this._send(data);
    }

    _send(data) {
        let jsonStr = JSON.stringify(data);
        debugWs('Sending>', jsonStr)
        this.ws.send(jsonStr);
    }
}

// start transfer process
let api = new SuperAPI();
```

Execution:
```sh
set DEBUG=ws:*
set DEBUG_COLORS=0
node index.js
```

Sample output:
```
Thu, 09 Feb 2017 06:53:58 GMT ws:app generateHash() hash= 2fcee901784f64184f488c9da830c01adef9b4a1

Thu, 09 Feb 2017 06:53:58 GMT ws:ws Connected

Thu, 09 Feb 2017 06:53:58 GMT ws:ws Sending> {"method":"cards.get_p2p_info","params":{"number":"5555555555555555"}}
Thu, 09 Feb 2017 06:53:58 GMT ws:ws Received> {"jsonrpc":"2.0","result":{"limits":{"count":999999,"amount":100000000},"owner":"JOHN DOE","bank":{"name":"KBZ Bank","logo":"http://cdn.paycom.uz/merchants/kbz.png"},"interbank":true,"terms":{"ru":"https://cdn.payme.uz/terms/ru/universalbank-p2p.html","uz":"https://cdn.payme.uz/terms/uz/universalbank-p2p.html"},"commissions":[{"from":1,"to":500000,"type":"a","value":10000},{"from":500001,"to":100000000,"type":"p","value":2}]},"debug":{"memory":{"rss":"44.445","total":"29.671","used":"25.778"},"time":"3ms"}}

Thu, 09 Feb 2017 06:53:58 GMT ws:ws Sending> {"method":"receipt.p2p","params":{"card":"6666666666666666","amount":500000}}
Thu, 09 Feb 2017 06:53:58 GMT ws:ws Received> {"jsonrpc":"2.0","result":{"receipt":{"_id":"7e119b352014bcf9e75e8a7f","create_time":1486623212330,"pay_time":0,"cancel_time":0,"state":0,"type":5,"external":false,"operation":-1,"category":null,"error":null,"description":"","detail":null,"amount":510000,"commission":10000,"account":[{"name":"card","title":{"ru":"Номер карты","en":"Номер карты","uz":"Номер карты"},"value":"666666******6666","raw":"6666666666666666"},{"name":"owner","title":{"ru":"Владелец карты","en":"Владелец карты","uz":"Владелец карты"},"value":"GEVERGES OLEG"}],"card":null,"merchant":{"_id":"55e30134f35ba21750f8eb7d","name":"KBZ Bank","organization":"","address":"","epos":{"merchantId":"0","terminalId":"0"},"logo":"https://cdn.paycom.uz/merchants/kbz.png","type":{"ru":"Перевод средств","uz":"Перевод средств"},"terms":{"ru":"https://cdn.payme.uz/terms/ru/universalbank-p2p.html","uz":"https://cdn.payme.uz/terms/uz/universalbank-p2p.html"}}}},"debug":{"memory":{"rss":"44.445","total":"29.671","used":"25.842"},"time":"9ms"}}
Thu, 09 Feb 2017 06:53:58 GMT ws:ws Cheque created> ID= 7e119b352014bcf9e75e8a7e

Thu, 09 Feb 2017 06:53:58 GMT ws:ws Sending> {"method":"receipt.pay_by_card","params":{"id":"7e119b352014bcf9e75e8a7f","card":{"number":"6666666666666666","expire":"1704"}}}
Thu, 09 Feb 2017 06:53:58 GMT ws:ws Received> {"jsonrpc":"2.0","result":{"receipt":{"_id":"7e119b352014bcf9e75e8a7f","create_time":1486623212330,"pay_time":1486623212352,"cancel_time":0,"state":4,"type":5,"external":false,"operation":-1,"category":null,"error":null,"description":"","detail":null,"amount":510000,"commission":10000,"account":[{"name":"card","title":{"ru":"Номер карты","en":"Номер карты","uz":"Номер карты"},"value":"666666******6666","raw":"6666666666666666"},{"name":"owner","title":{"ru":"Владелец карты","en":"Владелец карты","uz":"Владелец карты"},"value":"GEVERGES OLEG"}],"card":{"number":"6666666666666666","expire":"1704"},"merchant":{"_id":"55e30134f35ba21750f8eb7d","name":"KBZ Bank","organization":"","address":"","epos":{"merchantId":"0","terminalId":"0"},"logo":"https://cdn.paycom.uz/merchants/kbz.png","type":{"ru":"Перевод средств","uz":"Перевод средств"},"terms":{"ru":"https://cdn.payme.uz/terms/ru/universalbank-p2p.html","uz":"https://cdn.payme.uz/terms/uz/universalbank-p2p.html"}}}},"debug":{"memory":{"rss":"44.602","total":"29.671","used":"25.947"},"time":"15ms"}}
Thu, 09 Feb 2017 06:53:58 GMT ws:ws Cheque payed> ID= 7e119b352014bcf9e75e8a7f

Thu, 09 Feb 2017 06:53:58 GMT ws:ws Closed
```

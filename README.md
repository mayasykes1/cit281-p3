# CIT281 Project 3

## Learning Objectives:
<li>Gain experience interpreting functional descriptions and specifications to complete an assignment</li>
<li>Gain experience breaking a project into manageable components</li>
<li>Gain experience writing and executing non-web server Node.js JavaScript code using VSCode</li>
<li>Practice creating and using code modules</li>
<li>Practice refactoring using modern JavaScript syntax</li>
<li>Gain experience writing and executing web server Node.js JavaScript code using VSCode</li>
<li>Gain experience using Fastify with the GET verb, routes, and query parameters</li>
<li>Gain experience loading a file and using as a web page</li>

##Code:

#### p3-module.js

##### 1. validDenomination(coin)
<code>function validDenomination(coin) {
    let validCoin = [1, 5, 10, 25, 50, 100];
    let indexOf = validCoin.indexOf(coin);
    if(indexOf === -1) {
        return false;
    } 
    else {
        return true;
    }
}</code>

##### 2. valueFromCoinObject(obj)
<code>function valueFromCoinObject(obj) {
    const { denom = 0, count = 0 } = obj;
    if(validDenomination(denom)) {
        return denom * count;
    }
    else{
        return 0;
    }
} </code>

##### 3. valueFromArray(arr)
<code>let valueFromArray = (arr) => {
    return arr.reduce((total, coin) => {
        return total + valueFromCoinObject(coin);
    },0);
}</code>

##### 4. coinCount(...coinage)
<code> function coinCount(...coinage) {
    return valueFromArray(coinage);
}
module.exports = {coinCount};</code>

##### Tests: 
1. console.log("{}", coinCount({denom: 5, count: 3})); - 15
<br>
2. console.log("{}s", coinCount({denom: 5, count: 3},{denom: 10, count: 2}));
- 35
<br>
3. const coins = [{denom: 25, count: 2},{denom: 1, count: 7}];
- Undefined
<br>
4. console.log("...[{}]", coinCount(...coins));
- 57
<br>
5. console.log("[{}]", coinCount(coins));  // Extra credit
- 0
<br>

#### p3-server.js
<code>
const fs = require("fs");
const fastify = require("fastify")();
const { coinCount } = require("./p3-module.js");

fastify.get("/", (request, reply) => {
  fs.readFile(`${__dirname}/index.html`, (err, data) => {
    if (err) {
      reply
        .code(500)
        .header("Content-Type", "text/html; charset=utf-8")
        .send("<h1>Server error.</h1>");
    } else {
      reply
        .code(200)
        .header("Content-Type", "text/html; charset=utf-8")
        .send(data);
    }
  });
});
fastify.get("/coin", (request, reply) => {
  const { denom = 0, count = 0 } = request.query;
  const coinValue = coinCount({
    denom: parseInt(denom),
    count: parseInt(count),
  });
  reply
    .code(200)
    .header("Content-Type", "text/html; charset=utf-8")
    .send(
      `<h2>Value of ${count} of ${denom} is ${coinValue}</h2><br /><a href="/">Home</a>`
    );
});
fastify.get("/coins", (request, reply) => {
  let coinValue = 0;
  const coins = [
    { denom: 25, count: 2 },
    { denom: 1, count: 7 },
  ];
const { option } = request.query;
  switch (option) {
    case "1":
      coinValue = coinCount({ denom: 5, count: 3 }, { denom: 10, count: 2 });
      break;
    case "2":
      coinValue = coinCount(...coins);
      break;
    case "3":
      coinValue = coinCount(coins);
      break;
  }
  reply
    .code(200)
    .header("Content-Type", "text/html; charset=utf-8")
    .send(
      `<h2>Option ${option} value is ${coinValue}</h2><br /><a href="/">Home</a>`
    );
});
// Start server and listen to requests using Fastify
const listenIP = "localhost";
const listenPort = 8080;
fastify.listen(listenPort, listenIP, (err, address) => {
  if (err) {
    console.log(err);
    process.exit(1);
  }
  console.log(`Server listening on ${address}`);
});
</code>

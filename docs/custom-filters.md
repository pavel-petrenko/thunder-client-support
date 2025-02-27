# Customer Filters
- [Custom Filters](#custom-filters)
  - [Pre Request Filter](#pre-req-filter)
  - [Post Request Filter](#post-req-filter)
  - [Import Node Module](#import-node-module)
  - [Import JS Files](#import-js-files)
  - [Execute CLI Command](#cli-command)
  - [Execute Requests](#execute-requests)
  - [Retry Request](#retry-request)
  - [Assertions using Scripting](#assertions)
  - [Delay Function](#delay)
  - [tc object types](#tc-object)
- [Feedback](#feedback)

## Custom Filters

### Global Variables
- The global variables available are `tc`, `expect`, `assert`, `btoa`, and `atob`.
- `tc` is the main [object](https://github.com/rangav/thunder-client-support/blob/master/docs/tc-types.d.ts) for all API access.
- [Chai](https://www.chaijs.com/api/bdd/) library `expect` and `assert` for assertions.
- `btoa` and `atob` to base64 encoding and decoding.

### Built in Modules
- `ajv`, `ajv-formats`, `assert`, `atob`, `axios`, `btoa`, `buffer`, `chai`, `crypto-js`, `fast-xml-parser`, `fs`, `http`, `https`, `papaparse`, `stream`, `tough-cookie`, `url`
- To use built-in modules -  `var axios = require("axios");`

### How to Create Customer Filters

**Step 1:** Create Javascript file with custom filters
- The sample custom filters javascript file [custom-filters.js](https://github.com/rangav/thunder-client-support/blob/master/docs/custom-filters.js)

<img width="701" alt="custom-filter" src="https://user-images.githubusercontent.com/8637550/202422492-30ad7123-0964-40db-ba1c-31bbe67d57f4.png">

**Step 2:** Attach Custom filters JS files to `Collection Settings`
<img width="900" alt="col-sets" src="https://user-images.githubusercontent.com/8637550/202426659-972f5307-0b51-4100-b6bf-0f3fb2f33140.png">

**Step 3:** Use Custom filters in Request
<img width="900" alt="custom-filter-using" src="https://user-images.githubusercontent.com/8637550/202422840-76998a57-0cbf-46ef-9309-52965c72959c.png">

------
<a name="pre-req-filter"></a>

### Pre Request Filter in PreRun Tab
- Run Custom Filter directly in `Pre-Run` tab as Pre Request Script, useful to set Env Variables
<img width="705" alt="Pre Req Filter" src="https://github.com/rangav/thunder-client-support/assets/8637550/07420d12-c9de-4efc-af3f-f85a47412cf2">

- This Custom Filter will not have any `arguments` and return `no value`
<img width="543" alt="Pre Filter Function" src="https://user-images.githubusercontent.com/8637550/205506079-395de0b6-593e-4fa9-b38a-31a1d6a1545a.png">

------

<a name="post-req-filter"></a>

### Post Request Filter in Tests Tab
- Run Custom Filter directly in `Tests` tab as Post Request Script
- Useful to do clean-up tasks after request or set environment variables from the response for advanced use cases

<img width="705" alt="Post Req Script" src="https://github.com/rangav/thunder-client-support/assets/8637550/8b0769df-c796-4219-b4c9-a5c4fe36d9b0">

------

<a name="import-node-module"></a>

### Import Node Module (Beta)
- Now you can import any `node module` in [Custom Filters](https://github.com/rangav/thunder-client-support/blob/master/docs/custom-filters.js)
- This feature is still in `Beta`, not all modules are working
- e.g:   `var moment = await tc.loadModule("moment");`
<img width="762" alt="Screenshot 2022-12-04 at 17 33 03" src="https://user-images.githubusercontent.com/8637550/205506205-f01dd73d-d8b1-41eb-9f65-a0c88170d00f.png">

#### Use Node Modules to generate fake data
- You can use node libraries like [faker-js](https://www.npmjs.com/package/@faker-js/faker), or lightweight libraries [chance](https://www.npmjs.com/package/chance), [falso](https://www.npmjs.com/package/@ngneat/falso) to generate random data
- Create a Custom Filter and use it in `Pre Run` tab -> `Pre Request Script` to generate fake data.
- Example custom filter script
```js
async function fakeDataFilter() {
    // example code to load faker-js module
    var { faker } = await tc.loadModule("@faker-js/faker");
    tc.setVar("firstName", faker.person.firstName());

    // example code to load chance module
    var Chance = await tc.loadModule("chance");
    var chance = new Chance();
    tc.setVar("firstName", chance.name());

    // example code to load falso module
    var falso = await tc.loadModule("@ngneat/falso");
    var user = falso.randUser();
    tc.setVar("firstName", user.firstName);
}

module.exports = [fakeDataFilter];
```

-----
<a name="import-js-files"></a>

### Import Functions from js files
- You can import functions from other js files. Useful for code re-use and check-in to your git repo.
- The path should be relative to workspace if you use Git-Sync, otherwise use full-path. 
- You can right click on the file and choose `Copy Path` or `Copy Relative Path` accordingly 

```js
// test-import.js file
function helloWorld(){
   return "hello world";
}

module.exports = {
  helloWorld
}
```
- From filters import js file using `require`
```js

var {helloWorld} = require("thunder-tests/test-import.js");
var result = helloWorld();
console.log(result);
```

- You can also save the path in the `Environment variable`, so you don't have to change the path in every request if you move the file to different folder.
```js
var scriptPath = tc.getVar("scriptPath");
var {helloWorld} = require(scriptPath);
var result = helloWorld();
console.log(result);
```

-----
<a name="cli-command"></a>

### Execute CLI Command
- To execute a CLI command and get request use exec helper function
- `var result = await tc.exec("command");`

------

<a name="execute-requests"></a>

### Send Requests from Script
- Execute existing requests using the API - `await tc.runRequest("reqId");`
- Useful for request chaining programmatically from the Pre or Post request script.
```js
async function testReq(){
   var result = await tc.runRequest("reqId");
   console.log(result);
}

module.exports = [testReq]
``` 
- Execute custom requests using the [Axios library](https://axios-http.com/docs/api_intro)
- Useful for executing custom requests programmatically from the Pre or Post request script.
```js
const axios = require('axios');

async function testReq(){
   const response = await axios.get("https://www.thunderclient.com/welcome");
   console.log(response);
}

module.exports = [testReq]
```

### Retry Request
- Use the `await tc.retryRequest();` for retrying same request
- Use Sample code in `Post Request Script` and modify it as per your requirements

```js
let incrementCount = parseInt(tc.getVar('incrementCount') || "0");
let code = tc.response.status;

if(incrementCount <= 3 && code !== 200)
{
    incrementCount = incrementCount + 1
    tc.setVar('incrementCount', incrementCount)
    console.log("retrying request", incrementCount);

    await tc.delay(incrementCount * 1000); // exponential delay of 1 secs
    await tc.retryRequest();
}
else
{
   tc.setVar('incrementCount', 0);
   console.log("reset incrementCount = 0");
}
```

------

<a name="assertions"></a>
## Assertions using Scripting

#### Assertions without any library

```js

function testFilter() {
    let success = tc.response.status == 200;
    let json = tc.response.json;
    let containsThunder = json.message?.includes("thunder");

    tc.test("Response code is 200", success);
    tc.test("Response contains thunder word", containsThunder);

    // Assertions using function syntax
    tc.test("verifying multiple tests", function () {
            let success = tc.response.status == 200;
            let time = tc.response.time < 1000;
            return success && time;
    });
}

module.exports = [testFilter]
```
#### Assertions using Chai library
- expect and assert functions are available as global variables
```js

function testChaiFilter() {  
    tc.test("Response code expect to be 200", function () {
        expect(tc.response.status).to.equal(200);
    })

    tc.test("Response code is 200", function () {
        assert.equal(tc.response.status, 200)
    })
}

module.exports = [testChaiFilter]
```
------

<a name="delay"></a>

### Delay Helper Function
- To delay the script execution use the API - `await tc.delay(1000);`
```js
async function testReq(){
   // delay for 5 seconds
   var result = await tc.delay(5000);
   console.log("delayed for 5 seconds");
}

module.exports = [testReq]
```
------
<a name="tc-object"></a>

### tc object types
- Refer to `tc` object types file  [tc-types.d.ts](https://github.com/rangav/thunder-client-support/blob/master/docs/tc-types.d.ts) for the methods and properties available.
- Copy the types file to you project and refer at the top for VS Code autocomplete suggestions
```
/// copy tc-types.d.ts file for vscode autocompletion on tc object
/// <reference path="./tc-types.d.ts" />
```

<a name="feedback"></a>
### Feedback
- Let us know if you have any suggestions for improvement and also need any additional built-in filters


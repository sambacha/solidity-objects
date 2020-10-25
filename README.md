# Solidity Objects

Dump all the values from a deployed contract

## Quick Usage

```JavaScript
const Mapper = require('solidity-objects');

// automatically loads truffle.js and contracts from build/contracts
const mapper = new Mapper();

(async () => {
  const values = await mapper.map('MetaCoin', '0x254dffcd3277C0b1660F6d42EFbB754edaBAbC2B');
  console.log(values);
})();
```

## Examples

### Customize Output for Types

```JavaScript
const mapper = new Mapper({
  types: {
    // convert ALL int types to string
    int: function (val) {
      return Web3.utils.BN(val).toString();
    },
    // convert all uint256 to string; default is BN
    uint256: function(val) {
      return Web3.utils.BN(val).toString();
    }
  },
});
const values = await mapper.map('MetaCoin', '0x254dffcd3277C0b1660F6d42EFbB754edaBAbC2B');
```

### Provide Custom Output Mapping

See [Object Mapper](https://github.com/wankdanker/node-object-mapper#readme) for full details on output mapping.

```JavaScript

// all calls to .map will apply these transformations
const mapper = new Mapper({
  mapping: {
    // example: 'getState' is a method that returns an integer
    // which represents possible values in a struct
    //
    // This mapping will output two keys:
    //  'state' - the integer itself from getState
    //  'stateName' - the string representing the struct
    state: [
      {
        key: 'stateName',
        transform: function (val) {
          const index = Number(val.toString());
          const states = ['Funding', 'Active', 'Matured'];
          return states[index];
        },
      },
      'state',
    ],
  },
});


```

### Change Output Mapping Each Time

```JavaScript

// this custom mapping only applies to this call of .map
const values = await mapper.map('Token', '0x254dffcd3277C0b1660F6d42EFbB754edaBAbC2B', {
  // map the property 'name' to the output key 'tokenName'
  name: 'tokenName',
  // use a custom transformation
  status: {
    key: 'status',
    transform: function(val) {
      const statuses = {
        0: 'New',
        1: 'Active',
        2: 'Closed'
      };
      const index = val.toString();
      return statuses[index];
    }
  },
  // send a single value to multiple output values
  symbol: ['tokenSymbol', {
    key: 'symbol',
    transform: function(val) {
      return `XYZ:${val}`;
    }
  }],
});
```

## Methods

### map

- _(async)_ **contractName** - Name of the contract corresponding to one of the ABI files
- _(async)_ **at** - Address of deployed contract
- _(async)_ **mapping** - Hash of src:dest object mapping rules with optional transformations, applied only to this method invocation. See [Mapping](#Mapping). Can also be provided to constructor to apply to all calls.

## Options

See also: [lib/options.schema.json](lib/options.schema.json)

### networkName

**Type: _string_**

Network name corresponding to truffle config network name. Corresponds to truffle 'network' parameter

```JavaScript
const mapper = new Mapper({
  networkName: 'mainnet'
});
```

### workingDirectory

**Type: _string_**

Base directory to search for truffle config and contracts. Creates a truffle instance based on the contents of this directory using Truffle's auto-detection.

```JavaScript
const mapper = new Mapper({
  workingDirectory: path.join(__dirname, '/../my-truffle-project')
});
```

### types

**Type: _object_**

Hash of data type and how retrieved values of this type should be converted. Uses ABI definition to convert all properties of the specified type.

```JavaScript
const mapper = new Mapper({
  types: {
    // all 'int' types will be converted to string
    int: function (val) {
      return Web3.utils.BN(val).toString();
    }
  }
});
```

### mapping

**Type: _string|object|array_**

Hash of src:dest mapping; see the [Node Object Mapper Docs](https://github.com/wankdanker/node-object-mapper) for more information on object mapping. The src property will be mapped to one ore more dest properties.

#### string

Provide a simple string dest key:

```JavaScript
const mapper = new Mapper({
  mapping: {
    // value from contract property 'bytes32Example'
    // will be mapped to 'newPropertyName'
    'bytes32Example': 'newPropertyName'
  }
});
```

#### object

Provide an object with dest key and optional transformations:

```JavaScript
const mapper = new Mapper({
  mapping: {
    // value from contract property 'bytes32Example'
    // will be mapped to 'newPropertyName'
    'bytes32Example': {
      key: 'newPropertyName'
    },
    'intExample': {
      key: 'intExample',
      transform: function (val) {
        return Web3.utils.BN(val).toString();
      }
    }
  }
});
```

#### array

Use a list of destination mappings, which can be a mix of strings and/or objects:

```JavaScript
const mapper = new Mapper({
  mapping: {
    // value from contract property 'bytes32Example'
    // will be mapped to 'newPropertyName'
    'bytes32Example': [{
      key: 'newDestPropertyName'
    }, {
      key: 'otherDestPropertyName',
      transform: (val) => `${val}_modified_string`
    }]
  }
});
```

### provider

A pre-configured Web3 provider.

```JavaScript
const TruffleProvider = require('@truffle/provider');
const TruffleConfig = require('@truffle/config');
const mapper = new Mapper({
  provider: TruffleProvider.create(TruffleConfig.detect({
    network: 'development',
    workingDirectory: __dirname,
  })),
});
```

### contracts

**Type: _string|array|object_**

Glob path, array of paths, or pre-loaded ABI objects for the contracts to use.

#### string

Single glob path to the ABI contract files. A glob pattern to where contract ABI files are stored. Defaults to: **[workingDirectory]/build/contracts/\*.json**

```JavaScript
const mapper = new Mapper({
  contracts: path.join(__dirname, '/../my-truffle-project/build/**/*.json'),
});
```

#### array

List of glob path to the ABI contract files

```JavaScript
const mapper = new Mapper({
  contracts: [
    path.join(__dirname, '/../my-truffle-project/build/**/*.json'),
    path.join(__dirname, '/../other-project/build/**/*.json'),
  ],
});
```

#### object

A hash of contract name and the pre-loaded ABI. Use this if you don't need/want automatic contract detection and loading.

```JavaScript
const MyContract = require('./build/contracts/MetaCoin');
const mapper = new Mapper({
  contracts: { MyContract }
});
```

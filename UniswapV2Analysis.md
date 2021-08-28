# Uniswap V2 Protocol Analysis

## UniswapV2Factory.sol

``` solidity
pragma solidity =0.5.16;
import './interfaces/IUniswapV2Factory.sol';
import './UniswapV2Pair.sol';  //for calling UniswapV2Pair functions plus get creationCode
contract UniswapV2Factory is IUniswapV2Factory { //very simple inheritance
```

![](https://github.com/masaa-masaa/Uniswap-V2-Protocol-Analysis/blob/main/Factory%20Contract.png)

### The createPair Function 

```solidity
function createPair(address tokenA, address tokenB) external returns (address pair) {
		//checks
        require(tokenA != tokenB, 'UniswapV2: IDENTICAL_ADDRESSES');
        (address token0, address token1) = tokenA < tokenB ? (tokenA, tokenB) : (tokenB, tokenA);
        require(token0 != address(0), 'UniswapV2: ZERO_ADDRESS');
        require(getPair[token0][token1] == address(0), 'UniswapV2: PAIR_EXISTS'); 
        // get creationCode for the UniswapV2Pair contract 
        bytes memory bytecode = type(UniswapV2Pair).creationCode;
        bytes32 salt = keccak256(abi.encodePacked(token0, token1));
        //use assembly to create the new pair. create2 returns the new pair contract address
        assembly {
            pair := create2(0, add(bytecode, 32), mload(bytecode), salt)
        }
        //initialize new pair
        IUniswapV2Pair(pair).initialize(token0, token1);
        getPair[token0][token1] = pair;
        getPair[token1][token0] = pair; // populate mapping in the reverse direction
        //add to the exixting lists of pairs
        allPairs.push(pair);
        emit PairCreated(token0, token1, pair, allPairs.length);
    }
```

* The function has no access restrictions
* Anyone can create a pair
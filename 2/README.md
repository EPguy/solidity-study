## Solidity 0.8.0 미만 버전에서의 Overflow와 underflow
uint8, uint32, uint256 등 이런 타입들은 최대 저장될 수 있는 비트의 수가 존재합니다.
```
uint8 balance = 255;
```
만약 위 balance 변수에 1을 더하게 된다면 어떻게 될까요?\
balance를 2진수로 표현하면 11111111 입니다.\
여기서 1을 더하게 되면 비트의 수가 초과 되어 balance의 2진수 값은 00000000, 즉 0이 됩니다.\
이것은 마치 오래된 차의 주행계가 최대 측정할 수 있는 주행거리를 넘어가게 되면 다시 0이 되는것과 비슷합니다.\
우리는 이 현상을 overflow라고 부릅니다.
```
uint8 balance = 0;
```
그러면 반대로 위 balance 변수에 1을 뺀다면 어떤 값이 나올까요?\
uint는 unsigned int의 약자 즉 부호가 없기 때문에 양수만 표현할 수 있습니다.\
결국 0에서 1을 빼면 표현이 불가능하기 때문에 binary는 11111111, 10진수로 표현하면 255가 됩니다.\
우리는 이 현상을 underflow라고 부릅니다.

### 1. 해커들은 이걸 어떻게 악용할까?

아래의 예시 토큰 컨트랙트가 존재합니다.

```
// Token.sol
pragma solidity ^0.6.0;

contract Token {

    mapping(address => uint) balances;
    uint public totalSupply;

    constructor(uint _initialSupply) public {
        balances[msg.sender] = totalSupply = _initialSupply;
    }

    function transfer(address _to, uint _value) public returns (bool) {
        require(balances[msg.sender] - _value >= 0);
        balances[msg.sender] -= _value;
        balances[_to] += _value;
        return true;
    }

    function balanceOf(address _owner) public view returns (uint balance) {
        return balances[_owner];
    }
}

```
위 토큰 컨트랙트는 underflow 에 굉장히 치명적입니다.\
만약 여러분이 20개의 토큰을 들고있을 때 친구한테 21개의 토큰을 transfer 한다면 어떤일이 일어날까요?\
20 - 21은 -1이 아니라 underflow로 인해 (2 ^ 256) - 1이 됩니다.\
즉 갖고있는 토큰보다 더 많이 보내는 데에도 불구하고\
require 문이 통과되며 여러분은 (2^256) - 1 개의 토큰을 갖게됩니다.\
또한 여러분의 친구도 21개의 토큰을 갖게되죠.

### 2. 그러면 어떤 방법으로 overflow, underflow를 방어 할 수 있을까?
가장 쉬운 방법은 0.8.0 이상의 solidity 버전으로 컨트랙트를 작성하는 것입니다.\
0.8.0 이상의 버전에서는 컴파일러가 자동으로 overflows와 underflows를 체크하기 때문입니다.


overflow와 undeflow는 굉장히 치명적인 버그를 초래할 수 있기 때문에 0.8.0 이상의 버전으로 컨트랙트를 작성하는 것을 권장드립니다.

참고로 이 글은 다음 글을 참고하여 작성 하였습니다. 읽어 주셔서 감사합니다.\
[참고자료](https://hackernoon.com/hack-solidity-integer-overflow-and-underflow)

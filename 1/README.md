컨트랙트가 이더리움을 받기 위해서는 fallback 혹은 receive 함수가 있어야 합니다. 만약에 존재하지 않는다면 EVM 에서 에러를 던져 해당 트랜젝션을 revert 시킬 것 입니다. 하지만 그럼에도 불구하고 강제로 이더를 넣는 2가지 방법이 존재합니다. 이 2가지 방법에 대해서 소개해 드릴려고 합니다.



### 1. selfdestruct 메소드 사용

selfdestruct 블록체인 내에 존재하는 해당 컨트랙트의 데이터들을 지워주는 메소드입니다.
이 때 컨트랙트에 이더리움이 있는경우 해당 컨트랙트가 지워지면 이더가 영구적으로 손실이 될 수 있습니다.
그런 실수가 벌어지지 않도록 이 메소드는 주소를 파라미터를 받고있으며
그 주소로 컨트랙트가 소멸하기 전 현재 컨트랙트가 갖고있는 이더를 반환합니다.

이 로직을 이용하여 fallback, receive가 없는 컨트랙트에 이더를 보낼 수 있습니다.
아래는 예제 코드입니다.


```
// ForcingSendEther.sol
contract ForcingSendEther {

    fallback() external payable {

    }

    function attack(address _problemAddress) public payable{
        address payable addr = payable(_problemAddress);
        selfdestruct(addr);
    }
}

```

```
// ForcingSendEther.ts
import { ethers } from 'hardhat';

async function main() {
  const [addr1] = await ethers.getSigners();
  const ForcingSendEther = await ethers.getContractFactory("ForcingSendEther");
  const forcingSendEther = await ForcingSendEther.deploy();
  await forcingSendEther.deployed();

  await addr1.sendTransaction({
    to: forcingSendEther.address,
    value: ethers.utils.parseEther('0.001')
  });

  // 강제로 이더리움 보낼 컨트랙트 주소 
  const attackAddress = "0x1b0F3e38e8A943c076b19FCfDc32e981d78D0599"; 
  const result = await forcingSendEther.attack(attackAddress);

  console.log(result);
}

main().catch(err => {
  console.log(err);
  process.exitCode = 1;
});
```

위 코드는 간단한 예제 코드입니다.
먼저 컨트랙트를 하나 생성하고 그 컨트랙트에 0.001 이더를 보낸후
selfdestruct 메소드를 통해 자신은 소멸하면서 파라미터로 받은 주소로 이더리움을 반환합니다.
그렇게 되면 fallback, receive 함수가 없는 컨트랙트임에도 불구하고 0.001 이더를 갖고있게 됩니다.


### 2. 이더를 먼저 보낸 후 컨트랙트 배포하기

컨트랙트의 주소는 랜덤이 아닌 만들어지는 규칙이 존재합니다.
> 작성자(sender)의 주소와 작성자가 보낸 트랜잭션 수(nonce)로 결정적으로 계산된다. Sender와 nonce는 RLP로 인코딩된 다음 Kecak-256으로 해시된다.
Sender의 주소와 그들의 txnonce의 RLP 인코딩의 keccak256 해시의 가장 오른쪽 160비트이다.


아래 코드는 위 규칙을 Python으로 작성한 코드입니다.

```
def mk_contract_address(sender, nonce):
    return sha3(rlp.encode([normalize_address(sender), nonce]))[12:]
```

이렇게 컨트랙트가 배포되는 주소를 알 수 있기 때문에
컨트랙트를 배포하기 전 이더를 먼저 보낸 다음에 컨트랙트를 배포한다면 fallback, receive 함수가 없더라도 이더리움을 갖고 있을 수 있습니다.

참고로 이 글은 다음 글을 참고하여 작성 하였습니다. 읽어 주셔서 감사합니다.
[참고자료](https://medium.com/@alexsherbuck/two-ways-to-force-ether-into-a-contract-1543c1311c56)

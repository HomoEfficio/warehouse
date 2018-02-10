https://bitcoin.org/en/developer-guide 와 https://bitcoin.org/en/developer-reference 를 토대로 공부한 내용 정리

## 트랜잭션

- 비트코인은 

  >누군가의 지갑 주소에 담겨있는 것이 아니라,
  
  >누군가의 지갑 주소로 잠겨있다.

- 비트코인 트랜잭션(거래) 또는 비트코인 지불은 송금자가 송금자의 서명으로 코인의 잠금을 해제하고, 수금자의 정보로 코인을 잠그는 것이다.

  - 수금자의 정보로 코인을 잠그므로 나중에 수금자만이 그 코인을 잠금 해제하고 다른 수금자의 정보로 잠그는 지불을 할 수 있다.


- 트랜잭션은 `version`, 한 개 이상의 `input`, 한 개 이상의 `output`, `locktime`으로 구성된다.

  ![](https://bitcoin.org/img/dev/en-tx-overview.svg)


### UTXO(Unspent Transaction Output)

- 직역하면 소비되지 않은 거래 출력인데, 그보다는 `지불 가능한 코인`이라고 이해하는 것이 더 직관적이다.

- 지불 가능한 모든 코인은 지갑 주소로 인덱스되어 UTXO Set 안에 저장된다.

- UTXO 관련 정보는 https://statoshi.info/dashboard/db/unspent-transaction-output-set 에서 확인 가능하다.

  - 2018년 2월 10일 현재, UTXO Set에는 약 6천만개의 UTXO 즉 지불 가능한 코인이 있고, 3.2G의 공간을 차지한다.

    ![Imgur](https://i.imgur.com/EPmdA8N.png)

- 지갑은 UTXO set에서 자신의 지갑 주소로 찾을 수 있는 코인의 액수를 합해서 해당 지갑 주소에서 지불 할 수 있는 잔액을 계산한다.

### version

- 트랜잭션은 먼저 유효성이 검증되어야 다른 노드에 전파될 수 있다. 유효성 검증에 사용되는 검증 기준의 버전 번호가 `version`이다.

### input

- 송금자만이 잠금 해제할 수 있는 코인(UTXO)을 송금자가 잠금 해제함을 나타내는 정보 단위

- 한 트랜잭션 안에 여러 개의 `input`이 존재할 수 있다.

- 직전 거래의 `txid`와 `index`, 그리고 `sequence number`와 `sigScript`로 구성된다.

### output

- 송금자가 수금자로부터 제공 받은 해시값으로 코인을 잠가서 나중에 수금자만이 그 코인을 사용할 수 있도록 코인의 소유주를 수금자로 바꾸는 정보 단위

- 한 트랜잭션 안에 여러 개의 `output`이 존재할 수 있다.

- 트랜잭션 내의 `index`, `amount`, `pubkeyScript`로 구성된다.

### locktime

- 한 트랜잭션이 블록체인에 추가될 수 있는 가장 빠른 시간. 다시 말해 트랜잭션은 `locktime` 이후에 블록체인에 추가된다.

### 정리

- 트랜잭션은 거래이며 결국 지불이다.
- 내 서명으로 잠금을 해제하고, 받을 사람이 제공한 해시 정보로 다시 잠그는 것이 지불이다.
- 내가 다른 빋을 사람에게 비트코인을 보낸 다는 것은 
- 내게 비트코인을 보낸 사람이 내가 제공했던 해시 정보로 잠가둔 지불 가능한 코인을
- 내 서명으로 잠금 해제한 후
- 받을 사람이 내게 제공한 해시 정보로 잠그는 것이다.



### 주소

- `secp256k1` 타원 곡선상에서 Elliptic Curve Digital Signature Algorithm에 의해 개인키/공개키 쌍 생성
- 개인키는 256비트 랜덤 데이터
- 공개키는 개인키에서 결정론적으로 생성되므로, 특정 개인키에서 생성한 첫번쨰, 두번째, 세번쨰.. 공개키는 언제나 같은 값으로 재생성된다.
- 주소(pubkeh hash)는 주소 버전 번호, 공개키의 해시값, 체크섬을 base58로 인코딩한 값으로 QR코드로 인코딩 될 수 있다.

### 앨리스가 밥의 주소로 송금하는 과정

- 앨리스 지갑이 앨리스의 주소로 잠겨 있는 `output A`를 풀 수 있는 앨리스의 개인키가 포함된 `scriptSig`를 포함하는 `input A`를 생성
- 밥의 주소로 잠그는 `scriptPubKey`를 포함하는 `output B`를 생성
- `intput A`와 `output B`가 담긴 `tx T`을 네트워크에 전파하고 블록에 추가
- 블록의 작업 증명이 완료되면
- 풀 노드에 있는 UTXO set에서는 `output A`가 제거되고 `output B`가 새로 추가
- 밥의 주소를 알고 있는 밥의 지갑은 `output B`의 `amount`를 밥이 사용할 수 있는 금액으로 인식

### pubkeyScript

- 송금자가 수금자의 pubkey hash를 이용해서 만드는 스크립트로 `output`에 담기며 이 `output`이 나중에 지불될 때 실행되는 스크립트 ???
- 지불될 때 개인키를 입력받고 개인키에서 생성된 full pub key로 해시계산을 해서 pubkey hash 값이 나오는지 확인

### sigScript

- 송금자가 자신의 private key를 이용해서 `output`을 지불하기 위한 서명 과정에서 실행되는 스크립트로 `signature`와 `pubKey`를 `output`에 담겨있던 `pubkeyScript`에 제공하고 `pubkeyScript`를 실행한다.

### P2PKH

- Pay to Public Key Hash
- 수금자의 단순 Public Key Hash에 송금

### P2SH

- Pay to Script Hash
- MultiSig 등 수금자가 실행할 스크립트의 Hash에 송금

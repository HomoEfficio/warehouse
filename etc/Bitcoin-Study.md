## 트랜잭션

![](https://bitcoin.org/img/dev/en-tx-overview.svg)

- 트랜잭션은 `version`, 한 개 이상의 `input`, 한 개 이상의 `output`, `locktime`으로 구성된다.
- 비트코인은 주소에 담겨있지 않다.
- 송금 과정을 마친 모든 코인은 UTXO set 안에 저장된다.
- UTXO는 특정 주소에 의해 잠겨 있어서 이를 기반으로 주소별로 사용할 수 있는 금액이 계산된다.
- `version`으로 트랜잭션의 유효성 검증 규칙을 구분한다.

### output

- 코인이 소비될 수 있는 상태임을 나타내는 정보 단위
- 한 트랜잭션 안에 여러 개의 `output`이 존재할 수 있다.
- `트랜잭션 내의 index`, `amount`, `pubkeyScript`로 구성된다.
- `output`과 코인은 1:1 이다. ???

### input

- 코인이 소비됨을 나타내는 정보 단위
- 코인이 소비되기 직전에 속해있던 트랜잭션의 `txid`, `index`(`vout`(output vector를 의미)로 불리기도 함), `sequence number`, `sigScript`로 구성된다.

### 주소

- `secp256k1` 타원 곡선상에서 Elliptic Curve Digital Signature Algorithm에 의해 개인키/공개키 쌍 생성
- 개인키는 256비트 랜덤 데이터
- 공개키는 개인키에서 결정론적으로 생성되므로, 특정 개인키에서 생성한 첫번쨰, 두번째, 세번쨰.. 공개키는 언제나 같은 값으로 재생성된다.
- 주소(pubkeh hash)는 주소 버전 번호, 공개키의 해쉬값, 체크섬을 base58로 인코딩한 값으로 QR코드로 인코딩 될 수 있다.

### 앨리스가 밥의 주소로 송금하는 과정

- 앨리스 지갑이 앨리스의 주소로 잠겨 있는 `output A`를 풀 수 있는 앨리스의 개인키가 포함된 `scriptSig`를 포함하는 `input A`를 생성
- 밥의 주소로 잠그는 `scriptPubKey`를 포함하는 `output B`를 생성
- `intput A`와 `output B`가 담긴 `tx T`을 네트워크에 전파하고 블록에 추가
- 블록의 작업 증명이 완료되면
- 풀 노드에 있는 UTXO set에서는 `output A`가 제거되고 `output B`가 새로 추가
- 밥의 주소를 알고 있는 밥의 지갑은 `output B`의 `amount`를 밥이 사용할 수 있는 금액으로 인식

### pubkeyScript

- 송금자가 수금자의 pubkey hash를 이용해서 만드는 스크립트로 `output`에 담기며 이 `output`이 나중에 지불될 때 실행되는 스크립트 ???
- 지불될 때 개인키를 입력받고 개인키에서 생성된 full pub key로 해쉬계산을 해서 pubkey hash 값이 나오는지 확인

### sigScript

- 송금자가 자신의 private key를 이용해서 `output`을 지불하기 위한 서명 과정에서 실행되는 스크립트로 `signature`와 `pubKey`를 `output`에 담겨있던 `pubkeyScript`에 제공하고 `pubkeyScript`를 실행한다.

### P2PKH

- Pay to Public Key Hash
- 수금자의 단순 Public Key Hash에 송금

### P2SH

- Pay to Script Hash
- MultiSig 등 수금자가 실행할 스크립트의 Hash에 송금

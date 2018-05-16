Output이 내가 쓸 수 있는 코인

CTxOut : 잠그는 역할

코인베이스 Tx
Input이 0
소프트 포크 투표에도 사용 - CoinbaseCommitment
extra nonce
100컨펌(특정한 시각?)이 지나야 사용 가능

UTXO set: unspent를 역사를 뒤지지 않고도 쉽게 확인 가능하게 해줌
코드 읽기 어렵다
각 노드의 메모리에 저장
디비의 인덱스와 비슷한 역할
긴 블록에 밀릴 때 원복하는 코드도 복잡
coins.h, validation.h

Mem pool
아직 생성 블록에 포함되지 않은 트랙젝션의 목록
긴 체인에 의해 제외될 수 있으므로 undo 기능 필요
코드 복잡

## 포크



비트코인 개발자 가이드 가 엄청 도움 많이 됨
소스 코드 해설서




## 

ScriptPubkey: 잠금 스크립트





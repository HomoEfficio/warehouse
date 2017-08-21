# Ethererum dApp 만들기

거대한 분산 장부로서 기록의 멸실이나 위/변조 발생 가능성을 최소화시켜서 은행, 기업, 기관, 정부 등의 중앙 집중적인 제3자의 보증 없이도 당사자끼리 직접 신뢰할 수 있게 해주는 블록 체인 위에 스마트 계약(Smart Contract)를 얹고 실행할 수 있게 해주는 dApp(Decentralized Application)을 만들어 보자.

블록 체인에 대한 기본적인 구동 원리는 %%%를 참고하고, dApp을 이미 만들어져 있는 Ethereum의 TestNet 상에서 만들어 볼 수도 있지만, 여기에서는 아예 바닥부터 시작해서 나만의 Private Ethereum 블록 체인을 만들고, 그 위에서 dApp을 개발하는 방식으로 진행한다. 크게 다음의 순서대로 간략하게 다루고자 한다.

1. 개발 환경 구성 및 확인
2. 계정(Account) 생성 및 확인
3. 원조 블록(Genesis Block)의 생성 및 블록 체인 활성화
4. JSON RPC 를 이용한 송금 및 확인

---

# 개발 환경 구성 및 확인

개발 환경은 MacOS를 기준으로 한다.

## Go 설치

https://golang.org/dl/ 에서 Apple macOS 용 설치 파일을 다운로드 받아 설치한다.

![Imgur](http://i.imgur.com/uWGRWFz.png)

아래와 같이 설치 버전을 확인할 수 있다.

```
homo.efficio ~/study/ethereum
🍺  go version
go version go1.8.3 darwin/amd64
```

## geth 설치

### 소스에서 빌드

Git Repository에서 소스를 받아와서 직접 빌드할 수 있다. 아무 위치에 다운로드 받아도 관계 없으나 여기에서는 `~/gitRepo/ethereum/go-ethereum`라는 폴더에 다운로드 한다.

```
homo.efficio ~/gitRepo/ethereum
🍺  git clone https://github.com/ethereum/go-ethereum
Cloning into 'go-ethereum'...
remote: Counting objects: 63946, done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 63946 (delta 0), reused 0 (delta 0), pack-reused 63944
Receiving objects: 100% (63946/63946), 84.97 MiB | 264.00 KiB/s, done.
Resolving deltas: 100% (42676/42676), done.
Checking connectivity... done.
```
다운도르가 완료되면 아래와 같이 소스 디렉토리에 가서 `make geth` 명령으로 빌드 한다. 빌드하려면 Go 가 먼저 설치되어 있어야 한다.

```
homo.efficio ~/gitRepo/ethereum
🍺  cd go-ethereum/
homo.efficio ~/gitRepo/ethereum/go-ethereum
🍺  make geth
build/env.sh go run build/ci.go install ./cmd/geth
>>> /usr/local/go/bin/go install -ldflags -X main.gitCommit=bf1e2631281e1e439533f2abcf1e99a7b2f9552a -s -v ./cmd/geth
github.com/ethereum/go-ethereum/crypto/sha3
github.com/ethereum/go-ethereum/common/math

...

github.com/ethereum/go-ethereum/cmd/utils
github.com/ethereum/go-ethereum/cmd/geth
Done building.
Run "/Users/사용자계정이름/gitRepo/ethereum/go-ethereum/build/bin/geth" to launch geth.
```

`~/.bashrc`에 다음과 같이 alias를 만들어준다.

```
# geth
alias geth='~/gitRepo/ethereum/go-ethereum/build/bin/geth'
```

### Homebrew로 설치

아래와 같이 `brew` 명령으로 설치할 수도 있다.

```
brew tap ethereum/ethereum
brew install ethereum
```

### 설치 확인

`geth version`으로 설치 버전을 확인한다.

```
homo.efficio ~/gitRepo/ethereum/go-ethereum
🍺  geth version
Geth
Version: 1.7.0-unstable
Git Commit: bf1e2631281e1e439533f2abcf1e99a7b2f9552a
Architecture: amd64
Protocol Versions: [63 62]
Network Id: 1
Go Version: go1.8.3
Operating System: darwin
GOPATH=
GOROOT=/usr/local/go
```

## 개발 디렉토리 지정

`~/study/ethereum`을 최상위 기본 디렉토리로 하고, 그 아래에 `private-chain` 디렉토리를 만들어서 여기에 블록 체인 관련 데이터를 저장한다.

```
homo.efficio ~/study/ethereum
🍺  ll
total 0
drwxr-xr-x  3 1003604  staff  102  8 20 22:55 ./
drwxr-xr-x  8 1003604  staff  272  8 12 10:48 ../
drwxr-xr-x  2 1003604  staff   68  8 20 22:55 private-chain/
```

# 계정 생성 및 확인

이더리움 상에서 이더를 주고 받으려면 주는 계정과 받는 계정 이렇게 2개의 있어야 한다. 다음 명령으로 계정을 생성한다.

```
homo.efficio ~/study/ethereum
🍺  geth account new --datadir private-chain/
WARN [08-20|22:59:18] No etherbase set and no accounts found as default
Your new account is locked with a password. Please give a password. Do not forget this password.
Passphrase:
Repeat passphrase:
Address: {9cd6341e4d4de6651b0a498e28a90b1538695a8b}
```

`Address`라고 표시되어 있는 내용이 실제 이더를 주고 받을 때 사용되는, 은행으로 치면 계좌 번호 같은 공개 주소다. 비밀번호를 지정해서 계정 생성을 마치면 생성된 계정 정보는 `--datadir private-chain/` 옵션에 의해 아래와 같이 `private-chain/keystore/`에 생성된다. 옵션을 주지 않으면 계정 데이터는 디폴트로 `~/Library/Ethereum/keystore/`에 생성된다.

```
homo.efficio ~/study/ethereum
🍺  ll private-chain/keystore/
total 8
drwx------  3 1003604  staff  102  8 20 22:59 ./
drwxr-xr-x  3 1003604  staff  102  8 20 22:59 ../
-rw-------  1 1003604  staff  491  8 20 22:59 UTC--2017-08-20T13-59-24.809325230Z--9cd6341e4d4de6651b0a498e28a90b1538695a8b
```

계정을 하나 더 생성한다.

```
homo.efficio ~/study/ethereum
🍺  geth account new --datadir private-chain/
Your new account is locked with a password. Please give a password. Do not forget this password.
Passphrase:
Repeat passphrase:
Address: {b81cd479e1660897e00375142ee12cd66bb3087d}
```

생성된 계정은 다음과 같이 `geth account list` 명령으로도 확인할 수 있다.

```
homo.efficio ~/study/ethereum
🍺  geth account list --datadir private-chain/
Account #0: {9cd6341e4d4de6651b0a498e28a90b1538695a8b} keystore:///Users/1003604/study/ethereum/private-chain/keystore/UTC--2017-08-20T13-59-24.809325230Z--9cd6341e4d4de6651b0a498e28a90b1538695a8b
Account #1: {b81cd479e1660897e00375142ee12cd66bb3087d} keystore:///Users/1003604/study/ethereum/private-chain/keystore/UTC--2017-08-20T14-05-17.816233442Z--b81cd479e1660897e00375142ee12cd66bb3087d
```

# Genesis Block의 생성 및 블록 체인 활성화

블록 체인은 블록이 링크드 리스트로 이어진 자료 구조다. 따라서 최초 블록인 원조 블록(Genesis Block)은 직접 만들어야 한다. 원조 블록에 대한 정보는 `genesis 파일`이라고 부르는 JSON 파일로 설정해 줄 수 있으며, 여기에서는 `custom-genesis.json`이라는 파일에 간략한 항목만으로 작성한다.

## Genesis Block 설정 파일

```json
{
    "config": {
        "chainId": 23,
        "homesteadBlock": 0,
        "eip155Block": 0,
        "eip158Block": 0
    },
    "difficulty": "0x400000",
    "gasLimit": "2100000",
    "alloc": {
        "9cd6341e4d4de6651b0a498e28a90b1538695a8b": { "balance": "100000000" },
        "b81cd479e1660897e00375142ee12cd66bb3087d": { "balance": "300000000" }
    }
}
```

설정 내용은 다음과 같다.

### config

원조 블록의 각종 설정 값을 저장한다. 전체 설정 항목은 https://github.com/ethereum/go-ethereum/blob/feeccdf4ec1084b38dac112ff4f86809efd7c0e5/params/config.go#L71 에 있으며, 직접 지정하지 않으면 디폴트 값이 설정된다.

여기에서 사용한 항목과 설명은 다음과 같다.

- chainId: 블록 체인을 식별하는 integer 값. 예약된 값(https://ethereum.stackexchange.com/a/17101) 외의 값을 선정한다.
- homesteadBlock: Homestead 버전 적용을 위해 하드 포크(hard fork)된 블록의 번호. 새로 만들 private 블록 체인은 원조 블록부터 Homestead 버전이므로 0으로 지정
- eip155Block: EIP155가 적용되어 하드 포크된 블록의 번호. homesteadBlock과 마찬가지 이유로 private 블록 체인에서는 0으로 지정
- eip158Block: EIP158가 적용된 하드 포크된 블록의 번호. homesteadBlock과 마찬가지 이유로 private 블록 체인에서는 0으로 지정

참고로 실제 이더리움 Public Network의 설정 내용은 https://github.com/ethereum/go-ethereum/blob/feeccdf4ec1084b38dac112ff4f86809efd7c0e5/params/config.go#L28 에 있으며, 설정값은 https://github.com/ethereum/go-ethereum/blob/09777952ee476ff80d4b6e63b5041ff5ca0e441b/params/util.go#L25 에 있다.

### difficulty

블록 생성(채굴) 난이도를 지정한다. 큰 값을 지정하면 블록 생성에 오랜 시간이 걸리며 Mac Book Pro 13 기준으로 `0x400000`으로 지정하면 대략 30초 ~ 1분 사이에 블록이 생성되므로 테스트 용으로 적합하다.

### gasLimit

이더리움에서는 거래에 필요한 연산을 수행할 때마다 연산의 종류에 따라 정해진 비용을 지불해야 하는데 이 비용을 `gas`라고 한다. 예를 들어 더하기와 빼기는 3 gas가 소요되며, 곱하기와 나누기에는 5 gas가 소요된다. `gas`도 결국에는 `1 gas = 0.00000002 Ether`와 같이 이더로 지급하는데, 비용의 단위를 이더로 하지 않고 굳이 `gas`라는 개념을 중간에 둔 이유는 이더의 가치 변동이 크기 때문이다. 

이더를 보내는 쪽에서 지불할 의향이 있는 `gas`의 한도를 `gasLimit`이라고 하며, 어떤 거래를 처리할 때 사용되는 연산에 필요한 `gas`의 양이 이 `gasLimit`을 초과하면, 그 거래의 처리는 완료되지 않고 중단되며 블록에도 포함되지 않는다. `gasLimit`은 고의 또는 실수로 인한 무한루프의 실행을 막는 역할을 한다.

블록에서의 `gasLimit`는 하나의 블록에 포함된 거래의 `gasLimit`의 총합의 한도를 의미한다. 이 값이 너무 커지면 블록의 처리에 많은 연산이 필요하게 되어 처리가 늦어지게 되므로 적정한 한도를 둔다.

genesis 파일에서 사용된 `gasLimit`은 블록에서의 `gasLimit`을 의미한다.

현재 운영 중인 이더리움 네트워크에서 사용되는 `gas`의 정보는 http://ethgasstation.info/ 에서 확인할 수 있으며, `gas`에 대한 자세한 내용은 [Yellow Paper](https://ethereum.github.io/yellowpaper/paper.pdf) 나 https://ethereum.stackexchange.com/questions/3/what-is-meant-by-the-term-gas 를 참고한다.

### alloc

원조 블록 생성 시 특정 계좌에 정해진 액수의 이더를 지급할 때 사용한다.

## Genesis Block의 생성

genesis 파일 작성이 완료되면 `geth init` 명령으로 원조 블록을 생성할 수 있다.

```
homo.efficio ~/study/ethereum
🍺  geth init custom-genesis.json --datadir private-chain/
INFO [08-21|00:41:15] Allocated cache and file handles         database=/Users/1003604/study/ethereum/private-chain/geth/chaindata cache=16 handles=16
INFO [08-21|00:41:15] Writing custom genesis block
INFO [08-21|00:41:15] Successfully wrote genesis state         database=chaindata                                                  hash=ac90de…b5abe9
INFO [08-21|00:41:15] Allocated cache and file handles         database=/Users/1003604/study/ethereum/private-chain/geth/lightchaindata cache=16 handles=16
INFO [08-21|00:41:15] Writing custom genesis block
INFO [08-21|00:41:15] Successfully wrote genesis state         database=lightchaindata                                                  hash=ac90de…b5abe9
```

원조 블록은 물리적으로 `geth` 디렉토리에 생성된다.

```
homo.efficio ~/study/ethereum
🍺  ll private-chain/
total 0
drwxr-xr-x  4 1003604  staff  136  8 21 00:41 ./
drwxr-xr-x  4 1003604  staff  136  8 21 00:40 ../
drwxr-xr-x  4 1003604  staff  136  8 21 00:41 geth/
drwx------  4 1003604  staff  136  8 20 23:05 keystore/
```

나중에 블록 체인을 물리적으로 다른 곳으로 옮길 때는 블록 체인 정보가 저장되는 `geth` 디렉토리와 계정 정보가 저장되는 `keystore` 디렉토리를 포함하고 있는 디렉토리(여기에서는 `private-chain`)를 옮기면 된다.

## 블록 체인 활성화

원조 블록이 생성되었으므로 이제 이 원조 블록에 새 블록을 이어 붙이면서 블록 체인을 구동할 준비가 되었다. `geth` 명령으로 블록 체인을 구동할 수 있다.

옵션에 대한 설명은 다음과 같다.

- `--datadir private-chain`: 
- `--ethash.dagdir private-chain/.ethash`:

...

```
homo.efficio ~/study/ethereum
🍺  geth --datadir private-chain \
>   --ethash.dagdir private-chain/.ethash \
>   --identity "MyEthNode01" \
>   --mine \
>   --minerthreads 4 \
>   --etherbase 9cd6341e4d4de6651b0a498e28a90b1538695a8b \
>   --networkid 23 \
>   --nodiscover \
>   --maxpeers 0 \
>   --rpc \
>   --rpcapi "db,eth,net,web3" \
>   --rpccorsdomain "*"
INFO [08-21|00:57:50] Starting peer-to-peer node               instance=Geth/MyEthNode01/v1.7.0-unstable-6ca59d98/darwin-amd64/go1.8.3
INFO [08-21|00:57:50] Allocated cache and file handles         database=/Users/1003604/study/ethereum/private-chain/geth/chaindata cache=128 handles=1024
WARN [08-21|00:57:50] Upgrading database to use lookup entries
INFO [08-21|00:57:50] Database deduplication successful        deduped=0
INFO [08-21|00:57:50] Initialised chain configuration          config="{ChainID: 23 Homestead: 0 DAO: <nil> DAOSupport: false EIP150: <nil> EIP155: 0 EIP158: 0 Metropolis: <nil> Engine: unknown}"
INFO [08-21|00:57:50] Disk storage enabled for ethash caches   dir=/Users/1003604/study/ethereum/private-chain/geth/ethash count=3
INFO [08-21|00:57:50] Disk storage enabled for ethash DAGs     dir=private-chain/.ethash                                   count=2
WARN [08-21|00:57:50] Upgrading db log bloom bins
INFO [08-21|00:57:50] Bloom-bin upgrade completed              elapsed=71.302µs
INFO [08-21|00:57:50] Initialising Ethereum protocol           versions="[63 62]" network=23
INFO [08-21|00:57:50] Loaded most recent local header          number=0 hash=ac90de…b5abe9 td=4194304
INFO [08-21|00:57:50] Loaded most recent local full block      number=0 hash=ac90de…b5abe9 td=4194304
INFO [08-21|00:57:50] Loaded most recent local fast block      number=0 hash=ac90de…b5abe9 td=4194304
INFO [08-21|00:57:50] Regenerated local transaction journal    transactions=0 accounts=0
INFO [08-21|00:57:50] Starting P2P networking
INFO [08-21|00:57:50] RLPx listener up                         self="enode://aad004f194f5a9d6fd557969c95c1ab15daeefd8dc53e946db8377eca51277365274c7fdd03cb512261a3ae5ca4617c2c01fd1663f989b55607fc62c836b183c@[::]:30303?discport=0"
INFO [08-21|00:57:50] IPC endpoint opened: /Users/1003604/study/ethereum/private-chain/geth.ipc
INFO [08-21|00:57:50] HTTP endpoint opened: http://127.0.0.1:8545
INFO [08-21|00:57:50] Transaction pool price threshold updated price=18000000000
INFO [08-21|00:57:50] Starting mining operation
INFO [08-21|00:57:50] Commit new mining work                   number=1 txs=0 uncles=0 elapsed=157.156µs
INFO [08-21|00:57:52] Mapped network port                      proto=tcp extport=30303 intport=30303 interface="UPNP IGDv1-IP1"
INFO [08-21|00:57:52] Generating DAG in progress               epoch=0 percentage=0 elapsed=1.635s
INFO [08-21|00:57:54] Generating DAG in progress               epoch=0 percentage=1 elapsed=3.214s
INFO [08-21|00:57:55] Generating DAG in progress               epoch=0 percentage=2 elapsed=4.840s

...

INFO [08-21|01:01:00] Generating DAG in progress               epoch=0 percentage=98 elapsed=3m9.335s
INFO [08-21|01:01:02] Generating DAG in progress               epoch=0 percentage=99 elapsed=3m11.760s
INFO [08-21|01:01:02] Generated ethash verification cache      epoch=0 elapsed=3m11.763s
```

실행하면 genesis 파일과 실행 옵션으로 지정한 블록 체인의 정보가 표시되며, 블록 체인을 최초로 실행할 때는 `Generating DAG in progress`로 표시되는 것처럼 DAG가 생성된다.

`geth`를 실행한 터미널 말고 다른 터미널을 열어서 확인해보면 디렉토리와 파일이 몇 개 더 생긴 것을 확인할 수 있다.

```
homo.efficio ~/study/ethereum
🍺  ll private-chain/
total 0
drwxr-xr-x  6 1003604  staff  204  8 21 00:57 ./
drwxr-xr-x  4 1003604  staff  136  8 21 00:40 ../
drwxr-xr-x  4 1003604  staff  136  8 21 01:07 .ethash/
drwxr-xr-x  7 1003604  staff  238  8 21 00:57 geth/
srw-------  1 1003604  staff    0  8 21 00:57 geth.ipc=
drwx------  4 1003604  staff  136  8 20 23:05 keystore/
```

DAG는 `.ethash` 디렉토리 안에 생성된다.

`geth`를 실행한 터미널에서는 두 번째 DAG가 생성되며, 중간중간 블록도 생성된다.

```
...

INFO [08-21|01:01:32] Generating DAG in progress               epoch=1 percentage=6  elapsed=26.190s
INFO [08-21|01:01:35] Successfully sealed new block            number=1 hash=976032…350d9e
INFO [08-21|01:01:35] 🔨 mined potential block

...
```

다음과 같이 두 번째 DAG 생성이 완료되면 그 후에는 다음과 같이 블록 생성만 계속 이어진다.

```
...

INFO [08-21|01:07:26] Generating DAG in progress               epoch=1 percentage=98 elapsed=6m20.528s
INFO [08-21|01:07:30] Generating DAG in progress               epoch=1 percentage=99 elapsed=6m24.023s
INFO [08-21|01:07:30] Generated ethash verification cache      epoch=1 elapsed=6m24.027s
INFO [08-21|01:07:38] Successfully sealed new block            number=11 hash=1e06b0…5a67d7
INFO [08-21|01:07:38] 🔗 block reached canonical chain          number=6  hash=5a445c…15cac2
INFO [08-21|01:07:38] 🔨 mined potential block                  number=11 hash=1e06b0…5a67d7
INFO [08-21|01:07:38] Commit new mining work                   number=12 txs=0 uncles=0 elapsed=108.738µs
INFO [08-21|01:07:52] Successfully sealed new block            number=12 hash=cf444f…54d372
INFO [08-21|01:07:52] 🔗 block reached canonical chain          number=7  hash=6c8c87…2cc785
INFO [08-21|01:07:52] 🔨 mined potential block                  number=12 hash=cf444f…54d372

...
```

블록 체인은 `CTRL+C`로 아래와 같이 실행을 멈출 수 있다. 

```
INFO [08-21|01:22:20] Commit new mining work                   number=55 txs=0 uncles=0 elapsed=235.586µs
^CINFO [08-21|01:22:27] Got interrupt, shutting down...
INFO [08-21|01:22:27] HTTP endpoint closed: http://127.0.0.1:8545
INFO [08-21|01:22:27] IPC endpoint closed: /Users/1003604/study/ethereum/private-chain/geth.ipc
INFO [08-21|01:22:27] Blockchain manager stopped
INFO [08-21|01:22:27] Stopping Ethereum protocol
INFO [08-21|01:22:27] Ethereum protocol stopped
INFO [08-21|01:22:27] Transaction pool stopped
INFO [08-21|01:22:27] Database closed                          database=/Users/1003604/study/ethereum/private-chain/geth/chaindata
```

위 로그를 보면 55번째 블록 생성 중 실행이 멈췄는데, 앞에서 블록 체인을 구동했던 명령을 다시 실행하면 아래와 같이 55번째 블록부터 다시 생성한다.

```
homo.efficio ~/study/ethereum
🍺  geth --datadir private-chain \
  --ethash.dagdir private-chain/.ethash \
  --identity "MyEthNode01" \
  --mine \
  --minerthreads 4 \
  --etherbase 9cd6341e4d4de6651b0a498e28a90b1538695a8b \
  --networkid 23 \
  --nodiscover \
  --maxpeers 0 \
  --rpc \
  --rpcapi "db,eth,net,web3" \
  --rpccorsdomain "*"
INFO [08-21|01:23:22] Starting peer-to-peer node               instance=Geth/MyEthNode01/v1.7.0-unstable-6ca59d98/darwin-amd64/go1.8.3
INFO [08-21|01:23:22] Allocated cache and file handles         database=/Users/1003604/study/ethereum/private-chain/geth/chaindata cache=128 handles=1024
INFO [08-21|01:23:22] Initialised chain configuration          config="{ChainID: 23 Homestead: 0 DAO: <nil> DAOSupport: false EIP150: <nil> EIP155: 0 EIP158: 0 Metropolis: <nil> Engine: unknown}"
INFO [08-21|01:23:22] Disk storage enabled for ethash caches   dir=/Users/1003604/study/ethereum/private-chain/geth/ethash count=3
INFO [08-21|01:23:22] Disk storage enabled for ethash DAGs     dir=private-chain/.ethash                                   count=2
INFO [08-21|01:23:22] Initialising Ethereum protocol           versions="[63 62]" network=23
INFO [08-21|01:23:22] Loaded most recent local header          number=54 hash=898f50…f78592 td=214641335
INFO [08-21|01:23:22] Loaded most recent local full block      number=54 hash=898f50…f78592 td=214641335
INFO [08-21|01:23:22] Loaded most recent local fast block      number=54 hash=898f50…f78592 td=214641335
INFO [08-21|01:23:22] Loaded local transaction journal         transactions=0 dropped=0
INFO [08-21|01:23:22] Regenerated local transaction journal    transactions=0 accounts=0
WARN [08-21|01:23:22] Blockchain not empty, fast sync disabled
INFO [08-21|01:23:22] Starting P2P networking
INFO [08-21|01:23:22] RLPx listener up                         self="enode://aad004f194f5a9d6fd557969c95c1ab15daeefd8dc53e946db8377eca51277365274c7fdd03cb512261a3ae5ca4617c2c01fd1663f989b55607fc62c836b183c@[::]:30303?discport=0"
INFO [08-21|01:23:22] IPC endpoint opened: /Users/1003604/study/ethereum/private-chain/geth.ipc
INFO [08-21|01:23:22] HTTP endpoint opened: http://127.0.0.1:8545
INFO [08-21|01:23:22] Transaction pool price threshold updated price=18000000000
INFO [08-21|01:23:22] Starting mining operation
INFO [08-21|01:23:22] Commit new mining work                   number=55 txs=0 uncles=0 elapsed=396.617µs
INFO [08-21|01:23:25] Mapped network port                      proto=tcp extport=30303 intport=30303 interface="UPNP IGDv1-IP1"
INFO [08-21|01:23:55] Successfully sealed new block            number=55 hash=fea353…30e440
INFO [08-21|01:23:55] 🔨 mined potential block                  number=55 hash=fea353…30e440
INFO [08-21|01:23:55] Commit new mining work                   number=56 txs=0 uncles=0 elapsed=310.238µs
```

참고로 현재 운영 중인 이더리움의 블록 체인 정보는 https://ethstats.net/ 에서 확인할 수 있다.

# JSON RPC 를 이용한 송금 및 확인


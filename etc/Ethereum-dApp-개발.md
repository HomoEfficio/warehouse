# Ethererum dApp 만들기

거대한 분산 장부로서 기록의 멸실이나 위/변조 발생 가능성을 최소화시켜서 은행, 기업, 기관, 정부 등의 중앙 집중적인 제3자의 보증 없이도 당사자끼리 직접 신뢰할 수 있게 해주는 블록 체인 위에 스마트 계약(Smart Contract)를 얹고 실행할 수 있게 해주는 dApp(Decentralized Application)을 만들어 보자.

블록 체인에 대한 기본적인 구동 원리는 %%%를 참고하고, dApp을 이미 만들어져 있는 Ethereum의 TestNet 상에서 만들어 볼 수도 있지만, 여기에서는 아예 바닥부터 시작해서 나만의 Private Ethereum 블록 체인을 만들고, 그 위에서 dApp을 개발하는 방식으로 진행 한다. 크게 다음의 순서대로 간략하게 다루고자 한다.

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

`~/study/ethereum`을 최상위 기본 디렉토리로 하고, 그 아래에 `private-chain` 디렉토리를 만들어서 여기에 블록 체인 관련 모든 데이터를 저장해서 `private-chain`를 일종의 sandbox로 만들어서 작업한다.

```
homo.efficio ~/study/ethereum
🍺  ll
total 0
drwxr-xr-x  3 1003604  staff  102  8 26 10:40 ./
drwxr-xr-x  8 1003604  staff  272  8 12 10:48 ../
drwxr-xr-x  2 1003604  staff   68  8 26 10:40 private-chain/
```

# 계정 생성 및 확인

이더리움 상에서 이더를 주고 받으려면 주는 계정과 받는 계정 이렇게 2개의 있어야 한다. 다음 명령으로 계정을 생성한다.

```
homo.efficio ~/study/ethereum
🍺  geth account new --datadir private-chain/
WARN [08-26|10:41:24] No etherbase set and no accounts found as default
Your new account is locked with a password. Please give a password. Do not forget this password.
Passphrase:
Repeat passphrase:
Address: {4f27ca4553225cfdddb96fdb89346e7565e61627}
```

`Address`라고 표시되어 있는 내용이 실제 이더를 주고 받을 때 사용되는, 은행으로 치면 계좌 번호 같은 공개 주소다. 비밀번호를 지정해서 계정 생성을 마치면 생성된 계정 정보는 `--datadir private-chain/` 옵션에 의해 아래와 같이 `private-chain/keystore/`에 생성된다. 옵션을 주지 않으면 계정 데이터는 디폴트로 `~/Library/Ethereum/keystore/`에 생성된다.

```
homo.efficio ~/study/ethereum
🍺  ll private-chain/keystore/
total 8
drwx------  3 1003604  staff  102  8 26 10:41 ./
drwxr-xr-x  3 1003604  staff  102  8 26 10:41 ../
-rw-------  1 1003604  staff  491  8 26 10:41 UTC--2017-08-26T01-41-31.092111708Z--4f27ca4553225cfdddb96fdb89346e7565e61627
```

계정을 하나 더 생성한다.

```
homo.efficio ~/study/ethereum
🍺  geth account new --datadir private-chain/
Your new account is locked with a password. Please give a password. Do not forget this password.
Passphrase:
Repeat passphrase:
Address: {8128ff5a55370dc9555e5c18b716050bdf2c8440}
```

생성된 계정은 다음과 같이 `geth account list` 명령으로도 확인할 수 있다.

```
homo.efficio ~/study/ethereum
🍺  geth account list --datadir private-chain/
Account #0: {4f27ca4553225cfdddb96fdb89346e7565e61627} keystore:///Users/1003604/study/ethereum/private-chain/keystore/UTC--2017-08-26T01-41-31.092111708Z--4f27ca4553225cfdddb96fdb89346e7565e61627
Account #1: {8128ff5a55370dc9555e5c18b716050bdf2c8440} keystore:///Users/1003604/study/ethereum/private-chain/keystore/UTC--2017-08-26T01-42-30.477728967Z--8128ff5a55370dc9555e5c18b716050bdf2c8440
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
        "4f27ca4553225cfdddb96fdb89346e7565e61627": { "balance": "100000000" },
        "8128ff5a55370dc9555e5c18b716050bdf2c8440": { "balance": "300000000" }
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

블록 생성(채굴) 난이도를 지정한다. 큰 값을 지정하면 블록 생성에 오랜 시간이 걸리며 Mac Book Pro 13 기준으로 `0x400000`으로 지정하면 대략 30초 ~ 1분 사이에 블록이 생성되므로 테스트 용으로 적당하다.

### gasLimit

이더리움에서는 거래에 필요한 연산을 수행할 때마다 연산의 종류에 따라 정해진 비용을 지불해야 하는데 이 비용을 `gas`라고 한다. 예를 들어 더하기와 빼기는 3 gas가 소요되며, 곱하기와 나누기에는 5 gas가 소요된다. `gas`도 결국에는 `1 gas = 0.00000002 Ether`와 같이 이더로 지급하는데, 비용의 단위를 이더로 하지 않고 굳이 `gas`라는 개념을 중간에 둔 이유는 이더의 가치 변동이 크기 때문이다. 

이더를 보내는 쪽에서 지불할 의향이 있는 `gas`의 한도를 `gasLimit`이라고 하며, 어떤 거래를 처리할 때 사용되는 연산에 필요한 `gas`의 양이 이 `gasLimit`을 초과하면, 그 거래의 처리는 완료되지 않고 중단되며 블록에도 포함되지 않는다. `gasLimit`은 고의 또는 실수로 인한 무한루프의 실행을 막는 역할을 한다.

블록에서의 `gasLimit`는 하나의 블록에 포함된 거래의 `gasLimit`의 총합의 한도를 의미한다. 이 값이 너무 커지면 블록의 처리에 많은 연산이 필요하게 되어 처리가 늦어지게 되므로 적정한 한도를 둔다.

genesis 파일에서 사용된 `gasLimit`은 블록에서의 `gasLimit`을 의미한다.

현재 운영 중인 이더리움 네트워크에서 사용되는 `gas`의 정보는 http://ethgasstation.info/ 에서 확인할 수 있으며, `gas`에 대한 자세한 내용은 [Yellow Paper](https://ethereum.github.io/yellowpaper/paper.pdf) 나 https://ethereum.stackexchange.com/questions/3/what-is-meant-by-the-term-gas 를 참고한다.

### alloc

원조 블록 생성 시 특정 계정에 정해진 액수의 이더를 지급할 때 사용한다. 계정 주소를 key로 해서 최초 지급액을 `balance`에 지정한다. 

## Genesis Block의 생성

genesis 파일 작성이 완료되면 `geth init` 명령으로 원조 블록(genesis block)을 생성할 수 있다.

```
homo.efficio ~/study/ethereum
🍺  geth init custom-genesis.json --datadir private-chain/
INFO [08-26|10:48:33] Allocated cache and file handles         database=/Users/1003604/study/ethereum/private-chain/geth/chaindata cache=16 handles=16
INFO [08-26|10:48:33] Writing custom genesis block
INFO [08-26|10:48:33] Successfully wrote genesis state         database=chaindata                                                  hash=11966f…3d3ab9
INFO [08-26|10:48:33] Allocated cache and file handles         database=/Users/1003604/study/ethereum/private-chain/geth/lightchaindata cache=16 handles=16
INFO [08-26|10:48:33] Writing custom genesis block
INFO [08-26|10:48:33] Successfully wrote genesis state         database=lightchaindata                                                  hash=11966f…3d3ab9
```

원조 블록은 물리적으로 `geth` 디렉토리에 생성된다.

```
homo.efficio ~/study/ethereum
🍺  ll private-chain/
total 0
drwxr-xr-x  4 1003604  staff  136  8 26 10:48 ./
drwxr-xr-x  4 1003604  staff  136  8 26 10:44 ../
drwxr-xr-x  4 1003604  staff  136  8 26 10:48 geth/
drwx------  4 1003604  staff  136  8 26 10:42 keystore/
```

나중에 블록 체인을 물리적으로 다른 곳으로 옮길 때는 블록 체인 정보가 저장되는 `geth` 디렉토리와 계정 정보가 저장되는 `keystore` 디렉토리를 포함하고 있는 디렉토리(여기에서는 sandbox인 `private-chain`)를 옮기면 된다.

## 블록 체인 활성화

원조 블록이 생성되었으므로 이제 이 원조 블록에 새 블록을 이어 붙이면서 블록 체인을 구동할 준비가 되었다. `geth` 명령으로 블록 체인을 구동할 수 있다.

옵션에 대한 설명은 다음과 같다.

%%% 구동 옵션 설명 %%%

- `--datadir private-chain`: 
- `--ethash.dagdir private-chain/.ethash`:

...

geth --datadir private-chain \
  --ethash.dagdir private-chain/.ethash \
  --identity "MyEthNode01" \
  --mine \
  --minerthreads 4 \
  --etherbase 4f27ca4553225cfdddb96fdb89346e7565e61627 \
  --networkid 23 \
  --nodiscover \
  --maxpeers 0 \
  --rpc \
  --rpcapi "db,eth,net,web3" \
  --rpccorsdomain "*"


```
homo.efficio ~/study/ethereum
🍺  geth --datadir private-chain \
>   --ethash.dagdir private-chain/.ethash \
>   --identity "MyEthNode01" \
>   --mine \
>   --minerthreads 4 \
>   --etherbase 4f27ca4553225cfdddb96fdb89346e7565e61627 \
>   --networkid 23 \
>   --nodiscover \
>   --maxpeers 0 \
>   --rpc \
>   --rpcapi "db,eth,net,web3" \
>   --rpccorsdomain "*"
INFO [08-26|10:51:43] Starting peer-to-peer node               instance=Geth/MyEthNode01/v1.7.0-unstable-bf1e2631/darwin-amd64/go1.8.3
INFO [08-26|10:51:43] Allocated cache and file handles         database=/Users/1003604/study/ethereum/private-chain/geth/chaindata cache=128 handles=1024
WARN [08-26|10:51:43] Upgrading database to use lookup entries
INFO [08-26|10:51:43] Database deduplication successful        deduped=0
INFO [08-26|10:51:43] Initialised chain configuration          config="{ChainID: 23 Homestead: 0 DAO: <nil> DAOSupport: false EIP150: <nil> EIP155: 0 EIP158: 0 Metropolis: <nil> Engine: unknown}"
INFO [08-26|10:51:43] Disk storage enabled for ethash caches   dir=/Users/1003604/study/ethereum/private-chain/geth/ethash count=3
INFO [08-26|10:51:43] Disk storage enabled for ethash DAGs     dir=private-chain/.ethash                                   count=2
WARN [08-26|10:51:43] Upgrading db log bloom bins
INFO [08-26|10:51:43] Bloom-bin upgrade completed              elapsed=138.189µs
INFO [08-26|10:51:43] Initialising Ethereum protocol           versions="[63 62]" network=23
INFO [08-26|10:51:43] Loaded most recent local header          number=0 hash=11966f…3d3ab9 td=4194304
INFO [08-26|10:51:43] Loaded most recent local full block      number=0 hash=11966f…3d3ab9 td=4194304
INFO [08-26|10:51:43] Loaded most recent local fast block      number=0 hash=11966f…3d3ab9 td=4194304
INFO [08-26|10:51:43] Regenerated local transaction journal    transactions=0 accounts=0
INFO [08-26|10:51:43] Starting P2P networking
INFO [08-26|10:51:43] RLPx listener up                         self="enode://c447ea2f047334fb18559c252976a484ba4935a692996733a5731b3d6982bbdf494e4b1f0a995a53cefa4124523476e5b36de397d8e15cfb694c556efd73c80b@[::]:30303?discport=0"
INFO [08-26|10:51:43] HTTP endpoint opened: http://127.0.0.1:8545
INFO [08-26|10:51:43] IPC endpoint opened: /Users/1003604/study/ethereum/private-chain/geth.ipc
INFO [08-26|10:51:43] Transaction pool price threshold updated price=18000000000
INFO [08-26|10:51:43] Starting mining operation
INFO [08-26|10:51:43] Commit new mining work                   number=1 txs=0 uncles=0 elapsed=453.968µs
INFO [08-26|10:51:45] Mapped network port                      proto=tcp extport=30303 intport=30303 interface="UPNP IGDv1-IP1"
INFO [08-26|10:51:45] Generating DAG in progress               epoch=0 percentage=0 elapsed=1.665s
INFO [08-26|10:51:47] Generating DAG in progress               epoch=0 percentage=1 elapsed=3.279s
INFO [08-26|10:51:48] Generating DAG in progress               epoch=0 percentage=2 elapsed=4.834s

...

INFO [08-26|10:54:46] Generating DAG in progress               epoch=0 percentage=98 elapsed=3m2.833s
INFO [08-26|10:54:48] Generating DAG in progress               epoch=0 percentage=99 elapsed=3m4.779s
INFO [08-26|10:54:48] Generated ethash verification cache      epoch=0 elapsed=3m4.782s
```

실행하면 genesis 파일과 실행 옵션으로 지정한 블록 체인의 정보가 표시되며, 블록 체인을 최초로 실행할 때는 `Generating DAG in progress`로 표시되는 것처럼 DAG가 생성된다.

%%%DAG가 무엇인가 추가%%%

`geth`를 실행한 터미널 말고 다른 터미널을 열어서 확인해보면 디렉토리와 파일이 몇 개 더 생긴 것을 확인할 수 있다.

```
homo.efficio ~/study/ethereum
🍺  ll private-chain/
total 0
drwxr-xr-x  6 1003604  staff  204  8 26 10:51 ./
drwxr-xr-x  4 1003604  staff  136  8 26 10:44 ../
drwxr-xr-x  3 1003604  staff  102  8 26 10:51 .ethash/
drwxr-xr-x  7 1003604  staff  238  8 26 10:51 geth/
srw-------  1 1003604  staff    0  8 26 10:51 geth.ipc=
drwx------  4 1003604  staff  136  8 26 10:42 keystore/
```

DAG는 `.ethash` 디렉토리 안에 생성된다.

`geth`를 실행한 터미널에서는 두 번째 DAG가 생성되며, 중간중간 블록도 생성된다.

```
...
INFO [08-26|10:54:54] Generating DAG in progress               epoch=1 percentage=0  elapsed=3.236s
INFO [08-26|10:54:58] Generating DAG in progress               epoch=1 percentage=1  elapsed=6.751s
INFO [08-26|10:55:01] Generating DAG in progress               epoch=1 percentage=2  elapsed=10.163s

...

INFO [08-26|10:56:31] Generating DAG in progress               epoch=1 percentage=25 elapsed=1m40.337s
INFO [08-26|10:56:32] Successfully sealed new block            number=1 hash=69819c…e79624
INFO [08-26|10:56:32] 🔨 mined potential block                  number=1 hash=69819c…e79624

...
```

다음과 같이 두 번째 DAG 생성이 완료되면 그 후에는 다음과 같이 블록 생성만 계속 이어진다.

```
...

INFO [08-26|11:01:09] Generating DAG in progress               epoch=1 percentage=98 elapsed=6m18.225s
INFO [08-26|11:01:13] Generating DAG in progress               epoch=1 percentage=99 elapsed=6m22.063s
INFO [08-26|11:01:13] Generated ethash verification cache      epoch=1 elapsed=6m22.066s
INFO [08-26|11:01:29] Successfully sealed new block            number=12 hash=cda6e1…ac5043
INFO [08-26|11:01:29] 🔗 block reached canonical chain          number=7  hash=8184cc…5041d0
INFO [08-26|11:01:29] 🔨 mined potential block                  number=12 hash=cda6e1…ac5043
INFO [08-26|11:01:29] Commit new mining work                   number=13 txs=0 uncles=0 elapsed=110.092µs
INFO [08-26|11:02:02] Successfully sealed new block            number=13 hash=27c112…ebdaf2
INFO [08-26|11:02:02] 🔗 block reached canonical chain          number=8  hash=79948e…1eb69d
INFO [08-26|11:02:02] 🔨 mined potential block                  number=13 hash=27c112…ebdaf2

...
```

블록 체인 구동을 멈추고 싶으면 언제든지 `CTRL+C`로 멈출 수 있다. 멈추면 아래와 같이 HTTP endpoint, IPC endpoint, Blockchain manager, Ethereum protocol, Transaction pool, Database 등의 서비스가 중지되며 블록 체인 구동이 멈추게 된다.

```
INFO [08-26|11:08:11] Commit new mining work                   number=21 txs=0 uncles=0 elapsed=129.47µs
^CINFO [08-26|11:09:06] Got interrupt, shutting down...
INFO [08-26|11:09:06] HTTP endpoint closed: http://127.0.0.1:8545
INFO [08-26|11:09:06] IPC endpoint closed: /Users/1003604/study/ethereum/private-chain/geth.ipc
INFO [08-26|11:09:06] Blockchain manager stopped
INFO [08-26|11:09:06] Stopping Ethereum protocol
INFO [08-26|11:09:06] Ethereum protocol stopped
INFO [08-26|11:09:06] Transaction pool stopped
INFO [08-26|11:09:06] Database closed                          database=/Users/1003604/study/ethereum/private-chain/geth/chaindata
```

블록 체인을 다시 구동하려면 위에서 실행한 명령을 다시 실행하면 된다. 위 로그를 보면 21번째 블록 생성 중 실행이 멈췄는데, 블록 체인을 다시 구동하면 아래와 같이 21번째 블록부터 다시 생성한다.

```
homo.efficio ~/study/ethereum
🍺  geth --datadir private-chain \
>   --ethash.dagdir private-chain/.ethash \
>   --identity "MyEthNode01" \
>   --mine \
>   --minerthreads 4 \
>   --etherbase 4f27ca4553225cfdddb96fdb89346e7565e61627 \
>   --networkid 23 \
>   --nodiscover \
>   --maxpeers 0 \
>   --rpc \
>   --rpcapi "db,eth,net,web3" \
>   --rpccorsdomain "*"
INFO [08-26|11:10:16] Starting peer-to-peer node               instance=Geth/MyEthNode01/v1.7.0-unstable-bf1e2631/darwin-amd64/go1.8.3
INFO [08-26|11:10:16] Allocated cache and file handles         database=/Users/1003604/study/ethereum/private-chain/geth/chaindata cache=128 handles=1024
INFO [08-26|11:10:16] Initialised chain configuration          config="{ChainID: 23 Homestead: 0 DAO: <nil> DAOSupport: false EIP150: <nil> EIP155: 0 EIP158: 0 Metropolis: <nil> Engine: unknown}"
INFO [08-26|11:10:16] Disk storage enabled for ethash caches   dir=/Users/1003604/study/ethereum/private-chain/geth/ethash count=3
INFO [08-26|11:10:16] Disk storage enabled for ethash DAGs     dir=private-chain/.ethash                                   count=2
INFO [08-26|11:10:16] Initialising Ethereum protocol           versions="[63 62]" network=23
INFO [08-26|11:10:16] Loaded most recent local header          number=20 hash=85bb5e…38f8a5 td=82517715
INFO [08-26|11:10:16] Loaded most recent local full block      number=20 hash=85bb5e…38f8a5 td=82517715
INFO [08-26|11:10:16] Loaded most recent local fast block      number=20 hash=85bb5e…38f8a5 td=82517715
INFO [08-26|11:10:16] Loaded local transaction journal         transactions=0 dropped=0
INFO [08-26|11:10:16] Regenerated local transaction journal    transactions=0 accounts=0
WARN [08-26|11:10:16] Blockchain not empty, fast sync disabled
INFO [08-26|11:10:16] Starting P2P networking
INFO [08-26|11:10:16] RLPx listener up                         self="enode://c447ea2f047334fb18559c252976a484ba4935a692996733a5731b3d6982bbdf494e4b1f0a995a53cefa4124523476e5b36de397d8e15cfb694c556efd73c80b@[::]:30303?discport=0"
INFO [08-26|11:10:16] IPC endpoint opened: /Users/1003604/study/ethereum/private-chain/geth.ipc
INFO [08-26|11:10:16] HTTP endpoint opened: http://127.0.0.1:8545
INFO [08-26|11:10:16] Transaction pool price threshold updated price=18000000000
INFO [08-26|11:10:16] Starting mining operation
INFO [08-26|11:10:16] Commit new mining work                   number=21 txs=0 uncles=0 elapsed=189.174µs
INFO [08-26|11:10:18] Mapped network port                      proto=tcp extport=30303 intport=30303 interface="UPNP IGDv1-IP1"
INFO [08-26|11:10:30] Successfully sealed new block            number=21 hash=12aa85…7655a3
INFO [08-26|11:10:30] 🔨 mined potential block                  number=21 hash=12aa85…7655a3
INFO [08-26|11:10:30] Commit new mining work                   number=22 txs=0 uncles=0 elapsed=117.364µs
INFO [08-26|11:10:35] Successfully sealed new block            number=22 hash=40fd31…b1bb11
INFO [08-26|11:10:35] 🔨 mined potential block                  number=22 hash=40fd31…b1bb11
```

참고로 현재 운영 중인 이더리움의 블록 체인 정보는 https://ethstats.net/ 에서 확인할 수 있다.

# JSON RPC 를 이용한 송금 및 확인

블록 체인이 성공적으로 구동되고 있으므로 이제 드디어 이더를 실제로 주고 받을 수 있다. 여러가지 방법으로 이더를 주고 받을 수 있으나, 여기에서는 별다른 설정 없이 가장 간편하게 사용할 수 있는 JSON RPC로 송금을 해본다.

geth로 블록 체인을 구동할 때 `  --rpc --rpcapi "db,eth,net,web3" --rpccorsdomain "*"` 옵션을 추가했기 떄문에 RPC 서버가 기본값으로 8545포트에서 구동된다. 그리고 `curl`로 RPC 서버에 JSON 데이터를 전달해서 송금을 할 수 있다.

JSON RPC에서 사용 가능한 명령은 [Ethereum Wiki의 JSON RPC](https://github.com/ethereum/wiki/wiki/JSON-RPC)에 나와 있다.

## JSON RPC 몸풀기

먼저 몇 가지 조회 메서드로 몸을 풀어보면서 JSON RPC로 이더리움 블록 체인과 이야기를 나누는 방법을 알아보자.

### eth_getClientVersion

클라이언트의 버전과 정보를 확인할 수 있다.

```
homo.efficio ~/study/ethereum
🍺  curl -X POST --data '{"jsonrpc":"2.0","method":"web3_clientVersion","params":[],"id":67}' http://localhost:8545
```
`curl`의 `--data` 옵션으로 JSON 데이터를 서버에 보내면 응답을 받을 수 있다. JSON 데이터의 형식과 설명은 다음과 같다.

```json
{
  "jsonrpc": "2.0",  // JSON-RPC의 버전
  "method": "web3_clientVersion",  // 호출할 JSON RPC 메서드 이름 
  "params": [],  // JSON RPC 메서드에 넘겨줄 파라미터
  "id": 67  // JSON RPC 호출 식별자
}
```
위와 같이 `curl`을 실행해서 요청을 보내면 아래와 같은 응답을 받을 수 있다.

```json
{
  "jsonrpc": "2.0",  // JSON-RPC의 버전
  "id": 67,  // 요청 시에 전달받았던 JSON RPC 식별자
  "result": "Geth/MyEthNode01/v1.7.0-unstable-bf1e2631/darwin-amd64/go1.8.3"  // JSON-RPC의 메서드가 반환하는 값
}
```

이번에는 파라미터가 있는 메서드를 호출해보자.

### eth_getBalance

특정 계정의 이더 잔액을 확인할 수 있다.

```
homo.efficio ~/study/ethereum
🍺  curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getBalance","params":["0x4f27ca4553225cfdddb96fdb89346e7565e61627", "latest"],"id":1}' http://localhost:8545
```

`params` 항목은 잔액을 확인할 계정 공개 주소와 `"latest"`라는 문자열로 구성되는 배열이다. 이 `"latest"`는 default block parameter라고 하며, 정보 조회 시 기준이 되는 불록을 지정하는 역할을 담당한다. default block parameter는 블록 번호 또는 다음 값 중의 하나를 선택할 수 있다.

- `"earlist"`: 원조 genesis 블록
- `"latest"`: 가장 최근에 생성된 블록, 대부분의 경우 `"latest"`를 사용
- `"pending"`: 보류 상태의 블록 또는 트랜젹션

응답은 다음과 같다.

```json
{
  "jsonrpc":"2.0",
  "id":1,
  "result":"0x8f1d5c1cae969e100"  // 잔액
}
```

## 송금

`4f27ca4553225cfdddb96fdb89346e7565e61627` 주소에서 `8128ff5a55370dc9555e5c18b716050bdf2c8440` 주소로 30000 이더를 보내보자.

먼저 받는 쪽의 잔액도 미리 확인해두자.

```
homo.efficio ~/study/ethereum
🍺  curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getBalance","params":["0x8128ff5a55370dc9555e5c18b716050bdf2c8440", "latest"],"id":1}' http://localhost:8545
{"jsonrpc":"2.0","id":1,"result":"0x11e1a300"}
```

금액이 16진수로 표시되는데 http://www.binaryhexconverter.com/hex-to-decimal-converter 같은 온라인 계산기로 10진수로 변환할 수 있다. 변환해보면 값은 `300000000`이며 genesis 파일에 설정한 금액과 같다.

### eth_sendTransaction






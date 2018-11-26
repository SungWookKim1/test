# FastCampus HLF 2기 
## 사전 필요 사항 ##
* nodeJS v8.11.3 이상
* Docker 17.12.1-ce 이상
* HLF 1.2 Image

## NodeJS 설치
https://nodejs.org/ko/download/package-manager

```
curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -

sudo apt-get install -y nodejs

npm i npm@latest -g
```

## NVM 설치
```
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash

source ~/.bashrc
```

## Hyperledger Network Basic 개발 환경 구성
marbles/config/connection_profile.local.json 에서 인증서 관련 경로 설정이 하기와 같이 되어 있다.

```
	"organizations": {
		"Org1MSP": {
			"mspid": "Org1MSP",
			"peers": [
				"fabric-peer-org1"
			],
			"certificateAuthorities": [
				"fabric-ca"
			],
			"x-adminCert": {
				"path": "/$HOME/github/hyperledger-study01/network/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/admincerts/Admin@org1.example.com-cert.pem"
			},
			"x-adminKeyStore": {
				"path": "/$HOME/github/hyperledger-study01/network/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/keystore/"
			}
		}
	}
```

그러므도 git clone을 받을때 홈디렉토리에서 github 디렉토리를 만들고 clone 받기를 권장한다.
(단, 별도 디렉토리에 받을 경우 marbles/config/connection_profile.local.json 에서 인증서 경로 설정을 바꾸어주어야 한다.)

```
$)cd ~ #홈디렉토리로 이동
$)mkdir github
$)git clone https://github.com/simon0210/hyperledger-study01.git
```

클론받은 디렉토리로 이동후 npm install 명령으로 marbles 필요 팩키지를 설치한다. 필요 팩키지는 package.json 에 기술되어 있다.

```
cd hyperledger-study01/marbles
npm install
```
marbles 예제를 돌리기 위한 모듈을 설치 하였다면 Hyperledger Fabric Network 를 기동 한다.

```
$)cd ~/github/hyperledger-study01/network
$)./start.sh
```
HLF 도커 컨테이너가 정상적으로 올라왔는지 확인한다.

```
$)docker ps -a

CONTAINER ID        IMAGE                        COMMAND                  CREATED             STATUS              PORTS                                            NAMES
e1a08c77b210        hyperledger/fabric-peer      "peer node start"        16 seconds ago      Up 15 seconds       0.0.0.0:7051->7051/tcp, 0.0.0.0:7053->7053/tcp   peer0.org1.example.com
39c3a88ea19d        hyperledger/fabric-orderer   "orderer"                19 seconds ago      Up 15 seconds       0.0.0.0:7050->7050/tcp                           orderer.example.com
23394ca430ff        hyperledger/fabric-ca        "sh -c 'fabric-ca-se…"   19 seconds ago      Up 17 seconds       0.0.0.0:7054->7054/tcp                           ca.example.com
e3082169e0cc        hyperledger/fabric-couchdb   "tini -- /docker-ent…"   19 seconds ago      Up 17 seconds       4369/tcp, 9100/tcp, 0.0.0.0:5984->5984/tcp       couchdb

```
정상적인 작동 확인을 위하여 ORDERER 로그를 확인한다
```
$)docker logs -f orderer.example.com

... 중략
2018-09-18 00:22:29.513 UTC [orderer/common/server] Start -> INFO 006 Beginning to serve requests
2018-09-18 00:22:40.497 UTC [msp] DeserializeIdentity -> INFO 007 Obtaining identity
2018-09-18 00:22:40.497 UTC [msp] DeserializeIdentity -> INFO 008 Obtaining identity
2018-09-18 00:22:40.500 UTC [msp] DeserializeIdentity -> INFO 009 Obtaining identity
2018-09-18 00:22:40.503 UTC [fsblkstorage] newBlockfileMgr -> INFO 00a Getting block information from block storage
2018-09-18 00:22:40.514 UTC [orderer/commmon/multichannel] newChain -> INFO 00b Created and starting new chain mychannel
2018-09-18 00:22:40.704 UTC [msp] DeserializeIdentity -> INFO 00c Obtaining identity
2018-09-18 00:22:46.959 UTC [msp] DeserializeIdentity -> INFO 00d Obtaining identity
```
이제 marbles 예제를 시작할 준비가 되었다.
기존에 남아 있지 모를 키를 제거하고 체인코드를 설치해 보자.
```
$)rm -rf ~/github/hyperledger-study01/network/ca/hfc-key-store/*
$)cd ~/github/hyperledger-study01/marbles/scripts
$)node install_chaincode.js
$)node instantiate_chaincode.js
```
자 이제 marbles 예제를 실행 시켜 보자

```
$)gulp marbles_local
```

참고로 marbles_local의 의미는 gulpfiles.js 에서 다음과 같이 확인할수 있다.

```
gulp.task('marbles_local', ['env_local', 'watch-sass', 'watch-server', 'server']);	//run with command `gulp marbles_local` for a local network

// Local Fabric via Docker Compose
gulp.task('env_local', function () {
	env['creds_filename'] = 'marbles_local.json';
});
```
http://localhost:3001 에서 hlf basic network 연동된 marbles 예제를 확인 할 수 있다.

## Hyperledger Network Multi Orderer 개발 환경 구성
marbles 예제를 HLF 멀티 네트워크(org2, ca2, peer4, orderer3, zookeeper3, kafka4) 환경과 연동해 보자.

* 기존 basic-network 제거
```
$)docker rm -f $(docker ps -aq)
```
* 멀티 네트워크 시작
```
$)cd ~/github/hyperledger-study01/network/multi
$)./0-network-start.sh
```
* 채널 생성 및 조인
```
$)./1-create-channel.sh
```

* marbles 체인코드 install & instantiate
```
$)./2-setup-chaincode.sh
```
* ~/gihub/hyperledger-study01/marbles/config/connection_profile_multi.json 에 멀티 환경이 구성되어 있다
현재 org1 만 추가가 되어 있는데 코드를 분석하고 org2도 작동되게 바꿀 필요가 있다.
* marbles 실행 (기존에 실행 되었던 marbles_local은 ctrl+c로 셧다운 하자)
```
$)rm -rf ~/github/hyperledger-study01/network/ca/hfc-key-store/*
$)cd ~/github/hyperledger-study01/marbles
$)gulp marbles_multi
```

cd ~ #홈디렉토리로 이동

mkdir github

cd github

git clone https://github.com

cd hyperledger-study01/marbles

npm install


cd ~/github/hyperledger-study01/network
./start.sh


docker ps -a
docker logs -f orderer.example.com


rm -rf ~/github/hyperledger-study01/network/ca/hfc-key-store/*

cd ~/github/hyperledger-study01/marbles/scripts

node install_chaincode.js

node instantiate_chaincode.js

gulp marbles_local

http://localhost:3001 

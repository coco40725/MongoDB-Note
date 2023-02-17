## MongoDB on Docker
### 1. 不使用 Replica Set
```yml
version: '3'
services:
    mongodb:
        image: mongo
        container_name: m_mongo
        restart: always
        ports:
            - "27017:27017"
        volumes:
            - "./dbdata:/var/lib/mongodb"
        command: mongod --auth
        environment:
            - MONGO_INITDB_DATABASE=mydb
            - MONGO_INITDB_ROOT_USERNAME=root
            - MONGO_INITDB_ROOT_PASSWORD=aaaa

```
關於更改密碼的部分，我們可以關閉container，並重新設定 ``yml`` 來更改密碼。 (MySQL 不行)

### 2. 使用 Replica Set 且有 Auth 
#### 1. Replica Set 簡介
Replica set is a term used for defining a **database cluster** of multiple nodes with the master-slave replication and an automated failover configured between them.
<img src="https://miro.medium.com/v2/resize:fit:640/format:webp/0*wB7K2aXtLjMPJY8i.png">
primary 負責**write** 與 **read**，並且會將結果 **replicate**到 secondary，而 secondary 主要負責**read**，沒有權限進行**write**。當 primary 出問題時，secondary 會變成 primary，進而維持資料庫的穩定。

#### 2. 建立 one primary and two secondary 的模式
##### 2.1 首先要先建立 keyfile，讓兩個 database 可以互通資料:
```cmd
openssl rand -base64 756 > keyfile
```
這裡要特別注意，如果 keyfile的擁有者或權限不正確，在後面建立 Replica set 時會出現以下錯誤
```cmd
ACCESS [main] error opening file: /mongo/keyfile: bad file
```
這是因為keyfile的權限不是給 monogodb，因此我們需要更改其擁有者:
```cmd
sudo chown 999:999 keyfile 
sudo chmod 600 keyfile
```
(chown user:[group] file : 更改文件的擁有者)
(chmod <mode> <file> : 更改文件的權限，600為擁有者可讀寫，其他人不可讀寫執行)
(999是 container中 mongodb 用戶)

##### 2.2 建立完 key file後，便可使用 yml 來建立 Replica Set
```yml
version: "3.7"
services:
  rs1:
    image: mongo:4.4.9
    container_name: rs1
    restart: always
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: abc123Coco
    command: mongod --auth --keyFile /keyfile --bind_ip_all --replSet rs0
    volumes:
      - ./rs1:/data/db
      - ./keyfile:/keyfile
    ports:
      - "27017:27017"


  rs2:
    image: mongo:4.4.9
    container_name: rs2
    restart: always
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: abc123Coco
    command: mongod --auth --keyFile /keyfile --bind_ip_all --replSet rs0
    volumes:
      - ./rs2:/data/db
      - ./keyfile:/keyfile
    ports:
      - "27018:27017"


  rs3:
    image: mongo:4.4.9
    container_name: rs3
    restart: always
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: abc123Coco
    command: mongod --auth --keyFile /keyfile --bind_ip_all --replSet rs0
    volumes:
      - ./rs3:/data/db
      - ./keyfile:/keyfile
    ports:
      - "27019:27017"
```


##### 2.3 確認運行正常後，再來設定 replica set。
先進入monogdb container，選擇 master mongodb container進入
```docker
sudo docker exec -it mgomaster  bash
```
接著輸入帳密以開啟權限
```docker
mongo -u root -p aaaa
```
驗證正確後，開始設定 replica set
```monodb
rs.initiate(
  {
    _id : 'rs0',
    members: [
      { _id : 0, host : "rs1:27017" },
      { _id : 1, host : "rs2:27017" },
      { _id : 2, host : "rs3:27017" }
    ]
  }
)
```
並使用 ``rs.status()`` 來確認結果
```mongo
rs0:PRIMARY> rs.status()
{
        "set" : "rs0",
        "date" : ISODate("2023-02-05T06:12:01.937Z"),
        "myState" : 1,
        "term" : NumberLong(8),
        "syncSourceHost" : "",
        "syncSourceId" : -1,
        "heartbeatIntervalMillis" : NumberLong(2000),
        "majorityVoteCount" : 2,
        "writeMajorityCount" : 2,
        "votingMembersCount" : 2,
        "writableVotingMembersCount" : 2,
        "optimes" : {
                "lastCommittedOpTime" : {
                        "ts" : Timestamp(1675577514, 1),
                        "t" : NumberLong(8)
                },
                "lastCommittedWallTime" : ISODate("2023-02-05T06:11:54.186Z"),
                "readConcernMajorityOpTime" : {
                        "ts" : Timestamp(1675577514, 1),
                        "t" : NumberLong(8)
                },
                "readConcernMajorityWallTime" : ISODate("2023-02-05T06:11:54.186Z"),
                "appliedOpTime" : {
                        "ts" : Timestamp(1675577514, 1),
                        "t" : NumberLong(8)
                },
                "durableOpTime" : {
                        "ts" : Timestamp(1675577514, 1),
                        "t" : NumberLong(8)
                },
                "lastAppliedWallTime" : ISODate("2023-02-05T06:11:54.186Z"),
                "lastDurableWallTime" : ISODate("2023-02-05T06:11:54.186Z")
        },
        "lastStableRecoveryTimestamp" : Timestamp(1675577514, 1),
        "electionCandidateMetrics" : {
                "lastElectionReason" : "electionTimeout",
                "lastElectionDate" : ISODate("2023-02-04T13:56:51.950Z"),
                "electionTerm" : NumberLong(8),
                "lastCommittedOpTimeAtElection" : {
                        "ts" : Timestamp(0, 0),
                        "t" : NumberLong(-1)
                },
                "lastSeenOpTimeAtElection" : {
                        "ts" : Timestamp(1675518897, 1),
                        "t" : NumberLong(6)
                },
                "numVotesNeeded" : 2,
                "priorityAtElection" : 1,
                "electionTimeoutMillis" : NumberLong(10000),
                "numCatchUpOps" : NumberLong(0),
                "newTermStartDate" : ISODate("2023-02-04T13:56:51.963Z"),
                "wMajorityWriteAvailabilityDate" : ISODate("2023-02-04T13:56:53.141Z")
        },
        "members" : [
                {
                        "_id" : 0,
                        "name" : "mgomaster:27017",
                        "health" : 1,
                        "state" : 1,
                        "stateStr" : "PRIMARY",
                        "uptime" : 58522,
                        "optime" : {
                                "ts" : Timestamp(1675577514, 1),
                                "t" : NumberLong(8)
                        },
                        "optimeDate" : ISODate("2023-02-05T06:11:54Z"),
                        "syncSourceHost" : "",
                        "syncSourceId" : -1,
                        "infoMessage" : "",
                        "electionTime" : Timestamp(1675519011, 1),
                        "electionDate" : ISODate("2023-02-04T13:56:51Z"),
                        "configVersion" : 1,
                        "configTerm" : 8,
                        "self" : true,
                        "lastHeartbeatMessage" : ""
                },
                {
                        "_id" : 1,
                        "name" : "mgoslaver:27017",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 58520,
                        "optime" : {
                                "ts" : Timestamp(1675577514, 1),
                                "t" : NumberLong(8)
                        },
                        "optimeDurable" : {
                                "ts" : Timestamp(1675577514, 1),
                                "t" : NumberLong(8)
                        },
                        "optimeDate" : ISODate("2023-02-05T06:11:54Z"),
                        "optimeDurableDate" : ISODate("2023-02-05T06:11:54Z"),
                        "lastHeartbeat" : ISODate("2023-02-05T06:12:00.190Z"),
                        "lastHeartbeatRecv" : ISODate("2023-02-05T06:12:00.189Z"),
                        "pingMs" : NumberLong(0),
                        "lastHeartbeatMessage" : "",
                        "syncSourceHost" : "mgomaster:27017",
                        "syncSourceId" : 0,
                        "infoMessage" : "",
                        "configVersion" : 1,
                        "configTerm" : 8
                }
        ],
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1675577514, 1),
                "signature" : {
                        "hash" : BinData(0,"+THyCpffM4qWmjzA1QHbs87qhmo="),
                        "keyId" : NumberLong("7193602451616825348")
                }
        },
        "operationTime" : Timestamp(1675577514, 1)
}

```


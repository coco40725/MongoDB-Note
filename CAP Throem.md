## CAP Throem
### 1. 簡介 CAP Throem
<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/c/c6/CAP_Theorem_Venn_Diagram.png/800px-CAP_Theorem_Venn_Diagram.png" width="60%">

- **Consistency**
Every read receives the most recent write or an error.
(使用者不論從哪個 server 去抓取資料，都能抓到最新的資料。)

- **Availability**
Every request receives a (non-error) response, without the guarantee that it contains the most recent write.
(每個 request 都能得到回應，也就是即使部分的 server 掛了，使用者查詢也要可以查東西，但不保證是最新的資料)

- **Partition tolerance**
The system continues to operate despite an arbitrary number of messages being dropped (or delayed) by the network between nodes.
(servers 之間的資料同步可能會因為網路傳輸問題，造成部分區域的server更新異常，因此形成 "分區Partition"，這裡指的是使用者是否允許這樣的情況發生)

### 2. 為何不可能 三者同時達成?
- case 1:  Consistency + Availability，也就是要求資料必須在各個 server中呈現一致，且不能因為其中一個server掛了就不回傳 **最新** 結果給使用者。那麼就是不允許Partition的出現，而導致資料狀況不一致。

- case 2:  Consistency + Partition tolerance，那麼如果掛掉的是擁有最新資料的 server，則因為 Consistency 的關係，使用者不可查詢，會得到錯誤訊息，Availability不成立。

- case 3: Availability + Partition tolerance，允許 Partition 出現且又要求必然回應，換言之，使用找訪問到未更新的server資料，仍會回傳結果給他，可見 Consistency 的要求不成立。

### 3. Node, Server, and Cluster
(https://www.onixnet.com/insights/nodes-vs-clusters)
1. **Node**: 電腦科學領域，所謂的「node」是指資料結構當中，儲存和處理數據的一個基本單位。 這個定義非常廣泛，可能代表不同裝置或設備，包含個人電腦、伺服器、伺服器內不同元件、或是功能相似的數台伺服器。
**A node is a server**
2. **Cluster**: A cluster is a group of servers or nodes. 

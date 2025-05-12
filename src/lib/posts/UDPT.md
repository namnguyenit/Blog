---
title: "Elasticseach"
date: "2025-05-12"
updated: "2025-05-12"
categories:
  - "sveltekit"
  - "markdown"
coverImage: "https://images.viblo.asia/ec9626b4-c6f7-4b80-b0ba-f192f9265eaf.jpg"
coverWidth: 16
coverHeight: 9
excerpt: Check out how heading links work with this starter in this post.
---

# Báo Cáo Chi Tiết: Elasticsearch, Tích Hợp MongoDB và Vận Hành Hệ Thống Tìm Kiếm Phân Tán

**Ngày báo cáo:** 12 tháng 5 năm 2025
**Địa điểm:** Hà Nội, Việt Nam
**Dựa trên thông tin tại:** 17:08 Thứ Hai, ngày 12 tháng 5 năm 2025

---

## Mục Lục (Tổng quan)

1.  Tổng Quan Hệ Thống
2.  Elasticsearch - Nền Tảng Cốt Lõi
3.  Triển Khai và Vận Hành Cluster Elasticsearch (5 Nodes)
4.  Tích Hợp Elasticsearch với MongoDB
5.  Tối Ưu Hóa và Quản Lý Nâng Cao
6.  An Ninh Hệ Thống Phân Tán
7.  Kết Luận

---

## 1. Tổng Quan Hệ Thống

Báo cáo này đi sâu vào việc xây dựng, vận hành và tối ưu hóa một hệ thống tìm kiếm phân tán hiệu năng cao. Kiến trúc cốt lõi bao gồm:
* **MongoDB:** Đóng vai trò là cơ sở dữ liệu chính (Source of Truth - SoT), nơi lưu trữ dữ liệu gốc và xử lý các giao dịch ghi chính.
* **Elasticsearch (Cluster 5 Nodes):** Đóng vai trò là hệ thống tìm kiếm và phân tích thứ cấp, cung cấp khả năng tìm kiếm full-text nâng cao, gợi ý, và phân tích dữ liệu phức tạp với tốc độ cao.
* **Cơ Chế Đồng Bộ Hóa Dữ Liệu:** Thành phần trung gian quan trọng, chịu trách nhiệm chuyển và cập nhật dữ liệu từ MongoDB sang Elasticsearch.
* **Tầng Ứng Dụng (Application Layer):** Nơi chứa logic nghiệp vụ, xử lý yêu cầu từ người dùng, tương tác với cả MongoDB và Elasticsearch.
* **Các thành phần hỗ trợ khác:** Load Balancer, API Gateway, Hệ thống Giám sát & Logging, Giải pháp Sao lưu & Phục hồi.

Mục tiêu là xây dựng một hệ thống có khả năng mở rộng, chịu lỗi cao, đáp ứng được lượng truy cập lớn và đảm bảo tính nhất quán dữ liệu ở mức chấp nhận được (Eventual Consistency).

## 2. Elasticsearch - Nền Tảng Cốt Lõi

### 2.1. Kiến Trúc và Cách Hoạt Động

* **Kiến trúc phân tán:**
    * **Cluster:** Một hoặc nhiều node Elasticsearch làm việc cùng nhau.
    * **Node:** Một instance Elasticsearch. Các vai trò chính:
        * `master`: Quản lý trạng thái cluster (chỉ một node active tại một thời điểm).
        * `data`: Lưu trữ dữ liệu (shards), xử lý CRUD, tìm kiếm, aggregation.
        * `ingest`: Tiền xử lý document trước khi index.
        * `coordinating`: Nhận request từ client, điều phối đến các data node, tổng hợp kết quả. Bất kỳ node nào cũng có thể là coordinating.
    * **Index:** Tập hợp các document có cấu trúc tương tự.
    * **Document:** Đơn vị dữ liệu cơ bản (JSON).
    * **Shard:** Index được chia thành các shard.
        * **Primary Shard:** Bản ghi gốc của một phần dữ liệu index. Số lượng primary shard được quyết định khi tạo index và không thể thay đổi (trừ khi reindex).
        * **Replica Shard:** Bản sao của primary shard, nằm trên node khác.
    * **Replica:** Mục đích: Tăng tính sẵn sàng (HA) và tăng thông lượng đọc (tìm kiếm).
* **Quá trình Lập Chỉ Mục (Indexing):** Document được gửi đến cluster -> Coordinating node xác định primary shard -> Chuyển đến node chứa primary shard -> Node đó phân tích (analysis) văn bản thành tokens -> Xây dựng/cập nhật **Inverted Index** (cấu trúc dữ liệu cốt lõi: term -> list of documents) -> Lưu document (`_source`) -> Đồng bộ (replicate) dữ liệu sang các node chứa replica shard tương ứng.
* **Giao tiếp:** Thông qua **REST API** với JSON qua HTTP(S).

### 2.2. Cơ Chế Tìm Kiếm

1.  Client gửi **Query DSL** (JSON) đến Coordinating Node.
2.  Coordinating Node phân tích, xác định các shard cần truy vấn trên các data node.
3.  Gửi query song song đến các node chứa shard liên quan (primary hoặc replica).
4.  Data node thực thi query cục bộ trên shard của mình, sử dụng Inverted Index để tìm kiếm nhanh chóng, tính điểm **relevancy** (thường dùng thuật toán BM25).
5.  Kết quả cục bộ được gửi về Coordinating Node.
6.  Coordinating Node tổng hợp, sắp xếp, phân trang và trả về cho client.

### 2.3. Ưu Điểm Chính

* Tốc độ tìm kiếm full-text cực nhanh.
* Khả năng mở rộng ngang tốt.
* Khả năng chịu lỗi cao (HA).
* Phân tích (Aggregations) mạnh mẽ.
* Gần thời gian thực (NRT).
* Schema linh hoạt.
* API dễ sử dụng.
* Mã nguồn mở, cộng đồng lớn, hệ sinh thái phong phú (ELK/Elastic Stack).

### 2.4. Nhược Điểm và Thách Thức

* Độ phức tạp trong quản lý, vận hành, tối ưu.
* Yêu cầu tài nguyên phần cứng đáng kể (đặc biệt là RAM).
* Không thay thế RDBMS cho các tác vụ quan hệ phức tạp, ACID mạnh.
* Nguy cơ "Split-brain" (cần cấu hình quorum đúng).
* Learning curve cho Query DSL, mapping, tuning.
* Update/Delete kém hiệu quả hơn Insert.
* Eventual Consistency.

## 3. Triển Khai và Vận Hành Cluster Elasticsearch (5 Nodes)

### 3.1. Giao Tiếp Nội Bộ Cluster

Các node giao tiếp trực tiếp qua mạng (TCP port 9300 mặc định) để: Khám phá lẫn nhau, Đồng bộ trạng thái cluster, Sao chép dữ liệu (replication), Phân phối query, Bầu cử master, Giám sát sức khỏe.

### 3.2. Cấu Trúc Triển Khai Đề Xuất (5 Nodes)

Cân bằng giữa ổn định, hiệu năng và tận dụng tài nguyên:

* **Phương án 1 (Khuyến nghị):**
    * **3 Node:** `master`, `data`, `ingest` (Có thể làm master, lưu dữ liệu, xử lý ingest).
    * **2 Node:** `data`, `ingest` (Chỉ lưu dữ liệu, xử lý ingest).
    * *Ưu điểm:* Tách biệt tương đối vai trò master, quorum ổn định (cần 2/3 master-eligible). Toàn bộ 5 node tham gia lưu trữ và tìm kiếm.
* **Phương án 2 (Đơn giản hơn):**
    * **5 Node:** `master`, `data`, `ingest` (Tất cả đều master-eligible và lưu dữ liệu).
    * *Ưu điểm:* Cấu hình đơn giản, quorum ổn định (cần 3/5).
    * *Nhược điểm:* Node master đang active cũng chịu tải của data node.

### 3.3. Cấu Hình Cơ Bản (`elasticsearch.yml`, JVM Heap)

* **`elasticsearch.yml`:**
    * `cluster.name`: Nhất quán trên tất cả node.
    * `node.name`: Duy nhất cho từng node.
    * `node.roles: [ ... ]`: Định nghĩa vai trò.
    * `network.host`, `http.port`, `transport.port`.
    * `discovery.seed_hosts`: Danh sách IP:port (transport) của các node khác để khám phá.
    * `cluster.initial_master_nodes`: Danh sách tên hoặc IP các node master-eligible ban đầu (chỉ dùng khi khởi tạo cluster lần đầu).
* **JVM Heap (`jvm.options`):**
    * Đặt `-Xms` và `-Xmx` bằng nhau (ví dụ: `-Xms16g -Xmx16g`).
    * Không nên vượt quá 50% RAM vật lý của máy chủ.
    * Không nên vượt quá ngưỡng ~31GB (do tối ưu của JVM với compressed ordinary object pointers).

### 3.4. Chiến Lược Sharding và Replicas Cơ Bản

* **`number_of_shards` (Primary Shards):**
    * Quyết định khi tạo index.
    * Nên cân nhắc dựa trên tổng dung lượng dữ liệu dự kiến, tốc độ tăng trưởng, và số lượng data node.
    * Tránh over-sharding (quá nhiều shard nhỏ) hoặc under-sharding (quá ít shard quá lớn). Một shard thường nên giữ kích thước trong khoảng vài GB đến vài chục GB (ví dụ: 10-50GB).
    * Với 5 data node, bắt đầu với 5 primary shards có thể là hợp lý.
* **`number_of_replicas`:**
    * Số bản sao cho *mỗi* primary shard.
    * Đặt `number_of_replicas: 1` là cấu hình tối thiểu để đảm bảo HA (tổng cộng 2 bản copy dữ liệu). Cluster 5 node có thể chịu mất 1 node mà không mất dữ liệu.
    * Tăng replicas giúp tăng khả năng chịu lỗi và thông lượng đọc, nhưng tốn thêm dung lượng lưu trữ và tăng nhẹ độ trễ khi ghi.

### 3.5. Đồng Bộ Dữ Liệu Nội Bộ Cluster (Primary -> Replicas)

Khi dữ liệu được ghi vào primary shard trên một node, node đó sẽ **chỉ sao chép (đồng bộ)** dữ liệu đó đến các node khác đang giữ **replica shard** tương ứng của chính primary shard đó. Dữ liệu không được broadcast đến tất cả các node khác.

## 4. Tích Hợp Elasticsearch với MongoDB

### 4.1. Mô Hình Kiến Trúc

Sử dụng MongoDB làm Source of Truth và Elasticsearch làm Search Engine thứ cấp.

### 4.2. Các Thành Phần Hệ Thống Hoàn Chỉnh

Ngoài MongoDB và Elasticsearch cluster, cần có:
1.  **Cơ chế đồng bộ hóa dữ liệu (Sync Mechanism).**
2.  **Tầng Ứng Dụng (Application Layer).**
3.  **Load Balancer / API Gateway.**
4.  **Hệ thống Giám sát & Logging.**
5.  **Giải pháp Sao lưu & Phục hồi.**

### 4.3. [CHUYÊN SÂU] Cơ Chế Đồng Bộ Hóa Dữ Liệu (MongoDB -> ES)

Lựa chọn cơ chế phù hợp là rất quan trọng:

* **So sánh chi tiết các phương pháp:**
    * **Logstash (MongoDB Input + ES Output):**
        * *Ưu:* Dễ cài đặt, tích hợp tốt với Elastic Stack. Có thể dùng schedule hoặc Change Streams.
        * *Nhược:* Có thể tốn tài nguyên, độ trễ có thể không tối ưu bằng giải pháp custom. Khả năng tùy chỉnh logic phức tạp hạn chế hơn.
    * **MongoDB Change Streams + Custom Service (Node.js, Python, Java,...):**
        * *Ưu:* Gần thời gian thực nhất, linh hoạt cao trong xử lý logic biến đổi, xử lý lỗi.
        * *Nhược:* Cần tự phát triển và duy trì service, phức tạp hơn trong triển khai và đảm bảo HA cho chính service đồng bộ.
    * **Message Queue (Kafka, RabbitMQ) + Worker:**
        * *Ưu:* Tách rời (decoupling) hoàn toàn MongoDB, ứng dụng và Elasticsearch. Khả năng chịu lỗi, mở rộng tốt cho cả publisher và consumer (worker). Kafka hỗ trợ lưu trữ message tốt.
        * *Nhược:* Thêm một thành phần (Message Queue) cần quản lý, tăng độ phức tạp tổng thể của hệ thống.
* **Xử lý Initial Load/Backfill:** Cần có cơ chế riêng để đọc toàn bộ dữ liệu từ MongoDB và index vào Elasticsearch lần đầu hoặc khi cần re-index (ví dụ: dùng script đọc batch, hoặc Logstash chạy một lần).
* **Xử lý lỗi, Đảm bảo dữ liệu:**
    * Implement cơ chế retry khi ghi vào ES thất bại.
    * Sử dụng Dead Letter Queue (DLQ) để chứa các message/event lỗi không thể xử lý.
    * Đảm bảo "at-least-once delivery" (ít nhất một lần) là phổ biến. "Exactly-once" rất khó và thường không cần thiết trong ngữ cảnh này.
* **Data Transformation:** Logic để làm sạch, làm phẳng (flatten) cấu trúc JSON phức tạp từ MongoDB, chọn lọc trường cần index, làm giàu dữ liệu trước khi đẩy vào ES.
* **Giám sát quá trình đồng bộ:** Theo dõi độ trễ (lag), số lượng document được đồng bộ, tỷ lệ lỗi.

### 4.4. Phân Tích Tính Phân Tán

* **MongoDB:** Phân tán qua **Replica Set** (HA) và/hoặc **Sharding** (HA + Scale-out).
* **Elasticsearch:** Phân tán qua **Cluster**, **Shards**, **Replicas**.
* **Tầng Ứng Dụng:** Phân tán qua nhiều **instance** sau Load Balancer.
* **Cơ Chế Đồng Bộ:** Có thể phân tán (nhiều instance Logstash, nhiều worker/consumer Kafka).

### 4.5. Luồng Dữ Liệu và Tính Nhất Quán

* **Luồng Ghi:** App -> MongoDB -> (Sync Mechanism) -> Elasticsearch.
* **Luồng Tìm Kiếm:** App -> Elasticsearch.
* **Luồng Đọc Chi Tiết (Có thể):** App -> Elasticsearch (lấy IDs) -> MongoDB (lấy full data).
* **Tính nhất quán:** **Eventual Consistency**. Dữ liệu trong ES sẽ có độ trễ so với MongoDB. Ứng dụng cần được thiết kế để xử lý độ trễ này.

## 5. Tối Ưu Hóa và Quản Lý Nâng Cao

### 5.1. [CHUYÊN SÂU] Tối ưu hóa Hiệu năng Elasticsearch

* **JVM Tuning:**
    * `-Xms` và `-Xmx` bằng nhau.
    * Chọn Garbage Collector phù hợp (G1GC thường là lựa chọn tốt cho ES hiện đại). Theo dõi thời gian GC pause.
* **Index Design:**
    * **Mapping:** Định nghĩa explicit mapping thay vì dùng dynamic mapping cho production. Chọn đúng kiểu dữ liệu (`keyword` cho exact match/aggregation, `text` cho full-text search).
    * **Sharding Strategy:** Quyết định số primary shard hợp lý dựa trên dữ liệu và tải. Tránh quá nhiều hoặc quá ít shard.
    * **Shard Size:** Giữ kích thước shard trong khoảng tối ưu (vd: 10-50GB).
* **Query Optimization:**
    * Sử dụng `filter context` thay vì `query context` khi không cần tính điểm relevancy (tăng tốc độ, tận dụng cache).
    * Tránh query nặng như wildcard ở đầu (`*term`), script query phức tạp nếu có thể.
    * Sử dụng `_source` filtering để chỉ lấy về các trường cần thiết.
    * Phân tích query chậm bằng `profile` API.
* **Hardware/Network:** SSD/NVMe cho I/O nhanh, đủ RAM cho JVM Heap và OS cache, mạng độ trễ thấp giữa các node.
* **Caching:** Tận dụng File System Cache của HĐH, cân nhắc bật Query Cache (nếu dữ liệu ít thay đổi và query lặp lại), Shard Request Cache (cho aggregation không đổi).

### 5.2. [CHUYÊN SÂU] Thiết kế Mapping và Text Analysis

* **Advanced Data Types:**
    * `nested`: Cho mảng các object cần được truy vấn độc lập.
    * `join`: Cho quan hệ parent-child (ít dùng hơn `nested`).
    * `dense_vector`: Cho tìm kiếm vector (semantic search).
* **Custom Analyzers:** Rất quan trọng cho tiếng Việt hoặc các yêu cầu đặc thù.
    * Xây dựng analyzer với `tokenizer` (vd: `icu_tokenizer`, `standard`), `token filter` (vd: `lowercase`, `stop`, `synonym`, `icu_folding`, `edge_ngram`), `char filter` (vd: `html_strip`).
* **Dynamic vs. Explicit Mapping:** Luôn dùng Explicit Mapping trong production để kiểm soát kiểu dữ liệu và tối ưu index. `dynamic: false` hoặc `dynamic: strict`.
* **Mapping for Relevance and Performance:** Kiểu dữ liệu và analyzer ảnh hưởng trực tiếp đến cách dữ liệu được index và tìm kiếm. Ví dụ: dùng `keyword` cho ID, status, tags; dùng `text` với analyzer phù hợp cho content, description.
* **Index Templates, Aliases:** Sử dụng template để tự động áp dụng mapping/settings cho index mới. Dùng alias để trỏ đến index thực tế, giúp reindex hoặc quản lý time-series data dễ dàng.

### 5.3. [CHUYÊN SÂU] Quản Lý Cluster Nâng Cao

* **Upgrades:** Thực hiện Rolling Upgrades để nâng cấp cluster mà không cần downtime (yêu cầu tương thích phiên bản và cấu hình đúng).
* **Monitoring & Alerting:**
    * Sử dụng Kibana Stack Monitoring hoặc bộ Prometheus/Grafana.
    * Theo dõi chặt chẽ: Cluster Status (green/yellow/red), Unassigned Shards, JVM Heap Usage (%), GC activity, CPU Usage, Disk I/O, Network, Indexing Rate, Search Latency, Pending Tasks, Queue/Thread Pool Rejections, Circuit Breaker trips.
    * Thiết lập cảnh báo cho các ngưỡng quan trọng.
* **Troubleshooting:**
    * **Yellow Status:** Thường do replica shard chưa được phân bổ (vd: thiếu node).
    * **Red Status:** Primary shard bị mất/không thể phân bổ (nghiêm trọng).
    * **Unassigned Shards:** Dùng `Cluster Allocation Explain API` để chẩn đoán.
    * **High CPU/Memory:** Xác định query/indexing nặng, tối ưu hoặc scale cluster.
* **Snapshot/Restore Strategies:** Lên lịch snapshot tự động, lưu trữ snapshot ở nơi an toàn (S3, HDFS, shared FS), kiểm tra khả năng phục hồi định kỳ.
* **Automation:** Sử dụng công cụ quản lý cấu hình (Ansible, Chef, Puppet) hoặc Infrastructure as Code (Terraform) để quản lý cluster.

### 5.4. [CHUYÊN SÂU] Tối ưu hóa MongoDB cho Tích hợp

* **Schema Design:** Thiết kế schema trong MongoDB sao cho dễ dàng hơn cho việc đồng bộ (ví dụ: tránh lồng ghép quá sâu nếu không cần thiết, sử dụng timestamp cập nhật rõ ràng).
* **MongoDB Indexing Strategy:**
    * Tạo index trong MongoDB cho các trường dùng để truy vấn lấy dữ liệu chi tiết (ví dụ: `_id`) sau khi có kết quả từ ES.
    * Tạo index cho các trường dùng trong query của cơ chế đồng bộ (ví dụ: trường timestamp để batch sync).
* **Read Preference:** Sử dụng `secondaryPreferred` hoặc `nearest` có thể giảm tải cho PRIMARY khi tầng ứng dụng đọc chi tiết hoặc cơ chế đồng bộ cần đọc dữ liệu.
* **Change Streams Optimization:** Hiệu năng Change Streams phụ thuộc vào tải ghi trên cluster và tài nguyên của các node. Theo dõi `oplog` size.

### 5.5. Xử Lý Lượng Request Cao (Review)

Hệ thống xử lý tải cao nhờ:
* **Load Balancer:** Phân phối request đến nhiều instance ứng dụng.
* **Application Scaling:** Chạy nhiều instance ứng dụng song song.
* **Elasticsearch Parallelism:** Coordinating node phân phối query đến nhiều data node thực thi song song trên các shard.
* **MongoDB Scaling:** Replica Set phân tán tải đọc, Sharding phân tán tải đọc/ghi.

## 6. An Ninh Hệ Thống Phân Tán

Bảo mật là yếu tố then chốt cho hệ thống production.

### 6.1. [CHUYÊN SÂU] Bảo mật Elasticsearch (Elastic Security)

* **Mã hóa:**
    * **TLS/SSL cho Transport Layer (Node-to-node):** Đảm bảo giao tiếp nội bộ cluster được mã hóa và xác thực.
    * **TLS/SSL cho HTTP Layer:** Đảm bảo giao tiếp giữa client và cluster được mã hóa.
* **Xác thực (Authentication):**
    * Sử dụng các realm: `native` (user/pass lưu trong ES), `file`, `LDAP`, `Active Directory`, `PKI`, `SAML`, `Kerberos`, `OpenID Connect`.
* **Phân quyền (Authorization):**
    * **Role-Based Access Control (RBAC):** Định nghĩa roles với các quyền (privileges) trên cluster, index, hoặc document. Gán user vào các role.
    * **Document Level Security (DLS):** Giới hạn user chỉ thấy các document mà họ được phép (dựa trên query).
    * **Field Level Security (FLS):** Giới hạn user chỉ thấy các trường cụ thể trong document.
* **Audit Logging:** Ghi lại các hành động bảo mật quan trọng.

### 6.2. [CHUYÊN SÂU] Bảo mật MongoDB

* **Xác thực:** Bật chế độ xác thực (SCRAM, x.509, LDAP, Kerberos). Không chạy production mà không bật xác thực.
* **Phân quyền:** Sử dụng RBAC tích hợp của MongoDB để cấp quyền chi tiết cho user/application.
* **Mã hóa:**
    * **Encryption in Transit (TLS/SSL):** Mã hóa kết nối giữa client và server, giữa các member trong replica set/sharded cluster.
    * **Encryption at Rest:** Mã hóa dữ liệu trên disk (sử dụng các giải pháp của HĐH hoặc tính năng của MongoDB Enterprise).
* **Network Hardening:** Giới hạn truy cập mạng chỉ từ các địa chỉ IP/subnet tin cậy (firewall). Chạy MongoDB trên cổng không chuẩn.

### 6.3. Bảo mật Tầng Ứng Dụng & API Gateway

* Áp dụng các biện pháp chống tấn công phổ biến (OWASP Top 10).
* Xác thực và phân quyền cho API.
* Quản lý secret (API keys, database credentials) an toàn (ví dụ: HashiCorp Vault, AWS Secrets Manager).

### 6.4. Network Security

* Sử dụng Firewall để kiểm soát traffic vào/ra các server/cluster.
* Cân nhắc Network Segmentation (chia mạng thành các vùng bảo mật khác nhau).

## 7. Kết Luận

Xây dựng và vận hành một hệ thống tìm kiếm phân tán hiệu năng cao dựa trên Elasticsearch và MongoDB là một công việc phức tạp nhưng mang lại hiệu quả lớn. Thành công đòi hỏi sự hiểu biết sâu sắc về cả hai nền tảng, lựa chọn kiến trúc phù hợp (đặc biệt là cơ chế đồng bộ), cấu hình cẩn thận các yếu tố như sharding, replicas, và vai trò node. Việc tối ưu hóa hiệu năng, giám sát chặt chẽ, đảm bảo an ninh và lên kế hoạch sao lưu/phục hồi là tối quan trọng cho hoạt động ổn định và bền vững của hệ thống. Cần luôn cân bằng giữa hiệu năng, tính sẵn sàng, khả năng mở rộng, độ phức tạp quản lý và các yêu cầu về bảo mật.

---
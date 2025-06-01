---
title: "Báo cáo Bài tập lớn: Ứng dụng Tìm kiếm Phân tán với Elasticsearch"
date: "2025-06-02"
updated: "2025-06-02"
categories:
  - "sveltekit"
  - "markdown"
coverImage: "https://ant.ncc.asia/wp-content/uploads/2024/05/1_BmvPfSSm2G8C-khX1rhCGg.jpg"
coverWidth: 20
coverHeight: 10
excerpt: Báo cáo tổng hợp bài tập lơn về hệ thống phân tán
---


# Báo cáo Bài tập lớn: Ứng dụng Tìm kiếm Phân tán với Elasticsearch

| Thông tin                | Chi tiết |
|--------------------------|----------|
| **Sinh viên thực hiện**  | 23010018: Cao Đức Trung (K17_CNTT_1), 22010242: Trần Mai Anh (K16-CNTT_2) |
| **Giảng viên hướng dẫn** | Phạm Kim Thành |
| **Lớp**                  | Ứng dụng phân tán(N05)-CSE702063 |
| **Ngày hoàn thành**      | 02-06-2025 |
| **Link GitHub dự án**    | [Distributed Search System based on Elasticsearch](https://github.com/namnguyenit/Distributed-search-system-based-on-Elasticsearch) |

---

## Lời Mở Đầu

Trong kỷ nguyên số hiện nay, lượng dữ liệu được tạo ra và cần xử lý tăng trưởng với tốc độ chóng mặt. Việc tìm kiếm thông tin một cách nhanh chóng, chính xác và hiệu quả từ các tập dữ liệu khổng lồ đã trở thành một thách thức lớn đối với nhiều tổ chức và ứng dụng. Các hệ thống tìm kiếm truyền thống thường gặp khó khăn trong việc mở rộng, duy trì hiệu năng và đảm bảo tính sẵn sàng khi đối mặt với lượng truy cập và dữ liệu lớn.

Đồ án "Ứng dụng Tìm kiếm Phân tán với Elasticsearch" được thực hiện nhằm giải quyết những thách thức trên bằng cách xây dựng một hệ thống tìm kiếm sản phẩm có khả năng mở rộng, hiệu năng cao và khả năng chịu lỗi tốt. Elasticsearch, một công cụ tìm kiếm và phân tích phân tán mã nguồn mở, được lựa chọn làm nền tảng cốt lõi cho dự án nhờ vào các tính năng mạnh mẽ của nó như tìm kiếm toàn văn (full-text search), khả năng mở rộng theo chiều ngang, và giao diện API RESTful linh hoạt.

Báo cáo này sẽ trình bày chi tiết quá trình phân tích bài toán, thiết kế kiến trúc hệ thống, triển khai các thành phần, và đánh giá hiệu quả của ứng dụng. Đặc biệt, báo cáo sẽ đi sâu vào phân tích các khía cạnh phân tán của hệ thống, bao gồm cách Elasticsearch quản lý dữ liệu, xử lý truy vấn, cũng như đánh giá hệ thống dựa trên các tiêu chí kỹ thuật quan trọng như khả năng chịu lỗi, giao tiếp phân tán, cân bằng tải, và bảo mật. Cuối cùng, báo cáo sẽ đưa ra những nhận xét về các điểm đã đạt được, những hạn chế còn tồn tại và đề xuất các hướng phát triển tiềm năng trong tương lai.

## 1. Phân tích Bài toán và Lựa chọn Giải pháp Elasticsearch

### 1.1. Bài toán Đặt ra

Bài toán cốt lõi của đồ án là xây dựng một ứng dụng cho phép người dùng tìm kiếm thông tin sản phẩm một cách nhanh chóng và chính xác từ một cơ sở dữ liệu lớn. Các yêu cầu chính đối với hệ thống bao gồm:

- **Hiệu năng cao:** Thời gian phản hồi truy vấn tìm kiếm phải nhanh, ngay cả khi lượng dữ liệu và số lượng người dùng đồng thời tăng lên.
- **Khả năng mở rộng (Scalability):** Hệ thống phải có khả năng mở rộng dễ dàng để đáp ứng sự gia tăng về khối lượng dữ liệu và lưu lượng truy cập.
- **Tính sẵn sàng cao (High Availability) và Khả năng chịu lỗi (Fault Tolerance):** Hệ thống cần duy trì hoạt động ổn định, giảm thiểu thời gian chết ngay cả khi một hoặc nhiều thành phần gặp sự cố.
- **Tìm kiếm nâng cao:** Hỗ trợ các tính năng tìm kiếm phức tạp như tìm kiếm toàn văn, gợi ý, lọc kết quả, và xếp hạng relevancy.
- **Quản lý dữ liệu linh hoạt:** Khả năng dễ dàng cập nhật, thêm mới, xóa sản phẩm và đồng bộ hóa dữ liệu giữa nguồn chính (MongoDB) và công cụ tìm kiếm (Elasticsearch).

### 1.2. Tại sao chọn Elasticsearch?

Để giải quyết các yêu cầu trên, Elasticsearch đã được lựa chọn làm công cụ tìm kiếm chính. Elasticsearch là một công cụ tìm kiếm và phân tích phân tán, được xây dựng trên nền tảng Apache Lucene. Những lý do chính để lựa chọn Elasticsearch bao gồm:

- **Khả năng tìm kiếm mạnh mẽ:** Cung cấp tìm kiếm toàn văn tốc độ cao, hỗ trợ nhiều loại truy vấn phức tạp, phân tích văn bản đa ngôn ngữ, gợi ý tìm kiếm, và xếp hạng kết quả theo mức độ liên quan.
- **Kiến trúc phân tán:** Elasticsearch được thiết kế để hoạt động như một hệ thống phân tán. Dữ liệu được chia thành các **shards** (phân mảnh) và có thể có các **replicas** (bản sao). Các shards và replicas này được phân phối trên nhiều **nodes** (nút) trong một **cluster** (cụm). Điều này cho phép:
  - **Mở rộng theo chiều ngang (Horizontal Scaling):** Dễ dàng tăng dung lượng lưu trữ và khả năng xử lý bằng cách thêm nodes vào cluster.
  - **Tính sẵn sàng cao và chịu lỗi:** Nếu một node gặp sự cố, các replicas trên các nodes khác đảm bảo dữ liệu không bị mất và hệ thống vẫn tiếp tục hoạt động.
- **Tốc độ và Hiệu năng:** Nhờ vào việc sử dụng inverted index của Lucene và kiến trúc phân tán, Elasticsearch cho phép truy vấn dữ liệu lớn với thời gian phản hồi rất nhanh.
- **Giao diện RESTful API:** Cung cấp API HTTP dễ sử dụng để tương tác với dữ liệu (CRUD, tìm kiếm, quản trị cluster), cho phép tích hợp dễ dàng với nhiều ngôn ngữ lập trình và ứng dụng khác nhau, bao gồm Node.js được sử dụng trong đồ án này.
- **Schema-Free (Linh hoạt về lược đồ):** Mặc dù có thể định nghĩa mapping (tương tự schema), Elasticsearch cũng có thể tự động nhận diện kiểu dữ liệu, giúp việc lập chỉ mục dữ liệu ban đầu trở nên đơn giản hơn.
- **Cộng đồng lớn và Hệ sinh thái phong phú:** Có một cộng đồng người dùng và nhà phát triển lớn, cùng với nhiều công cụ hỗ trợ như Logstash (thu thập log), Kibana (trực quan hóa dữ liệu - mặc dù đồ án này không sử dụng trực tiếp Kibana để giám sát node), Beats (thu thập dữ liệu).

Với những ưu điểm vượt trội này, Elasticsearch là một lựa chọn phù hợp để xây dựng nền tảng cho ứng dụng tìm kiếm phân tán của đồ án.

## 2. Phân tích Hệ thống

### 2.1. Kiến trúc Tổng thể

Hệ thống được thiết kế theo kiến trúc microservices, bao gồm các thành phần chính giao tiếp với nhau qua mạng.

Sơ đồ kiến trúc tổng quan:

![Sơ đồ kiến trúc](https://cdn.discordapp.com/attachments/1280340705088114792/1378657827438067753/moi.drawio.png?ex=683d66a8&is=683c1528&hm=9a929fccae92791374e1decb51178ac3d25e1048d71a9837d57651feaaa2fc3e&)

- **Client (Người dùng):** Tương tác với hệ thống thông qua giao diện web trên trình duyệt.
- **Nginx:** Đóng vai trò là reverse proxy và load balancer, tiếp nhận các yêu cầu HTTP từ client và phân phối chúng đến các instance của Web Application.
- **Web Application (Node.js):** Là tầng backend xử lý logic nghiệp vụ. Ứng dụng này cung cấp các API cho việc tìm kiếm sản phẩm (tương tác với Elasticsearch) và quản lý dữ liệu sản phẩm (tương tác với MongoDB).
- **Elasticsearch Cluster:** Cụm Elasticsearch bao gồm nhiều node (trong đồ án này là 3 nodes: `es01`, `es02`, `es03`) chịu trách nhiệm lưu trữ, lập chỉ mục (indexing) dữ liệu sản phẩm và thực thi các truy vấn tìm kiếm.
- **MongoDB:** Hệ quản trị cơ sở dữ liệu NoSQL, được sử dụng làm nguồn lưu trữ chính cho thông tin chi tiết của sản phẩm, thông tin người dùng và lịch sử tìm kiếm. Dữ liệu từ MongoDB sẽ được đồng bộ hóa sang Elasticsearch để phục vụ cho việc tìm kiếm.

### 2.2. Các Thành phần Chính

#### 2.2.1. Elasticsearch Cluster

- **Cấu hình:** Cụm Elasticsearch trong đồ án được triển khai với 3 nodes (`es01`, `es02`, `es03`) sử dụng Docker. Các node này được cấu hình để tự khám phá (discovery) lẫn nhau và hình thành một cluster.
  - Trong `docker-compose.yml`, các node được định nghĩa với `discovery.seed_hosts: "es01,es02"` và `cluster.initial_master_nodes: "es01,es02"`.
  - Mỗi node chạy phiên bản Elasticsearch `8.13.2`.
- **Vai trò của Nodes:** Trong một cluster Elasticsearch, các node có thể đảm nhận các vai trò khác nhau (master, data, coordinating). Trong cấu hình mặc định của đồ án, mỗi node có thể đảm nhận nhiều vai trò.
  - **Master-eligible node:** Chịu trách nhiệm quản lý trạng thái của cluster, ví dụ như tạo/xóa index, theo dõi các node trong cluster, và quyết định shard nào được phân bổ ở đâu.
  - **Data node:** Lưu trữ dữ liệu (shards) và thực thi các thao tác liên quan đến dữ liệu như CRUD, tìm kiếm, và tổng hợp.
  - **Coordinating node:** Nhận yêu cầu từ client, chuyển tiếp chúng đến các data node thích hợp, sau đó tổng hợp kết quả và trả về cho client. Bất kỳ node nào cũng có thể đóng vai trò này.
- **Lập chỉ mục (Indexing):** Dữ liệu sản phẩm từ MongoDB được lập chỉ mục vào Elasticsearch. Mỗi sản phẩm trở thành một document trong một index của Elasticsearch.
- **Tìm kiếm (Searching):** Web Application gửi các truy vấn tìm kiếm đến Elasticsearch cluster. Elasticsearch xử lý truy vấn trên các shard liên quan và trả về kết quả.

#### 2.2.2. MongoDB

- **Vai trò:** Là cơ sở dữ liệu chính cho dữ liệu sản phẩm, người dùng, và các thông tin khác không trực tiếp phục vụ cho tìm kiếm tốc độ cao.
- **Cấu hình:** Một instance MongoDB (service `mongo` trong `docker-compose.yml`) được sử dụng.
- **Tương tác:** Web Application sử dụng Mongoose (thư viện tươn tác dành cho NodeJs) để tương tác với MongoDB cho các thao tác CRUD dữ liệu.
- **Đồng bộ dữ liệu:** Cần có cơ chế để đồng bộ dữ liệu từ MongoDB sang Elasticsearch khi có sự thay đổi (thêm, sửa, xóa sản phẩm). Trong đồ án này, việc đồng bộ có thể được thực hiện thông qua các hàm trong `productController.js` và `elasticsearchService.js` khi có thao tác cập nhật sản phẩm.

#### 2.2.3. Web Application (Node.js)

- **Công nghệ:** Xây dựng bằng Node.js và framework Express.js.
- **Chức năng:**
  - Cung cấp RESTful APIs cho client (ví dụ: tìm kiếm sản phẩm, xem chi tiết sản phẩm, quản lý sản phẩm cho admin).
  - Xử lý logic nghiệp vụ của ứng dụng.
  - Kết nối và tương tác với Elasticsearch cho các chức năng tìm kiếm thông qua client `@elastic/elasticsearch`. Client này được cấu hình trong `webapp/services/elasticsearchService.js` để kết nối đến các node Elasticsearch được định nghĩa trong biến môi trường `ELASTICSEARCH_HOSTS`.
  - Kết nối và tương tác với MongoDB để quản lý dữ liệu gốc.
- **Triển khai:** Được đóng gói thành Docker image và chạy dưới dạng service `webapp` trong `docker-compose.yml`.

#### 2.2.4. Nginx (Load Balancer)

- **Vai trò:**
  - Đóng vai trò là reverse proxy, che giấu kiến trúc bên trong và cung cấp một điểm truy cập duy nhất cho client.
  - Là load balancer, phân phối các yêu cầu đến nhiều instance của Web Application (nếu có nhiều instance được triển khai), giúp tăng khả năng chịu tải và sẵn sàng.
  - Có thể cấu hình upstream để cân bằng tải cho cả Web Application hoặc các node Elasticsearch (nếu cần).
- **Cấu hình:**
  - Được định nghĩa là service `nginx` trong `docker-compose.yml` và sử dụng file cấu hình `nginx.conf`.
  - Trong file cấu hình, có thể sử dụng khối `upstream` để liệt kê các backend (Web Application hoặc Elasticsearch nodes).
  - Sử dụng `proxy_pass` để chuyển tiếp request từ client đến backend tương ứng.
  - Thiết lập các header như `X-Real-IP`, `X-Forwarded-For`, `X-Forwarded-Proto` để giữ thông tin gốc của client.
  - Có thể cấu hình các tham số như `client_max_body_size` để hỗ trợ upload file lớn.
  - Sử dụng `proxy_next_upstream` để đảm bảo nếu một backend gặp lỗi, request sẽ được chuyển sang backend khác.
- **Lợi ích:**
  - Tăng tính sẵn sàng và khả năng mở rộng của tầng ứng dụng web.
  - Đảm bảo hệ thống hoạt động ổn định ngay cả khi một số backend gặp sự cố.
  - Dễ dàng mở rộng số lượng backend mà không ảnh hưởng đến client.

## 3. Phân tích Tính Phân tán của Hệ thống

Tính phân tán là một đặc điểm cốt lõi của hệ thống này, chủ yếu được thể hiện qua cách Elasticsearch hoạt động và cách các thành phần tương tác với nhau.

### 3.1. Phân tán Dữ liệu (Data Sharding and Replication)

Elasticsearch đạt được khả năng phân tán dữ liệu thông qua cơ chế sharding và replication:

- **Sharding (Phân mảnh):**
  - Khi một index được tạo trong Elasticsearch, nó được chia thành một hoặc nhiều **primary shards**. Mỗi shard là một instance độc lập của Lucene và có thể được coi như một công cụ tìm kiếm riêng lẻ.
  - Dữ liệu (documents) được phân phối đều vào các primary shards này dựa trên một thuật toán hashing của ID document hoặc một routing key tùy chỉnh.
  - **Lợi ích của sharding:**
    - **Mở rộng dung lượng lưu trữ:** Cho phép lưu trữ lượng dữ liệu lớn hơn khả năng của một node đơn lẻ.
    - **Cải thiện hiệu năng xử lý song song:** Các thao tác trên index (ví dụ: lập chỉ mục, tìm kiếm) có thể được thực hiện song song trên nhiều shards, giúp tăng tốc độ.
  - Số lượng primary shards của một index được xác định lúc tạo index và không thể thay đổi sau đó.

- **Replication (Sao chép):**
  - Mỗi primary shard có thể có một hoặc nhiều **replica shards**. Replica shard là một bản sao đầy đủ của primary shard.
  - **Lợi ích của replication:**
    - **Tính sẵn sàng cao và chịu lỗi:** Nếu một node chứa primary shard gặp sự cố, một replica shard trên node khác có thể được "thăng cấp" (promoted) thành primary shard, đảm bảo dữ liệu không bị mất và hệ thống vẫn hoạt động.
    - **Tăng thông lượng tìm kiếm:** Các truy vấn tìm kiếm có thể được xử lý bởi cả primary shard và replica shards, giúp phân tán tải và tăng khả năng xử lý đồng thời.
  - Replica shards không bao giờ được đặt trên cùng một node với primary shard của nó. Số lượng replica shards có thể thay đổi bất cứ lúc nào.

Trong đồ án này, khi tạo index sản phẩm, cấu hình số lượng primary shards và replica shards sẽ quyết định mức độ phân tán và khả năng chịu lỗi của dữ liệu tìm kiếm.

### 3.2. Phân tán Xử lý Tìm kiếm (Distributed Search Execution)

Khi một truy vấn tìm kiếm được gửi đến một node bất kỳ trong Elasticsearch cluster (node này đóng vai trò coordinating node):

1.  **Phase 1: Query Phase (Phân tán truy vấn):**
    - Coordinating node chuyển tiếp truy vấn đến tất cả các primary shards hoặc replica shards liên quan của index đó. Mỗi shard thực thi truy vấn cục bộ trên dữ liệu của nó và trả về một danh sách các ID document khớp và thông tin cần thiết để xếp hạng.
2.  **Phase 2: Fetch Phase (Tổng hợp kết quả):**
    - Coordinating node thu thập kết quả (ID và điểm số) từ tất cả các shards.
    - Nó sắp xếp các kết quả này để xác định danh sách cuối cùng các document cần trả về cho client (ví dụ: top N kết quả).
    - Sau đó, coordinating node chỉ yêu cầu các shard liên quan trả về nội dung đầy đủ của các document trong danh sách cuối cùng này.
    - Cuối cùng, coordinating node tổng hợp các document này và gửi kết quả cuối cùng cho client.

Quá trình này diễn ra song song trên nhiều shards và nodes, giúp đảm bảo tốc độ truy vấn cao ngay cả khi dữ liệu lớn và phân tán trên nhiều máy.

### 3.3. Phân tán Log và Giám sát (Hiện trạng và Giải pháp)

- **Hiện trạng:**
    - **Logging:** Mỗi thành phần (Elasticsearch, MongoDB, Webapp, Nginx) tạo ra log riêng trên container của nó. Việc truy cập và phân tích log này đòi hỏi phải vào từng container, gây khó khăn cho việc theo dõi và gỡ lỗi hệ thống tổng thể.
    - **Giám sát:** Bạn đề cập đến việc xem sức khỏe và thông tin node qua API của Elasticsearch (`/_cat/health`, `/_cat/nodes`, `/_nodes/stats`). Đây là một cách cơ bản để kiểm tra trạng thái, nhưng nó không cung cấp cái nhìn tổng quan, lịch sử hoạt động, hoặc cảnh báo tự động. Hệ thống giám sát hiện tại chưa chặt chẽ, chưa realtime và chưa đầy đủ.

- **Giải pháp cải thiện (Tập trung vào Elasticsearch Exporter):**
    Để giải quyết điểm yếu về giám sát, một giải pháp hiệu quả là sử dụng **Elasticsearch Exporter** kết hợp với **Prometheus** và **Grafana**:
    1.  **Elasticsearch Exporter:**
        - Là một công cụ mã nguồn mở, được thiết kế để thu thập các metrics (chỉ số hoạt động) từ Elasticsearch cluster thông qua API của nó.
        - Các metrics này bao gồm: trạng thái cluster, sức khỏe của index, số lượng document, hiệu năng tìm kiếm và lập chỉ mục, tài nguyên sử dụng của node (CPU, memory, disk), hoạt động của JVM, v.v.
        - Exporter sẽ cung cấp các metrics này dưới dạng một endpoint HTTP mà Prometheus có thể scrape (thu thập).
    2.  **Prometheus:**
        - Là một hệ thống giám sát và cảnh báo mã nguồn mở.
        - Prometheus sẽ được cấu hình để định kỳ scrape metrics từ endpoint của Elasticsearch Exporter.
        - Nó lưu trữ các metrics này dưới dạng time-series data (dữ liệu chuỗi thời gian), cho phép truy vấn và phân tích lịch sử hoạt động.
        - Prometheus cũng có hệ thống cảnh báo (Alertmanager) mạnh mẽ, có thể cấu hình để gửi thông báo khi có sự cố hoặc khi các chỉ số vượt ngưỡng.
    3.  **Grafana:**
        - Là một nền tảng trực quan hóa dữ liệu mã nguồn mở.
        - Grafana kết nối với Prometheus làm nguồn dữ liệu.
        - Người dùng có thể tạo các dashboards tùy chỉnh hoặc sử dụng các dashboards có sẵn để trực quan hóa các metrics của Elasticsearch một cách realtime, theo dõi xu hướng, và phát hiện sớm các vấn đề.

    **Cách thức thực hiện:**
    - **Thêm Elasticsearch Exporter vào `docker-compose.yml`:**
      ```yaml
      services:
        # ... các services khác ...
        elasticsearch-exporter:
          image: bitnami/elasticsearch-exporter:latest # Hoặc một image exporter khác
          command:
            - '--es.uri=http://es01:9200,http://es02:9200,http://es03:9200' # Trỏ đến các node ES
            # - '--es.all' # Thu thập tất cả metrics
            # - '--es.indices' # Thu thập metrics của từng index
          ports:
            - "9114:9114" # Port mặc định của exporter
          networks:
            - app-network # Cùng network với ES
          depends_on:
            - es01
            - es02
            - es03
      ```
    - **Triển khai Prometheus và Grafana:** Tương tự, thêm services cho Prometheus và Grafana vào `docker-compose.yml`. Cấu hình Prometheus để scrape target là `elasticsearch-exporter:9114`.
    - **Tạo Dashboards trong Grafana:** Import các dashboards có sẵn cho Elasticsearch từ Grafana Labs hoặc tự xây dựng dashboards.

    **Đối với logging:**
    - Xem xét triển khai một giải pháp tập trung log như **ELK Stack (Elasticsearch, Logstash, Kibana)** hoặc **EFK Stack (Elasticsearch, Fluentd, Kibana)**. Fluentd hoặc Logstash sẽ thu thập log từ tất cả các container và gửi chúng đến một instance Elasticsearch riêng (có thể là cùng cluster hoặc một cluster khác dành cho log). Kibana sau đó được sử dụng để tìm kiếm, phân tích và trực quan hóa log.
    - Một giải pháp đơn giản hơn là cấu hình Docker logging driver để gửi log đến một dịch vụ quản lý log tập trung (ví dụ: AWS CloudWatch Logs, Google Cloud Logging, Splunk).

## 4. Đánh giá Hệ thống theo các Tiêu chí Kỹ thuật

Dưới đây là phân tích chi tiết về các tiêu chí kỹ thuật của dự án, những điểm đã đạt được, những điểm còn yếu và phương hướng cải thiện.

### 4.1. Fault Tolerance (Khả năng chịu lỗi)
* **Đánh giá hiện trạng:** Đạt, nhưng chưa tốt
* **Đánh giá hiện trạng:**
    * **Elasticsearch:** Đạt được ở mức độ tốt. Với 3 nodes và cấu hình replica shards (mặc định là 1 replica cho mỗi primary shard nếu cluster có đủ node), hệ thống có thể chịu lỗi khi một node Elasticsearch gặp sự cố. Dữ liệu không bị mất và cluster vẫn hoạt động, các replica shard sẽ được thăng cấp thành primary shard.
    * **Web Application (Node.js):** Hiện tại, `docker-compose.yml` chỉ định nghĩa một instance của `webapp`. Nếu instance này lỗi, toàn bộ ứng dụng sẽ ngừng hoạt động. Đây là một điểm yếu.
    * **MongoDB:** Tương tự webapp, chỉ có một instance `mongo`. Nếu instance này lỗi, các chức năng liên quan đến dữ liệu gốc (ví dụ: đăng nhập, xem chi tiết sản phẩm từ DB) sẽ bị ảnh hưởng.
    * **Nginx:** Hoạt động như một single point of failure nếu chỉ có một instance Nginx. Tuy nhiên, Nginx rất ổn định.
* **Điểm mạnh:** Khả năng chịu lỗi của Elasticsearch cluster là một điểm mạnh rõ ràng.
* **Điểm yếu/Hạn chế:**
    * Tầng Web Application và MongoDB là các điểm lỗi đơn lẻ (Single Points of Failure - SPOF).
    * Chưa có cơ chế tự động phục hồi hoặc chuyển đổi dự phòng rõ ràng cho webapp và MongoDB trong cấu hình hiện tại.
* **Hướng cải thiện/Giải pháp:**
    * **Web Application:**
        * Triển khai nhiều instances của `webapp` service trong `docker-compose.yml` (sử dụng `deploy: replicas: N` trong Docker Swarm mode, hoặc chạy nhiều container và cấu hình Nginx cân bằng tải qua chúng).
        * Nginx đã có sẵn để cân bằng tải, chỉ cần cấu hình để nó biết đến nhiều backend webapp instances.
    * **MongoDB:**
        * Triển khai MongoDB dưới dạng một **Replica Set**. Một replica set bao gồm nhiều MongoDB instances (một primary và nhiều secondaries). Nếu primary lỗi, một secondary sẽ được bầu chọn làm primary mới. Điều này đòi hỏi cấu hình phức tạp hơn trong `docker-compose.yml` và cho các instances MongoDB.
    * **Nginx:** Để tăng khả năng chịu lỗi cho Nginx, có thể sử dụng các giải pháp như Keepalived với Virtual IP (VIP) để có cơ chế failover giữa nhiều Nginx instances, tuy nhiên điều này thường phức tạp hơn cho một đồ án.

### 4.2. Distributed Communication (Giao tiếp phân tán)

* **Đánh giá hiện trạng:** Đạt.
    * Các node Elasticsearch giao tiếp với nhau qua mạng (transport layer, default port 9300) để duy trì trạng thái cluster, sao chép dữ liệu, và phân tán truy vấn.
    * Web Application giao tiếp với Elasticsearch cluster qua HTTP (default port 9200) sử dụng client `@elastic/elasticsearch`.
    * Web Application giao tiếp với MongoDB qua TCP/IP sử dụng driver MongoDB.
    * Client giao tiếp với Nginx qua HTTP. Nginx giao tiếp với Web Application qua HTTP.
    * Hệ thống được thiết kế để hoạt động trên nhiều máy (thông qua Docker và mạng ảo của Docker, có thể mở rộng ra nhiều host vật lý với Docker Swarm hoặc Kubernetes).
* **Điểm mạnh:** Sử dụng các giao thức mạng chuẩn và phổ biến.
* **Điểm yếu/Hạn chế:**
    - Giao tiếp giữa các thành phần (ví dụ: webapp ↔ elasticsearch, webapp ↔ mongodb, nginx ↔ webapp) mặc định có thể chưa được mã hóa (HTTP thay vì HTTPS, kết nối database không có SSL/TLS).
* **Hướng cải thiện/Giải pháp:**
    * **Mã hóa giao tiếp:**
        * Cấu hình SSL/TLS cho Elasticsearch (cả transport layer và HTTP layer).
        * Cấu hình SSL/TLS cho kết nối đến MongoDB.
        * Sử dụng HTTPS giữa client và Nginx, và giữa Nginx và Web Application.
        * Điều này được đề cập chi tiết hơn ở mục **Security Features**.

### 4.3. Sharding hoặc Replication (Phân mảnh hoặc Sao chép dữ liệu)

* **Đánh giá hiện trạng:** Đạt.
    * **Elasticsearch:**
        * **Sharding:** Mặc định, khi tạo index, Elasticsearch sẽ chia index thành các primary shards (ví dụ, 1 primary shard theo mặc định của ES 8.x). Số lượng primary shards có thể được cấu hình khi tạo index.
        * **Replication:** Mặc định, Elasticsearch sẽ cố gắng tạo 1 replica cho mỗi primary shard nếu có đủ số lượng data nodes trong cluster (ít nhất 2 node để replica không nằm cùng node với primary).
        * Hệ thống sử dụng cả sharding và replication trong Elasticsearch.
        * Việc phân phối các primary shard và replica shard hiện tại là hoàn toàn tự động bởi Elasticsearch.
    * **MongoDB:** Trong cấu hình hiện tại (một instance đơn), MongoDB không thực hiện replication.
* **Điểm mạnh:** Tận dụng tốt cơ chế sharding và replication của Elasticsearch để tăng khả năng mở rộng và chịu lỗi cho dữ liệu tìm kiếm.
* **Điểm yếu/Hạn chế:**
    * MongoDB chưa có replication, là điểm yếu về tính sẵn sàng của dữ liệu gốc.
* **Hướng cải thiện/Giải pháp:**
    * **MongoDB:** Triển khai MongoDB Replica Set như đã đề cập ở mục **Fault Tolerance**.
    * **Elasticsearch:** Có thể điều chỉnh lại các index, shard cho phù hợp với thực tế thông qua các API sau:

      **Ví dụ tạo index với số lượng shard/replica tùy chỉnh:**
      ```http
      PUT /my-index
      {
        "settings": {
          "number_of_shards": 5,
          "number_of_replicas": 1
        }
      }
      ```
      
      **Các API liên quan:**
      - Tạo index mới với số shard/replica: `PUT /<index-name>` với phần `settings` như trên.
      - Thay đổi số replica cho index đã tồn tại:
        ```http
        PUT /my-index/_settings
        {
          "number_of_replicas": 2
        }
        ```

### 4.4. Simple Monitoring / Logging (Giám sát và Ghi log đơn giản)
* **Đánh giá hiện trạng:** Đạt, nhưng chưa đầy đủ.
* **Đánh giá hiện trạng:** Đạt ở mức cơ bản, nhưng còn yếu như đã phân tích.
    * **Logging:** Log được ghi riêng lẻ trong từng container.
    * **Monitoring:** Sử dụng API của Elasticsearch (`/_cat/*`, `/_nodes/stats`) để kiểm tra thủ công.
* **Điểm yếu/Hạn chế:**
    * Thiếu hệ thống giám sát tập trung, realtime và có cảnh báo.
    * Khó khăn trong việc tổng hợp và phân tích log từ nhiều service.
    * Không có khả năng theo dõi lịch sử hoạt động một cách dễ dàng.
* **Hướng cải thiện/Giải pháp:**
    * **Monitoring:**
        * **Triển khai Elasticsearch Exporter + Prometheus + Grafana** như đã trình bày chi tiết ở mục **3.3. Phân tán Log và Giám sát**. Đây là giải pháp được bạn đề xuất và rất phù hợp.
        * **Ví dụ thực tế:** Nhiều công ty lớn sử dụng stack này để giám sát các hệ thống phân tán của họ, bao gồm cả Elasticsearch. Grafana Labs cung cấp nhiều dashboard mẫu cho Elasticsearch có thể import và sử dụng ngay.
    * **Logging:**
        * **Tập trung hóa log:**
            * **ELK/EFK Stack:** Gửi log từ Docker containers (sử dụng Fluentd hoặc Filebeat làm log forwarder) đến một cluster Elasticsearch (có thể là một cluster riêng biệt hoặc cùng cluster nếu tài nguyên cho phép, nhưng khuyến nghị cluster riêng cho log) và sử dụng Kibana để trực quan hóa.
            * **Docker Logging Drivers:** Cấu hình Docker để gửi log đến các dịch vụ như AWS CloudWatch, Google Cloud Logging, Splunk, hoặc một syslog server.


### 4.5. Basic Stress Test (Kiểm tra tải cơ bản)
* **Đánh giá hiện trạng:** Đạt.

* **Đánh giá hiện trạng:**
    * Hệ thống cung cấp chức năng trong trang quản trị (admin) cho phép thực hiện kiểm thử với nhiều người dùng và nhiều truy vấn. Điều này cho phép quản trị viên có thể chủ động tạo ra một lượng tải nhất định lên hệ thống để quan sát hành vi.
    * Chức năng này là một bước khởi đầu tốt để đánh giá sơ bộ khả năng chịu tải của ứng dụng.
* **Điểm mạnh:**
    * Có sẵn công cụ tích hợp trong hệ thống để thực hiện kiểm thử tải cơ bản mà không cần cài đặt thêm công cụ bên ngoài cho các kịch bản đơn giản.
    * Giúp quản trị viên dễ dàng thực hiện các bài test nhanh để kiểm tra sau khi có thay đổi hoặc cập nhật.
* **Điểm yếu/Hạn chế:**
    * **Phạm vi kiểm thử:** Chức năng tích hợp sẵn có thể không cung cấp đầy đủ các tùy chọn cấu hình kịch bản tải phức tạp như các công cụ chuyên dụng (ví dụ: mô phỏng chính xác số lượng người dùng đồng thời lớn, các kiểu tải khác nhau như spike test, soak test, ramp-up/ramp-down chi tiết).
    * **Thu thập và phân tích metrics:** Khả năng thu thập và phân tích chi tiết các chỉ số hiệu năng (thời gian phản hồi chi tiết từng request, tỷ lệ lỗi theo thời gian, tài nguyên sử dụng của từng thành phần dưới tải) có thể hạn chế so với các công cụ chuyên dụng.
    * **Tính tự động hóa và lặp lại:** Việc thực hiện các bài test lớn, lặp đi lặp lại và tích hợp vào quy trình CI/CD có thể khó khăn nếu chỉ dựa vào chức năng thủ công trong trang admin.
    * **Thiếu kết quả kiểm thử định lượng:** Báo cáo cần bổ sung kết quả cụ thể từ các bài kiểm thử đã thực hiện bằng chức năng này (ví dụ: hệ thống xử lý được bao nhiêu truy vấn/giây với X người dùng mô phỏng, thời gian phản hồi trung bình là bao nhiêu, có lỗi xảy ra không).
* **Hướng cải thiện/Giải pháp:**
    * **Tận dụng tối đa chức năng sẵn có:**
        * **Xác định kịch bản:** Sử dụng chức năng admin để mô phỏng các kịch bản sử dụng phổ biến: nhiều người dùng tìm kiếm đồng thời các từ khóa khác nhau, tìm kiếm các từ khóa phức tạp, tìm kiếm khi hệ thống đang có hoạt động ghi dữ liệu (nếu có thể mô phỏng).
        * **Quan sát hệ thống:** Trong quá trình kiểm thử bằng chức năng admin, cần kết hợp với các công cụ giám sát đã đề xuất (Prometheus, Grafana, hoặc ít nhất là `docker stats` và API `/_nodes/stats` của Elasticsearch) để theo dõi:
            * Thời gian phản hồi của các truy vấn.
            * Tỷ lệ lỗi (nếu có).
            * Mức sử dụng CPU, bộ nhớ, I/O của các service (webapp, elasticsearch, nginx, mongo).
            * Trạng thái của Elasticsearch cluster (số lượng active shards, pending tasks)..


### 4.6. System Recovery (Rejoin after Failure - Khả năng phục hồi hệ thống)
* **Đánh giá hiện trạng:** Đạt.

* **Đánh giá hiện trạng:**
    * **Elasticsearch:** Đạt. Các node Elasticsearch được thiết kế để tự động tham gia lại cluster sau khi khởi động lại hoặc mất kết nối tạm thời. Master node sẽ quản lý việc này, phân bổ lại shard nếu cần.
    * **Web Application / MongoDB / Nginx:** Trong `docker-compose.yml`, các services này có thể được cấu hình với `restart: always` hoặc `restart: unless-stopped`. Điều này đảm bảo Docker sẽ tự động khởi động lại container nếu nó bị dừng hoặc lỗi.
* **Điểm mạnh:** Khả năng tự phục hồi của Elasticsearch cluster và cơ chế restart của Docker.
* **Điểm yếu/Hạn chế:**
    * Việc restart một service đơn lẻ (webapp, mongo) có thể gây gián đoạn dịch vụ ngắn. Nếu không có replica, không có failover tức thời.
* **Hướng cải thiện/Giải pháp:**
    * **Đảm bảo `restart: always`:** Kiểm tra và đảm bảo tất cả các services quan trọng trong `docker-compose.yml` đều có chính sách restart phù hợp.
    * **Triển khai Replicas:** Như đã đề cập ở **Fault Tolerance**, việc có nhiều replicas cho webapp và MongoDB (replica set) sẽ giúp hệ thống phục hồi nhanh hơn và mượt mà hơn sau lỗi, vì traffic có thể được chuyển sang các instance còn hoạt động trong khi instance lỗi khởi động lại.
    * **Kiểm tra thực tế:** Mô phỏng lỗi (ví dụ: `docker stop <container_id>`) và quan sát quá trình hệ thống tự phục hồi, node tham gia lại cluster.

### 4.7. Load Balancing (Cân bằng tải)
* **Đánh giá hiện trạng:** Đạt.

* **Hiện trạng:**
    * Nginx được sử dụng để cân bằng tải các request đến nhiều instance của webapp (nếu triển khai nhiều instance).
    * Nginx cũng có thể cân bằng tải các request đến nhiều node Elasticsearch thông qua upstream block.
* **Điểm mạnh:**
    * Đã có Nginx làm load balancer phía server cho cả webapp và Elasticsearch.
* **Hạn chế:**
    * Nếu chỉ có một instance webapp, Nginx chỉ đóng vai trò reverse proxy.
    * Cần cấu hình Nginx với upstream block để tận dụng cân bằng tải.
    * Client Elasticsearch không tự động phân phối request, việc cân bằng tải phụ thuộc vào Nginx.
* **Giải pháp:**
    * Triển khai nhiều instance webapp để Nginx thực sự cân bằng tải.
    * Sử dụng upstream block trong nginx.conf để định nghĩa các backend webapp hoặc Elasticsearch nodes.
    * Có thể bổ sung health check cho backend trong Nginx.

### 4.8. Consistency Guarantees (Đảm bảo tính nhất quán)
* **Đánh giá hiện trạng:** Chưa đạt, còn yếu.
#### Ưu điểm

- **Nhất quán cuối cùng (Eventual Consistency):**
    - Elasticsearch đảm bảo dữ liệu sẽ được đồng bộ giữa các node (primary và replica) sau một khoảng thời gian ngắn, giúp hệ thống phục hồi dữ liệu khi có sự cố node.
- **Tính sẵn sàng cao:**
    - Nhờ có replica shard, nếu một node hoặc một shard bị lỗi, các node khác vẫn có thể phục vụ request đọc/ghi, đảm bảo hệ thống không bị gián đoạn.
- **Khả năng mở rộng và cân bằng tải:**
    - Dữ liệu được phân mảnh (shard) và phân phối tự động trên nhiều node, giúp tăng hiệu năng đọc/ghi và khả năng mở rộng của hệ thống.
- **Điều chỉnh mức độ nhất quán khi ghi:**
    - Elasticsearch hỗ trợ các tham số như `wait_for_active_shards`, `refresh`cho phép điều chỉnh mức độ nhất quán khi cần thiết (ví dụ: đảm bảo dữ liệu đã được ghi vào nhiều node trước khi trả về thành công).

#### Nhược điểm

- **Không đảm bảo strong consistency mặc định:**
    - Khi ghi dữ liệu, Elasticsearch trả về thành công ngay khi ghi xong vào primary shard, các replica sẽ được đồng bộ sau. Nếu đọc ngay sau khi ghi (và đọc từ replica), có thể không thấy dữ liệu mới (read-after-write inconsistency).
- **Chưa đảm bảo được tất cả dữ liệu sẽ được tìm kiếm:**
    - Nếu request đọc được chuyển đến replica chưa đồng bộ, kết quả trả về có thể không phản ánh dữ liệu vừa ghi.
- **Không thể thay đổi số lượng primary shard sau khi đã tạo index:**
    - Nếu cần thay đổi số lượng shard để tối ưu hiệu năng hoặc dung lượng, phải tạo index mới và reindex dữ liệu.
- **Tăng mức nhất quán sẽ giảm hiệu năng:**
    - Nếu cấu hình mức nhất quán cao (ví dụ: `wait_for_active_shards=all`), tốc độ ghi sẽ chậm lại do phải chờ nhiều node xác nhận.


### 4.9. Leader Election (Bầu chọn Leader)

* **Đánh giá hiện trạng:** Đạt.
    * **Elasticsearch:** Có cơ chế bầu chọn **master node** tự động. Master node chịu trách nhiệm quản lý cluster. Nếu master node hiện tại gặp sự cố, các master-eligible nodes khác sẽ tiến hành bầu chọn một master mới từ trong số chúng.
        * Cấu hình `cluster.initial_master_nodes` trong `docker-compose.yml` (`es01`, `es02`) giúp quá trình bootstrapping cluster và bầu chọn master ban đầu.
        * Để tránh "split-brain" (khi cluster bị chia thành nhiều phần, mỗi phần tự bầu master)
    * **MongoDB:** Nếu triển khai dưới dạng Replica Set, MongoDB cũng có cơ chế bầu chọn **primary node** tự động.
* **Điểm mạnh:** Cơ chế bầu chọn leader tự động và mạnh mẽ của Elasticsearch.
* **Điểm yếu/Hạn chế:**
    * Cấu hình hiện tại với `cluster.initial_master_nodes: ["es01", "es02"]` và 3 node tổng cộng (`es01`, `es02`, `es03`) là ổn cho việc khởi tạo. Cần đảm bảo rằng trong quá trình hoạt động, số lượng node có thể làm master được duy trì để tránh mất quorum.
    * MongoDB chưa có replica set nên chưa có leader election cho database.
* **Hướng cải thiện/Giải pháp:**
    * **Elasticsearch:** Đảm bảo cấu hình `discovery.seed_hosts` và `cluster.initial_master_nodes` là chính xác. Với 3 node, nếu cả 3 đều là master-eligible, thì quorum sẽ là 2. Nếu một node lỗi, cluster vẫn hoạt động. Nếu 2 node lỗi, cluster sẽ mất quorum và không thể bầu master mới (read-only).
    * **MongoDB:** Triển khai Replica Set.

### 4.10. Security Features (Tính năng bảo mật)

* **Đánh giá hiện trạng:** Còn yếu.
    * **Elasticsearch:** Phiên bản Elasticsearch 8.x mặc định bật một số tính năng bảo mật cơ bản (như yêu cầu xác thực và TLS nếu được cấu hình tự động). Tuy nhiên, trong `docker-compose.yml` hiện tại, có vẻ như các biến môi trường như `xpack.security.enabled=false` hoặc tương tự không được đặt rõ ràng, hoặc mật khẩu mặc định `elastic` có thể đang được sử dụng nếu security được auto-configure. Cần kiểm tra kỹ cấu hình thực tế khi cluster khởi chạy.
    * **MongoDB:** Mặc định, MongoDB có thể không bật xác thực.
    * **Giao tiếp mạng:**
        * Giao tiếp giữa client và Nginx có thể là HTTP.
        * Giao tiếp giữa Nginx và Webapp có thể là HTTP.
        * Giao tiếp giữa Webapp và Elasticsearch/MongoDB có thể không được mã hóa.
    * **Web Application:** Có cơ chế xác thực người dùng (`authController.js`, `authMiddleware.js`), đây là một điểm tốt. Tuy nhiên, cần xem xét các khía cạnh bảo mật khác như XSS, CSRF, SQL Injection (mặc dù dùng NoSQL nhưng vẫn có các rủi ro tương tự như NoSQL injection).
* **Điểm yếu/Hạn chế:**
    * Thiếu mã hóa cho giao tiếp giữa các thành phần.
    * Xác thực và phân quyền có thể chưa được cấu hình đầy đủ cho Elasticsearch và MongoDB.
    * Mật khẩu mặc định hoặc yếu có thể đang được sử dụng.
* **Hướng cải thiện/Giải pháp:**
    * **Mã hóa mọi nơi (Encryption in transit):**
        * Client ↔ Nginx: Cấu hình HTTPS trên Nginx (sử dụng SSL/TLS certificate, có thể dùng Let's Encrypt cho môi trường public hoặc self-signed certificate cho môi trường nội bộ/test).
        * Nginx ↔ Webapp: Nếu Nginx và Webapp chạy trên cùng một Docker network được bảo vệ, HTTP có thể chấp nhận được, nhưng HTTPS vẫn tốt hơn.
        * Webapp ↔ Elasticsearch: Kích hoạt TLS trên HTTP layer của Elasticsearch. Cấu hình Node.js client để sử dụng HTTPS và tin tưởng certificate của Elasticsearch.
        * Webapp ↔ MongoDB: Kích hoạt TLS/SSL cho kết nối MongoDB.
        * Elasticsearch internode communication: Kích hoạt TLS cho transport layer giữa các node Elasticsearch.
    * **Xác thực và Phân quyền (Authentication & Authorization):**
        * **Elasticsearch:**
            * Kích hoạt X-Pack Security (nếu chưa).
            * Thay đổi mật khẩu mặc định cho user `elastic` và các built-in users khác.
            * Tạo các user riêng cho Web Application với quyền hạn tối thiểu cần thiết (ví dụ: chỉ quyền đọc/ghi trên index sản phẩm, không có quyền quản trị cluster). Sử dụng API keys hoặc role-based access control (RBAC).
        * **MongoDB:**
            * Kích hoạt xác thực.
            * Tạo user riêng cho Web Application với quyền hạn tối thiểu trên database.
    * **Bảo mật Web Application:**
        * Sử dụng thư viện như `helmet` trong Express.js để bảo vệ khỏi các lỗ hổng web phổ biến.
        * Validate và sanitize tất cả dữ liệu đầu vào từ người dùng.
        * Lưu trữ mật khẩu người dùng một cách an toàn (sử dụng hashing mạnh như bcrypt).
    * **Quản lý Secrets:** Không hardcode mật khẩu, API keys trong code hoặc `docker-compose.yml`. Sử dụng Docker secrets, biến môi trường được inject lúc runtime, hoặc các công cụ quản lý secret như HashiCorp Vault.
    * **Network Policies:** Nếu triển khai trên Kubernetes, sử dụng Network Policies để giới hạn giao tiếp giữa các pods. Trong Docker, sử dụng các Docker networks riêng biệt để cô lập các nhóm service.

### 4.11. Deployment Automation (Tự động hóa triển khai)

* **Đánh giá hiện trạng:** Đạt ở mức tốt.
    * Hệ thống sử dụng `docker-compose.yml` để định nghĩa và quản lý việc triển khai tất cả các services (Elasticsearch, MongoDB, Webapp, Nginx). Điều này cho phép triển khai toàn bộ hệ thống bằng một lệnh duy nhất (`docker-compose up`).
* **Điểm mạnh:** Dễ dàng thiết lập môi trường phát triển và thử nghiệm. Đảm bảo tính nhất quán của môi trường triển khai.
* **Điểm yếu/Hạn chế:**
    * `docker-compose` phù hợp cho single-host deployment hoặc môi trường dev/test. Đối với production trên nhiều host, cần các công cụ điều phối container mạnh mẽ hơn.
    * Chưa có quy trình CI/CD (Continuous Integration/Continuous Deployment) tự động.
* **Hướng cải thiện/Giải pháp:**
    * **Môi trường Production:**
        * **Docker Swarm:** Là giải pháp điều phối container tích hợp sẵn trong Docker, dễ học hơn Kubernetes.
        * **Kubernetes (K8s):** Là tiêu chuẩn công nghiệp cho việc điều phối container ở quy mô lớn, cung cấp khả năng tự phục hồi, mở rộng tự động, rolling updates mạnh mẽ. Việc chuyển đổi sang K8s đòi hỏi viết các file manifest (YAML) cho Deployments, Services, ConfigMaps, Secrets, etc.
    * **CI/CD Pipeline:**
        * Sử dụng các công cụ như Jenkins, GitLab CI/CD, GitHub Actions.
        * **Quy trình mẫu:**
            1.  Developer push code lên Git repository.
            2.  CI server tự động build Docker images cho Web Application.
            3.  Chạy unit tests, integration tests.
            4.  Push Docker images lên một container registry (Docker Hub, AWS ECR, Google GCR).
            5.  CD server tự động triển khai phiên bản mới của ứng dụng lên môi trường staging/production (ví dụ: cập nhật image tag trong Kubernetes Deployment hoặc Docker Compose file rồi chạy lại).
        * **Ví dụ (GitHub Actions):** Tạo một workflow file `.github/workflows/ci-cd.yml` để tự động build và push Docker image khi có commit vào branch `main`.

## 5. Tổng kết và Đánh giá

### 5.1. Các Tiêu chí Đã Đạt được

Dựa trên phân tích source code và cấu hình hiện tại, dự án đã đạt được các tiêu chí sau ở mức độ tốt hoặc cơ bản:

* **Distributed Communication:** Các thành phần giao tiếp qua mạng.
* **Sharding hoặc Replication:** Elasticsearch sử dụng cả hai cơ chế này hiệu quả.
* **System Recovery (Rejoin after Failure):** Elasticsearch có khả năng tự phục hồi tốt. Docker `restart:always` giúp các service khác tự khởi động lại.
* **Load Balancing:** Nginx được sử dụng cho webapp, và client Elasticsearch cũng có khả năng phân phối request.
* **Leader Election:** Elasticsearch có cơ chế bầu chọn master node.
* **Deployment Automation:** Sử dụng `docker-compose` giúp tự động hóa triển khai trên một host.

### 5.2. Các Tiêu chí Chưa Đạt được hoặc Cần Cải thiện Mạnh mẽ

* **Fault Tolerance:** Tầng webapp và MongoDB là SPOF. Cần triển khai replicas.
* **Simple Monitoring / Logging:** Hiện tại rất cơ bản. Cần giải pháp tập trung như Elasticsearch Exporter + Prometheus + Grafana cho monitoring và ELK/EFK cho logging.
* **Basic Stress Test:** Cần được thực hiện để có đánh giá thực tế.
* **Consistency Guarantees:** Cần hiểu rõ và cấu hình phù hợp hơn với yêu cầu nghiệp vụ, đặc biệt là đồng bộ giữa MongoDB và Elasticsearch.
* **Security Features:** Đây là điểm yếu lớn, cần cải thiện toàn diện từ mã hóa giao tiếp, xác thực, phân quyền đến bảo mật ứng dụng.

### 5.3. Hạn chế của Đồ án

* **Giám sát và Ghi log:** Hệ thống giám sát và ghi log còn sơ sài, chưa đáp ứng nhu cầu theo dõi và vận hành một hệ thống phân tán phức tạp.
* **Bảo mật:** Các khía cạnh bảo mật chưa được chú trọng đúng mức, tiềm ẩn nhiều rủi ro.
* **Khả năng chịu lỗi của toàn hệ thống:** Mặc dù Elasticsearch có khả năng chịu lỗi tốt, các thành phần khác như Web Application và MongoDB (trong cấu hình hiện tại) vẫn là điểm yếu.
* **Quy trình đồng bộ dữ liệu:** Cơ chế đồng bộ dữ liệu giữa MongoDB và Elasticsearch cần được thiết kế kỹ lưỡng hơn để đảm bảo tính nhất quán và độ tin cậy, đặc biệt khi có lỗi xảy ra.
* **Kiểm thử:** Thiếu kiểm thử toàn diện, đặc biệt là stress test và kiểm thử khả năng chịu lỗi.

### 5.4. Hướng Phát triển Tương lai

1.  **Hoàn thiện Giám sát và Logging:** Ưu tiên hàng đầu là triển khai giải pháp giám sát (Prometheus, Grafana, Elasticsearch Exporter) và logging tập trung (ELK/EFK stack).
2.  **Tăng cường Bảo mật:** Áp dụng các biện pháp bảo mật đã đề xuất: mã hóa, xác thực mạnh, phân quyền chi tiết, bảo vệ ứng dụng web.
3.  **Nâng cao Khả năng chịu lỗi:** Triển khai Web Application với nhiều replicas và MongoDB dưới dạng Replica Set.
4.  **Tối ưu hóa Hiệu năng:** Thực hiện stress test, phân tích các truy vấn chậm, cải thiện mapping, sử dụng các loại query phù hợp.
5.  **Cải thiện Cơ chế Đồng bộ Dữ liệu:** Sử dụng message queue (Kafka, RabbitMQ) để đồng bộ dữ liệu giữa MongoDB và Elasticsearch một cách bất đồng bộ và đáng tin cậy.
6.  **Triển khai trên Môi trường Production:** Xem xét sử dụng Kubernetes hoặc Docker Swarm. Tự động hóa quy trình CI/CD (Continuous Integration/Continuous Deployment).
7.  **Mở rộng Tính năng:**
    * Gợi ý tìm kiếm (Search-as-you-type / Autocomplete).
    * Tìm kiếm theo vị trí địa lý (Geospatial search).
    * Phân tích dữ liệu tìm kiếm để hiểu hành vi người dùng.
    * Cá nhân hóa kết quả tìm kiếm.
8.  **Backup và Restore:** Thiết lập chiến lược backup (snapshot) định kỳ cho Elasticsearch và MongoDB, cùng quy trình restore tin cậy.

## Lời Kết

Đồ án "Ứng dụng Tìm kiếm Phân tán với Elasticsearch" đã thành công trong việc xây dựng một hệ thống tìm kiếm cơ bản, tận dụng được các ưu điểm của Elasticsearch về khả năng phân tán và hiệu năng. Việc phân tích dựa trên các tiêu chí kỹ thuật đã chỉ ra những mặt đã đạt được cũng như các khía cạnh cần cải thiện, đặc biệt là về giám sát, bảo mật và khả năng chịu lỗi của toàn hệ thống. Các giải pháp và hướng phát triển được đề xuất sẽ là kim chỉ nam quan trọng để tiếp tục hoàn thiện và nâng cao chất lượng của ứng dụng, đưa nó đến gần hơn với một hệ thống sẵn sàng cho môi trường production.

---

**Tài liệu tham khảo:**

- Tài liệu chính thức của Elasticsearch: https://www.elastic.co/guide/index.html
- Tài liệu Docker: https://docs.docker.com/
- Tài liệu Node.js: https://nodejs.org/en/docs/
- Tài liệu MongoDB: https://www.mongodb.com/docs/
- Tài liệu Nginx: https://nginx.org/en/docs/
- Prometheus: https://prometheus.io/docs/
- Grafana: https://grafana.com/docs/
- Elasticsearch Exporter: https://github.com/prometheus-community/elasticsearch_exporter


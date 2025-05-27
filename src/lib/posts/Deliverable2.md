---
title: "Deliverable 2: Website"
date: "2025-05-11"
updated: "2025-05-11"
categories:
  - "sveltekit"
  - "markdown"
coverImage: "https://cdn2.fptshop.com.vn/unsafe/2024_5_6_638506204957187926_rabbitmq_.jpg"
coverWidth: 16
coverHeight: 9
excerpt: Check out how heading links work with this starter in this post.
---


# Kế Hoạch Dự Án: Xây Dựng Hệ Thống Tìm Kiếm Phân Tán (Ý Tưởng Ban Đầu)

Bài viết này trình bày ý tưởng và kế hoạch ban đầu để xây dựng một hệ thống tìm kiếm phân tán. Mục tiêu là hình dung một nền tảng cho phép người dùng tìm kiếm sản phẩm một cách nhanh chóng và chính xác, đồng thời phác thảo giao diện quản trị để quản lý dữ liệu, tất cả đều ở giai đoạn ý tưởng trước khi triển khai.

## 1. Kiến Trúc Hệ Thống Dự Kiến

Kiến trúc dự kiến của hệ thống sẽ hướng tới mô hình microservices, với ý tưởng sử dụng công nghệ container hóa (ví dụ: Docker) để đóng gói và triển khai các thành phần. Sơ đồ dưới đây mô tả kiến trúc tổng thể được hình dung:

![Sơ đồ kiến trúc hệ thống](/image.png)

**Chú thích sơ đồ (Ý tưởng ban đầu):**

* **UserBrowser/AdminBrowser:** Hình dung người dùng cuối và quản trị viên tương tác với hệ thống qua trình duyệt.
* **Load Balancer / API Gateway (LB):** Ý tưởng về một thành phần phân phối tải và quản lý truy cập API.
* **WebApp (Ứng dụng Web Chính):** Dự kiến là một ứng dụng Node.js, đóng vai trò trung tâm xử lý logic và giao tiếp.
* **MongoDB Database:** Ý tưởng sử dụng MongoDB để lưu trữ dữ liệu chính của hệ thống.
* **Cụm Elasticsearch (Elasticsearch Cluster):** Hình dung một cụm gồm nhiều máy chủ Elasticsearch để phục vụ tìm kiếm.
* **ESClusterInterface:** Một lớp trừu tượng hoặc module trong WebApp để tương tác với Elasticsearch.
* **Kibana:** Ý tưởng sử dụng Kibana để trực quan hóa dữ liệu từ Elasticsearch.
* **Logstash:** Ý tưởng sử dụng Logstash cho việc xử lý log và có thể là đồng bộ dữ liệu.

## 2. Mô Tả Chi Tiết Các Thành Phần Dự Kiến

### a. Ứng dụng Web Chính (Node.js)

* **Vai trò hình dung:**
    * Cung cấp các điểm cuối API (API endpoints) cho các chức năng như tìm kiếm, quản lý sản phẩm, quản lý người dùng.
    * Xử lý logic nghiệp vụ cốt lõi.
    * Tương tác với MongoDB để lưu trữ và truy xuất dữ liệu.
    * Gửi yêu cầu tìm kiếm đến cụm Elasticsearch và nhận kết quả.
    * Có thể phục vụ giao diện người dùng hoặc chỉ là một backend API.
    * Xử lý các vấn đề liên quan đến xác thực và phân quyền người dùng.
* **Cách thức hoạt động hình dung:**
    * Sử dụng một framework web của Node.js (ví dụ: Express.js) để xây dựng.
    * Logic có thể được tổ chức theo các module chức năng.
    * Việc tương tác với cơ sở dữ liệu và Elasticsearch sẽ thông qua các thư viện hoặc module phù hợp.

### b. Cơ sở dữ liệu MongoDB

* **Vai trò hình dung:**
    * Lưu trữ thông tin tài khoản người dùng (ví dụ: tên đăng nhập, thông tin xác thực, vai trò).
    * Lưu trữ thông tin chi tiết về sản phẩm (ví dụ: tên, mô tả, giá, danh mục). Đây được xem là nguồn dữ liệu gốc.
    * Có thể lưu trữ lịch sử tìm kiếm của người dùng.
* **Cách thức hoạt động hình dung:**
    * Ứng dụng Web Chính sẽ kết nối và thực hiện các thao tác đọc/ghi dữ liệu với MongoDB.
    * Dữ liệu sản phẩm từ đây sẽ là nguồn để đưa vào Elasticsearch cho mục đích tìm kiếm.
    * **Sự phối hợp:** Khi một sản phẩm mới được thêm vào hoặc thông tin sản phẩm được cập nhật trong MongoDB (thông qua Ứng dụng Web Chính), một cơ chế (có thể là event-driven hoặc batch processing dùng Logstash, hoặc logic trong ứng dụng) sẽ được kích hoạt để đồng bộ những thay đổi này sang Elasticsearch. Điều này đảm bảo rằng dữ liệu tìm kiếm luôn được cập nhật.

### c. Cụm Elasticsearch

* **Vai trò hình dung:**
    * Cung cấp khả năng tìm kiếm toàn văn bản (full-text search) hiệu suất cao.
    * Lưu trữ một bản sao hoặc một phiên bản đã được xử lý (indexed) của dữ liệu sản phẩm để tối ưu cho tìm kiếm.
    * Hỗ trợ các tính năng tìm kiếm nâng cao như tìm kiếm mờ, gợi ý, lọc kết quả.
    * Được thiết kế để có khả năng mở rộng và chịu lỗi thông qua việc phân tán dữ liệu trên nhiều node.
* **Cách thức hoạt động hình dung:**
    * **Thiết lập cụm:** Bao gồm nhiều máy chủ (node) Elasticsearch hoạt động cùng nhau. Các node giao tiếp với nhau để duy trì trạng thái của cụm.
    * **Sharding (Phân mảnh):**
        * **Vai trò:** Để xử lý lượng lớn dữ liệu và tăng tốc độ truy vấn, dữ liệu của một "index" (tương đương một bảng trong cơ sở dữ liệu quan hệ, ví dụ: index "products") sẽ được chia thành nhiều phần nhỏ hơn gọi là "shards" (mảnh). Mỗi shard là một Lucene index độc lập và đầy đủ chức năng.
        * **Cách hoạt động:** Khi một tài liệu (ví dụ: một sản phẩm) được index, Elasticsearch sẽ quyết định shard nào sẽ lưu trữ tài liệu đó. Thuật toán mặc định thường là: `shard_num = hash(routing_value) % num_primary_shards`. `routing_value` mặc định là ID của tài liệu. `num_primary_shards` là số lượng shard chính được cấu hình khi tạo index. Điều này đảm bảo tài liệu được phân bổ đều trên các shard.
        * **Lợi ích:** Cho phép lưu trữ dữ liệu vượt quá khả năng của một node đơn lẻ và song song hóa các hoạt động tìm kiếm trên nhiều shard, sau đó tổng hợp kết quả.
    * **Replication (Sao chép):**
        * **Vai trò:** Để đảm bảo tính sẵn sàng cao (high availability) và khả năng chịu lỗi (fault tolerance), cũng như tăng thông lượng đọc, mỗi primary shard có thể có một hoặc nhiều "replica shards" (bản sao).
        * **Cách hoạt động:** Replica shard là một bản sao chính xác của primary shard. Nó được lưu trữ trên một node khác với primary shard của nó. Mọi thao tác ghi (index, update, delete) trước tiên được thực hiện trên primary shard, sau đó được sao chép đồng bộ hoặc bất đồng bộ (tùy cấu hình) sang các replica shard.
        * **Logic khi có sự cố:** Nếu node chứa primary shard gặp lỗi, một trong các replica shard của nó sẽ tự động được "thăng cấp" (promoted) thành primary shard mới. Quá trình này được quản lý bởi master node của cụm Elasticsearch.
        * **Lợi ích:** Dữ liệu không bị mất khi một node gặp sự cố. Các truy vấn đọc (tìm kiếm) có thể được phục vụ bởi cả primary và replica shards, giúp phân tán tải đọc.
    * **Logic tìm kiếm tổng quát:**
        1.  Ứng dụng Web Chính (thông qua `ESClusterInterface`) gửi yêu cầu tìm kiếm đến một node bất kỳ trong cụm Elasticsearch. Node này được gọi là "coordinating node" cho yêu cầu đó.
        2.  Coordinating node phân tích truy vấn và xác định những shard nào (cả primary và replica của các index liên quan) cần tham gia vào việc tìm kiếm.
        3.  Coordinating node chuyển tiếp yêu cầu tìm kiếm đến các shard đó.
        4.  Mỗi shard thực thi tìm kiếm cục bộ trên dữ liệu của mình và trả về kết quả (thường là ID tài liệu và điểm relevancy) cho coordinating node.
        5.  Coordinating node tổng hợp kết quả từ tất cả các shard, sắp xếp chúng (ví dụ, theo điểm relevancy), lấy thông tin chi tiết của các tài liệu cần thiết từ các shard liên quan, và cuối cùng trả về kết quả hoàn chỉnh cho Ứng dụng Web Chính.
    * **Sự phối hợp:** Elasticsearch nhận dữ liệu đã được xử lý (indexed) từ MongoDB (thông qua Logstash hoặc logic đồng bộ trong Ứng dụng Web Chính). Khi người dùng thực hiện tìm kiếm trên Ứng dụng Web Chính, yêu cầu sẽ được chuyển đến Elasticsearch để tận dụng khả năng tìm kiếm mạnh mẽ của nó.

### d. Công cụ Kibana

* **Vai trò hình dung:**
    * Cung cấp giao diện đồ họa để xem, khám phá và phân tích dữ liệu được lưu trong Elasticsearch.
    * Hỗ trợ việc theo dõi hoạt động của cụm Elasticsearch và gỡ lỗi các truy vấn tìm kiếm.
* **Cách thức hoạt động hình dung:** Kibana kết nối với cụm Elasticsearch (thông qua `ESClusterInterface` hoặc trực tiếp nếu cấu hình cho phép) và cho phép người quản trị tạo các biểu đồ, bảng điều khiển (dashboards) dựa trên dữ liệu trong Elasticsearch. Nó không trực tiếp tham gia vào luồng xử lý tìm kiếm của người dùng cuối nhưng rất quan trọng cho việc vận hành và giám sát.

### e. Công cụ Logstash

* **Vai trò hình dung:**
    * Thu thập log từ các thành phần khác nhau của hệ thống (ví dụ: từ Ứng dụng Web Chính).
    * Xử lý (ví dụ: phân tích, làm giàu) log và gửi chúng đến một nơi lưu trữ tập trung, có thể là Elasticsearch.
    * **Đồng bộ dữ liệu (ý tưởng):** Có thể được cấu hình để đọc dữ liệu từ MongoDB và ghi vào Elasticsearch, giúp giữ cho công cụ tìm kiếm luôn cập nhật.
* **Cách thức hoạt động hình dung:**
    * Hoạt động dựa trên các "pipeline" được cấu hình, định nghĩa nguồn dữ liệu đầu vào (ví dụ: file log, database), các bộ lọc (filter) để chuyển đổi dữ liệu, và đích đầu ra (ví dụ: Elasticsearch).
    * **Trong việc đồng bộ dữ liệu:** Logstash có thể sử dụng một input plugin để kết nối với MongoDB, định kỳ truy vấn các thay đổi (ví dụ, dựa trên timestamp hoặc một trường cờ). Sau đó, nó sử dụng các filter plugin để chuyển đổi dữ liệu này thành định dạng phù hợp cho Elasticsearch và cuối cùng sử dụng output plugin để ghi dữ liệu vào index tương ứng trong Elasticsearch.
    * **Sự phối hợp:** Logstash hoạt động như một cầu nối, giúp di chuyển và chuyển đổi dữ liệu từ MongoDB sang Elasticsearch một cách tự động, giảm tải cho Ứng dụng Web Chính trong việc thực hiện đồng bộ.

### f. Giao thức giao tiếp dự kiến

- Người dùng đến Ứng dụng Web Chính: HTTPS để đảm bảo an toàn.
- Ứng dụng Web Chính đến MongoDB: Giao thức TCP/IP, thông qua thư viện kết nối MongoDB (ví dụ: driver Node.js cho MongoDB).
- Ứng dụng Web Chính đến Elasticsearch: Giao thức HTTP REST API, thông qua thư viện client của Elasticsearch cho Node.js.
- Giữa các node Elasticsearch: Giao thức transport nội bộ của Elasticsearch qua TCP/IP (thường cổng 9300) để trao đổi thông tin trạng thái cụm, sao chép shard, và điều phối truy vấn.
- Kibana/Logstash đến Elasticsearch: Giao thức HTTP REST API (thường cổng 9200).

## 3. Công Nghệ và Thư Viện Dự Kiến Sử Dụng

Việc lựa chọn công nghệ sẽ dựa trên các yếu tố như sự phổ biến, cộng đồng hỗ trợ, hiệu năng, sự quen thuộc của đội ngũ phát triển (là bạn) và khả năng đáp ứng yêu cầu dự án. Dưới đây là các công nghệ và thư viện đang được cân nhắc:

* **Nền tảng Backend:**
    * **Node.js:**
        * **Lý do chọn:** Với kiến thức JavaScript sẵn có của bạn, Node.js là lựa chọn tự nhiên. Mô hình I/O không chặn (non-blocking I/O) của Node.js rất phù hợp cho các ứng dụng web có nhiều thao tác đọc/ghi, như tương tác với cơ sở dữ liệu và dịch vụ tìm kiếm. Cộng đồng lớn và hệ sinh thái thư viện (npm) phong phú.
    * **Express.js (hoặc một framework tương tự như Fastify, Koa):**
        * **Lý do chọn:** Express.js là một framework web tối giản, linh hoạt và rất phổ biến cho Node.js, giúp đơn giản hóa việc xây dựng API, quản lý route và middleware. Bạn đã có kinh nghiệm với Express, điều này sẽ tăng tốc độ phát triển.
* **Cơ sở dữ liệu chính (Nguồn dữ liệu gốc):**
    * **MongoDB:**
        * **Lý do chọn:** Là cơ sở dữ liệu NoSQL dạng document, cung cấp sự linh hoạt cao trong việc định nghĩa schema, phù hợp để lưu trữ thông tin sản phẩm có thể có nhiều thuộc tính đa dạng và thay đổi. Khả năng mở rộng tốt.
    * **Mongoose (hoặc một ODM/Thư viện tương tự):**
        * **Lý do chọn:** Mongoose cung cấp một giải pháp dựa trên schema để mô hình hóa dữ liệu ứng dụng cho MongoDB, bao gồm validation, query building, và business logic hooks. Nó giúp làm việc với MongoDB từ Node.js một cách có cấu trúc và dễ quản lý hơn.
* **Công cụ tìm kiếm:**
    * **Elasticsearch:**
        * **Lý do chọn:** Là một trong những công cụ tìm kiếm và phân tích phân tán mạnh mẽ và phổ biến nhất hiện nay, dựa trên Lucene. Cung cấp khả năng tìm kiếm toàn văn bản hiệu suất cao, khả năng mở rộng tốt, tính sẵn sàng cao, và hỗ trợ nhiều loại truy vấn phức tạp. Phù hợp với mục tiêu xây dựng hệ thống tìm kiếm chất lượng.
    * **Thư viện client Elasticsearch cho Node.js (ví dụ: `@elastic/elasticsearch`):**
        * **Lý do chọn:** Thư viện chính thức từ Elastic, cung cấp giao diện lập trình thuận tiện và đầy đủ để tương tác với cụm Elasticsearch từ ứng dụng Node.js.
    * **Cân nhắc giải pháp đồng bộ dữ liệu từ MongoDB sang Elasticsearch:**
        * **Logstash:** Như đã đề cập, Logstash có thể đảm nhận vai trò này một cách độc lập với ứng dụng chính.
        * **Mongoosastic (Plugin cho Mongoose):** Nếu muốn tích hợp chặt chẽ việc đồng bộ vào Mongoose models.
        * **Logic tùy chỉnh trong ứng dụng:** Viết code trong Ứng dụng Web Chính để lắng nghe sự kiện thay đổi trong MongoDB và cập nhật Elasticsearch. Lựa chọn này cho phép kiểm soát cao nhất nhưng cũng đòi hỏi nhiều công sức phát triển hơn.
* **Frontend (Lựa chọn linh hoạt, tùy thuộc vào yêu cầu giao diện):**
    * **Render phía máy chủ với template engine (ví dụ: EJS, Pug, Handlebars):**
        * **Lý do chọn (nếu có):** Đơn giản, dễ tích hợp với Express.js, phù hợp cho các trang không yêu cầu tương tác phức tạp phía client.
    * **Xây dựng Single Page Application (SPA) với React, Vue.js, hoặc Angular:**
        * **Lý do chọn (nếu có):** Mang lại trải nghiệm người dùng mượt mà, tương tác cao. Phù hợp nếu hệ thống có nhiều chức năng phức tạp phía client. Với kinh nghiệm JavaScript của bạn, việc học một trong các framework này sẽ không quá khó khăn.
* **Xác thực & Bảo mật:**
    * **JSON Web Tokens (JWT):**
        * **Lý do chọn:** Một chuẩn mở (RFC 7519) phổ biến để tạo access token, phù hợp cho việc xác thực API trong kiến trúc microservices hoặc khi frontend và backend tách biệt.
    * **Thư viện mã hóa mật khẩu (ví dụ: `bcryptjs` hoặc `argon2`):**
        * **Lý do chọn:** `bcryptjs` là một lựa chọn tốt và phổ biến để hash mật khẩu một cách an toàn, chống lại các cuộc tấn công brute-force và rainbow table. `argon2` là một thuật toán mới hơn và mạnh mẽ hơn nhưng có thể yêu cầu nhiều tài nguyên hơn.
* **Containerization & Triển khai:**
    * **Docker:**
        * **Lý do chọn:** Công nghệ container hóa hàng đầu, giúp đóng gói ứng dụng và tất cả các dependencies của nó vào một container di động và nhất quán. Đơn giản hóa việc thiết lập môi trường phát triển và đảm bảo tính nhất quán giữa các môi trường.
    * **Docker Compose:**
        * **Lý do chọn:** Công cụ để định nghĩa và chạy các ứng dụng Docker đa container. Rất hữu ích để quản lý toàn bộ hệ thống (WebApp, MongoDB, Elasticsearch cluster, Kibana, Logstash) trên môi trường phát triển cục bộ bằng một file cấu hình duy nhất.
    * **(Kế hoạch mở rộng cho Production) Kubernetes hoặc Docker Swarm:**
        * **Lý do chọn:** Khi hệ thống cần mở rộng và yêu cầu tính sẵn sàng cao hơn trong môi trường production, các công cụ điều phối container như Kubernetes (mạnh mẽ, nhiều tính năng, cộng đồng lớn) hoặc Docker Swarm (đơn giản hơn, tích hợp sẵn với Docker) sẽ là cần thiết.
* **Quản lý cấu hình:**
    * **Sử dụng biến môi trường (ví dụ: qua file `.env` với thư viện như `dotenv`):**
        * **Lý do chọn:** Một thực hành tốt để quản lý các thông tin cấu hình nhạy cảm (như chuỗi kết nối cơ sở dữ liệu, khóa bí mật) tách biệt khỏi mã nguồn, giúp dễ dàng thay đổi cấu hình giữa các môi trường khác nhau (development, staging, production).
* **Quản lý Session (Nếu ứng dụng web có trạng thái phía server và không hoàn toàn dựa vào JWT cho mọi tương tác):**
    * **Middleware quản lý session cho Express (ví dụ: `express-session`).**
    * **Lưu trữ session vào một nơi persistent (ví dụ: `connect-mongo` để lưu vào MongoDB, hoặc Redis):**
        * **Lý do chọn:** Để session không bị mất khi server khởi động lại hoặc khi có nhiều instance của WebApp chạy sau một load balancer.
* **Giao diện người dùng (Styling):**
    * **Sử dụng một CSS Framework (ví dụ: Tailwind CSS, Bootstrap, Materialize):**
        * **Lý do chọn:** Giúp tăng tốc độ phát triển giao diện người dùng, cung cấp các component dựng sẵn và hệ thống lưới responsive, đảm bảo giao diện trông chuyên nghiệp và nhất quán.

## 4. Mô Hình Dữ Liệu Dự Kiến (Mô tả ý tưởng)

Ở giai đoạn này, chúng ta sẽ phác thảo các loại thông tin (các trường dữ liệu chính) dự kiến cần lưu trữ cho mỗi thực thể chính trong MongoDB, thay vì đi sâu vào định nghĩa schema code cụ thể.

### a. Thông tin Người dùng (User Entity)

* **Mục đích:** Quản lý tài khoản người dùng, xác thực và phân quyền.
* **Các trường dữ liệu dự kiến:**
    * `userId`: Định danh duy nhất cho mỗi người dùng (ví dụ: ObjectId của MongoDB).
    * `username`: Tên đăng nhập duy nhất.
    * `email`: Địa chỉ email duy nhất, có thể dùng để khôi phục mật khẩu.
    * `passwordHash`: Mật khẩu đã được hash an toàn (không bao giờ lưu mật khẩu gốc).
    * `role`: Vai trò của người dùng trong hệ thống (ví dụ: 'user', 'admin') để phân quyền truy cập.
    * `createdAt`: Dấu thời gian khi tài khoản được tạo.
    * `updatedAt`: Dấu thời gian khi thông tin tài khoản được cập nhật lần cuối.
    * `isActive`: Trạng thái tài khoản (ví dụ: true/false).

### b. Thông tin Sản phẩm (Product Entity)

* **Mục đích:** Lưu trữ tất cả thông tin chi tiết về các sản phẩm sẽ được tìm kiếm. Đây là "source of truth" cho dữ liệu sản phẩm.
* **Các trường dữ liệu dự kiến:**
    * `productId`: Định danh duy nhất cho mỗi sản phẩm.
    * `name`: Tên sản phẩm (quan trọng cho tìm kiếm).
    * `description`: Mô tả chi tiết về sản phẩm (quan trọng cho tìm kiếm).
    * `price`: Giá sản phẩm.
    * `currency`: Đơn vị tiền tệ (ví dụ: 'VND', 'USD').
    * `category`: Danh mục chính của sản phẩm (ví dụ: 'Điện thoại', 'Laptop', 'Thời trang nam').
    * `subCategory`: Danh mục phụ (nếu có).
    * `brand`: Thương hiệu của sản phẩm.
    * `sku`: Mã SKU (Stock Keeping Unit) nếu có.
    * `imageUrl`: Mảng các đường dẫn đến hình ảnh sản phẩm.
    * `stockQuantity`: Số lượng tồn kho.
    * `attributes`: Một đối tượng hoặc mảng các đối tượng chứa các thuộc tính đặc tả khác của sản phẩm (ví dụ: `{ "color": "Đen", "size": "XL" }`, hoặc `[ { "key": "color", "value": "Đen"}, {"key": "storage", "value": "256GB"} ]`). Quan trọng cho việc lọc kết quả tìm kiếm.
    * `tags`: Mảng các từ khóa hoặc thẻ (tags) liên quan đến sản phẩm (ví dụ: ['gaming', 'new arrival', 'best seller']). Hỗ trợ tìm kiếm.
    * `averageRating`: Điểm đánh giá trung bình của sản phẩm (từ 0 đến 5).
    * `numberOfRatings`: Số lượng người đã đánh giá sản phẩm.
    * `isPublished`: Trạng thái sản phẩm có được hiển thị công khai hay không (true/false).
    * `createdAt`: Dấu thời gian khi sản phẩm được thêm vào hệ thống.
    * `updatedAt`: Dấu thời gian khi thông tin sản phẩm được cập nhật lần cuối.

### c. Thông tin Lịch sử tìm kiếm (Search History Entity)

* **Mục đích:** Lưu lại các truy vấn tìm kiếm của người dùng để phân tích hành vi, gợi ý tìm kiếm hoặc cá nhân hóa.
* **Các trường dữ liệu dự kiến:**
    * `historyId`: Định danh duy nhất cho mỗi bản ghi lịch sử.
    * `userId`: Định danh của người dùng thực hiện tìm kiếm (nếu người dùng đã đăng nhập). Có thể null nếu cho phép tìm kiếm ẩn danh.
    * `queryText`: Nội dung (chuỗi ký tự) mà người dùng đã nhập vào ô tìm kiếm.
    * `filtersApplied`: Một đối tượng lưu trữ các bộ lọc đã được áp dụng trong lần tìm kiếm đó (ví dụ: `{ "category": "Laptop", "brand": "Dell", "price_min": 10000000 }`).
    * `resultCount`: Số lượng kết quả trả về cho truy vấn đó.
    * `timestamp`: Thời điểm thực hiện tìm kiếm.
    * `sessionId`: (Tùy chọn) Định danh phiên làm việc của người dùng.

**Mối quan hệ dự kiến giữa các thực thể:**
* Một `User` có thể có nhiều `SearchHistory` records.

## 5. Chiến Lược Triển Khai và Cấu Hình Hệ Thống Dự Kiến

### a. Chiến lược triển khai (Ý tưởng ban đầu)

* **Môi trường Phát triển (Local):**
    * **Công cụ chính:** **Docker** và **Docker Compose**.
    * **Mục tiêu:** Tạo một môi trường phát triển nhất quán, dễ dàng khởi tạo và tái tạo toàn bộ hệ thống (bao gồm Ứng dụng Web Chính, MongoDB, cụm Elasticsearch, Kibana, Logstash) trên máy cá nhân của bạn.
    * **Cách thực hiện:**
        1.  Viết `Dockerfile` cho Ứng dụng Web Chính (Node.js) để đóng gói ứng dụng và các dependencies của nó.
        2.  Tạo file `docker-compose.yml` để định nghĩa tất cả các dịch vụ (services):
            * `webapp`: Dịch vụ cho Ứng dụng Web Chính, build từ Dockerfile.
            * `mongo`: Dịch vụ cho MongoDB, sử dụng image chính thức từ Docker Hub.
            * `elasticsearch_nodes` (ví dụ: `es01`, `es02`, `es03`...): Các dịch vụ cho từng node Elasticsearch, sử dụng image chính thức.
            * `kibana`: Dịch vụ cho Kibana, sử dụng image chính thức.
            * `logstash`: Dịch vụ cho Logstash, sử dụng image chính thức.
        3.  Trong `docker-compose.yml`, định nghĩa:
            * Mạng (networks) để các container có thể giao tiếp với nhau.
            * Volumes để lưu trữ dữ liệu persistent cho MongoDB và Elasticsearch, tránh mất dữ liệu khi container bị xóa hoặc khởi động lại.
            * Ports được expose ra ngoài để bạn có thể truy cập ứng dụng và các công cụ từ trình duyệt.
            * Biến môi trường cần thiết cho từng dịch vụ.
        4.  Sử dụng file `.env` (được `docker-compose.yml` tham chiếu) để lưu trữ các cấu hình nhạy cảm hoặc thay đổi thường xuyên cho Ứng dụng Web Chính.
        5.  Chạy lệnh `docker-compose up --build -d` để build image (nếu cần) và khởi chạy tất cả các container ở chế độ detached.
* **Môi trường Staging/Production (Định hướng tương lai, khi dự án phát triển lớn hơn):**
    * **Container Orchestration:**
        * **Kubernetes (K8s):** Là lựa chọn hàng đầu cho các hệ thống phức tạp, yêu cầu khả năng tự động scale mạnh mẽ, tự phục hồi, quản lý cấu hình và secret nâng cao. Các nhà cung cấp đám mây lớn (AWS, Google Cloud, Azure) đều có dịch vụ Kubernetes được quản lý (EKS, GKE, AKS).
        * **Docker Swarm:** Đơn giản hơn Kubernetes, tích hợp sẵn với Docker Engine. Phù hợp nếu yêu cầu không quá phức tạp và muốn một giải pháp dễ tiếp cận hơn.
    * **Managed Services (Dịch vụ được quản lý bởi nhà cung cấp đám mây):**
        * **MongoDB Atlas, Amazon DocumentDB, Azure Cosmos DB (API cho MongoDB):** Giảm tải công việc quản trị, sao lưu, vá lỗi, và mở rộng cơ sở dữ liệu MongoDB.
        * **Elastic Cloud, Amazon OpenSearch Service (trước đây là Elasticsearch Service):** Tương tự, cung cấp cụm Elasticsearch được quản lý.
    * **CI/CD Pipeline (Continuous Integration/Continuous Deployment):**
        * Sử dụng các công cụ như **GitLab CI/CD, GitHub Actions, Jenkins, CircleCI** để tự động hóa quy trình:
            1.  Push code lên Git repository.
            2.  Trigger pipeline: Build Docker image, chạy unit tests, integration tests.
            3.  Đẩy image lên một container registry (ví dụ: Docker Hub, AWS ECR, Google Artifact Registry).
            4.  Triển khai phiên bản mới lên môi trường Staging (để kiểm thử thêm) và sau đó là Production.
    * **Load Balancing:** Sử dụng các dịch vụ Load Balancer của nhà cung cấp đám mây hoặc các giải pháp như Nginx, HAProxy để phân phối lưu lượng truy cập đến nhiều instance của Ứng dụng Web Chính, tăng tính sẵn sàng và khả năng chịu tải.
    * **Monitoring & Logging:**
        * **Prometheus & Grafana:** Để thu thập metrics và trực quan hóa trạng thái hoạt động của hệ thống.
        * Sử dụng ELK Stack (Elasticsearch, Logstash, Kibana) một cách đầy đủ hơn cho việc quản lý log tập trung, hoặc các dịch vụ quản lý log của nhà cung cấp đám mây (ví dụ: AWS CloudWatch Logs, Google Cloud Logging).

### b. Cấu hình hệ thống dự kiến (Các điểm chính cần lưu ý cho từng thành phần)

* **Ứng dụng Web Chính (Node.js - thường qua file `.env` hoặc config files):**
    * `PORT`: Cổng mà ứng dụng Node.js sẽ lắng nghe (ví dụ: 3000, 8080).
    * `MONGO_URI`: Chuỗi kết nối đến MongoDB (bao gồm host, port, tên database, thông tin xác thực nếu có).
    * `ELASTICSEARCH_HOSTS`: Danh sách các địa chỉ (host:port) của các node Elasticsearch trong cụm mà ứng dụng sẽ kết nối đến.
    * `JWT_SECRET`: Khóa bí mật dùng để ký và xác minh JSON Web Tokens.
    * `SESSION_SECRET` (nếu dùng session): Khóa bí mật cho việc mã hóa session ID.
    * `LOG_LEVEL`: Mức độ chi tiết của log (ví dụ: 'debug', 'info', 'warn', 'error').
    * `CLIENT_URL` (nếu có frontend riêng): URL của ứng dụng frontend để cấu hình CORS.
    * Các cấu hình khác liên quan đến giới hạn request, thời gian timeout, v.v.
* **Elasticsearch (thường qua file `elasticsearch.yml` cho mỗi node, hoặc biến môi trường trong Docker):**
    * `cluster.name`: Tên của cụm Elasticsearch (tất cả các node trong cụm phải có cùng tên này).
    * `node.name`: Tên duy nhất cho từng node (ví dụ: `es01`, `es02`).
    * `node.roles`: Vai trò của node (ví dụ: `master`, `data`, `ingest`, `ml`). Một node có thể có nhiều vai trò.
    * `network.host`: Địa chỉ IP mà node sẽ lắng nghe (ví dụ: `0.0.0.0` để chấp nhận kết nối từ bất kỳ đâu trong mạng Docker nội bộ, hoặc địa chỉ IP cụ thể trong production).
    * `http.port`: Cổng HTTP cho API (mặc định 9200).
    * `transport.port`: Cổng cho giao tiếp nội bộ giữa các node (mặc định 9300).
    * `discovery.seed_hosts`: Danh sách các địa chỉ của các master-eligible node khác trong cụm để node hiện tại có thể khám phá và gia nhập cụm (ví dụ: `["es01:9300", "es02:9300"]`).
    * `cluster.initial_master_nodes`: Danh sách tên của các master-eligible node có thể được bầu làm master node khi cụm khởi tạo lần đầu (ví dụ: `["es01", "es02"]`).
    * `ES_JAVA_OPTS`: (Biến môi trường) Để cấu hình heap size cho JVM của Elasticsearch (rất quan trọng, ví dụ: `-Xms1g -Xmx1g` cho heap 1GB, thường nên đặt bằng 50% RAM của node, nhưng không quá 30-32GB).
    * `path.data`: Đường dẫn đến thư mục lưu trữ dữ liệu của node.
    * `path.logs`: Đường dẫn đến thư mục lưu trữ log của node.
    * Cấu hình số lượng primary shards và replica shards mặc định cho các index mới (có thể ghi đè khi tạo index).
* **MongoDB (thường qua biến môi trường trong Docker hoặc file config của MongoDB `mongod.conf`):**
    * `MONGO_INITDB_ROOT_USERNAME`, `MONGO_INITDB_ROOT_PASSWORD`: Để tạo tài khoản root khi MongoDB khởi tạo lần đầu (chỉ có tác dụng khi `data/db` trống).
    * `bindIp`: Địa chỉ IP mà MongoDB lắng nghe (ví dụ: `0.0.0.0` để chấp nhận kết nối từ mọi nơi trong mạng Docker).
    * `port`: Cổng MongoDB lắng nghe (mặc định 27017).
    * `dbpath`: Đường dẫn đến thư mục lưu trữ dữ liệu.
    * Cấu hình về journaling, wiredTiger cache size, v.v.
* **Kibana (thường qua file `kibana.yml` hoặc biến môi trường trong Docker):**
    * `server.port`: Cổng mà Kibana sẽ chạy (mặc định 5601).
    * `server.host`: Địa chỉ IP mà Kibana lắng nghe (ví dụ: `0.0.0.0`).
    * `server.name`: Tên của instance Kibana.
    * `elasticsearch.hosts`: Danh sách các URL của cụm Elasticsearch mà Kibana sẽ kết nối đến (ví dụ: `["http://es01:9200", "http://es02:9200"]`).
    * `elasticsearch.username`, `elasticsearch.password` (nếu Elasticsearch có bật bảo mật).
    * `logging.dest`: Nơi ghi log của Kibana (ví dụ: `stdout`).
* **Logstash (thường qua file pipeline config, ví dụ `logstash.conf`, và file `logstash.yml` hoặc biến môi trường):**
    * **Pipeline config (`*.conf`):**
        * `input { ... }`: Định nghĩa nguồn đầu vào (ví dụ: `beats { port => 5044 }`, `mongodb { uri => "mongodb://user:pass@mongo:27017/mydb" collection => "products" ... }`).
        * `filter { ... }`: Định nghĩa các bước xử lý, chuyển đổi dữ liệu (ví dụ: `grok { ... }`, `mutate { ... }`, `json { ... }`, `date { ... }`).
        * `output { ... }`: Định nghĩa đích đầu ra (ví dụ: `elasticsearch { hosts => ["es01:9200"] index => "my-app-logs-%{+YYYY.MM.dd}" user => "elastic" password => "changeme" }`, `stdout { codec => rubydebug }`).
    * **`logstash.yml` (cấu hình Logstash):**
        * `http.host`: Địa chỉ IP Logstash lắng nghe cho API (nếu bật).
        * `http.port`: Cổng Logstash API (9600-9700).
        * `path.data`: Đường dẫn lưu trữ dữ liệu nội bộ của Logstash.
        * `pipeline.workers`: Số lượng worker thread cho pipeline.
        * `pipeline.batch.size`: Kích thước batch xử lý.
    * **`jvm.options` (cấu hình JVM cho Logstash):**
        * `-Xms...`, `-Xmx...`: Cấu hình heap size cho JVM của Logstash.

## Lộ Trình Phát Triển Dự Kiến (Gợi ý các giai đoạn chính)

1.  **Giai đoạn 1: Xây dựng nền tảng và chức năng tìm kiếm cốt lõi (MVP - Minimum Viable Product)**
    * **Mục tiêu:** Tạo ra một phiên bản hoạt động được với các chức năng cơ bản nhất.
    * **Công việc chính:**
        * Thiết lập môi trường Docker Compose với Ứng dụng Web (Node.js/Express.js), MongoDB (1 instance), và Elasticsearch (1 node ban đầu).
        * Phác thảo và triển khai schema cơ bản cho `User` và `Product` models trong MongoDB sử dụng Mongoose.
        * Xây dựng API xác thực người dùng cơ bản (đăng ký, đăng nhập bằng JWT).
        * Xây dựng API CRUD (Create, Read, Update, Delete) cơ bản cho sản phẩm (chỉ dành cho admin).
        * Thiết lập cơ chế đồng bộ dữ liệu sản phẩm ban đầu từ MongoDB sang Elasticsearch (có thể dùng `mongoosastic` hoặc một script đồng bộ đơn giản).
        * Xây dựng API tìm kiếm sản phẩm cơ bản (dựa trên `match` query trên tên và mô tả sản phẩm).
        * Tạo giao diện người dùng (frontend) ở mức tối thiểu để người dùng có thể xem danh sách sản phẩm và thực hiện tìm kiếm đơn giản.
2.  **Giai đoạn 2: Mở rộng cụm Elasticsearch và nâng cao tính năng tìm kiếm**
    * **Mục tiêu:** Tăng cường khả năng chịu lỗi, hiệu năng tìm kiếm và bổ sung các tính năng tìm kiếm hữu ích hơn.
    * **Công việc chính:**
        * Mở rộng cụm Elasticsearch lên nhiều node (ví dụ: 3 nodes - 1 master-eligible, 2 data nodes hoặc cả 3 đều master-eligible và data).
        * Cấu hình chi tiết hơn về sharding (ví dụ: quyết định số lượng primary shards cho index `products`) và replication (ví dụ: `number_of_replicas: 1` hoặc `2` tùy số lượng data node).
        * Bổ sung các tính năng tìm kiếm nâng cao:
            * Tìm kiếm trên nhiều trường (`multi_match`).
            * Lọc kết quả theo thuộc tính (category, brand, price range, attributes) sử dụng `term`, `terms`, `range` queries.
            * Sắp xếp kết quả theo relevancy, giá, ngày tạo, v.v.
            * Tìm kiếm mờ (fuzzy search) để xử lý lỗi chính tả.
            * Gợi ý tự động (autocomplete/suggestions) khi người dùng gõ vào ô tìm kiếm.
        * Bắt đầu lưu trữ lịch sử tìm kiếm của người dùng vào MongoDB.
        * Cải thiện giao diện tìm kiếm để hỗ trợ các bộ lọc và sắp xếp.
3.  **Giai đoạn 3: Hoàn thiện giao diện người dùng và chức năng quản trị**
    * **Mục tiêu:** Cung cấp trải nghiệm tốt hơn cho cả người dùng cuối và quản trị viên.
    * **Công việc chính:**
        * Phát triển giao diện quản trị viên đầy đủ hơn: quản lý sản phẩm (thêm, sửa, xóa, quản lý hình ảnh, thuộc tính), quản lý người dùng (xem danh sách, phân quyền), xem thống kê cơ bản về tìm kiếm, sản phẩm.
        * Tích hợp Kibana để theo dõi và phân tích dữ liệu Elasticsearch: tạo các dashboard hữu ích (ví dụ: các truy vấn tìm kiếm phổ biến, sản phẩm được xem nhiều, hiệu suất tìm kiếm).
        * Cân nhắc tích hợp Logstash để thu thập và quản lý log ứng dụng một cách tập trung, gửi log vào Elasticsearch để phân tích qua Kibana.
        * Cải thiện UI/UX cho trang người dùng: giao diện hiển thị chi tiết sản phẩm tốt hơn, quy trình đặt hàng (nếu có), quản lý tài khoản cá nhân.
        * Thêm tính năng đánh giá sản phẩm.
4.  **Giai đoạn 4: Tối ưu hóa, kiểm thử và chuẩn bị cho môi trường thực tế**
    * **Mục tiêu:** Đảm bảo hệ thống ổn định, bảo mật và sẵn sàng cho việc triển khai.
    * **Công việc chính:**
        * Tập trung vào việc tối ưu hóa hiệu năng:
            * Tối ưu hóa các truy vấn Elasticsearch (sử dụng Explain API, điều chỉnh mapping, query DSL).
            * Tối ưu hóa truy vấn MongoDB (sử dụng index phù hợp).
            * Caching (nếu cần thiết, ví dụ: cache kết quả tìm kiếm phổ biến).
        * Viết unit tests, integration tests cho cả backend và frontend.
        * Thực hiện kiểm thử tải (load testing) để đánh giá khả năng chịu tải của hệ thống và xác định điểm nghẽn cổ chai.
        * Kiểm thử bảo mật (security testing): rà soát các lỗ hổng tiềm ẩn (XSS, SQL Injection - mặc dù ít hơn với NoSQL, CSRF, vấn đề về JWT).
        * Hoàn thiện tài liệu kỹ thuật (kiến trúc, API, hướng dẫn cài đặt) và hướng dẫn sử dụng.
        * Lên kế hoạch chi tiết cho việc triển khai lên môi trường staging (để kiểm thử cuối cùng) và production (sử dụng các công cụ và chiến lược đã đề cập ở mục 5.a).
        * Thiết lập quy trình sao lưu và phục hồi dữ liệu cho MongoDB và Elasticsearch.

Đây là bản phác thảo kế hoạch và những ý tưởng ban đầu cho dự án. Các chi tiết sẽ được làm rõ và điều chỉnh trong quá trình phân tích sâu hơn và khi bắt đầu giai đoạn thiết kế cụ thể.

# 🚀 serverText

<p align="center">
  <img src="https://raw.githubusercontent.com/congtuhaixinhzai/serverText/main/logo.png" width="140" alt="serverText Logo"/>
</p>

<p align="center">
  <img src="https://img.shields.io/github/stars/congtuhaixinhzai/serverText?style=for-the-badge"/>
  <img src="https://img.shields.io/github/forks/congtuhaixinhzai/serverText?style=for-the-badge"/>
  <img src="https://img.shields.io/github/license/congtuhaixinhzai/serverText?style=for-the-badge"/>
  <img src="https://img.shields.io/github/last-commit/congtuhaixinhzai/serverText?style=for-the-badge"/>
</p>

---

# Tài Liệu Phân Tích Mã Nguồn Server Avatar (Phiên Bản Nâng Cao - Tác Giả: Hải)

Tài liệu này cung cấp một cái nhìn chi tiết, toàn diện và sâu sắc từ nhỏ đến lớn về cấu trúc thư mục, kiến trúc hệ thống, các lớp xử lý và cơ chế hoạt động của mã nguồn **Server Avatar** nằm tại thư mục `H:\DATA\SEVER_AVATAR`. 

Đây là một phiên bản máy chủ Avatar 2D được phát triển nâng cao bởi tác giả **Hải**, tích hợp các cơ chế nhập vai RPG (đánh Boss, điểm tiềm năng, khảm đá quý) cùng hệ thống đấu trường PvP, cờ tướng và bảng điều khiển quản trị trực quan (GUI).

---

## 1. Tổng Quan Công Nghệ và Kiến Trúc Hệ Thống

Dự án được xây dựng trên một kiến trúc máy chủ game hiệu năng cao, tối ưu hóa cho hàng ngàn kết nối đồng thời và quản trị trực quan.

*   **Ngôn ngữ lập trình:** Java 11 / Java 19 (Maven quản lý dự án).
*   **Kết nối mạng (Network):** Sử dụng **TCP Socket** truyền thống kết hợp với mô hình đa luồng (`ExecutorService`). Dữ liệu truyền tải dưới dạng gói tin nhị phân (**binary packet**), giúp giảm thiểu tối đa băng thông và độ trễ truyền dữ liệu.
*   **Cơ sở dữ liệu (Database):**
    *   **MySQL:** Lưu trữ thông tin tài khoản, người chơi, bang hội, rương đồ, lịch sử giao dịch và bảng xếp hạng.
    *   **HikariCP:** Connection Pool hiệu năng cao giúp quản lý kết nối MySQL tối ưu, chống nghẽn luồng.
    *   **Redis (Jedis):** Sử dụng làm caching và đồng bộ các dữ liệu truy cập nhanh.
*   **Bộ nhớ đệm (Caching):** Tích hợp **Caffeine Cache** để lưu trữ đệm các dữ liệu cấu hình tĩnh và dữ liệu động truy cập thường xuyên.
*   **Tường lửa (Firewall):** Tích hợp tường lửa ứng dụng (`GameFirewall.java`) để theo dõi IP, giới hạn số kết nối clone và tự động chặn các hành vi tấn công DDoS.
*   **Hệ thống QR Code:** Sử dụng **VietQR** kết hợp với thư viện **ZXing** để tạo chuỗi và mã hình QR thanh toán ngân hàng tự động hiển thị trong game cho người chơi quét ứng dụng ngân hàng nạp tiền.
*   **Cổng API Quản trị (ToolServer):** Cổng socket độc lập bảo mật bằng mã khóa (`tool.key`) để quản lý server từ xa.
*   **Giao diện Admin (GUI):** Tích hợp giao diện quản trị Swing (`AdminGUI.java`) hiện đại với chế độ Dark Mode/Light Mode để cấu hình trực quan tỉ lệ game, chi phí và quản lý người chơi.

---

## 2. Phân Tích Chi Tiết Chức Năng Từ Nhỏ Đến Lớn (5 Lớp Kiến Trúc)

Mã nguồn được phân tách rõ ràng thành 5 tầng kiến trúc liên kết chặt chẽ:

### Lớp 1: Hệ Thống Kết Nối & Mạng (Hạ tầng mạng)
*   **`Session.java`:** Quản lý vòng đời kết nối của từng Client. Thực hiện đọc dữ liệu nhị phân từ socket và đẩy vào hàng đợi xử lý, đồng thời gửi phản hồi về client.
*   **`Message.java`:** Đại diện cho một gói tin nhị phân trong game, bao gồm mã lệnh (`command`), luồng đọc ghi dữ liệu (`DataInputStream`/`DataOutputStream`) và mảng byte dữ liệu thô.
*   **`GameFirewall.java`:** Lớp bảo mật lọc các địa chỉ IP kết nối, giới hạn tối đa số kết nối từ cùng một IP và quản lý danh sách IP Whitelist.
*   **`GameScheduler.java`:** Lập lịch các tác vụ chạy ngầm định kỳ trong hệ thống.

### Lớp 2: Quản Lý Dữ Liệu & Đồng Bộ (Database & Caching)
*   **`DbManager.java`:** Quản lý kết nối MySQL thông qua HikariCP. Thực thi các truy vấn SQL bằng PreparedStatement để chống tấn công SQL Injection.
*   **`AutoSaveManager.java`:** Tự động gọi hàm lưu dữ liệu người chơi định kỳ xuống database để đề phòng mất mát dữ liệu do sự cố.
*   **`OnlineManager.java`:** Quản lý danh sách phiên làm việc (Session) và người chơi đang thực sự online trong game.
*   **`GameDataManager.java` & `ItemManager.java`:** Tải toàn bộ tài nguyên game (vật phẩm, trang phục, chỉ số, dữ liệu quay số...) từ database lên RAM khi khởi chạy máy chủ.
*   **`GameConfig.java`:** Đọc và ghi các tham số cấu hình hệ thống (tỷ lệ đập đồ, tỷ lệ câu cá, tỷ lệ casino, giá cả...) và hỗ trợ đồng bộ thời gian thực với bảng điều khiển AdminGUI.

### Lớp 3: Mô Hình Đối Tượng (Model)
*   **`User.java`:** Đối tượng trung tâm đại diện cho người chơi, lưu trữ:
    *   Chỉ số tương tác: Thân thiện, Nghịch ngợm, Sành điệu, Vui vẻ, Đói.
    *   Tiền tệ: Xu, Xu ngân hàng, Lượng, Lượng khóa, Xèng (chơi Casino).
    *   Trang phục đang mặc (`wearing`), rương đồ cá nhân (`chests`), rương nhà (`chestsHome`), kho thuê (`warehouseItems`) kèm mật khẩu hòm đồ.
    *   Tiến trình thần thú: Ấp trứng rồng, luyện rồng ngũ hành, giải cứu kỳ lân.
    *   *Thuộc tính RPG mới:* Cấp cường hóa nông trại (`enhanceLevel`), kho phụ nông sản (`auxiliaryStorage`), điểm tiềm năng (`potential`).
*   **`Potential.java`:** Mô hình lưu trữ cấp độ tiềm năng, kinh nghiệm tiềm năng và các điểm cộng chỉ số chiến đấu: Sức đánh (Damage), Máu (HP), Năng lượng (Mana), Giáp (Armor).
*   **`Item.java` / `Part.java`:** Đối tượng trang bị và vật phẩm game, định nghĩa giới tính sử dụng, độ bền, hạn sử dụng và cấp độ nâng cấp.
*   **`Boss.java` / `BossType.java`:** Đối tượng Boss thế giới kế thừa từ `Npc`, quản lý HP, sát thương boss, kỹ năng tấn công diện rộng và tọa độ di chuyển.
*   **`SoloArena.java`:** Mô hình quản lý trận đấu PvP 1v1 giữa hai người chơi tại đấu trường.

### Lớp 4: Xử Lý Logic Game (Handler & Service)
*   **`MessageHandler.java`:** Nhận packet nhị phân từ `Session`, phân tích mã lệnh `Cmd` và chuyển tiếp đến các handler xử lý logic tương ứng.
*   **`GlobalHandler.java`:** Xử lý các logic cốt lõi như chat, di chuyển bản đồ, tương tác cảm xúc, kết bạn, giao dịch, xin tiền ăn xin.
*   **`NpcHandler.java`:** Class cực kỳ lớn (hơn 200KB) xử lý toàn bộ các hội thoại và menu lựa chọn khi người chơi tương tác với tất cả NPC trong game.
*   **`FarmMsgHandler.java` / `FarmService.java`:** Xử lý các hành động gieo hạt, bón phân, tưới nước, chữa bệnh, thu hoạch nông sản, chăm sóc vật nuôi và nâng cấp đất đai.
*   **`SoloArenaManager.java`:** Quản lý hàng đợi đăng ký thách đấu PvP, tự động xếp cặp đấu và bắt đầu trận đấu tại Map 34.
*   **`KhamDaService.java`:** Xử lý logic khảm các loại đá quý vào trang phục để gia tăng chỉ số chiến đấu cho nhân vật.
*   **`UnicornRescueService.java` & `DragonHatchingService.java`:** Quản lý logic nhiệm vụ giải cứu kỳ lân cổ tích và ấp trứng rồng tí nị hàng ngày.
*   **`TienLenService.java` / `PhomService.java` / `BauCuaService.java` / `DiamondService.java`:** Hiện thực luật chơi và các gói tin tương tác của các mini game Casino.
*   **`VietQR.java` / `Bank.java`:** Xây dựng chuỗi dữ liệu VietQR thông minh chuyển đổi động nội dung nạp tiền và số tiền để hiển thị trực tiếp cho người chơi.

### Lớp 5: Bản Đồ & Quản Lý Phân Khu (Map & Zone)
*   **`MapManager.java` & `Map.java`:** Quản lý 60 bản đồ trong game (ID từ 0 đến 59) bao gồm các khu vực như Công viên, Khu dân cư, Khu sinh thái, Nông trại, Đấu trường, Khu mua sắm...
*   **`Zone.java`:** Chia nhỏ mỗi bản đồ thành nhiều khu vực (mặc định 20 khu vực, các bản đồ trung tâm có 40 khu vực). Giới hạn tối đa 20 người chơi mỗi khu vực để chống lag. Đồng bộ hóa vị trí, hiển thị hành động di chuyển, chat và hiệu ứng của tất cả thực thể trong cùng một khu vực.

---

## 3. Các Chức Năng Gameplay Nổi Bật

### A. Hệ Thống Tiềm Năng & Nhập Vai RPG (Mới & Độc đáo)
Hệ thống tích hợp các cơ chế RPG giúp gia tăng sức mạnh nhân vật phục vụ việc săn Boss thế giới:
*   **Cấp Tiềm Năng:** Người chơi tích lũy EXP tiềm năng thông qua các hoạt động trong game (ví dụ tiêu diệt Boss, làm nhiệm vụ...). Khi lên cấp sẽ được tặng 5 điểm tiềm năng.
*   **Phân bổ điểm:** Người chơi có thể tự do cộng điểm tiềm năng vào:
    *   *Sức đánh (Damage):* Tăng sát thương cơ bản khi tấn công Boss.
    *   *HP (Máu tối đa):* Tăng giới hạn sinh mệnh.
    *   *Mana (Năng lượng):* Tăng giới hạn năng lượng sử dụng kỹ năng.
    *   *Giáp (Armor):* Tăng phòng thủ giúp giảm sát thương nhận vào từ Boss.

### B. Hệ Thống Khảm Đá Trang Bị (Mới)
*   Cho phép người chơi khảm các loại đá quý vào trang phục theo phân loại bộ phận (`zOrder`).
*   Khảm thành công tiêu tốn đá quý + xu, cộng thêm các chỉ số ngẫu nhiên cho trang bị: *Sát thương đánh Boss, giảm thời gian đào mỏ, tỉ lệ đánh trúng, tỉ lệ chí mạng (Crit) và sát thương chí mạng*.
*   Hệ thống tự động xếp hạng **Top 10 Khảm Đá** dựa trên tổng điểm khảm đá của người chơi.

### C. Boss Thế Giới & Hộp Quà Khung Giờ Vàng (Mới)
*   Boss thế giới xuất hiện trên các bản đồ quy định, tự động di chuyển ngẫu nhiên và liên tục quét tấn công tất cả người chơi đứng gần bằng kỹ năng diện rộng.
*   Người chơi tiêu diệt Boss sẽ nhận được rương phần thưởng giá trị và EXP tiềm năng.
*   **Sự kiện Rơi Quà:** Khi Boss bị tiêu diệt vào các khung giờ vàng quy định (10:00 - 14:00 và 19:00 - 23:00), hệ thống sẽ tự động tạo ra 4 hộp quà (`GiftBox`) xung quanh vị trí Boss chết để người chơi nhặt vật phẩm khuyến khích.

### D. Đấu Trường PvP Solo Arena (Mới)
*   Người chơi đến Map 34 gặp NPC Đấu Trường (Solo Arena - ID 911) để đăng ký vào danh sách chờ đấu.
*   Hệ thống `SoloArenaManager` sẽ tự động ghép cặp những người chơi đang chờ để đưa vào phòng PvP thách đấu trực tiếp sức mạnh tiềm năng của nhau.

### E. Cường Hóa Nông Trại & Kho Phụ (Mới)
*   Khi người chơi sở hữu nông trại đạt cấp độ cao (48 ô đất, chuồng cấp 10, ao cấp 9), họ có thể tiến hành **Cường hóa nông trại** tại NPC Lái Buôn với chi phí 50k xu và các loại nông sản.
*   Cường hóa thành công giúp mở khóa **Kho phụ nông sản** (`Auxiliary Storage`) có sức chứa lớn và tăng tỷ lệ thu hoạch thêm nông sản khi làm vườn.

### F. Hệ Thống Nhiệm Vụ Tiên Sử Cốt Truyện (Mới)
*   Tích hợp hệ thống nhiệm vụ cốt truyện Tiên Sử kéo dài 55 cấp độ nhiệm vụ (`nhiem_vu_tien_su_1_55.txt`). Người chơi thực hiện các yêu cầu của NPC Thị Trưởng để mở khóa cốt truyện và nhận các phần thưởng giá trị.

### G. Hệ Thống Chăn Nuôi & Nông Trại Cơ Bản
Hệ thống nông trại được thiết kế chi tiết với cơ chế nâng cấp đa dạng và mở rộng tối đa không gian phát triển:
*   **Hệ thống Đất Trồng (Tối đa 48 ô đất):** 
    *   Người chơi khởi đầu bằng việc mua các ô đất thường để gieo hạt. Chi phí mở rộng đất tăng dần theo từng ô (từ 10.800 xu đến 662.700 xu).
    *   **Nâng cấp Đất Đỏ (Level 2):** Cải tạo đất thường thành đất đỏ để giảm thời gian trồng cây và tăng năng suất. Chi phí từ 1.9 triệu đến 7.3 triệu xu mỗi ô.
    *   **Nâng cấp Đất Vàng (Level 3):** Cấp bậc đất cao nhất, tối ưu hoá lợi nhuận cực đại. Chi phí rất đắt đỏ (từ 11.2 triệu đến gần 25 triệu xu mỗi ô).
*   **Hệ thống Chuồng Trại & Vật Nuôi:** 
    *   Trại gia súc (heo, bò, gà, vịt, chó canh trộm) có thể nâng cấp tối đa 10 cấp độ (`Constants.FARM_SIZE`). Ở cấp cao nhất, chuồng có thể nuôi đến 20 con vật. Chi phí nâng cấp tốn hàng triệu xu.
    *   Ao cá có thể nâng cấp lên đến 10 cấp (`Constants.FISH_SIZE`), chứa tối đa 10 con cá. Chi phí nâng cấp ao cá trải dài từ 50k xu đến 4 triệu xu.
*   **Cây Khế Tài Lộc (Tối đa Cấp 50):** 
    *   Một cây khế mặc định có ở góc vườn, người chơi có thể dùng xu để nâng cấp cây. Cấp bậc càng cao, số lượng khế thu hoạch được càng lớn (tối đa 600 quả ở cấp 50). Chi phí nâng cấp khế cũng tăng tiến liên tục.
*   **Nấu Ăn & Chế Biến:** Sử dụng sản phẩm thu hoạch (trứng, sữa...) để chế biến thành các món ăn (Bánh trứng, cơm trứng, nước ép...) phục vụ cho việc sử dụng (hồi điểm đói/sức khỏe) hoặc làm nhiệm vụ.
*   **Cơ Chế Canh Gác & Trộm Cắp:** Người chơi có thể mua Chó để canh chừng nông trại. Khi người chơi khác sang trộm nông sản, Chó sẽ bảo vệ tỉ lệ nhất định.

### H. Trò Chơi Trực Tuyến Casino & Cờ Tướng
*   Chơi cờ tướng đối kháng trực tiếp thông qua lớp xử lý luật chơi cờ tướng (`XiangqiRules.java`).
*   Các game bài Casino truyền thống: Tiến Lên Miền Nam, Tá Lả (Phỏm), Bầu Cua Tôm Cá và mini game xếp kim cương đối kháng.

### I. Các Tính Năng Xã Hội Khác
*   **Hôn nhân & Gia đình:** Kết hôn, làm nhiệm vụ hẹn hò hàng ngày, sở hữu căn hộ đôi.
*   **Học tập với Cô Giáo:** Tham gia trả lời câu hỏi đố vui tiếng Anh/tiếng Việt để tích lũy điểm chuyên cần đổi quà.
*   **Đào mỏ & Câu cá:** Mua xẻng đào mỏ tìm đá quý hoặc mua cần câu ra khu sinh thái câu cá bán lấy xu.
*   **Quay số & Đấu giá:** Vòng quay may mắn nhận trang phục vĩnh viễn/ngày, đấu giá ngược vật phẩm hiếm.

---

## 4. Chức Năng Dành Riêng Cho ADMIN & Ban Quản Trị

Server cung cấp ba kênh quản trị vô cùng mạnh mẽ và toàn diện:

### A. Giao Diện Quản Trị Trực Quan (AdminGUI.java - Swing GUI)
Một bảng điều khiển bằng giao diện đồ họa tự động khởi chạy khi mở Server (ở các hệ điều hành hỗ trợ giao diện):
*   **Đăng nhập xác thực:** Có dialog đăng nhập bảo mật bằng mật khẩu Admin, trang trí hiệu ứng chòm sao Cyberpunk và hạt ánh sáng lấp lánh có chức năng rung lắc cảnh báo khi nhập sai mật khẩu.
*   **Chế độ màu:** Hỗ trợ giao diện tối màu (**Dark Mode**) và sáng màu (**Light Mode**).
*   **Điều chỉnh trực quan:** Admin có thể cấu hình trực tiếp thông qua các bảng nhập số (Spinner):
    *   *Câu cá:* Tỷ lệ câu dính cá, tỷ lệ câu dính rác, giá bán cá.
    *   *Quay số:* Tỷ lệ trúng vật phẩm vĩnh viễn, tỷ lệ ra xu/exp, giá quay số bằng xu/lượng.
    *   *Đập đồ:* Tỷ lệ đập đồ thành công từ cấp +1 đến +15, số đá cần thiết.
    *   *Khảm đá:* Cấu hình chi phí xu cho mỗi lần khảm đá các loại (sức đánh boss, thời gian đào, tỉ lệ chí mạng...).
    *   *Thế giới:* Thay đổi thời tiết, bật/tắt bảo trì game, quản lý danh sách người chơi online.

### B. Công Cụ Điều Khiển Từ Xa (ToolServer)
Lắng nghe kết nối TCP trên cổng `tool.port` (mặc định `19129`) và yêu cầu khóa `tool.key` để thực thi lệnh. Đây là công cụ đắc lực để Admin quản lý server từ xa qua trang web quản trị hoặc bot telegram:

| Mã Lệnh | Tham Số | Chức Năng Chi Tiết |
| :--- | :--- | :--- |
| `ON_GAME` | `0` / `1` / `2` | Bật hoặc tắt trạng thái bảo trì phân khu OnGame/Casino hoặc bảo trì toàn bộ máy chủ. |
| `WEATHER` | `0` đến `3` | Thay đổi thời tiết tức thì toàn server (Mưa, Tuyết nhỏ, Bình thường, Tuyết lớn). |
| `NOTIFY` | `Tên_User\|Loại\|Nội_dung` | Gửi thông báo chữ chạy hoặc hộp thoại bắt buộc đến một người chơi cụ thể hoặc toàn bộ server (sử dụng từ khóa `___`). |
| `KICK` | `Tên_User` | Trục xuất người chơi ra khỏi game ngay lập tức. |
| `REMAND` | `Tên_User\|Thời_gian\|Lý_do` | **Tống giam người chơi vào nhà tù** (Bản đồ tù giam - ID 18) có thời hạn (phút) kèm lý do rõ ràng. Hệ thống tự động dịch chuyển và khóa chat/di chuyển. |
| `UNREMAND` | `Tên_User` | **Phóng thích/Thả tự do** người chơi khỏi tù trước thời hạn. |
| `BAN` | `Tên_User\|Thời_gian\|Lý_do` | **Khóa tài khoản** người chơi có thời hạn hoặc vĩnh viễn (Permanent) và kick ra khỏi game ngay lập tức. |
| `UNBAN` | `Tên_User` | **Mở khóa tài khoản** bị khóa để người chơi có thể đăng nhập lại. |
| `ADD_XU` / `ADD_LUONG` | `Tên_User\|Số_tiền` | Tặng tiền Xu hoặc Lượng trực tiếp vào tài khoản người chơi (hỗ trợ cộng tiền offline thông qua database). |
| `RECHARGE` | `Tên_User\|Loại\|Số_tiền` | Cộng tiền ủng hộ thủ công (loại 1: xu, loại 2: lượng) quy đổi theo tỷ giá nạp và lưu lịch sử giao dịch. |
| `PROCESS_TRANSACTION` | `User\|Mã_Giao_Dịch\|Số_tiền` | **Duyệt giao dịch tự động:** Chuyển trạng thái giao dịch nạp tiền từ `PENDING` thành `SUCCESS`. Cộng xu/lượng và trao tặng kèm các vật phẩm khuyến mãi của gói nạp tương thích với giới tính của người chơi. |
| `CREATE_GIFTCODE` | `Mã\|Tin_nhắn\|Xu\|Lượng\|Vật_phẩm\|Lượt_dùng\|Số_lượng\|Bắt_đầu\|Kết_thúc` | Tạo mã quà tặng (Giftcode) mới với đầy đủ cấu hình hạn sử dụng và lượt giới hạn. |
| `UPDATE_GIFTCODE` | `Mã\|Tin_nhắn\|Xu\|Lượng\|Vật_phẩm\|Lượt_dùng\|...` | Cập nhật cấu hình của mã Giftcode đang tồn tại trong hệ thống. |
| `SEND_ITEMS` | `Tên_User\|Mã_vật_phẩm:Số_lượng:Hạn_dùng` | Gửi trang bị trực tiếp vào rương đồ của danh sách người chơi (tự động gộp vật phẩm tiêu hao và lọc giới tính). |

### C. Menu Quản Trị Trực Tiếp Trong Game (NPC Admin)
Khi các nhân vật có phân quyền Admin/Moderator tương tác với NPC Admin (ID 1), hệ thống sẽ mở các tùy chọn:
*   **Bảng xếp hạng Donate:** Xem danh sách Top 20 người chơi ủng hộ nhiều nhất hệ thống để trao danh hiệu.
*   **Quản lý VIP:** Tra cứu điểm tích lũy VIP của người chơi và phân phối các gói quà VIP tương ứng.

---

## 5. Tóm Tắt & Đánh Giá Tổng Quan Mã Nguồn

*   **Tính đột phá:** Đây là một phiên bản game Avatar 2D cực kỳ độc đáo và hiếm có khi kết hợp các yếu tố nhập vai RPG (Đánh Boss thế giới, Đấu trường PvP Arena, Khảm đá trang bị, Cộng điểm tiềm năng Sức đánh/HP/Giáp) vào lối chơi nông trại mạng xã hội truyền thống.
*   **Kiến trúc sạch (Clean Architecture):** Mã nguồn được phân chia mô-đun rất mạch lạc, tách biệt logic các tính năng mới giúp việc bảo trì hoặc nâng cấp độc lập diễn ra dễ dàng mà không ảnh hưởng tới gameplay cốt lõi.
*   **Tính bảo mật cao:** Hệ thống tường lửa `GameFirewall` hoạt động tốt chống spam clone kết nối; cơ chế `ShutdownHook` đảm bảo đồng bộ lưu trữ toàn bộ dữ liệu an toàn trước khi tắt máy chủ.
*   **Trải nghiệm quản trị tuyệt vời:** Việc tích hợp đồng thời cổng lệnh ToolServer và giao diện đồ họa AdminGUI Swing hiện đại giúp quản trị viên dễ dàng vận hành và cấu hình game trực quan mà không cần chỉnh sửa file properties hay database thủ công.


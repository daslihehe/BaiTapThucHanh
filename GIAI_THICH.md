# TÀI LIỆU GIẢI THÍCH MÃ NGUỒN & HƯỚNG DẪN VẤN ĐÁP
## DỰ ÁN CỔNG THÔNG TIN ĐIỆN ẢNH "Phim Mới.net" (5 TRANG HOÀN CHỈNH)

---

## PHẦN 1: SƠ ĐỒ CẤU TRÚC DỰ ÁN & LUỒNG DỮ LIỆU

### 1.1. Cấu trúc tệp tin và trách nhiệm từng phần
```text
phimmoi/
├── [5 Trang HTML chính]
│   ├── index.html            -> Trang chủ: Giới thiệu, Hero Slider, Phim mới, Thể loại, Top tuần, Phim vừa xem
│   ├── danh-sach-phim.html   -> Danh sách phim: Tìm kiếm realtime, Lọc đa tiêu chí, Đếm kết quả, Tab Xem sau
│   ├── chi-tiet-phim.html    -> Chi tiết phim: Đọc ?id= URL, Tóm tắt, Bảng Cast, Chấm sao, Trailer Modal, Phim cùng thể loại
│   ├── top-phim.html         -> Bảng xếp hạng: Bảng chuẩn ngữ nghĩa (caption, thead, tbody), Sort cột, 3 Tabs
│   └── trac-nghiem.html      -> Trắc nghiệm: 7 câu hỏi, tính điểm theo thể loại, gợi ý 3 phim phù hợp
├── css/
│   ├── style.css             -> Biến CSS (:root nền tối điện ảnh), Reset, Typography, Header, Footer, BEM Thẻ phim, Modal, Nút lên đầu
│   ├── home.css              -> Slider nổi bật, Banner gợi ý phim ngẫu nhiên, Top 5 tuần
│   ├── movies.css            -> Panel bộ lọc, Hero chi tiết, Bảng cast, Widget chấm sao
│   └── quiz.css              -> Tiến trình câu hỏi, nút lựa chọn, thẻ kết quả phân tích
├── js/
│   ├── movies-data.js        -> Biến toàn cục MOVIES_DATABASE (20 phim), MOVIE_GENRES, MOVIE_COUNTRIES
│   ├── main.js               -> Menu mobile, Nút lên đầu, Watchlist, Phim vừa xem, renderMovieCardHTML
│   ├── slider.js             -> Class MovieHeroSlider: Autoplay, Prev/Next, Dots, Pause on hover, Phím mũi tên
│   ├── search-filter.js      -> Lọc kết hợp (Keyword + Genre + Year + Country), Sắp xếp, Empty State, URL Query
│   ├── movie-detail.js       -> Đọc location.search, render dữ liệu, xử lý ID lỗi, Chấm sao, Watchlist toggle
│   └── quiz.js               -> Class MovieQuizApp: Câu hỏi, chấm điểm, chọn gu điện ảnh, đề xuất 3 phim
└── images/
    ├── logo.svg & favicon.svg
    ├── banners/ (5 banner cho Slider)
    └── posters/ (20 poster phim SVG)
```

### 1.2. Sơ đồ luồng hoạt động dữ liệu giữa các trang
```text
[js/movies-data.js] (Mảng MOVIES_DATABASE gồm 20 phim tĩnh)
        │
        ├──────────────────────────┬──────────────────────────┬──────────────────────────┐
        ▼                          ▼                          ▼                          ▼
  [index.html]            [danh-sach-phim.html]       [top-phim.html]          [trac-nghiem.html]
  - Slider 5 phim         - Lọc realtime              - Lấy Top 10             - Tính điểm gu phim
  - 8 phim mới            - Sắp xếp đa dạng           - Xếp theo cột           - Lấy 3 phim cùng gu
  - Gợi ý ngẫu nhiên      - Tab Xem sau               - 3 Tabs quốc gia
        │                          │
        └──────────────────────────┴──────────────────────────┐
                                                              ▼
                                                    [chi-tiet-phim.html?id=X]
                                                    - Đọc id từ URL
                                                    - Lưu vào "Phim vừa xem"
                                                    - Tương tác chấm điểm sao
                                                    - Thêm vào "Xem sau"
```

---

## PHẦN 2: GIẢI THÍCH CHI TIẾT TỪNG CHỨC NĂNG JAVASCRIPT

### 1. Quản lý lưu trữ an toàn (Safe LocalStorage) - `js/main.js`
- **Hàm `safeGetStorage(key, defaultValue)` & `safeSetStorage(key, value)`:**
  - *Mục đích:* Tránh lỗi Crash mã nguồn khi mở trực tiếp bằng giao thức `file://` trên một số trình duyệt kích hoạt chế độ chặn Cookie/Storage bên thứ ba.
  - *Luồng chạy:* Bọc câu lệnh `localStorage.getItem` và `localStorage.setItem` trong khối `try...catch`. Nếu xảy ra lỗi hoặc khóa chưa có dữ liệu, trả về an toàn giá trị mặc định (`defaultValue`).

### 2. Danh sách "Xem sau" (Watchlist) dùng chung - `js/main.js`
- **Hàm `getWatchlist()`, `isInWatchlist(id)`, `toggleWatchlist(id)`, `updateWatchlistBadge()`:**
  - *Mục đích:* Lưu trữ mảng các ID phim mà người xem muốn xem lại.
  - *Luồng chạy:*
    1. Khi bấm lưu phim tại trang chi tiết hoặc pop-up, gọi `toggleWatchlist(id)`.
    2. Kiểm tra nếu ID đã có trong mảng thì xóa bỏ; nếu chưa có thì thêm vào (`push`).
    3. Lưu lại mảng vào `localStorage` qua khóa `"phimmoi_watchlist"`.
    4. Gọi `updateWatchlistBadge()` để cập nhật số đếm trên toàn bộ huy hiệu ở thanh Navbar.

### 3. Slider Carousel Phim Nổi Bật - `js/slider.js`
- **Class `MovieHeroSlider`:**
  - *Mục đích:* Trình chiếu 5 slide phim nổi bật tại Trang chủ với điều khiển tự động và tương tác mượt mà.
  - *Luồng chạy:*
    1. `play()`: Thiết lập hàm `setInterval()` chạy sau mỗi khoảng thời gian `SLIDER_AUTOPLAY_INTERVAL` (mặc định 5000ms = 5 giây) để chuyển sang slide kế tiếp qua `nextSlide()`.
    2. `goToSlide(index)`: Gỡ bỏ class `.is-active` ở slide và dot hiện tại, thêm class `.is-active` cho slide và dot tại vị trí `index`.
    3. `pause()`: Khi người dùng rê chuột vào vùng slider (`mouseenter`), xóa `timer` bằng `clearInterval()` để người xem có thể đọc nội dung mà không bị trôi trang.
    4. `mouseenter` / `mouseleave`: Tự động dừng và chạy lại.
    5. Hỗ trợ phím bàn phím: Bắt sự kiện `keydown` phím `ArrowLeft` và `ArrowRight`.

### 4. Tìm kiếm realtime & Lọc đa tiêu chí - `js/search-filter.js`
- **Đối tượng `currentFilterState`:** Lưu trạng thái `{ keyword, genre, year, country, sortBy, viewTab }`.
- **Hàm `getFilteredAndSortedMovies()`:**
  - *Luồng lọc kết hợp 5 bước:*
    1. Bước 1: Kiểm tra Tab ("all" hay "watchlist"). Nếu là "watchlist", chỉ lấy phim trong mảng Xem sau.
    2. Bước 2: Lọc theo từ khóa `keyword` (chuyển chữ thường, tìm kiếm trong Tên phim, Tên gốc, Đạo diễn và Dàn diễn viên qua `Array.prototype.some()`).
    3. Bước 3: Lọc theo `genre` (nếu khác "Tất cả").
    4. Bước 4: Lọc theo `year` (so sánh bằng hoặc khoảng năm `2020-2022`, `2010-2019`, `< 2010`).
    5. Bước 5: Lọc theo `country` (nếu khác "Tất cả").
    6. Bước 6: Sắp xếp theo `sortBy` (`newest`: giảm dần theo năm, `rating-desc`: giảm dần theo điểm, `title-asc`: hàm `localeCompare(..., "vi")` sắp xếp tên tiếng Việt chuẩn chữ cái).
- **Hàm `renderMoviesList()`:**
  - Bơm chuỗi HTML vào `#moviesListGrid`.
  - Cập nhật số đếm kết quả: "Hiển thị X / 20 bộ phim".
  - Nếu số kết quả = 0: Render khối `empty-state-card` thân thiện kèm nút bấm đặt lại bộ lọc.

### 5. Trang Chi tiết Phim đọc Query URL - `js/movie-detail.js`
- **Hàm `getMovieIdFromUrl()`:**
  - Đọc chuỗi truy vấn bằng `new URLSearchParams(window.location.search).get("id")`.
  - Dùng `parseInt(..., 10)` để trích xuất số nguyên ID.
- **Xử lý ID lỗi hoặc không có:**
  - Nếu `!movieId` hoặc không tìm thấy phim trong `MOVIES_DATABASE`, gọi hàm `renderMovieNotFound()` hiển thị trang 404 thân thiện kèm nút quay về Kho phim, ngăn chặn lỗi màn hình trắng.
- **Hàm `initDetailEvents()` - Chấm điểm sao:**
  - Lắng nghe sự kiện `mouseenter` trên từng sao (1 đến 5 sao) để tô màu vàng các sao từ 1 đến vị trí con trỏ chuột.
  - Khi `click`: Lưu điểm số vào `localStorage`, đồng thời tính lại điểm trung bình hiển thị:
    $$\text{Điểm mới} = \frac{(\text{Rating cũ} \times \text{Lượt cũ}) + (\text{Sao chọn} \times 2)}{\text{Lượt cũ} + 1}$$

### 6. Trắc nghiệm gu điện ảnh - `js/quiz.js`
- **Class `MovieQuizApp`:**
  - Lưu trữ mảng 7 câu hỏi `QUIZ_QUESTIONS`.
  - Đối tượng điểm `scores`: `{ "Hành động": 0, "Tình cảm": 0, "Tâm lý": 0, "Khoa học viễn tưởng": 0, "Kinh dị": 0, "Hoạt hình": 0 }`.
  - Khi người chơi chọn 1 đáp án: Cộng 1 điểm vào thể loại tương ứng, cập nhật thanh tiến trình `quiz-progress-fill` theo công thức `(câu_hiện_tại / tổng_số_câu) * 100%`.
  - Khi kết thúc câu 7: Duyệt qua `scores` để tìm thể loại đạt điểm cao nhất, hiển thị bài phân tích tính cách tương ứng từ `GENRE_PROFILES` và tự động lọc lấy 3 phim thuộc thể loại đó từ `MOVIES_DATABASE` để đề xuất.

---

## PHẦN 3: BẢNG TRA CỨU "MUỐN SỬA X THÌ SỬA Ở ĐÂU?"

Bảng tra cứu nhanh giúp nhóm tự tin chỉnh sửa trực tiếp khi giảng viên yêu cầu thay đổi mã nguồn trong buổi bảo vệ:

| Yêu cầu cần sửa | Tệp tin (File) | Vị trí cụ thể / Dòng code | Hướng dẫn cách chỉnh sửa |
| :--- | :--- | :--- | :--- |
| **Đổi màu chủ đạo (Primary color)** | `css/style.css` | Khối `:root` (khoảng dòng 27) | Đổi mã màu `--primary: #e50914;` thành màu mong muốn (ví dụ màu xanh dương: `#0066cc`). Toàn bộ nút, viền, badge trên 5 trang sẽ tự động đổi theo. |
| **Đổi số cột của lưới phim (Grid columns)** | `css/style.css` | Class `.movies-grid` | Thay đổi `minmax(215px, 1fr)` thành kích thước khác (ví dụ: `minmax(180px, 1fr)` để tăng số cột mỗi hàng, hoặc `repeat(4, 1fr)` để cố định đúng 4 cột). |
| **Đổi thời gian chuyển Slide tự động** | `js/slider.js` | Đầu file (khoảng dòng 15) | Đổi hằng số `const SLIDER_AUTOPLAY_INTERVAL = 5000;` (5000 = 5 giây, muốn đổi thành 3 giây thì sửa thành `3000`). |
| **Thêm một bộ phim mới vào hệ thống** | `js/movies-data.js` | Trong mảng `MOVIES_DATABASE` | Thêm một phần tử object mới gồm đủ các trường: `id: 21`, `title`, `year`, `genres`, `duration`, `director`, `cast`, `synopsis`, `rating`, `poster`, `trailerInfo`. Phim sẽ tự xuất hiện trên trang chủ, kho phim và bảng xếp hạng! |
| **Đổi Breakpoint di động (Responsive)** | `css/style.css` | Cuối file | Thay đổi các giá trị `@media (max-width: 768px)` hoặc `@media (max-width: 480px)` theo chuẩn màn hình mong muốn. |
| **Đổi số phim hiển thị ở Top tuần trang chủ** | `index.html` | Thẻ `<script>` cuối trang | Đổi đoạn code `.slice(0, 5)` thành `.slice(0, 6)` hoặc số lượng tùy thích. |
| **Đổi số lượng phim gợi ý trong Trắc nghiệm** | `js/quiz.js` | Hàm `renderResult()` | Thay đổi `.slice(0, 3)` thành `.slice(0, 4)` để gợi ý 4 bộ phim thay vì 3 bộ phim. |
| **Đổi cách tính điểm bài Trắc nghiệm** | `js/quiz.js` | Mảng `QUIZ_QUESTIONS` | Điều chỉnh giá trị thuộc tính `genre: "Hành động"` trong từng tùy chọn đáp án `options` để gán điểm cho thể loại khác. |

---

## PHẦN 4: 10 CÂU HỎI VẤN ĐÁP THƯỜNG GẶP & CÂU TRẢ LỜI MẪU

### Câu 1: Tại sao nhóm không sử dụng Fetch API hoặc ES Modules (`import/export`) để nạp file dữ liệu `movies-data.js`?
> **Gợi ý trả lời:**  
> "Dạ thưa Thầy/Cô, khi chạy một website tĩnh bằng cách mở trực tiếp file HTML qua giao thức `file://` (không qua web server localhost), cơ chế bảo mật Same-Origin Policy của các trình duyệt hiện đại (Chrome, Edge, Firefox) sẽ chặn các yêu cầu `fetch()` đọc file cục bộ và chặn `type="module"`, gây ra lỗi CORS. Vì vậy, nhóm khai báo dữ liệu thành mảng JavaScript toàn cục trong file `js/movies-data.js` và nạp bằng thẻ `<script>` truyền thống trước các file logic. Cách làm này đảm bảo website hoạt động ổn định 100% trên mọi máy tính của Giảng viên mà không cần cài đặt môi trường server."

### Câu 2: Làm thế nào mà chỉ với một file `chi-tiet-phim.html` duy nhất, trang web có thể hiển thị thông tin cho 20 bộ phim khác nhau?
> **Gợi ý trả lời:**  
> "Dạ, nhóm sử dụng cơ chế Dynamic URL Parameters qua đối tượng `window.location.search`. Khi người dùng bấm vào một phim (ví dụ `chi-tiet-phim.html?id=8`), file `js/movie-detail.js` sử dụng `URLSearchParams` để lấy ra giá trị `id=8`. Sau đó, dùng hàm `Array.prototype.find()` tra cứu trong mảng `MOVIES_DATABASE` để tìm bộ phim có `id === 8` và render động toàn bộ poster, tiêu đề, đạo diễn, diễn viên và tóm tắt vào các phần tử DOM tương ứng."

### Câu 3: Hãy giải thích cách nhóm xử lý trường hợp người dùng nhập sai ID trên thanh địa chỉ (ví dụ: `chi-tiet-phim.html?id=999` hoặc `?id=abc`)?
> **Gợi ý trả lời:**  
> "Dạ, trong file `js/movie-detail.js`, nhóm kiểm tra giá trị trả về của `getMovieIdFromUrl()`. Nếu `id` không phải là số hợp lệ (`isNaN`), hoặc kết quả tìm kiếm trong cơ sở dữ liệu trả về `undefined`, hệ thống sẽ lập tức gọi hàm `renderMovieNotFound()`. Hàm này sẽ hiển thị một khối giao diện thông báo lỗi thân thiện (404 Not Found) kèm nút 'Quay lại Kho Phim' và cập nhật lại `document.title`, tránh việc hiển thị trang trắng hay phát sinh lỗi JavaScript trên Console."

### Câu 4: Vì sao trong dự án này nhóm sử dụng CSS Grid cho danh sách phim và Flexbox cho thanh điều hướng, thay vì dùng `float` hay `position: absolute`?
> **Gợi ý trả lời:**  
> "Dạ, Flexbox là công cụ lý tưởng cho bố cục một chiều (1D), giúp căn giữa logo, menu và các nút tiện ích trên thanh điều hướng Navbar một cách linh hoạt. Trong khi đó, CSS Grid là chuẩn bố cục hai chiều (2D), cho phép nhóm sử dụng thuộc tính `grid-template-columns: repeat(auto-fill, minmax(215px, 1fr))` để tạo lưới thẻ phim có khả năng tự động co giãn và dàn hàng hoàn hảo trên nhiều độ phân giải khác nhau mà không cần viết quá nhiều Media Queries. Nhóm hạn chế `position: absolute` và chỉ sử dụng cho các huy hiệu điểm số hoặc lớp phủ (overlay) trên ảnh poster."

### Câu 5: Hệ thống làm thế nào để đảm bảo tính năng lưu "Xem sau" (Watchlist) và "Phim vừa xem" không bị mất khi người dùng tải lại trang hoặc tắt trình duyệt?
> **Gợi ý trả lời:**  
> "Dạ, nhóm sử dụng Web Storage API (`localStorage`). Khi người dùng bấm lưu phim hoặc vào xem chi tiết một bộ phim, dữ liệu ID được chuyển đổi thành chuỗi JSON qua `JSON.stringify()` và ghi vào `localStorage`. Khi bất kỳ trang nào trong 5 trang được tải, hàm `DOMContentLoaded` sẽ đọc lại dữ liệu bằng `JSON.parse()`. Đặc biệt, nhóm bọc các câu lệnh này trong khối `try...catch` thông qua hàm `safeGetStorage()` để đảm bảo website không bao giờ bị lỗi ngay cả khi trình duyệt chặn lưu trữ cục bộ."

### Câu 6: Trình bày thuật toán lọc phim kết hợp nhiều tiêu chí cùng lúc (Realtime Multi-filter) trong `js/search-filter.js`?
> **Gợi ý trả lời:**  
> "Dạ, nhóm xây dựng một đối tượng trạng thái `currentFilterState` lưu trữ các giá trị hiện tại của từ khóa, thể loại, năm, quốc gia và tiêu chí sắp xếp. Khi người dùng nhập ô tìm kiếm hoặc thay đổi bất kỳ ô `select` nào, hàm `getFilteredAndSortedMovies()` sẽ lấy mảng gốc `MOVIES_DATABASE`, sau đó lần lượt áp dụng phương thức `Array.prototype.filter()` qua từng tiêu chí theo nguyên tắc logic AND. Cuối cùng, mảng kết quả được sắp xếp bằng `Array.prototype.sort()` rồi chuyển sang hàm `renderMoviesList()` để vẽ lại giao diện và cập nhật số lượng kết quả."

### Câu 7: Hãy giải thích cách hoạt động của tính năng dừng Slider khi di chuột vào (Pause on Hover) trong `js/slider.js`?
> **Gợi ý trả lời:**  
> "Dạ, cơ chế tự động chuyển slide dựa vào hàm `setInterval(..., 5000)`. Khi người dùng di chuột vào khung slider, sự kiện `mouseenter` được kích hoạt và gọi hàm `pause()`, bên trong sử dụng lệnh `clearInterval(this.timer)` để hủy tiến trình đếm giờ. Khi chuột rời khỏi slider, sự kiện `mouseleave` kích hoạt hàm `play()`, khởi tạo lại một bộ đếm `setInterval` mới để tiếp tục chuyển slide tự động."

### Câu 8: Điểm đánh giá trung bình hiển thị tại trang chi tiết được tính toán lại như thế nào khi người dùng chấm sao?
> **Gợi ý trả lời:**  
> "Dạ, mỗi bộ phim có sẵn điểm trung bình `rating` ban đầu và số lượt đánh giá `voteCount`. Khi người dùng chọn từ 1 đến 5 sao, nhóm quy đổi sang thang điểm 10 (nhân đôi) và tính lại điểm bình quân mới theo công thức tính trung bình cộng gia quyền:  
> `Điểm mới = ((Điểm cũ * Số lượt cũ) + Điểm người dùng) / (Số lượt cũ + 1)`.  
> Sau đó, điểm số mới và lượt đánh giá tăng thêm 1 được cập nhật ngay lập tức lên màn hình và lưu điểm cá nhân của người dùng vào `localStorage` để họ không thể chấm lặp lại nhiều lần trên cùng một máy."

### Câu 9: Thuật toán xác định gu phim và đề xuất trong `js/quiz.js` hoạt động như thế nào?
> **Gợi ý trả lời:**  
> "Dạ, mỗi tùy chọn đáp án trong bộ 7 câu hỏi trắc nghiệm đều gắn liền với một thể loại phim cụ thể. Hệ thống sử dụng một đối tượng tích lũy điểm số cho từng thể loại. Khi người chơi hoàn thành tất cả các câu hỏi, hàm `renderResult()` sẽ duyệt qua đối tượng này để tìm ra thể loại có điểm số cao nhất. Sau đó, hệ thống tự động lọc trong `MOVIES_DATABASE` lấy 3 bộ phim có điểm số cao nhất thuộc thể loại đó để đề xuất kèm đường link xem chi tiết."

### Câu 10: Website có đáp ứng chuẩn Accessibility (khả năng tiếp cận) và Responsive trên thiết bị di động không? Hãy nêu các ví dụ cụ thể trong mã nguồn?
> **Gợi ý trả lời:**  
> "Dạ có, nhóm đã áp dụng đầy đủ các tiêu chuẩn tiếp cận:
> 1. Về ngữ nghĩa: Sử dụng đúng các thẻ HTML5 (`<header>`, `<nav>`, `<main>`, `<article>`, `<section>`, `<aside>`, `<footer>`), bảng xếp hạng có thẻ `<caption>` và `th scope="col"`, các nút bấm đều có thuộc tính `aria-label` và `title`.
> 2. Về kích thước font: Cỡ chữ thân bài tối thiểu là `16px`, đảm bảo độ tương phản màu sắc cao trên nền tối điện ảnh.
> 3. Về phím bấm: Hỗ trợ viền `outline` rõ ràng khi dùng phím Tab qua `:focus-visible`, đóng modal bằng phím `Escape`, chuyển slide bằng phím mũi tên.
> 4. Về Responsive: Menu tự động chuyển thành nút bấm Hamburger trên màn hình $\le 768px$, bảng dữ liệu được bọc trong container có thuộc tính `overflow-x: auto` chống vỡ layout, và hình ảnh poster có tỉ lệ cố định `aspect-ratio: 2 / 3` chống hiện tượng giật cục giao diện (Cumulative Layout Shift)."

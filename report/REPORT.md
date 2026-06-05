# Báo Cáo Lab 7: Embedding & Vector Store

**Họ tên:** Nguyễn Sĩ Việt
**Nhóm:** Baking Recipe
**Ngày:** 05/06/2026

---

## 1. Warm-up (5 điểm)

### Cosine Similarity (Ex 1.1)

**High cosine similarity nghĩa là gì?**
High cosine similarity means two text embeddings point in nearly the same direction in vector space, indicating that the sentences have similar meanings or semantic content.

**Ví dụ HIGH similarity:**
- Sentence A: I love studying artificial intelligence and machine learning
- Sentence B: I enjoy learning about AI and machine learning technologies.
- Tại sao tương đồng: Both sentences address the same topic as they both refer to learning artificial intelligence and machine learning.

**Ví dụ LOW similarity:**
- Sentence A: I love studying artificial intelligence and machine learning.
- Sentence B: The weather is very hot today.
- Tại sao khác: The first sentence is about learning AI, the second is about the weather. The two sentences are not semantically related.

**Tại sao cosine similarity được ưu tiên hơn Euclidean distance cho text embeddings?**
Cosine similarity focuses on the angle between vectors rather than their magnitude, making it more suitable for comparing semantic similarity in text embeddings. Euclidean distance can be affected by vector length, while cosine similarity better captures meaning regardless of embedding magnitude.

### Chunking Math (Ex 1.2)

**Document 10,000 ký tự, chunk_size=500, overlap=50. Bao nhiêu chunks?**

> *Trình bày phép tính:*

step = chunk_size - overlap  
     = 500 - 50  
     = 450  

chunks = ceil((10000 - 500) / 450) + 1  
       = ceil(9500 / 450) + 1  
       = ceil(21.11) + 1  
       = 22 + 1  
       = 23  

> *Đáp án:* 23 chunks

**Nếu overlap tăng lên 100, chunk count thay đổi thế nào? Tại sao muốn overlap nhiều hơn?**

step = chunk_size - overlap  
     = 500 - 100  
     = 400  

chunks = ceil((10000 - 500) / 400) + 1  
       = ceil(9500 / 400) + 1  
       = ceil(23.75) + 1  
       = 24 + 1  
       = 25  

> Khi overlap tăng từ 50 lên 100, số chunks tăng từ 23 lên 25 vì bước nhảy giảm từ 450 xuống 400. Overlap nhiều hơn giúp giữ ngữ cảnh giữa các chunk tốt hơn, giảm nguy cơ mất thông tin khi một câu hoặc một ý bị cắt ở ranh giới giữa hai chunk.
---

## 2. Document Selection — Nhóm (10 điểm)

### Domain & Lý Do Chọn

**Domain:** Baking Recipe (Công thức làm bánh từ các nguồn chính thống quốc tế)

**Tại sao nhóm chọn domain này?**

> Nhóm chọn chủ đề công thức làm bánh từ các học viện ẩm thực danh tiếng thế giới (Le Cordon Bleu, King Arthur Baking) nhằm đảm bảo tính chính xác và chất lượng chuẩn của dữ liệu. Cấu trúc của công thức làm bánh yêu cầu tính chính xác cực cao về tỉ lệ và thời gian, rất phù hợp để kiểm chứng khả năng bảo toàn ngữ cảnh của các phương pháp chunking.

### Data Inventory

| #   | Tên tài liệu              | Nguồn                   | Số ký tự | Metadata đã gán                                                          |
| --- | ------------------------- | ----------------------- | -------- | ------------------------------------------------------------------------ |
| 1   | croissant.md              | Học viện Le Cordon Bleu | 1316     | source_authority: "le_cordon_bleu", category: "laminated_pastry"         |
| 2   | baguette.md               | Học viện Le Cordon Bleu | 1266     | source_authority: "le_cordon_bleu", category: "bread"                    |
| 3   | macaron.md                | Học viện Le Cordon Bleu | 1466     | source_authority: "le_cordon_bleu", category: "cookies"                  |
| 4   | tiramisu.md               | Học viện CIA (Mỹ)       | 1588     | source_authority: "culinary_institute_of_america", category: "dessert"   |
| 5   | apple_pie.md              | King Arthur Baking      | 1621     | source_authority: "king_arthur_baking", category: "pie"                  |
| 6   | sourdough.md              | King Arthur Baking      | 1559     | source_authority: "king_arthur_baking", category: "bread"                |
| 7   | creme_brulee.md           | Học viện Le Cordon Bleu | 1568     | source_authority: "le_cordon_bleu", category: "dessert"                  |
| 8   | cinnamon_rolls.md         | King Arthur Baking      | 1616     | source_authority: "king_arthur_baking", category: "bread"                |
| 9   | chocolate_chip_cookies.md | King Arthur Baking      | 1772     | source_authority: "king_arthur_baking", category: "cookies"              |
| 10  | choux_pastry.md           | Học viện Le Cordon Bleu | 2138     | source_authority: "le_cordon_bleu", category: "pastry"                   |
| 11  | red_velvet_cake.md        | Học viện CIA (Mỹ)       | 1387     | source_authority: "culinary_institute_of_america", category: "cake"      |
| 12  | lemon_tart.md             | Học viện Le Cordon Bleu | 1399     | source_authority: "le_cordon_bleu", category: "tart"                     |
| 13  | brownies.md               | King Arthur Baking      | 1353     | source_authority: "king_arthur_baking", category: "cookies"              |
| 14  | pain_au_chocolat.md       | Học viện Le Cordon Bleu | 1267     | source_authority: "le_cordon_bleu", category: "laminated_pastry"         |
| 15  | cheesecake.md             | Học viện CIA (Mỹ)       | 1630     | source_authority: "culinary_institute_of_america", category: "cake"      |
| 16  | pavlova.md                | Học viện Le Cordon Bleu | 1700     | source_authority: "le_cordon_bleu", category: "dessert"                  |
| 17  | banana_bread.md           | King Arthur Baking      | 1417     | source_authority: "king_arthur_baking", category: "bread"                |
| 18  | waffles.md                | Học viện CIA (Mỹ)       | 1416     | source_authority: "culinary_institute_of_america", category: "breakfast" |
| 19  | shortbread_cookies.md     | King Arthur Baking      | 1498     | source_authority: "king_arthur_baking", category: "cookies"              |
| 20  | eclairs.md                | Học viện Le Cordon Bleu | 1731     | source_authority: "le_cordon_bleu", category: "pastry"                   |

### Metadata Schema

| Trường metadata  | Kiểu   | Ví dụ giá trị                          | Tại sao hữu ích cho retrieval?                                                               |
| ---------------- | ------ | -------------------------------------- | -------------------------------------------------------------------------------------------- |
| source_authority | String | "le_cordon_bleu", "king_arthur_baking" | Cho phép lọc và truy xuất công thức theo nguồn học viện tin cậy cụ thể.                      |
| category         | String | "bread", "dessert", "pie"              | Hỗ trợ phân loại nhanh loại bánh người dùng muốn làm để loại bỏ nhiễu từ các nhóm bánh khác. |

---

## 3. Chunking Strategy — Cá nhân chọn, nhóm so sánh (15 điểm)

### Baseline Analysis

Nhóm sử dụng domain **baking recipes / công thức làm bánh**. Tôi chạy `ChunkingStrategyComparator().compare()` trên 3 tài liệu mẫu trong thư mục `data/` với `chunk_size=500`: `apple_pie.md`, `croissant.md`, và `tiramisu.md`.

| Tài liệu | Strategy | Chunk Count | Avg Length | Preserves Context? |
|-----------|----------|-------------|------------|-------------------|
| apple_pie.md | FixedSizeChunker (`fixed_size`) | 4 | 405.2 | Trung bình, có thể cắt ngang section hoặc danh sách nguyên liệu |
| apple_pie.md | SentenceChunker (`by_sentences`) | 7 | 230.3 | Tốt, giữ ranh giới câu nhưng chunk hơi nhỏ |
| apple_pie.md | RecursiveChunker (`recursive`) | 5 | 322.8 | Tốt, giữ được đoạn/section tự nhiên |
| croissant.md | FixedSizeChunker (`fixed_size`) | 3 | 438.7 | Trung bình, chunk dài và có thể chứa nhiều ý |
| croissant.md | SentenceChunker (`by_sentences`) | 6 | 218.2 | Tốt, dễ đọc nhưng có thể thiếu context |
| croissant.md | RecursiveChunker (`recursive`) | 4 | 327.8 | Cân bằng nhất giữa độ dài và context |
| tiramisu.md | FixedSizeChunker (`fixed_size`) | 4 | 397.0 | Chấp nhận được nhưng kém linh hoạt |
| tiramisu.md | SentenceChunker (`by_sentences`) | 6 | 263.5 | Tốt, giữ nguyên câu |
| tiramisu.md | RecursiveChunker (`recursive`) | 5 | 316.2 | Tốt, giữ cấu trúc recipe rõ hơn |

### Strategy Của Tôi

**Loại:** RecursiveChunker

**Mô tả cách hoạt động:**  
Tôi chọn `RecursiveChunker` cho bộ dữ liệu công thức làm bánh. Strategy này không cắt text một cách cứng theo số ký tự ngay từ đầu, mà thử tách theo thứ tự tự nhiên hơn: đoạn văn `\n\n`, dòng `\n`, câu `. `, từ `" "`, rồi cuối cùng mới cắt theo ký tự nếu đoạn vẫn quá dài. Cách này giúp chunk giữ được cấu trúc của recipe như phần nguyên liệu, cách làm, lưu ý và nguồn tham khảo.

**Tại sao tôi chọn strategy này cho domain nhóm?**  
Domain công thức làm bánh thường có cấu trúc rõ ràng theo section. Nếu dùng `FixedSizeChunker`, chunk có thể bị cắt ngang giữa danh sách nguyên liệu hoặc một bước làm bánh, làm giảm chất lượng retrieval. `RecursiveChunker` phù hợp hơn vì nó cố gắng giữ một ý hoặc một section trong cùng chunk, nhưng vẫn đảm bảo chunk không quá dài.

**Code snippet (nếu custom):**


```python
from src import RecursiveChunker

chunker = RecursiveChunker(chunk_size=500)
chunks = chunker.chunk(text)
```

### So Sánh: Strategy của tôi vs Baseline

| Tài liệu | Strategy | Chunk Count | Avg Length | Retrieval Quality? |
|-----------|----------|-------------|------------|--------------------|
| apple_pie.md | FixedSizeChunker | 4 | 405.2 | Ít chunk hơn nhưng có thể cắt ngang ý |
| apple_pie.md | **RecursiveChunker của tôi** | 5 | 322.8 | Chunk tự nhiên hơn, giữ section tốt hơn |
| croissant.md | FixedSizeChunker | 3 | 438.7 | Chunk dài, dễ chứa nhiều ý khác nhau |
| croissant.md | **RecursiveChunker của tôi** | 4 | 327.8 | Cân bằng hơn giữa độ dài và context |
| tiramisu.md | FixedSizeChunker | 4 | 397.0 | Chấp nhận được nhưng kém linh hoạt |
| tiramisu.md | **RecursiveChunker của tôi** | 5 | 316.2 | Giữ cấu trúc recipe rõ hơn |

### So Sánh Với Thành Viên Khác

Vì các thành viên trong nhóm thử các strategy khác nhau trên cùng bộ data recipe, tôi so sánh dựa trên khả năng giữ context và chất lượng top-k retrieval cho 5 benchmark queries của nhóm.

| Thành viên | Strategy | Retrieval Score (/10) | Điểm mạnh | Điểm yếu |
|-----------|----------|----------------------|-----------|----------|
| Bùi Minh Hiếu - 2A202600876 | RecursiveChunker | 8 | Giữ cấu trúc recipe tốt, chunk vừa phải, phù hợp với section nguyên liệu/cách làm | Tạo nhiều chunk hơn fixed-size |
| Nguyễn Sĩ Việt - 2A202600658 | FixedSizeChunker | 6 | Đơn giản, số chunk ít, dễ kiểm soát độ dài | Dễ cắt ngang danh sách nguyên liệu hoặc bước làm bánh |
| Trương Hải Quân - 2A202600898 | SentenceChunker | 7 | Giữ nguyên ranh giới câu, chunk dễ đọc | Chunk có thể quá nhỏ và thiếu context của cả section |
| Lương Quốc Đoàn - 2A202600564 | RecursiveChunker với chunk_size nhỏ hơn | 7 | Tạo chunk tập trung, dễ retrieve các chi tiết nhỏ như định lượng nguyên liệu | Có thể tách rời phần nguyên liệu và cách làm nếu chunk quá nhỏ |
| Trịnh Vũ Anh Tuấn - 2A202600847 | RecursiveChunker với chunk_size lớn hơn | 7 | Giữ được nhiều context trong cùng một chunk, phù hợp với recipe ngắn | Chunk dài hơn nên đôi khi chứa nhiều ý, score có thể kém tập trung |
| Hoàng Phương Thảo - 2A202600951 | SentenceChunker với nhiều câu mỗi chunk | 7 | Giữ câu nguyên vẹn và có thêm context so với sentence chunk nhỏ | Vẫn phụ thuộc vào dấu câu, không tận dụng tốt section markdown |
| Ngô Thanh Tình - 2A202600919 | FixedSizeChunker có overlap | 6 | Overlap giúp giảm mất thông tin ở ranh giới chunk | Vẫn có nguy cơ cắt ngang section hoặc danh sách nguyên liệu |
| Phùng Văn Thạch - 2A202601004 | Custom section/header chunker | 8 | Phù hợp nhất với markdown recipe vì tách theo heading như Nguyên liệu/Cách làm | Cần viết logic riêng, phụ thuộc format tài liệu |

**Strategy nào tốt nhất cho domain này? Tại sao?**

Với domain recipe, `RecursiveChunker` là strategy phù hợp nhất trong ba strategy đã thử. Lý do là recipe thường được viết theo section, ví dụ nguyên liệu, cách làm, lưu ý, và `RecursiveChunker` có xu hướng giữ các section này nguyên vẹn hơn. So với `FixedSizeChunker`, nó ít cắt ngang ý hơn; so với `SentenceChunker`, nó giữ được nhiều context hơn trong mỗi chunk.

---

## 4. My Approach — Cá nhân (10 điểm)

Giải thích cách tiếp cận của bạn khi implement các phần chính trong package `src`.

### Chunking Functions

**`SentenceChunker.chunk`** — approach:

Tôi dùng regex `(?<=[.!?])(?:\s+|$)` để tách câu tại khoảng trắng hoặc cuối chuỗi sau các dấu `.`, `!`, `?`. Sau đó loại bỏ khoảng trắng thừa và gom mỗi `max_sentences_per_chunk` câu thành một chunk. Edge case được xử lý là text rỗng trả về `[]`, và `max_sentences_per_chunk` nhỏ hơn 1 sẽ được ép tối thiểu thành 1.

**`RecursiveChunker.chunk` / `_split`** — approach:

Thuật toán thử tách văn bản theo thứ tự separator tự nhiên: đoạn văn, dòng, câu, từ, rồi cuối cùng mới cắt ký tự. Nếu một đoạn sau khi tách vẫn dài hơn `chunk_size`, hàm `_split` gọi đệ quy với separator tiếp theo. Base case là text rỗng, text đã ngắn hơn `chunk_size`, hoặc không còn separator thì cắt cứng theo `chunk_size`.

### EmbeddingStore

**`add_documents` + `search`** — approach:

Mỗi `Document` được chuyển thành record chuẩn gồm `id`, `content`, `metadata`, và `embedding`; metadata luôn có thêm `doc_id` để hỗ trợ delete. Khi search, query được embed bằng cùng embedding function, sau đó store tính dot product giữa query vector và từng document vector. Kết quả được sort giảm dần theo `score` và trả về tối đa `top_k`.

**`search_with_filter` + `delete_document`** — approach:

Với `search_with_filter`, tôi filter các record theo metadata trước rồi mới chạy similarity search trên tập đã lọc, giúp giảm nhiễu khi người dùng biết domain/category. Với `delete_document`, tôi tìm các record có `metadata["doc_id"] == doc_id`, xóa khỏi list in-memory và trả về `True` nếu có ít nhất một record bị xóa. Nếu dùng ChromaDB, các id tương ứng cũng được gửi vào `collection.delete`.

### KnowledgeBaseAgent

**`answer`** — approach:
Agent gọi `store.search(question, top_k)` để lấy các chunk liên quan, nối nội dung chunk thành một khối `Context`, rồi tạo prompt gồm instruction, context, question và nhãn `Answer:`. Context được inject trực tiếp vào prompt để LLM trả lời dựa trên tài liệu retrieved thay vì chỉ dựa vào kiến thức chung. Cuối cùng agent gọi `llm_fn(prompt)` và trả về chuỗi answer.

### Test Results

```
============================= test session starts ==============================
platform darwin -- Python 3.13.7, pytest-9.0.3, pluggy-1.6.0
rootdir: /Users/thachphung/Documents/AI Thực Chiến/Day 7/2A202601004-Phung-Van-Thach
collecting ... collected 42 items

tests/test_solution.py::TestProjectStructure::test_root_main_entrypoint_exists PASSED
tests/test_solution.py::TestProjectStructure::test_src_package_exists PASSED
tests/test_solution.py::TestClassBasedInterfaces::test_chunker_classes_exist PASSED
tests/test_solution.py::TestClassBasedInterfaces::test_mock_embedder_exists PASSED
tests/test_solution.py::TestFixedSizeChunker::test_chunks_respect_size PASSED
tests/test_solution.py::TestFixedSizeChunker::test_correct_number_of_chunks_no_overlap PASSED
tests/test_solution.py::TestFixedSizeChunker::test_empty_text_returns_empty_list PASSED
tests/test_solution.py::TestFixedSizeChunker::test_no_overlap_no_shared_content PASSED
tests/test_solution.py::TestFixedSizeChunker::test_overlap_creates_shared_content PASSED
tests/test_solution.py::TestFixedSizeChunker::test_returns_list PASSED
tests/test_solution.py::TestFixedSizeChunker::test_single_chunk_if_text_shorter PASSED
tests/test_solution.py::TestSentenceChunker::* PASSED
tests/test_solution.py::TestRecursiveChunker::* PASSED
tests/test_solution.py::TestEmbeddingStore::* PASSED
tests/test_solution.py::TestKnowledgeBaseAgent::* PASSED
tests/test_solution.py::TestComputeSimilarity::* PASSED
tests/test_solution.py::TestCompareChunkingStrategies::* PASSED
tests/test_solution.py::TestEmbeddingStoreSearchWithFilter::* PASSED
tests/test_solution.py::TestEmbeddingStoreDeleteDocument::* PASSED

============================== 42 passed in 0.04s ==============================
```

**Số tests pass:** 42/42

---

## 5. Similarity Predictions — Cá nhân (5 điểm)

| Pair | Sentence A | Sentence B | Dự đoán    | Actual Score | Đúng? |
| ---- | ---------- | ---------- | ---------- | ------------ | ----- |
| 1    | I love studying artificial intelligence and machine learning. | I enjoy learning about AI and machine learning technologies. | high | -0.160 | Không |
| 2    | I love studying artificial intelligence and machine learning. | The weather is very hot today. | low | -0.216 | Có |
| 3    | Croissant dough needs cold butter and folding. | Laminated pastry is made by folding butter into dough. | high | 0.190 | Có |
| 4    | Tiramisu uses espresso and mascarpone cream. | Sourdough bread uses starter and long fermentation. | low | -0.065 | Có |
| 5    | Apple pie filling contains cinnamon and apples. | Pie crust uses flour, butter, and cold water. | high | -0.161 | Không |

**Kết quả nào bất ngờ nhất? Điều này nói gì về cách embeddings biểu diễn nghĩa?**

Kết quả bất ngờ nhất là pair 1: hai câu gần nghĩa nhưng score lại âm khi dùng `_mock_embed`. Điều này cho thấy mock embedding của lab chỉ phù hợp để test pipeline vì nó tạo vector deterministic từ hash, không thật sự biểu diễn nghĩa ngôn ngữ. Nếu dùng embedding semantic như `all-MiniLM-L6-v2` hoặc OpenAI embedding, tôi kỳ vọng các cặp cùng chủ đề sẽ có score cao hơn rõ rệt.

---

## 6. Results — Cá nhân (10 điểm)

Chạy 5 benchmark queries của nhóm trên implementation cá nhân của bạn trong package `src`. **5 queries phải trùng với các thành viên cùng nhóm.**

### Benchmark Queries & Gold Answers (nhóm thống nhất)

| #   | Query | Gold Answer |
| --- | ----- | ----------- |
| 1   | Croissant cần tạo bao nhiêu lớp bơ-bột khi cán? | Công thức croissant cán và gấp bột 3 lần, mỗi lần gấp làm 3, tạo 27 lớp bơ-bột xen kẽ. |
| 2   | Tiramisu cần ủ lạnh tối thiểu bao lâu trước khi ăn? | Tiramisu cần để lạnh ít nhất 6 tiếng, hoặc qua đêm, trước khi ăn. |
| 3   | Apple pie nướng ở nhiệt độ nào và trong bao lâu? | Apple pie nướng 200 độ C trong 20 phút đầu, sau đó giảm xuống 190 độ C và nướng tiếp 35-40 phút. |
| 4   | Sourdough cần ủ chậm trong tủ lạnh bao lâu? | Sourdough được bọc kín và ủ chậm trong tủ lạnh ở 4 độ C từ 12-16 tiếng. |
| 5   | Tart chanh cần để lạnh bao lâu trước khi dùng? | Tart chanh cần để đông lạnh trong tủ lạnh ít nhất 2 tiếng trước khi dùng. |

### Kết Quả Của Tôi

Embedding backend: `_mock_embed`. Strategy: `RecursiveChunker(chunk_size=500)`. Các query được chạy bằng `search_with_filter()` khi có metadata rõ ràng để mô phỏng retrieval trong domain recipe.

| #   | Query | Top-1 Retrieved Chunk (tóm tắt) | Score | Relevant? | Agent Answer (tóm tắt) |
| --- | ----- | ------------------------------- | ----- | --------- | ---------------------- |
| 1   | Croissant cần tạo bao nhiêu lớp bơ-bột khi cán? | `pain_au_chocolat.md`: phần làm bột ngàn lớp giống croissant, 3 lần cán gập tạo 27 lớp bơ bột. | 0.193 | Có, trong top-3 có chunk croissant; top-1 cũng cùng kỹ thuật laminated pastry | 27 lớp bơ-bột. |
| 2   | Tiramisu cần ủ lạnh tối thiểu bao lâu trước khi ăn? | `tiramisu.md`: chunk ráp bánh và ủ lạnh, nhắc lớp bánh/kem và phần hoàn thiện. | 0.177 | Có | Ít nhất 6 tiếng hoặc qua đêm. |
| 3   | Apple pie nướng ở nhiệt độ nào và trong bao lâu? | `apple_pie.md`: chunk các bước làm vỏ, nhân và nướng bánh apple pie. | 0.231 | Có | 200 độ C trong 20 phút, rồi 190 độ C trong 35-40 phút. |
| 4   | Sourdough cần ủ chậm trong tủ lạnh bao lâu? | `cinnamon_rolls.md`: top-1 bị nhiễu trong nhóm bread; top-3 có chunk `sourdough.md`. | 0.214 | Có, relevant trong top-3 | 12-16 tiếng ở 4 độ C. |
| 5   | Tart chanh cần để lạnh bao lâu trước khi dùng? | `lemon_tart.md`: chunk các bước làm vỏ tart, nướng mù, làm nhân và hoàn thành. | 0.040 | Có | Ít nhất 2 tiếng trước khi dùng. |

**Bao nhiêu queries trả về chunk relevant trong top-3?** 5 / 5

---

## 7. What I Learned (5 điểm — Demo)

**Điều hay nhất tôi học được từ thành viên khác trong nhóm:**

Tôi học được rằng cùng một bộ tài liệu recipe nhưng tham số chunking khác nhau có thể làm retrieval thay đổi rất mạnh. Các bạn dùng chunk theo section/header giữ được phần "Nguyên liệu" và "Các bước thực hiện" rõ hơn, đặc biệt khi query hỏi đúng một bước cụ thể.

**Điều hay nhất tôi học được từ nhóm khác (qua demo):**

Tôi học được cách dùng metadata như một lớp lọc trước similarity search để giảm nhiễu. Với mock embedding, filter theo `category` hoặc `source_authority` cải thiện kết quả rõ ràng hơn so với chỉ search toàn bộ collection.

**Nếu làm lại, tôi sẽ thay đổi gì trong data strategy?**

Nếu làm lại, tôi sẽ thiết kế metadata chi tiết hơn như `dish_name`, `step_type`, `ingredient`, và `temperature` thay vì chỉ có `category` và `source_authority`. Tôi cũng muốn thử custom chunker tách theo heading markdown để mỗi chunk chỉ chứa một phần rõ ràng như nguyên liệu, cách làm, hoặc thời gian/nhiệt độ. Failure case lớn nhất là query tự nhiên không filter có thể retrieve sai vì `_mock_embed` không hiểu nghĩa, nên dùng semantic embedding thật sẽ là cải tiến quan trọng.

---

## Tự Đánh Giá

| Tiêu chí                    | Loại    | Điểm tự đánh giá |
| --------------------------- | ------- | ---------------- |
| Warm-up                     | Cá nhân | 5 / 5            |
| Document selection          | Nhóm    | 9 / 10           |
| Chunking strategy           | Nhóm    | 14 / 15          |
| My approach                 | Cá nhân | 10 / 10          |
| Similarity predictions      | Cá nhân | 4 / 5            |
| Results                     | Cá nhân | 9 / 10           |
| Core implementation (tests) | Cá nhân | 30 / 30          |
| Demo                        | Nhóm    | 4 / 5            |
| **Tổng**                    |         | **95 / 100**     |

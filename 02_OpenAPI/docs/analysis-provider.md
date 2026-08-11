# Phân tích yêu c?u — vai Provider

- C?p ðàm phán: Camera Stream ? AI Vision
- Product: Smart Campus AI Vision
- Provider service: AI Vision Service
- Consumer service: Camera Stream Service
- Ngý?i vi?t: Nhóm FIT4110
- Ngày: 2026-08-11

---

## 1. Resource chính

| Resource | Mô t? | Thu?c tính b?t bu?c | Thu?c tính tùy ch?n |
|---|---|---|---|
| VisionRequest | Yêu c?u phát hi?n ð?i tý?ng t? ?nh | inputType, correlationId, imageUrl ho?c imageBase64, contentType | none |
| VisionDetectionResult | K?t qu? phát hi?n ð?i tý?ng | detectionId, status, detectedAt, objects, riskLevel | modelVersion, notes |
| VisionModelInfo | Metadata mô h?nh AI Vision | modelName, modelVersion, supportedTypes, updatedAt | none |

---

## 2. Action/API d? ki?n

| Method | Path | M?c ðích | Consumer g?i khi nào? |
|---|---|---|---|
| POST | /vision/detect | G?i ?nh ho?c URL ?nh ð? AI Vision phát hi?n | Khi camera nh?n frame m?i ho?c metadata ?nh ðý?c kích ho?t |
| GET | /vision/detections/{detectionId} | L?y k?t qu? phát hi?n ð? x? l? | Khi consumer c?n truy v?n tr?ng thái/phát hi?n c?a request trý?c ðó |
| GET | /vision/models/info | L?y thông tin model hi?n t?i | Khi c?n xác nh?n model và kh? nãng phát hi?n h? tr? |
| GET | /health | Ki?m tra service c?n ho?t ð?ng | Trong bý?c kh?i t?o ho?c giám sát ðõn gi?n |

---

## 3. Error case

T?i thi?u 5 case.

| Status | T?nh hu?ng | Response body d? ki?n |
|---:|---|---|
| 400 | Payload sai ð?nh d?ng ho?c không có inputType | Problem |
| 401 | Thi?u Bearer token ho?c token không h?p l? | Problem |
| 404 | detectionId không t?n t?i | Problem |
| 422 | imageUrl không truy c?p ðý?c ho?c imageBase64 không ðúng ð?nh d?ng | Problem |
| 500 | L?i n?i b? khi x? l? ?nh | Problem |
| 409 | Idempotency key trùng l?p n?u consumer g?i l?i cùng request (týõng lai) | Problem |

---

## 4. Gi? ð?nh b? sung

- Provider s? h? tr? c? URL ?nh và d? li?u ?nh base64 trong cùng m?t API.
- Consumer ph?i g?i `correlationId` ð? truy v?t request, nhýng service không ép bu?c tính duy nh?t toàn c?c.
- Model AI Vision s? tr? k?t qu? ð?ng b? n?u x? l? nhanh, ho?c state PROCESSING n?u c?n thêm th?i gian.
- `imageUrl` ph?i là URL truy c?p ðý?c t? m?ng internal ho?c public.
- `imageBase64` không ðý?c vý?t quá gi?i h?n backend (m?t vài MB).

---

## 5. Câu h?i cho Consumer

1. Consumer có ýu tiên g?i URL ?nh hay g?i base64 n?u ?nh l?y t? camera n?i b??
2. Trý?ng h?p không th? truy c?p `imageUrl`, service nên tr? 422 hay gi? request ð? retry n?i b??
3. N?u k?t qu? chýa s?n sàng, consumer có c?n nh?n thông báo push hay ch? polling endpoint?

---

## 6. R?i ro tích h?p

| R?i ro | Tác ð?ng | Ð? xu?t x? l? |
|---|---|---|
| inputType gi?a hai bên không ð?ng nh?t | Request b? reject | Ch?t enum IMAGE_URL / IMAGE_BASE64 trong contract |
| URL ?nh không truy c?p ðý?c | Detection th?t b?i / consumer retry không ðúng | Ð?nh ngh?a r? 422 và yêu c?u URL truy c?p ðý?c |
| Sai ð?nh d?ng base64 | L?i parsing | Tr? Problem Details ð? consumer s?a payload |
| Không có authentication | API t? ch?i truy c?p | Dùng Bearer JWT v?i l?i 401 |
| Consumer hi?u khác v? status | Sai cách polling | Ch?t status PROCESSING / COMPLETED / FAILED |

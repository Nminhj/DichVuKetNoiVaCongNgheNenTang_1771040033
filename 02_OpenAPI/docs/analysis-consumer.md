# Phân tích yêu c?u — vai Consumer

- C?p ðàm phán: Camera Stream ? AI Vision
- Consumer service: Camera Stream Service
- Provider service: AI Vision Service
- Ngý?i vi?t: Nhóm FIT4110
- Ngày: 2026-08-11

---

## 1. Resource Consumer c?n nh?n/g?i

| Resource | Consumer dùng ð? làm g?? | Field b?t bu?c v?i Consumer | Field có th? tùy ch?n |
|---|---|---|---|
| VisionRequest | G?i ?nh ð? AI Vision phân tích | inputType, correlationId, imageUrl ho?c imageBase64, contentType | none |
| VisionDetectionResult | Nh?n k?t qu? phát hi?n | detectionId, status, detectedAt, objects, riskLevel | modelVersion, notes |
| VisionModelInfo | Xác nh?n model hi?n t?i | modelName, modelVersion, supportedTypes, updatedAt | none |

---

## 2. API Consumer c?n g?i

| Method | Path | Lúc nào g?i? | K? v?ng response |
|---|---|---|---|
| POST | /vision/detect | Sau khi camera nh?n frame m?i c?n phân tích | Nh?n detectionId và tr?ng thái PROCESSING ho?c COMPLETED |
| GET | /vision/detections/{detectionId} | Sau khi nh?n detectionId ð? ki?m tra tr?ng thái | Nh?n k?t qu? detection khi hoàn thành |
| GET | /vision/models/info | Khi c?n bi?t model và lo?i ð?i tý?ng ðý?c h? tr? | Nh?n metadata model hi?n t?i |
| GET | /health | Kh?i t?o và giám sát ðõn gi?n | Nh?n status ok |

---

## 3. Error case Consumer c?n x? l?

T?i thi?u 5 case.

| Status | Consumer hi?u là g?? | Consumer s? x? l? th? nào? |
|---:|---|---|
| 400 | Yêu c?u g?i payload sai c?u trúc | Ki?m tra l?i request body và s?a n?u c?n |
| 401 | Chýa xác th?c ho?c token h?t h?n | Ðãng nh?p l?i / c?p nh?t token |
| 404 | detectionId không t?n t?i | Hi?n th? l?i ho?c g?i l?i request m?i |
| 422 | d? li?u ?nh không h?p l? | Báo l?i và g?i l?i ?nh ðúng ð?nh d?ng |
| 500 | L?i server | Retry theo backoff ho?c báo c?nh báo v?n hành |
| 409 | D? li?u trùng l?p trong týõng lai | N?u có idempotency, tránh g?i l?i |

---

## 4. Gi? ð?nh b? sung

- Consumer s? g?n `Bearer` token h?p l? cho m?i request.
- `correlationId` ph?c v? truy v?t và không ð?ng nh?t v?i `detectionId`.
- Consumer có th? g?i `imageUrl` cho ?nh ð? t?i s?n ho?c `imageBase64` n?u ch? có d? li?u byte.
- Th?i gian x? l? detection ðý?c gi?i h?n trong vài giây ð? consumer có th? polling h?p l?.

---

## 5. Câu h?i cho Provider

1. N?u `imageUrl` và `imageBase64` ð?u xu?t hi?n, service ýu tiên trý?ng nào?
2. Service s? tr? status PROCESSING hay COMPLETED n?u detection nhanh?
3. `notes` trong k?t qu? có dùng ð? gi?i thích gi?i thu?t hay ch? dùng n?i b??

---

## 6. R?i ro tích h?p

| R?i ro | Tác ð?ng | Ð? xu?t x? l? |
|---|---|---|
| inputType không kh?p | Request b? reject | Ch?t enum và dùng discriminator |
| image URL b? firewall ch?n | Request không x? l? ðý?c | Nêu r? URL ph?i truy c?p ðý?c t? m?ng campus |
| Base64 b? chunk sai | Parse l?i | Yêu c?u contentType và base64 h?p l? |
| status chýa r? | Consumer polling không ðúng | Ch?t tr?ng thái PROCESSING / COMPLETED / FAILED |
| error response không chu?n | Consumer không parse ðý?c | Dùng Problem Details chu?n RFC 7807 |

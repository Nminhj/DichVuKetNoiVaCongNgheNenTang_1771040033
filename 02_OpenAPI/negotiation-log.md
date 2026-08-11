# Biên b?n ðàm phán h?p ð?ng API

- C?p ðàm phán: Camera Stream ? AI Vision
- Product: Smart Campus Vision
- Provider: AI Vision Service
- Consumer: Camera Stream Service
- Phiên: v1.0
- Ngày: 2026-08-11

---

## Issue #1

- Raised by: Consumer
- Endpoint: POST /vision/detect
- Concern: Consumer c?n g?i ?nh t? camera n?i b?. Không r? provider ch?p nh?n URL hay base64.
- Proposal: H?p ð?ng cho phép c? hai cách s? d?ng qua discriminator `inputType`.
- Resolution: Accepted
- Rationale: Gi?m rào c?n tri?n khai cho camera và cho phép both URL và inline ?nh.
- Impact: Provider c?n ki?m tra `inputType` và xác th?c `imageUrl` / `imageBase64` týõng ?ng.

---

## Issue #2

- Raised by: Provider
- Endpoint: POST /vision/detect
- Concern: C?n bi?t cách x? l? khi ?nh g?i chýa truy c?p ðý?c ho?c base64 sai.
- Proposal: Tr? l?i 422 v?i Problem Details n?u ?nh không th? phân tích due semantic validation.
- Resolution: Accepted
- Rationale: 422 phân bi?t payload h?p l? v? JSON nhýng không h?p l? v? n?i dung nghi?p v?.
- Impact: Consumer ph?i xác ð?nh l?i ?nh và g?i l?i n?u URL không kh? d?ng.

---

## Issue #3

- Raised by: Consumer
- Endpoint: GET /vision/detections/{detectionId}
- Concern: Consumer c?n bi?t status khi detection chýa hoàn thành.
- Proposal: Tr? `status: PROCESSING` khi k?t qu? chýa s?n sàng và `COMPLETED` khi xong.
- Resolution: Accepted
- Rationale: Cho phép polling ðõn gi?n và consumer bi?t khi nào c?n retry.
- Impact: Provider ph?i lýu tr?ng thái và tr? ðúng tr?ng thái hi?n t?i.

---

## Issue #4

- Raised by: Provider
- Endpoint: GET /vision/models/info
- Concern: Consumer c?n xác nh?n lo?i ð?i tý?ng ðý?c h? tr?.
- Proposal: Thêm endpoint metadata tr? `supportedTypes` và phiên b?n model.
- Resolution: Accepted
- Rationale: Giúp consumer ki?m tra kh? nãng trý?c khi g?i request.
- Impact: Provider c?n duy tr? metadata model trong API.

---

## Issue #5

- Raised by: Consumer
- Endpoint: POST /vision/detect
- Concern: Có th? c?n truy xu?t theo `correlationId` ð? giám sát.
- Proposal: Yêu c?u trý?ng `correlationId` ð? trace request.
- Resolution: Accepted
- Rationale: Correlation ID h? tr? ghi log và debug khi nhi?u camera cùng g?i request.
- Impact: Consumer cung c?p UUID, provider lýu theo request.

---

## Issue #6

- Raised by: Provider
- Endpoint: t?t c?
- Concern: Ngý?i nh?n c?n x? l? l?i theo chu?n chung.
- Proposal: Dùng RFC 7807 Problem Details cho t?t c? l?i 4xx / 5xx.
- Resolution: Accepted
- Rationale: Consumer d? parse và hi?n th? l?i nh?t quán.
- Impact: Provider thi?t k? schema `Problem` và tr? `application/problem+json`.

---

# Ch?t h?p ð?ng v1.0

Provider sign-off: AI Vision Team
Consumer sign-off: Camera Stream Team
Witness (GV/TA):
Date: 2026-08-11

---

## Ghi chú warning n?u Spectral c?n c?nh báo

| Warning | L? do ch?p nh?n t?m th?i | K? ho?ch s?a |
|---|---|---|
| | | |

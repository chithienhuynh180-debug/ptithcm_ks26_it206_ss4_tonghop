# Bài tập thực hành tổng hợp

## 1. Báo cáo thiết kế

Bài toán: cửa hàng trà sữa nhận đơn qua tin nhắn, cần tự tạo đơn hàng chuẩn.

- Input: tin nhắn khách, menu
- Output: đơn hàng JSON (món, size, số lượng, tổng tiền, trạng thái, lỗi, câu hỏi lại)
- Chuỗi 3 prompt chạy tuần tự, đầu ra bước trước là đầu vào bước sau: B1 trích xuất, B2 kiểm tra, B3 tạo đơn
- Chain-of-Thought: B2 và B3 ghi suy luận ngắn từng bước trước khi ra kết quả
- Xử lý đứt gãy: nếu bước nào trả sai định dạng thì thử lại 1 lần, vẫn sai thì trả "CẦN HỎI LẠI", không để chương trình crash
- Tối ưu token: prompt ngắn, chỉ truyền dữ liệu cần thiết sang bước sau, đầu ra chỉ JSON

## 2. Prompt hệ thống

**Prompt 1 (trích xuất)**

```
Đọc [TIN_NHẮN], trích xuất thành JSON: tên, sđt, địa_chỉ, món (tên, size, số_lượng). Thiếu thì để null, không đoán. Chỉ trả JSON.
```

**Prompt 2 (kiểm tra)**

```
Kiểm tra JSON với [MENU]. Suy luận từng bước, mỗi bước 1 dòng. Báo lỗi nếu: thiếu tên, sđt không đủ 10 số, thiếu địa chỉ, món hoặc size không có trong menu, số lượng không rõ hoặc bằng 0. Trả JSON gồm món_hợp_lệ và lỗi.
```

**Prompt 3 (tạo đơn)**

```
Tính thành tiền từng món hợp lệ và tổng tiền theo [MENU]. Có lỗi thì trạng_thái = "CẦN HỎI LẠI" và viết câu_hỏi_lại, không có lỗi thì "HỢP LỆ". Chỉ trả JSON, không lời chào, không lời kết.
```

**Code chạy chuỗi**

```python
import json

def chay_buoc(prompt, du_lieu):
    for _ in range(2):
        try:
            return json.loads(goi_ai(prompt + du_lieu))
        except Exception:
            pass
    return None

def xu_ly_don(tin_nhan, menu):
    b1 = chay_buoc(P1, tin_nhan)
    if b1 is None:
        return {"trạng_thái": "CẦN HỎI LẠI", "lỗi": ["không đọc được tin nhắn"]}
    b2 = chay_buoc(P2, json.dumps(b1) + menu)
    if b2 is None:
        return {"trạng_thái": "CẦN HỎI LẠI", "lỗi": ["không kiểm tra được đơn"]}
    b3 = chay_buoc(P3, json.dumps(b2) + menu)
    if b3 is None:
        return {"trạng_thái": "CẦN HỎI LẠI", "lỗi": ["không tạo được đơn"]}
    return b3
```

## 3. Kết quả chạy thử

Dữ liệu dị biệt: "2 trà đào size M, 1 cà phê sữa size XL, ship 5 Lê Lợi, sđt 09123"

**Kết quả bước 1**

```json
{"tên": null, "sđt": "09123", "địa_chỉ": "5 Lê Lợi",
 "món": [{"tên": "Trà đào", "size": "M", "số_lượng": 2},
         {"tên": "Cà phê sữa", "size": "XL", "số_lượng": 1}]}
```

**Kết quả bước 2**

- Thiếu tên khách
- sđt 09123 không đủ 10 số
- Cà phê sữa size XL không có trong menu

```json
{"món_hợp_lệ": [{"tên": "Trà đào", "size": "M", "số_lượng": 2}],
 "lỗi": ["thiếu tên khách", "sđt không đủ 10 số", "cà phê sữa size XL không có trong menu"]}
```

**Kết quả bước 3 (đầu ra cuối)**

```json
{"danh_sách_món": [{"tên": "Trà đào", "size": "M", "số_lượng": 2, "thành_tiền": 60000}],
 "tổng_tiền": 60000,
 "trạng_thái": "CẦN HỎI LẠI",
 "lỗi": ["thiếu tên khách", "sđt không đủ 10 số", "cà phê sữa size XL không có trong menu"],
 "câu_hỏi_lại": "Bạn cho mình xin tên và số điện thoại đầy đủ nhé. Cà phê sữa chỉ có size M và L, bạn chọn size nào."}
```

Dữ liệu sai nhưng hệ thống vẫn chạy hết chuỗi và ra kết quả sạch, không crash.

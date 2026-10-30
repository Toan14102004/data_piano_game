# Piano Game — Data

Kho dữ liệu bài hát của app Piano Game. App đọc trực tiếp các file ở đây qua
`raw.githubusercontent.com`, nên **repo này phải để public**.

Thêm bài mới ở đây là app tự thấy, không cần phát hành bản app mới.

## Cấu trúc

```
catalog.json          danh sách bài
songs/<id>.mid        file MIDI, mỗi bài một file
artworks/<id>.png     ảnh bìa (không bắt buộc)
```

## catalog.json

```json
{
  "version": 1,
  "songs": [
    {
      "id": "fur_elise",
      "title": "Für Elise",
      "artist": "Beethoven",
      "difficulty": "easy",
      "isPremium": false,
      "midiPath": "songs/fur_elise.mid",
      "artworkPath": "artworks/fur_elise.png"
    }
  ]
}
```

| Trường | Bắt buộc | Ghi chú |
|---|---|---|
| `id` | có | Không trùng nhau, không dấu, không khoảng trắng. Dùng làm khoá lưu tiến độ chơi — **đổi id là mất sao của người chơi**. |
| `title` | có | Tên hiển thị, có dấu thoải mái. |
| `artist` | có | Để `"Traditional"` nếu là dân ca / khuyết danh. |
| `midiPath` | có | Đường dẫn tính từ gốc repo. Phân biệt hoa thường. |
| `difficulty` | không | `easy` \| `medium` \| `hard` \| `expert`. Bỏ trống = `easy`. Quyết định tốc độ nốt rơi. |
| `isPremium` | không | Bỏ trống = `false`. |
| `artworkPath` | không | Bỏ trống thì app tự vẽ ảnh thay thế từ `id`. |

`version` để tăng khi nào đổi cấu trúc file; thêm bài thì giữ nguyên.

## Yêu cầu với file MIDI

- Standard MIDI File **format 0 hoặc 1** (`.mid`). Đuôi `.midi` phải đổi lại thành `.mid`.
- Nên tách giai điệu thành một track riêng — app tự đoán track nào là melody bằng
  cách lấy track có cao độ trung bình cao nhất, tách sẵn thì chính xác hơn.
- Tempo lấy từ meta event trong file, không cần khai trong `catalog.json`.
- Nốt ngắn dưới 60ms sẽ được kéo lên 60ms để vẽ được thành tile.
- Giữ file dưới ~100KB cho nhẹ; nhạc piano solo thường chỉ vài chục KB.

## Ảnh bìa

- PNG hoặc JPG, vuông, nên 512×512.
- Giữ dưới ~200KB, app tải ảnh mỗi lần mở danh sách (có cache).

## Kiểm tra trước khi push

```bash
python3 -m json.tool catalog.json > /dev/null && echo "JSON hợp lệ"
# mọi midiPath/artworkPath khai trong catalog đều phải tồn tại
python3 - <<'PY'
import json, os
c = json.load(open('catalog.json'))
ids = [s['id'] for s in c['songs']]
dup = {i for i in ids if ids.count(i) > 1}
if dup:
    print('ID TRÙNG:', dup)
for s in c['songs']:
    for k in ('midiPath', 'artworkPath'):
        p = s.get(k)
        if p and not os.path.exists(p):
            print('THIẾU FILE:', s['id'], k, p)
print('đã kiểm tra', len(c['songs']), 'bài')
PY
```

## Lưu ý khi cập nhật

`raw.githubusercontent.com` có CDN cache khoảng 5 phút. Push xong mà app chưa thấy
bài mới thì đợi vài phút rồi mở lại app.

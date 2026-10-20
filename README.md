# PixelSeal

**An image that looks the same, with a source record hidden inside its pixels.**

A local browser tool for adding and verifying an invisible source marker. Open `index.html` in recent Chrome or Edge. No upload, API, account, server or build step. Web Crypto requires a secure context; modern Chrome supports local `file://` pages. If unavailable, run `python -m http.server 8080` in this directory and open `http://localhost:8080`.

## 사용법

1. `index.html`을 크롬 또는 엣지에서 엽니다.
2. 이미지를 선택하고 제작자·출처를 입력합니다.
3. **새 키 생성 → 키 파일 저장**으로 비밀키를 보관합니다. 자동 보관되지 않습니다.
4. **출처 삽입 → PNG 다운로드**를 누릅니다.
5. 검증하려면 내려받은 PNG를 다시 선택하고 같은 키를 불러온 뒤 **출처 검증**을 누릅니다.
6. 키 없이도 출처 내용을 읽을 수 있지만 진위는 미확인입니다. 키는 공유하지 마세요.

삽입 전후 이미지를 나란히 보여줍니다. 완전히 불투명한 픽셀의 파란색 최하위 비트만 사용하므로 채널 값 차이는 최대 1/255입니다. 사람 눈에 차이가 잘 드러나지 않도록 설계했지만, 모든 이미지·디스플레이에서 지각적으로 동일하다고 보장하지 않습니다. 실제 픽셀은 다릅니다.

## What verification means

- `found`: a PixelSeal packet was parsed. Unauthenticated fields can be forged.
- `authenticated: true`: the supplied 256-bit secret key matches the HMAC-SHA256 source record. Anyone with the key can create a matching record. This is **not** public-key authorship verification.
- `intact: true`: dimensions and a normalized pixel hash match the signed record. The hash clears the blue LSB of fully opaque pixels. Changes confined to those bits outside the payload are intentionally not detected. It is **not** an exact image/file hash.
- `authenticated: null`: no key supplied; do not treat the displayed source as authenticated.

No result establishes legal ownership, creation priority or unauthorized use. The timestamp is the local computer time and is not independently certified. A removed or missing marker is not proof of independent creation. There is no web search or automatic infringement monitoring.

## Design and limits

Versioned length-prefixed UTF-8 source record + HMAC-SHA256, written sequentially into blue LSBs of fully opaque pixels. The record includes creator, source, random image ID, local timestamp, dimensions and normalized SHA-256 pixel hash. This is steganographic attribution, **not encryption** and not a new watermarking algorithm. The marker and source record are readable without a key.

This first version targets lossless PNG preservation. JPEG compression, resizing, cropping, screenshots, palette conversion, color management and deliberate attacks may destroy the marker. It is **not robust or removal-resistant**. Metadata stripping alone preserves it only if decoded pixels stay unchanged. Transparent and partly transparent pixels are not modified. PNG preserves their alpha. Input decoding may normalize color profiles or transparent pixel values; comparisons are to browser-decoded input pixels, not original file bytes.

Max input: 16 million pixels. Payload source limits: 200 UTF-8 bytes for creator, 500 for source. Images need several thousand fully opaque pixels; capacity errors are shown. Keys are never stored automatically or embedded in images/reports. Image data remains in the page's memory until reload or replacement. No external dependencies or network calls.

## Tests

```sh
node --test tests/*.test.cjs
```

Automated tests cover authenticated roundtrip, incorrect/missing keys, image tampering, malformed payloads, transparency, capacity, and per-channel bounds. Browser UI and actual browser PNG re-import have not been visually verified in the build environment. Test with a non-sensitive image before use.

MIT license; see LICENSE.

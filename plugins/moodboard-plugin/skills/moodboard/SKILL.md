---
name: moodboard
description: 핀터레스트에서 키워드로 이미지를 자동 수집해 지정 폴더의 raw/ 하위에 저장하는 스킬. Playwright MCP로 브라우저를 열고 JS로 URL을 누적 수집한 뒤 Python으로 다운로드한다. 키워드와 수집 장수(기본 30장)를 입력받는다.
---

# 무드보드 이미지 수집 스킬

## 목표
Playwright MCP로 핀터레스트를 열어 키워드 검색 후 이미지를 지정 장수만큼 수집하고
`~/{저장폴더}/{키워드}/raw/` 폴더에 저장한다.

## 입력
- `keyword` (필수): 수집할 컨셉 키워드 (예: "미니멀 베이지 카페")
- `count` (선택, 기본값 30): 수집할 이미지 수
- `save_dir` (선택, 기본값: `~/Desktop/moodboard`): 저장 상위 폴더

## 실행 단계

### 1. 저장 폴더 준비
```python
import os
keyword_safe = keyword.replace(' ', '')
save_path = os.path.expanduser(f"{save_dir}/{keyword_safe}/raw")
os.makedirs(save_path, exist_ok=True)
```

### 2. Playwright MCP로 핀터레스트 접속
```
URL: https://www.pinterest.com/search/pins/?q={keyword}
- 페이지 로드 대기: 3초
- 로그인 팝업 있으면 닫기 버튼 클릭 후 진행
```

### 3. JS로 이미지 URL 누적 수집 (스크롤 반복)
페이지에서 아래 JS를 실행해 URL을 전역 Set에 누적한다:
```javascript
// 전역 Set 초기화 (최초 1회)
window.__pinUrls = window.__pinUrls || new Set();

// 현재 화면 이미지 수집 함수
const collect = () => {
  document.querySelectorAll('img[srcset], img[src]').forEach(img => {
    if (img.srcset) {
      const parts = img.srcset.split(',').map(s => s.trim().split(' ')[0]);
      const best = parts.find(u => u.includes('736x')) || parts[parts.length-1];
      if (best && best.includes('pinimg.com')) window.__pinUrls.add(best);
    } else if (img.src && img.src.includes('pinimg.com')) {
      window.__pinUrls.add(img.src);
    }
  });
};

// 스크롤하며 수집 (1.2초 간격, 8회)
collect();
let step = 0;
const scroll = setInterval(() => {
  window.scrollBy(0, 1500);
  collect();
  step++;
  if (step >= 8) { clearInterval(scroll); window.__pinCollectDone = true; }
}, 1200);
```
- 약 12초 대기 후 결과 회수:
```javascript
() => [...window.__pinUrls]
```

### 4. Python으로 이미지 다운로드
수집된 URL 중 앞에서부터 `count`장만 다운로드:
```python
import urllib.request

headers = {'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7)'}
saved, failed = 0, 0

for i, url in enumerate(urls[:count]):
    try:
        req = urllib.request.Request(url, headers=headers)
        with urllib.request.urlopen(req, timeout=10) as r:
            data = r.read()
        with open(f"{save_path}/image_{i+1:03d}.jpg", 'wb') as f:
            f.write(data)
        saved += 1
    except Exception:
        failed += 1
```

### 5. 무결성 검증
```python
from PIL import Image
broken = []
for fname in os.listdir(save_path):
    try:
        Image.open(os.path.join(save_path, fname)).verify()
    except:
        broken.append(fname)
```

### 6. 결과 요약 출력
```
✅ 수집 완료
- 저장 위치: {save_path}
- 수집 장수: {saved}장 / 실패: {failed}장
- 깨진 파일: {len(broken)}개
- 키워드: {keyword}
```

## 실수 방지 규칙
- URL은 Set으로 관리 → 중복 자동 제거
- 다운로드 실패 시 건너뛰고 계속 진행 (전체 중단 없음)
- 수집된 URL이 count보다 적으면 가능한 최대치 수집 후 알림
- 이미 raw/ 폴더에 파일이 있으면 기존 파일 유지, 새 파일만 추가

## 주의사항
- 웹 검색(텍스트)이 아닌 Playwright MCP(브라우저)를 반드시 사용
- 핀터레스트는 가상 스크롤 사용 → 스크롤하며 URL을 누적 수집해야 함
- 이미지 직접 다운로드(URL)가 개별 스크린샷보다 빠르고 선명함

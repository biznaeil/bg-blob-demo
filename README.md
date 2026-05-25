# bg-blob-demo

Google Stitch 스타일의 비디오 배경 데모.

내일플러스 별도 사이트 적용 전 학습/실험용으로 만들어졌습니다.

## 데모 보기

GitHub Pages 배포: https://biznaeil.github.io/bg-blob-demo/

## 구성

- **bg.mp4** — 원본 8초 영상 (1280×720 H.264, 3.86MB). 좌우로 흐르는 네온 오로라.
- **poster.jpg** — 비디오 미로드/모바일/저전력 시 폴백 정적 이미지 (80KB, 4초 시점 추출)
- **index.html** — 4개 레이어 합성 (z:0 스크롤러+비디오 → z:1 비네트 → z:2 노이즈 → z:3 본문)

## 스크롤 모션

비디오 2개를 옆에 붙여놓고 컨테이너를 우→좌로 translateX:

| 시간 | 속도 (px/s) | 체감 |
|---|---|---|
| 0~30s | 30 → 600 (제곱 가속) | 점점 빨라짐 |
| 30s~ | 600 유지 | 최대속도 무한 스크롤 |

두 비디오는 동일 src + autoplay loop으로 같은 프레임 재생 → wrap 순간 이음새 안 보임. drift 방지용 1초 주기 currentTime 동기화 포함.

## 레이어 구조

```
z:3  ▶ 본문 (.content)
z:2  ▶ 노이즈 오버레이 (.noise)
z:1  ▶ 비네트 그라데이션 (.vignette)
z:0  ▶ 비디오 / 폴백 이미지 (.bg-video, .bg-fallback)
```

## 모바일/저전력 최적화

다음 조건에서 비디오 자동재생을 차단하고 poster 이미지만 사용:

- `(max-width: 768px)` — 소형 화면
- `navigator.connection.saveData === true` — 데이터 절약 모드
- `@media (prefers-reduced-data: reduce)` — OS 데이터 절약 설정
- `battery.level < 0.2 && !battery.charging` — 저배터리

터치 디바이스에서는 네온 트레일 캔버스도 비활성화 (마우스 없음).

## 포스터 재추출

```bash
ffmpeg -ss 00:00:04 -i bg.mp4 -vframes 1 -q:v 3 poster.jpg
```

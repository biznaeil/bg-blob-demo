# bg-blob-demo

Google Stitch 스타일의 비디오 배경 데모.

내일플러스 별도 사이트 적용 전 학습/실험용으로 만들어졌습니다.

## 데모 보기

GitHub Pages 배포: https://biznaeil.github.io/bg-blob-demo/

## 구성

- **bg.mp4** — 원본 8초 영상 (1280×720 H.264, 3.86MB). 소스 자산.
- **bg-loop.mp4** — 8초 → ping-pong 16초 → concat 2번 = **32초 매끄러운 무한 루프** (원본 속도 유지, 오디오 제거, 5.55MB). 배포용.
- **poster.jpg** — 비디오 미로드/모바일/저전력 시 폴백 정적 이미지 (70KB, 16초 시점 추출)
- **index.html** — 4개 레이어 합성 (z:0 비디오 → z:1 비네트 → z:2 노이즈 → z:3 본문)

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

## 영상 재생성 방법

```bash
# 8초 원본 → 32초 매끄러운 루프 (핑퐁 16s + concat 2번)
ffmpeg -i bg.mp4 \
  -filter_complex "[0:v]reverse[r];[0:v][r]concat=n=2:v=1[pp];[pp]split[a][b];[a][b]concat=n=2:v=1[v]" \
  -map "[v]" -c:v libx264 -crf 24 -preset slow \
  -movflags +faststart -an \
  bg-loop.mp4

# 포스터 추출 (16초 = 중간 지점)
ffmpeg -ss 00:00:16 -i bg-loop.mp4 -vframes 1 -q:v 3 poster.jpg
```

# bg-blob-demo

Google Stitch 스타일의 비디오 배경 + 마우스를 따라 출렁이는 별빛 파티클 트레일 데모.

내일플러스 별도 사이트 적용 전 학습/실험용으로 만들어졌습니다.

## 데모 보기

GitHub Pages 배포: https://biznaeil.github.io/bg-blob-demo/

## 구성

- **bg-loop.mp4** — ffmpeg 핑퐁 처리(forward + reverse concat)로 끝-시작 매끄러운 16초 루프, 오디오 제거 + faststart 최적화 (2.79MB)
- **poster.jpg** — 비디오 미로드/모바일/저전력 시 폴백 정적 이미지 (70KB)
- **index.html** — 5개 레이어 합성 (z:0 비디오 → z:1 비네트 → z:2 노이즈 → z:3 네온 캔버스 → z:4 본문)

## 레이어 구조

```
z:4  ▶ 본문 (.content)
z:3  ▶ 네온 트레일 캔버스 (#neon-canvas)
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

원본 영상(`bg.mp4`)에서 다시 핑퐁 처리하려면:

```bash
ffmpeg -i bg.mp4 \
  -filter_complex "[0:v]reverse[r];[0:v][r]concat=n=2:v=1[v]" \
  -map "[v]" -c:v libx264 -crf 24 -preset slow \
  -movflags +faststart -an \
  bg-loop.mp4

# 포스터 추출 (4초 지점)
ffmpeg -ss 00:00:04 -i bg-loop.mp4 -vframes 1 -q:v 3 poster.jpg
```

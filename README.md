# bg-blob-demo

1인칭 네온 터널 워프 효과 (3D 원근 투영 캔버스 애니메이션).

내일플러스 별도 사이트 적용 전 학습/실험용으로 만들어졌습니다.

## 데모 보기

GitHub Pages 배포: https://biznaeil.github.io/bg-blob-demo/

## 구성

- **index.html** — 3D 원근 투영 기반 캔버스 네온 터널. 시간에 따라 속도/밀도가 곡선으로 증가.
- **bg-loop.mp4, poster.jpg** — 이전 비디오 배경 모드의 자산. 현재 미사용 (코드에서 display:none + src 제거). 필요시 복원 가능.

## 시간 곡선 (네온 터널)

| 시간 | 속도 (z 감소율) | 파티클 수 | 체감 |
|---|---|---|---|
| 0~10s | 1.0 → 1.5 | 180 | 잔잔한 정찰 |
| 10~30s | 1.5 → 26.5 (제곱 가속) | 180 → 1200 | 점점 빨라짐 |
| 30s~ | 26.5 유지 | 1200 유지 | 워프 폭주 |

## 레이어 구조

```
z:5  ▶ 본문 (.content)
z:3  ▶ 노이즈 오버레이 (.noise)
z:2  ▶ 비네트 그라데이션 (.vignette)
z:1  ▶ 네온 터널 캔버스 (#tunnel-canvas)
hidden ▶ 비디오/폴백 (display:none, src 제거)
```

## 모바일/저전력 최적화

다음 조건에서 비디오 자동재생을 차단하고 poster 이미지만 사용:

- `(max-width: 768px)` — 소형 화면
- `navigator.connection.saveData === true` — 데이터 절약 모드
- `@media (prefers-reduced-data: reduce)` — OS 데이터 절약 설정
- `battery.level < 0.2 && !battery.charging` — 저배터리

터치 디바이스에서는 네온 트레일 캔버스도 비활성화 (마우스 없음).

## 영상 재생성 방법

원본 영상(`bg.mp4`)에서 다시 만들려면 2단계:

```bash
# 1단계: 8초 원본 → 16초 핑퐁 루프 (forward + reverse concat)
ffmpeg -i bg.mp4 \
  -filter_complex "[0:v]reverse[r];[0:v][r]concat=n=2:v=1[v]" \
  -map "[v]" -c:v libx264 -crf 24 -preset slow \
  -movflags +faststart -an \
  bg-loop-16s.mp4

# 2단계: 16초 → 30초 슬로다운 (setpts 1.875x + 24fps 출력)
ffmpeg -i bg-loop-16s.mp4 \
  -filter:v "setpts=1.875*PTS,fps=24" \
  -c:v libx264 -crf 24 -preset slow \
  -movflags +faststart -an \
  bg-loop.mp4

# 포스터 추출 (15초 지점 = 중앙)
ffmpeg -ss 00:00:15 -i bg-loop.mp4 -vframes 1 -q:v 3 poster.jpg
```

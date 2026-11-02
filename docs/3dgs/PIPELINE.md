# 실사 3DGS 베이스 파이프라인 (옵션 2)

실차 실내를 폰으로 촬영해 3D Gaussian Splatting으로 학습하고, 뷰어의
"실사 3DGS 베이스 열기" 버튼으로 불러오는 전체 절차. 비용: GPU 시간만
(Colab 무료 T4로 가능, 또는 RunPod 등 시간당 수백~수천 원).

## 1. 촬영 (10분)

- 차량 화면(센터 디스플레이) 끄기, 도어 모두 닫고 실내등 일정하게
- 폰 가로, 4K/30fps, 손 천천히 — 운전석→조수석→2열 순으로
  실내를 원을 그리며 2~3바퀴 (같은 장면을 다른 각도로 많이 겹치게)
- 1~2분 분량이면 충분. 반사 심한 유리·화면은 이후 크롭

## 2. 프레임 추출

    ffmpeg -i interior.mp4 -vf "fps=2" frames/img_%04d.jpg

(200~300장 권장)

## 3. COLMAP + 3DGS 학습 — nerfstudio (Colab 가능)

    pip install nerfstudio
    ns-process-data images --data frames/ --output-dir processed/
    ns-train splatfacto --data processed/ --max-num-iterations 15000
    ns-export gaussian-splat --load-config outputs/.../config.yml --output-dir export/

→ `export/splat.ply` 생성. 대안: gsplat 직접 사용, 또는 데스크톱 앱
Brush(https://github.com/ArthurBrussee/brush)는 COLMAP 없이도 동작.

## 4. 차량 외부 크롭 + SPZ 압축

    npx @playcanvas/splat-transform export/splat.ply -c out/interior.spz
    # (크롭 필터·중심 이동 옵션은 splat-transform --help 참고)

SuperSplat 웹 에디터(https://superspl.at/editor)에서 마우스로 외부
가우시안을 지우고 .ply/.spz로 다시 내보내는 방법이 가장 쉽다.

## 5. 뷰어 적용

1. 뷰어 첫 화면(차량 선택)의 **실사 3DGS 베이스 열기**로 .spz/.ply 로드
2. 뒤집혀 보이면 `상하 반전`, `↻ 90°`·`크기 ±`·`높이 ±`로 보정
3. 상품 장착 → 실사 배경 위 합성 확인 (계획서 5.3 실사감 평가용)

뷰어의 스플랫 렌더러는 Spark(@sparkjsdev/spark, MIT)를 사용한다.

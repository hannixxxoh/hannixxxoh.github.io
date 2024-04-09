---
title: "Camera Calibration(카메라 캘리브레이션)"
date: 2024-04-06 1:00:00 +0900
categories: [3D]
tags: [Camera, 3D]
---

본 내용은 다크프로그래머님의 블로그(https://darkpgmr.tistory.com/32)를 바탕으로 작성되었습니다.


# Introduction
- 우리가 실제 눈으로 보는 세상은 3차원, 이걸 카메라로 찍으면 2차원의 이미지
- 실제 이미지에는 카메라의 위치와 방향 같은 extrinsic parameter 외에도 사용된 렌즈, 렌즈와 이미지 센서와의 거리, 렌즈와 이미지 센서가 이루는 각 등 intrinsic parameter가 영향을 끼침
- 3차원 점이 이미지에 투영된 위치를 구하거나 역으로 이미지에서 3차원 좌표를 복원할 때는 이러한 내부 요인을 제거해야 함
- 이러한 intrinsic/extrinsic parameter를 구하는 과정이 **Camera Calibration**


## Camera Calibration
- 핀홀 카메라에서 3차원 공간의 점과 2차원 이미지의 점은 다음처럼 변환된다.
![image](https://github.com/hannixxxoh/product_serving/assets/91474981/54ca10d8-c3e7-451b-9269-7ac090b13c1b)
    - (x, y): 2차원 이미지의 좌표
    - (X, Y, Z): 월드 좌표계(world coordinate) 상의 3차원 좌표
    - A: intrinsic camera matrix
    - \[R\|t\]: 회전/이동변환 행렬, extrinsic parameter
    - A\[R\|t\]: camera matrix, projection matrix

![image](https://github.com/hannixxxoh/product_serving/assets/91474981/2ca2db06-76ad-4dd4-acf1-f394b4fcd9e3)


# Intrinsic Parameter
- 초점거리(focal length): fx, fy
- 주점(principal point): cx, cy
- 비대칭계수(skew coefficient): skew_c = $$\tan a$$

## focal length(초점 거리)
- 렌즈 중심과 이미지 센서와의 거리
- 카메라 모델에서 f(초점 거리)는 pixel(픽셀) 단위로 표현
    - 이미지의 pixel은 이미지 센서의 cell에 대응됨. 따라서 focal length가 pixel로 표현된다는 건, cell 크기에 대한 상대적인 값으로 표현된다는 의미.
    - 예를 들어 이미지 센서의 cell 크기가 0.1mm, 카메라의 focal length가 500 pixel 일 때, focal length는 50 mm.
- focal length를 f라 표현하지 않고 fx, fy로 구분하는 경우: 이미지 센서의 물리적인 cell 간격이 가로 방향과 세로 방향이 다를 때
    - fx는 focal length가 가로 방향 cell 크기의 몇 배인지
    - fy는 focal length가 세로 방향 cell 크기의 몇 배인지
    - 현대의 카메라는 대부분 f=fx=fy임
- 이미지 해상도를 1/2로 낮추면 focal length도 1/2로 작아짐
    - 실제 물리적 focal length는 같지만 한 pixel에 대응하는 물리적 크기가 변하기 때문에 카메라 모델에서의 focal length는 달라짐
    - 이미지 해상도를 1/2로 낮추면 이미지 센서의 2 x 2 cell이 합쳐져서 하나의 pixel이 됨. 한 픽셀에 대응하는 물리적 크기는 2배, focal length는 1/2.

### Normalized image plane, Image plane
![image](https://github.com/hannixxxoh/product_serving/assets/91474981/6b3d51b5-2a35-4ee0-8e3f-08c799f0db33)
- Normalized image plane
    - 초점으로부터 거리가 1(unit distance)인 평면
    - normalized image coordinate: normalized image plane의 좌표
    - camera coordinate의 한 점(Xc, Yc, Zc)를 image coordinate으로 변환할 때, 먼저 Xc, Yc를 Zc로 나누면 normalized image coordinate이 됨. 여기에 focal length를 곱하면 image coordinate이 되는데, image coordinate은 이미지의 중심이 아닌 이미지의 좌상단 모서리를 기준으로 하기 때문에 실제 최종적인 image coordinate은 principal point(cx, cy)를 더한 값임.
    - 수식으로 표현하면 $$x=f_{x}X/Z+cx, y=f_{y}Y/Z+cy$$
- principal point에 대한 자세한 설명은 다음 그림 참고.
![image](https://github.com/hannixxxoh/product_serving/assets/91474981/d7b92238-8fce-4b36-9ac0-e66f426ba15b)

## principal point(주점)
- 주점 cx, cy: 카메라 렌즈의 중심에서 이미지 센서에 내린 수선의 발의 영상 좌표(단위: 픽셀)

## skew coefficient(비대칭 계수)
- 이미지 센서의 cell array의 y축이 기울어진 정도
- skew_c = $$\tan a$$
- 요즘 카메라는 skew 에러가 거의 없기 때문에 대부분 skew_c = 0

# Extrinsic Parameter
- 카메라 좌표계와 월드 좌표계 사이의 변환 관계를 설명하는 파라미터
- 두 좌표계 사이의 회전(rotation)과 평행이동(translation)으로 푷ㄴ
---
title: "Axis Angle Rotation"
date: 2024-05-10 13:00:00 +0900
categories: [3D]
tags: [3D]
---

# Axis Angle Rotation

* $$R^2$$ 공간에서 벡터 i=(1,0), j=(0,1)이 있을 때, 벡터 i를 회전시 다음과 같이 나타낼 수 있다.


![image](https://github.com/hannixxxoh/hannixxxoh/assets/91474981/c505c8f1-87c5-4139-b59e-597aa75696a1)


* 다음으로 i, j 대신 기저벡터로 v_perp, w를 사용하고, 회전축은 A를 사용하여 v_perp을 회전시키면 다음과 같다.
    + v_prop: A축에 대해 projection
    + v_perp: A축에 대해 perpendicular

![image](https://github.com/hannixxxoh/hannixxxoh/assets/91474981/4c68eb9e-2b01-4523-a5db-c75c35e7d385)

* v의 수직 성분과 수평 성분을 더하면 회전된 v를 계산할 수 있다. v_prop은 회전축 A에 대하여 회전을 하여도 변하지 않게 된다.


![image](https://github.com/hannixxxoh/product_serving/assets/91474981/60ebf962-e775-4bc1-8e26-983e1a0185b1)

* v_prop: 회전축 A에 v를 projection
    + $$hat{A}$$: A의 unit vector

![image](https://github.com/hannixxxoh/product_serving/assets/91474981/cb4a93e0-a0ee-45bc-ad93-472896df8f7f)

* v_perp: v에서 수평 성분인 v_prop을 빼서 구함

![image](https://github.com/hannixxxoh/product_serving/assets/91474981/62394a21-c41b-4e8c-a369-748f79cf0c13)

* w: 회전축 A와 v의 외적

![image](https://github.com/hannixxxoh/product_serving/assets/91474981/233f4824-b41e-4932-a354-7102554e0319)

* v의 회전식을 지금까지 구해진 식으로 풀어서 적으면

![image](https://github.com/hannixxxoh/product_serving/assets/91474981/7f7d54e3-c49c-4fe7-8576-9f3802a84abd)

* v를 분리하기 위해 tensor product와 skew matrix를 사용하여 정리한다.


## tensor product
* 3개의 벡터 u, v, w가 있을 때 다음과 같은 식이 성립

![image](https://github.com/hannixxxoh/product_serving/assets/91474981/821ed36b-fecc-4b09-80e7-76a11399b48a)

![image](https://github.com/hannixxxoh/product_serving/assets/91474981/e7b75c3a-3554-44f3-ac0a-28657a186023)


## skew matrix
* 외적을 행렬로 표현하고 싶을 때 사용
* 벡터 v를 skew matrix로 표현한다.
* 그리고 여기에 벡터 w를 곱해주면 외적이 행렬로 표현된다.

![image](https://github.com/hannixxxoh/product_serving/assets/91474981/cb31b76e-ffb5-4559-9adb-73d6d9fcaeb7)

![image](https://github.com/hannixxxoh/product_serving/assets/91474981/79c8f190-628e-4549-80da-7e4690202708)

![다운로드](https://github.com/hannixxxoh/product_serving/assets/91474981/41bc7818-4aed-4495-b590-ebb43bea0c67)

* v의 회전식을 다시 정리하면
    * S: skew matrix of A
    * I: identity matrix

![image](https://github.com/hannixxxoh/product_serving/assets/91474981/f4984207-120b-4109-8dfe-a4a129dd7870)

![image](https://github.com/hannixxxoh/product_serving/assets/91474981/c2a25b73-cf24-4bef-aaed-eb66e4d935a3)

* 위의 식에서 벡터 v에 곱해지는 부분이 바로 벡터 v를 회전시키는 행렬 부분이 된다.
* 행렬 부분만 풀어 적게 되면 다음과 같이 된다.

![image](https://github.com/hannixxxoh/product_serving/assets/91474981/00fa4ba3-2852-440c-a0d2-d666e73f41a0)

![image](https://github.com/hannixxxoh/product_serving/assets/91474981/7896976f-de7b-4b46-b727-5b1cfcb68e66)

위의 행렬을 임의의 벡터 v에 곱하게 되면 임의의 회전 축 A를 기준으로 회전된 벡터 v를 구할 수 있다.




출처 <br>
https://mycom333.blogspot.com/2014/01/axis-angle-rotation.html
https://nobilitycat.tistory.com/entry/%EC%9E%84%EC%9D%98%EC%9D%98-%EC%B6%95-%ED%9A%8C%EC%A0%84-Axis-Angle-Rotation

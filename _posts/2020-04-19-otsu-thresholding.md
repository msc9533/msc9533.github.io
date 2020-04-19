---
layout: post
title: "Otsu's Method 구현하기"
description: Otsu's Method 구현하기
tags: Algorithm
date: 2020-04-19 20:22:04
comments: true
---

<!-- 무슨알고리즘인지 간략하게 -->
Otsu method는 1979년 Nobuyuki Otsu가 발표한 논문에 실려있는 방법입니다. 이진화는 0 이나 1로 변환하는 것을 말하는데, 영상처리에서 이진화는 영상에서 의미 있는 픽셀을 구분하는 방법중 하나로 많이 사용되고 있습니다.  

## Histogram

이진화 문제에서 0아니면 1로 구분하는 값을 threshold, 임계값이라고 합니다. 간단하게 영상에서 예제를 들겠습니다.  

![](https://github.com/msc9533/msc9533.github.io/raw/master/_files/otsu_test.jpg){: width="200" height="200"}  

- [원본이미지 다운로드](https://github.com/msc9533/msc9533.github.io/raw/master/_files/otsu_test.jpg)

위 이미지에서 물체가 있는 픽셀을 구분 하는 예제입니다. 이하 이미지는 grayscale을 가정하겠습니다. 이미지의 픽셀의 값은 byte단위의 숫자, 즉 0-255의 값을 가집니다. 이 픽셀값의 분포를 그래프로 나타낸 것을 Histogram이라고 합니다. 예의 이미지의 히스토그램은 다음과 같습니다.  

![](https://github.com/msc9533/msc9533.github.io/raw/master/_files/_200414Figure_1.png)  

픽셀의 값이 50 근처인 값과, 200근처인 값으로 나뉜다면 물체가 있는 영역을 쉽게 구할수 있게 됩니다.  

## Otsu's method
<!-- 핵심 알고리즘 원리 -->
<!-- 수식적으로 해석 -->
위의 그림에서 임계값은 약 125정도일때 적절하다는 것을 직관적으로 알 수 있습니다. 하지만 조명의 변화와 같이 전체 영상의 밝기에 영향을 미치거나, 배경이 바뀌는 등의 변화가 있을 경우 임계값도 다른 값을 선택해야 할 것입니다.

### Algorithm

Otsu method는 histogram에서 적절한 임계값을 선택하는 방법입니다. Otsu method에서는 이것을 between-class variance($\sigma^2_B$)가 최대가 되는 지점이라고 정의합니다.class 를 0 또는 1로 나누었을때 판별분석을 이용해 between-class variance와 within-class variance($\sigma^2_W$)는 다음과 같이 나타낼 수 있습니다.

$$
\sigma^2_W = \omega_0 \sigma^2_0 + \omega_1 \sigma^2_1 \\
\sigma^2_B = \omega_0 \omega_1 (\mu_1 - \mu_0)^2
$$

여기서 각각의 $\omega$는 확률, 즉 threshold($k$) 기준으로 나뉜 픽셀의 수 / 전체 픽셀의 수 ($N$)으로 구할 수 있고, 각 클래스의 mean은 조건부 확률이므로 

$$
\mu_0 = \sum_{i=1}^k ip_i / \omega_0 \\
\mu_1 = \sum_{i=k+1}^N ip_i / \omega_1
$$

와 같이 구할 수 있습니다. 

<!-- 직접구현 cpp? python? -->
<!-- 파생 알고리즘은 뭐가 있는지 -->
### 구현

threshold ($k$)를 모든 값에 대해 계산해 가장 높은 값을 찾는 방법으로 구현됩니다. 여기서 확률 w0,w1에 대해 w0 + w1 = 1입니다. 또한 w0,w1 둘중 하나라도 0일 경우는 어느 한쪽에 모두 포함 되는 경우이므로 제외합니다.

```py
import matplotlib.pyplot as plt
from PIL import Image
import numpy as np
# 원본 이미지 확인
img = np.array(Image.open('test.bmp').convert('L'))
plt.imshow(img,cmap='gray',vmin=0,vmax=255)
plt.show()

histo = []
def otsu(arg):
    # 입력 이미지의 픽셀수
    N = len(arg)*len(arg[0])
    # 히스토그램
    histogram = [0]*256
    for line in arg:
        for pixel in line:
            histogram[pixel] = histogram[pixel] + 1

    threshold = 0
    sumTotal = 0.0 
    sumB = 0.0
    w0 = 0
    w1 = 0
    varMax = 0.0
    # 전체 weighted sum calculation
    for i in range(len(histogram)):
        sumTotal += i*histogram[i]
    
    for i in range(len(histogram)):
        w0 += histogram[i]
        w1 = N - w0
        if w0 == 0:
            continue
        if w1 == 0:
            break
        # sigma_(i=1 to k) i*p_i
        sumB += float(i*histogram[i])
        # calc left m1, right m2 - mean
        m1 = sumB / w0
        m2 = (sumTotal - sumB) / w1
        #between-class variance
        varBetween = float(w0) * float(w1) * (m1 - m2) * (m1 - m2)
        if varBetween > varMax:
            varMax = varBetween
            threshold = i
    return threshold

thres = otsu(img)
for i in range(len(img)):
    for j in range(len(img[i])):
        if thres > img[i][j]:
            img[i][j] = 0
        else:
            img[i][j] = 255

plt.imshow(img,cmap='gray',vmin=0,vmax=255)
plt.show()
```

### 결과화면

![img](https://i.imgur.com/ApdjKRk.png)

---

<details>
<summary>참고문서</summary>
<div markdown="1">

- [wikipedia-Otsu's method](https://en.wikipedia.org/wiki/Otsu%27s_method)
- ["A threshold selection method from gray-level histograms",Otsu, N. (1979).](http://webserver2.tecgraf.puc-rio.br/~mgattass/cg/trbImg/Otsu.pdf)
- [Otsu 방법을 사용해서 이미지 이진화하기 (matlab 소스코드 포함)](https://bskyvision.com/49)
- [영상 이진화(binarization, thresholding)-다크프로그래머](https://darkpgmr.tistory.com/115)
- [히스토그램 - wikipedia](https://ko.wikipedia.org/wiki/%ED%9E%88%EC%8A%A4%ED%86%A0%EA%B7%B8%EB%9E%A8)

</div>
</details>
<script id="dsq-count-scr" src="//msc9533.disqus.com/count.js" async></script>


# Exponentially weighted averages

## Exponentially weighted averages

![Exponentially%20weighted%20averages%2074aa847775f443cca9373691ae853e1a/Untitled.png](Exponentially%20weighted%20averages%2074aa847775f443cca9373691ae853e1a/Untitled.png)

- 파란색이 런던의 1년간 날씨

> $v_t=\beta v_{t-1}+(1-\beta)\theta_t$

$if~~\beta = 0.9$

means that $\frac{1}{1-\beta}=\frac{1}{0.1}=10\approx$ 약 최근 10일간의 평균을 구한 것과 비슷하다

빨간 선

$if~~\beta = 0.98$

means that $\frac{1}{1-\beta}=\frac{1}{0.02}=50\approx$ 약 최근 50일간의 평균을 구한 것과 비슷하다

초록 선

$if~~\beta = 0.5$

means that $\frac{1}{1-\beta}=\frac{1}{0.5}=2\approx$ 약 최근 2일간의 평균을 구한 것과 비슷하다

노란색

> Intutition and understanding

1. 더 긴 기간을 평균을 냈으니 더 smooth해지는 것을 볼 수 있음
2. 그리고 $\beta$가 더 커질 수록 오른쪽으로 더 이동한 모습을 볼 수 있음
    - 변하는 온도를 더 늦게 반영하는 것. 약간의 latency가 있다.
    - 왜냐면 $\beta$가 커질 수록, $\theta_t$ 에 반영되는 weight가 $(1-\beta)$니까 더더 작아진다.
    - 그렇게 되면 변하는 현재의 온도가 average에 덜 반영되고, previous 온도가 더 많이 반영된다.
    - 즉 반영 되는데 시간이 더 걸린다는 것. 그래서 latency가 생기고 오른쪽으로 이동하는 모양이 나오는 것.
3. $\beta$ 가 작아질 수록 noise가 심해진다 → 더 짧은 기간의 평균을 내는 것이니까
    - outlier에 취약해진다
    - 하지만 기온의 변화를 빠르게 반영할 수 있다
4. 수학적 이해
    - 수식 설명

        ![Exponentially%20weighted%20averages%2074aa847775f443cca9373691ae853e1a/Untitled%201.png](Exponentially%20weighted%20averages%2074aa847775f443cca9373691ae853e1a/Untitled%201.png)

        여기서 각 $\theta_t$들의 weight를 다 더하면 1이 되거나 1에 근사한다. 그래서 가중평균이라고 하는 것.

        ![Exponentially%20weighted%20averages%2074aa847775f443cca9373691ae853e1a/Untitled%202.png](Exponentially%20weighted%20averages%2074aa847775f443cca9373691ae853e1a/Untitled%202.png)

        - 위의 그림이 t에 따른 $\theta_t$ 이고, 아래 그림이 t에 따른 weight를 그린 것
        - 두 vector를 elementwise product 하고 sum 하면 $v_{100}$ 를 구할 수 있음
    - 왜 $\frac{1}{1-\beta}$ days 동안의 온도의 평균과 비슷한 값을 가질까

        exponential weight 곡선을 볼 때, $\theta_{100}$ 에 해당하는 weight의 1/3 정도 되는 지점이 $(1-\epsilon)^{1/\epsilon}\approx0.35\approx1/e$ 가 되고, 그 이후로는 weight가 빠르게 줄어든다함

        $1/\epsilon$ 이 days가 되는 것 

        **솔직히 잘 이해 안간다. 정식 수학적 표현은 아니라고 한다**

### Implementing Exponentially weighted averages

$v_{\theta}=0$

$v_{\theta}:=\beta v_{\theta} + (1-\beta)\theta_1$

$v_{\theta}:=\beta v_{\theta} + (1-\beta)\theta_2$

...

---

$v_{\theta}=0$

repeat {

get next $\theta_t$

$v_{\theta}:=\beta v_{\theta} + (1-\beta)\theta_t$

}

이렇게 구하면 장점이 메모리가 덜 든다. 값 하나만 가지고 계속 더하고 빼고 하는거기 떄문에

근데 moving average를 해버리면 그 period의 값을 메모리에 다 올려야한다. 물론 moving averag를 하는게 좀 더 정확한 estimate를 얻을 수 있음

### bias correction

![Exponentially%20weighted%20averages%2074aa847775f443cca9373691ae853e1a/Untitled%203.png](Exponentially%20weighted%20averages%2074aa847775f443cca9373691ae853e1a/Untitled%203.png)

- 실제로 저 값을 구해보면, 초록선이 아니라 보라선처럼 되는데, 초반부가 굉장히 안맞는다.
- 그 이유는 이전 관측치가 없기 때문에, weighted average가 아니라 그냥 매우 작은 값이 나온다.
    - 그 예로 V1과 V2를 보면 weight의 합이 0.02, 0.04 정도밖에 안된다.
    - 그래서 이 bias를 제거해주어야 함
- 오른쪽에 보면 $v_t$에다가 $\frac{1}{1-\beta^t}$ 를 곱해주는데 이렇게 해주면 신기하게도 weight의 합이 1이 된다.
    - t가 커질수록 $\beta^t$는 0에 가까워지기 때문에 bias correction 영향이 줄어든다.
    - 즉 초반부에만 correction이 강하게 되고
- 사실 ML에서는 bias correction을 적용하는 것에 별로 신경을 안쓴다
- 초기 구간이 지나가길 기다리면 되기 때문에, 하지만 내가 초기구간이 중요하다면 bias correction을 사용하면 됨.
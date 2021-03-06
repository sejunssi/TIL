# 7. Ulysses' Compass

학자들은 더 간결한 이론을 선호하는 경우가 많다. 이 "선호"라는 것이 조금 애매하기는 해서, 단순히 아름답기 때문에 간결한 것을 좋아할 때도 있다. 
또는 단순한 이론일수록 다루기 쉽기 때문에 선호하는 경우도 있다. 
과학자들은 "가정이 더 적은 모형을 선호한다"는 **오캄의 면도날(Ockham's Razor)** 원칙을 자주 사용한다. 
그런데 정확도가 같은 두 모형이 있다면 단순한 모형이 더 좋다는 결론을 내릴 수 있겠지만, 일반적으로는 정확도와 단순함을 모두 고려하여 모형을 평가하게 된다. 
이 두 가지 기준을 어떻게 절충하여 사용할 수 있을까?

이번 장에서는 그러한 트레이드 오프를 다루기 위해 사용하는 도구들에 대해서 다룬다. 
이 도구들은 단순성에 대한 부분은 기능을 포함하고 있기 때문에 오캄의 면도날과 비교될 수 있다. 
하지만 명시적으로 단순함과 정확도 사이의 트레이드-오프를 이야기하는 오캄의 면도날과는 달리, 정확도를 높이기 위해서 사용하는 것이다.

따라서 여기서는 오캄의 면도날 대신 **율리시스의 나침반** 을 생각해보자. 율리시스는 호메로스의 오디세이에 등장하는 영웅이다. 
여행 중에 좁은 해협을 지나게 되었는데, 스킬라(Scylla) 라고 하는 머리가 여러 개 달린 괴물이 절벽 위에서 공격한다. 
바다에서는 카립디스(Charybdis) 라는 괴물이 배와 선원들을 물 아래로 끌어내리려 한다. 이 괴물들에 가까이 다가가면 공격당하기 때문에 잘 피해서 해협을 통과해야만 한다. 
과학적인 모형에 빗대어 보면 각각의 괴물들이 다음과 같은 두 가지 통계적인 오류를 나타낸다고 볼 수 있다.

1. **Overfitting** : 데이터를 너무 많이 학습하는 바람에 정확도가 낮아지는 경우
2. **Underfitting** : 데이터를 너무 적게 학습하는 바람에 정확도가 낮아지는 경우

그리고 세 번째 괴물로 바로 이전 장에서 다루었던 confounding 이 있다. 이번 장에서는 교란된 모형이 정상적인 모형보다 높은 정확도가 나올 수 있다는 것을 보여 줄 것이다.
따라서 특정한 통계 모형을 설계할 때는 정확도만 사용하는 것이 아니라 원인까지 고려해야 한다는 점을 확인해보자. 
인과 관계를 파악하기 위한 모형과 예측을 위한 모형은 다르기 때문에 애초에 다른 종류의 모형을 사용해야 할 수 있다. 
하지만 인과 관계를 파악하기 위해서라도 과적합(Overfitting) 을 다룰 수 있어야 한다. 

우리가 해야 될 작업은 이러한 괴물들을 조심스럽게 파악하는 것이다. 크게 두 가지 접근 방식이 있다. 
두 방식 모두 자연/사회과학 분야에서 널리 사용되고 있으며, 필요하다면 두 방식 모두 적용하기도 한다.

1. **Regularizing Prior** : 모형이 데이터에 너무 큰 영향을 받지 않도록 조절한다 
2. **Scoring** (Information Criteria, Cross Validation) : 예측 정확도를 수치로 표현한다

# 7.1 The problem with parameters

이전 장에서는 믿을만한 인과 모형이 없을 경우, 변수를 추가하는 것이 오히려 독이 되는 상황을 살펴보았다. 그런데 가끔은 인과 관계가 아니라 그냥 예측을 잘 하기 위한 모형을 만들어야 할 때가 있다. 이런 경우에는 그냥 변수를 추가하기만 하면 좋은 성능을 낼 수 있을까?

정답은 "아니오" 다. 변수를 추가하는 데는 두 가지 문제가 있다. 

1. 파라미터를 추가하면 거의 항상 모형이 데이터에 더 적합(fit)해진다
    - 여기서 적합(fit)하다는 것은 모형을 통해 학습에 사용한 데이터를 얼마나 원본에 근접하게 예측할 수 있는지를 의미한다
    - R-squared 가 이런 종류에 해당하는 대표적인 지표다
2. 모형이 복잡해져서 데이터를 더 잘 설명하게 되면, 새로운 데이터를 잘 예측하지 못하게 되는 경우가 생긴다
    - 파라미터가 많은 복잡한 모형은 단순한 모형에 비해 과적합되기 쉽다

간단한 예제를 통해 두 가지 케이스를 살펴보자.

## 7.1.1 More parameters (almost) always improve fit

과적합(Overfitting)은 모형이 샘플 데이터를 너무 과하게 학습할 때 발생한다. 

영장류 7가지의 평균 두뇌 크기와 중량 데이터를 살펴보자. 두 변수를 연결하는 가장 단순한 방법은 선형 관계를 가정하는 것이다. 

```r
library(rethinking)
library(tidyverse)

brain_mass_raw <- tibble(
    species = c("afarensis", "africanus", "habilis", "boisei", "rudolfensis", "ergaster", "sapiens"),
    brain = c(438, 452, 612, 521, 752, 871, 1350),
    mass = c(37.0, 35.5, 34.5, 41.5, 55.5, 61.0, 53.5)
)

# 변수를 표준화한다
# * brain 변수는 0을 기준점으로 삼기 위해 표준화하지 않는다
brain_mass <- brain_mass_raw %>% 
    mutate(mass_std = (mass - mean(mass)) / sd(mass),
            brain_std = brain / max(brain))

# 선형 모형을 구성한다
m_bm_01 <- quap(
    alist(
        brain_std ~ dnorm(mu, exp(log_sigma)),
        mu <- a + b * mass_std,
        # Prior for alpha : 말도 안되게 넓은 prior로 설정
        # * 가능한 값이 [0, 1]인데 89% 신용구간이 [-1, 2] 정도가 나옴
        a ~ dnorm(0.5, 1),
        # Prior for beta : flat prior
        b ~ dnorm(0, 10),
        log_sigma ~ dnorm(0, 1)
    ),
    data = brain_mass
)
```

그래프를 그리기 전에, R-squared에 집중해보자. 이 값은 전체 분산 중에서 모형을 통해 설명할 수 있는 비율을 의미한다. 
위에서 작성한 선형 모형의 R-squared 값을 구해보자.

```r
R2_is_bad <- function(data, quap_fit) {
    # 각 관측치에 대한 Posterior predictive distribution 을 구한다
    s <- sim(quap_fit, refresh = 0)
    # 모형의 잔차를 구한다
    r <- apply(s, 2, mean) - data$brain_std
    # 기존 var 함수는 frequentist 의 추정치이기 때문에 분모가 다르다
    # * 따라서 전통적인 방식으로 계산할 수 있도록 rethinking 라이브러리의 var2 함수를 쓴다
    # * var2 = mean(x^2) - mean(x)^2
    # R^2 = 1 - (Residual Variance / Outcome Variance)
    1 - rethinking::var2(r) / rethinking::var2(data$brain_std)
}

R2_is_bad(brain_mass, m_bm_01)
# [1] 0.4826186
```

이번에는 더 복잡한 모형들을 학습해 보자. 
파라미터의 수가 늘어날수록 R-squared가 증가하다가, 항이 6개가 되는 순간 R-squared의 최대치인 1이 되는 것을 볼 수 있다.

```r
#### 파라미터 2개로 학습 ####
m_bm_02 <- quap(
    alist(
        brain_std ~ dnorm(mu, exp(log_sigma)),
        mu <- a + b[1] * mass_std + b[2] * mass_std^2,
        a ~ dnorm(0.5, 1),
        b ~ dnorm(0, 10),
        log_sigma ~ dnorm(0, 1)
    ),
    data = brain_mass,
    start = list(b = rep(0, 2))
)

R2_is_bad(brain_mass, m_bm_02)
# [1] 0.5257412

#### 파라미터 6개로 학습 ####
m_bm_06 <- quap(
    alist(
        # 여기서 sd를 상수 0.001로 고정하지 않으면 모형이 작동하지 않는다
        brain_std ~ dnorm(mu, 0.001),
        mu <- a + b[1] * mass_std + b[2] * mass_std^2 +
            b[3] * mass_std^3 + b[4] * mass_std^4 +
            b[5] * mass_std^5 + b[6] * mass_std^6,
        a ~ dnorm(0.5, 1),
        b ~ dnorm(0, 10)
    ),
    data = brain_mass, 
    start = list(b = rep(0, 6))
)

R2_is_bad(brain_mass, m_bm_06)
# [1] 1
```

왜 6차 다항함수를 사용하면 완전히 적합해버리는 것일까? 왜냐하면 각 데이터 포인트 각각에 정확히 할당할 수 있는 파라미터 개수를 가지게 되었기 때문이다. 
6차 다항함수는 7개의 파라미터를 가지고, 우리는 7개의 데이터 포인트가 있다. 파라미터 수가 넉넉하면 모형은 데이터에 완전히 적합할 수 있다. 
하지만 그런 모형으로는 관측하지 못했던 부분에서 오히려 이상한 예측을 하게 된다.

## 7.1.2 Too few parameters hurts, too

모형이 과소적합(Underfitting)될 경우, 샘플에 있든 없든 부정확한 결과가 나온다. 너무 학습이 조금 되어서 샘플로부터 특징을 뽑아내는데 실패한 상태다. 또는 모형이 데이터에 너무 조금 반응하는 경우를 나타내기도 한다. 

# 7.2 Entropy and accuracy

과적합과 과소적합을 피해가려면 어떻게 해야 할까? **정규화(Regularization)** 또는 **정보 기준(Information Criteria)** 중 하나를 선택하더라도 
가장 먼저 해야 할 일은 모형을 평가하기 위한 기준을 정하는 것이다. 이 평가 기준을 **target** 이라고 해보자. 
이번에는 정보 이론이 어떻게 유용한 타겟을 제공하는지 살펴볼 것이다.

그렇지만 out-of-sample deviance를 구하는 것은 어렵다. 이를 위해서는 다음과 같은 작업들이 필요하다.

1. 완벽한 정확성과 얼마나 거리가 있는지 측정하는 척도를 세운다
2. 완벽한 정확성과의 상대적인 거리를 추정한 편차(Deviance)를 정의한다
3. 우리가 알아야 하는 것은 샘플을 벗어난 편차라는 것을 확인해야 한다

## 7.2.1 Firing the weatherperson

정확도는 평가 기준을 어떻게 정의하는지에 따라 달라진다. 평가 기준을 정의할 때는 크게 다음과 같은 요소를 고민해야 한다.

1. 틀렸을 때 얼마만큼의 비용이 발생하는가?
2. "정확도"의 맥락
    - 우리는 결합 확률(Joint Probability)을 측정해야 한다
    - 발생할 수 있는 사건들의 상대적인 비율을 측정하는 도구이기 때문이다
    - 모형이 실제 확률 분포와 같을 때 평균 확률 또는 결합 확률이 높아진다
    - 보통 결합 확률에 로그를 취해서 정확도의 척도로 사용하곤 한다 : **Log Scoring Rule**

## 7.2.2 Information and uncertainty

데이터의 로그 확률을 정확도의 기준으로 삼기로 했다. 그렇다면 완벽한 정확도와 얼마나 차이가 나는지 측정하는 것은 어떻게 해야 할까? 
이를 위해서는 예측이 목표로부터 얼마나 떨어져 있는지 거리를 측정할 수 있어야 한다. 여기에 어떤 "거리"를 사용해야 할까?

기본적인 컨셉은 다음과 같은 질문에서 시작한다. *결과를 하나 알게 된다면 불확실성이 얼마나 감소할까?* 이것을 바탕으로 정보를 다시 정의할 수 있다.

> 정보 (Information) : 결과를 알게 되었을 때 감소하는 불확실성의 정도

그리고 불확실성을 측정하기 위해 Information Entropy 를 사용한다. 다음과 같이 간단하게 정리할 수 있다. 
만약 비가 올 확률이 0.3, 맑을 확률이 0.7 이라면 `H(p) = -(0.3*log(0.3) + 0.7*log(0.7)) = 0.61` 이 된다.

```
H(p) = -E(log(p_i)) = - SUM_{i=1}^{n} p_i*log(p_i)
-> 확률분포의 불확실성은 사건의 로그 확률에 평균을 취하는 방식으로 구할 수 있다

* n : 가능한 사건의 수
* i : 각 사건
* p_i : 각 사건이 발생할 확률
```

엔트로피 만으로는 아직 무엇을 할 수 있는지 와닿지 않을 것이다. 엔트로피를 통해 정확도를 측정할 방법을 구해보자.

## 7.2.3 From entropy to accuracy

H를 통해 불확실성을 측정하는 방법을 알게 되었다. 정확하게는 목표를 맞히기가 얼마나 어려운지를 측정할 수 있게 되었다. 
그렇다면 모형이 목표로부터 얼마나 떨어져 있는지 측정하려면 어떻게 해야 할까? 발산(Divergence)을 통해 살펴보자.

> 발산 (Divergence) : 한 분포의 확률을 통해 다른 분포를 설명하고자 할 때 늘어나는 불확실성의 정도

이 방식을 제안한 사람의 이름을 따서 쿨백-라이블러 발산 (Kullback-Leibler Divergence) 또는 간단하게 KL Divergence 라고 한다.

실제 사건의 확률 분포가 `p1=0.3, p2=0.7` 이라고 해보자. 만약 우리가 확률이 `q1=0.25, q2=0.75` 일 것이라고 생각하고 있다면, 
q를 통해 p를 근사하면서 발생하는 불확실성이 얼마나 될까? 이에 대한 답은 H를 통해 계산할 수 있다.

```
D_KL(p, q) = SUM_{i}[ p_i * (log(p_i) - log(q_i)) ]
            = SUM_{i}[ p_i * log(p_i / q_i) ]

* Divergence는 target p와 모형 q 사이의 로그 확률의 차이로 평균을 구한 값을 말한다
* p=q 라면 D_KL(p, q) = 0 이다
```

발산(Divergence)을 통해 다양한 분포 중에서 목표와 가장 근사한 분포를 찾아낼 수 있다. 이를 통해 예측 모형의 정확도를 계산할 수 있다.

## 7.2.4 Estimating divergence

KL Divergence를 사용해서 모형을 비교하려면 목표로 하는 확률 분포(p)를 알아야 한다. 그 동안 예제에서는 p를 알고 있다고 가정했다. 
그런데 실제와 가장 비슷한 모형 q를 찾고자 할 때는 p를 직접 알 수 있는 방법이 없다. p를 알고 있다면 애초에 통계적 추론을 할 필요가 없을 것이다.

그런데 이런 문제를 피해갈 수 있는 방법이 있다. 우리가 하려는 것은 서로 다른 모형 후보 q와 r을 비교하는 것이다. 
이 경우에 p와 관련된 항이 대부분 소거된다. 따라서 p를 알지 못하더라도 q와 r이 얼마나 떨어져 있는지, 그리고 어떤 것이 목표와 더 가까운지 알 수 있다. 
이 계산을 위해서는 각 모형의 평균 로그 확률이 필요하다. 각 관측치에 대한 로그 확률을 더하기만 하면 `E[log(q_i)]` 의 근사치로 사용할 수 있다. 
하지만 그 수치만으로는 어떤 해석도 할 수 없다. 다른 모형의 값과 차이를 구했을 때만 각 모형이 목표 확률 분포인 p와 얼마나 떨어져 있는지 알고자 하는 목적으로 사용할 수 있다. 

실제로 사용할 때는 보통 모든 관측치에 대해 더해서 특정 모형에 대한 스코어를 구하는 경우가 많다. 
이런 스코어를 로그 확률 점수라고 하고, 서로 다른 모형의 예측 정확도를 구하기 위해 많이 사용한다.

```
S(q) = SUM_{i}[ log(q_i) ]

* 모형 q에 대한 Score값
```

베이지안 모형을 위해 스코어를 구하기 위해서는 모든 Posterior 분포를 활용해야 한다. 
실제 계산을 할 때는 몇 가지 미묘한 부분이 있기 때문에, rethinking 라이브러리에서는 lppd (Log Pointwise Predictive Density) 함수를 제공한다. 

```r
set.seed(123)
lppd(m_bm_01, n=1e4)
# [1]  0.6100859  0.6489015  0.5486945  0.6256770  0.4697209  0.4371187 -0.8548447
```

각각의 값은 관측치별 로그 확률 점수를 의미한다 (데이터에는 7개의 관측치가 있었다). 이 값을 모두 더하면 모형과 데이터에 대한 로그 확률 점수가 된다. 
이 값이 높을수록 더 높은 정확도를 가지고 있다고 볼 수 있다. Deviance(편차) 를 사용하는 경우도 있는데, lppd 에 -2를 곱한 값을 말한다. 
이 경우에는 값이 작을수록 정확한 모형이다.

## 7.2.5 Scoring the right data

로그 확률 점수는 목표 분포와 얼마나 떨어져있는지 거리를 재기 위한 가장 원칙적인 접근 방법이다. 
하지만 이 스코어는 R-squared와 동일한 문제가 있다. 모형이 복잡해지면 무조건 점수가 높아진다. 

```r
set.seed(123)
map_dbl(list(m_bm_01, m_bm_02, m_bm_06), ~ sum(lppd(.x)))
# [1]  2.495427  2.649391 39.481466
```

우리가 정말 알고자 하는 것은 새로운 데이터에서 나올 점수다. 어떻게 하면 새로운 데이터에서 점수를 높일 수 있을지 고민해보기 전에, 
샘플 안과 바깥 모두에서 점수를 구해보는 시뮬레이션을 해보자. 우리는 학습용 샘플에서 모형을 학습하고, 테스트용 샘플에서 결과를 측정할 것이다. 
이 사고 실험의 전체 프로세스를 정리해보면 다음과 같다.

1. 크기 N의 학습용 샘플이 있다고 가정한다
2. 학습용 샘플에서 모형을 작성하고 Posterior 분포를 구한다. 그리고 학습용 샘플에 대해 스코어를 구한다
    - 이 값을 `D_train` 이라고 한다
3. 다른 크기 N의 샘플(테스트용 샘플)에 대해 동일한 작업을 수행한다
4. 학습용 샘플을 통해 학습한 Posterior 분포를 평가용 샘플에 적용하여 스코어를 구한다
    - 이 값을 `D_test` 라고 한다

10000번 시뮬레이션 후에 파라미터 개수별 편차(Deviance, `-2 * lppd` , 따라서 작을 수록 높은 정확도를 나타낸다)를 비교해보자. 
학습용 샘플에서는 파라미터 수가 늘어날수록 편차가 작아지지만, 테스트용 샘플에서는 특정 파라미터 개수를 넘어가면 다시 편차가 커지기 시작한다. 

참고로 실제 데이터 생성 프로세스에서 테스트용 샘플 편차가 가장 작아질 것이라는 보장이 없다는 점을 주의해야 한다. 
편차(Deviance)는 모형의 정확도를 평가하기 위한 수단이지, 진실 여부를 판단하기 위한 값이 아니다.

# 7.3 Golem Taming : Regularization

과적합(Overfitting)은 학습에 사용한 샘플 데이터에 모형이 과도하게 반응하여 발생한다. 
Prior가 flat에 가까워지면 모형은 모든 파라미터에 동일한 가치를 부여한다. 따라서 이 경우 학습 데이터를 최대한 반영한 Posterior를 얻게 된다.

과적합을 막기 위한 방법 중 하나는 회의적인 Prior를 사용하는 것이다. 여기서 "회의적" 이라는 것은 샘플 데이터에서 배우는 속도를 늦추는 것을 말한다. 
회의적인 Prior 중에서 가장 일반적인 것은 정규화(Regularizing) Prior다. 

이전 장에서는 Prior Predictive 분포가 합리적인 결과를 낼 때까지 모형을 계속 수정해야 했다. 샘플 데이터의 수가 적을 때는 이러한 작업이 매우 도움이 된다. 

회의적인 Prior를 어느 정도의 강도로 설정할지는 데이터와 모형에 따라 다르다. 5개의 모형에 3가지 Prior를 각각 적용해서 1만번씩 시뮬레이션 해보자. 
샘플 사이즈 N=20 일 때는 학습 데이터에서 파라미터 수가 늘어날수록 Deviance가 계속 줄어드는 것을 볼 수 있다. 
특히 가장 회의적인 Prior를 썼을 때는 편차가 가장 커진다. 그런데 테스트용 데이터에서는 가장 좋은 성능을 보인다. 
**Prior가 점점 더 회의적일수록 모형이 복잡해지면서 발생하는 악영향을 덜 받게된다. 샘플 사이즈가 N=100 일 때는 Prior의 영향이 전체적으로 줄어든다.** 
증거가 많아졌기 때문이다. Prior가 도움을 주긴 하지만, 데이터가 Prior을 압도할 정도로 풍부한 정보를 얻을 수 있다. 
Prior를 정규화하는 것은 과적합을 억제하는데 도움이 되지만, 과할 경우에는 모형이 데이터로부터 학습하는 것을 방해할 수 있다.

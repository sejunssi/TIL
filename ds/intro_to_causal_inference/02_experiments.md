# Introduction

## Introduction to Experiments

상관관계가 있는 두 변수가 인과관계가 있는지 여부를 어떻게 알 수 있을까?

- 이베이 총판매량과 Sonoma County의 강수량은 높은 상관관계 (0.999) 를 보인다
- 이베이 총판매량과 애플 아이폰 판매량 사이에도 높은 상관관계 (0.994) 가 있다
- 두 경우 모두 높은 상관관계가 있지만 인과관계가 존재하는지는 알 수 없다

인과관계를 파악하는 것은 어렵다.

- 데이터를 많이 가지고 있지 않기 때문에, 그저 우연히 발생한 사건일 수도 있다
- 정확한 판단을 위해 실험을 하고싶어도, 비가 내릴지 말지를 우리가 결정할 수는 없다

## Controlled Experiments

이번에는 인과 관계를 학습하기 위한 다른 방법에 대해서 알아보자.
가장 오래되었으면서도 확실한 방법은 실험(**Experiments**)을 하는 것이다.

Unit Level Causal Effect에 대해서 생각해보자.

- 다른 모든 변수들이 고정되어 있을 때 Treatment 그룹과 Control 그룹의 결과값 차이를 말한다
- Unit Level Causal Effect 를 찾는 가장 쉬운 방법은 **treat 여부만 다르고 나머지는 모두 동일한 2개의 unit을 찾아서 비교**하는 것이다
- 이게 바로 **Controlled Experiments** 의 컨셉이다
- 광고에서 종종 나오는, 자사 제품과 타사 제품을 직접 비교하는 것이 controlled experiment의 일종이다

만약 모든 변수를 완벽하게 통제할 수 있다면 단 2개의 관측값만 가지고도 causal effect를 구할 수 있다.
실제 문제에서는 더 많은 정보들을 수집하겠지만, 인과관계를 확인하는 측면에서 중요한 것은 한 unit은 treat되고 다른 한 unit은 treat 되지 않아야 한다는 점이다

하지만 사회과학의 문제들에는 Controlled Experiments 를 적용할 수 없다.

1. 변수를 측정하는 것이 너무 어렵다
    - 우선, 목표로 하는 변수를 측정하는 것이 어렵다
    - 그리고 다른 변수들을 모두 동일하게 유지해야 하는데, 동일한지 아닌지 정확하게 판단하기 어렵다
2. 고려해야 할 변수들이 너무 많다
    - 각각의 변수들을 모두 측정한다고 하더라도 동일한 조건으로 맞추기가 어렵다

위와 같은 문제점들로 인해 Controlled Experiments 를 사용할 수 없다. 그러면 어떻게 해야 할까?

이런 상황에서는 **Randomized Experiments** 를 사용해야 한다.


# Randomized Experiments & Statistical Inference

## Randomized Experiments

**Randomized Experiments** 는 많은 수의 unit이 있을 때 treatment 또는 control 그룹으로 랜덤하게 나누는 방식의 실험을 말한다.
treatment 그룹에는 treat 된 unit들만, control 그룹에는 treat 되지 않은 unit 들만 존재한다.

이런 방식의 실험이 인과관계를 밝히는데 어떻게 도움이 될까?

- **Controlled Experiments** 에서는 두 unit이 treat 여부를 제외한 모든 부분에서 동일해야 한다
- **Randomized Experiments** 에서는 treat 여부를 결정하기 이전에 **모든 unit을 랜덤하게 배치** 한다
    - 두 그룹에 unit을 랜덤하게 배치하고 그룹의 규모가 크다면, 두 그룹은 서로 비슷해진다
    - 이렇게 하면 두 그룹의 특성이 정확하게 동일하다고 볼 수는 없지만, 각 그룹의 평균은 같아질 것이다
- 정말 랜덤하게 잘 배치되었다면, 두 그룹의 밸런스가 유지된다
    - 예를 들면, 특정 그룹에는 여성이 더 많이 속해있다거나 하는 일이 발생하지 않게 된다

우리가 다룰 변수들은 세 가지 종류가 있다

- **Outcome** Variables
- **Treatment (Policy)** Variables
- **Pretreatment** Variables : treatment 반영하기 이전에 존재했던 변수들

Randomization은 pretreatment variable 들이 균형잡히게 나누어지도록 배치한다.
treatment variable에는 적용되지 않는다. treatment variable 들은 treat 여부에 따라 그룹이 달라진다.

따라서 **Randomized Experiments 는 다음과 같은 효과** 를 갖는다.

- Randomization 을 통해 분리된 두 그룹은 모든 pretreatment variable이 평균적으로 동일하다
- 두 그룹은 treatment 여부만 다르다
- 이러한 상황에서는 **어떤 평균적인 차이가 발생할 경우, treatment 여부에 따른 결과** 라고 볼 수 있다
- *"The treatment has a causal effect on outcomes."*

Randomized experiments 를 가지고 인과 관계가 있다고 말하기 위해서는 각 그룹에 많은 unit이 있어야 한다.

- Controlled experiments 에서는 2개의 unit만 있으면 된다
- 하지만 Randomized experiments 의 경우 unit의 수가 적으면 극단적인 값이 있을 때 평균에 큰 영향을 미칠 수 있다
- 따라서 두 그룹이 평균적으로 비슷해지기 위해서는 각 그룹의 크기가 충분히 커져야 한다

실제로 활용하기 위해서는 각 그룹의 크기가 어느 정도 되어야 하는지 궁금할 것이다.

- 잘 알려진 통계적 기법들을 활용해 정확한 수치를 구할 수 있다
- 여기서 직접 해당 기법들을 다루지는 않겠지만, 인과관계를 밝히기 위해서는 많은 데이터가 필요하다는 것을 명심하자

Randomization 기법을 사용할 경우 이후의 분석 작업이 굉장히 간단해진다는 장점이 있다.

- treatment 그룹과 control 그룹 간의 평균의 차이를 계산하면 바로 ATE를 구할 수 있다
- ATE = Average Value of Treatment Outcome - Average Value of Control Outcome

## p Values, Confidence Intervals, and Hypothesis Tests

신뢰구간은 P-value, 가설 검정과 어떻게 연관되어 있을까? 다트 게임을 통해 알아보자.

- Control Group 은 일반 다트를 사용한다
- Treatment Group은 레이저 조준경을 사용한다
- 일단 두 그룹의 평균만 놓고 보면 Treatment Group의 평균이 중앙과 더 가까운 것을 확인했다
- 그렇다면 레이저 조준경을 사용하는 것이 높은 득점을 올리는데 도움이 되었을까?

다트를 반복해서 던지다 보면 **다트가 중앙에서 얼마나 떨어져있는지에 대한 분포** 를 구할 수 있다.
CLT (중심극한정리) 에 의해 해당 분포가 정규분포의 형태를 따른다는 것을 알 수 있다.

- **전통적인 가설 검정(Hypothesis Test)** 에서는 다음과 같은 형태로 효과를 확인한다
    - Treatment로 인해 발생한 실제 효과가 없다고 (0이라고) 가정한다
    - 실험을 여러번 반복할 경우 두 그룹간 평균의 차이는 어떤 형태의 분포를 따르게 될까?
    - CLT에 의해 정규분포를 따를 것이므로 95%의 확률로 0 부근에 존재할 것이다
    - 정확히는 [-1.96, 1.96] 구간 안에 95% 확률로 존재한다
    - 만약 해당 구간을 넘어간다면, 평균의 차이가 0이라는 기존의 가설을 기각할 수 있을 것이다
- 이제 이 과정을 **P-value** 를 이용해 진행해보자
    - 평균의 차이가 0일 때, 데이터를 통해 계산한 ATE 값이 나올 수 있는 확률을 계산한다
    - 이것을 P-value 라고 한다
    - 따라서 P-value가 0.05 보다 작을 경우 효과가 없다는 귀무가설을 기각한다
- **신뢰구간(Confidence intervals)** 을 통해서도 비슷한 테스트를 할 수 있다
    - treatment에 따른 차이가 없을 때를 가정한 분포가 존재한다
    - 실험을 여러 번 반복하다 보면 ATE를 중심으로 하는 분포로 이동한다
    - 이동한 분포를 기준으로 95% 신뢰구간을 구한다 : `[ATE - 1.96*SE, ATE + 1.96*SE]`
    - 신뢰구간이 0을 포함하지 않을 경우, 귀무가설을 기각할 수 있다

세 가지 방식 모두 기본적인 컨셉은 비슷하다.
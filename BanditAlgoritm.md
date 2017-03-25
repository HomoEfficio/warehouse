# Multi-armed Bandit Algorithm

## 용어

### Reward

A quantitative measure of success. In business, the ultimate rewards are profits, but we can often treat simpler metrics like click-through rates for ads or sign-up rates for new users as rewards. What matters is that (A) there is a clear quantitative scale and (B) it’s better to have more reward than less reward.

정량적으로 측정 가능한 포상, 적게 받는 것보다 많이 받는 것이 좋은 어떤 것

### Arm

What options are available to us? What actions can we take? In this book, we’ll refer to the options available to us as arms. The reasons for this naming convention will be easier to understand after we’ve discuss a little bit of the history of the Multiarmed Bandit Problem.

취할 수 있는 옵션

### Bandit

A bandit is a collection of arms. When you have many options available to you, we call that collection of options a Multiarmed Bandit. A Multiarmed Bandit is a mathematical model you can use to reason about how to make decisions when you have many actions you can take and imperfect information about the rewards you would receive after taking those actions. The algorithms presented in this book are ways of trying to solve the problem of deciding which arms to pull when. We refer to the problem of choosing arms to pull as the Multiarmed Bandit Problem.

옵션의 모음. 옵션이 많을 경우 멀티 암드 밴딧이라고 한다.
Multiarmed Bandit은 여러 조치를 취해볼 순 있지만, 그 조치에 뒤따르는 reward에 대한 정보가 불확실할 때 어떻게 결정을 내리는지를 고민하는데 사용되는 수학적 모델 

### Play/Trial

When you deal with a bandit, it’s assumed that you get to pull on each arm multiple times. Each chance you have to pull on an arm will be called a play or, more often, a trial. The term “play” helps to invoke the notion of gambling that inspires the term “arm”, but the term trial is quite commonly used.

밴딧을 다룰 때는 각 arm을 여러번 당길 수 있다. 한 번 당기는 것을 play 또는 trial이라 한다.

### Horizon

How many trials will you have before the game is finished? The number of trials left is called the horizon. If the horizon is short, you will often use a different strategy than you would use if the horizon were long, because having many chances to play each arm means that you can take greater risks while still having time to recover if anything goes wrong.

arm을 당길 수 있는 잔여 회수. horizon이 많으면 더 위험을 더 감수할 수 있으므로 horizon에 따라 다른 전략을 취하게 된다.


### Exploitation

An algorithm for solving the Multiarmed Bandit Problem exploits if it plays the arm with the highest estimated value based on previous plays.

앞서 수행한 탐색을 토대로 이익을 쓸어담기

### Exploration

An algorithm for solving the Multiarmed Bandit Problem explores if it plays any arm that does not have the highest estimated value based on previous plays. In other words, exploration occurs whenever exploitation does not.

앞선 play에서의 결과를 토대로 할 때 가장 많은 이익을 주지 않는 arm을 당기는 것. exploitation이 아니면 exploration이다. 

### Explore/Exploit Dilemma

The observation that any learning system must strike a compromise between its impulse to explore and its impulse to exploit. The dilemma has no exact solution, but the algorithms described in this book provide useful strategies for resolving the conflicting goals of exploration and exploitation.

정보의 확실성을 높이기 위해 exploration을 많이 하면 이익을 쓸어담는 exploitation 기회가 줄어들고,
이익을 쓸어담는 기회를 늘리기 위해 exploitation을 많이 하면 exploration이 줄어들어 정보의 정확도가 줄어든다.  

### Annealing 

An algorithm for solving the Multiarmed Bandit Problem anneals if it explores less
over time.

시간이 지남에 따라 explore를 줄여가는 것을 annealing 담금질 이라 한다.

### Temperature

A parameter that can be adjusted to increase the amount of exploration in the Softmax algorithm for solving the Multiarmed Bandit Problem. If you decrease the temperature parameter over time, this causes the algorithm to anneal.

softmax 알고리듬에서 exploration의 양을 조절하는데 사용되는 파라미터, temperature를 지속적으로 줄이면 알고리듬은 점점 annealing 된다.


### Streaming Algorithms

An algorithm is a streaming algorithm if it can process data one piece at a time. This is the opposite of batch processing algorithms that need access to all of the data in order to do anything with it.

배치 알고리듬과 다르게 한 번에 하나의 데이터만 처리하는 알고리듬

### Online Learning

An algorithm is an online learning algorithm if it can not only process data one piece at a time, but can also provide provisional results of its analysis after each piece of data is seen.

한 번에 하나의 데이터를 처리하는 것 뿐아니라, 각 데이터의 처리 후 예상 결과를 제공하는 알고리듬

### Active Learning

An algorithm is an active learning algorithm if it can decide which pieces of data it wants to see next in order to learn most effectively. Most traditional machine learning algorithm are not active: they passively accept the data we feed them and do not tell us what data we should collect next.

가장 최적화 된 습득을 위해 다음에 어떤 데이터를 득하는 것이 좋은지 결정할 수 있는 알고리듬.
주어진 데이터를 읽는 일반적인 머신 러닝은 active learning이 아니다.


### Bernoulli

A Bernoulli system outputs a  1 with probability  p and a  0 with probability  1 – p

가능성이 p이면 1을 출력 가능성이 1-p 이면 0을 출력하는 시스템 


## A/B 테스트의 단점

- Exploration 단계에서 Exploitation 단계로 부드럽게 전환되지 않고, A와 B 중 A가 더 나은 안이라고 결론지어지면 B를 바로 폐기하는 방식으로 단절되어 진행된다.

- B가 불리하다는 점을 확인하기 위해 소요된 자원은 매몰된다.

## Epsilon-Greedy Algorithm

- epsilon의 비율로 exploration 하고, 1-epsilon의 비율로 exploitation 한다.
    - exploration하게 되면 반은 역사적인 best arm으로 exploration하고, 나머지 반은 worst arm으로 exploration 한다.

- epsilon이 1.0이면 exploration만 하므로 사실 상 A/B 테스트와 같아진다.
- epsilon이 0.0이면 exploitation만 하므로 사실 상 현재 안 유지와 같아진다.

### 평균 구하는 새로운 방법

- 기존 : sum(n) / n
- 살짝 변형 : n - 1 개 까지의 평균 a을 알고, n 번째의 값 v를 알 때 전체 n의 평균 Result

    ```    
    Result = sum(n) / n
    Result = sum(n - 1) / n + v / n
    Result = sum(n - 1) / (n - 1)   *   (n - 1) / n   +   v / n
    a = sum(n - 1) / n -1

    Result = a * (n - 1) / n   +   v / n
    ```

### epsilon-greedy 알고리듬의 단점

- n 가지 안이 존재한다면, 언제나 epsilon * (n - 1)/n 만큼은 좋지 않은 안임에도 불구하고 계속 exploration을 하는 낭비 발생
- 두 가지 안이 존재하고 보상률이 각각 10%, 13% 인 시나리오 A와 10%, 99% 인 시나리오 B가 있을 때, A와 B는 보상률의 차이가 크다는 특징이 있음에도 불구하고 epsilon-greedy 알고리듬 상에서는 똑같은 로직이 적용됨
    - 즉, 시나리오 B에서는 각 arm 별 보상률의 차이가 크므로 10%인 arm에 대해 exploration을 많이 할 필요가 없다. 
    - 하지만 시나리오 A에서는 각 arm 별 보상률의 차이가 크지 않으므로 어느 안이 좋은 안인지 확실히 알기 위해 더 많은 exploration을 하는 것이 타당하다.


# MAB의 장점 over A/B Test
- 무작위로 진행되는 A/B 테스트에 비해 exploration에 의해 발생하는 기회비용을 줄일 수 있다.
  - 이론적으로는 그렇지만, 현실적으로는 MAB를 제대로 이해해야만 MAB로 기회비용을 줄일 수 있다는 견해도 있음(https://www.chrisstucchio.com/blog/2015/dont_use_bandits.html)

# 책의 전체 내용 흐름
- n 개의 대안들의 효용(conversion rate)을 이미 알고 있다고 가정하고,
- 그중에 가장 나은 안을 S(superior)라고 가정하면,
- S를 포함한 n 개의 대안들과,
- epsilon, temperature 등 알고리듬 별 특성치를 바꿔가면서 입력값으로 해서
- epsilon-greedy, softmax, UCB1 알고리듬을 돌리고,
- 어떤 알고리듬이 더 짧은 시간에, 더 안정적으로, S안을 선택하는 것이 낫다는 결론을 도출해주는가를 비교

# 궁금한 점
- 결국 어떤 안이 가장 나은 안 인지를 가장 적은 비용으로 알아낼 수 있는 알고리듬을 찾아내서 최종 효용을 극대화 하자..는 것이 목적인 것 같습니다.
- 하지만, 대안 들의 특성에 따라 적합한 알고리듬 및 설정값이 달라질 수 있을 것 같은데요,
  . 예를 들면 A, B, C, D 안의 효용이 각각 0.1, 0.1, 0.1, 0.9 일때 적합한 알고리듬 및 설정값과
    효용이 각각 0.3, 0.4, 0.6, 0.8 일때 적합한 알고리듬 및 설정값이 달라질 수 있다.. 라는 생각인데
- 대안 들의 특성을 몰라서, 그것을 알아내는데 가장 비용이 적게 드는 최적의 알고리듬을 찾자는 목적과,
  - 대안 들의 특성을 알아야만 최적의 알고리듬을 알아낼 수 있다는 현실이 서로 상충되는 게 아닌가.. 하는 의문이 들면서부터 진도가 잘 안나가고 있습니다.
- 이 의문이 제가 뭔가 잘못 이해하고 있는 부분에서 나온 거라면 어떤 부분을 잘못 이해하고 있는 것인지 의견 나누고 싶고요,
  - 이 의문이 타당한 의문이라면 이 문제를 어떻게 해결할 수 있는지 궁금합니다.
# Adaptive Residual Control Strategy through the Integration of MPC and RL

## Problem Definition: Limitations of MPC and RL and the Necessity of Their Integration
In continuous physical control problems, MPC and RL each have their own strengths and weaknesses, but their limitations become evident when used independently. Model Predictive Control (MPC) demonstrates high stability and performance by utilizing a given system model to predict future behavior and compute optimal control inputs. However, it heavily depends on an accurate environmental model and is vulnerable to model uncertainty and dynamic environmental changes. Even slight inaccuracies in the model can degrade MPC's performance, and in complex environments, the real-time computational cost becomes significant. <br>

In contrast, **Reinforcement Learning (RL)** learns from experience through interaction with the environment, enabling it to adaptively discover optimal policies even without an accurate model. This makes RL robust in responding to unpredictable and dynamic situations. However, RL suffers from inefficiencies during the initial exploration phase and may exhibit unstable behavior, posing risks during training. It also requires a vast amount of data to achieve satisfactory performance. Applying RL directly to real-world robots or engineering systems can raise safety concerns due to its trial-and-error learning process. In this regard, RL remains vulnerable compared to classical control methods in terms of data efficiency and stability. <br>

Given these contrasting characteristics, combining MPC and RL to leverage their respective strengths and mitigate their weaknesses has gained significant attention. For instance, prior studies have reported approaches such as Residual Reinforcement Learning, where an RL agent learns residual actions on top of a baseline controller (e.g., MPC or PID feedback control), thereby enhancing overall performance. In real-world robotic control, the integration of traditional controllers with RL has been shown to improve sample efficiency and maintain safety while effectively learning complex tasks. <br>

However, in conventional Residual RL architectures, the output of the RL policy is added to the baseline controller in a uniform manner (e.g., simple summation) across all states. In other words, the contribution ratio between MPC and RL is fixed and does not adapt to changes in environmental states. Such a static combination can be inefficient in certain scenarios. For example, in some states, the control provided by MPC alone may be sufficient and near-optimal, so adding residual actions from RL might only introduce unnecessary noise. Conversely, in other situations, more aggressive intervention from RL might be needed. Without a state-dependent adaptive combination strategy, such mismatches are difficult to address. <br>

The Adaptive Residual architecture originates from this very concern. The proposed solution in this project is to use MPC as the initial policy while adaptively adjusting the contribution of RL-based residual control through a state-dependent weighting factor, $\lambda(s)$. This approach aims to achieve an optimal combination tailored to each situation by dynamically modulating the influence of RL based on the state $s$. By doing so, the goal is to integrate the stability of MPC with the learning capabilities of RL on a per-state basis, thereby overcoming the limitations of both methods in continuous control problems.<br>

## Contribution: Implementation and Evaluation of Adaptive Residual RL in PyRocketCraft
In this project, the above Adaptive Residual RL concept was implemented and evaluated using the PyRocketCraft simulator, resulting in the following contributions: <br>

+ Implementation of the Adaptive Residual Control Architecture: I designed and implemented an Adaptive Residual algorithm that integrates an MPC-based control policy with an RL-based residual policy in the open-source rocket landing environment, PyRocketCraft. PyRocketCraft uses the PyBullet physics engine to simulate 3D rocket landings and originally features a built-in nonlinear MPC (NMPC) controller. In this project, I extended the system by incorporating a reinforcement learning agent that corrects the output of the MPC through a residual policy, and dynamically combines the two based on the current state.
+ Comparative Experiments and Performance Analysis: To validate the effectiveness of the implemented Adaptive Residual method, I conducted comparative experiments using three control strategies under the same environment: (1) MPC only, (2) baseline MPC+RL Residual, and (3) the proposed MPC+RL Adaptive Residual approach. Reinforcement learning training was performed for each method, and the final performance metrics (final rewards) were measured and analyzed. Through this process, I evaluated the advantages and limitations of the Adaptive Residual structure compared to existing approaches, with particular attention to monitoring the role of the state-based weighting factor $\lambda(s)$.
+ Examination of Residual Weight Behavior: I investigated and interpreted the behavior of the adaptive weight $\lambda(s)$—the core component of the Adaptive Residual method—through experimental analysis. By examining the distribution of $\lambda(s)$ values and identifying which states exhibited greater or lesser reliance on RL residuals (relative to MPC), I assessed whether the proposed method achieved state-dependent control blending. Furthermore, I evaluated how these dynamic combinations influenced overall performance.

## 참신성: 상태 기반 Adaptive Residual 결합의 도입
제안하는 Adaptive Residual 구조의 가장 큰 차별점(참신성)은 상태에 따라 달라지는 MPC-RL 결합 비율을 도입했다는 것입니다. 기존의 residual RL 접근법에서는 RL이 기존 제어기의 행동을 일률적으로 보정하는 데 그쳤다면, 본 연구에서는 상황에 따라 보정 정도를 달리할 수 있도록 설계했습니다. 이를 구현하기 위해 상태 의존적 가중치 함수 $\lambda(s)$를 두어, 매 시점 상태 $s$에서 MPC 출력과 RL 잔차 출력을 가중 합하도록 만들었습니다. 최종 제어 입력 $u_{\text{total}}(s)$는 다음과 같이 주어집니다: <br>

$u_{\text{total}}(s)$ = $(1-\lambda(s))u_{\text{MPC}}(s) + \lambda(s)u_{\text{RL}}(s)$ <br>

이 때 $\lambda(s)$는 $0 \le \lambda(s) \le 1$ 범위를 가지는 적응형 가중치로, $\lambda(s)=0$이면 MPC의 결정만 사용하고 $\lambda(s)=1$이면 RL 정책만 따르는 극단을 의미합니다. 대부분의 경우 $\lambda(s)$는 0과 1 사이의 값을 취하면서 MPC와 RL의 비중을 조절하게 됩니다. <br>

이 상태 기반 가중치 결합 아이디어는 안전성 강화 등을 위해 최근 제안된 일부 RL 기법과 맥락을 같이합니다. 예를 들어, Imperial College London 연구진의 RL-AR (Reinforcement Learning with Adaptive Regularization) 알고리즘에서는 **“포커스 모듈”** 을 통해 상태별로 안전 정책(기존 제어기)과 RL 정책을 조합하여, 미탐색 상태에서는 안전한 기존 정책에 더 의존하고 충분히 탐색된 상태에서는 RL 정책의 비중을 높이는 방식을 채택하였습니다. 이러한 접근은 상태에 따라 두 정책의 신뢰도를 동적으로 조절함으로써 안전성과 최적성 두 마리 토끼를 잡는 것을 목표로 합니다. 본 연구의 Adaptive Residual 구조도 이와 유사하게, 상태별로 MPC 대 RL의 신뢰도를 조정하여 필요한 경우에만 RL의 창의성을 끌어내고 불필요할 때는 MPC의 안정성에 의존하도록 합니다. 고정 비율 결합의 단점을 넘어서는 새로운 시도라는 점에서 본 접근법의 의의를 찾을 수 있습니다.

## 구현 및 실험 요약: PyRocketCraft 환경에서의 평가
시뮬레이션 환경 및 구현: 실험은 PyRocketCraft 로켓 착륙 시뮬레이터 환경에서 이루어졌습니다. 이 환경은 OpenAI Gym 인터페이스를 따르는 연속 제어 태스크로, 로켓을 지정된 지점에 연착륙시키는 것이 목표입니다. 기본적으로 PyRocketCraft에는 NMPC (비선형 MPC) 제어기가 구현되어 있어, 로켓의 현재 상태를 받아 최적 추력을 계산해주는 모듈이 있습니다. 이 구조를 확장하여, RL 에이전트가 출력하는 추가적인 보정 힘(residual force)을 MPC의 출력에 더해 최종 제어입력을 생성하도록 했습니다. 적응형 가중치 $\lambda(s)$는 별도의 경량 신경망으로 상태 → [0,1] 값을 출력하도록 설계하였고, 이 값을 통해 MPC 출력과 RL 출력의 가중치를 결정합니다. 요약하면, MPC (전문가 정책)가 기본 제어 신호를 제공하고 RL (학습 정책)이 그 신호를 수정 보완하며, $\lambda(s)$ 신경망이 현재 상태에서 얼마나 RL의 수정을 수용할지를 조절하는 구조입니다.<br>

+ MPC Only: 기존 MPC 제어만으로 로켓을 제어합니다.
+ MPC + RL Residual: 고정 결합의 Residual RL 방식으로, MPC 출력 + RL 잔차를 단순 합합니다. 즉, 모든 상태에 동일한 비율로 RL 잔차를 적용하는 residual 학습 방법입니다 (기존 연구들의 방식).
+ MPC + RL Adaptive Residual: 제안하는 방식으로, 상태마다 $\lambda(s)$를 계산하여 MPC와 RL 출력의 비율을 조절합니다. 상태에 따라서는 MPC 위주(작은 $\lambda$), 다른 상황에서는 RL 위주(큰 $\lambda$)로 제어가 이루어질 수 있습니다.

모든 방법에서 강화학습 알고리즘으로는 PPO 알고리즘을 사용하여, 에피소드 단위로 정책을 학습시켰습니다. RL 정책의 보상 함수는 착륙 오차를 최소화하고 안정적으로 착지하도록 설계되었으며, MPC Only의 경우에는 학습 과정 없이 기존 MPC 모듈을 실행했습니다. MPC+RL Residual과 Adaptive Residual의 초기 단계에서는 MPC 제어가 기본적으로 수행되므로, 에이전트의 초기 탐색이 안정적으로 이루어진다는 장점이 있습니다. 이는 초기부터 탐색하는 순수 RL에 비해 수렴을 빠르게 하고 안전성을 높여줍니다. <br>

성능 비교: 실험 결과, 세 가지 방식 모두 일정 수준의 성능을 달성하였으며 최종적인 임무 성공률이나 보상에서 큰 차이는 나타나지 않았습니다. 아래는 세 접근법의 착륙 결과입니다. <br>

**MPC Only**

https://github.com/user-attachments/assets/fcaca0df-2e1d-487a-965f-1437d1f4f625




<br> **MPC + RL Residual**



https://github.com/user-attachments/assets/725381c9-c8a8-447e-afe6-7d9dbc86fe69





<br> **MPC + RL Adaptive Residual**



https://github.com/user-attachments/assets/b9b0c7d9-d146-436b-82e7-25bf984093e6




<br>
기대와 달리 Adaptive Residual 방식이 두 다른 방법보다 눈에 띄게 우수한 성능 우위를 보이지는 못하였으며, 학습 속도나 최종 수렴값 모두 비슷한 수준을 나타냈습니다. 이는 해당 시뮬레이션 과제에서 MPC 단독으로도 이미 충분히 좋은 성능을 발휘했고, RL이 추가로 개선할 여지가 크지 않았기 때문으로 추정됩니다. 또한 RL 정책의 학습 안정성/효율의 한계로 인해 Residual이 적극적으로 활용되지 못했을 가능성도 있습니다. <br><br>

적응형 가중치 $\lambda(s)$ 분석: Adaptive Residual 구조의 핵심 지표인 $\lambda(s)$의 값을 모니터링한 결과, 전반적으로 $\lambda(s)$가 0에 가까운 값을 취하는 경향이 관찰되었습니다. 이는 대부분의 상태에서 MPC의 제어 신호를 거의 그대로 사용하고 RL 잔차는 매우 적게 더해졌음을 의미합니다. $\lambda(s)$ 분포를 보면 에피소드 동안 평균적으로 0.1 이하의 값이 많았고, 특정 상황을 제외하면 $\lambda(s)$가 크게 증가하는 경우는 드물었습니다. 다시 말해, Adaptive 구조임에도 불구하고 실제 제어는 MPC 위주로 이루어지는 빈도가 높았다는 것입니다. 이러한 결과 때문에 $\lambda(s)$가 상태에 따라 어떻게 변동하는지 뚜렷이 해석하기가 어려웠습니다. $\lambda(s)$가 거의 항상 낮게 유지되었다는 것은, RL 정책이 MPC 대비 얻는 이득이 미미하거나 RL이 자신감 있게 제어를 할만한 상황이 적었다고 볼 수 있습니다. 이는 본 환경에서 MPC의 성능이 우수하여 RL이 굳이 개입하지 않아도 되는 측면, 혹은 RL의 학습 부족으로 인해 신뢰할 만한 보정 값을 학습하지 못한 측면 둘 다 원인일 수 있습니다. 결과적으로 상태 기반 가중치의 역동적인 변화는 기대만큼 나타나지 않아, Adaptive Residual의 의도했던 이점(상황에 따른 RL 개입 조절)을 확인하기에는 어려움이 있었습니다.

## 한계점: 실험 및 방법상의 제약
이번 연구 및 실험을 통해 몇 가지 한계점이 드러났습니다:
+ RL 학습 성능 부족: RL 잔차 정책이 기대만큼의 성능 향상을 이루지 못했습니다. 이는 강화학습 알고리즘의 미성숙이나 파라미터 설정의 한계로 인해 RL 에이전트가 MPC를 능가하는 교정 신호를 학습하지 못했기 때문으로 보입니다. 결국 Adaptive 구조에서도 RL의 기여가 미미하여, MPC 단독과 큰 차이를 내지 못했습니다.
+ 하이퍼파라미터 튜닝 미비: RL 기반 방법의 성능은 학습률, 보상 구조, 탐험 계수 등 하이퍼파라미터에 크게 좌우되는데, 본 실험에서는 이러한 매개변수를 최적화할 시간이 부족했습니다. 파라미터 튜닝이 충분하지 않으면 RL의 학습 효율 저하와 불안정으로 이어져, Residual 정책의 잠재력이 제한되었습니다.
+ $\lambda(s)$ 해석의 불확실성: Adaptive Residual의 핵심인 $\lambda(s)$ 값에 대한 명확한 해석을 얻지 못했습니다. $\lambda(s)$가 주로 0에 가까운 현상에 대해, 왜 그런 분포가 나왔는지 (예: RL 정책의 불확실성으로 항상 MPC를 따르는 것인지, 환경 자체가 RL 개입을 덜 필요로 하는 것인지) 명확히 규명하지 못했습니다. $\lambda(s)$를 관찰하더라도 상태 특성별로 유의미한 패턴을 찾기 어려웠고, 이는 Adaptive 전략의 작동 원리를 이해하는 데 한계를 주었습니다.
+ 시뮬레이터 state 다양성 부족: 사용된 PyRocketCraft 환경이 상대적으로 단순하고 state 다양성이 제한적인 점도 한계로 작용했습니다. 로켓 착륙이라는 과제 자체가 유사한 시나리오의 반복(예: 초기 높이와 속도 조건이 큰 차이 없음)으로 이루어지다 보니, RL이 학습을 통해 새로운 교정 전략을 발휘해야 할 상황이 적었습니다. 또한 바람이나 센서 노이즈 등의 환경 변화 요인도 제한적이어서, Adaptive Residual 구조가 빛을 발할 만한 극단 상황이 충분히 주어지지 않았습니다.

## 결론 및 향후 연구: Adaptive Residual의 가능성과 개선 방향
본 프로젝트를 통해 MPC와 RL의 결합을 모색한 Adaptive Residual 제어 전략의 가능성과 한계를 모두 확인할 수 있었습니다. 장점으로는, MPC를 초기 정책으로 활용함으로써 강화학습의 초기 탐색을 안정화하고 학습 효율을 높일 수 있다는 점을 실험적으로 확인했습니다. 또한 상태 기반으로 RL의 개입을 조절하려는 시도 자체는 기존 residual 학습 보다 유연한 통합 제어로 나아가는 흥미로운 방향임을 알 수 있었습니다. <br>

그러나 이번 실험에서는 Adaptive Residual 구조의 기대만큼의 성능 개선을 도출하지 못한 한계도 명백했습니다. $\lambda(s)$ 값이 대부분 낮게 형성되어 RL의 영향력이 제한적이었고, 결과적으로 세 가지 비교 방법 간 성능 차이가 크지 않게 나타났습니다. 이는 한편으로는 MPC의 강력한 기본 성능을 재확인시켜주지만, 다른 한편으로는 RL을 결합하는 부가 복잡성이 정당화될 만큼의 이득을 보이지 못한 상황이 되었습니다. <br>

향후 연구에서는 이러한 한계를 극복하고 Adaptive Residual 접근의 효과를 극대화하기 위한 몇 가지 방향을 제안합니다: <br>
+ 보다 복잡하고 다양한 환경에서 검증: 현재의 로켓 착륙보다 상태 공간과 동적 요인이 더 풍부한 시뮬레이션이나 실제 로봇 환경에 Adaptive Residual 구조를 적용해 볼 필요가 있습니다. 예를 들어 **환경 변화(wind 등 외란)**나 목표 조건의 다양성을 늘려, MPC 단독으로는 대응이 어려운 상황을 만들면 RL 잔차 정책의 유용성이 드러날 수 있습니다. 복잡한 환경일수록 Adaptive 구조의 장점(상태별 RL 개입 조절)이 발휘될 것으로 기대합니다.
+ 강화학습 알고리즘 및 보상 설계 개선: RL 잔차 정책의 성능을 높이기 위해 학습 알고리즘과 보상 함수를 개선해야 합니다. 우선, 하이퍼파라미터 튜닝을 체계적으로 수행하고, 필요하다면 더 나은 탐험 전략이나 안정화 기법(예: 목표 네트워크, 배치 정규화 등)을 도입해 RL의 학습 성능을 향상시켜야 합니다. 보상 함수도 MPC가 이미 잘 수행하는 기본 목표 외에, MPC가 놓치는 세부 최적화 목표를 추가로 포함하는 reward shaping을 고려할 수 있습니다. 예를 들어 연착륙 충격 최소화나 연료 효율 향상 등의 부가 목표를 보상에 포함하면, RL 잔차 정책이 MPC 대비 이점을 얻을 수 있는 방향으로 학습하도록 유도할 수 있습니다.



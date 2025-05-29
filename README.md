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

## Novelty: Adoption of State-Based Adaptive Residual Integration
The key novelty of the proposed Adaptive Residual architecture lies in the incorporation of a state-dependent blending ratio between MPC and RL. While conventional residual RL approaches merely apply a fixed correction from RL to the baseline controller's output, this study introduces a design that allows the degree of correction to vary dynamically with the state. To achieve this, I implemented a state-dependent weighting function, $\lambda(s)$, which enables a weighted combination of the MPC output and the RL residual output at each time step, based on the current state $s$. The final control input, $u_{\text{total}}(s)$, is defined as follows: <br>

$u_{\text{total}}(s)$ = $(1-\lambda(s))u_{\text{MPC}}(s) + \lambda(s)u_{\text{RL}}(s)$ <br>

Here, $\lambda(s)$ is an adaptive weight constrained within the range $0 \le \lambda(s) \le 1$. A value of $\lambda(s) = 0$ indicates exclusive reliance on the MPC decision, while $\lambda(s) = 1$ represents a full reliance on the RL policy. In most cases, $\lambda(s)$ takes a value between 0 and 1, dynamically adjusting the balance between MPC and RL contributions. <br>

This idea of state-based weighted blending aligns with the context of several recently proposed RL methods aimed at enhancing safety. For instance, the RL-AR (Reinforcement Learning with Adaptive Regularization) algorithm developed by researchers at Imperial College London employs a “focus module” to combine a safety policy (i.e., a traditional controller) and an RL policy based on the current state. In this approach, the agent leans more heavily on the safety policy in unexplored states and increases reliance on the RL policy in well-explored regions. Such a method aims to dynamically balance trust between the two policies depending on the state, thereby pursuing both safety and optimality.

Similarly, the Adaptive Residual architecture proposed in this study adjusts the relative trust between MPC and RL according to the state. This design enables the system to draw on the creativity of RL only when necessary, while relying on the stability of MPC in other situations. By moving beyond the limitations of fixed-ratio combinations, this approach represents a novel attempt to integrate the strengths of both paradigms.

## Implementation and Experiment Summary: Evaluation in the PyRocketCraft Environment
Simulation Environment and Implementation: The experiments were conducted in the PyRocketCraft rocket landing simulator. This environment, following the OpenAI Gym interface, represents a continuous control task where the objective is to achieve a soft landing of a rocket at a designated target location. PyRocketCraft includes a built-in NMPC (Nonlinear Model Predictive Control) controller that computes optimal thrust based on the rocket’s current state.

I extended this setup to allow an RL agent to generate an additional corrective force (residual force), which is added to the MPC output to produce the final control input. The adaptive weight $\lambda(s)$ was implemented as a lightweight neural network designed to map the current state to a value in the [0,1] range, determining the blending ratio between MPC and RL outputs. In summary, the MPC (expert policy) provides the base control signal, the RL (learned policy) supplies corrective adjustments, and the $\lambda(s)$ network regulates how much of the RL correction is accepted at each state.<br>

+ MPC Only: The rocket is controlled solely using the conventional MPC controller.
+ MPC + RL Residual: A fixed-combination Residual RL approach where the final control input is a simple sum of the MPC output and the RL residual. In this method, the RL residual is applied at a constant ratio across all states—representing the residual learning technique commonly used in prior studies.
+ MPC + RL Adaptive Residual: In the proposed approach, the blending ratio between MPC and RL outputs is adjusted by computing a state-dependent weight $\lambda(s)$. Depending on the state, control may rely more heavily on MPC (when $\lambda$ is small) or shift toward RL (when $\lambda$ is large).

In all approaches, the PPO algorithm was used as the reinforcement learning method, and the policy was trained on an episode basis. The reward function for the RL policy was designed to minimize landing error and ensure a stable touchdown. In the MPC Only setting, no learning process was involved; instead, the pre-existing MPC module was directly executed.

For both MPC+RL Residual and Adaptive Residual methods, MPC control was primarily used in the early training stages, allowing the agent to explore in a more stable manner. This setup provides a significant advantage over pure RL, which starts exploration from scratch, by accelerating convergence and enhancing safety from the beginning. <br>

Performance Comparison: The experimental results showed that all three approaches achieved a comparable level of performance, with no significant differences observed in final task success rates or reward outcomes. The following presents the landing results of the three methods. <br>

**MPC Only**

https://github.com/user-attachments/assets/fcaca0df-2e1d-487a-965f-1437d1f4f625




<br> **MPC + RL Residual**



https://github.com/user-attachments/assets/725381c9-c8a8-447e-afe6-7d9dbc86fe69





<br> **MPC + RL Adaptive Residual**



https://github.com/user-attachments/assets/b9b0c7d9-d146-436b-82e7-25bf984093e6




<br>
Contrary to expectations, the Adaptive Residual method did not exhibit a clear performance advantage over the other two approaches. Both the learning speed and the final convergence values were found to be at similar levels. This is likely because the standalone MPC already demonstrated sufficiently strong performance for this particular simulation task, leaving limited room for RL to provide further improvements. Additionally, the limited learning stability and efficiency of the RL policy may have hindered the effective utilization of the residual component. <br><br>

Analysis of Adaptive Weight $\lambda(s)$: Monitoring the values of $\lambda(s)$—the core indicator of the Adaptive Residual structure—revealed a general tendency for $\lambda(s)$ to remain close to zero. This indicates that, in most states, the control signal from MPC was used almost as-is, with minimal contribution from the RL residual. The distribution of $\lambda(s)$ values showed that the average often stayed below 0.1 throughout episodes, and significant increases in $\lambda(s)$ were rare, except in a few specific situations.

In other words, despite being an adaptive structure, control was predominantly driven by MPC. As a result, it was difficult to clearly interpret how $\lambda(s)$ varied with different states. The consistently low values suggest either that the RL policy offered little additional benefit compared to MPC, or that there were few situations where RL was confident enough to intervene effectively. This could be due to two potential factors: the strong performance of MPC making RL involvement largely unnecessary, or insufficient learning on the RL side, preventing it from producing reliable corrective signals.

Consequently, the expected dynamic modulation of RL involvement through state-based weighting did not manifest as anticipated, making it difficult to confirm the intended advantages of the Adaptive Residual approach—namely, selectively adjusting RL intervention based on situational needs.

## Limitations: Constraints in Experimentation and Methodology
Several limitations emerged through the course of this study and its experiments:
+ Insufficient RL Learning Performance: The RL residual policy did not achieve the level of performance improvement that was initially expected. This is likely due to limitations in the reinforcement learning algorithm itself or suboptimal parameter settings, which prevented the RL agent from learning corrective signals that could outperform those generated by MPC. As a result, even within the Adaptive structure, the contribution of RL remained marginal, leading to outcomes that were not significantly different from using MPC alone.
+ Insufficient Hyperparameter Tuning: The performance of RL-based methods is highly sensitive to hyperparameters such as learning rate, reward structure, and exploration coefficients. However, in this experiment, there was limited time to optimize these parameters. Inadequate tuning can lead to reduced learning efficiency and instability in RL, which in turn constrained the potential of the residual policy.
+ Uncertainty in Interpreting $\lambda(s)$: A clear interpretation of the $\lambda(s)$ values—the core component of the Adaptive Residual method—could not be obtained. The observed tendency of $\lambda(s)$ to remain close to zero raised questions about the underlying cause: whether this was due to the RL policy's uncertainty leading to default reliance on MPC, or because the environment itself required little intervention from RL. Even with direct observation of $\lambda(s)$, it was difficult to identify meaningful patterns across different state characteristics, which limited the understanding of how the Adaptive strategy was functioning in practice.
+ Limited State Diversity in the Simulator: Another limitation stemmed from the relatively simple and constrained nature of the PyRocketCraft environment. The rocket landing task involved repeated scenarios with minimal variation—such as consistent initial altitude and velocity conditions—which reduced opportunities for the RL agent to develop and apply novel corrective strategies through learning. Moreover, environmental variations such as wind disturbances or sensor noise were minimal, meaning that the Adaptive Residual architecture was rarely exposed to extreme conditions where its benefits could truly stand out.

## 결론 및 향후 연구: Adaptive Residual의 가능성과 개선 방향
본 프로젝트를 통해 MPC와 RL의 결합을 모색한 Adaptive Residual 제어 전략의 가능성과 한계를 모두 확인할 수 있었습니다. 장점으로는, MPC를 초기 정책으로 활용함으로써 강화학습의 초기 탐색을 안정화하고 학습 효율을 높일 수 있다는 점을 실험적으로 확인했습니다. 또한 상태 기반으로 RL의 개입을 조절하려는 시도 자체는 기존 residual 학습 보다 유연한 통합 제어로 나아가는 흥미로운 방향임을 알 수 있었습니다. <br>

그러나 이번 실험에서는 Adaptive Residual 구조의 기대만큼의 성능 개선을 도출하지 못한 한계도 명백했습니다. $\lambda(s)$ 값이 대부분 낮게 형성되어 RL의 영향력이 제한적이었고, 결과적으로 세 가지 비교 방법 간 성능 차이가 크지 않게 나타났습니다. 이는 한편으로는 MPC의 강력한 기본 성능을 재확인시켜주지만, 다른 한편으로는 RL을 결합하는 부가 복잡성이 정당화될 만큼의 이득을 보이지 못한 상황이 되었습니다. <br>

향후 연구에서는 이러한 한계를 극복하고 Adaptive Residual 접근의 효과를 극대화하기 위한 몇 가지 방향을 제안합니다: <br>
+ 보다 복잡하고 다양한 환경에서 검증: 현재의 로켓 착륙보다 상태 공간과 동적 요인이 더 풍부한 시뮬레이션이나 실제 로봇 환경에 Adaptive Residual 구조를 적용해 볼 필요가 있습니다. 예를 들어 **환경 변화(wind 등 외란)**나 목표 조건의 다양성을 늘려, MPC 단독으로는 대응이 어려운 상황을 만들면 RL 잔차 정책의 유용성이 드러날 수 있습니다. 복잡한 환경일수록 Adaptive 구조의 장점(상태별 RL 개입 조절)이 발휘될 것으로 기대합니다.
+ 강화학습 알고리즘 및 보상 설계 개선: RL 잔차 정책의 성능을 높이기 위해 학습 알고리즘과 보상 함수를 개선해야 합니다. 우선, 하이퍼파라미터 튜닝을 체계적으로 수행하고, 필요하다면 더 나은 탐험 전략이나 안정화 기법(예: 목표 네트워크, 배치 정규화 등)을 도입해 RL의 학습 성능을 향상시켜야 합니다. 보상 함수도 MPC가 이미 잘 수행하는 기본 목표 외에, MPC가 놓치는 세부 최적화 목표를 추가로 포함하는 reward shaping을 고려할 수 있습니다. 예를 들어 연착륙 충격 최소화나 연료 효율 향상 등의 부가 목표를 보상에 포함하면, RL 잔차 정책이 MPC 대비 이점을 얻을 수 있는 방향으로 학습하도록 유도할 수 있습니다.



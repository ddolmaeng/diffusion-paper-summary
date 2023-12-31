Diffusion models 논문 요약

1. **DDPM (Denoising Diffusion Probabilistic Models)**   [paper](https://arxiv.org/abs/2006.11239)  
   - assumption : gaussian noise, markovian process, $\Sigma_{\theta} = \sigma_t^2 \mathbf{I}$  
   - objective function : variational bound 중 $L_{1:T-1}$ term -> $L_simple$ 을 optimizing 하는 것과 유사
   - $L_{\text{simple}}(\theta) \coloneqq \mathbb{E}_{t, x_0, \epsilon} \left[ \left\| \epsilon - \epsilon_{\theta} \left( \sqrt{\bar{\alpha}_t} x_0 + \sqrt{1 - \bar{\alpha}_t} \epsilon, t \right) \right\|^2 \right]$

   - goal : forward process가 given 일 때 reverse process $\mu^{\tilde}_{t}$, $\Sigma^{\tilde}_{t}$ fitting를 $\mu_{\theta}$, $\Sigma_{\theta}$ fitting; 이 때 $\Sigma_{\theta}$ 는 fixed(not training)
   - 해결한 문제 : training 할 때는 $x_0$의 정보를 사용하나 sampling 할 때는 $x_0$의 정보를 사용하지 않는다. $x_t$를 통해 $x_0$를 추정해야 한다. gaussain noise가 iid 이기 때문에 $x_t$와 $\epsilon$ 을 안다면 $x_0$를 추정할 수 있다.
   - architecture : u-net


2. **Improved Denoising Diffusion Probabilistic Models**   [paper](https://arxiv.org/abs/2102.09672)
   - goal : improving DDPM
   - improving log likelihood
      - learning $\Sigma_{\theta}$
         - $\Sigma_{\theta}$ term 을 training 하지 않았었는데 이를 구하기 위해서 $L_{hybrid}$ 이용 (기존 $L_{simple}$ 에는 $\Sigma_{\theta} 와 관련된 항이 존재 X (L_{simple} 에서 \sigma_t term 을 제외하였음); $L_{hybrid} = L_{simple} + \lambda L_{vlb} \qquad (\lambda = 0.001)$
         - $\Sigma_{\theta}(x_t, t) = \exp(v \log \beta_t + (1 - v) \log \tilde{\beta}_t)$
           $v$를 fitting, 즉 $\Sigma_\theta$도 자유도를 가진다.
   
      - improving the noise schedule
         - forward process 기준 마지막으로 가면 갈수록 너무 nosie의 비율이 높아 noise를 천천히 추가함
         - linear schedule -> cosine schedule
           
      - reducing gradient noise
         - $L_{vlb} = L_0 + \cdots + L_T$ 의 noise가 크다고 판단해 importance sampling 진행

   - improving sampling speed
      - $L_{simple}$ 보다 적은 step에서 FID score가 안정화 되는 것을 관측할 수 있다.
      - 하지만 DDIM도 사실 비슷한 성능

3. **Denoising Diffusion Implicit Models**   [paper](https://arxiv.org/abs/2010.02502)
   - goal : assumption(non-markovian) 변경을 통한 DDPM 가속화(적은 step으로도 학습 가능하게)
   - 가속화 하는 법 : 병렬 처리가 가능한 구조 ($x_{t-1}$이 $x_{t}$에 cascade하지 않게, 바꿔서 말하면 deterministic 하게)
   - Assumption
   - $q_{\sigma}(X_{1:T}|X_0) = q_{\sigma}(X_1|X_0) \Pi_{t=2}^{T} q_{\sigma} (X_{t}|X_{t-1}, X_0) = q_{\sigma} (X_T|X_0) \Pi_{t=2}^{T} q_{\sigma} (X_{t-1}|X_{t}, X_0)$
   - $q_{\sigma} (X_{t-1}|X_{t}, X_0)$ is gaussian (이 때 mu 는 DDPM과 똑같은 세팅, 그리고 여기서 variance를 0으로 만들어준다면 deterministic한 결과를 얻을 수 있음, DDPM과 똑같은 variance를 넣는 것도 가)

   - Question : 그러면 왜 deterministic한 result를 얻기 위해서는 왜 non-markovian process를 사용하여야하는가?
   - Answer : markovian process를 가정하면 $q(x_{t-1}|x_t, x_0)$ 의 분산을 0으로 컨트롤 할 수 없다. 이 때 분산을 0으로 만들어주기 위해서는 $\alpha_t$ 를 1로 만들어야하는데 이 의미는 forward process에서 어떠한 noise도 더하지 않겠다는 의미

   - sampling acceleration : sampling 과정에서 sub-sequecne를 이용하게 하면 적은 step으로도 sampling 가능


4. **Generative Modeling by Estimating Gradients of the Data Distribution**   [paper](https://arxiv.org/abs/1907.05600)
   - goal : reverse SDE에서 유도된 $\nabla_x \log p(x)$ 를 효과적으로 score matching 시키는 방법
   - advantage : no adversarial training, no surrogate losses, no sampling from the score network during training
   - Score matching for score estimation : 효과적인 방식으로 $s_{\theta}(x) \sim \nabla_x \log p(x)$ 예측
      - trace based : $tr(\nabla_x s_{theta}(x))$ 이용; 하지만 high demensional data 일 때 $tr(\nabla_x s_{theta}(x))$를 구하는 것은 힘들다
      - sliced score matching : $tr(\nabla_x s_{theta}(x))$ 대신 $v^t \nabla_x s_{\theta}(x) v$ 이용 (v : multivariate standard normal vector); 직접 trace에 접근하지 않아도 되어 efficient

   - Sampling with Langevin dynamics : $\tilde{x}_t = \tilde{x}_{t-1} + \frac{\epsilon}{2} \nabla_x \log p(\tilde{x}_{t-1}) + \sqrt{\epsilon} z_t$; 이 때 $s_{\theta}(x)$를 $\nabla_x \log p(x)$ 대신 사용

   - Problem (Vanilla Sampling with Langevin dynamics)
      - under the manifold hypothesis, $p_{data}$가 존재하지 않는 x 에서 $s_\theta(x)$가 정의되지 않음
      - low data density 영역에서의 부정확
      - mixture data distribution에 대한 분별능력 X
         - e.g. $p_{data} = 0.2 \mathcal{N} ((0,0), I)) + 0.8  \mathcal{N} ((1,1), I))$ 라고 하면, 이상적인 경우 20%는 $\mathcal{N} ((0,0), I))$, 그리고 80%는 $\mathcal{N} ((1,1), I))$로 분류하기를 원함; 하지만, 임의의 점에서 시작한다면 거의 50:50으로 분류 (why? $\mathcal{N} ((0,0), I))$ 근방에서는 $(0,0)$ 방향으로 gradient가 끌어당기는 힘이 더 강하고, $\mathcal{N} ((1,1), I))$ 근방에서는 $(1,1)$ 방향으로 gradient가 끌어당기는 힘이 더 강하기 때문에) <br />
           ![image](https://github.com/ddolmaeng/diffusion-paper-summary/assets/112860653/c5c1866d-8cf8-4332-b75d-67202758a27c)


   - Solution (perturbed data distribution)
      - $\{\sigma_i\}^{L}_{i=1}$, $\frac{\sigma_1}{\sigma_2} = \cdots = \frac{\sigma_{L-1}}{\sigma_L} > 1$ 을 만족하게 sequence 잡은 후
      - $q_{\sigma}(x) = \int p_{\text{data}}(t) \mathcal{N} (x | t, \sigma^2 I) \mathrm{d}t$ 로 각 step 마다 점점 perturbed noise가 작아지게 data distribution을 setting 한다.
      - 대신 추정해야하는 $s_{\theta} (x)$ 도 더이상 $x$에만 영향을 받지 않고 $\sigma$에 영향을 받게 setting. $s_{\theta} (x, \sigma)$

   - denoising score matching : $s_\theta(x,t) \sim \nabla \log q_{\sigma_t}(x)$
      - $\arg \min_{\theta} \Sigma_{t=1}^{T} \lambda(\sigma_t) \mathbb{E}_{q_{\sigma_t}} [\|s_{\theta}(x, t) - \nabla \log q_{\sigma_t} (x_t)\|_2^2]$
      - $\lambda(\sigma_t)$ : coefficient, 논문에서는 $\lambda(\sigma) = \sigma^2$ 이용 (why? variance가 $\sigma^2 I$인 정규분포로 perturbed 했기 때문에 $\|s_{\theta} (x, \sigma)\|_2 \propto 1/{\sigma}$)

   - algorithm <br />
     ![image](https://github.com/ddolmaeng/diffusion-paper-summary/assets/112860653/99cfb858-de57-464c-b165-861616a6170f)



5. **Score-Based Generative Modeling through Stochastic Differential Equations**   [paper](https://arxiv.org/abs/2011.13456)
   - goal : SDE를 이용한 diffusion model 개선
   - assumption : Stochastic process, Ito process
   - SMLD (VE SDE)
      - discrete form : $x_{i} = x_{i-1} + \sqrt{\sigma^{2}_{i} - \sigma^{2}_{i - 1}} z_{i-1}, \qquad i = 1, ..., N$ , $z_{i-1} \sim \mathcal{N}(0, I)$
      - continuous form : $dx = \sqrt{\frac{d[\sigma^2 (t)]}{\mathrm{d}t} \Delta t} z(t)$
      - VE(variance explosion) SDE
      - $p_{0t}(x(t)|x(0)) = \mathcal{N}(x(t);x(0), [\sigma^2(t) - \sigma^2(0)] I)$
    
   - DDPM (VP SDE)
      - discrete form : $x_i = \sqrt{1 - \beta_i} x_{i-1} + \sqrt{\beta_i} z_{i-1}$
      - continuous form : $\mathrm{d}x = - \frac{1}{2} \beta (t) x \mathrm{d}t + \sqrt{\beta (t)} \mathrm{d}w$
      - $\frac {\mathrm{d}\Sigma (t)}{\mathrm{d}t} = \beta (t) (I - \Sigma (t))$
      - $\Sigma(t) = I + e^{\int_{0}^{t} - \beta (s) \mathrm{d}s} (\Sigma(0) - I)$, if $\Sigma (0) = I$, then $\Sigma_{VP}(t)$ always bounded
      - $p_{0t}(x(t)|x(0)) = \mathcal{N}(x(t);x(0) e^{-\frac{1}{2}\int^t_0 \beta(s) \mathrm{d}s}, [1-e^{-\int^t_0 \beta(s) \mathrm{d}s}]I)$

   - sub-VP SDE
      - continuous form : $\mathrm{d}x = - \frac{1}{2} \beta (t) x \mathrm{d}t + \sqrt{\beta (t) (1-e^{-2 \int_{0}^{t} \beta(s) \mathrm{d}s})} \mathrm{d}w$
      - $\Sigma(t) = I + e^{- 2 \int_{0}^{t} \beta (s) \mathrm{d}s} I + e^{- \int_{0}^{t} \beta (s) \mathrm{d}s} (\Sigma(0) - 2I)$
      - $\Sigma_{sub-VP}(t) \preccurlyeq \Sigma_{VP}(t)$
      - $p_{0t}(x(t)|x(0)) = \mathcal{N}(x(t);x(0) e^{-\frac{1}{2}\int^t_0 \beta(s) \mathrm{d}s}, [1-e^{-\int^t_0 \beta(s) \mathrm{d}s}]^2 I)$

   - reverse SDE
      - forward SDE : $dx = f(x,t) dt + G(t) dw$
      - reverse time SDE : $dx = [f(x,t) - G(t)G(t)^T \nabla_x \log p_t(x)]dt + G(t)d\bar{w}$
      - discrete form : $x_i = x_{i+1} - f_{i+1}(x_{i+1}) + G_{i+1}G_{i+1}^T s_{\theta^*}(x_{i+1}, i+1) + G_{i+1}z_{i+1}$

   - sampling algorithm
      ![image](https://github.com/ddolmaeng/diffusion-paper-summary/assets/112860653/faf49f54-0c1a-41c3-92d3-7551cafb14fd)
      
      ![image](https://github.com/ddolmaeng/diffusion-paper-summary/assets/112860653/9f85a132-2910-4fc8-a27f-64e05da961be)
      
      - corrector : langevin dynamics (langevin MCMC)
         - langevin dynamics
            - $x_{i+1} <- x_i + \epsilon (\nabla_x \log p(x))|_{x=x_i} + \sqrt{2\epsilon}z_i, qquad z_i~\mathcal{N}(0,I)$
            - when $\epsilon \rightarrow 0$ and $T \rightarrow \infty$, then $p(x)$ converge


6. **Diffusion Models Beat GANs on Image Synthesis**   [paper](https://arxiv.org/abs/2105.05233)
   - goal : architecture tuning과 classifier guidance를 통한 improving fidelity
   - assumption : conditional markovian noising process ($p(x_{t+1}|x_t,y) = p(x_{t+1}|x_t)$, $p(x_{1:T}|x_0, y) = \Pi_{t=1}^{T} p(x_t|x_{t-1}, y)$
   - lemma : $p(y|x_t,x_{t+1}) = p(y|x_t)$
      - pf : $p(y|x_t, x_{t+1}) = \frac{p(x_t, x_{t+1}, y)}{p(x_t, x_{t+1})} = \frac{p(x_{t+1}|x_t, y)p(x_t, y)}{p(x_{t+1}|x_t) p(x_t)} = p(y|x_t)$

   - classifier guidance
      - $p(x_t|x_{t+1}, y) = \frac{p(x_t,x_{t+1},y}{p(x_{t+1}, y)} = \frac{p(y|x_t,x_{t+1})p(x_t|x_{t+1})p(x_{t+1})}{p(x_{t+1},y)} = \frac{p(y|x_t)p(x_t|x_{t+1})p(x_{t+1})}{p(x_{t+1},y)}$
      - $\nabla_{x_t} p(x_t|x_{t+1}, y) = \nabla_{x_t} p(y|x_t) + \nabla_{x_t} p(x_t|x_{t+1})$
      - classifier : $\nabla_{x_t} p(y|x_t)$

   - algorithm
      ![image](https://github.com/ddolmaeng/diffusion-paper-summary/assets/112860653/7aa3c31a-38a7-4cdc-b169-999125c45bda)

   - add hyperparameter $\gamma$ : $\nabla_{x_t} p(x_t|x_{t+1}, y) = \nabla_{x_t} p(y|x_t) + \gamma \nabla_{x_t} p(x_t|x_{t+1})$


7. **Classifier-Free Diffusion Guidance**   [paper](https://arxiv.org/abs/2207.12598)
   - goal : classifier 없이 classifer guidance 만큼의 fidelity 보장
   - why : classifier가 이미지 품질을 높이는 것이 아닌 단순히 adversarial training을 통해 score만 올리는 건지에 대한 의문
   - $\nabla_{x_t} p(y|x_t) = \nabla_{x_t} p(x_t|x_{t+1}, y) - \nabla_{x_t} p(x_t|x_{t+1})$ 이용
   - $\nabla_{x_t} p(x_t|x_{t+1}, y) = \nabla_{x_t} p(y|x_t) + \gamma (\nabla_{x_t} p(x_t|x_{t+1}, y) - \nabla_{x_t} p(x_t|x_{t+1})) = (1+\gamma)\nabla_{x_t} p(y|x_t) - \gamma\nabla_{x_t} p(x_t|x_{t+1})$

   - algorithm
     ![image](https://github.com/ddolmaeng/diffusion-paper-summary/assets/112860653/2515856f-acaf-4614-95a6-bb670a045223)

     ![image](https://github.com/ddolmaeng/diffusion-paper-summary/assets/112860653/3fbe4e5f-ed68-47dd-80f3-0eb808b9dc3a)

   - disadvantage : sampling speed (classifier의 크기가 classifier free 보다 작기 때문에)



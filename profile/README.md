# Hi, FacialScore 👋

## Phân tích Topo-Hình học: Fiber Bundle cho Digital Human Emergent Ability

## 1. Các Không gian Đầu ra

$$\mathcal{T} \cong (\mathbb{Z}/V)^n \quad \text{(Text — discrete)}$$

$$\mathcal{A} \subset L^2(\mathbb{R}) \quad \text{(Audio — Hilbert space)}$$

$$\mathcal{M} = SO(3)^J \quad \text{(Motion — compact Lie group)}$$

---

## 2. Tại sao Projection thông thường PHÁ VỠ cấu trúc

### 2.1 No-Go Theorem cho $SO(3)$

$SO(3)$ là đa tạp compact với fundamental group:

$$\pi_1(SO(3)) = \mathbb{Z}/2\mathbb{Z}$$

**Hệ quả:** Không tồn tại ánh xạ liên tục toàn cục $\varphi: \mathbb{R}^d \to SO(3)$ vừa surjective vừa bảo toàn geodesic. Mọi parameterization đều có singularity:

$$\text{Euler angles: } \varphi: \mathbb{R}^3 \to SO(3) \quad \Rightarrow \text{Gimbal Lock tại } \theta = \frac{\pi}{2}$$

$$\text{Quaternion: } \varphi: S^3 \to SO(3) \quad \Rightarrow \text{double cover } (2:1), \text{ không injective}$$

$$\text{6D repr: } \varphi: \mathbb{R}^6 \to SO(3) \quad \Rightarrow \text{injective nhưng image là submanifold cong}$$

### 2.2 Vấn đề với Audio Space

Mọi projection hữu hạn chiều:

$$\varphi_A: \mathbb{R}^d \to \mathbb{R}^{T \times F}$$

đều mất thông tin phase và coherence của waveform, vì $L^2(\mathbb{R})$ vô hạn chiều.

---

## 3. Sai lầm của Framework "Giao điểm" — Nhìn thẳng vào vấn đề

Nếu yêu cầu $z \in \mathcal{M}_T \cap \mathcal{M}_A \cap \mathcal{M}_M$, thì $z$ phải thỏa mãn đồng thời:

$$f_T(z) = 0, \quad f_A(z) = 0, \quad f_M(z) = 0$$

Sau một bước update:

$$z_{t+1} = z_t - \eta \cdot \nabla L$$

$z_{t+1}$ gần như chắc chắn **không còn nằm trên giao điểm** — model không biết đang update cho task nào. Đây là lý do cần cấu trúc khác.

---

## 4. Cấu trúc Đúng: Fiber Bundle

### 4.1 Định nghĩa

Fiber Bundle $E = (E, B, \pi, F)$ với:

- $B$: Base manifold — không gian semantic chung, $\dim B = d_{\text{base}}$
- $F = F_T \times F_A \times F_M$: Fiber cho mỗi modality
- $\pi: E \to B$: Projection về base

Tại mỗi $b \in B$:

$$\pi^{-1}(b) \cong F_T \times F_A \times F_M$$

### 4.2 Tangent Space Decomposition

Tại mỗi điểm $e \in E$, tangent space tách thành hai thành phần trực giao:

$$T_e(E) = V_e(E) \oplus H_e(E)$$

trong đó:

- $V_e(E) = \ker(d\pi_e)$: **Vertical space** — tiếp tuyến dọc theo fiber
- $H_e(E)$: **Horizontal space** — lift của $T_b(B)$ lên $E$

---

## 5. Connection $\omega$ — Cơ chế tự động phân tách Gradient

Connection là một $\mathfrak{g}$-valued 1-form:

$$\omega: T_e(E) \to \mathfrak{g}_{\text{fiber}}$$

thỏa mãn:

$$\omega(v) = 0 \iff v \in H_e(E)$$

### 5.1 Phân tách Gradient tự động

Với gradient $g_i = \frac{\partial L_i}{\partial e}$ từ task $i$:

$$g_i = \underbrace{\omega(g_i)}_{\nabla_V L_i \in V_e(E)} + \underbrace{(g_i - \omega(g_i))}_{\nabla_H L_i \in H_e(E)}$$

- $\nabla_V L_i$: chỉ update fiber của modality $i$ — _"decoder này sai"_
- $\nabla_H L_i$: update base semantic — _"ý nghĩa chung cần thay đổi"_

Model **không cần biết** khi nào update gì — $\omega$ tự động phân tách.

---

## 6. Backpropagation — Từng bước

**Bước 1:** Forward pass

$$x_{\text{text}} \xrightarrow{\text{Encoder}} b \in B \xrightarrow{\sigma_i} f_i \in F_i \xrightarrow{\text{Decoder}_i} y_i$$

**Bước 2:** Loss gradients tại output

$$g_i = \frac{\partial L_i}{\partial y_i}, \quad i \in \{T, A, M\}$$

**Bước 3:** Vertical backprop — chỉ update decoder modality $i$

$$\frac{\partial L_i}{\partial \sigma_i(b)} = g_i \cdot \frac{\partial y_i}{\partial \sigma_i}$$

**Bước 4:** Horizontal component qua Connection

$$\nabla_H L_i = \left(\mathrm{Id} - \omega\right) \cdot \frac{\partial L_i}{\partial b}$$

**Bước 5:** Tổng hợp tại Base và Riemannian update

$$\nabla_B L = \sum_{i \in \{T,A,M\}} \nabla_H L_i$$

$$b_{t+1} = \exp_b\!\left(-\eta \cdot \nabla_B L\right)$$

**Bước 6:** Backprop thông thường từ $b$ về Encoder weights (sống trong $\mathbb{R}^n$).

---

## 7. Riemannian Gradient trên $SO(3)$

Metric tensor trên $SO(3)$:

$$g_{SO(3)}(R)[U, V] = \mathrm{tr}(U^\top V), \quad U, V \in T_R SO(3)$$

Riemannian gradient:

$$\mathrm{grad}_{SO(3)} f(R) = g^{-1}(R) \cdot df(R)$$

Update hợp lệ trên $SO(3)$ dùng Exponential Map:

$$R_{t+1} = R_t \cdot \exp_{\mathfrak{so}(3)}\!\left(-\eta \cdot \Omega\right)$$

trong đó $\Omega \in \mathfrak{so}(3)$ là skew-symmetric matrix (Lie algebra element).

Geodesic loss:

$$\mathcal{L}_{\text{geo}}(Q_{\text{pred}}, Q_{\text{gt}}) = \sum_j \arccos\!\left(\frac{\mathrm{tr}(Q_{\text{pred},j}^\top Q_{\text{gt},j}) - 1}{2}\right)$$

---

## 8. Holonomy — Thách thức ẩn

Khi parallel transport một vector dọc vòng kín $\gamma$ trên đa tạp cong, vector bị xoay một góc bởi holonomy operator:

$$\mathrm{Hol}(\gamma, z) \in \mathrm{Aut}(F_z)$$

Với $SO(3)$: holonomy group $= SO(3)$ chính nó — gradient bị distort qua các bước training nếu không hiệu chỉnh.

**Holonomy-Aware update:**

$$z_{t+1} = \exp_{z_t}\!\left(-\eta \cdot \Gamma_t \cdot \nabla L\right)$$

trong đó $\Gamma_t = \mathcal{P}(\gamma_{0 \to t})$ là parallel transport operator dọc path $\gamma$.

---

## 9. Điều kiện để Emergent Ability xuất hiện

**Điều kiện 1 — Submersion:**

$$\mathrm{rank}(d\varphi_z) = \dim(\mathcal{M}_i) \quad \forall z$$

**Điều kiện 2 — Transversality:**

$$\mathrm{codim}\!\left(\bigcap_i \mathcal{M}_i\right) = \sum_i \mathrm{codim}(\mathcal{M}_i)$$

**Điều kiện 3 — Bounded Holonomy:**

$$\|\mathrm{Hol}(\mathcal{M}_i, z)\| < C < \infty$$

**Điều kiện 4 — Horizontal Gradient Alignment** (điều kiện Emergence):

$$\mathbb{E}\!\left[\cos\angle\!\left(\nabla_H L_i,\, \nabla_H L_j\right)\right] > 0 \quad \text{sau } T^* \text{ bước training}$$

Tức là các tasks dần học cùng hướng trên Base — đây là _fingerprint_ của Emergent Ability.

---

## 10. Connection $\omega$ được học như thế nào

$\omega$ được parameterize và học cùng model:

$$\omega_\theta: T_e(E) \to \mathfrak{g}_{\text{fiber}}$$

Enforce constraint $\omega_\theta(v) = 0$ với $v \in H_e(E)$ qua regularization:

$$\mathcal{L}_{\text{conn}} = \left\|\omega_\theta\!\left(\nabla_H L\right)\right\|^2 \to 0$$

Tổng loss:

$$\mathcal{L}_{\text{total}} = \mathcal{L}_{\text{task}} + \lambda \cdot \mathcal{L}_{\text{conn}}$$

# Parameterize $\omega_\theta$ bằng Neural Network

## 1. Nhắc lại: $\omega_\theta$ cần thỏa mãn điều gì?

$\omega_\theta$ là một $\mathfrak{g}$-valued 1-form trên $E$, cần thỏa mãn **hai constraint cứng**:

$$\omega_\theta(v) = 0 \iff v \in H_e(E) \quad \text{(Horizontal vectors → không có fiber component)}$$

$$\omega_\theta(v) \in \mathfrak{g}_{\text{fiber}} \iff v \in V_e(E) \quad \text{(Vertical vectors → Lie algebra element)}$$

Và một **constraint mềm** (learned):

$$\omega_\theta \text{ phải equivariant với group action của fiber } G$$

Tức là với $g \in G$ acting trên $E$:

$$\omega_\theta(R_g \cdot v) = \mathrm{Ad}_g \cdot \omega_\theta(v)$$

---

## 2. Vấn đề: Neural Network thông thường KHÔNG thỏa mãn các constraints này

```
Một MLP thông thường:
ω_θ(v) = W_n · σ(W_{n-1} · ... · σ(W_1 · v))

Vấn đề 1: Không đảm bảo ω_θ(v) = 0 với v ∈ H_e(E)
           → Phá vỡ horizontal/vertical decomposition

Vấn đề 2: Output nằm trong R^k tùy ý
           → Không đảm bảo output ∈ g_fiber (Lie algebra)

Vấn đề 3: Không có equivariance
           → Gradient distort khi fiber bị transformed
```

Cần thiết kế kiến trúc có **cấu trúc hình học được bake in**.

---

## 3. Decomposition tổng quát của $\omega_\theta$

### 3.1 Tách $\omega_\theta$ thành hai phần

Tại điểm $e = (b, f) \in E$ với $b \in B$, $f \in F$, mọi vector $v \in T_e(E)$ viết được:

$$v = v_H + v_V, \quad v_H \in H_e(E),\ v_V \in V_e(E)$$

$\omega_\theta$ chỉ "nhìn thấy" $v_V$:

$$\omega_\theta(v) = \omega_\theta(v_V) = \Phi_\theta(v_V, b, f)$$

trong đó $\Phi_\theta: V_e(E) \times B \times F \to \mathfrak{g}$ là neural network cần thiết kế.

### 3.2 Cách tách $v_V$ từ $v$

Dùng **Vertical Projector** $\mathcal{P}_V$:

$$\mathcal{P}_V = \mathrm{Id} - J_\pi^\dagger J_\pi$$

trong đó $J_\pi = d\pi_e$ là Jacobian của projection $\pi: E \to B$, và $J_\pi^\dagger$ là Moore-Penrose pseudoinverse.

$$v_V = \mathcal{P}_V \cdot v, \qquad v_H = (\mathrm{Id} - \mathcal{P}_V) \cdot v$$

---

## 4. Kiến trúc $\Phi_\theta$: Lie Algebra Network (LANet)

### 4.1 Yêu cầu output nằm trong $\mathfrak{g}$

Với $SO(3)$, Lie algebra $\mathfrak{so}(3)$ là tập các **skew-symmetric matrices** $3\times 3$:

$$\mathfrak{so}(3) = \left\{ \Omega \in \mathbb{R}^{3\times 3} \mid \Omega^\top = -\Omega \right\}$$

Mọi $\Omega \in \mathfrak{so}(3)$ có thể viết:

$$\Omega = \begin{pmatrix} 0 & -\omega_3 & \omega_2 \\ \omega_3 & 0 & -\omega_1 \\ -\omega_2 & \omega_1 & 0 \end{pmatrix} = [\boldsymbol{\omega}]_\times, \quad \boldsymbol{\omega} \in \mathbb{R}^3$$

**Trick:** Thay vì output $\Omega$ trực tiếp, network output $\boldsymbol{\omega} \in \mathbb{R}^3$ rồi map lên $\mathfrak{so}(3)$:

$$\Phi_\theta(v_V, b, f) = \left[\mathrm{MLP}_\theta(v_V, b, f)\right]_\times$$

Điều này **đảm bảo output luôn nằm trong $\mathfrak{so}(3)$** — không cần constraint thêm.

### 4.2 Kiến trúc tổng thể của LANet

$$\underbrace{v_V \in \mathbb{R}^{d_V}}_{\text{vertical input}} \xrightarrow{\text{Embed}} h_1 \xrightarrow{\text{Transformer Block}} h_2 \xrightarrow{\text{Fiber Context}} h_3 \xrightarrow{\text{Lie Head}} \boldsymbol{\omega} \in \mathbb{R}^3 \xrightarrow{[\cdot]_\times} \Omega \in \mathfrak{so}(3)$$

Chi tiết từng block:

**Block 1 — Vertical Embedding:**
$$h_1 = \mathrm{LayerNorm}(W_{\text{emb}} \cdot v_V + b_{\text{emb}})$$

**Block 2 — Base Context Injection** (biết "đang ở đâu" trên Base):
$$h_2 = h_1 + \mathrm{CrossAttention}(Q=h_1,\ K=\phi_B(b),\ V=\phi_B(b))$$

trong đó $\phi_B: B \to \mathbb{R}^{d_B}$ là learned embedding của base point $b$.

**Block 3 — Fiber State Injection** (biết "trạng thái" của fiber):
$$h_3 = h_2 + W_f \cdot \phi_F(f)$$

với $\phi_F: F \to \mathbb{R}^{d_F}$ encode trạng thái fiber hiện tại.

**Block 4 — Lie Head:**
$$\boldsymbol{\omega} = W_{\text{lie}} \cdot \tanh(h_3) \in \mathbb{R}^3$$
$$\Omega = [\boldsymbol{\omega}]_\times \in \mathfrak{so}(3)$$

---

## 5. Equivariance Constraint — Bake vào kiến trúc

### 5.1 Yêu cầu

Với group action $\rho: G \times F \to F$, cần:

$$\Phi_\theta(\rho(g) \cdot v_V,\ b,\ \rho(g) \cdot f) = \mathrm{Ad}_g \cdot \Phi_\theta(v_V, b, f)$$

### 5.2 Giải pháp: Equivariant Layer dùng Group Convolution

Thay Block 3 bằng **$G$-equivariant layer**:

$$h_3 = \bigoplus_{g \in G} W_g \cdot \phi_F(\rho(g^{-1}) \cdot f) \cdot \rho(g) \cdot h_2$$

Với $SO(3)$ liên tục, tích phân thay vì tổng:

$$h_3 = \int_{SO(3)} \kappa_\theta(R) \cdot \phi_F(R^{-1} \cdot f) \, d\mu(R)$$

trong đó $\kappa_\theta: SO(3) \to \mathbb{R}$ là **learned kernel** trên group.

**Trong thực tế**, xấp xỉ bằng **Spherical Harmonics expansion** (truncated):

$$\kappa_\theta(R) \approx \sum_{\ell=0}^{L} \sum_{m=-\ell}^{\ell} c_{\ell m}^\theta \cdot Y_\ell^m(R)$$

---

## 6. Training $\omega_\theta$ — Loss Function

### 6.1 Loss chính: Enforce Horizontal = 0

$$\mathcal{L}_{\text{conn}} = \mathbb{E}_{v \sim \nabla L}\left[\left\|\omega_\theta\!\left((\mathrm{Id} - \mathcal{P}_V) \cdot v\right)\right\|_{\mathfrak{g}}^2\right]$$

Tức là: gradient của horizontal vectors phải bị map về 0.

### 6.2 Loss phụ: Equivariance residual

$$\mathcal{L}_{\text{equiv}} = \mathbb{E}_{g \sim G,\, v \sim T_e E}\left[\left\|\Phi_\theta(\rho(g) \cdot v_V, b, \rho(g) \cdot f) - \mathrm{Ad}_g \cdot \Phi_\theta(v_V, b, f)\right\|_{\mathfrak{g}}^2\right]$$

### 6.3 Loss phụ: Curvature regularization

Để tránh connection quá cong (gây holonomy lớn), penalize curvature $2$-form $\Omega_\omega = d\omega + \frac{1}{2}[\omega, \omega]$:

$$\mathcal{L}_{\text{curv}} = \left\|\Omega_\omega\right\|_{L^2}^2$$

### 6.4 Total loss

$$\mathcal{L}_{\text{total}} = \underbrace{\mathcal{L}_T + \mathcal{L}_A + \mathcal{L}_M}_{\text{task losses}} + \lambda_1 \mathcal{L}_{\text{conn}} + \lambda_2 \mathcal{L}_{\text{equiv}} + \lambda_3 \mathcal{L}_{\text{curv}}$$

---

## 7. Gradient Flow qua $\omega_\theta$ — Chain Rule đầy đủ

Trong backward pass, $\omega_\theta$ tự thân cũng nhận gradient:

$$\frac{\partial \mathcal{L}_{\text{total}}}{\partial \theta} = \frac{\partial \mathcal{L}_{\text{conn}}}{\partial \omega_\theta} \cdot \frac{\partial \omega_\theta}{\partial \theta} + \frac{\partial \mathcal{L}_{\text{equiv}}}{\partial \omega_\theta} \cdot \frac{\partial \omega_\theta}{\partial \theta}$$

**Điểm tinh tế:** Gradient qua Lie Head $[\cdot]_\times$ là:

$$\frac{\partial \mathcal{L}}{\partial \boldsymbol{\omega}} = \frac{\partial \mathcal{L}}{\partial \Omega} \cdot \frac{\partial [\boldsymbol{\omega}]_\times}{\partial \boldsymbol{\omega}}$$

Vì $[\boldsymbol{\omega}]_\times$ là linear map, Jacobian của nó là hằng số:

$$\frac{\partial [\omega]_\times^{ij}}{\partial \omega_k} = \epsilon_{ijk}$$

trong đó $\epsilon_{ijk}$ là Levi-Civita symbol — **hoàn toàn differentiable, không cần approximation.**

---

## 8. Toàn bộ kiến trúc tổng hợp

$$
\boxed{
\begin{aligned}
&\text{INPUT: } x_{\text{text}} \\
&\downarrow \text{ Encoder (Transformer)} \\
&b \in B \subset \mathbb{R}^{d_{\text{base}}} \quad \text{(Base embedding)} \\
&\downarrow \text{ Section } \sigma_i(b) \\
&f_i \in F_i \quad \text{(Fiber state cho mỗi modality)} \\
&\downarrow \text{ LANet } \Phi_\theta \\
&\Omega_i = [\mathrm{MLP}_\theta(v_V, b, f_i)]_\times \in \mathfrak{so}(3) \quad \text{(Connection output)} \\
&\downarrow \text{ Vertical/Horizontal split} \\
&\nabla_V L_i \to \text{update Decoder}_i \quad \text{(fiber-specific)} \\
&\nabla_H L_i \to \text{update } b \quad \text{(shared base)} \\
&\downarrow \text{ Riemannian update} \\
&b_{t+1} = \exp_b\!\left(-\eta \sum_i \nabla_H L_i\right)
\end{aligned}
}
$$

---

## 9. Tóm tắt: Tại sao kiến trúc này hoạt động

| Yêu cầu                       | Giải pháp                                                    |
| ----------------------------- | ------------------------------------------------------------ |
| Output $\in \mathfrak{so}(3)$ | Lie Head $[\cdot]_\times$ — baked in, không cần constraint   |
| $\omega(v_H) = 0$             | Vertical Projector $\mathcal{P}_V$ trước khi vào network     |
| Equivariance với $G$          | Spherical Harmonics kernel trên $SO(3)$                      |
| Differentiable hoàn toàn      | Jacobian qua $[\cdot ]_{ \times }$ là $\epsilon_{ijk}$ — constant |
| Không overfit geometry        | Curvature regularization $\mathcal{L}_{\text{curv}}$         |

---

Bước tiếp theo tự nhiên là **chứng minh convergence** của scheme này — liệu $\nabla_H L_i$ từ các tasks có dần align nhau không (Điều kiện 4 của Emergent Ability)? Bạn muốn tiếp tục không?

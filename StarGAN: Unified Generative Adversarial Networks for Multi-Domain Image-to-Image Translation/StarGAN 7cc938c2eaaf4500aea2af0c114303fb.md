# StarGAN

**StarGAN: Unified Generative Adversarial Networks
for Multi-Domain Image-to-Image Translation**

---

## Abstract

- 이전의 Image-to-Image translation task의 접근방법에서는 여러 도메인에 대한 학습을 진행할 때에 다수의 모델이 생성되어야만 했음.
- StarGAN에서는 동시에 여러 도메인의 데이터를 학습할 수 있고, 이것이 StarGAN에서 생성되는 이미지의 퀄리티를 높였음(결국 여러 모델들이 아닌 하나의 모델에 많은 데이터가 들어가기 때문에).

## Star Generative Adversarial Networks

### Multi-Domain Image-to-Image Translation

StarGAN의 최종 목표는 여러 도메인의 특징을 학습하는 generator G를 학습시키는 것이다. 따라서 G가 입력 이미지인 x를 타겟 도메인 c에 대한 출력 이미지인 y로 만들도록 학습한다($G(x,c)$→$y$). 컨디션인 c는 랜덤하게 생성되는데, 이는 G가 이미지 변환의 유연성을 가질 수 있게 하기 위해서이다.

auxiliary classifier을 추가해 discriminator이 여러 도메인을 컨드롤 할 수 있도록 한다.

**Adversarial Loss** : 생성된 이미지가 진짜 이미지와 구분되지 않도록 한다.

![Untitled](StarGAN%207cc938c2eaaf4500aea2af0c114303fb/Untitled.png)

- $G(x,c)$ : $G$ 는 이미지의 생성자로써, input image인 $x$ 와 target domain label인 $c$ 를 입력 받아 이미지를 생성한다.
- $D_{src}(x)$ : $D$ 는 판별자로써, 진짜와 가짜 이미지를 판별한다. 본 논문에서는 $D_{src}(x)$ 를 $D$ 가 내보낸 source의 확률분포로 정의한다.
- 결국 $G$ 는 loss 값을 최소화 하려고 하며 $D$ 는 loss 값을 최대화 하려고 한다.

**Domain Classification Loss**

- input image인 $x$ 와 target domain label인 $c$ 를 통해 $x$ 를 output image인 $y$ 로 변환하는 것이 목표이다. 즉 생성된 이미지가 적절하게 domain $c$ 로 분류되도록 하는 것이다. → auxiliary classifier을 $D$  위에 추가해 $D$  와 $G$ 를 최적화할 때 classification loss를 부과한다.

![Untitled](StarGAN%207cc938c2eaaf4500aea2af0c114303fb/Untitled%201.png)

- $D_{cls}(\acute{c}|x)$ : $D$ 가 계산한 domain label에 대한 확률분포로 정의된다.
- 이 식의 값이 작아지며 $D$ 는 진짜 이미지를 원본 도메인인 $\acute{c}$ 로 구분하는 것을 학습한다(input image and label인 $(x,\acute{c})$ 는 학습 데이터에서 주어진다).

![Untitled](StarGAN%207cc938c2eaaf4500aea2af0c114303fb/Untitled%202.png)

- 가짜 이미지의 도메인 분류를 위한 손실 함수

**Reconstruction Loss** : ****CycleGAN의 cycle-consistency loss와 같은 개념

![Untitled](StarGAN%207cc938c2eaaf4500aea2af0c114303fb/Untitled%203.png)

- $G(G(x,c),\acute{c})$ : 생성된 이미지를 다시 원본 이미지로 돌렸을 때 원본이미지 $x$ 와의 차이를 통해 loss 값을 책정

**Full Objective**

![Untitled](StarGAN%207cc938c2eaaf4500aea2af0c114303fb/Untitled%204.png)

- $\lambda_{cls}=1,\lambda_{rec}=10$

## Mask Vector

- 데이터셋에 성별 등에 관한 label은 있었으나, 표정과 같은 label은 없었다. 이 문제를 해결하기 위해 mask vector을 제안하였다.

![Untitled](StarGAN%207cc938c2eaaf4500aea2af0c114303fb/Untitled%205.png)

- n차원의 one-hot vector을 사용하였다.
- $m$ : mask vector, $n$ : 데이터셋의 수, $[\cdot]$ : concatenation, $c_i$ : $i$번째 데이터셋의 label에 대한 vector
- Known label은 zero value를 배정한다.
- 이를 통해 mask vector에 어떤 dataset인지를 명시하여 해당 dataset의 attribute에 관련된 label에 집중한다.

## Implementation

**Improved GAN Training**

![Untitled](StarGAN%207cc938c2eaaf4500aea2af0c114303fb/Untitled%206.png)

- $\lambda_{gp}=10$
- $\lambda_{gp}E_{\hat{x}}[(||\triangledown_{\hat{x}}D_{src}(\hat{x})||_2-1)^2]$ : Gradient penalty, 값을 1에 가깝게 만들어 이를 통해 더욱 퀄리티가 좋은 이미지를 생성하도록 한다.
    - $\hat{x}$ 는 한 쌍의 실제 이미지와 생성된 이미지 사이의 직선을 따라 균일하게 랜덤 샘플링된다.

**Network Architecture**

- Generator은 2개의 convolutional layer과, 6개의 residual block, 2개의 transposed convolutional layer로 구성된다.
- Generator은 Instance Normalization를 사용한다(Discriminator은 normalization 사용 x).
- Discriminator은 이미지를 patch 별로 나누어 real/fake를 판별한다.
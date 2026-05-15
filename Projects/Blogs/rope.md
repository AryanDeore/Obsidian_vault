---
title: "RoPE: Rotary Position Embedding"
status: "published"
author:
  name: "Aryan Deore"
  picture: ""
slug: "rope"
description: "From first principles, the small mathematical change that unlocked long-context reasoning"
coverImage: "//media/absolute_vs_rope-U4Mj.png"
audioSrc: "https://aryan-portfolio-692147994069.s3.us-east-2.amazonaws.com/blogs/audio/blog1_rope.mp3"
publishedAt: "2026-01-22T12:00:00.000Z"
tags:
  - "RoPE"
  - "Transformers"
  - "LLM"
---

When a transformer receives token embeddings, it does not see a sequence. It sees a bag of tokens—a jumbled set of words with no inherent order. The transformer has no idea which word comes before or after another.\
So the sentence `The quick brown fox jumps over the lazy dog` is indistinguishable from `brown dog fox jumps lazy over quick the the`.

> How do we give tokens a sense of position?

The earliest transformers answered this very literally—by adding a position embedding to the token embedding. This came to be known as Absolute Positional Embedding.

```python
token_with_position = token_vector + position_vector
```

At first glance, this seems reasonable. But consider two sentences:

- Sentence A: The **blue** sky is bright.
- Sentence B: I love the **blue** sky.

With absolute positional embeddings, the token "blue" is represented as:

- In Sentence A: `embedding(blue) + position(1)`
- In Sentence B: `embedding(blue) + position(3)`

Even though "**blue**" means the same thing in both sentences, its vector representation is now different—because we added something to it. This means the model must relearn the same relationships across positions.

> So how do we encode position into a vector without changing its meaning?

We don't add position to the vector—we rotate it by an angle that represents position. Position of the vector maps directly to rotation:

- position 0 → rotation by 0
- position 1 → rotation by $\theta$
- position $k$ → rotation by $k \times \theta$

The further the token, the more it is rotated. In other words:

```python
angle = position × base_angle
```

> But how do we rotate a 4096-dimensional vector?

By decomposing it into 2D pairs. A 4096-D vector becomes 2048 independent 2-D vectors. A single large rotation is replaced by many small, pairwise rotations. To rotate a 2D vector $v$, we multiply it by a rotation matrix $R(\theta)$:

$$
v' = R(\theta) \cdot v
$$

where:

$$
R(\theta) = \begin{bmatrix} \cos\theta & -\sin\theta \\ \sin\theta & \cos\theta \end{bmatrix}
$$

This rotation is applied to every 2D pair in the embedding, using an angle derived from the token's position.

Now we have a way to rotate each part of the embedding based on position. But if all pairs rotate at the same speed, then after a while, distant words can begin to look like nearby ones. The sense of distance is lost. Think of it like a ruler that only has centimeter markings- you can measure small distances precisely, but large distances blur together. A good ruler has millimeters, centimeters, and meters—so you can measure both small and large distances without losing relative scale. RoPE uses the same principle. It assigns a different rotational speed to each pair:

- Early pairs rotate faster, later pairs rotate slower. The rotation angle for the $m$-th pair at position $p$ is:

$$
\theta_m = p \times 10000^{\left(-\frac{2m}{d}\right)}
$$

So the entire RoPE algorithm is surprisingly simple:

1. Decompose the embedding into 2D pairs
2. Assign each pair a rotation frequency (faster for early pairs, slower for later)
3. Compute the rotation angle from $\text{position} \times \text{frequency}$
4. Rotate each pair using a 2D rotation matrix

This is plain trigonometric rotation, not learned weights. This means models can reason with longer sequences than they've ever seen during training.
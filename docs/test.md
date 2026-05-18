# 一级标题测试(1)
一级标题将在正文中显示 右侧显示二级标题 从二级标题开始编号是最佳的  
## Code Test
```python
print("Hello world!")
# 代码测试
def main():
    print("Wiki")
if __name__ == "__main__":
    main()
```
### 三级标题测试(1)
三级标题测试1。 
### 三级标题测试(2)
三级标题测试2。  

## other Test
```C
#include <stdio.h>
```
`Code Block Test`


### 视频测试
[29 残差网络 ResNet\[动手学深度学习v2\]](https://www.bilibili.com/video/BV1bV41177ap)

### 公式测试
放公式时前后需要空一格才能在mkdocs中正常显示。

$$ 
\mathcal{L}(\theta) = \sum_{i=1}^{n} \left(y_i - f_\theta(x_i)\right)^2 + \lambda \lVert \theta \rVert_2^2 
$$

行内公式：$f_\theta(x) = \theta_0 + \theta_1 x$ 可以直接书写。

参考官方文档 [mkdocs](https://squidfunk.github.io/mkdocs-material/reference/math/#mathjax-mkdocsyml) :   
**Using block syntax**  
Blocks must be enclosed in `#!latex $$...$$` or `#!latex \[...\]` on separate
lines:

``` latex title="block syntax"

$$
\cos x=\sum_{k=0}^{\infty}\frac{(-1)^k}{(2k)!}x^{2k}
$$

```

$$
\cos x=\sum_{k=0}^{\infty}\frac{(-1)^k}{(2k)!}x^{2k}
$$

**Using inline block syntax**  
Inline blocks must be enclosed in `#!latex $...$` or `#!latex \(...\)`:

``` latex title="inline syntax"
The homomorphism $f$ is injective if and only if its kernel is only the
singleton set $e_G$, because otherwise $\exists a,b\in G$ with $a\neq b$ such
that $f(a)=f(b)$.
```

The homomorphism $f$ is injective if and only if its kernel is only the
singleton set $e_G$, because otherwise $\exists a,b\in G$ with $a\neq b$ such
that $f(a)=f(b)$.


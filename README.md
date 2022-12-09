# 从线性代数到张量分解

[![MIT License](https://img.shields.io/badge/license-MIT-green.svg)](https://opensource.org/licenses/MIT)
![Python 3.7](https://img.shields.io/badge/Python-3.7-blue.svg)
[![GitHub stars](https://img.shields.io/github/stars/xinychen/tensor-book.svg?logo=github&label=Stars&logoColor=white)](https://github.com/xinychen/tensor-book)

<h6 align="center">Made by Xinyu Chen (陈新宇) • :globe_with_meridians: <a href="https://xinychen.github.io">https://xinychen.github.io</a></h6>

## 目录

- **Kronecker积与Kronecker分解**
  - Kronecker积定义
  - Kronecker积基本性质
  - Kronecker积特殊性质
  - 朴素Kronecker积分解
  - 广义Kronecker积分解
  - 模型参数压缩问题

## 作者申明

- 撰写本文的初衷在于传播知识，为感兴趣的读者提供参考素材。
- 禁止将本文放在其他网站上，唯一下载网址为[https://xinychen.github.io/books/tensor_book.pdf](https://xinychen.github.io/books/tensor_book.pdf)。
- 禁止将本文用于任何形式的商业活动。

<h2 align="center">代数结构</h2>
<p align="right"><a href="#从线性代数到张量分解"><sup>▴ 回到顶部</sup></a></p>

**例.** 写出张量的lateral切片与horizontal切片。

```python
import numpy as np

X = np.zeros((2, 2, 2))
X[:, :, 0] = np.array([[1, 2], [3, 4]])
X[:, :, 1] = np.array([[5, 6], [7, 8]])
print('lateral slices:')
print(X[:, 0, :])
print()
print(X[:, 1, :])
print()
print('horizonal slices:')
print(X[0, :, :])
print()
print(X[1, :, :])
print()
```

<h2 align="center">Kronecker分解与Kronecker分解</h2>
<p align="right"><a href="#从线性代数到张量分解"><sup>▴ 回到顶部</sup></a></p>

**例.** 计算Kronecker积并求Kronecker分解。

- 计算Kronecker积

```python
import numpy as np

A = np.array([[1, 2], [3, 4]])
B = np.array([[5, 6, 7], [8, 9, 10]])
X = np.kron(A, B)
print('X = ')
print(X)
```

- 求Kronecker分解

```python
def permute(mat, A_dim1, A_dim2, B_dim1, B_dim2):
    ans = np.zeros((A_dim1 * A_dim2, B_dim1 * B_dim2))
    for j in range(A_dim2):
        for i in range(A_dim1):
            ans[A_dim1 * j + i, :] = mat[i * B_dim1 : (i + 1) * B_dim1,
                                         j * B_dim2 : (j + 1) * B_dim2].reshape(B_dim1 * B_dim2, order = 'F')
    return ans

X_tilde = permute(X, 2, 2, 2, 3)
print('X_tilde = ')
print(X_tilde)
print()
u, s, v = np.linalg.svd(X_tilde, full_matrices = False)
A_hat = np.sqrt(s[0]) * u[:, 0].reshape(2, 2, order = 'F')
B_hat = np.sqrt(s[0]) * v[0, :].reshape(2, 3, order = 'F')
print('A_hat = ')
print(A_hat)
print()
print('B_hat = ')
print(B_hat)
```

**例.** 采用广义 Kronecker 分解重构灰度图像。

```python
import numpy as np

def permute(mat, A_dim1, A_dim2, B_dim1, B_dim2):
    ans = np.zeros((A_dim1 * A_dim2, B_dim1 * B_dim2))
    for j in range(A_dim2):
        for i in range(A_dim1):
            ans[A_dim1 * j + i, :] = mat[i * B_dim1 : (i + 1) * B_dim1,
                                         j * B_dim2 : (j + 1) * B_dim2].reshape(B_dim1 * B_dim2, order = 'F')
    return ans

def kron_decomp(mat, A_dim1, A_dim2, B_dim1, B_dim2, rank):
    mat_tilde = permute(mat, A_dim1, A_dim2, B_dim1, B_dim2)
    u, s, v = np.linalg.svd(mat_tilde, full_matrices = False)
    A_hat = np.zeros((A_dim1, A_dim2, rank))
    B_hat = np.zeros((B_dim1, B_dim2, rank))
    for r in range(rank):
        A_hat[:, :, r] = np.sqrt(s[r]) * u[:, r].reshape([A_dim1, A_dim2], order = 'F')
        B_hat[:, :, r] = np.sqrt(s[r]) * v[r, :].reshape([B_dim1, B_dim2], order = 'F')
    mat_hat = np.zeros(mat.shape)
    for r in range(rank):
        mat_hat += np.kron(A_hat[:, :, r], B_hat[:, :, r])
    return mat_hat
```

```python
from skimage import color
from skimage import io

img = io.imread('data/gaint_panda.bmp')
imgGray = color.rgb2gray(img)

io.imshow(imgGray)
plt.axis('off')
plt.imsave('gaint_panda_gray.png', imgGray, cmap = plt.cm.gray)
plt.show()
```

```python
import imageio
import matplotlib.pyplot as plt

img = io.imread('data/gaint_panda.bmp')
mat = color.rgb2gray(img)
io.imshow(mat)
plt.axis('off')
plt.show()

A_dim1 = 16
A_dim2 = 32
B_dim1 = 32
B_dim2 = 16
for rank in [5, 10, 50, 100]:
    mat_hat = kron_decomp(mat, A_dim1, A_dim2, B_dim1, B_dim2, rank)
    mat_hat[mat_hat > 1] = 1
    mat_hat[mat_hat < 0] = 0
    io.imshow(mat_hat)
    plt.axis('off')
    plt.imsave('gaint_panda_gray_R{}.png'.format(rank), 
              mat_hat, cmap = plt.cm.gray)
    plt.show()
```

<h2 align="center">模态积与Tucker张量分解</h2>
<p align="right"><a href="#从线性代数到张量分解"><sup>▴ 回到顶部</sup></a></p>

**例.** 计算张量与矩阵的模态积。

```python
import numpy as np

X = np.zeros((2, 2, 2))
X[:, :, 0] = np.array([[1, 2], [3, 4]])
X[:, :, 1] = np.array([[5, 6], [7, 8]])
A = np.array([[1, 2], [3, 4], [5, 6]])
Y = np.einsum('ijh, ki -> kjh', X, A)
print('frontal slices of Y:')
print(Y[:, :, 0])
print()
print(Y[:, :, 1])
print()
```

<h2 align="center">低秩时序模型</h2>
<p align="right"><a href="#从线性代数到张量分解"><sup>▴ 回到顶部</sup></a></p>

**例.** 使用考虑平滑处理的矩阵分解对灰度图像进行复原。

```python
import numpy as np

def compute_rse(var, var_hat):
    return np.linalg.norm(var - var_hat, 2) / np.linalg.norm(var, 2)

def generate_Psi(n):
    mat1 = np.append(np.zeros((n - 1, 1)), np.eye(n - 1), axis = 1)
    mat2 = np.append(np.eye(n - 1), np.zeros((n - 1, 1)), axis = 1)
    Psi = mat1 - mat2
    return Psi

def update_cg(var, r, q, Aq, rold):
    alpha = rold / np.inner(q, Aq)
    var = var + alpha * q
    r = r - alpha * Aq
    rnew = np.inner(r, r)
    q = r + (rnew / rold) * q
    return var, r, q, rnew

def ell_w(ind, W, X, Psi1, rho, lmbda):
    return X @ ((W.T @ X) * ind).T + rho * W + lmbda * W @ Psi1.T @ Psi1

def conj_grad_w(sparse_mat, ind, W, X, Psi1, rho, lmbda, maxiter = 5):
    rank, dim1 = W.shape
    w = np.reshape(W, -1, order = 'F')
    r = np.reshape(X @ sparse_mat.T 
                   - ell_w(ind, W, X, Psi1, rho, lmbda), -1, order = 'F')
    q = r.copy()
    rold = np.inner(r, r)
    for it in range(maxiter):
        Q = np.reshape(q, (rank, dim1), order = 'F')
        Aq = np.reshape(ell_w(ind, Q, X, Psi1, rho, lmbda), -1, order = 'F')
        w, r, q, rold = update_cg(w, r, q, Aq, rold)
    return np.reshape(w, (rank, dim1), order = 'F')

def ell_x(ind, W, X, Psi2, rho, lmbda):
    return W @ ((W.T @ X) * ind) + rho * X + lmbda * X @ Psi2.T @ Psi2

def conj_grad_x(sparse_mat, ind, W, X, Psi2, rho, lmbda, maxiter = 5):
    rank, dim2 = X.shape
    x = np.reshape(X, -1, order = 'F')
    r = np.reshape(W @ sparse_mat 
                   - ell_x(ind, W, X, Psi2, rho, lmbda), -1, order = 'F')
    q = r.copy()
    rold = np.inner(r, r)
    for it in range(maxiter):
        Q = np.reshape(q, (rank, dim2), order = 'F')
        Aq = np.reshape(ell_x(ind, W, Q, Psi2, rho, lmbda), -1, order = 'F')
        x, r, q, rold = update_cg(x, r, q, Aq, rold)
    return np.reshape(x, (rank, dim2), order = 'F')

def smoothing_mf(dense_mat, sparse_mat, rank, rho, lmbda, maxiter = 50):
    dim1, dim2 = sparse_mat.shape
    W = 0.01 * np.random.randn(rank, dim1)
    X = 0.01 * np.random.randn(rank, dim2)
    ind = sparse_mat != 0
    pos_test = np.where((dense_mat != 0) & (sparse_mat == 0))
    dense_test = dense_mat[pos_test]
    del dense_mat
    Psi1 = generate_Psi(dim1)
    Psi2 = generate_Psi(dim2)
    show_iter = 10
    for it in range(maxiter):
        W = conj_grad_w(sparse_mat, ind, W, X, Psi1, rho, lmbda)
        X = conj_grad_x(sparse_mat, ind, W, X, Psi2, rho, lmbda)
        mat_hat = W.T @ X
        if (it + 1) % show_iter == 0:
            temp_hat = mat_hat[pos_test]
            print('Iter: {}'.format(it + 1))
            print(compute_rse(temp_hat, dense_test))
            print()
    return mat_hat, W, X
```

```python
import numpy as np
np.random.seed(1)
import matplotlib.pyplot as plt
from skimage import color
from skimage import io

img = io.imread('data/gaint_panda.bmp')
imgGray = color.rgb2gray(img)
M, N = imgGray.shape
missing_rate = 0.9

sparse_img = imgGray * np.round(np.random.rand(M, N) + 0.5 - missing_rate)
io.imshow(sparse_img)
plt.axis('off')
plt.show()
```

```python
lmbda = 1e+1
for rank in [5, 10, 50]:
    rho = 1e-1
    maxiter = 100
    mat_hat, W, X = smoothing_mf(imgGray, sparse_img, rank, rho, lmbda, maxiter)

    mat_hat[mat_hat < 0] = 0
    mat_hat[mat_hat > 1] = 1
    io.imshow(mat_hat)
    plt.axis('off')
    plt.imsave('gaint_panda_gray_recovery_90_mf_rank_{}_lmbda_10.png'.format(rank), 
              mat_hat, cmap = plt.cm.gray)
    plt.show()
```

```python
lmbda = 1e-10
for rank in [5, 10, 50]:
    rho = 1e-1
    maxiter = 100
    mat_hat, W, X = smoothing_mf(imgGray, sparse_img, rank, rho, lmbda, maxiter)

    mat_hat[mat_hat < 0] = 0
    mat_hat[mat_hat > 1] = 1
    io.imshow(mat_hat)
    plt.axis('off')
    plt.imsave('gaint_panda_gray_recovery_90_mf_rank_{}_lmbda_0.png'.format(rank), 
              mat_hat, cmap = plt.cm.gray)
    plt.show()
```

**例.** 使用考虑空间自回归的矩阵分解对灰度图像进行复原。

```python
import numpy as np

def compute_rse(var, var_hat):
    return np.linalg.norm(var - var_hat, 2) / np.linalg.norm(var, 2)

def generate_Psi(n):
    mat1 = np.append(np.zeros((n - 1, 1)), np.eye(n - 1), axis = 1)
    mat2 = np.append(np.eye(n - 1), np.zeros((n - 1, 1)), axis = 1)
    return mat1, mat2

def update_cg(var, r, q, Aq, rold):
    alpha = rold / np.inner(q, Aq)
    var = var + alpha * q
    r = r - alpha * Aq
    rnew = np.inner(r, r)
    q = r + (rnew / rold) * q
    return var, r, q, rnew

def ell_w(ind, W, X, Aw, Phi0, Phi1, rho, lmbda):
    temp1 = W @ Phi0.T - Aw @ W @ Phi1.T
    temp2 = lmbda * temp1 @ Phi0 - lmbda * Aw.T @ temp1 @ Phi1
    return X @ ((W.T @ X) * ind).T + rho * W + temp2

def conj_grad_w(sparse_mat, ind, W, X, Aw, Phi0, Phi1, rho, lmbda, maxiter = 5):
    rank, dim1 = W.shape
    w = np.reshape(W, -1, order = 'F')
    r = np.reshape(X @ sparse_mat.T 
                   - ell_w(ind, W, X, Aw, Phi0, Phi1, rho, lmbda), -1, order = 'F')
    q = r.copy()
    rold = np.inner(r, r)
    for it in range(maxiter):
        Q = np.reshape(q, (rank, dim1), order = 'F')
        Aq = np.reshape(ell_w(ind, Q, X, Aw, Phi0, Phi1, rho, lmbda), -1, order = 'F')
        w, r, q, rold = update_cg(w, r, q, Aq, rold)
    return np.reshape(w, (rank, dim1), order = 'F')

def ell_x(ind, W, X, Ax, Psi0, Psi1, rho, lmbda):
    temp1 = X @ Psi0.T - Ax @ X @ Psi1.T
    temp2 = lmbda * temp1 @ Psi0 - lmbda * Ax.T @ temp1 @ Psi1
    return W @ ((W.T @ X) * ind) + rho * X + temp2

def conj_grad_x(sparse_mat, ind, W, X, Ax, Psi0, Psi1, rho, lmbda, maxiter = 5):
    rank, dim2 = X.shape
    x = np.reshape(X, -1, order = 'F')
    r = np.reshape(W @ sparse_mat 
                   - ell_x(ind, W, X, Ax, Psi0, Psi1, rho, lmbda), -1, order = 'F')
    q = r.copy()
    rold = np.inner(r, r)
    for it in range(maxiter):
        Q = np.reshape(q, (rank, dim2), order = 'F')
        Aq = np.reshape(ell_x(ind, W, Q, Ax, Psi0, Psi1, rho, lmbda), -1, order = 'F')
        x, r, q, rold = update_cg(x, r, q, Aq, rold)
    return np.reshape(x, (rank, dim2), order = 'F')

def spatial_autoregressive_mf(dense_mat, sparse_mat, rank, rho, lmbda, maxiter = 50):
    dim1, dim2 = sparse_mat.shape
    W = 0.01 * np.random.randn(rank, dim1)
    X = 0.01 * np.random.randn(rank, dim2)
    Aw = 0.01 * np.random.randn(rank, rank)
    Ax = 0.01 * np.random.randn(rank, rank)
    ind = sparse_mat != 0
    pos_test = np.where((dense_mat != 0) & (sparse_mat == 0))
    dense_test = dense_mat[pos_test]
    del dense_mat
    Phi0, Phi1 = generate_Psi(dim1)
    Psi0, Psi1 = generate_Psi(dim2)
    show_iter = 10
    for it in range(maxiter):
        W = conj_grad_w(sparse_mat, ind, W, X, Aw, Phi0, Phi1, rho, lmbda)
        X = conj_grad_x(sparse_mat, ind, W, X, Ax, Psi0, Psi1, rho, lmbda)
        Aw = W @ Phi0.T @ np.linalg.pinv(W @ Phi1.T)
        Ax = X @ Psi0.T @ np.linalg.pinv(X @ Psi1.T)
        mat_hat = W.T @ X
        if (it + 1) % show_iter == 0:
            temp_hat = mat_hat[pos_test]
            print('Iter: {}'.format(it + 1))
            print(compute_rse(temp_hat, dense_test))
            print()
    return mat_hat, W, X, Aw, Ax
```

```python
import numpy as np
np.random.seed(1)
import matplotlib.pyplot as plt
from skimage import color
from skimage import io

img = io.imread('data/gaint_panda.bmp')
imgGray = color.rgb2gray(img)
M, N = imgGray.shape
missing_rate = 0.9

sparse_img = imgGray * np.round(np.random.rand(M, N) + 0.5 - missing_rate)
io.imshow(sparse_img)
plt.axis('off')
plt.show()
```

```python
lmbda = 5e-1
for rank in [5, 10, 50]:
    rho = 1e-1
    maxiter = 100
    mat_hat, W, X, Aw, Ax = spatial_autoregressive_mf(imgGray, sparse_img, rank, rho, lmbda, maxiter)

    mat_hat[mat_hat < 0] = 0
    mat_hat[mat_hat > 1] = 1
    io.imshow(mat_hat)
    plt.axis('off')
    plt.imsave('gaint_panda_gray_recovery_90_spatial_ar_mf_rank_{}.png'.format(rank), 
              mat_hat, cmap = plt.cm.gray)
    plt.show()
```

**例.** 根据卷积定理计算循环卷积。

```python
import numpy as np

x = np.array([0, 1, 2, 3, 4])
y = np.array([2, -1, 3])
fx = np.fft.fft(x)
fy = np.fft.fft(np.append(y, np.zeros(2), axis = 0))
z = np.fft.ifft(fx * fy).real
```

**例.** 根据卷积定理计算向量**y**。

```python
import numpy as np

x = np.array([0, 1, 2, 3, 4])
z = np.array([5, 14, 3, 7, 11])
fx = np.fft.fft(x)
fz = np.fft.fft(z)
y = np.fft.ifft(fz / fx).real
```

**例.** 根据卷积定理计算二维循环卷积。

```python
import numpy as np

X = np.array([[1, 2, 3, 4], [4, 5, 6, 7], 
              [7, 8, 9, 10], [10, 11, 12, 13]])
K = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])
pad_K = np.zeros(X.shape)
pad_K[: K.shape[0], : K.shape[1]] = K
Y = np.fft.ifft2(np.fft.fft2(X) * np.fft.fft2(pad_K)).real
```

**例.** 使用循环矩阵核范数最小化算法对灰度图像进行复原。

```python
import numpy as np

def compute_rse(var, var_hat):
    return np.linalg.norm(var - var_hat, 2) / np.linalg.norm(var, 2)

def prox(z, w, lmbda):
    T = z.shape[0]
    temp = np.fft.fft(z - w / lmbda)
    temp1 = np.abs(temp) - T / lmbda
    temp1[temp1 <= 0] = 0
    return np.fft.ifft(temp / np.abs(temp) * temp1).real

def update_z(y_train, pos_train, x, w, lmbda, eta):
    z = x + w / lmbda
    z[pos_train] = (lmbda / (lmbda + eta) * z[pos_train] 
                    + eta / (lmbda + eta) * y_train)
    return z

def update_w(x, z, w, lmbda):
    return w + lmbda * (x - z)

def circ_nnm(y_true, y, lmbda, eta, maxiter = 50):
    pos_train = np.where(y != 0)
    y_train = y[pos_train]
    pos_test = np.where((y_true != 0) & (y == 0))
    y_test = y_true[pos_test]
    z = y.copy()
    w = y.copy()
    del y_true, y
    show_iter = 10
    for it in range(maxiter):
        x = prox(z, w, lmbda)
        z = update_z(y_train, pos_train, x, w, lmbda, eta)
        w = update_w(x, z, w, lmbda)
        if (it + 1) % show_iter == 0:
            print(it + 1)
            print(compute_rse(y_test, x[pos_test]))
            print()
    return x
```

```python
import numpy as np
np.random.seed(1)
import matplotlib.pyplot as plt
from skimage import color
from skimage import io

img = io.imread('data/gaint_panda.bmp')
imgGray = color.rgb2gray(img)
M, N = imgGray.shape
missing_rate = 0.9

sparse_img = imgGray * np.round(np.random.rand(M, N) + 0.5 - missing_rate)
io.imshow(sparse_img)
plt.axis('off')
plt.show()
```

```python
lmbda = 1e-3 * M * N
eta = 100 * lmbda
maxiter = 100
vec_hat = circ_nnm(imgGray.reshape(M * N, order = 'F'), 
                   sparse_img.reshape(M * N, order = 'F'), 
                   lmbda, eta, maxiter)

vec_hat[vec_hat < 0] = 0
vec_hat[vec_hat > 1] = 1
io.imshow(vec_hat.reshape([M, N], order = 'F'))
plt.axis('off')
plt.imsave('gaint_panda_gray_recovery_90_circ_nnm.png', 
           vec_hat.reshape([M, N], order = 'F'), cmap = plt.cm.gray)
plt.show()
```

**例.** 使用一维低秩拉普拉斯卷积模型对灰度图像进行复原。

```python
import numpy as np

def compute_rse(var, var_hat):
    return np.linalg.norm(var - var_hat, 2) / np.linalg.norm(var, 2)

def laplacian(n, tau):
    ell = np.zeros(n)
    ell[0] = 2 * tau
    for k in range(tau):
        ell[k + 1] = -1
        ell[-k - 1] = -1
    return ell

def prox(z, w, lmbda, denominator):
    T = z.shape[0]
    temp = np.fft.fft(lmbda * z - w) / denominator
    temp1 = 1 - T / (lmbda * np.abs(temp))
    temp1[temp1 <= 0] = 0
    return np.fft.ifft(temp * temp1).real

def update_z(y_train, pos_train, x, w, lmbda, eta):
    z = x + w / lmbda
    z[pos_train] = (lmbda / (lmbda + eta) * z[pos_train] 
                    + eta / (lmbda + eta) * y_train)
    return z

def update_w(x, z, w, lmbda):
    return w + lmbda * (x - z)

def lap_conv_1d(y_true, y, gamma, lmbda, eta, tau, maxiter = 50):
    T = y.shape
    pos_train = np.where(y != 0)
    y_train = y[pos_train]
    pos_test = np.where((y_true != 0) & (y == 0))
    y_test = y_true[pos_test]
    z = y.copy()
    w = y.copy()
    denominator = lmbda + gamma * np.fft.fft(laplacian(T, tau)) ** 2
    del y_true, y
    show_iter = 10
    for it in range(maxiter):
        x = prox(z, w, lmbda, denominator)
        z = update_z(y_train, pos_train, x, w, lmbda, eta)
        w = update_w(x, z, w, lmbda)
        if (it + 1) % show_iter == 0:
            print(it + 1)
            print(compute_rse(y_test, x[pos_test]))
            print()
    return x
```

```python
import numpy as np
np.random.seed(1)
import matplotlib.pyplot as plt
from skimage import color
from skimage import io

img = io.imread('data/gaint_panda.bmp')
imgGray = color.rgb2gray(img)
M, N = imgGray.shape
missing_rate = 0.9

sparse_img = imgGray * np.round(np.random.rand(M, N) + 0.5 - missing_rate)
io.imshow(sparse_img)
plt.axis('off')
plt.show()
```

```python
lmbda = 5e-3 * M * N
gamma = 1 * lmbda
eta = 100 * lmbda
tau = 1
maxiter = 100
vec_hat = lap_conv_1d(imgGray.reshape(M * N, order = 'F'), 
                      sparse_img.reshape(M * N, order = 'F'), 
                      gamma, lmbda, eta, tau, maxiter)

vec_hat[vec_hat < 0] = 0
vec_hat[vec_hat > 1] = 1
io.imshow(vec_hat.reshape([M, N], order = 'F'))
plt.axis('off')
plt.imsave('gaint_panda_gray_recovery_90_lap_conv_1d.png', 
           vec_hat.reshape([M, N], order = 'F'), cmap = plt.cm.gray)
plt.show()
```

**例.** 使用二维低秩拉普拉斯卷积模型对灰度图像进行复原。

```python
import numpy as np

def compute_rse(var, var_hat):
    return np.linalg.norm(var - var_hat, 2) / np.linalg.norm(var, 2)

def laplacian(T, tau):
    ell = np.zeros(T)
    ell[0] = 2 * tau
    for k in range(tau):
        ell[k + 1] = -1
        ell[-k - 1] = -1
    return ell

def prox(z, w, lmbda, denominator):
    T = z.shape[0]
    temp = np.fft.fft2(lmbda * z - w) / denominator
    temp1 = 1 - T / (lmbda * np.abs(temp))
    temp1[temp1 <= 0] = 0
    return np.fft.ifft2(temp * temp1).real

def update_z(y_train, pos_train, x, w, lmbda, eta):
    z = x + w / lmbda
    z[pos_train] = (lmbda / (lmbda + eta) * z[pos_train] 
                    + eta / (lmbda + eta) * y_train)
    return z

def update_w(x, z, w, lmbda):
    return w + lmbda * (x - z)

def laplacian_conv_2d(y_true, y, lmbda, gamma, eta, tau, maxiter = 50):
    M, N = y.shape
    pos_train = np.where(y != 0)
    y_train = y[pos_train]
    pos_test = np.where((y_true != 0) & (y == 0))
    y_test = y_true[pos_test]
    z = y.copy()
    w = y.copy()
    ell_1 = laplacian(M, tau)
    ell_2 = laplacian(N, tau)
    denominator = lmbda + gamma * np.fft.fft2(np.outer(ell_1, ell_2)) ** 2
    del y_true, y
    show_iter = 10
    for it in range(maxiter):
        x = prox(z, w, lmbda, denominator)
        z = update_z(y_train, pos_train, x, w, lmbda, eta)
        w = update_w(x, z, w, lmbda)
        if (it + 1) % show_iter == 0:
            print(it + 1)
            print(compute_rse(y_test, x[pos_test]))
            print()
    return x
```

```python
import numpy as np
np.random.seed(1)
import matplotlib.pyplot as plt
from skimage import color
from skimage import io

img = io.imread('data/gaint_panda.bmp')
imgGray = color.rgb2gray(img)
M, N = imgGray.shape
missing_rate = 0.9

sparse_img = imgGray * np.round(np.random.rand(M, N) + 0.5 - missing_rate)
io.imshow(sparse_img)
plt.axis('off')
plt.imsave('gaint_panda_gray_missing_rate_90.png', 
           sparse_img, cmap = plt.cm.gray)
plt.show()
```

```python
lmbda = 1e-4 * M * N
gamma = 1 * lmbda
eta = 100 * lmbda
tau = 2
maxiter = 100
mat_hat = laplacian_conv_2d(imgGray, sparse_img, lmbda, gamma, eta, tau, maxiter)

mat_hat[mat_hat < 0] = 0
mat_hat[mat_hat > 1] = 1
io.imshow(mat_hat)
plt.axis('off')
plt.imsave('gaint_panda_gray_recovery_90_gamm1_tau{}.png'.format(tau), 
           mat_hat, cmap = plt.cm.gray)
plt.show()
```

**例.** 计算延迟嵌入矩阵的奇异值分解、还原向量**x**。

```python
import numpy as np

def delay_embedding(vec, kernel_size):
    n = vec.shape[0]
    mat = np.zeros((n, kernel_size))
    mat[:, 0] = vec
    for k in range(1, kernel_size):
        mat[:, k] = np.append(vec[k :], vec[: k], axis = 0)
    return mat

vec = np.array([0, 1, 2, 3, 4])
T = vec.shape[0]
kernel_size = 3
mat = delay_embedding(vec, kernel_size)

u, s, v = np.linalg.svd(mat, full_matrices = False)
x_hat = np.zeros(T)
for r in range(kernel_size):
    fu = np.fft.fft(u[:, r])
    fv = np.fft.fft(np.append(v[r, :], np.zeros(T - kernel_size), axis = 0))
    x_hat += s[r] * np.fft.ifft(fu * fv).real
x_hat = x_hat / kernel_size
print(x_hat)
```


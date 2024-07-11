# Matrix Exponentiation

First post! This blog is pretty cool, don't you think :D

Anyways, let's talk about topic of this blog---**matrix exponentiation**. Matrix exponentiation is basically a way to solve linear recurrences in $O(\log n)$ time. To see what this means, let's take a look at a prototypical matrix exponentiation problem: [CSES Fibonacci Numbers](https://cses.fi/problemset/task/1722).

> Let $F_0=0$, $F_1=1$, and $F_n=F_{n-1}+F_{n-2}$. Given $n$ $\left(0\leq n\leq 10^{18}\right)$, calculate $F_n\mod 10^9+7$.

At first, this problem seems intractable. We obviously can't just use recursion or iteration to get to $F_n$, because $n$ can be up to $10^{18}$. We can't use an $O(1)$ Fibonacci formula either, since we're asked for the answer mod $10^9+7$, which requires exact precision. Instead, we make use of the following key idea.

**Key Idea:** An $n\times n$ matrix encodes a linear map of a $1\times n$ matrix. The linear map can be "applied" by [multiplying](https://en.wikipedia.org/wiki/Matrix_multiplication) the $1\times n$ matrix by the $n\times n$ matrix.

For example, the matrix

$$ \begin{pmatrix} 1 & 1 \\ 1 & 0 \end{pmatrix} $$

encodes the map

$$ \begin{pmatrix} a & b \end{pmatrix} \mapsto \begin{pmatrix} 1a+1b & 1a+0b \end{pmatrix} = \begin{pmatrix} a+b & a \end{pmatrix}. $$

This can be seen by just multiplying the two matrices. The $i$th column encodes the coefficients that make up the $i$th element of the row.

But wait. That same matrix can be used to encode the Fibonacci recurrence! Applying it to the first two elements yields

$$ \begin{pmatrix} F_1 & F_0 \end{pmatrix}\begin{pmatrix} 1 & 1 \\ 1 & 0 \end{pmatrix} = \begin{pmatrix}F_1+F_0 & F_1\end{pmatrix} = \begin{pmatrix}F_2 & F_1\end{pmatrix}. $$

Furthermore, we can iterate this multiplication, so that

$$ \begin{pmatrix} F_1 & F_0 \end{pmatrix}\begin{pmatrix} 1 & 1 \\ 1 & 0 \end{pmatrix}^n = \begin{pmatrix}F_{n+1} & F_n\end{pmatrix}. $$

It might seem like this matrix multiplication is the same as just calculating each element, but there's one final trick.

**Key Idea:** Matrix multiplication is *associative*, meaning that it doesn't matter which multiplications we do first.

This idea means that we can calculate

$$M=\begin{pmatrix} 1 & 1 \\ 1 & 0 \end{pmatrix}^n$$

first before multiplying the "initial state" matrix by $M$. And calculating something to the power of $n$ can be done in $O(\log n)$ time using [binary exponentiation](https://cp-algorithms.com/algebra/binary-exp.html)!

Once we compute $M$ using binary exponentiation, we can just read off the answer:

$$ \begin{pmatrix} 1 & 0 \end{pmatrix}M = \begin{pmatrix} M_{11} & M_{12} \end{pmatrix} \implies F_n = M_{12}. $$

(And if you're wondering why we chose to raise the matrix to the power of $n$ rather than $n-1$, it's because this way doesn't need a special case for $n=0$.)

## Code

Here's my code for matrix multiplication:
```cpp
V<V<Mint>> operator*(const V<V<Mint>>& a, const V<V<Mint>>& b) {
    // assume a = n * p, b = p * m
    int n = sz(a), p = sz(a[0]), m = sz(b[0]);
    V<V<Mint>> res(n, V<Mint>(m, 0));
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < m; j++) {
            for (int k = 0; k < p; k++) {
                res[i][j] += a[i][k] * b[k][j];
            }
        }
    }
    return res;
}

V<V<Mint>>& operator*=(V<V<Mint>>& a, const V<V<Mint>>& b) { return a = a * b; }
```
(A `Mint` is a class that I made; it encapsulates an integer mod some number. You can find it [here](https://github.com/ajxu2/competitive-programming/blob/main/Templates/Mint.cpp).)

And after that, we can implement binary exponentiation:
```cpp
V<V<Mint>> pow(V<V<Mint>> a, ll b) {
    // only for square matrices
    int n = sz(a);
    V<V<Mint>> res(n, V<Mint>(n, 0));
    for (int i = 0; i < n; i++) res[i][i] = 1;
    while (b > 0) {
        if (b & 1) res *= a;
        a *= a; b >>= 1;
    }
    return res;
}
```

Once that's done, the code for calculating the $n$th Fibonacci number is quite simple:
{% raw %}
```cpp
int main() {
    cin.tie(0)->sync_with_stdio(0);
    V<V<Mint>> a = {{1, 1}, {1, 0}};
    ll n; cin >> n;
    cout << pow(a, n)[0][1] << "\n";
    return 0;
}
```
{% endraw %}

And this passes all test cases on CSES!

## Other problems

Of course, it would be misguided to assume that matrix exponentiation can only calculate Fibonacci numbers. It can solve a lot of other linear recurrences. For example, [CSES Throwing Dice](https://cses.fi/problemset/task/1096) has the following recurrence:

$$ dp_i=\begin{cases}0 & i<0 \\ 1 & i=0 \\ \sum_{j=1}^6 dp_{i-j} & i > 0\end{cases} $$

and the answer pops out by calculating

$$ \begin{pmatrix}1 & 0 & 0 & 0 & 0 & 0\end{pmatrix}\begin{pmatrix}1 & 1 & 0 & 0 & 0 & 0 \\ 1 & 0 & 1 & 0 & 0 & 0 \\ 1 & 0 & 0 & 1 & 0 & 0 \\ 1 & 0 & 0 & 0 & 1 & 0 \\ 1 & 0 & 0 & 0 & 0 & 1 \\ 1 & 0 & 0 & 0 & 0 & 0\end{pmatrix}^n.$$

Furthermore, [CSES Graph Paths I](https://cses.fi/problemset/task/1723), a seemingly unrelated problem, has a beautifully simple solution: just raise the adjacency matrix to the power of $k$. (This essentially works because the adjacency matrix and the process of matrix multiplication encode the exact elements that need to be added in each stage of the DP.)

## Farewell

That's all for this blog! I've enabled comments using [giscus](https://giscus.app/), so you can leave your feedback below.

Have a great day!

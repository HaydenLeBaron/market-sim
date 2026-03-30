

# Q1: if MRS (and MRT) is a scalar in a 2-dimensional economic model, how about in N dimensions?

**Answer**:

In an \(n\)-good model, MRS and MRT are no longer single scalars globally; they are defined as families of pairwise rates, or more compactly as ratios of components of gradient vectors. [en.wikipedia](https://en.wikipedia.org/wiki/Marginal_rate_of_substitution)

## Two goods (2D) recap

For two goods \(x\) and \(y\):

- Utility \(u(x,y)\) ⇒ MRS of \(x\) for \(y\):  
  \[
  MRS_{xy} = \frac{MU_x}{MU_y} = \frac{\partial u / \partial x}{\partial u / \partial y}
  \]
  which is a single scalar at each bundle. [en.wikipedia](https://en.wikipedia.org/wiki/Marginal_rate_of_substitution)
- Similarly for a 2-good PPF \(f(x,y)=0\), MRT is the (signed) slope of the frontier, again a scalar at each point. [en.wikipedia](https://en.wikipedia.org/wiki/Production%E2%80%93possibility_frontier)

## N goods: gradient and pairwise MRS

Let there be \(n\) goods, with bundle \(x=(x_1,\dots,x_n)\) and utility \(u(x)\).

- The **marginal utilities** form the gradient vector  
  \[
  \nabla u(x) = \left(\frac{\partial u}{\partial x_1},\dots,\frac{\partial u}{\partial x_n}\right).
  \]
- For any ordered pair \((i,j)\), the **MRS of good \(i\) for good \(j\)** is still a scalar:  
  \[
  MRS_{ij}(x) = \frac{MU_i(x)}{MU_j(x)} = \frac{\partial u / \partial x_i}{\partial u / \partial x_j}.
  \]
 [en.wikipedia](https://en.wikipedia.org/wiki/Marginal_rate_of_substitution)

So in \(n\) dimensions you do not get a single global scalar MRS; you get a whole collection of scalar pairwise MRSs, one for each pair of goods, all determined by the gradient vector. The “object” that summarizes all substitution possibilities is **the gradient \(\nabla u\)** (defined up to a positive scalar multiple, since utility is ordinal).

Equivalently, you can think of MRS as the **directional trade-off** along the indifference surface: for any direction in \(\mathbb{R}^n\) tangent to the indifference surface, the implied marginal trade-offs are read from the ratios of gradient components.

## N goods: MRT

For production with \(n\) outputs (or \(n\) choice variables) constrained by a technology \(F(x_1,\dots,x_n)=0\):

- The **MRT between good \(i\) and \(j\)** at a point on the technology frontier is  
  \[
  MRT_{ij}(x) = -\frac{\partial F / \partial x_i}{\partial F / \partial x_j},
  \]
  a scalar for each pair \((i,j)\). [en.wikipedia](https://en.wikipedia.org/wiki/Production%E2%80%93possibility_frontier)

Again, the full set of possible marginal transformations is encoded in the gradient \(\nabla F(x)\); pairwise MRTs are the ratios of its components.

## Optimality condition in N dimensions

With \(n\) goods, the familiar condition “MRS = MRT” generalizes pairwise:

- At an interior optimum with a single budget or technology constraint,  
  \[
  \frac{MU_i}{MU_j} = MRT_{ij} \quad \text{for all } i,j,
  \]
  meaning the **gradient of utility is collinear with the gradient of the constraint**. [1cademy](https://1cademy.com/node/the-two-trade-offs-in-optimal-choice-mrs-vs-mrt/43T46jpzcqZG75ueJdCc)

So: in 2D, MRS/MRT are often described as *the* scalar slope; in \(n\) dimensions, the fundamental objects are gradient vectors, and the scalar MRS/MRT values appear as their pairwise component ratios.

# Q2: ok, so indifference curves are 3 dimensional with 3 goods, right? So how would I calculate MRS. Let's say the "goods" in question are "food", "water", "wood"


With three goods (food, water, wood), the indifference “curves” become 2‑dimensional surfaces living in 3‑dimensional space; MRS is still computed as a ratio of marginal utilities and is always defined **pairwise** (e.g. food for water). [en.wikipedia](https://en.wikipedia.org/wiki/Indifference_curve)

## Geometry with three goods

- A utility function is \(u(f,w,d)\), where \(f\)=food, \(w\)=water, \(d\)=wood.  
- An “indifference curve” generalizes to an **indifference surface**: the set of points \((f,w,d)\) such that \(u(f,w,d)=\bar{u}\) for some fixed utility level \(\bar{u}\). [en.wikipedia](https://en.wikipedia.org/wiki/Indifference_curve)

## Marginal utilities

Compute the marginal utilities (partial derivatives):

- \(MU_f(f,w,d) = \dfrac{\partial u(f,w,d)}{\partial f}\)  
- \(MU_w(f,w,d) = \dfrac{\partial u(f,w,d)}{\partial w}\)  
- \(MU_d(f,w,d) = \dfrac{\partial u(f,w,d)}{\partial d}\)  

These three numbers at a point \((f,w,d)\) form the gradient \(\nabla u = (MU_f,MU_w,MU_d)\), which is perpendicular to the indifference surface at that point. [econgraphs](https://www.econgraphs.org/explanations/consumer/preferences/indifference_curves)

## How to calculate MRS with 3 goods

You always pick two goods and hold utility constant (and, implicitly, allow the third to adjust along the surface). The MRS is the rate at which the consumer is willing to give up one good for the other, *keeping utility unchanged*.

At a bundle \((f,w,d)\):

- **MRS of food for water** (how many units of water the person is willing to give up if they get one extra unit of food):  
  \[
  MRS_{f,w}(f,w,d) = \frac{MU_f(f,w,d)}{MU_w(f,w,d)}.
  \]
- **MRS of food for wood**:  
  \[
  MRS_{f,d}(f,w,d) = \frac{MU_f(f,w,d)}{MU_d(f,w,d)}.
  \]
- **MRS of water for wood**:  
  \[
  MRS_{w,d}(f,w,d) = \frac{MU_w(f,w,d)}{MU_d(f,w,d)}.
  \]

Each of these is a scalar at the chosen bundle. They are related via simple algebra, for example  
\[
MRS_{f,w} = \frac{MRS_{f,d}}{MRS_{w,d}}.
\]

## Concrete example

Suppose utility is Cobb–Douglas in the three goods:
\[
u(f,w,d) = f^{\alpha} w^{\beta} d^{\gamma}, \quad \alpha,\beta,\gamma>0.
\]

Then:

- \(MU_f = \alpha f^{\alpha-1} w^{\beta} d^{\gamma}\)  
- \(MU_w = \beta f^{\alpha} w^{\beta-1} d^{\gamma}\)  
- \(MU_d = \gamma f^{\alpha} w^{\beta} d^{\gamma-1}\)

Hence:

- \(MRS_{f,w}(f,w,d) = \dfrac{MU_f}{MU_w} = \dfrac{\alpha}{\beta}\cdot \dfrac{w}{f}\)  
- \(MRS_{f,d}(f,w,d) = \dfrac{MU_f}{MU_d} = \dfrac{\alpha}{\gamma}\cdot \dfrac{d}{f}\)  
- \(MRS_{w,d}(f,w,d) = \dfrac{MU_w}{MU_d} = \dfrac{\beta}{\gamma}\cdot \dfrac{d}{w}\)

So with three goods you do exactly what you do with two goods: take the ratio of the relevant marginal utilities; you just now have three different pairwise MRSs (food–water, food–wood, water–wood) instead of a single one.


## Q3: okay, how about for 4 dimensions. Let's say: food, water, gold, silver

In four goods (food, water, gold, silver), the MRS is still defined pairwise as a scalar for any two goods, and it is computed exactly the same way: as a ratio of marginal utilities. [intelligenteconomist](https://www.intelligenteconomist.com/marginal-rate-of-substitution/)

## Geometry with four goods

- Utility is \(u(f,w,g,s)\) where \(f\)=food, \(w\)=water, \(g\)=gold, \(s\)=silver.  
- The indifference set \(u(f,w,g,s)=\bar{u}\) is now a 3‑dimensional **hypersurface** in 4‑dimensional space, not something that can be drawn, but the local trade‑offs are still captured by slopes between pairs of goods. [books.core-econ](https://books.core-econ.org/the-economy-v1/book/text/leibniz-03-02-01.html)

## Marginal utilities in 4D

Compute the partial derivatives:

- \(MU_f = \dfrac{\partial u(f,w,g,s)}{\partial f}\)  
- \(MU_w = \dfrac{\partial u(f,w,g,s)}{\partial w}\)  
- \(MU_g = \dfrac{\partial u(f,w,g,s)}{\partial g}\)  
- \(MU_s = \dfrac{\partial u(f,w,g,s)}{\partial s}\)  

These form the gradient \(\nabla u = (MU_f,MU_w,MU_g,MU_s)\), which encodes all local trade‑offs. [en.wikipedia](https://en.wikipedia.org/wiki/Marginal_rate_of_substitution)

## Pairwise MRS formulas

Pick any two goods; MRS is “how much of the second you are willing to give up for one more unit of the first, holding utility constant” and is given by the ratio of the corresponding marginal utilities:

- **Food for water**:  
  \[
  MRS_{f,w} = \frac{MU_f}{MU_w}.
  \]
- **Food for gold**:  
  \[
  MRS_{f,g} = \frac{MU_f}{MU_g}.
  \]
- **Food for silver**:  
  \[
  MRS_{f,s} = \frac{MU_f}{MU_s}.
  \]
- **Water for gold**:  
  \[
  MRS_{w,g} = \frac{MU_w}{MU_g}.
  \]
- **Water for silver**:  
  \[
  MRS_{w,s} = \frac{MU_w}{MU_s}.
  \]
- **Gold for silver**:  
  \[
  MRS_{g,s} = \frac{MU_g}{MU_s}.
  \]

Each is a scalar evaluated at a particular bundle \((f,w,g,s)\). [intelligenteconomist](https://www.intelligenteconomist.com/marginal-rate-of-substitution/)

## Example: 4‑good Cobb–Douglas

Take
\[
u(f,w,g,s) = f^{\alpha} w^{\beta} g^{\gamma} s^{\delta}, \quad \alpha,\beta,\gamma,\delta>0.
\]

Then:

- \(MU_f = \alpha f^{\alpha-1} w^{\beta} g^{\gamma} s^{\delta}\)  
- \(MU_w = \beta f^{\alpha} w^{\beta-1} g^{\gamma} s^{\delta}\)  
- \(MU_g = \gamma f^{\alpha} w^{\beta} g^{\gamma-1} s^{\delta}\)  
- \(MU_s = \delta f^{\alpha} w^{\beta} g^{\gamma} s^{\delta-1}\)

So, for instance:

- \(MRS_{f,w} = \dfrac{MU_f}{MU_w} = \dfrac{\alpha}{\beta}\cdot \dfrac{w}{f}\)  
- \(MRS_{f,g} = \dfrac{MU_f}{MU_g} = \dfrac{\alpha}{\gamma}\cdot \dfrac{g}{f}\)  
- \(MRS_{g,s} = \dfrac{MU_g}{MU_s} = \dfrac{\gamma}{\delta}\cdot \dfrac{s}{g}\)

The pattern is identical in any number of dimensions: define \(u\), take partial derivatives to get marginal utilities, then form **ratios** for whatever pairwise MRS you care about. [study](https://study.com/academy/lesson/marginal-rate-of-substitution-definition-formula-example.html)



## Q4: I guess what I mean is that in a bartering economy there are `n` goods, and `n(n-1)/2` trading pairs, but if you have money there are only `n` trading pairs (each good is only tradable for money). Am I missing something?


You have the key idea exactly right: moving from barter to money collapses the *network* of trading relationships from \(n(n-1)/2\) direct good–good links down to \(n\) good–money links. [investopedia](https://www.investopedia.com/terms/m/mediumofexchange.asp)

## Barter: dense network of pairs

- With \(n\) goods and only barter, each distinct pair of goods might have its own exchange ratio, so in principle you face up to \(\dfrac{n(n-1)}{2}\) potential trading relationships.  
- On top of that, trade requires a double coincidence of wants: for good A to be exchanged for good B, the holder of A must want B and the holder of B must want A at the same time. [corporatefinanceinstitute](https://corporatefinanceinstitute.com/resources/economics/bartering/)

## Money: a hub that replaces pairwise links

- When money is used, each good just needs a **price in money**, so you effectively have \(n\) prices (good–money pairs) instead of separate prices for every possible good–good pair. [study](https://study.com/academy/lesson/money-as-a-medium-of-exchange-definition-function-examples.html)
- Money removes the double coincidence requirement: you sell what you have for money (even if the buyer doesn’t want what you will later buy), then use that money later to buy what you actually want from someone else. [vaia](https://www.vaia.com/en-us/textbooks/economics/economics-1-edition/chapter-2/problem-70-what-is-the-advantage-of-using-money-in-trade-as-/)

## Where MRS sits in this picture

- MRS describes a person’s *internal* trade‑off between two goods (their “subjective exchange rate”). That object exists whether the economy is barter or monetary.  
- In a monetary economy, equilibrium prices in money summarize the relevant trade‑offs, so instead of thinking about all good–good MRS pairs, people compare their MRS for a given good with its **price relative to money (or to a composite ‘all other goods’)**.

So you are not missing anything crucial: one way to see the benefit of money is exactly that it reduces the combinatorial explosion of trading pairs by introducing a single, universally accepted intermediate good.


## Q5: So suppose I'm modeling an economy with N commodities and 1 currency. Then there would be N trading pairs against currency. Wouldn't that give me an N-dimensional vector of MRS values (against the currency)?


Yes, exactly. This is a profound and fundamental insight in economic theory, and it is exactly how economists bridge the gap between high-dimensional preference spaces and real-world prices.

When you designate one of the goods as a currency or "numéraire" good (let's call it $m$ for money), and you have $N$ other commodities, your utility function is $U(x_1, \dots, x_N, m)$. 

By doing this, you instantly collapse the complex web of $\frac{N(N-1)}{2}$ direct good-to-good trade-offs into a clean **$N$-dimensional vector of MRS values against money**:

$$ \left( MRS_{1,m}, MRS_{2,m}, \dots, MRS_{N,m} \right) = \left( \frac{MU_1}{MU_m}, \frac{MU_2}{MU_m}, \dots, \frac{MU_N}{MU_m} \right) $$

## Why this is so powerful

**1. It perfectly matches the price vector**
In a market economy, consumers don't have to think about the exchange rate between apples and haircuts. They only have to look at the price of apples in money ($P_1$) and the price of haircuts in money ($P_2$). 

At the optimal choice (consumer equilibrium), a person will buy goods until their internal, subjective MRS for each good against money equals the market price of that good:
$$ MRS_{i,m} = P_i $$
*(Assuming the price of money $P_m$ is exactly 1).* [sciencedirect](https://www.sciencedirect.com/science/article/pii/S0047272796016106)

So, that $N$-dimensional vector of MRS values against currency is exactly the **vector of prices** the consumer is willing to pay. 

**2. It mathematically routes all other pairs**
Even though you only explicitly compute the $N$ pairs against money, you haven't lost the $\frac{N(N-1)}{2}$ other combinations. They are implicitly locked in by the math. If you ever *did* want to know the MRS between good 1 and good 2, you just divide their MRS against money:
$$ MRS_{1,2} = \frac{MRS_{1,m}}{MRS_{2,m}} = \frac{P_1}{P_2} $$
 [princeton](https://www.princeton.edu/~mwatson/papers/npi_may_2007.pdf)

## The Numéraire 
In microeconomics, the good against which all other MRS values are evaluated is called the **numéraire** good.  While it is often literal money (currency), economic models sometimes just pick one random commodity (like "gold" or "leisure" or "a composite of all other goods") and set its price to 1. [sciencedirect](https://www.sciencedirect.com/science/article/pii/S0047272796016106)

By using a numéraire, you formally reduce the complexity of the economy from an explosion of pairwise combinations down to a simple $N$-dimensional price system, which is exactly why monetary economies are vastly more efficient for information processing than barter economies.
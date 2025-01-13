@def title = "elfofrit.com | The critical hits problem of Fire Emblem"

[**_VERSIÓN EN ESPAÑOL AQUÍ_**](/ElProblemaDeLosGolpesCriticosDeFireEmblem/)

# The critical hits problem of Fire Emblem

## Introduction
I came across this image on Discord the other day:

~~~
<img src="/assets/FireEmblemCrit.jpg" />
~~~

Some commented that the probability of both hits being critical is 25%. Another shared a screenshot from a forum arguing that the probability is 33%. Intrigued, I decided to crunch the numbers.

## Demonstration

Using Bayes' theorem:

\begin{equation}
    P(A | B) = \dfrac{P(B | A) \cdot P(A)}{P(B)}
\end{equation}

Where:

$P(A|B)$ is the probability of event $A$ occurring given that event $B$ has occurred.

$P(B|A)$ is the probability of event $B$ occurring given that event $A$ has occurred.

$P(A)$ is the probability of event $A$ occurring.

$P(B)$ is the probability of event $B$ occurring.

Let $A$ be the event where both hits are critical.

Let $B$ be the event where at least one hit is critical.

Since the critical hit probability is always 50% (assuming each hit is an independent event that neither influences nor is influenced by the outcomes of other hits), the probability of both hits being critical is $0.50 \times 0.50 = 0.25$, meaning:

\begin{equation}
    P(A) = 0.25 \ .
\end{equation}

If both hits are critical, then at least one of them is critical. Basic logic, hence:

\begin{equation}
    P(B|A) = 1 \ .
\end{equation}

The probability that at least one of the hits is critical is more interesting to calculate.

Let ⭕ represent a critical hit and ❌ a non-critical hit. The possible outcomes for two hits are:

1. ❌❌

2. ❌⭕

3. ⭕❌

4. ⭕⭕

This assumes a 50% chance of landing a critical hit and a 50% chance of missing it.

Here lies the crux of the problem: What is the value of $P(B)$?

If we consider all possible outcomes, in three of them there is at least one ⭕. This means that in 3 out of 4 cases, at least one critical hit occurs, therefore

\begin{equation}
    P(B) = 3/4 = 0.75 \ .
\end{equation}

However, Robin states that at least one of the hits is critical, so the first scenario is excluded from all possible cases, leaving:

1. ❌⭕

2. ⭕❌

3. ⭕⭕

Now, in all these cases, there is at least one ⭕. Does this mean $P(B) = 1$?

Bayes' theorem only applies to **mutually exclusive events**, meaning **each hit is an independent event that neither influences nor is influenced by the outcomes of the other hits**.

We must not fall into Robin's trap and set $P(B) = 1$. We must stand firm and declare that $P(B) = 0.75$.

Substituting $(2)$, $(3)$, and $(4)$ into $(1)$:

\begin{equation}
    P(A | B) = \dfrac{1 \cdot 0.25}{0.75} = 1/3
\end{equation}

Therefore, the probability that both hits are critical is 33%. You're welcome, Robin.
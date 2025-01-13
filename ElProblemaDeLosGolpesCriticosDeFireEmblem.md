@def title = "elfofrit.com | El problema de los golpes críticos de Fire Emblem"

[**_ENGLISH VERSION HERE_**](/TheCriticalHitsProblemOfFireEmblem/)

# El problema de los golpes críticos de Fire Emblem

## Introducción
Me topé con esta imagen en Discord el otro día:

~~~
<img src="/assets/FireEmblemCrit.jpg" />
~~~

Algunos comentaron que la probabilidad de que ambos golpes sean críticos es del 25%. Otro compartió captura de pantalla de un foro donde argumentaban que la probabilidad es del 33%. Intrigado, decidí echar cuentas.

## Demostración

Usando el teorema de Bayes:

\begin{equation}
    P(A | B) = \dfrac{P(B | A) \cdot P(A)}{P(B)}
\end{equation}

Donde:

$P(A|B)$ es la probabilidad de ocurra el evento $A$ si ocurre el evento $B$.

$P(B|A)$ es la probabilidad de ocurra el evento $B$ si ocurre el evento $A$.

$P(A)$ es la probabilidad de ocurra el evento $A$.

$P(B)$ es la probabilidad de ocurra el evento $B$.

Sea $A$ el evento donde ambos golpes son críticos

Sea $B$ el evento donde al menos un golpe es crítico.

Como la probabilidad de crítico es siempre 50% (se supone que cada golpe es un evento aislado que no afecta ni es afectado por los resultados de los otros golpes), la probabilidad de que ambos golpes sean críticos es $0.50 \times 0.50 = 0.25$, es decir

\begin{equation}
    P(A) = 0.25 \ .
\end{equation}

Si ambos golpes son críticos, entonces al menos uno de ellos es crítico. Lógica básica, por ende

\begin{equation}
    P(B|A) = 1 \ .
\end{equation}

La probabilidad de que al menos uno de los golpes sea crítico es más interesante de calcular.

Sea ⭕ un golpe crítico y sea ❌ un golpe no crítico. Los resultados posibles para dos golpes son:

1. ❌❌

2. ❌⭕

3. ⭕❌

4. ⭕⭕

Esto suponiendo que es 50% probable asestar un crítico, así como es 50% probable no lograrlo.

Y es aquí donde yace la disputa del problema: ¿Cuál es el valor de $P(B)$?

Si vemos todos los resultados posibles, en tres de ellos hay mínimo un ⭕, lo que significa que en 3 de 4 casos ocurre al menos un golpe crítico, es decir

\begin{equation}
    P(B) = 3/4 = 0.75 \ .
\end{equation}

Pero la mona china dice que al menos uno de los golpes es crítico, por lo que el primer escenario se descarta de todos los casos posibles, siendo entonces:

1. ❌⭕

2. ⭕❌

3. ⭕⭕

Ahora en todos los casos hay mínimo un ⭕, ¿entonces $P(B) = 1$?

El teorema de Bayes solo aplica para **eventos mutuamente excluyentes**, es decir, **cada golpe es un evento aislado que no afecta ni es afectado por los resultados de los otros golpes**.

No debemos caer en la trampa de la mona china y establecer que $P(B) = 1$. Hay que permanecer firmes y declarar que $P(B) = 0.75$.

Sustituyendo $(2)$, $(3)$ y $(4)$ en $(1)$:

\begin{equation}
    P(A | B) = \dfrac{1 \cdot 0.25}{0.75} = 1/3
\end{equation}

Por lo tanto, la probabilidad de que ambos golpes sean críticos es del 33%. De nada, Robin.
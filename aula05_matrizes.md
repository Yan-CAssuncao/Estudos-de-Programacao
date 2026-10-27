# AULA 5 — Matrizes para Matemática Computacional e Modelagem

### Metas da Aula

1. Representar tabelas científicas, campos discretos e operadores lineares usando listas de listas.
2. Implementar soma, transposta, produtos e cálculo de resíduos, entendendo o papel dos índices.
3. Construir a matriz de diferenças finitas para um problema de contorno unidimensional.
4. Montar a matriz de rigidez de uma barra a partir das contribuições dos elementos e da conectividade.
5. Relacionar cálculos matriciais a hipóteses, unidades, condições de contorno e verificações do modelo.

### Ao término desta aula, você será capaz de:

1. Criar, preencher e percorrer matrizes retangulares com linhas independentes.
2. Traduzir um somatório matemático em laços aninhados.
3. Distinguir multiplicar uma matriz por um vetor de resolver um sistema linear.
4. Verificar simetria, equilíbrio e resíduos de exemplos discretos.
5. Realizar pequenos experimentos de modelagem, variando parâmetros e interpretando os resultados.

---

## 5.1 Da lista à matriz: o significado vem do modelo

Na Aula 4, uma lista guardava os nós de uma barra, os valores de uma derivada ou a trajetória de Euler. Agora precisamos organizar duas direções de informação. Uma matriz pode armazenar temperaturas em uma grade espacial, mas também pode guardar os coeficientes de um sistema de equações. A estrutura em Python pode ser a mesma; **o significado de cada linha e coluna é diferente**.

| Objeto | Significado da linha | Significado da coluna | Entrada |
|---|---|---|---|
| Tabela de experimentos | Experimento | Grandeza medida | Medida correspondente |
| Campo em grade 2D | Posição vertical | Posição horizontal | Valor do campo naquele ponto |
| Matriz de um sistema linear | Equação discreta | Incógnita | Coeficiente da incógnita na equação |
| Conectividade do MEF | Elemento | Nó local | Índice do nó global |

Antes de programar, escreva o significado desses índices. Isso evita, por exemplo, confundir uma temperatura com um coeficiente que multiplica uma temperatura.

### Exercício de Exemplo — temperaturas em uma grade

Considere medições em três alturas e quatro posições horizontais, em graus Celsius:

```python
T = [[20.0, 21.0, 22.0, 23.0],
     [24.0, 25.0, 26.0, 27.0],
     [28.0, 29.0, 30.0, 31.0]]

print(T[1][2])       # 26.0: segunda altura, terceira posição horizontal
print(len(T))       # 3 linhas
print(len(T[0]))     # 4 colunas
```

Uma matriz é representada aqui por uma **lista de listas**: `T[i]` seleciona uma linha e `T[i][j]` seleciona uma entrada dessa linha. Ambos os índices começam em zero. Em C, uma declaração como `double T[3][4]` define um arranjo retangular; a lista de listas de Python permite linhas de comprimentos diferentes. Para as operações desta aula, adotaremos matrizes retangulares não vazias, com entradas numéricas.

**Convenção:** em uma matriz de formato `m × n`, `m` é o número de linhas e `n` o número de colunas. No campo espacial, usaremos `T[i][j]` para o ponto de coordenadas `(x[j], y[i])`.

## 5.2 Criando matrizes e evitando linhas compartilhadas

Para montar um operador ou um campo, geralmente começamos com zeros e depois preenchemos as entradas necessárias:

```python
m = 3
n = 4
T = [[0.0 for j in range(n)] for i in range(m)]
T[1][2] = 26.0
print(T)
```

A compreensão externa repete a criação das linhas; a interna cria as entradas de cada linha. Também é possível usar `[[0.0] * n for i in range(m)]`.

A forma abaixo é inadequada quando pretendemos alterar entradas de modo independente:

```python
T = [[0.0] * 4] * 3
T[1][2] = 26.0
print(T)  # a coluna 2 muda nas três linhas!
```

O operador externo `* 3` repete referências à **mesma** lista. As três linhas não são independentes. Esse detalhe de referências será aprofundado na Aula 11; por enquanto, crie uma lista nova a cada linha.

Outra situação importante é guardar um estado antigo antes de atualizá-lo. `T_nova = T` não cria outra matriz; para copiar as linhas de uma matriz de números, use:

```python
T_nova = [linha[:] for linha in T]
```

O fatiamento de cada linha, visto na Aula 4, copia seus valores. Apenas `T[:]` copia a lista externa e ainda compartilha as linhas internas. Isso será decisivo em esquemas temporais: todos os valores novos devem ser calculados a partir do estado antigo quando essa for a regra do método.

## 5.3 Preenchendo, percorrendo e interpretando os dados

Os laços aninhados da Aula 3 agora passam por linhas e colunas. Use índices quando precisar localizar ou alterar uma entrada; use diretamente os valores quando só precisar percorrê-los.

### Exercício de Exemplo — leitura de uma tabela experimental

*Leia a temperatura em três posições de uma barra, em dois instantes. Cada linha representa um instante e cada coluna uma posição. Imprima a tabela e a média espacial das três medições em cada instante.*

```python
n_instantes = 2
n_posicoes = 3
T = [[0.0] * n_posicoes for i in range(n_instantes)]

for i in range(n_instantes):
    for j in range(n_posicoes):
        T[i][j] = float(input(f"Temperatura no instante {i}, posição {j}: "))

for i, linha in enumerate(T):
    for valor in linha:
        print(f"{valor:8.2f}", end=" ")
    print(f" | Média das medições: {sum(linha) / n_posicoes:.2f}")
```

O formato `8.2f` reserva oito caracteres, com duas casas decimais, alinhando a tabela. A média calculada é a média aritmética das medições. Ela não é automaticamente a temperatura média contínua da barra: para aproximar uma integral espacial, precisamos considerar posições e pesos de quadratura.

### Exercício de Exemplo — gerar um campo conhecido

*Avalie o campo adimensional $u(x,y)=x(1-x)y(1-y)$ em uma grade do quadrado unitário.*

```python
nx = 4                     # número de subintervalos em x
ny = 2                     # número de subintervalos em y
x = [j / nx for j in range(nx + 1)]
y = [i / ny for i in range(ny + 1)]
U = [[0.0] * (nx + 1) for i in range(ny + 1)]

for i in range(ny + 1):
    for j in range(nx + 1):
        U[i][j] = x[j] * (1 - x[j]) * y[i] * (1 - y[i])

for linha in U:
    print(linha)
```

Há `nx + 1` pontos em cada linha e `ny + 1` linhas: **número de pontos e número de subintervalos não coincidem**. Os valores da borda são zero porque a expressão escolhida se anula quando `x` ou `y` vale 0 ou 1. Aqui avaliamos uma função conhecida; ainda não resolvemos uma EDP.

## 5.4 Operações matriciais: transformar fórmulas em laços

As listas não executam álgebra linear automaticamente. `A + B` concatena listas; `A * B`, quando ambas são listas, gera um erro de tipo. Implementaremos as operações entrada a entrada para compreender os algoritmos. As dimensões dos exemplos são compatíveis por construção.

### Soma e multiplicação por escalar

Se dois mecanismos contribuem para o mesmo operador, podemos precisar combinar suas matrizes. Para $C=\alpha A+B$, com ambas de formato `m × n`, temos $C_{ij}=\alpha A_{ij}+B_{ij}$:

```python
A = [[2.0, -1.0], [-1.0, 2.0]]
B = [[1.0, 0.0], [0.0, 1.0]]
alpha = 0.5
m = len(A)
n = len(A[0])
C = [[0.0] * n for i in range(m)]

for i in range(m):
    for j in range(n):
        C[i][j] = alpha * A[i][j] + B[i][j]

print(C)  # [[2.0, -0.5], [-0.5, 2.0]]
```

No modelo, a soma também precisa ser dimensionalmente consistente: não basta que as matrizes tenham o mesmo formato; os coeficientes físicos devem tornar suas contribuições compatíveis em unidades.

### Transposta

A transposta troca linhas por colunas: $(A^T)_{ji}=A_{ij}$. Uma matriz `m × n` produz uma transposta `n × m`:

```python
A = [[1, 2, 3], [4, 5, 6]]
m = len(A)
n = len(A[0])
At = [[0] * m for j in range(n)]

for i in range(m):
    for j in range(n):
        At[j][i] = A[i][j]

print(At)  # [[1, 4], [2, 5], [3, 6]]
```

Uma matriz quadrada é simétrica quando $A_{ij}=A_{ji}$ para todos os índices. Matrizes de rigidez do modelo de barra desta aula têm essa propriedade. Para comparações entre resultados de ponto flutuante, usaremos uma tolerância, por exemplo `abs(A[i][j] - A[j][i]) <= 1e-10`.

### Produto matriz-vetor

Se $A$ tem formato `m × n`, $x$ precisa de `n` componentes; $y=Ax$ terá `m` componentes:

$$y_i=\sum_{j=0}^{n-1}A_{ij}x_j.$$

```python
A = [[2.0, -1.0], [-1.0, 2.0]]
x = [1.0, 3.0]
m = len(A)
n = len(A[0])
y = [0.0] * m

for i in range(m):
    for j in range(n):
        y[i] += A[i][j] * x[j]

print(y)  # [-1.0, 5.0]
```

O índice `i` escolhe qual componente estamos calculando; `j` percorre os termos do somatório. A acumulação começa em zero antes do laço que soma. Multiplicar `A` por um `x` conhecido não é resolver `Ax=b`: resolver significa determinar o `x` desconhecido a partir de `A` e `b`.

### Produto de matrizes

Se $A$ tem formato `m × p` e $B$ tem formato `p × n`, então $C=AB$ tem formato `m × n` e

$$C_{ij}=\sum_{k=0}^{p-1}A_{ik}B_{kj}.$$

```python
A = [[1, 2, 0], [0, 1, 1]]
B = [[1, 0], [0, 1], [1, 1]]
m = len(A)
p = len(A[0])
n = len(B[0])
C = [[0.0] * n for i in range(m)]

for i in range(m):
    for j in range(n):
        for k in range(p):
            C[i][j] += A[i][k] * B[k][j]

print(C)  # [[1.0, 2.0], [1.0, 2.0]]
```

Os dois primeiros laços escolhem uma entrada do resultado; o terceiro executa seu somatório. O custo desse algoritmo é da ordem de `m * n * p`; no caso quadrado, da ordem de $n^3$. Um produto pode representar a composição de duas transformações, e a ordem importa: em geral, $AB\ne BA$.

## 5.5 Resíduo: conferir uma solução proposta

Para um sistema $Ax=b$, o **resíduo** de um vetor de teste $x$ é

$$r=b-Ax,\qquad \|r\|_\infty=\max_i|r_i|.$$

Uma componente de `r` mede quanto a equação correspondente deixa de ser satisfeita. O código abaixo testa uma solução conhecida de um sistema pequeno e adimensional:

```python
A = [[2.0, -1.0], [-1.0, 2.0]]
x = [1.0, 3.0]
b = [-1.0, 5.0]
r = [0.0] * len(b)

for i in range(len(A)):
    Ax_i = 0.0
    for j in range(len(x)):
        Ax_i += A[i][j] * x[j]
    r[i] = b[i] - Ax_i

norma_residuo = max([abs(valor) for valor in r])
print(r)                # [0.0, 0.0]
print(norma_residuo)     # 0.0
```

Experimente trocar `x[0]` por `1.01`. Um resíduo pequeno indica que as equações discretas são aproximadamente satisfeitas. Ele **não garante sozinho** erro pequeno na solução: isso depende também do condicionamento do sistema. Além disso, mesmo um sistema resolvido com precisão pode representar mal a situação física. Verificar o algoritmo e validar o modelo são tarefas distintas.

## 5.6 Diferenças finitas: de uma equação à matriz tridiagonal

Considere o problema adimensional

$$-u''(x)=f(x),\quad 0<x<1,\qquad u(0)=a,\quad u(1)=b.$$

Divida o intervalo em `n + 1` subintervalos de tamanho $h=1/(n+1)$. Há `n` incógnitas internas, nos pontos $x_i=ih$, $i=1,\ldots,n$. Pela diferença central,

$$\frac{-u_{i-1}+2u_i-u_{i+1}}{h^2}=f(x_i).$$

A linha do sistema associa o ponto `i` a ele mesmo e aos dois vizinhos. Para três pontos internos:

$$A=\frac{1}{h^2}\begin{bmatrix}2&-1&0\\-1&2&-1\\0&-1&2\end{bmatrix},\qquad
F=\begin{bmatrix}f(x_1)+a/h^2\\f(x_2)\\f(x_3)+b/h^2\end{bmatrix}.$$

Os valores de fronteira não são incógnitas desse sistema; suas contribuições passam para o lado direito. Na primeira equação, por exemplo, $-a/h^2$ passa para a direita com sinal positivo.

**Tradução dos índices:** o ponto matemático de índice `i` corresponde à entrada Python de índice `i - 1`. No código abaixo, `j = 0` representa o primeiro ponto interno, $x_1=h$.

### Exercício de Exemplo — montar e verificar sem resolver

*Use $u(x)=x(1-x)+1+x$, que satisfaz $-u''=2$, $u(0)=1$ e $u(1)=2$. Monte o sistema e confira o resíduo do vetor com os valores exatos nos pontos internos.*

```python
n = 3
h = 1.0 / (n + 1)
a = 1.0
b = 2.0
A = [[0.0] * n for i in range(n)]
F = [2.0] * n
x = [(j + 1) * h for j in range(n)]

for i in range(n):
    A[i][i] = 2.0 / h**2
    if i > 0:
        A[i][i - 1] = -1.0 / h**2
    if i < n - 1:
        A[i][i + 1] = -1.0 / h**2

F[0] += a / h**2
F[-1] += b / h**2
u = [ponto * (1 - ponto) + 1 + ponto for ponto in x]
r = [0.0] * n

for i in range(n):
    Au_i = 0.0
    for j in range(n):
        Au_i += A[i][j] * u[j]
    r[i] = F[i] - Au_i

print(F)  # [18.0, 2.0, 34.0]
print("Resíduo máximo:", max([abs(valor) for valor in r]))  # 0.0
```

Para esse polinômio, a segunda diferença reproduz a segunda derivada exatamente, salvo arredondamento. Esse resultado é uma verificação da montagem, e não uma demonstração de exatidão para funções gerais. Para `n = 1`, as duas contribuições de fronteira entram na mesma posição de `F`; usar `+=` mantém ambas.

Os `if` também têm significado matemático: `A[0][-1]` é um acesso válido em Python, mas colocaria uma conexão com a última incógnita, indevida neste problema com condições de Dirichlet.

### Visualizando o padrão de entradas não nulas

Acrescente ao exemplo anterior:

```python
import matplotlib.pyplot as plt

plt.spy(A)
plt.title("Padrão não nulo da matriz de diferenças finitas")
plt.xlabel("Coluna: incógnita")
plt.ylabel("Linha: equação")
plt.show()
```

Com `n = 30`, aparecem três diagonais. Embora a matriz possua $n^2$ posições, apenas $3n-2$ entradas são não nulas para `n >= 1`. Nossa lista de listas ainda armazena todos os zeros; formatos esparsos e o algoritmo de Thomas serão estudados depois.

## 5.7 Elementos finitos: da matriz local à montagem global

Retome a barra com nós `nos = [0, 2, 5, 10]` das Aulas 2–4. Agora cada elemento linear de dois nós contribuirá para a matriz global. O modelo é de pequenas deformações axiais, sem flexão, com $E$ e área transversal constantes dentro de cada elemento:

$$k^{(e)}=\frac{E_e A_e}{L_e}\begin{bmatrix}1&-1\\-1&1\end{bmatrix}.$$

Aqui $E$ é o módulo de Young, $A_e$ é a área da seção e $L_e$ o comprimento. No SI, $EA/L$ tem unidade N/m, de modo que matriz de rigidez vezes deslocamento em metros resulta em força em newtons. Usaremos `EA = 300.0` N, igual nos três elementos.

A **conectividade** identifica quais nós globais correspondem aos dois nós locais de cada elemento:

| Elemento Python | Nós globais | Comprimento (m) | $EA/L$ (N/m) |
|---|---|---|---|
| 0 | 0 e 1 | 2 | 150 |
| 1 | 1 e 2 | 3 | 100 |
| 2 | 2 e 3 | 5 | 60 |

### Exercício de Exemplo — montagem com conectividade

```python
nos = [0.0, 2.0, 5.0, 10.0]
conec = [[0, 1], [1, 2], [2, 3]]
EA = 300.0
n_nos = len(nos)
K = [[0.0] * n_nos for i in range(n_nos)]

for e in range(len(conec)):
    no_esq = conec[e][0]
    no_dir = conec[e][1]
    comprimento = nos[no_dir] - nos[no_esq]
    k = EA / comprimento
    k_local = [[k, -k], [-k, k]]

    for a_local in range(2):
        i_global = conec[e][a_local]
        for b_local in range(2):
            j_global = conec[e][b_local]
            K[i_global][j_global] += k_local[a_local][b_local]

for linha in K:
    print(linha)
```

Saída:

```text
[150.0, -150.0, 0.0, 0.0]
[-150.0, 250.0, -100.0, 0.0]
[0.0, -100.0, 160.0, -60.0]
[0.0, 0.0, -60.0, 60.0]
```

Os índices locais `a_local` e `b_local` variam entre 0 e 1; os globais percorrem os nós da barra. A conectividade traduz um conjunto de índices no outro. A entrada `K[1][1]` recebe `150 + 100` porque o nó global 1 pertence a dois elementos. **É necessário acumular com `+=`**, preservando contribuições já montadas.

Neste caso, a numeração sequencial dos nós produz uma matriz tridiagonal. Em outras malhas ou numerações, o padrão pode mudar. A simetria permanece para o modelo elástico considerado, pois cada matriz local é simétrica e as contribuições são montadas consistentemente.

### O que verificar antes de resolver?

Acrescente ao código anterior:

```python
tol = 1e-10
simetrica = True
somas_nulas = True

for i in range(n_nos):
    if abs(sum(K[i])) > tol:
        somas_nulas = False
    for j in range(n_nos):
        if abs(K[i][j] - K[j][i]) > tol:
            simetrica = False

print("Simetria:", simetrica)
print("Somas das linhas nulas:", somas_nulas)
```

A soma das linhas igual a zero significa que $Kc=0$ para um deslocamento constante $c$: a translação rígida não deforma a barra. Por isso a matriz sem apoios é singular. Essa propriedade vale para a barra livre deste exemplo; não deve ser atribuída a toda matriz de rigidez de qualquer modelo.

### Apoio, forças e reação

Se fixamos `u[0] = 0`, podemos eliminar a primeira linha e coluna para trabalhar com os deslocamentos livres:

```python
K_red = [linha[1:] for linha in K[1:]]
```

A expressão fatia cada linha; apenas `K[1:][1:]` não faria esse trabalho. Para deslocamento prescrito zero, o lado direito reduzido contém as forças aplicadas nos nós livres. Para um valor prescrito não nulo $u_0=g$, ele precisa de correção:

$$K_{ff}u_f=F_f-K_{f0}g.$$

A equação do nó fixo será usada para recuperar a reação: $R_0=(Ku)_0-F_0$, sendo $F_0$ a força aplicada conhecida nesse nó. Na aula anterior, a reação era lida para conferir equilíbrio; em um problema resolvido, ela pode ser calculada. Para esta barra conectada, com rigidezes positivas, fixar um nó remove o modo de translação e torna a matriz reduzida positiva definida. A resolução será retomada na Aula 15.

## 5.8 Modelagem discreta: uma matriz também pode descrever transições

Para ampliar o contexto além das equações diferenciais, considere dois estados de um equipamento: funcionando e em manutenção. Por dia, admita que 90% dos equipamentos funcionando continuem funcionando e 10% entrem em manutenção. Dos que estão em manutenção, 40% retornam ao funcionamento e 60% permanecem em manutenção. As probabilidades são constantes e dependem apenas do estado atual.

$$P=\begin{bmatrix}0.9&0.1\\0.4&0.6\end{bmatrix}.$$

A linha indica o estado de origem, a coluna o destino. Cada linha soma 1. Com distribuição representada por **vetor-linha**, a evolução é $\pi_{k+1}=\pi_kP$, e não $P\pi_k$:

$$\pi_{k+1,j}=\sum_i\pi_{k,i}P_{ij}.$$

### Exercício de Exemplo — propagar a distribuição

```python
P = [[0.9, 0.1], [0.4, 0.6]]
pi = [1.0, 0.0]              # inicialmente, todos funcionando
historico = [pi[:]]

for passo in range(10):
    pi_nova = [0.0, 0.0]
    for j in range(2):
        for i in range(2):
            pi_nova[j] += pi[i] * P[i][j]
    pi = pi_nova
    historico.append(pi[:])

print("Distribuição após 10 dias:", pi)
print("Soma das probabilidades:", sum(pi))
```

A atualização usa uma nova lista para não misturar componentes antigas e novas. A distribuição se aproxima de `[0.8, 0.2]`. Isso descreve probabilidades ou frações esperadas de uma população, não a trajetória determinística de um equipamento individual. Alterar as taxas e comparar os resultados é um pequeno experimento de sensibilidade do modelo.

## 5.9 Resumo da Aula

- Matrizes são representadas por listas de listas; declare o significado dos índices e as dimensões antes do cálculo.
- Crie linhas independentes e copie cada linha ao preservar um campo numérico anterior.
- Soma e transposta usam dois laços; o produto matricial usa um terceiro índice para o somatório.
- Produtos e resíduos permitem verificar equações discretas, sem necessariamente resolver o sistema.
- Em DF, cada linha relaciona um ponto aos seus vizinhos; condições de contorno entram na montagem do lado direito.
- Em MEF, a conectividade relaciona índices locais e globais, e `+=` acumula contribuições dos elementos.
- Simetria, movimentos rígidos, unidades e equilíbrio ajudam a detectar erros.
- Matrizes de transição modelam evoluções probabilísticas; a orientação do vetor define a ordem do produto.

Na Aula 6, tuplas, dicionários e outras estruturas ajudarão a organizar os dados de nós, elementos e parâmetros. Funções ficam para a Aula 8. Nesta aula, todos os algoritmos são escritos com os recursos já apresentados: variáveis, decisões, laços e listas, além do `matplotlib` da Aula 4.

---

## 5.10 Exercícios da Aula

Todos os exercícios têm contexto matemático ou de modelagem. Tipos: **[MC]** matemática computacional, **[M]** modelagem matemática, **[DF]** diferenças finitas, **[MEF]** elementos finitos. Use listas e laços, sem definir funções (`def`) e sem NumPy. Os enunciados fornecem as fórmulas novas para que você possa concentrar-se em traduzi-las para o código.

Em cada exercício: identifique o significado dos índices, apresente pelo menos uma verificação numérica e interprete o resultado. Valide dimensões positivas e parâmetros físicos positivos quando forem lidos do usuário. Tolerâncias devem ser compatíveis com a escala dos valores; `1e-10` é suficiente para os pequenos testes adimensionais indicados aqui.

### Nível Fácil

1. **[M] Temperatura medida em uma barra.** Use `T = [[20, 22, 24], [21, 24, 27], [23, 26, 29]]`, com linhas correspondendo aos tempos 0, 1 e 2 s, e colunas às posições 0, 0.5 e 1 m. Imprima a temperatura do ponto central em cada instante e a variação entre o primeiro e o último instante em cada posição. A barra está aquecendo igualmente em todos os pontos?

2. **[MC] Transformação linear de coordenadas.** A transformação $y=Ax$, com $A=[[0,-1],[1,0]]$, representa uma rotação de 90 graus no plano. Calcule sua ação sobre os vetores `[1, 0]`, `[0, 1]` e `[2, 3]`. Compare a norma ao quadrado $x_0^2+x_1^2$ antes e depois. O que é preservado?

3. **[DF] Campo em uma grade retangular.** Gere os pontos do retângulo $[0,2]\times[0,1]$ com 4 subintervalos em `x` e 2 em `y`. Construa a matriz dos valores de $u(x,y)=x+2y$. Imprima o formato da matriz e a entrada correspondente a $(x,y)=(1,0.5)$. Verifique esse valor manualmente.

4. **[MEF] Rigidez e movimento rígido.** Leia $E>0$, área $A_e>0$ e $L>0$ de uma barra no SI. Construa sua matriz local. Calcule as forças para deslocamentos `[0, 0.001]` m e `[0.001, 0.001]` m. Explique por que o segundo caso não gera forças elásticas.

5. **[MC] Operador com reação.** Construa a matriz $A=T+\mu I$, com $T=[[2,-1],[-1,2]]$, identidade $I$ e $\mu=0,1,5$. Para cada valor, calcule $Ax$ com `x = [1, 1]`. Interprete como o termo diagonal adicional altera a ação do operador sobre um vetor constante.

6. **[M] Uma transição de estados.** No modelo de equipamento da Seção 5.8, use $\pi_0=(0.5,0.5)$ e calcule $\pi_1$. Verifique componentes não negativas e soma igual a 1 com tolerância. Qual fração esperada estará funcionando no dia seguinte?

### Nível Médio

7. **[MC] Composição de transformações.** Use a rotação do exercício 2 e a escala $S=[[2,0],[0,1]]$. Calcule os produtos `AS` e `SA` com três laços e aplique ambos a `[1, 1]`. A ordem de rotação e alongamento produz o mesmo resultado? Relacione os cálculos às duas operações geométricas.

8. **[DF] Montagem com fronteira não homogênea.** Monte o sistema de $-u''=0$ em $(0,1)$, com $u(0)=10$ e $u(1)=30$, para 5 pontos internos. Use $u(x)=10+20x$ para calcular o resíduo. Repita sem as contribuições de fronteira em `F` e compare. Em que equações aparecem os erros? Não é necessário resolver o sistema.

9. **[MEF] Montagem em uma malha não uniforme.** Retome `nos = [0, 2, 5, 10]`, com `EA = 300`. Monte `K` por conectividade, confirme a entrada `K[1][1]` e verifique simetria e somas das linhas. Calcule `K u` para `u = [0.01, 0.01, 0.01, 0.01]`. Interprete o resultado sem impor apoios.

10. **[MC] Resíduo e tamanho da perturbação.** Considere $A=[[2,-1],[-1,2]]$, $b=[-1,5]$ e os vetores $x=(1+\varepsilon,3)$, com $\varepsilon=10^{-1},10^{-2},10^{-3}$. Calcule a norma máxima do resíduo e o erro em relação a `(1, 3)`. Faça uma tabela das duas quantidades. A comparação deste exemplo autoriza concluir que ambas seriam iguais para qualquer matriz? Justifique.

11. **[M] Sensibilidade das frações em manutenção.** Mantenha a chance de falha diária em 0.1 e varie a de reparo $q=0.2,0.4,0.8$, usando $P=[[0.9,0.1],[q,1-q]]$. Parta de `(1, 0)`, calcule 30 dias e plote a fração em manutenção para cada `q`. Confira a conservação da probabilidade em todos os passos e interprete o efeito do reparo.

12. **[DF] Segunda derivada em um campo 2D.** Na grade uniforme $[0,1]^2$ com 4 subintervalos em cada direção, armazene $u(x,y)=x^2+y^2$. Nos pontos internos, calcule

    $$\Delta_h U_{ij}=\frac{U_{i,j-1}-2U_{ij}+U_{i,j+1}}{h^2}+\frac{U_{i-1,j}-2U_{ij}+U_{i+1,j}}{h^2}.$$

    Compare com $\Delta u=4$. Guarde apenas os resultados internos em uma matriz `3 × 3`. Por que esse stencil não pode ser aplicado diretamente nos pontos da borda sem informações adicionais?

### Nível Difícil (elaborados)

13. **[DF] Consistência e ordem observada.** Para $u(x)=\sin(\pi x)$ em $[0,1]$, use `n = 9, 19, 39, 79` pontos internos; assim $h=1/(n+1)$ é dividido exatamente por dois a cada refinamento. Monte `A`, calcule `A u` e compare com $\pi^2\sin(\pi x_i)$. Defina $e_h$ como o erro máximo dessa aplicação do operador e estime $p=\log(e_h/e_{h/2})/\log 2$. Use `import math`, `math.sin`, `math.pi` e `math.log`. Plote com `plt.loglog(hs, erros, "o-")`. O esperado é ordem próxima de 2. Explique por que você mediu o erro do operador discreto aplicado a uma função conhecida, e não o erro de uma solução numérica de um sistema resolvido.

14. **[MEF] Apoio e reação de uma barra tracionada.** Use `nos = [0, 2, 5, 10]`, `EA = 300` N, força de 3 N no último nó, demais forças aplicadas nulas e apoio `u[0] = 0`. A solução nodal esperada é $u_i=3x_i/EA$. (a) Monte `K`. (b) Construa `K_red` e `F_red`. (c) Verifique o sistema reduzido com o vetor esperado. (d) Calcule a reação $R_0=(Ku)_0-F_0$ e confira a soma das forças aplicadas com a reação. (e) Calcule a energia elástica $\frac12\sum_i u_i(Ku)_i$ e compare com $\frac12\,3u_{\mathrm{último}}$. Não conte a reação novamente dentro do vetor de forças aplicadas.

15. **[MN] Jacobi para um sistema discreto.** Monte $A=\operatorname{tridiag}(-1,3,-1)$ de ordem 20, com `b = [1.0] * 20` e aproximação inicial nula. Implemente

    $$x_i^{(k+1)}=\frac{b_i-\sum_{j\ne i}A_{ij}x_j^{(k)}}{A_{ii}}.$$

    Calcule todas as componentes novas a partir de `x` antigo, usando outra lista. Após cada atualização, guarde $\|b-Ax\|_\infty$. Pare quando o resíduo for menor que `1e-8` ou após 1000 atualizações. Informe o motivo da parada e plote o histórico com `plt.semilogy`. A matriz é estritamente diagonalmente dominante por linhas, uma condição suficiente de convergência de Jacobi neste caso. Este exercício introduz a resolução iterativa; não é necessário implementar um método direto.

16. **[MEF] A numeração muda o padrão, não a barra.** Considere a mesma barra com coordenadas guardadas na ordem `nos = [0, 5, 2, 10]` e conectividade `[[0, 2], [2, 1], [1, 3]]`. Os elementos continuam orientados da esquerda para a direita. Monte `K` para `EA = 300` e compare `plt.spy(K)` com o exemplo da Seção 5.7. Confira simetria e ausência de forças para uma translação rígida. A matriz ainda é tridiagonal? Explique por que esses novos índices não representam uma mudança física da barra.

17. **[M] Espera por três dias consecutivos de chuva.** Suponha dias independentes, com probabilidade constante $p$ de chuva. Antes de atingir a sequência desejada, os estados 0, 1 e 2 contam dias consecutivos de chuva no fim do período observado. O estado 3 significa que já ocorreu uma sequência de pelo menos três dias e é absorvente. Use

    $$P=\begin{bmatrix}1-p&p&0&0\\1-p&0&p&0\\1-p&0&0&p\\0&0&0&1\end{bmatrix}.$$

    Partindo de `(1, 0, 0, 0)`, propague sete passos. A quarta componente é a probabilidade de ocorrência da sequência em algum momento da semana. Para $p=0.5$, confira $47/128=0.3671875$. Repita para `p = 0.3, 0.4, 0.5, 0.6, 0.7` e plote a probabilidade final. Discuta o papel das hipóteses de independência e probabilidade constante na interpretação meteorológica.

18. **[DF] Uma atualização explícita do calor em 2D.** Considere $T_t=\kappa\Delta T$ no quadrado unitário, com temperatura zero na borda. Use 4 subintervalos por direção, $\kappa=1$, $h=1/4$, e um estado inicial nulo exceto no ponto central, onde `T[2][2] = 100`. Calcule **uma** atualização nos pontos internos:

    $$T^{novo}_{ij}=T_{ij}+r\big(T_{i,j-1}+T_{i,j+1}+T_{i-1,j}+T_{i+1,j}-4T_{ij}\big),\quad r=\frac{\kappa\Delta t}{h^2}.$$

    Crie outra matriz para o resultado e mantenha a borda zero. Compare `r = 0.2` e `r = 0.3`. Para esse esquema com espaçamentos iguais, a condição usual de estabilidade é $r\le1/4$. Observe a temperatura central e o mínimo da matriz após o passo. Interprete o valor negativo que aparece no segundo caso, lembrando que os dados iniciais e a fronteira são não negativos. O exercício mostra por que escolher o passo temporal faz parte do método, não apenas da programação.

---

## Referências e ligação com os materiais fornecidos

- **Aulas 01–04:** entrada e saída, operadores, decisões, laços, listas, fatiamento, compreensão de lista e gráficos. Foram mantidos a estrutura de metas, exemplos comentados, resumo e exercícios graduados, além da continuidade da barra com nós `[0, 2, 5, 10]`.
- **Roteiro do curso (`pasted.txt`):** matrizes na Aula 5; estruturas de dados na Aula 6; funções na Aula 8; DF e MEF nas Aulas 14 e 15. As aplicações desta aula preparam essas etapas.
- **MENEZES, Nilo Ney Coutinho. Introdução à Programação com Python, 2ª edição, Seção 6.15 — Listas dentro de listas:** acesso por índices e organização de dados aninhados, aqui adaptados a dados científicos.
- **DOWNEY, Allen B. Pense em Python, Capítulo 10, especialmente Seções 10.1–10.4:** listas aninhadas, mutabilidade, travessia e diferença entre operações de listas e operações matemáticas.
- **LANGTANGEN, Hans Petter. A Primer on Scientific Programming with Python, Seção 2.4 — Nested lists:** tabelas como listas de linhas ou colunas e organização dos percursos em computação científica. Os exemplos desta aula usam sintaxe Python 3 e não dependem da sintaxe de versões antigas do livro.
- **MEERSCHAERT, Mark M. Mathematical Modeling, Seção 1.3 — Sensitivity and Robustness e Seção 8.1 — Markov Chains:** explicitação de hipóteses, análise de sensibilidade e evolução matricial de distribuições. Os exemplos foram formulados para o nível desta aula.
- **Texto de referência de estilo enviado:** organização concisa das seções, ligação entre matrizes, DF e montagem por conectividade, e exercícios com interpretação matemática. O conteúdo desta versão foi reescrito e ampliado para o foco solicitado.

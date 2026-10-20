# AULA 4 — Listas (Vetores)

### Metas da Aula
1. Entender e praticar o uso de listas em Python como estrutura homogênea para armazenar vários valores.
2. Aplicar listas em situações variadas, combinando-as com laços e decisões já vistos.
3. Introduzir a visualização de dados com a biblioteca `matplotlib`.

### Ao término desta aula, você será capaz de:
1. Criar, atribuir e acessar valores em uma lista.
2. Percorrer listas com laços, das formas idiomáticas de Python.
3. Usar fatiamento (*slicing*) e compreensão de lista (*list comprehension*).
4. Gerar gráficos simples a partir de listas de dados.

---

## 4.1 Por que listas?

Até aqui, cada variável guardava exatamente um valor. Se fosse preciso armazenar a idade de 2 pessoas, seriam necessárias 2 variáveis; para 10 pessoas, 10 variáveis. Isso deixa de ser viável rapidamente — imagine escrever 100 variáveis para 100 pessoas, ou pior, para os 100 nós de uma malha numérica.

Em C, esse problema é resolvido com **vetores**: um bloco de memória de tamanho fixo, declarado com um tipo e um tamanho definidos antes da execução (`int idades[10];`). Python resolve o mesmo problema com **listas**, que têm uma diferença estrutural importante em relação aos vetores de C: **não têm tamanho fixo, nem tipo fixo**. Uma lista pode crescer, encolher, e (embora não seja uma boa prática misturar tipos sem motivo) pode até guardar valores de tipos diferentes ao mesmo tempo.

## 4.2 Criando listas

Existem várias formas de criar uma lista em Python:

```python
vazia = []                        # lista vazia
idades = [10, 20, 35, 42]          # lista já com valores
zeros = [0] * 5                    # [0, 0, 0, 0, 0]
sequencia = list(range(5))         # [0, 1, 2, 3, 4]
```

Diferente de C, não é preciso declarar quantos elementos a lista vai ter — ela pode ser criada vazia e crescer conforme valores são adicionados, com o método `.append()`:

```python
idades = []
idades.append(10)
idades.append(20)
# idades agora e [10, 20]
```

A função `len()` retorna quantos elementos a lista tem no momento — o equivalente prático ao "tamanho do vetor" que em C precisava ser controlado manualmente:

```python
print(len(idades))   # 2
```

## 4.3 Atribuição e acesso a valores

O acesso a uma posição segue o mesmo princípio de C: índices começam em **0**, e o último elemento de uma lista com `n` elementos está no índice `n-1`.

```python
idades = [10, 20, 30, 40]

print(idades[0])    # 10 (primeiro elemento)
print(idades[3])    # 40 (ultimo elemento)

idades[1] = 25       # substitui o segundo valor
```

Assim como em C, acessar um índice fora da faixa válida é um erro (`IndexError` em Python, em vez do comportamento indefinido e silencioso de C ao acessar memória fora do vetor) — o interpretador Python, pelo menos, avisa explicitamente quando isso acontece, o que já é uma vantagem prática sobre C nesse ponto.

Python tem um recurso que C não tem: **índices negativos**, que contam a partir do final da lista.

```python
idades = [10, 20, 30, 40]

print(idades[-1])   # 40 (ultimo elemento)
print(idades[-2])   # 30 (penultimo elemento)
```

## 4.4 Percorrendo listas com laços

Assim como em C, é possível percorrer uma lista com `for` usando um índice:

```python
notas = []

for i in range(5):
    valor = float(input("Informe a nota: "))
    notas.append(valor)

for i in range(len(notas)):
    print(f"Nota {i+1}: {notas[i]}")
```

Essa forma (baseada em `range(len(...))`) é a mais parecida com C, mas em Python raramente é a mais idiomática. Lembre-se da Seção 3.2: o `for` de Python itera diretamente sobre os elementos de um iterável — e uma lista é, ela mesma, um iterável. Assim, quando não é necessário o índice, o padrão preferido é:

```python
for nota in notas:
    print(f"Nota: {nota}")
```

Quando o índice **também** é necessário (por exemplo, para numerar a listagem), a forma idiomática não é `range(len(...))`, e sim `enumerate()`, que entrega o par (índice, valor) a cada iteração:

```python
for i, nota in enumerate(notas):
    print(f"Nota {i+1}: {nota}")
```

### Exercício de Exemplo

*Armazene 15 números inteiros em uma lista e, ao final, imprima todos os números lidos.*

```python
num = []

for i in range(15):
    valor = int(input("Informe um numero: "))
    num.append(valor)

for valor in num:
    print(f"Numero: {valor}")
```

Note que os dois laços têm propósitos diferentes: o primeiro usa `range(15)` porque o objetivo é repetir a leitura 15 vezes (o índice `i` em si não importa, só a contagem); o segundo itera direto sobre `num`, porque o objetivo agora é os valores, não a contagem.

### Exercício de Exemplo (busca do maior valor)

*Leia 5 valores inteiros e encontre o maior deles.*

```python
lista = []

for i in range(5):
    valor = int(input("Informe um numero: "))
    lista.append(valor)

maior = lista[0]

for valor in lista[1:]:
    if valor > maior:
        maior = valor

print(f"Maior = {maior}")
```

`lista[1:]` usa **fatiamento** (Seção 4.5) para iterar a partir do segundo elemento, já que o primeiro (`lista[0]`) foi usado para inicializar `maior`. Vale registrar: Python já tem uma função pronta para isso, `max(lista)` — mas construímos a busca manualmente aqui de propósito, porque o objetivo é entender o mecanismo por dentro. Esse mesmo mecanismo (comparar e substituir o "melhor candidato até agora") reaparece, por exemplo, dentro de métodos numéricos iterativos, onde nem sempre existe uma função pronta para o que se precisa.

## 4.5 Fatiamento (*slicing*)

Fatiamento é um recurso que C não tem: extrair uma sublista sem escrever um laço, usando a sintaxe `lista[inicio:fim]` (o mesmo `inicio` incluído / `fim` excluído de `range()`, visto na Aula 3).

**Tabela 1 – Formas de fatiamento**

| Sintaxe | Resultado |
|---|---|
| `lista[2:5]` | Elementos dos índices 2, 3 e 4 |
| `lista[:3]` | Do início até o índice 2 |
| `lista[3:]` | Do índice 3 até o final |
| `lista[:]` | Cópia da lista inteira |
| `lista[::-1]` | A lista inteira, invertida |
| `lista[::2]` | Elementos de índice par (passo 2) |

```python
numeros = [10, 20, 30, 40, 50]

print(numeros[1:4])     # [20, 30, 40]
print(numeros[::-1])    # [50, 40, 30, 20, 10]
```

## 4.6 Métodos úteis de lista

**Tabela 2 – Métodos e funções comuns com listas**

| Uso | Efeito |
|---|---|
| `lista.append(valor)` | Adiciona `valor` ao final |
| `lista.insert(i, valor)` | Insere `valor` na posição `i` |
| `lista.remove(valor)` | Remove a **primeira ocorrência** de `valor` |
| `lista.pop(i)` | Remove e retorna o elemento da posição `i` (último, se omitido) |
| `valor in lista` | Testa se `valor` está na lista (retorna `bool`) |
| `lista.index(valor)` | Retorna o índice da primeira ocorrência de `valor` |
| `len(lista)` | Número de elementos |
| `sum(lista)` | Soma dos elementos |
| `min(lista)` / `max(lista)` | Menor / maior elemento |
| `sorted(lista)` | Retorna uma **nova** lista ordenada (a original não muda) |

Esses recursos resolvem "de fábrica" boa parte do que, em C, exigia escrever um laço manualmente (busca, soma, mínimo, máximo, ordenação). Nos exercícios desta apostila, alguns pedirão explicitamente para implementar isso "na mão" — não por ignorar que a função pronta existe, mas porque o próprio algoritmo por trás da função é o que interessa aprender (isso volta a valer especialmente na Aula 10, dedicada a algoritmos de busca e ordenação, e nos métodos numéricos, onde é comum não haver função pronta para o problema específico).

## 4.7 Compreensão de lista (*list comprehension*)

Um padrão extremamente comum é criar uma lista nova a partir de outra, aplicando uma transformação a cada elemento — por exemplo, o quadrado de cada valor de um vetor. Da forma "tradicional":

```python
vet = [1, 2, 3, 4, 5]
quadrados = []

for x in vet:
    quadrados.append(x ** 2)
```

Python tem uma sintaxe dedicada e mais direta para exatamente esse padrão, chamada **compreensão de lista**:

```python
vet = [1, 2, 3, 4, 5]
quadrados = [x ** 2 for x in vet]
```

As duas formas produzem o mesmo resultado. A compreensão de lista também aceita uma condição, filtrando quais elementos entram na lista nova:

```python
pares = [x for x in vet if x % 2 == 0]
```

Não é uma sintaxe obrigatória — o laço tradicional sempre funciona igual — mas é o estilo esperado em código Python idiomático para transformações simples de uma lista inteira, e vamos usá-la com frequência a partir de agora.

## 4.8 Visualizando dados com `matplotlib`

Até aqui, o único jeito de "ver" um resultado era `print()`. Para dados numéricos — como a evolução de um método iterativo, ou os valores ao longo de uma malha — um gráfico costuma comunicar muito mais do que uma lista de números impressos. A biblioteca padrão para isso em Python científico é o **`matplotlib`**, especificamente seu módulo `pyplot`.

```python
import matplotlib.pyplot as plt

x = [0, 1, 2, 3, 4]
y = [0, 1, 4, 9, 16]

plt.plot(x, y, marker="o")
plt.xlabel("x")
plt.ylabel("y")
plt.title("y = x^2")
plt.show()
```

`plt.plot(x, y)` recebe duas listas de mesmo tamanho — coordenadas x e y — e desenha a curva ligando os pontos; `marker="o"` também marca cada ponto individualmente. `plt.xlabel()`, `plt.ylabel()` e `plt.title()` rotulam o gráfico, e `plt.show()` efetivamente exibe a janela com a figura (se você estiver rodando em um ambiente sem tela — um servidor remoto, por exemplo —, `plt.savefig("nome.png")` salva a figura em arquivo em vez de exibi-la).

### Exercício de Exemplo

*Aplique 6 iterações do método de Newton-Raphson para $f(x) = x^2 - 2$, guardando o valor de $x$ a cada iteração, e plote a convergência.*

```python
import matplotlib.pyplot as plt

iteracoes = []
valores_x = []

x = 2.0  # chute inicial

for i in range(6):
    iteracoes.append(i)
    valores_x.append(x)
    x = x - (x**2 - 2) / (2 * x)

plt.plot(iteracoes, valores_x, marker="o")
plt.xlabel("Iteracao")
plt.ylabel("x")
plt.title("Convergencia do metodo de Newton-Raphson")
plt.show()
```

Note que este exercício é a primeira vez que **guardamos** o histórico de um método iterativo, em vez de só acumular o valor mais recente (como fizemos na Aula 3). É exatamente essa mudança — de "uma variável que se atualiza" para "uma lista que registra tudo o que aconteceu" — que separa "rodar o método" de "poder analisar como o método se comportou", e é o que vamos explorar em vários dos exercícios desta aula.

## 4.9 Resumo da Aula

Nesta aula, os programas passaram a poder armazenar coleções de valores, não apenas um de cada vez:

- Listas em Python são dinâmicas — não têm tamanho fixo como os vetores de C, e crescem com `.append()`.
- O acesso por índice segue a mesma lógica de C (0 a n-1), mas Python adiciona índices negativos e fatiamento (`lista[a:b]`).
- Percorrer uma lista com `for valor in lista:` é preferível a `for i in range(len(lista)):` quando o índice não é necessário; `enumerate()` resolve quando os dois são necessários juntos.
- Funções e métodos prontos (`len`, `sum`, `min`, `max`, `sorted`, `.append`, `in`) resolvem tarefas comuns sem laço manual — mas vale entender como implementá-las na mão, porque essa lógica reaparece em contextos sem equivalente pronto.
- Compreensão de lista (`[expressao for item in lista]`) é a forma idiomática de gerar uma lista nova a partir de outra.
- `matplotlib.pyplot` permite visualizar listas de dados como gráficos, o que passa a ser especialmente útil para acompanhar a convergência de métodos numéricos.

A partir da Aula 5, listas de listas (matrizes) vão permitir representar dados em duas dimensões — o que é exatamente o que é preciso para montar, de fato, a primeira matriz de rigidez de um problema de Elementos Finitos.

---

## 4.10 Exercícios da Aula

Tipos: **[P]** programação geral, **[M]** modelagem, **[MN]** métodos numéricos, **[DF]** diferenças finitas, **[MEF]** Método dos Elementos Finitos.

### Nível Fácil

1. **[P]** Leia 10 números inteiros, armazene em uma lista e imprima a lista na ordem inversa da leitura, usando fatiamento (`[::-1]`).
2. **[P]** Leia 15 números em uma lista. Em seguida, peça um número a ser buscado e informe em qual posição ele foi encontrado (ou "Nao encontrado", caso contrário) — implemente a busca com um laço manual, sem usar `.index()`.
3. **[M]** Leia as notas de 10 alunos, armazene em uma lista e imprima a média da turma.
4. **[P]** Dada a lista `salarios = [2500, 3200, 1800, 4100, 2950]`, gere uma nova lista com os salários reajustados em 8%, usando compreensão de lista.
5. **[DF]** Leia $x_0$ e um passo $h$, e crie, com uma compreensão de lista, os 11 pontos da malha $x_0, x_0+h, \ldots, x_0+10h$ (a mesma malha do exercício da Aula 3, agora efetivamente armazenada em uma lista).
6. **[MEF]** Crie a lista `nos = [0, 2, 5, 10]`, representando os nós de uma barra dividida em 3 elementos. Escreva um laço que imprima, para cada elemento, seu comprimento (a diferença entre as coordenadas de nós consecutivos).

### Nível Médio

7. **[P]** Leia 20 números em uma lista e, com compreensão de lista, gere uma segunda lista contendo apenas os valores pares da primeira.
8. **[M]** Leia o preço de compra e o preço de venda de 20 mercadorias (armazenados em duas listas paralelas) e conte quantas têm lucro abaixo de 10%, entre 10% e 20%, e acima de 20%.
9. **[MN]** Implemente o método de Newton-Raphson para $f(x) = x^2 - 2$ armazenando **todas as iterações** em uma lista (não apenas a última). Imprima a lista completa e plote (matplotlib) o valor de $x$ ao longo das iterações.
10. **[DF]** Para a malha do exercício 5 e $f(x) = x^2$, calcule a derivada aproximada em cada ponto (progressiva no primeiro ponto, regressiva no último, central nos internos, como na Aula 3), armazenando os resultados em uma lista, e plote $x$ contra a derivada aproximada.
11. **[MEF]** A partir da lista de nós do exercício 6, escreva um laço que, dado um ponto $x$ informado pelo usuário, percorra a lista de nós e determine em qual elemento o ponto está — generalizando, para qualquer malha, a ideia de localização de ponto já vista na Aula 2 e na Aula 3 (que funcionava só para 3 elementos fixos).
12. **[P]** Leia dois conjuntos de 10 números inteiros cada, armazenados em duas listas, e imprima os elementos que aparecem em ambas — usando apenas laços e `if` (sem usar `set`).

### Nível Difícil (elaborados)

13. **[MN]** *Bisseção com histórico de convergência.* Implemente a bisseção completa (Aula 3) para $f(x)=x^3-x-2$, armazenando em uma lista o valor de $|b-a|$ a cada iteração. Plote (com `plt.yscale("log")`) o tamanho do intervalo em função da iteração — isso evidencia visualmente que a bisseção converge **linearmente** (o erro cai por um fator aproximadamente constante a cada passo), o que será útil para comparar, mais adiante, com a velocidade de convergência de Newton-Raphson.
14. **[DF]** *Euler com trajetória completa, três passos.* Implemente o método de Euler para $\dfrac{dy}{dt} = -2y + t$, armazenando $t$ e $y$ de cada passo em duas listas. Repita para três valores de passo ($h=0{,}5$; $h=0{,}1$; $h=0{,}01$) e plote as três curvas na mesma figura (chame `plt.plot()` três vezes antes de `plt.show()`), com legenda (`plt.legend()`) identificando cada uma — visualizando diretamente como refinar o passo aproxima a solução numérica da solução real.
15. **[MEF]** *Vetor de carga nodal.* Para a barra de 4 nós do exercício 6, leia a força concentrada aplicada em cada nó (0 se não houver) e armazene em uma lista `F` de tamanho 4 — na prática, o vetor de forças nodais que, na Aula 5, será combinado com a matriz de rigidez para resolver o sistema $Ku = F$. Verifique se a soma dos valores de `F` é compatível com o equilíbrio estático de uma barra (a soma de todas as forças, incluindo reações de apoio com sinal oposto, deve ser zero) — para isso, o programa deve também pedir a reação de apoio e conferir a soma total.
16. **[P]** *Remoção de duplicatas preservando a ordem, sem `set`.* Leia 20 números inteiros (podem repetir) e construa uma segunda lista contendo apenas os valores distintos, na ordem em que apareceram pela primeira vez — usando apenas listas, laços e o operador `in` (não use `set()`).
17. **[M]** *Pesquisa de opinião com listas paralelas.* Leia, para até 30 pessoas (ou até o usuário digitar uma idade negativa para encerrar antes), a idade e a opinião sobre um produto (nota de 1 a 5), armazenando em duas listas paralelas. Calcule a média de idade de quem deu nota 5 e a porcentagem de quem deu nota 1 — sem criar nenhuma lista além das duas já usadas para a leitura.
18. **[MEF]** *Refinamento de malha.* Para o domínio $[0, 10]$, gere (com compreensão de lista) três malhas igualmente espaçadas: com 3, 10 e 30 elementos. Para cada malha, calcule o comprimento médio de elemento e armazene os três resultados em uma lista. Plote (gráfico de linha ou de barras) o comprimento médio do elemento em função do número de elementos da malha — ilustrando o conceito de **refinamento de malha**, central tanto em Diferenças Finitas quanto em Elementos Finitos: mais elementos significam elementos menores, o que em geral (mas nem sempre, como será visto para o caso instável de Euler) melhora a precisão do método.

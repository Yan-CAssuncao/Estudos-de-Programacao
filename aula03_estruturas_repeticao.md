# AULA 3 — Estruturas de Repetição

### Metas da Aula
1. Entender e praticar as duas cláusulas de repetição de Python: `for` e `while`.
2. Compreender a diferença estrutural entre o `for` de Python (baseado em iteráveis) e o `for` de C (baseado em contador).
3. Escrever programas que processem várias entradas de dados, acumulando somas, contagens, médias, mínimos e máximos.

### Ao término desta aula, você será capaz de:
1. Escrever programas que repitam um bloco de instruções um número conhecido ou desconhecido de vezes.
2. Combinar laços com as estruturas de decisão da Aula 2 para resolver problemas mais elaborados.
3. Evitar (e reconhecer) laços infinitos.

---

## 3.1 Estruturas de Repetição

Até aqui, mesmo com decisão (Aula 2), cada instrução do programa era executada no máximo uma vez. Muitos problemas exigem repetir um mesmo bloco de instruções várias vezes — ler 20 notas de alunos, por exemplo, sem escrever a leitura 20 vezes no código. Isso é resolvido com **estruturas de repetição**, também chamadas de laços ou *loops*.

Python tem duas cláusulas de repetição: `for` e `while`. Diferente de C, **não existe `do-while`** — a Seção 3.9 mostra a alternativa.

## 3.2 Cláusula `for`

Esta é a diferença mais importante entre C e Python nesta aula: o `for` de C é controlado por um contador (inicialização, condição, incremento); o `for` de Python **itera sobre os elementos de uma sequência** (uma lista, uma string, ou, o caso mais comum para controlar repetições, um objeto `range`).

```python
for variavel in iteravel:
    instrucao
```

A cada volta do laço, `variavel` recebe o próximo elemento de `iteravel`, até que os elementos se esgotem. Para repetir um bloco um número fixo de vezes — o uso mais parecido com o `for` de C — usa-se `range()`:

**Tabela 1 – Formas de `range()`**

| Chamada | Sequência gerada |
|---|---|
| `range(n)` | `0, 1, 2, ..., n-1` (n valores) |
| `range(inicio, fim)` | `inicio, inicio+1, ..., fim-1` |
| `range(inicio, fim, passo)` | `inicio, inicio+passo, ...` até o último valor **menor que** `fim` (ou maior, se `passo` for negativo) |

O detalhe que mais causa confusão em quem vem de C: **o valor `fim` nunca é incluído**. `range(10)` gera 10 valores (0 a 9), não 11.

### Exercício de Exemplo

*Leia 10 valores e imprima a média aritmética entre eles.*

```python
soma = 0

for i in range(10):
    num = float(input("Informe o numero: "))
    soma += num

media = soma / 10
print(f"A media e: {media:.2f}")
```

`range(10)` gera exatamente 10 valores (0 a 9), então o bloco indentado é executado 10 vezes — o valor de `i` em si não é usado dentro do laço, ele só está ali para controlar quantas vezes o bloco repete.

Se a quantidade de valores não for fixa, mas informada pelo usuário, basta usar essa quantidade como argumento de `range()`:

```python
qtd_num = int(input("Informe a quantidade de numeros: "))
soma = 0

for i in range(qtd_num):
    num = float(input("Informe o numero: "))
    soma += num

media = soma / qtd_num
print(f"A media e: {media:.2f}")
```

`range()` também aceita passo diferente de 1, inclusive negativo — o que permite laços com incremento maior que 1 ou decrescentes, sem precisar de nenhuma cláusula especial:

```python
# multiplos de 10 entre 0 e 100
for i in range(0, 101, 10):
    print(f"Multiplo de 10: {i}")
```

```python
# de 10 até 0, decrescente
for i in range(10, -1, -1):
    print(f"Numero: {i}")
```

**Uma diferença que vale destacar:** em C, é possível (por erro de programação) criar um `for` que nunca termina, bagunçando o incremento em relação à condição de parada. Em Python isso **não acontece com `for`**: a sequência gerada por `range()` é definida por completo antes da primeira iteração, então um `for` sobre um `range()` sempre é finito. Um laço infinito em Python só acontece com `while` (Seção 3.8).

## 3.3 `for` aninhado

Assim como o `if`, um `for` pode conter outro `for` em seu bloco — útil, por exemplo, para preencher uma tabela linha por linha, coluna por coluna.

### Exercício de Exemplo

*Imprima uma tabuada de multiplicação de 1 a 4, em formato de tabela.*

```python
for i in range(1, 5):
    for j in range(1, 5):
        print(j * i, end="\t")
    print()
```

O `for` externo (`i`) controla as linhas; o `for` interno (`j`) controla as colunas, sendo totalmente reiniciado a cada volta do laço externo. O parâmetro `end="\t"` faz o `print()` terminar com uma tabulação em vez da quebra de linha padrão — assim, os valores de uma mesma linha ficam lado a lado; o `print()` vazio depois do laço interno é o que força a quebra de linha entre uma linha da tabela e a próxima.

## 3.4 Cláusula `while`

Quando o número de repetições não é conhecido de antemão — depende de uma condição que só pode ser avaliada durante a execução —, usa-se `while`:

```python
while condicao:
    instrucao
```

A condição é avaliada **antes** de cada iteração: se for falsa logo de início, o bloco pode não executar nenhuma vez.

### Exercício de Exemplo

*Leia um número $n$ e calcule a soma de todos os inteiros de 1 a $n$.*

```python
n = int(input("Informe o numero n: "))
i = 1
soma = 0

while i <= n:
    soma += i
    i += 1

print(f"Soma: {soma}")
```

Note o que muda em relação ao `for`: aqui é preciso inicializar `i` **antes** do laço e incrementá-lo manualmente **dentro** do bloco (`i += 1`). No `for`, isso era automático. Essa é justamente a razão de existirem as duas cláusulas: `for` quando a "contagem" cuida de si mesma; `while` quando ela depende de alguma lógica própria do problema.

O `while` também aceita condições compostas, com `and`/`or`/`not`, exatamente como o `if`.

### Exercício de Exemplo (condição composta)

*Uma agência bancária cadastra no máximo 10 mil clientes. Leia o número da conta, o nome e o saldo de cada cliente até o usuário digitar -999 ou o limite ser atingido. Ao final, informe o total de clientes com saldo negativo e o total de clientes cadastrados.*

```python
c_tot_neg = 0
c_tot = 0

conta = int(input("Digite o numero da conta ou -999 para terminar: "))

while conta > 0 and c_tot < 10000:
    c_tot += 1
    nome = input("Nome: ")
    saldo = float(input("Saldo: "))

    if saldo < 0:
        c_tot_neg += 1
        print(f"{conta} - {saldo:.2f} - negativo")
    else:
        print(f"{conta} - {saldo:.2f} - positivo")

    conta = int(input("Digite o numero da conta ou -999 para terminar: "))

print(f"\nTotal de clientes com saldo negativo: {c_tot_neg}")
print(f"Total de clientes da agencia: {c_tot}")
```

Repare que a leitura de `conta` aparece **duas vezes**: uma antes do `while` (para que a variável já tenha um valor na primeira avaliação da condição) e outra ao final do bloco (para que a condição seja reavaliada com um novo valor a cada volta). Esse padrão — ler antes do laço, processar, ler de novo no fim do laço — é comum sempre que o critério de parada depende do próprio dado lido.

## 3.5 Validação de dados com `while`

Na Aula 2, o exemplo do peso ideal aceitava silenciosamente qualquer entrada diferente de "M" como se fosse "F". Agora, com `while`, é possível **forçar** que o usuário informe um valor válido antes de o programa continuar.

### Exercício de Exemplo

*Leia 10 números positivos e imprima o quadrado de cada um. Se um número não positivo for digitado, o programa deve pedir novamente até receber um valor válido.*

```python
for i in range(10):
    num = float(input("Informe um numero: "))

    while num <= 0:
        print("ATENCAO! Informe um numero maior que zero:")
        num = float(input())

    print(f"Quadrado: {num * num:.2f}")
```

O `while` interno só executa (e só volta a executar) enquanto o número informado for inválido; assim que o usuário digita um valor positivo, a condição `num <= 0` se torna falsa e o laço de validação é abandonado, seguindo para o cálculo do quadrado.

## 3.6 `while` aninhado

Da mesma forma que o `for`, um `while` pode conter outro `while` (ou um `for`) em seu bloco.

### Exercício de Exemplo

*Um material radioativo perde 25% de sua massa a cada 30 segundos. Leia repetidamente a massa inicial de uma amostra e calcule quanto tempo leva até a massa ficar abaixo de 0,10 grama. Repita para quantas amostras o usuário desejar.*

```python
resp = input("Digite S se desejar novo calculo ou qualquer letra para terminar: ").upper()

while resp == "S":
    massa = float(input("Digite a massa em gramas do material: "))
    con_tempo = 0

    while massa >= 0.10:
        con_tempo += 1
        massa *= 0.75

    tempo = (con_tempo * 30) / 60
    print(f"O tempo foi de: {tempo:.2f} minutos.")
    resp = input("\nDigite S se desejar novo calculo ou qualquer letra para terminar: ").upper()
```

O `while` externo controla quantas amostras serão calculadas; o `while` interno, reiniciado a cada amostra (`con_tempo = 0` a cada volta do laço externo), calcula o tempo necessário para aquela amostra específica.

## 3.7 Loop infinito

Um laço infinito acontece quando a condição de parada nunca se torna falsa. Em `for` sobre `range()` isso não ocorre (Seção 3.2); em `while`, é um erro real e comum — geralmente porque a variável de controle não é atualizada dentro do bloco, ou é atualizada no lugar errado.

```python
# ERRADO: loop infinito
idade = 0
resp = 1

while resp == 1:
    idade = int(input("Digite a idade: "))
    print(f"A idade e: {idade}")

print("\nDigite 1 para continuar ou outro numero para terminar: ")
resp = int(input())
```

O erro aqui é o mesmo tipo de erro clássico do livro de C: as duas últimas linhas, que atualizam `resp`, estão **fora** do bloco do `while` — como `resp` nunca muda dentro do laço, a condição `resp == 1` permanece verdadeira para sempre. A correção é mover essas linhas para dentro do bloco:

```python
# CORRIGIDO
resp = 1

while resp == 1:
    idade = int(input("Digite a idade: "))
    print(f"A idade e: {idade}")
    print("\nDigite 1 para continuar ou outro numero para terminar: ")
    resp = int(input())
```

Regra prática: sempre que escrever um `while`, pergunte-se "o que, dentro deste bloco, pode eventualmente tornar a condição falsa?" Se a resposta for "nada", há um laço infinito.

## 3.8 E o `do-while`? `break` e `continue`

Python **não tem** cláusula `do-while` (a estrutura que executa o bloco pelo menos uma vez, testando a condição só no final). A alternativa idiomática combina `while True` — uma condição sempre verdadeira, ou seja, um laço proposital e controladamente infinito — com a palavra-chave **`break`**, que interrompe o laço imediatamente de dentro do bloco:

```python
while True:
    instrucao
    if condicao_de_saida:
        break
```

### Exercício de Exemplo

*Leia números e imprima o quadrado de cada um, até que o número lido seja múltiplo de 6 — que também deve ter seu quadrado impresso antes de o programa parar.*

```python
while True:
    num = int(input("\nDigite um numero ou multiplo de 6 para encerrar: "))
    print(f"Quadrado: {num * num}")

    if num % 6 == 0:
        break
```

Como o teste (`if num % 6 == 0`) está **depois** de calcular e imprimir o quadrado, o número múltiplo de 6 também é processado antes de o laço parar — reproduzindo exatamente o comportamento do `do-while` de C.

Esse padrão também é o mais usado para implementar menus:

```python
while True:
    print("1-Soma varios numeros")
    print("2-Multiplica varios numeros")
    print("3-Encerrar o programa")
    op = int(input("Opcao: "))

    if op == 1:
        soma = 0
        num = float(input("\nDigite numero ou -999 para finalizar: "))
        while num != -999:
            soma += num
            num = float(input("\nDigite numero ou -999 para finalizar: "))
        print(f"\nSoma: {soma}")
    elif op == 2:
        prod = 1
        num = float(input("\nDigite numero ou -999 para finalizar: "))
        while num != -999:
            prod *= num
            num = float(input("\nDigite numero ou -999 para finalizar: "))
        print(f"\nProduto: {prod}")
    elif op == 3:
        print("\nPrograma encerrado!")
        break
    else:
        print("\nOpcao nao disponivel!")
```

Existe também a palavra-chave **`continue`**, que não interrompe o laço, mas pula direto para a próxima iteração, ignorando o restante do bloco atual — útil, por exemplo, para descartar um valor inválido sem encerrar a leitura. Ela aparecerá com mais frequência a partir da Aula 4, combinada com listas.

## 3.9 Exemplos adicionais: contar, somar, mínimo e máximo

Boa parte dos problemas com laço se resume a quatro operações: **contar**, **somar** (para depois calcular média), e obter **mínimo**/**máximo**.

### Exercício de Exemplo (contar e somar)

*Uma transportadora quer saber a quantidade e o peso total de caixas que serão carregadas, uma a uma, até o usuário indicar que não há mais caixas.*

```python
qtd_volumes = 0
peso_total = 0

resp = input("Deseja cadastrar uma caixa? <S/N>: ").upper()

while resp == "S":
    qtd_volumes += 1
    peso = float(input("Informe o peso da caixa: "))
    peso_total += peso
    resp = input("Deseja cadastrar uma caixa? <S/N>: ").upper()

peso_medio = peso_total / qtd_volumes

print(f"Quantidade de volumes: {qtd_volumes}")
print(f"Peso total dos volumes: {peso_total:.2f}")
print(f"Peso medio dos volumes: {peso_medio:.2f}")
```

### Exercício de Exemplo (mínimo e máximo)

*Um frigorífico registra 90 bois, cada um com identificação e peso. Encontre o boi mais pesado e o mais leve.*

```python
id_boi_gordo = None
id_boi_magro = None
peso_boi_gordo = 0
peso_boi_magro = 0

for i in range(90):
    id_boi = int(input("Informe a identificacao do boi: "))
    peso_boi = float(input("Informe o peso do boi: "))

    if peso_boi > peso_boi_gordo:
        id_boi_gordo = id_boi
        peso_boi_gordo = peso_boi

    if peso_boi < peso_boi_magro or i == 0:
        id_boi_magro = id_boi
        peso_boi_magro = peso_boi

print(f"Identificacao do boi mais gordo: {id_boi_gordo}")
print(f"Peso do boi mais gordo: {peso_boi_gordo:.2f}")
print(f"Identificacao do boi mais magro: {id_boi_magro}")
print(f"Peso do boi mais magro: {peso_boi_magro:.2f}")
```

A lógica de mínimo/máximo "manual" (sem usar `min()`/`max()`, que só fazem sentido a partir de uma coleção de valores — Aula 4) segue o mesmo raciocínio do livro de C: compara-se cada novo valor lido com o melhor valor encontrado até então, atualizando quando um valor "vence" a comparação. O truque de `i == 0` garante que o primeiro boi lido sempre inicialize `peso_boi_magro`, já que `0` não é um limite inferior confiável (pesos reais são sempre maiores que zero, então começar comparando com `0` funcionaria por acaso aqui — mas `i == 0` é a forma robusta, que funcionaria mesmo se os valores pudessem ser negativos).

## 3.10 Resumo da Aula

Nesta aula, os programas passaram a poder repetir blocos de instruções:

- `for variavel in iteravel:` repete um número de vezes definido por uma sequência, normalmente `range()`. Em Python, isso não pode gerar loop infinito.
- `while condicao:` repete enquanto uma condição for verdadeira, sendo a escolha certa quando o número de repetições não é conhecido de antemão.
- Não existe `do-while` em Python; o equivalente é `while True:` combinado com `break`.
- `break` interrompe um laço imediatamente; `continue` pula para a próxima iteração.
- Laços podem ser aninhados, e podem (e frequentemente devem) ser combinados com `if`/`elif`/`else` da Aula 2.
- A maioria dos problemas com laço se resolve com as operações de contar, somar e comparar para mínimo/máximo — sempre inicializando a variável de acumulação **antes** do laço.

A partir da Aula 4, com listas, será possível armazenar todos os valores lidos (não só acumular um resultado), o que abre caminho para operações mais sofisticadas — inclusive o início de verdade da montagem de estruturas usadas em Elementos Finitos.

---

## 3.11 Exercícios da Aula

Os exercícios usam os recursos vistos até aqui: `if`/`elif`/`else`, `for`, `while`, `break`. Ainda não utilize listas (`[]`) ou `def` — vêm nas próximas aulas. Tipos: **[P]** programação geral, **[M]** modelagem, **[MN]** métodos numéricos, **[DF]** diferenças finitas, **[MEF]** Método dos Elementos Finitos.

### Nível Fácil

1. **[P]** Imprima todos os números de 1 até 100, usando `for`.
2. **[P]** Imprima todos os números pares de 100 até 0, em ordem decrescente.
3. **[P]** Leia 5 números e imprima a soma e a média entre eles.
4. **[M]** Leia o salário de 10 funcionários e, ao final, imprima o total da folha de pagamento e o salário médio.
5. **[MN]** Leia um número inteiro $n$ e calcule a soma dos $n$ primeiros termos da série harmônica, $1 + \dfrac{1}{2} + \dfrac{1}{3} + \cdots + \dfrac{1}{n}$.
6. **[DF]** Leia um ponto inicial $x_0$ e um passo $h$, e imprima, usando `for`, os 11 pontos de uma malha unidimensional igualmente espaçada: $x_0, x_0+h, x_0+2h, \ldots, x_0+10h$ — a base de qualquer malha usada em diferenças finitas.

### Nível Médio

7. **[P]** Leia números, um a um, até que o usuário digite 0. Ao final, informe quantos dos números digitados estavam entre 100 e 200 (incluindo os extremos).
8. **[M]** Calcule e imprima os 20 primeiros termos da sequência de Fibonacci (o primeiro e o segundo termos valem 1; cada termo seguinte é a soma dos dois anteriores).
9. **[MN]** Implemente o método da **bisseção completo** para $f(x) = x^3 - x - 2$: leia $a$, $b$ e uma tolerância; repita reduzindo o intervalo até que $|b - a|$ seja menor que a tolerância **ou** até um número máximo de iterações (por exemplo, 100), o que ocorrer primeiro. Imprima a raiz aproximada e quantas iterações foram usadas.
10. **[DF]** Implemente o método de **Euler completo**: dado $y_0$, $t_0$, um passo $h$ e um número de passos $n$, calcule $y_1, y_2, \ldots, y_n$ para $\dfrac{dy}{dt} = -2y + t$, imprimindo o valor de $t$ e de $y$ a cada passo.
11. **[MEF]** Uma barra é composta por $n$ elementos de mola em série, cada um com rigidez $k_i$. Leia $n$ e, em um laço, leia a rigidez de cada elemento, acumulando a rigidez equivalente da associação: $\dfrac{1}{k_{eq}} = \displaystyle\sum_{i=1}^{n} \dfrac{1}{k_i}$. Imprima $k_{eq}$.
12. **[M]** Leia a população inicial e a taxa de natalidade anual de dois países, A e B (sabendo que a população de A é menor que a de B). Usando `while`, calcule ano a ano em quantos anos a população de A ultrapassa a de B.

### Nível Difícil (elaborados)

13. **[MN]** Implemente o método de **Newton-Raphson completo, com critério de segurança**, para $f(x) = x^3 - 2x - 5$: leia $x_0$ e uma tolerância; repita $x_{i+1} = x_i - f(x_i)/f'(x_i)$ até que $|x_{i+1}-x_i|$ seja menor que a tolerância **ou** até atingir um número máximo de iterações (ex.: 100) — informando ao final se o método convergiu ou se foi interrompido por exceder o limite.
14. **[DF]** *Derivadas por diferenças finitas ao longo de uma malha.* Considere uma malha de $n+1$ pontos igualmente espaçados, de $x_0$ a $x_n$ (passo $h$), para $f(x) = x^2$. Percorra os pontos com um laço e, para cada um, calcule a derivada aproximada usando o esquema apropriado: diferença progressiva no primeiro ponto, regressiva no último, e central nos pontos internos (combine `for` com `if`/`elif`). Imprima o índice, o $x$ e a derivada aproximada de cada ponto.
15. **[MEF]** *Localização de múltiplos pontos em uma malha de elementos.* Reaproveitando a malha de 3 elementos da Aula 2, leia quantos pontos o usuário deseja consultar e, em um laço, para cada ponto leia sua coordenada $x$, determine o elemento correspondente e sua coordenada natural $\xi$, e informe ao final quantos pontos caíram em cada um dos três elementos.
16. **[M]** Uma pousada cobra R\$30,00 de diária mais uma taxa de serviço diária de R\$15,00 (menos de 10 dias) ou R\$8,00 (10 dias ou mais). Leia dados de clientes, um a um (nome e número de dias), até que o usuário digite 0 como número de dias, calculando e imprimindo a conta de cada cliente e, ao final, o total faturado pela pousada.
17. **[MN]** *Integração numérica pela regra do trapézio composta.* Generalize o exercício da Aula 1 (um único trapézio) para $n$ subintervalos: leia $a$, $b$ e $n$, calcule $h = \frac{b-a}{n}$ e, em um laço, some a contribuição de cada trapézio para aproximar $\int_a^b x^2\,dx$. Compare com o valor exato ($\frac{b^3-a^3}{3}$) e imprima o erro absoluto. Teste o próprio programa para $n=1$, $n=10$ e $n=100$, observando como o erro diminui ao refinar a malha — a ideia central por trás de qualquer método numérico baseado em discretização, seja em DF ou em MEF.
18. **[DF]** *Instabilidade do método de Euler.* Para $\dfrac{dy}{dt} = -k y$ com $k=50$, implemente o método de Euler completo para dois valores de passo: $h=0{,}01$ (estável, pois $h < 2/k$) e $h=0{,}1$ (instável). Rode ambos por 20 passos e imprima os valores de $y$ a cada passo nos dois casos, observando diretamente nos números impressos como o caso instável diverge (valores crescendo em módulo e alternando de sinal), enquanto o estável converge suavemente a zero.

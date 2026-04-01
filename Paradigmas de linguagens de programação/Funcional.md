
# A arte de não mudar nada
No mundo imperativo que estamos acostumados (C, Java sem OO), a gente trata a memória como um quadro branco que a gente fica riscando e apagando (isso é um estado mutável), no funcional a gente passa a tratar a memória como um livro: uma vez escrito, o dado é eterno. Quer mudar algo? Escreve uma página nova (cria um novo dado).
> [!TIP] 
> #### Larga o dogmaaaa
> Larga o dogma de imperativo, é tipo português, é só um jeito de falar, mas tem o inglês, o francês, o espanhol, o latim... Aprender funcional expande sua semântica de resolução de problemas.

## Pilar da imutabilidade
Aqui não tem essa de `x = x + 1`. Se `x` é 10, ele vai morrer sendo 10, isso garante que ele permaneça desde sempre no estado imutável e sem permissão para outra função por exemplo, vir e mudar o valor de `x`.
- **Vantagem:** Não existe mais bug de "quem que foi o abençoado que mudou a variável na linha 469 da BTree?"
- **Consequência:** É naturalmente Thread-Safety. Se ninguém muda nada, ninguém briga pelo dado, e isso torna o código bem mais previsível pois nada muda por baixo dos panos.
```haskell
x = 10
y = x + 1 -- y é 11 e x continua 10
```
## Funções puras
Uma função funcional é como uma fórmula matemática $f(x) = y$. Ela tem o comportamento determinístico, ela não olha pro banco de dados, não imprime no console, não muda variável global, ela é totalmente isolada pra cumprir sua única função e trabalhar totalmente dentro dos seus inputs.
Propriedades legais:
* **Determinismo:** mesmo input -> mesmo output
* **Sem efeitos colaterais:** não altera nada fora dela
>[!NOTE]
>#### Referencial Transparency: 
 >Você pode substituir a chamada da função pelo resultado sem mudar o comportamento do programa. Ex: Se $f(10) = 20$, qualquer lugar com $f(10)$ pode virar $20$. Isso facilita Otimização, Paralelismo e Testes.
## Map, Filter e Reduce
Em vez de fazer `for` ou `while` infinitos ou não, a gente usa transformações de listas.
### Map
Aplica uma função a cada elemento da lista e retorna uma lista.
```haskell
map (*2) [1,2,3] -- isso retorna uma lista com [2,4,6]
```
### Filter
Filtra os elementos com base numa condição e retorna uma lista
```haskell
filter even [1,2,3,4,5] -- retorna [2,4]
```
### Reduce (ou fold em algumas langs pelo que vi)
Se o Map transforma os elementos e o Filter limpa o que não interessa, o reduce resume. Ele percorre a lista carregando um acumulador e vai processando o que encontrar pelo caminho. 
Pra ele funcionar, precisa ter 3 coisas:
* Um acumulador (acc): onde o resultado parcial fica guardado.
* O elemento atual (curr): o item da lista que está sendo processado agora
* Um valor inicial: de onde a conta começa (se for uma soma, começa em 0. Se for uma multiplicação, começa em 1)
Exemplo:
```haskell
-- somando tudo (acc + atual) começando em 0
foldl (+) 0 [1,2,3,4,5] -- retorna 15
```
O que diabos tá acontecendo? Magia dos magos... segue uma estrutura geral de como funciona:
```haskell
foldl f acc []     = acc -- Caso base: lista vazia, retorna o acumulador
foldl f acc (x:xs) = foldl f (f acc x) xs -- Caso recursivo: aplica a função e vai pro resto
```

Para sua cabecinha imperativa entender vou me rebaixar ao seu nível, veja como se fosse um algoritmo em C que você estivesse fazendo o rastreamento do que a função faz passo a passo:

|**Passo**|**Acumulador (acc)**|**Elemento Atual (x)**|**Operação (acc + x)**|**Lista Restante**|
|---|---|---|---|---|
|**0 (Init)**|`0`|-|-|`[1,2,3,4,5]`|
|**1**|`0`|`1`|`0 + 1 = 1`|`[2,3,4,5]`|
|**2**|`1`|`2`|`1 + 2 = 3`|`[3,4,5]`|
|**3**|`3`|`3`|`3 + 3 = 6`|`[4,5]`|
|**4**|`6`|`4`|`6 + 4 = 10`|`[5]`|
|**5**|`10`|`5`|`10 + 5 = 15`|`[]`|

É como se ela fosse formando recursivamente uma expressão matemática em que coloca o `elemento atual` na esquerda $((((0+1)+2)+3)+4)+5 = 15$.
## O For deixa o protagonismo, viva à recursão!
Se a gente não tem estado mutável, a gente não tem como usar o clássico contador `int i = 0` que vai iterando no for. Então como a gente faz isso?

A ideia é em vez de falar "faça isso 10 vezes" como a nossa mente de programador pensa, a gente diz pra máquina "faça isso para o primeiro item da lista e depois chame a mesma função para o resto da lista".
Exemplo:
```haskell
meuSum [] = 0
meuSum (x:xs) = x + meuSum xs
```
Percebe? descrevi um caso base para quando a lista estiver vazia (que retorna 0 pois não há elementos para somar), e o caso de lista que tiver uma cabeça (qualquer elemento) `x` e resto `xs` que não importa para a função no momento atual, mas que passo recursivamente para a mesma função trabalhar ela depois, a soma é sempre x + soma do resto.

## Funções de primeira classe
Antes de falar de HOFs, entenda: no funcional, a função não é só um bloco de código que você "chama". Ela é um **valor**, igualzinho a um `int` ou uma `string`.
* Você pode passar uma função para uma variável.
* Você pode colocar funções dentro de uma lista
* Você pode passar uma função como "frete" pra outra função aplicar.
## Funções de Alta Ordem (HOF)
Uma função é considerada de Alta Ordem se ela fizer pelo menos uma dessas coisas:
1. Receber uma ou mais funções como argumento.
2. Retornar uma função como resultado.
### Tá...legal...pq raios eu faria isso?
Por dois motivos:
#### Injeção de comportamento
Em vez de escrever 10 funções quase iguais, tu pode escrever uma HOF que recebe o "detalhe" do que deveria ser feito. Imagine que você quer processar uma lista de números, mas o _como_ processar muda.
Exemplo:
```haskell
-- HOF que recebe uma função (Int -> Int) e uma lista 
processarLista :: (Int -> Int) -> [Int] -> [Int] 
processarLista func lista = map func lista 

-- agora injetamos o comportamento na chamada: 
main = do 
let nums = [1, 2, 3] 
print (processarLista (\x -> x * x) nums) -- injetamos "elevar ao quadrado" 
print (processarLista (\x -> x + 10) nums) -- Injetamos "somar dez"
```
> [!NOTE]
> #### Comentário nerd
> Perceba que `processarLista` é genérica. Ela não sabe o que vai acontecer com o número, ela só sabe como percorrer a estrutura e aplicar a função injetada.
#### Fábrica de funções
Aqui entra a mágica de retornar funções, tu pode criar uma função que gera outras funções customizadas. Você cria uma função "mestra" que, ao ser chamada, te devolve uma função nova e já pré-configurada.
Exemplo:
```haskell
-- Função que gera um "multiplicador" customizado
gerarMultiplicador :: Int -> (Int -> Int)
gerarMultiplicador fator = (\numero -> numero * fator)

-- Criando funções específicas na nossa "fábrica"
double = gerarMultiplicador 2
triple = gerarMultiplicador 3

main = do
    print (double 5) -- Retorna 10
    print (triple 5) -- Retorna 15
```
>[!TIP]
>#### O que aconteceu aqui?
Quando rodamos `double = gerarMultiplicador 2`, a variável `double` não guarda um número, ela guarda uma **função viva** que "lembra" que o seu `fator` é 2.

## Monads
er...ainda tô bugado nessa parte, o professor não comentou mas o Gemini falou que é importante, vou ver melhor depois mas para quem quiser, já anexei um link que me ajudou +/- (ultimo link)
![](../assets/holy_shit.png)

## Ref:
- https://learnyouahaskell.github.io
- https://medium.com/@marcellguilherme/aprenda-tudo-sobre-reduce-ou-fold-fd71de86ce53
- https://medium.com/rung-brasil/ent%C3%A3o-voc%C3%AA-ainda-n%C3%A3o-entende-monads-1ea62e0c14a7
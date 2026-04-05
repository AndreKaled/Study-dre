Este algoritmo ao invés de testar a "primalidade" de cada número individualmente, o que seria $O(\sqrt(N))$ por número, o crivo constrói uma Lookup Table de todos os primos até N em um número quase linear de $O(n\log(\log(n)))$.

Implementação:
```java
public boolean[] crivo(int n){
	// false = primo, true = nao primo
	boolean[] isNotPrime = new boolean[n + 1];

	//só precisamos ir até a raiz quadrada de n
	for(int i = 2; i * i <= n; i++){
		if(!isNotPrime[i]){
			//começamos a marcar a partir de i*i
			//pq numeros multiplos menores ja foram marcados
			//por primos menores
			for(int j = i*i; j <= n; j += i){
				isNotPrime[j] = true;
			}
		}
	}
	return isNotPrime;
}
```
Esse algoritmo retorna uma lista de quais numeros são primos, cujo o indice representa o numero n. É uma lista pré-processada então se for fazer muitas iterações para numeros individuais, recomendo mais fazer a função  simples mesmo. Como abaixo:
```java
public boolean isPrime(int n){
	if(n < 2) return false;
	if(n == 2 || n == 3) return true;
	if(n%2 == 0 || n%3 == 0) return true;
	for(int i = 5; i*i <= n; i++){
		if(n%i ==0 || n%(i+2) == 0) return false;
	}
	return true;
}
```
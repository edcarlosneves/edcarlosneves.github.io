---
layout: post
title:  "Criando Exceções Customizadas em Python"
date:   2025-09-16 00:00:00 -0300
categories: posts python exceptions
---
Este tutorial irá guiá-lo através do processo de criação de suas próprias exceções em Python, tornando seu código mais claro, legível e fácil de depurar.

#### **Por que criar exceções customizadas?**

Às vezes, as exceções padrão do Python, como `ValueError` ou `TypeError`, não são específicas o suficiente para descrever um erro em seu programa. Criar suas próprias exceções permite que você forneça mensagens de erro mais detalhadas e específicas para o seu código.

**Exemplo:** A função `math.sqrt()` do Python não aceita números negativos e levanta um `ValueError`. Embora isso nos diga que há um problema com o valor, uma exceção customizada poderia nos dar uma mensagem mais específica como "Erro: A função de raiz quadrada não aceita números negativos."


#### **1. Identificando a Classe Base para Exceções**

Antes de criar uma exceção customizada, é útil entender a hierarquia de exceções do Python. A classe base para a maioria das exceções é a `Exception`. Você pode inspecionar a hierarquia de uma exceção existente, como `ValueError`, para ver de onde ela herda.


#### **2. Criando uma Exceção Customizada Simples**

Vamos começar criando uma exceção simples que herda da classe `Exception`.

```python
class NaoPodeSerNegativo(Exception):
    pass
```

Isso é tudo\! Agora você tem uma nova classe de exceção chamada `NaoPodeSerNegativo`.


#### **3. Implementando a Exceção no Código**

Agora, vamos usar nossa nova exceção em uma função que calcula a raiz quadrada.

```python
import math

class NaoPodeSerNegativo(Exception):
    pass

def raiz_quadrada(numero):
    if numero < 0:
        raise NaoPodeSerNegativo("Erro: O número não pode ser negativo.")
    return math.sqrt(numero)

# Exemplo de uso
try:
    print(raiz_quadrada(25))
    print(raiz_quadrada(-4))
except NaoPodeSerNegativo as e:
    print(e)
```

**Saída:**

```
5.0
Erro: O número não pode ser negativo.
```

Como você pode ver, agora recebemos uma mensagem de erro muito mais descritiva.


#### **4. Aprimorando a Exceção com Métodos Especiais**

Podemos tornar nossa exceção ainda mais informativa sobrescrevendo os métodos `__init__` e `__str__`.

  * `__init__`: Permite que você passe argumentos para sua exceção quando ela for levantada.
  * `__str__`: Controla a mensagem que é impressa quando a exceção é capturada.

<!-- end list -->

```python
import math

class NaoPodeSerNegativo(Exception):
    def __init__(self, texto, valor):
        super().__init__(texto)
        self.valor = valor

    def __str__(self):
        return f'{super().__str__()} O valor que causou o erro foi: {self.valor}'

def raiz_quadrada(numero):
    if numero < 0:
        raise NaoPodeSerNegativo("Erro: O número não pode ser negativo.", numero)
    return math.sqrt(numero)

# Exemplo de uso
try:
    print(raiz_quadrada(25))
    print(raiz_quadrada(-9))
except NaoPodeSerNegativo as e:
    print(e)
```

**Saída:**

```
5.0
Erro: O número não pode ser negativo. O valor que causou o erro foi: -9
```

Agora, nossa mensagem de erro não apenas nos diz qual é o problema, mas também o valor que o causou.


#### **Conclusão**

Criar exceções customizadas é uma ótima maneira de melhorar a clareza e a robustez do seu código Python. Ao fornecer mensagens de erro específicas e detalhadas, você torna seu código mais fácil de usar e depurar para você e para outros desenvolvedores.
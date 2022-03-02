# Trabalho realizado na Semana #8 e #9

## Task 1
Depois de seguir os passos que nos foram dados, para imprimir todas as informações da funcionária Alice presentes na tabela `credential`, apenas temos de correr o seguinte comando `SELECT * FROM credential WHERE EID=10000`. Este comando vai selecionar todos os atributos de uma linha da tabela `credential` onde o atributo `EID` é `10000` (_Employee ID_ da funcionária Alice). 

## Task 2.1
Após uma análise ao código php da aplicação web em questão, percebemos que este utiliza diretamente um _input_ externo para efetuar uma _dynamic sql query_ à base de dados. Deste modo, para conseguirmos efetuar o _login_ como `admin` (considerando que não sabemos qual é que é a _password_ correspondente) teremos de nos aproveitar de uma vulnerabilidade presente no código. Como a _query_ é feita da seguinte forma:
```php
$input_uname = $_GET[’username’];
$input_pwd = $_GET[’Password’];
$hashed_pwd = sha1($input_pwd);
...
$sql = "SELECT id, name, eid, salary, birth, ssn, address, email, nickname, Password
        FROM credential
        WHERE name= ’$input_uname’ and Password=’$hashed_pwd’";
$result = $conn -> query($sql);
```
e como controlamos os _inputs_, o que temos de fazer é omitir a verificação da _password_ através de `SQL injection`. Assim, e como os comentários em SQL são iniciados por `-- `, apenas teremos de passar para o username o valor `Admin'-- ` e a _query_ efetuada pelo php vai selecionar os atributos em questão de uma linha da tabela `credential` que possua um username `Admin` sendo comentada, e por isso não executada, o resto da verificação.
Deste modo, vamos conseguir obter o `id` do `Admin` para que no momento da verficação do _login_, este valor não esteja a `null` e o _login_ seja efetuado com sucesso.

## Task 2.2
O objetivo desta tarefa passa por obter os mesmos resultados da tarefa anterior, ou seja, efetuar o _login_ como `admin` sem qualquer conhecimento da _password_ do mesmo, mas desta vez sem acesso à _webpage_. Deste modo, teremos de utilizar o comando `curl` (_Client URL_) da seguinte forma:
```
 curl 'www.seed-server.com/unsafe_home.php?username=Admin%27--%20'
```
passando o valor `Admin%27--%20` (sendo %27 e %20 os `ASCII codes` correspondentes a `'` e ao `space`) para que conseguissemos excluir a verificação da _password_ no momento da autenticação (seguindo o pensamento da tarefa anterior).

## Task 2.3
Inicialmente, na autenticação, tentamos (porém sem sucesso, pois era sempre apresentada uma mensagem de erro) fazer uso do `;` (utilizado em SQL para separar _queries_) para conseguirmos manipular a aplicação _web_ a executar duas _queries_ no momento do pedido à base de dados.
De seguida, procuramos também executar o mesmo procedimento nos _inputs_ de edição de perfil, porém, e mais uma vez, sem sucesso (apesar de que nestas tentativas não nos era apresentada qualquer mensagem de erro, apenas não era verificada qualquer alteração nos dados).
Deste modo, concluímos que uma medida para evitar ataques do tipo `SQL injection` passa por não executar possíveis _dynamic queries_ que sejam na realidade, mais do que uma _query_ evitando, assim, possíveis ataques que possam ocorrer devido ao controlo de um _input_ por parte de um atacante.
Após pesquisa, concluimos que existe uma contramedida na função `mysqli::query` que deteta o uso do caracter `;` de forma a evitar este tipo de ataque. Para correr várias _queries_ teria que estar a ser utilizado `mysqli::multi_query`.

## Task 3.1
É possível entender que o que é realizado na `Edit Profile Page`, através da análise do código, é um update na table `credential`, mas apenas de algumas colunas, sendo que `salary` não é uma delas.

```php
$hashed_pwd = sha1($input_pwd);
$sql = "UPDATE credential SET
nickname='$input_nickname',
email=’$input_email',
address=’$input_address',
Password=’$hashed_pwd',
PhoneNumber=’$input_phonenumber'
WHERE ID=$id;";
$conn->query($sql);
```
Para entrar na `Edit Profile Page` da alice basta fazer o login da forma descrita nos pontos anteriores mas com `username = 'Alice'`. Para alterar a coluna `salary` será necessário utilizar `SQL Injection` aproveitando o facto de a query ser gerada dinamicamente. Se no campo `nickname` passarmos um nome com plica no fim, o que acontece é que depois da plica podemos adicionar aquilo que desejamos de `SQL` que a query irá correr normalmente, para completar a tarefa seria necessário colocar no campo de  `nickname` a string `alice', salary = '999999` que transformaria a query em:
```php
$sql = "UPDATE credential SET
nickname='alice', salary = '999999',
email=’$input_email',
address=’$input_address',
Password=’$hashed_pwd',
PhoneNumber=’$input_phonenumber'
WHERE ID=$id;";
``` 
Assim a coluna `salary` da Alice passaria a ter um valor de 999999.

## Task 3.2
Para realizar esta tarefa, será necessário combinar os ataques da tarefas _2.1_ e _3.1_.Primeiramente será necessário entrar na conta do `Boby` utilizando o raciocínio da tarefa _2.1_ com a string `Boby'-- ` e seguidamente realizar o mesmo processo da tarefa _3.1_ utilizando a string `alice', salary = '1`.


## CTF

### Desafio 1
À semelhança da task 2 realizada na aula, foi realizada uma `SQL injection`. De maneira a conseguirmos fazer login como admin, sem saber a password, colocamos como username `admin' -- ` . Este input explora a vulnerabilidade da query `$query = "SELECT username FROM user WHERE username = '".$username."' AND password = '".$password."'";` tornando-a `$query = "SELECT username FROM user WHERE username = 'admin'`. A parte da query que incide sobre a password fica comentada e o login é efetuado com sucesso, sem conhecimento da password.

### Desafio 2
Verificamos que a funcionalidade `Ping a Host` está disponível a qualquer utilizador mesmo que não esteja autenticado. Após testarmos com alguns valores, pe. 0, reparamos que essa funcionalidade faz uso do utilitário de linux `ping` e imprime no ecrã o resultado. Este website está, portanto, suscetível a `Command Injection`, em que a ideia de base é correr mais utilitários linux do que aqueles inicialmente previstos; para isso faz-se uso de `&&` que possibilita correr mais do que um comando. Neste caso concreto a maneira de chegar à flag seria com a string `0 && cat /flag.txt`, ou seja o programa realiza o `ping` e seguidamente o `cat`, imprimindo o conteúdo do ficheiro `flag.txt`. 

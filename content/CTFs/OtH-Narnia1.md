---
layout: default
---

## OvertheWire: Narnia0

Hola a todos!

En los siguientes post les dejaré mi write up sobre los 4 primeros desafíos del reto [Narnia](https://overthewire.org/wargames/narnia/) de la plataforma OverTheWire

Bien, cada rato cuenta con un binario SetUID y el código fuente en C (he dejado el código fuente de los binarios abajo), por lo que podemos tener una idea de como se compone el desafío

Para resolver los desafíos hay que comprender cómo funciona el stack y los conceptos de [Buffers Overflows](../Exploiting/post1.md)

### Empezando(Desafío 0)

Lo primero que haremos será conectarnos a la máquina remota mediante SSH con las credenciales que nos brinda la misma página:
```
user: narnia0   
pass: narnia0
puerto: 2226
```
`ssh narnia0@narnia.labs.overthewire.org -p 2226`

Ahora nos dirigimos al directorio /narnia/ y buscamos el código fuente y el binario, aquí está el código fuente:

```c
#include <stdlib.h>

int main(){
    long val=0x41414141;  /*Puts the value we want to overwrite on the stack*/
    char buf[20];         /*Sets the buffer length to 20 bytes*/

    printf("Correct val's value from 0x41414141 -> 0xdeadbeef!\n");
    printf("Here is your chance: ");
    scanf("%24s",&buf); /*Reads input into buffer (24 char limit on buffer, which is enough to fill the buffer and then the 4 bytes for deadbeef)*/

    printf("buf: %s\n",buf); /*Prints contents of buffer*/
    printf("val: 0x%08x\n",val); /*Outputs value we want to overwrite*/

    if(val==0xdeadbeef){ /*If value == 0xdeadbeef*/
        setreuid(geteuid(),geteuid()); /*Make the binary use the SUID and GUID*/
        system("/bin/sh"); /*Run /bin/sh to spawn a shell*/
    }
    else { /*If the value isn't 0xdeadbeef then*/
        printf("WAY OFF!!!!\n"); /*Print "WAY OFF!!!!" and then exit*/
        exit(1);
    }

    return 0;
}
```

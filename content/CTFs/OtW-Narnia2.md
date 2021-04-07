---
layout: default
---

## OverTheWire: Narnia2

En este desafío entraremos en un binxp "real" con un payload extremadamente basico que llena ESP con NOPS(\x90), excepto por un shellcode al final, y luego sobrescribiendo la dirección ret para que sea el inicio de ESP.

### Empezando

Lo primero que haremos como siempre es conectarnos a la máquina por SSH con las credenciales del desafío anterior:

`ssh narnia2@narnia.labs.overthewire.org -p 2226`

Aquí está el código fuente del binario:

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

int main(int argc, char * argv[]){
    char buf[128]; /*Declares the buffer length to be 128 bytes*/

    if(argc == 1){
        printf("Usage: %s argument\n", argv[0]); /*Display usage*/
        exit(1);
    }
    strcpy(buf,argv[1]); /*Copy contents of arg 1 to buffer*/
    printf("%s", buf); /*Print the buffer*/

    return 0;
}
```

Entonces, podemos comenzar obteniendo el offset de falla (que será alrededor de 128, ya que este es el tamaño del buffer, aunque si hay algo entre el final del buffer y el inicio de la dirección ret en la pila, entonces necesitaremos para jugar con la ubicación de la dirección ret en el payload), para esto hice un bucle de bash realmente simple que aumenta lentamente el relleno.

```bash
for i in $(seq 1 300); do echo $i; ./narnia2 $(python -c 'print "A"*'$i';'); done
```

Todo lo que hace son bucles de 1 a 300 y repite el número cada vez, pero también imprime "A" esa cantidad de veces mientras la pasa como argumento al binario, de modo que podamos seguir los números hasta encontrar Segmentation fault.

```
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA132
Segmentation fault
```

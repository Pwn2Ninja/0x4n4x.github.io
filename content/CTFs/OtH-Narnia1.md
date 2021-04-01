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



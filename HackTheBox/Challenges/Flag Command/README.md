>Al comenzar el challenge podemos ver una especie de historia con una línea de comandos debajo:

![image](https://github.com/user-attachments/assets/518a3402-1427-4e3b-888d-e5d5e117f112)

>Tras probar diversas inyecciones de comandos no funciona por lo que vamos a analizar en el debugger del navegador los archivos .js que están interviniendo.

>Aquí podemos ver que en el archivo main.js hay un condicional que dice que si el mensaje contiene 'HTB{' el usuario ha ganado:

![image](https://github.com/user-attachments/assets/05dba1e9-e90a-44dd-94a3-bfc3a1a3377f)

>Si seguimos investigando, en las opciones de red, podemos ver algo interesante que nos muestra todas las opciones disponibles que la página puede interpretar, destacando una opción secreta:

![image](https://github.com/user-attachments/assets/45c20845-8aa9-423a-a6ab-c1e23d453b02)

>Si empleamos el contenido de la opción secreta en la línea de comandos tras haber iniciado el juego con 'start' habremos pasado el challenge:

![image](https://github.com/user-attachments/assets/031759d6-496c-4f0d-a1e4-0c3cdb800057)


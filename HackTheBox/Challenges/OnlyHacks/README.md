>Al entrar a la página asignada vemos un panel de login, vamos a registrarnos:

![image](https://github.com/user-attachments/assets/b339c348-d36c-4379-ae00-05480e46c88b)

>Por ahora nos registraremos de forma normal, posteriormente podríamos probar si alguno de estos inputs es vulnerable a XSS o arbitrary file upload:

![image](https://github.com/user-attachments/assets/05a8ace2-06ea-4873-be7f-d394dbd68296)

>Una vez registrados parece que estamos en una típica página de citas:

![image](https://github.com/user-attachments/assets/0d6272a3-5834-4319-81e8-e92bb44d7d85)

>En el apartado matches podemos ver que al enviar un mensaje con un payload de XSS este se interpreta pero una vez recargues el chat deja de funcionar:

![image](https://github.com/user-attachments/assets/4590ef3f-8d62-443b-9cb2-c8f23b2ff128)

>En la url podemos darnos cuenta que hay un query param que apunta a un número, tras ir probando diversos números he logrado identificar una vulnerabilidad IDOR (Insecure Direct Object Reference) que permite al establecer el valor del query param rid a 3 ver la flag:

![image](https://github.com/user-attachments/assets/6e6c3530-4f07-402f-8b60-954f48bcd911)


# SQLinyection

SQL (lenguaje de consulta estructurado) es un lenguaje rico en funciones que se utiliza para consultar bases de datos; estas consultas SQL se conocen mejor como declaraciones.

El más simple de los comandos que cubriremos en esta tarea se usa para recuperar (seleccionar), actualizar, insertar y eliminar datos. Aunque algo similar, algunos servidores de bases de datos tienen su propia sintaxis y pequeños cambios en el funcionamiento de las cosas. Todos estos ejemplos se basan en una base de datos MySQL. Después de aprender las lecciones, podrá buscar fácilmente una sintaxis alternativa en línea para los diferentes servidores. Vale la pena señalar que la sintaxis SQL no distingue entre mayúsculas y minúsculas.

### Select
El primer tipo de consulta que aprenderemos es la consulta SELECT utilizada para recuperar datos de la base de datos. 
```
select * from users;
```
```
select username,password from users;
```
```
select * from users LIMIT 1;
```
La siguiente consulta, como la primera, devuelve todas las columnas usando el selector * y luego la cláusula "LIMIT 1" obliga a la base de datos a devolver solo una fila de datos. Cambiar la consulta a "LIMIT 1,1" obliga a la consulta a omitir el primer resultado, y luego "LIMIT 2,1" omite los dos primeros resultados, y así sucesivamente. Debe recordar que el primer número le dice a la base de datos cuántos resultados desea omitir, y el segundo número le dice a la base de datos cuántas filas devolver.


Por último, vamos a utilizar la cláusula where; así es como podemos seleccionar con precisión los datos exactos que necesitamos devolviendo datos que coincidan con nuestras cláusulas específicas:
```
select * from users where username='admin';
```
![General] (https://github.com/KZIJoseIgnacio/SQLinyection/blob/main/Captura%20de%20pantalla%202022-02-06%20183556.png)


### Select

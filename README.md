# Resumen para el examen final de Base de Datos

## Índice
1. [Vistas](#1-Vistas)
3. [Transacciones](#2-Transacciones)
4. [Funciones](#3-Funciones)
5. [Cursores](#4-Cursores)
6. [Procedimientos Almacenados](#5-Procedimientos-Almacenados)

# 1. Vistas

Una vista en SQL Server es una consulta almacenada que presenta datos de una o varias tablas como si fuera una tabla virtual. No guarda los datos, sino la consulta, por lo que cada vez que se accede a la vista, esta ejecuta la consulta para mostrar la información actualizada. Las vistas simplifican el acceso a datos complejos y permiten realizar operaciones de lectura y, en ciertos casos, de escritura sobre las tablas subyacentes.

Las vistas se usan para facilitar consultas frecuentes, restringir el acceso a datos sensibles sin exponer tablas completas, y establecer una representación consistente de los datos. También pueden mejorar el rendimiento si se convierten en vistas indexadas, que almacenan parcialmente los resultados para consultas rápidas.

## Ejemplo 1

### Enunciado

Crear una vista llamada AuthorPubUSA en la base de datos pubs que muestre información de las publicaciones de editores ubicados en los Estados Unidos. La vista debe incluir el nombre de la publicación, la ciudad y el estado del editor, así como el título y el precio del libro. Esta vista debe filtrar únicamente las publicaciones de editores cuyo país sea "USA".

### SQL:

```sql
USE pubs;
GO

-- Eliminar la vista si ya existe
IF OBJECT_ID('dbo.AuthorPubUSA', 'V') IS NOT NULL
    DROP VIEW dbo.AuthorPubUSA;
GO

-- Crear la vista
CREATE VIEW dbo.AuthorPubUSA AS
SELECT 
    p.pub_name AS Nombre_Publicacion,
    p.city AS Ciudad,
    p.state AS Estado,
    t.title AS Titulo_Libro,
    t.price AS Precio
FROM 
    dbo.publishers p
INNER JOIN 
    dbo.titles t ON p.pub_id = t.pub_id
WHERE 
    p.country = 'USA';
GO

-- Confirmar la creación de la vista
PRINT 'Vista AuthorPubUSA creada correctamente.';
GO

-- Probar la vista
BEGIN TRY
    SELECT * FROM dbo.AuthorPubUSA;
END TRY
BEGIN CATCH
    PRINT 'Error al seleccionar los datos de la vista AuthorPubUSA: ' + ERROR_MESSAGE();
END CATCH
GO

```
### Explicación:

- Contexto de la base de datos: Utiliza la base de datos pubs con USE pubs;.
- Eliminación de la vista existente: Si la vista AuthorPubUSA existe, se elimina para evitar conflictos al recrearla.
- Creación de la vista: Define la vista AuthorPubUSA que muestra información sobre publicaciones y títulos de editores en USA.
- Selección de datos: Incluye el nombre de la publicación, ciudad, estado, título del libro y precio.
- Unión de tablas: Realiza un INNER JOIN entre publishers y titles usando el pub_id común a ambas tablas.
- Condición de filtrado: Utiliza WHERE p.country = 'USA' para filtrar los editores que están en Estados Unidos.
- Confirmación de creación y prueba de la vista: Muestra un mensaje de confirmación y luego intenta seleccionar datos de la vista, manejando cualquier error que pueda ocurrir.


## Ejemplo 2

### Enunciado
Crear una vista llamada infoClien en la base de datos Datos que muestre la información de los clientes y sus saldos para aquellos que residen en el distrito de Nicoya. La vista debe incluir el nombre, primer apellido, segundo apellido del cliente, el saldo actual, y el distrito en el que se encuentran. Esta vista debe filtrar exclusivamente los clientes que pertenecen al distrito especificado, utilizando la relación entre las tablas saldos, clientes y rutas.

### SQL:

```sql
USE Datos;
GO

-- Eliminar la vista si ya existe
IF OBJECT_ID('dbo.infoClien', 'V') IS NOT NULL
    DROP VIEW dbo.infoClien;
GO

-- Crear la vista
CREATE VIEW infoClien AS
SELECT 
    C.NNOMBRE AS Nombre,
    C.IAPELLI AS Apellido1,
    C.IIAPELL AS Apellido2,
    S.SAMONTO AS Saldo,
    R.DISTRITO AS Distrito
FROM 
    saldos S
JOIN 
    clientes C ON S.NCEDULA = C.NCEDULA
JOIN 
    rutas R ON R.CODRUTA = C.CODRUTA
WHERE 
    R.DISTRITO = 'Nicoya';
GO

-- Confirmar que la vista fue creada
IF OBJECT_ID('dbo.infoClien', 'V') IS NOT NULL
    PRINT '<<< VISTA dbo.infoClien CREADA EXITOSAMENTE >>>';
ELSE
    PRINT '<<< ERROR AL CREAR LA VISTA dbo.infoClien >>>';
GO

```

### Explicación:

- Contexto de la base de datos: Se establece el uso de la base de datos Datos.
- Eliminación de la vista existente: Si la vista infoClien existe, se elimina para evitar conflictos al recrearla.
- Creación de la vista infoClien: La vista muestra un listado de clientes y sus saldos para aquellos que se encuentran en el distrito de Nicoya.
- Selección de datos: Incluye el nombre y apellidos del cliente, el saldo y el distrito.
- Uniones de tablas: Une las tablas saldos, clientes y rutas mediante las columnas NCEDULA y CODRUTA.
- Condición de filtrado: Utiliza WHERE R.DISTRITO = 'Nicoya' para filtrar los clientes que pertenecen al distrito de Nicoya.
- Confirmación y visualización: Verifica la creación exitosa de la vista y luego selecciona todos los registros para mostrarlos.


## 2. Transacciones

En SQL Server, una transacción es una secuencia de operaciones que se ejecutan como una unidad indivisible. Esto significa que todas las operaciones en la transacción deben completarse con éxito para que los cambios se guarden en la base de datos. Si alguna operación falla, la transacción se revierte y se restauran los datos al estado anterior, garantizando la consistencia y la integridad de los datos.

Las transacciones son esenciales para proteger los datos de errores o problemas de concurrencia cuando múltiples usuarios acceden y modifican la base de datos simultáneamente. Las propiedades ACID (Atomicidad, Consistencia, Aislamiento, Durabilidad) definen cómo las transacciones aseguran que cada cambio se ejecute correctamente o no se ejecute en absoluto, minimizando el riesgo de datos corruptos o inconsistentes.

## Ejemplo 1

### Enunciado

Cree un script SQL en la base de datos DATOSOFIS que copie la tabla Employees de Northwind como JuniorEmployees y luego elimine de esta todos los empleados con más de 5 años de antigüedad (calculada desde HireDate hasta la fecha actual). El script debe realizar esta operación en una transacción y verificar, después de la eliminación, si el número total de empleados en JuniorEmployees es menor a 4. Si es así, la transacción debe revertirse, dejando la tabla en su estado original. Use transacciones, etiquetas y control de flujo para manejar la eliminación y posible reversión.

### SQL:

```sql
USE DATOSOFIS
GO

IF OBJECT_ID('dbo.JuniorEmployees', 'U') IS NOT NULL
    DROP TABLE dbo.JuniorEmployees;
GO

BEGIN TRY
   BEGIN TRANSACTION CopiaJuniorEmployees
   -- Copiar la tabla Employees de Northwind a JuniorEmployees
   SELECT * INTO dbo.JuniorEmployees
   FROM Northwind.dbo.Employees

  -- Eliminar empleados con más de 5 años de antigüedad
   DELETE FROM dbo.JuniorEmployees
   WHERE  DATEDIFF(YEAR, HireDate, GETDATE()) >= 5;

   -- Verificar si el número total de empleados es menor a 4
   DECLARE @EmployeeCount INT 
   SELECT @EmployeeCount = COUNT(*) FROM dbo.JuniorEmployees


   IF @EmployeeCount < 4
   BEGIN 
     ROLLBACK TRANSACTION CopiaJuniorEmployees
	 PRINT 'Transacción revertida. La tabla JuniorEmployees ha sido restaurada a su estado original.'
	 PRINT 'Porque el numero total de registros son: ' + CAST(@EmployeeCount AS VARCHAR);
   END
   ELSE
   BEGIN 
     COMMIT TRANSACTION CopiaJuniorEmployees
	 PRINT 'Transacción completada.los registros ah sido eliminados de JuniorEmployees.';
	  PRINT 'Porque el numero total de registros son: ' + CAST(@EmployeeCount AS VARCHAR);
   END


END TRY
BEGIN CATCH
  ROLLBACK TRANSACTION CopiaJuniorEmployees
   PRINT 'Error en la transacción. La operaci�n ha sido revertida.';
    PRINT ERROR_MESSAGE();
END CATCH
GO
   ```

### Explicación:

- Contexto de la base de datos: Se utiliza la base de datos DATOSOFIS.
- Eliminación de la tabla existente: Si la tabla JuniorEmployees ya existe, se elimina para evitar conflictos al copiar
- Inicio de la transacción: Se inicia una transacción llamada CopiaJuniorEmployees para agrupar las operaciones.
- Copia de la tabla Employees: Se copia la tabla Employees de la base de datos Northwind a la tabla JuniorEmployees en la base de datos actual.
- Eliminación de empleados: Se eliminan los empleados de JuniorEmployees que tienen más de 5 años de antigüedad.
- Conteo de empleados: Se cuenta el número total de empleados en JuniorEmployees.
- Verificación del número de empleados: Si el número total de empleados es menor a 4, se revierte la transacción y se imprime un mensaje.
- Confirmación de la transacción: Si el número total de empleados es mayor o igual a 4, se confirma la transacción y se imprime un mensaje.
- Manejo de errores: En caso de error
- Finalización de la transacción: Se finaliza la transacción y se manejan los errores en caso de que ocurran.


## Ejemplo 2

### Enunciado

Cree un script SQL en la base de datos DATOS que copie la tabla Suppliers de Northwind y la nombre como InternationalSuppliers; luego, elimine de esta todos los proveedores ubicados en USA o Canada. Utilice una transacción que verifique, tras la eliminación, si el número de proveedores en InternationalSuppliers es menor o igual a 5; si es así, la transacción debe revertirse, restaurando la tabla a su estado inicial. Asegúrese de implementar transacciones, etiquetas y control de flujo para realizar la eliminación y posible reversión.

### SQL:

```sql
USE DATOS;
GO

IF OBJECT_ID('dbo.InternationalSuppliers', 'U') IS NOT NULL
    DROP TABLE dbo.InternationalSuppliers;
GO

BEGIN TRY
    -- Inicia la transacción
    BEGIN TRANSACTION CopiaProveedores;

    -- Crea una copia de la tabla Suppliers de Northwind
    SELECT *
    INTO dbo.InternationalSuppliers
    FROM Northwind.dbo.Suppliers;

    -- Elimina proveedores de USA y Canada
    DELETE FROM dbo.InternationalSuppliers
    WHERE Country IN ('USA', 'Canada');

    -- Cuenta el número de proveedores restantes
    DECLARE @ProveedorCount INT;
    SELECT @ProveedorCount = COUNT(*) FROM dbo.InternationalSuppliers;

    -- Condición para confirmar o revertir la transacción
    IF @ProveedorCount <= 5
    BEGIN
        -- Revertir la transacción
        ROLLBACK TRANSACTION CopiaProveedores;
        PRINT 'Transacción revertida. La tabla InternationalSuppliers ha sido restaurada a su estado original.';
    END
    ELSE
    BEGIN
        -- Confirmar la transacción
        COMMIT TRANSACTION CopiaProveedores;
        PRINT 'Transacción completada. Los proveedores de USA y Canada han sido eliminados de InternationalSuppliers.';
    END
END TRY
BEGIN CATCH
    -- Manejo de errores
    ROLLBACK TRANSACTION CopiaProveedores;
    PRINT 'Error en la transacción. La operación ha sido revertida.';
    PRINT ERROR_MESSAGE();
END CATCH;

GO
```

### Explicación:

- Contexto de la base de datos: Se utiliza la base de datos DATOS.
- Inicio de la transacción: Se inicia una transacción llamada CopiaProveedores para agrupar las operaciones.
- Copia de la tabla Suppliers: Se copia la tabla Suppliers de la base de datos Northwind a la tabla InternationalSuppliers en la base de datos actual.
- Eliminación de proveedores: Se eliminan los proveedores de InternationalSuppliers que están ubicados en USA o Canada.
- Conteo de proveedores: Se cuenta el número total de proveedores en InternationalSuppliers.
- Verificación del número de proveedores: Si el número total de proveedores es menor o igual a 5, se revierte la transacción y se imprime un mensaje.
- Confirmación de la transacción: Si el número total de proveedores es mayor a 5,
- Manejo de errores: En caso de error
- Finalización de la transacción: Se finaliza la transacción y se manejan los errores en caso de que ocurran.

## 3. Funciones

Las funciones en SQL Server son bloques de código reutilizables que realizan operaciones específicas y devuelven un valor. Pueden ser funciones definidas por el usuario (UDF) o funciones integradas proporcionadas por el sistema. Las funciones UDF permiten a los usuarios crear funciones personalizadas para realizar cálculos, manipulaciones de cadenas, conversiones de datos y otras tareas.

## Ejemplo 1

### Enunciado

Crear una función en la base de datos Northwind que calcule la edad de un cliente a partir de su fecha de nacimiento almacenada en la columna BirthDate de la tabla Customers. La función, llamada fn_CalcularEdad, debe recibir como parámetro la fecha de nacimiento (BirthDate) y devolver la edad en años. Utilice la función DATEDIFF para calcular la diferencia entre la fecha actual y la fecha de nacimiento.

### SQL:

```sql
USE Northwind;
GO

-- Eliminar la función si ya existe
IF OBJECT_ID('dbo.fn_CalcularEdad', 'FN') IS NOT NULL
BEGIN
    DROP FUNCTION dbo.fn_CalcularEdad;
END;
GO

-- Crear la función
BEGIN TRY
    CREATE FUNCTION dbo.fn_CalcularEdad (@BirthDate DATE)
    RETURNS INT
    AS
    BEGIN
        DECLARE @Edad INT;
        SET @Edad = DATEDIFF(YEAR, @BirthDate, GETDATE());
        
        -- Ajustar si el cumpleaños aún no ha ocurrido en el año actual
        IF (DATEADD(YEAR, @Edad, @BirthDate) > GETDATE())
            SET @Edad = @Edad - 1;
        
        RETURN @Edad;
    END;
    PRINT 'Función fn_CalcularEdad creada exitosamente.';
END TRY
BEGIN CATCH
    PRINT 'Error al crear la función fn_CalcularEdad: ' + ERROR_MESSAGE();
END CATCH;
GO

-- Probar la función
BEGIN TRY
    SELECT CustomerID, dbo.fn_CalcularEdad(BirthDate) AS Edad
    FROM Customers;
    PRINT 'Función fn_CalcularEdad ejecutada exitosamente.';
END TRY
BEGIN CATCH
    PRINT 'Error al ejecutar la función fn_CalcularEdad: ' + ERROR_MESSAGE();
END CATCH;
GO
```

### Explicación:

- Contexto de la base de datos: Se utiliza la base de datos Northwind.
- Eliminación de la función existente: Si la función fn_CalcularEdad ya existe, se elimina para evitar conflictos al recrearla.
- Creación de la función: Se crea la función fn_CalcularEdad que calcula la edad de un cliente a partir de su fecha de nacimiento.
- Parámetros y tipo de retorno: La función recibe un parámetro de tipo DATE (BirthDate) y devuelve un valor entero (INT) que representa la edad en años.
- Cálculo de la edad: Utiliza la función DATEDIFF para calcular la diferencia en años entre la fecha actual y la fecha de nacimiento.
- Ajuste de la edad: Verifica si el cumpleaños del cliente ya ha ocurrido en el año actual y ajusta la edad en consecuencia.
- Retorno de la edad: Devuelve el valor de la edad calculada.
- Confirmación de creación y manejo de errores: Muestra un mensaje de confirmación si la función se crea correctamente y maneja cualquier error que pueda ocurrir durante la creación.


## Ejemplo 2

### Enunciado

Crear una función en la base de datos Northwind que devuelva el nombre completo de un empleado en formato "Apellido, Nombre SegundoNombre". La función, llamada fn_FormatoNombreCompleto, debe recibir como parámetros el nombre (FirstName), segundo nombre (MiddleName), y apellido (LastName) y devolver una cadena formateada.


### SQL:

```sql
USE Northwind;
GO

-- Eliminar la función si ya existe
IF OBJECT_ID('dbo.fn_FormatoNombreCompleto', 'FN') IS NOT NULL
BEGIN
    DROP FUNCTION dbo.fn_FormatoNombreCompleto;
END;
GO

-- Crear la función
BEGIN TRY
    CREATE FUNCTION dbo.fn_FormatoNombreCompleto (
        @FirstName NVARCHAR(50),
        @MiddleName NVARCHAR(50),
        @LastName NVARCHAR(50)
    )
    RETURNS NVARCHAR(150)
    AS
    BEGIN
        DECLARE @NombreCompleto NVARCHAR(150);
        
        -- Formato "Apellido, Nombre SegundoNombre" (omite segundo nombre si es NULL)
        SET @NombreCompleto = @LastName + ', ' + @FirstName +
                             ISNULL(' ' + @MiddleName, '');
        
        RETURN @NombreCompleto;
    END;
    PRINT 'Función fn_FormatoNombreCompleto creada exitosamente.';
END TRY
BEGIN CATCH
    PRINT 'Error al crear la función fn_FormatoNombreCompleto: ' + ERROR_MESSAGE();
END CATCH;
GO

-- Probar la función
BEGIN TRY
    SELECT EmployeeID, dbo.fn_FormatoNombreCompleto(FirstName, NULL, LastName) AS NombreCompleto
    FROM Employees;
    PRINT 'Función fn_FormatoNombreCompleto ejecutada exitosamente.';
END TRY
BEGIN CATCH
    PRINT 'Error al ejecutar la función fn_FormatoNombreCompleto: ' + ERROR_MESSAGE();
END CATCH;
GO
```

### Explicación:

- Contexto de la base de datos: Se utiliza la base de datos Northwind.
- Eliminación de la función existente: Si la función fn_FormatoNombreCompleto ya existe, se elimina para evitar conflictos al recrearla.
- Creación de la función: Se crea la función fn_FormatoNombreCompleto que devuelve el nombre completo de un empleado en formato "Apellido, Nombre SegundoNombre".
- Parámetros y tipo de retorno: La función recibe tres parámetros de tipo NVARCHAR (FirstName, MiddleName, LastName) y devuelve una cadena formateada de hasta 150 caracteres.
- Formato del nombre completo: La función concatena el apellido, nombre y segundo nombre en el formato requerido, omitiendo el segundo nombre si es NULL.
- Retorno del nombre completo: Devuelve la cadena formateada con el nombre completo del empleado.
- Confirmación de creación y manejo de errores: Muestra un mensaje de confirmación si la función se crea correctamente y maneja cualquier error que pueda ocurrir durante la creación.
- Prueba de la función: Se selecciona el EmployeeID y el nombre completo de los empleados utilizando la función fn_FormatoNombreCompleto, y se muestra un mensaje de confirmación si la prueba se realiza con éxito.

## 4. Cursores

Los cursores en SQL Server son estructuras que permiten recorrer filas de un conjunto de resultados de forma secuencial. Aunque los cursores pueden ser menos eficientes que las consultas de conjunto de resultados, son útiles para realizar operaciones detalladas fila por fila, como actualizaciones, eliminaciones o cálculos complejos.

## Ejemplo 1

### Enunciado
Realice un cursor que resuelva lo siguiente: 

- Construya en su declaración de datos, una tabla con la información del cliente: cédula, fecha del contrato y código de la ruta, únicamente. Para aquellos contratos que son mayor o igual al 1 de octubre de 2023 y cuyo género sea femenino.
- Recorra el cursor y muestre:
- Los nombres y apellidos de las personas,
El distrito al que pertenecen
Monto o Saldo de la cuenta


### SQL:

```sql
-- USAR LA BASE PUBS
USE PUBS;
GO

-- CREAR UNA TABLA TEMPORAL CON INFORMACIÓN DE AUTORES QUE RESIDEN EN CALIFORNIA
SELECT
    AU_ID,
    AU_LNAME,
    STATE
INTO #AUTORESCA
FROM AUTHORS
WHERE STATE = 'CA';

-- DECLARAR EL CURSOR
DECLARE AUTORESCURSOR CURSOR FOR
SELECT
    A.au_fname,
    A.au_lname,
    A.city,
    COUNT(T.title_id) AS NUMLIBROS
FROM #AUTORESCA AC
INNER JOIN authors A ON AC.au_id = A.au_id
LEFT JOIN titleauthor TA ON A.au_id = TA.au_id
LEFT JOIN titles T ON TA.title_id = T.title_id
GROUP BY A.au_fname, A.au_lname, A.city;

-- DECLARAR VARIABLES PARA ALMACENAR LOS DATOS DEL CURSOR
DECLARE
    @NOMBRE VARCHAR(20),
    @APELLIDO VARCHAR(40),
    @CIUDAD VARCHAR(20),
    @NUMLIBROS INT;

-- ABRIR EL CURSOR
OPEN AUTORESCURSOR;

-- OBTENER EL PRIMER REGISTRO
FETCH NEXT FROM AUTORESCURSOR INTO @NOMBRE, @APELLIDO, @CIUDAD, @NUMLIBROS;

-- RECORRER EL CURSOR
WHILE @@FETCH_STATUS = 0
BEGIN
    PRINT 'AUTOR: ' + @NOMBRE + ' ' + @APELLIDO;
    PRINT 'CIUDAD: ' + @CIUDAD;
    PRINT 'NÚMERO DE LIBROS PUBLICADOS: ' + CAST(@NUMLIBROS AS VARCHAR(10));
    PRINT '-----------------------------------------';

    -- OBTENER EL SIGUIENTE REGISTRO
    FETCH NEXT FROM AUTORESCURSOR INTO @NOMBRE, @APELLIDO, @CIUDAD, @NUMLIBROS;
END

-- CERRAR Y LIBERAR EL CURSOR
CLOSE AUTORESCURSOR;
DEALLOCATE AUTORESCURSOR;

-- ELIMINAR LA TABLA TEMPORAL
DROP TABLE #AUTORESCA;
GO
```

### Explicación:

- Contexto de la base de datos: Se utiliza la base de datos PUBS.
- Creación de una tabla temporal: Se crea una tabla temporal #AUTORESCA con la información de los autores que residen en California.
- Declaración del cursor: Se declara un cursor AUTORESCURSOR que selecciona el nombre, apellido, ciudad y número de libros publicados de los autores en California.
- Declaración de variables: Se declaran variables para almacenar los datos del cursor.
- Apertura del cursor: Se abre el cursor AUTORESCURSOR.
- Obtención del primer registro: Se obtiene el primer registro del cursor y se almacena en las variables correspondientes.
- Recorrido del cursor: Se recorre el cursor mientras haya registros disponibles.
- Impresión de datos: Se imprime el nombre, apellido, ciudad y número de libros publicados de cada autor.
- Obtención del siguiente registro: Se obtiene el siguiente registro del cursor.
- Cierre y liberación del cursor: Se cierra y libera el cursor AUTORESCURSOR.
- Eliminación de la tabla temporal: Se elimina la tabla temporal #AUTORESCA al finalizar el script.

## Ejemplo 2

Realice un cursor que resuelva lo siguiente:

- Construya en su declaración de datos una tabla con la información del cliente: cédula, género y fecha de contrato, únicamente. Para aquellos clientes femeninos que viven en el cantón de 'Alajuela'.
- Recorra el cursor y muestre:
- Los nombres y apellidos de las personas.
- La provincia y distrito al que pertenecen.
- El saldo actual de su cuenta.

### SQL:

```sql
USE Datos
GO

DECLARE
    @cedula VARCHAR(9),
    @codruta VARCHAR(6),
    @genero VARCHAR(1),
    @fecha_Contrato DATE,
    @nombre VARCHAR(30),
    @Apellido1 VARCHAR(30),
    @Apellido2 VARCHAR(30)

-- Declaración del cursor para obtener los primeros 10 registros de la tabla CLIENTES
DECLARE Cursor_Datos CURSOR FOR 
SELECT TOP 10 
    NCEDULA,
    CODRUTA,
    IGENERO,
    FVECONT,
    NNOMBRE,
    IAPELLI,
    IIAPELL 
FROM CLIENTES

OPEN Cursor_Datos

-- Obtener el primer registro del cursor
FETCH NEXT FROM Cursor_Datos INTO 
    @cedula, 
    @codruta, 
    @genero, 
    @fecha_Contrato, 
    @nombre, 
    @Apellido1, 
    @Apellido2

-- Iterar mientras haya registros en el cursor
WHILE (@@FETCH_STATUS = 0)
BEGIN
    -- Imprimir los resultados de cada registro
    PRINT 'Nombre: ' + @nombre + 
          ' Apellido 1: ' + @Apellido1 + 
          ' Apellido 2: ' + @Apellido2 + 
          ' Cedula: ' + @cedula + 
          ' Genero: ' + @genero

    -- Obtener el siguiente registro del cursor
    FETCH NEXT FROM Cursor_Datos INTO 
        @cedula, 
        @codruta, 
        @genero, 
        @fecha_Contrato, 
        @nombre, 
        @Apellido1, 
        @Apellido2
END

-- Cerrar y liberar el cursor
CLOSE Cursor_Datos
DEALLOCATE Cursor_Datos
GO

```


### Explicación:

- Contexto de la base de datos: Se utiliza la base de datos Datos.
- Declaración de variables: Se declaran las variables necesarias para almacenar los datos del cursor.
- Declaración del cursor: Se declara un cursor Cursor_Datos que selecciona los primeros 10 registros de la tabla CLIENTES.
- Apertura del cursor: Se abre el cursor Cursor_Datos.
- Obtención del primer registro: Se obtiene el primer registro del cursor y se almacena en las variables correspondientes.
- Recorrido del cursor: Se recorre el cursor mientras haya registros disponibles.
- Impresión de datos: Se imprime el nombre, apellidos, cédula, género y fecha de contrato de cada cliente.
- Obtención del siguiente registro: Se obtiene el siguiente registro del cursor.
- Cierre y liberación del cursor: Se cierra y libera el cursor Cursor_Datos al finalizar el script.


## 5. Procedimientos Almacenados

Los procedimientos almacenados en SQL Server son bloques de código que se almacenan en la base de datos y se pueden ejecutar de forma independiente. Los procedimientos almacenados pueden aceptar parámetros de entrada, realizar operaciones complejas y devolver resultados o modificar datos en la base de datos.

## Ejemplo 1

### Enunciado

Cree un procedimiento almacenado llamado usp_CreateSimpleOrder en la base de datos Northwind. Este debe crear una nueva orden para un cliente específico con un producto en el detalle. Reciba como parámetros el CustomerID, ProductID y la Quantity del producto. Verifique que el cliente y el producto existan; si alguno no existe, muestre un error y termine la operación. Si ambos existen, inserte la orden en la tabla Orders y el detalle en Order Details. Utilice transacciones para asegurar la integridad y RAISEERROR para manejar errores.

### SQL:

```sql
USE Northwind
GO

IF OBJECT_ID('usp_CreateSimpleOrder','P') IS NOT NULL
    DROP PROCEDURE usp_CreateSimpleOrder
GO

CREATE PROCEDURE usp_CreateSimpleOrder
    @CustomerID NCHAR(5),
    @ProductID INT,
    @Quantity INT
AS
BEGIN
    DECLARE @OrderID INT;

    BEGIN TRY
        BEGIN TRANSACTION;

        -- Verificar si el cliente existe
        IF NOT EXISTS (SELECT 1 FROM Customers WHERE CustomerID = @CustomerID)
        BEGIN
            RAISERROR ('El cliente no existe.', 16, 1);
            RETURN;
        END

        -- Verificar si el producto existe
        IF NOT EXISTS (SELECT 1 FROM Products WHERE ProductID = @ProductID)
        BEGIN
            RAISERROR ('El producto no existe.', 16, 1);
            RETURN;
        END

        -- Insertar una nueva orden
        INSERT INTO Orders (CustomerID, OrderDate, RequiredDate, ShipVia, Freight)
        VALUES (@CustomerID, GETDATE(), DATEADD(day, 7, GETDATE()), 1, 10);

        -- Obtener el ID de la orden generada
        SET @OrderID = SCOPE_IDENTITY();

        -- Insertar el detalle de la orden
        INSERT INTO [Order Details] (OrderID, ProductID, UnitPrice, Quantity, Discount)
        SELECT @OrderID, @ProductID, UnitPrice, @Quantity, 0
        FROM Products
        WHERE ProductID = @ProductID;

        COMMIT TRANSACTION;
        PRINT 'La orden se ha creado exitosamente con OrderID: ' + CAST(@OrderID AS NVARCHAR(10));

    END TRY
    BEGIN CATCH
        ROLLBACK TRANSACTION;
        PRINT ERROR_MESSAGE()
    END CATCH;
END
GO
```


### Explicación:

- Contexto de la base de datos: Se utiliza la base de datos Northwind.
- Eliminación del procedimiento almacenado existente: Si el procedimiento almacenado usp_CreateSimpleOrder ya existe, se elimina para evitar conflictos al recrearlo.
- Creación del procedimiento almacenado: Se crea el procedimiento almacenado usp_CreateSimpleOrder con los parámetros @CustomerID, @ProductID y @Quantity.
- Inicio del bloque TRY: Se inicia un bloque TRY para manejar posibles errores durante la ejecución del procedimiento.
- Inicio de la transacción: Se inicia una transacción para agrupar las operaciones de inserción.
- Verificación de la existencia del cliente: Se verifica si el cliente con el CustomerID proporcionado existe en la tabla Customers.
- Verificación de la existencia del producto: Se verifica si el producto con el ProductID proporcionado existe en la tabla Products.
- Inserción de la orden: Se inserta una nueva orden en la tabla Orders con los datos del cliente y la fecha actual.
- Obtención del ID de la orden: Se obtiene el ID de la orden recién insertada.
- Inserción del detalle de la orden: Se inserta el detalle de la orden en la tabla Order Details con el ProductID, la cantidad y el precio del producto.
- Confirmación de la transacción: Se confirma la transacción si todas las operaciones se realizan con éxito.
- Manejo de errores: En caso de error durante la ejecución del procedimiento, se revierte la transacción y se imprime el mensaje de error.


## Ejemplo 2

### Enunciado

Cree un procedimiento almacenado llamado usp_UpdateProductStock en la base de datos Northwind que actualice la cantidad en stock de un producto. Reciba como parámetros el ProductID y el NewStock. Verifique que el producto exista; si no, genere un error. Si existe, actualice el campo UnitsInStock en la tabla Products con el nuevo valor. Utilice transacciones para asegurar la operación y RAISEERROR para manejar errores.

### SQL:

```sql
USE Northwind
GO

IF OBJECT_ID('usp_UpdateProductStock','P') IS NOT NULL
   DROP PROCEDURE usp_UpdateProductStock
GO

CREATE PROCEDURE usp_UpdateProductStock
    @ProductID INT,
    @NewStock INT
AS
BEGIN
    BEGIN TRY
        BEGIN TRANSACTION;

        -- Verificar si el producto existe
        IF NOT EXISTS (SELECT 1 FROM Products WHERE ProductID = @ProductID)
        BEGIN
            RAISERROR ('El producto especificado no existe.', 16, 1);
            RETURN;
        END

        -- Actualizar la cantidad en stock del producto
        UPDATE Products
        SET UnitsInStock = @NewStock
        WHERE ProductID = @ProductID;

        COMMIT TRANSACTION;
        PRINT 'El stock del producto se ha actualizado correctamente para ProductID: ' + CAST(@ProductID AS NVARCHAR(10));

    END TRY
    BEGIN CATCH
        ROLLBACK TRANSACTION;
        PRINT ERROR_MESSAGE();
    END CATCH;
END
GO
```

### Explicación:

- Contexto de la base de datos: Se utiliza la base de datos Northwind.
- Eliminación del procedimiento almacenado existente: Si el procedimiento almacenado usp_UpdateProductStock ya existe, se elimina para evitar conflictos al recrearlo.
- Creación del procedimiento almacenado: Se crea el procedimiento almacenado usp_UpdateProductStock con los parámetros @ProductID y @NewStock.
- Inicio del bloque TRY: Se inicia un bloque TRY para manejar posibles errores durante la ejecución del procedimiento.
- Inicio de la transacción: Se inicia una transacción para agrupar las operaciones de actualización.
- Verificación de la existencia del producto: Se verifica si el producto con el ProductID proporcionado existe en la tabla Products.
- Actualización de la cantidad en stock: Se actualiza el campo UnitsInStock del producto con el nuevo valor proporcionado.
- Confirmación de la transacción: Se confirma la transacción si la actualización se realiza con éxito.
- Manejo de errores: En caso de error durante la ejecución del procedimiento, se revierte la transacción y se imprime el mensaje de error.  




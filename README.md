# SistemaPP

SistemaPP es una aplicación de escritorio desarrollada en C# con Windows Forms y una base de datos SQL Server. Permite administrar usuarios, productos y movimientos de stock, además de consultar reportes y llevar un registro de auditoría de las operaciones realizadas.

Este repositorio contiene el código fuente del sistema y la documentación utilizada para el Trabajo Integrador Individual de Modelo y Sistemas de 7.º año. Para el trabajo se eligió como caso principal el registro de scrap, es decir, la baja de productos dañados o que ya no pueden utilizarse.

## Funcionalidades

### Gestión de usuarios

El sistema cuenta con un módulo de usuarios que permite:

- Iniciar sesión con usuario y contraseña.
- Trabajar con dos roles: administrador y usuario.
- Solicitar el cambio de la contraseña provisoria en el primer ingreso.
- Configurar preguntas de seguridad.
- Recuperar el acceso mediante un código enviado por correo electrónico.

### Catálogo de productos y stock

Desde este módulo se administran los productos y sus existencias. Sus principales funciones son:

- Registrar y consultar productos asociados a una marca y una categoría.
- Administrar información como precios de compra y venta, lote, vencimiento y niveles de stock.
- Registrar ingresos, egresos y ajustes de inventario.
- Dar de baja productos dañados o inservibles mediante el registro de scrap.
- Consultar el kardex, donde se conserva el historial de movimientos de cada producto.
- Generar reportes de stock, incluyendo productos bajo el punto de reposición, próximos a vencer y consultas por marca o categoría.
- Consultar la auditoría de los cambios realizados en el sistema.

## Tecnologías utilizadas

- C# y .NET 8.
- Windows Forms para la interfaz de escritorio.
- SQL Server como base de datos.
- Procedimientos almacenados para las operaciones sobre la base de datos.
- Microsoft.Data.SqlClient para la conexión desde C#.

## Organización del proyecto

El sistema está dividido en capas para separar la interfaz, las reglas del programa y el acceso a los datos.

| Carpeta o proyecto | Función |
| --- | --- |
| `WinFormsApp1` | Contiene los formularios de Windows Forms. El proyecto de interfaz se llama `CapaVista`. |
| `CapaLogica` | Contiene las validaciones y las reglas de funcionamiento. |
| `CapaDatos` | Se conecta con SQL Server y ejecuta los procedimientos almacenados. |
| `Entidades` | Define las clases utilizadas para representar los datos del sistema. |
| `Utilidades` | Reúne funciones auxiliares relacionadas con correo, cifrado y registro de errores. |

El archivo `SistemaPP.sln` permite abrir la solución completa desde Visual Studio.

## Instalación y configuración

### Requisitos

Para abrir y ejecutar el proyecto se necesita:

- Windows.
- Visual Studio con la carga de trabajo de desarrollo de escritorio con .NET.
- SDK de .NET 8.
- Una instancia de SQL Server y una herramienta para ejecutar scripts SQL, como SQL Server Management Studio.

### Pasos de instalación

1. Descargar el repositorio o clonarlo con Git:

   ```bash
   git clone https://github.com/matiasarenales2007-afk/trabajo-de-investigacion.git
   ```

2. Preparar la base de datos en SQL Server. El repositorio incluye estos scripts:
   - `SistemaUsuariosDB.sql`: tablas y procedimientos del módulo de usuarios.
   - `Grupo1_CatalogoStockAuditoria.sql`: estructuras y procedimientos del catálogo, stock, scrap y auditoría.
   - `Grupo1_Correcciones.sql`: correcciones de procedimientos y controles de integridad que se aplican después del script anterior.

   Los scripts del módulo de stock necesitan que existan las tablas de usuarios, categorías y marcas con las claves correspondientes. Antes de ejecutarlos en una base nueva hay que comprobar esas dependencias, ya que no todas las definiciones necesarias para una instalación desde cero están incluidas en el repositorio.

3. Configurar la conexión a SQL Server. En `WinFormsApp1` se incluye el archivo `appsettings.example.json`, que debe copiarse como `appsettings.json` y modificarse con el servidor y el nombre de la base de datos que se utilicen. El sistema busca este archivo junto al ejecutable.

   Ejemplo de estructura:

   ```json
   {
     "ConnectionStrings": {
       "SistemaUsuariosDB": "Server=localhost;Database=SistemaUsuariosDB;Trusted_Connection=True;TrustServerCertificate=True;"
     }
   }
   ```

4. Configurar los datos de correo necesarios para la recuperación de contraseñas. El proyecto incluye funcionalidades de configuración y envío de correo, pero no incorpora credenciales listas para usar.

5. Abrir `SistemaPP.sln` en Visual Studio, restaurar las dependencias y seleccionar `CapaVista` como proyecto de inicio.

6. Compilar y ejecutar la aplicación. Para iniciar sesión se necesita un usuario registrado y habilitado en la base de datos.

Los archivos de configuración con datos privados no deben subirse al repositorio. La instalación puede requerir ajustes según la instancia de SQL Server y la configuración del equipo.

## Registro de scrap

El scrap es el procedimiento utilizado para dar de baja unidades de un producto cuando están rotas, dañadas o ya no sirven. Además de descontar esas unidades del stock, el sistema calcula el costo de la pérdida y guarda un historial de la operación.

En el Trabajo Integrador se documenta este proceso como el caso de uso **CU-02: Registrar scrap de producto dañado**.

### Requisitos previos

- Haber iniciado sesión con un usuario activo de rol administrador.
- Si se trata del primer ingreso, haber cambiado la contraseña provisoria y completado las preguntas de seguridad.
- Tener el producto registrado y habilitado en el catálogo.
- Contar con stock suficiente para la cantidad que se desea dar de baja.
- Indicar el motivo de la baja.

### Procedimiento

1. Iniciar sesión en el sistema.
2. Desde el panel del administrador, abrir **Catálogo y Stock**.
3. Seleccionar la opción **Scrap** para entrar a la ventana **Registros de Scrap**.
4. Buscar y seleccionar el producto dañado.
5. Escribir la cantidad de unidades que se van a dar de baja. Debe ser un número entero mayor que cero.
6. Completar el motivo de la baja.
7. Presionar **Registrar Scrap**.
8. Aceptar el mensaje de confirmación y comprobar que la operación aparezca en el listado.
9. Revisar las fechas **Desde** y **Hasta** si el registro no aparece en el período seleccionado.

Cuando el registro termina correctamente, el sistema muestra el mensaje **"Scrap registrado correctamente."**. La cantidad y el motivo se limpian, el stock se actualiza y se muestra el costo valorizado en el listado y en el total del período.

### Validaciones y errores

Antes de registrar la baja se comprueba que se haya seleccionado un producto, que la cantidad sea válida y que el motivo no esté vacío. También se verifica que el producto exista, esté habilitado y tenga stock disponible.

| Situación | Respuesta del sistema |
| --- | --- |
| No se seleccionó un producto | Solicita seleccionar un producto. |
| La cantidad es cero, negativa, decimal o está vacía | Solicita ingresar un número entero mayor que cero. |
| No se escribió un motivo | Indica que se debe completar el motivo del scrap. |
| El producto está deshabilitado | Informa que el producto está dado de baja y no admite scrap. |
| No hay stock suficiente | Rechaza la baja e informa que no hay stock suficiente para esa cantidad. |
| Ocurre un error al guardar | Cancela la transacción para evitar registros incompletos. |

### Funcionamiento interno

Una vez superadas las validaciones, la base de datos calcula la pérdida de la siguiente manera:

`Costo valorizado = Cantidad dada de baja × Precio de compra`

Si la operación se confirma, se realizan estos cambios:

1. Se guarda el registro de scrap junto con el producto, la cantidad y el motivo.
2. Se descuenta la cantidad del stock actual.
3. Se crea un egreso en el kardex con origen Scrap.
4. Se generan dos registros de auditoría, correspondientes al scrap y al movimiento de stock.

El procedimiento almacenado realiza estos cambios dentro de una misma transacción. De esta forma, si ocurre un error durante el guardado, se revierte la operación y no quedan datos registrados parcialmente.

El código relacionado con esta funcionalidad se encuentra principalmente en `WinFormsApp1/frmScrap.cs`, `CapaLogica/CL_servicioStock.cs`, `CapaDatos/CD_scrapDatos.cs` y los procedimientos SQL de scrap.

## Autor y contexto

**Matías Arenales**  
Trabajo Integrador Individual — Modelo y Sistemas, 7.º año, escuela técnica.  
Profesor: Gian Franco Denaro.  
Fecha de entrega: 13/11/2026.

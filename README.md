# ReadM

Actualizado: November 8, 2022 4:45 PM
Creado: October 17, 2022 5:52 PM
Estado: Hecho

<div align="center">
    <h1 align="center">CargaAuto-Engine KDM</h1>
    <p align="center">
         <a href="https://www.notion.so/Carga-Auto-KDM-9a4f42b248744cda87b541d08c430434">Explora en Notion »</a>
	 <a href="https://documenter.getpostman.com/view/22970528/2s83znoLyv">Explora la API »</a>
         <br />
      </p>
</div>

<br />

# Tabla de Contenido
- [Descripción del proyecto](#descripción)
- [Funcionalidades](#funcionalidades)
- [Interacción entre aplicativos](#interacción-entre-aplicativos)
- [Instalación](#instalación)
  - [Pre Requisitos](#pre-requisitos)
  - [Pasos de instalación](#pasos-de-instalación)
- [Actualizaciones](#actualizaciones)
- [Notas](#notas)
- [Lenguaje](#lenguaje)
- [Categoría](#categoría)
- [Temas](#temas)
- [Licencia](#licencia)

## Descripción

El proyecto Carga Auto - Engine KDM se divide en 2 aplicativos los cuales son:

- Carga Auto KDM
- Engine KDM

Este aplicativo está construido con la finalidad de integrar el formato txt de ventas del cliente Holding KDM, mostrarlas en una nueva interfaz y poder emitirlas a Docele. También incluye algunas configuraciones pequeñas de receptores, sucursales y folios que el propio cliente realizará.

También está para soportar un modelo de negocio de recepción, el cual el carga auto recolecta los documentos recibidos del cliente holding y las resguarda en carpetas separadas por rut de abonado.

<p align="right">(<a href="#top">back to top</a>)</p>

## Funcionalidades

Las funcionalidades del aplicativo constituye de varios puntos:

| Función | Descripción |
| --- | --- |
| CargaAuto Task procesar TXT | Mediante un TASK, lee archivos txt que poseen un formato particular por ruta diferente de abonados. Para leer dichos archivos, el aplicativo primero se autentica en el AUT con user_pass que obtiene de su .properties y con el token de login rescata la lista de abonados asociados a dicho user autenticado. Con la lista de los abonados, el servicio comienza a buscar txt en las rutas correspondiente a los abonados activos en AUT y las envía a la API VENTA del ENGINE KDM utilizando la operación Crear Venta, si el proceso es correcto mueve los txt a otra ruta para no volverlas a tomar en su próximo recorrido. |
| ENGINE KDM | Recibe las ventas TXT entrantes en base 64, las lee, interpreta y registra en su base de datos. |
| Task Emisión Automática [init 20 secs / delay 10min] | Si un abonado posee en sus propiedades el valor de EMISION_AUTOMATICO como ACTIVO, entonces se buscan todas las ventas en estado pendiente para ser emitidos al DOL. |
| Task Consultar Estado | Realiza la consulta de todas las ventas que ya fueron enviados y aprobados por el DOL, para luego consultarlas al DOL para verificar si fueron aprobados o rechazados o por reparar o si aún siguen pendientes ante el SII. |
| Task enviar correos | Realiza la consulta de las ventas que ya poseen el estado aprobado o reparo, para así poder enviar el xml y pdf al correo del receptor. |
| Carga Auto TASK DTE Recibido | posee otro TASK el cual permite resguardar en carpetas los archivos XML/PDF de los documentos recibidos de cada Abonado del holding KDM. |

<p align="right">(<a href="#top">back to top</a>)</p>

## Interacción entre aplicativos

![image](./../../documentation/static/comunicacion.png)

| Interacción | Descripción |
| --- | --- |
| KDM.ERP → FileSystem ← CargaAutoKDM: | El servicio ERP realiza la generación de archivos TXT para luego dejarlas en una carpeta diferente según el rut de Abonado dentro del servidor. Las carpetas se dividen de la siguiente manera. Donde posteriormente el carga auto tiene una tarea repetitiva para buscar los archivos TXT dentro de las carpetas. También mencionar que el CargaAuto así como lee los txt de entrada para la emisión, también obtiene xml/pdf de los documentos recibidos de cada abonado y los respalda en la misma ruta por tipo de abonado. |
| CargaAutoKDM → AUT: | El servicio cargaAuto posee dentro de sus propiedades un usuario y password que sirve para autenticarse ante el AUT y poder obtener los abonados asociados al usuario del cargaAuto. El fin de obtener los abonados asociados, es para que el aplicativo verifique que carpetas de abonado leer si en caso a futuro se adicionan mas carpetas de abonados no registrados en nuestro servicio y evitar inconvenientes. |
| CargaAutoKDM → XPC: | Cada vez que se presente una excepción o error inesperada dentro del servicio CargaAutoKDM, este será reportado al servicio XPC. |
| CargaAutoKDM → DOCELE-KDM: | Cuando el CargaAutoKDM lee los txt de entrada en los FileSystem, el paso siguiente es proceso los txt de uno a uno, encondeandolos a Base64 y envíandolas al método crear de Venta Controller proporcionado por el servicio DOCELE-KDM. |
| DOCELE-KDM → KDM_BD: | Docele-KDM al recibir los txt de entrada enviados por el cargaAuto las comienza a validar y registrar en la tabla VENTA de su base de datos. Así mismo registra los datos del receptor que poseeía el TXT y si en caso la VENTA ya existía anteriormente registrada con el mismo folio pero aún seguía con ESTADO PENDIENTE, dicha venta es reemplazada por la nueva VENTA que llegue. |
| DOCELE-KDM → DOL: | El servicio DOCELE-KDM se comunica con el DOL para poder realizar la emisión electrónica de las ventas, también para consultar el estado SII de dichas ventas emitidas y obtener el PDF. Así mismo también utiliza el servicio de Recepción del DOL para obtener la lista de documentos recibidos, luego obtener el PDF y XML y finalmente dejar un estado de MARCA de cada documentos recibido. |
| DOCELE-KDM → SEM: | El servicio DOCELE-KDM se comunica con el SEM porque tiene dos maneras de enviar correos: (1)Por la interfaz gráfica del usuario y (2)Mediante el task de enviar correos automático. |
| DOCELE-KDM ← UI DOCELE-KDM: | El servicio DOCELE-KDM despliega muchos Controladores |

<p align="right">(<a href="#top">back to top</a>)</p>

## Instalación

### Pre requisitos

- JAVA 8 de preferencia JAVA CORRETO 8 de Amazon
- Instalador del servicio que puedes descargar desde [aquí](https://drive.google.com/drive/folders/1EIAwMyvffu1NUyRKFFZvEIohgLkM2Azh?usp=sharing).

<p align="right">(<a href="#top">back to top</a>)</p>

### Pasos de instalación

- Para instalar el carga auto en el servidor del cliente debes de seguir los siguientes pasos:
    
    ### 1. Descargar instalador:
    
    En la ruta entregada, encontrarás el comprimido del instalador CargaAuto. Con ello descargarás el comprimido y la llevarás a la ruta C:\\Progam Files\\Facele\\ y la descomprimes en dicha ruta. También deberás de colocar el JAR compilado del carga auto en la carpeta descompirmida.
    ![image](./../../documentation/static/install-1.png)
    
    ### 1.1. Archivo jar:
    
    El archivo jar la puedes obtenerlo compilando el proyecto desde eclipse o solicitando a desarrollo que genere el archivo jar del proyecto.
    
    ### 2. Configurar properties:
    
    Dentro de la carpeta deberás de configurar el archivo application.properties donde se definirán:
    
    - El proceso que el carga auto trabajará, para este proyecto los valores “KDMProceso” y “KDMTransformados”.
    ![image](./../../documentation/static/install-2.png)
    
    ### 3. Configurar KDM properties:
    
    El archivo KDM Properties debe ser colocado en la siguiente ruta: 
    `C:\Users\<nombreUsuario>\Facele\cargaAuto` 
    
    Los datos a llenar en el archivo [kdm.properties](http://kdm.properties):
    
    ![image](./../../documentation/static/install-3-1.png)
    
    Los datos como aut.url y kdm.url deben variar según sea demo o producción.
    
    ### 4. Instalar CargaAuto por CMD:
    
    Deberás abrir CMD como administrador e ingresar a la ruta de la carpeta cargaAuto. Dentro de la carpeta, ejecutamos el comando: ***Facele_Integrador.exe install***
    
    ![image](./../../documentation/static/install-3.png)
        
    Ahora procederemos a configurar el servicio; abriendo la ventana de servicios windows y la encontraremos con el nombre de Facele Integrador.
    
    ![image](./../../documentation/static/install-4.png)
    
    Clic derecho e ingresar a PROPIEDADES, donde nos dirigiremos a "iniciar sesion"
    
    ![image](./../../documentation/static/install-5.png)
    
    Aplicaremos los cambios y aceptaremos. Posterior iniciamos el servicio
    
    ![image](./../../documentation/static/install-6.png)
        
    Una vez iniciado el servicio, comenzará a correr los tasks de lectura y envio de archivos txt y obtención de xml/pdf de los documentos recibidos.
    

<p align="right">(<a href="#top">back to top</a>)</p>

## Actualizaciones

<p align="right">(<a href="#top">back to top</a>)</p>

## Notas

Algunos de los valores en las tablas dentro de la base de datos, solo se almacenan con códigos, a continuación se describen la relación de los códigos y su significado de las columnas de que tabla:

### T.Abonado:Column: ”estado”

| Valor | Descripción |
| --- | --- |
| 0 | HABILITADO |
| 1 | DESHABILITADO |

### T.Venta:Column: ”estado”

| Valor | Descripción |
| --- | --- |
| 0 | PENDIENTE |
| 1 | ENVIADO |
| 2 | APROBADO |
| 3 | REPARO |
| 4 | RECHAZADO |
| 5 | CORREO_ENVIADO |
| 6 | CORREO_INACTIVO |

### T.Venta:Column: ”tipo_documento”

| Valor | Descripción |
| --- | --- |
| 0 | DTE_33 |
| 1 | DTE_34 |
| 2 | DTE_43 |
| 3 | DTE_46 |
| 4 | DTE_61 |
| 5 | DTE_56 |
| 6 | DTE_52 |

### T.Folio:Column: ”estado”

| Valor | Descripción |
| --- | --- |
| 0 | HABILITADO |
| 1 | DESHABILITADO |

### T.Referencia:Column: ”codigo_referencia”

| Valor | Descripción |
| --- | --- |
| 0 | ANULA_DOCUMENTO |
| 1 | CORRIGE_TEXTO |
| 2 | CORRIGE_MONTO |

<p align="right">(<a href="#top">back to top</a>)</p>

## Lenguaje

![Java](https://img.shields.io/badge/java-%23ED8B00.svg?style=for-the-badge&logo=java&logoColor=white)
![MySQL5](https://img.shields.io/badge/mysql-%2300f.svg?style=for-the-badge&logo=mysql&logoColor=white)

## Categoría

`Engine`

## Temas

## Licencia

Distribuido por `Licencia de Facele`.

<p align="right">(<a href="#top">back to top</a>)</p>


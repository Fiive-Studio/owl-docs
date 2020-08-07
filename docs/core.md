# Owl

## 1. Introduccion

Owl nace a partir de la necesidad de crear una herramienta capaz de hacer transformaciones de datos comunes y que son repetitivas para el área sin que implique tener que recompilar o reescribir el objeto cada vez que haya cambios en estándares o se agreguen campos. 

Owl provee las clases con los tipos de datos comunes, pero a su vez unas interfaces para que se puedan implementar procesamiento de datos personalizados definiendo un formato XML al cual se deben ajustar para poder integrase a la herramienta.

Owl utiliza su propio lenguaje llamado XPML (eXtensible Owl Markup Language), este lenguaje se utiliza para configurar los Owl Output Config permitiendo realizar configuraciones más robustas y con más opciones, se basa en XML. El lenguaje fue introducido a partir de la versión 2.0 de Owl. 

Es importante mencionar que Owl es sensible a mayúsculas y minúsculas por lo cual usted deberá tener cuidado cuando realice configuraciones.

## 2. Estructura del Owl

Para poder trabajar con Owl se requiere lo siguiente: 

 - Owl Output Config: En este archivo se define la estructura de salida,
   con sus validaciones y mapeo del documento, este archivo se configura
   con la sintaxis XPML (explicada más adelante).
   
 - OwlSettings: Estructura que nos define las configuraciones del mapeo como: 
 o Ruta u objeto del Owl Output Config.
 o Ruta u objeto del Owl Output Config base. 
 o Objeto de referencias cruzadas. 
 o Separador de decimales.
 o Configuración de instancia.
  
  - IFormatInput: Define el tipo de entrada para el proceso de datos.
  
  - IFormatOutput: Define el tipo de salida para la generación de datos.
  
  - OwlHandler: Clase que se encarga de controlar todo el proceso de transformación de datos, para este proceso se crea un objeto del tipo de la clase pasándole el objeto OwlSettings y se invocan sus métodos LoadConfigMap, LoadInput y WriteOutput.

### 2.1 Owl Output Config. 

En este archivo es donde se define la estructura del archivo a generar con el mapeo y validaciones requeridas, el siguiente ejemplo muestra un archivo de configuración:

    <?xml version="1.0" encoding="utf-8" ?>
    <Configuracion Version="5.0">
     <Documento>
     <Salida Tipo="Plano">
     <Estructura Itera="EFACT_D98B_ORDERS">
     <Seccion Nombre="Encabezado" Separador=",">
     <Elemento Nombre="Id" Predeterminado="H" />
     <Elemento Nombre="Emisor" />
     <Elemento Nombre="Receptor" />
     </Seccion>
     <Seccion Nombre="Detalle" Itera="LINs">
     <Elemento Nombre="Id" Predeterminado="D" />
     <Elemento Nombre="Producto" />
     <Elemento Nombre="Cantidad" />
     </Seccion>
     </Estructura>
     </Salida>
     </Documento>
    </Configuracion>

### 2.2 OwlSettings
Esta estructura se utiliza para realizar la configuración de mapeo de Owl, en esta estructura se define el archivo Owl Output Config, y si aplica en Owl Output Config base, el DataSet de referencias, el separador de decimales que se usa a la entrada y a la salida, también podemos configurar si se requiere una instancia.

    OwlSettings settings = new OwlSettings
    {
     References = new DataSet(),
     PathConfig = @"D:\ruta\config_5.0.xml",
     PathConfigBase = @"D:\ruta\configbase.xml"
    };

### 2.3 IFormatInput.

Interfaz para las estructuras de entrada, define métodos para la carga y obtención de datos. Owl provee soporte para entradas basadas en XML mediante la clase XmlInput, pero cada desarrollador puede crear sus tipos de entradas personalizados.

### 2.4 IFormatOutput.

Interfaz para las estructuras de salida, define métodos para la generación de datos. Owl provee soporte para salidas de tipo Archivo Plano, EDI, ANSI, XML, Json y SQL mediante las clases FlatFileOutput, EDIOutput, ANSIOutput, XmlOutput, JsonOutput y SQLOutput respectivamente, pero cada desarrollador puede crear sus tipos de salidas personalizados.

### 2.5 OwlHandler.

Clase que se encarga de controlar todo el proceso de transformación de datos, para este proceso se crea un objeto del tipo de la clase pasándole el objeto de PGSettings y se invoican sus métodos LoadConfigMap, LoadInput y WriteOutput para iniciar el proceso de transformación:

    Owl = new OwlHandler(settings); Owl.LoadConfigMap(); Owl.LoadInput(new XmlInput(@"D:\ruta\20266352337-01-F001-00000110.xml")); foreach (GenericOutputValue value in Owl.WriteOutput(new FlatFileOutput())) { // Proceso }

Adicional si se requiere generar una instancia (archivo de muestra) de la configuración se puede realizar asignando la propiedad Instance igual a true en el objeto OwlSettings, al asignar dicho valor indicamos que solo se va a generar una instancia, por lo tanto Owl no realiza validaciones, no genera eventos y solo se van a tomar en cuenta las palabras clave Predeterminado, Variable y Reservada para los valores de los elementos; los demás valores van a ser el Nombre que se configure al elemento.

|MUNDO REAL: Generar una instancia sirve para ver una muestra del documento que genera la configuración sin necesidad de tener un documento de entrada, archivos de referencia, externos, etc.|  |
|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--|
|                                                                                                                                                                                             

## 3. XPML

El XPML (eXtensible Owl Markup Language / Lenguaje Extensible de Marcado para Owl) es un lenguaje de Owl para configurar los Owl Output Config permitiendo realizar configuraciones más robustas y con más opciones, se basa en XML. El lenguaje fue introducido a partir de la versión 2.0 de Owl.

La sintaxis de XPML se utiliza para configurar las propiedades de un determinado objeto y se puede realizar de la siguiente forma:

|**TIPO**|**EJEMPLO**  |
|--|--|
|Atributo: Se coloca la propiedad como atributo Xml y se asigna directamente el valor de la propiedad  | < Elemento Longitud=”12” /> |
|Atributo con Inline XPML: Se coloca la propiedad como atributo Xml y el valor de la propiedad con una palabra clave seguida de dos puntos y luego el valor, las palabras claves soportadas son: - Predeterminado - Variable - Reservada - Buscar|< Elemento Longitud=”Variable:LongitudConfig” /> < Elemento Descripcion=”Buscar:val/@val” />|
|Etiqueta: Se coloca una subetiqueda con el nombre del objeto, punto “.”, y el nombre de la propiedad, para asignar el valor se utiliza una palabra clave|< Elemento > <Elemento.Longitud>  < Palabra Clave>  </Elemento.Longitud> </ Elemento>|
|Listas: Cuando una propiedad del objeto sea una lista, se puede configurar como atributo enviando los valores separados por coma “,” o se puede configurar como etiqueta creando por cada valor una etiqueta nueva con la estructura (nombre objeto, punto “.”, propiedad, punto “.”, Valor (texto fijo)). Cuando se envíe como atributo con los valores separados por coma, cada valor se puede configurar con Inline XPML| Atributo:<Referencia CamposObtener="Campo1,Campo2"Atributo con Inline XPML: <Referencia CamposObtener="Buscar:val/@val,Campo2" Etiqueta: < Referencia >  <Referencia.CamposObtener>  <Referencia.CamposObtener.Valor>  < Palabra Clave>  </Referencia.CamposObtener.Valor>  <Referencia.CamposObtener.Valor>  < Palabra Clave>  </Referencia.CamposObtener.Valor>  </Referencia.CamposObtener> </ Referencia> |

Dentro de la configuración es posible combinar diferentes propiedades como atributo y como etiqueta, en caso de que la misma propiedad se configure como atributo y etiqueta solo se tendrá en cuenta el atributo. 

**Nota: Algunos objetos tienen restricciones en la configuración XPML y solo permiten configurar algunas propiedades como atributo o solo como etiqueta.**

##  4. Palabras Clave

Para poder definir el valor que se mapea en un determinado elemento se debe hacer por medio de palabras claves, que son las que nos permiten obtener un valor, existen dos tipos de palabras claves, las de valores y las de funciones.

Palabras clave - Valores: Son las que nos permiten mapear un valor directamente.

|**NOMBRE**|**DESCRIPCION**  |
|--|--|
|Predeterminado |Valor por defecto  |
|Variable|Valor de una variable que se configura en la clase OwlHandler o con las opciones VariableGlobal y Contador|
|Reservada|Valor que se obtiene a partir de una palabra reservada en la estructura de salida|
|Buscar|Valor que se busca en la estructura de entrada|
|Referencia|Valor que se obtiene del DataSet de referencias configurado en OwlSettings|

Palabras clave - Funciones: Son las que nos permiten realizar alguna función previa a los valores para retornar los datos

|**NOMBRE**  |**DESCRIPCION**  |
|--|--|
|Concatenar  |Función para concatenar múltiples valores  |
|SubCadena  |Función para obtener una subcadena  |
| Longitud |Función para obtener la longitud de un valor  |
|EsNumero  |Función para validar si el valor representa un número  |
|IndiceDe  |Función para obtener la posición donde se encuentra una subcadena dentro del valor  |
|EsVacio  |Función para validar si el valor está vacío  |
|LimpiarEspacios  |Función para quitar los espacios en blanco al valor  |
|Si  |Función para hacer un validación entre dos valores  |

Las palabras claves se colocan en el Owl Output Config con la sintaxis XPML, sin embargo, las palabras clave Predeterminado, Variable, Reservada y Buscar se pueden colocar como atributos: 

 - Atributo: El Owl busca el atributo que corresponda a una palabra
   clave permitida como atributo y obtiene el valor, en caso de que se
   coloquen varios atributos que correspondan a palabras claves solo se
   toma uno (para definir cual se toma, Owl tiene en cuenta el orden
   dado en la tabla anterior).

### 4.1 Predeterminado 

Valor por defecto, la sintaxis es: 
        
 - Atributo: Predeterminado=”VALOR”
 -   XPML: Las siguientes dos opciones:

      <Predeterminado Valor="" />
    <Predeterminado>
     <Predeterminado.Valor></Predeterminado.Valor>
    </Predeterminado>

### 4.2 Variable 
Valor de una variable que se configura en la clase OwlHandler o con las opciones VariableGlobal y Contador. 

Para agregar variables por código se realiza de la siguiente forma:

    Owl["Line"] = "CustomValue";
    
La sintaxis es: 

 -  Atributo: Variable=”NOMBRE_VARIABLE”  
 -   XPML: Las siguientes dos
   opciones:

    <Variable Valor="" />
    <Variable>
     <Variable.Valor></Variable.Valor>
    </Variable>

Si se trata de acceder a una variable que no existe se genera una excepción de tipo OwlException. 

### 4.3 Reservada
Valor que se obtiene a partir de una palabra reservada en la estructura de salida, la sintaxis es: 

 -  Atributo: Reservada=”PALABRA_RESERVADA” 
 -   XPML: Las siguientes dos
   opciones:

    <Reservada Valor="" />
    <Reservada>
     <Reservada.Valor></Reservada.Valor>
    </Reservada>
    
Las salidas de Owl incluyen las siguientes palabras reservadas: 

 - **CONSECUTIVO_ITERA**: Entrega el consecutivo de la iteración actual.
 
 - **CONSECUTIVO_ITERA_PADRE**: Entrega el consecutivo de la iteración actual de la sección padre.

 - **FECHAHORA_SISTEMA:FORMATO**: Fecha y hora del sistema. El formato es opcional, si no se coloca formato, por defecto es “yyyyMMdd”, los formatos soportados son los mismos de .Net

  - Ejemplo:
 
    - Si la fecha es 2011/11/21 el Owl devolverá lo siguiente:
    - FECHAHORA_SISTEMA  Devuelve 20111221 
    - FECHAHORA_SISTEMA:yyyyMMddHHmm  Devuelve 201111210840

 - **CONTADOR_SEGMENTOS**: Contador de segmentos (Secciones).

  - **CONSECUTIVO_ESTRUCTURA**: Entrega el consecutivo de la iteración de la estructura actual.

En caso de colocar una palabra reservada no soportada se genera una excepción de tipo OwlException.

### 4.4 Buscar

Valor que se busca en la estructura de entrada, la sintaxis es:

 -  Atributo: uscar=”EXPRESION_A_EVALUAR”
 -   XPML: Las siguientes dos
   opciones:

    <Buscar Valor="" />
    <Buscar>
     <Buscar.Valor></Buscar.Valor>
    </Buscar>

Para los tipos soportados por Owl en caso de haber un error con la expresión se genera una excepción de tipo OwlSectionException. 

**NOTA: La EXPRESION_A_EVALUAR la define la estructura de entrada (por defecto Owl da soporte a entradas XML y las expresiones usadas se dan en Xpath).**

### 4.5 Referencia 

Valor que se obtiene del DataSet de referencias configurado en OwlSettings, la sintaxis es: 

 -   XPML: Las siguientes dos opciones:
 
    <Referencia Tabla="" ValorDefecto="" Formato="" CamposObtener="," CamposBuscar=","
    ValoresBuscar="," />
    <Referencia>
     <Referencia.Tabla></Referencia.Tabla>
     <Referencia.ValorDefecto></Referencia.ValorDefecto>
     <Referencia.Formato></Referencia.Formato>
     <Referencia.CamposObtener>
     <Referencia.CamposObtener.Valor></Referencia.CamposObtener.Valor>
     </Referencia.CamposObtener>
     <Referencia.CamposBuscar>
     <Referencia.CamposBuscar.Valor></Referencia.CamposBuscar.Valor>
     </Referencia.CamposBuscar>
     <Referencia.ValoresBuscar>
     <Referencia.ValoresBuscar.Valor></Referencia.ValoresBuscar.Valor>
     </Referencia.ValoresBuscar>
    </Referencia>
    
 -  Tabla: Nombre de la tabla dentro del DataSet donde se va a realizar la búsqueda. 
   
 - ValorDefecto: Valor a retornar si la búsqueda no
   retorna datos.
   
 -  Formato: Se configura el formato para los campos a
   obtener, se soportan los mismos de .Net para strings.  
   
 -  CamposObtener: Lista de campos a retornar 
 
 -  CamposBuscar: Lista de campos donde se va a buscar (Los CamposBuscar deben ser la misma)
   cantidad que ValoresBuscar, si no se cumple se genera una excepción
   de tipo OwlExcepcion.)  
 
 -  ValoresBuscar: Lista de valores a buscar en
   los campos  
   
 -  Configuración opcional: Formato, ValorDefecto (si no se
   configura queda una cadena vacía).

### 4.6 Concatenar 
Función para concatenar múltiples valores, la sintaxis es: 

 -   XPML: Las siguientes dos opciones:

    <Concatenar Separador="" Formato="" Valores="," />
    <Concatenar>
     <Concatenar.Separador></Concatenar.Separador>
     <Concatenar.Formato></Concatenar.Formato>
     <Concatenar.Valores>
     <Concatenar.Valores.Valor></Concatenar.Valores.Valor>
     </Concatenar.Valores>
    </Concatenar>

 -  Separador: Valor con el que se van a separar los campos.  
 
 -  Formato: Se configura el formato para los Valores a concatenar, se soportan
   los mismos de .Net para strings.

 - Valores: Lista de Valores a
   concatenar.  

 -  Configuración opcional: Separador, Formato.
 -   Si se configuran Separador y Formato al mismo tiempo, solo se tiene en cuenta separador.

### 4.7 SubCadena
 Función para obtener una subcadena, la sintaxis es:

 -   XPML: Las siguientes dos opciones:

    <SubCadena Valor="" Inicio="" Largo="" Limpiar="" />
    <SubCadena>
     <SubCadena.Valor></SubCadena.Valor>
     <SubCadena.Inicio></SubCadena.Inicio>
     <SubCadena.Largo></SubCadena.Largo>
     <SubCadena.Limpiar></SubCadena.Limpiar>
    </SubCadena>

 -  Valor: Valor original  
 
 -  Inicio: Posición inicial de donde se va a
   iniciar a tomar la subcadena (Si el valor es menor a 0, Owl lo vuelve 0).
   
 -  Largo: Longitud, a partir de la posición de Inicio que se va a
   tomar para la subcadena (si es menor que la posición de inicio,
   retorna una cadena vacía. Si el largo es mayor al valor original
   retorna hasta la última posición).
   
 -  Limpiar (Si, No): Indica si
   quitan los espacios en blanco al inicio y fin de la subcadena.
   

-    Configuración opcional: Largo, Limpiar.

### 4.8 Longitud 
Función para obtener la longitud de un valor, la sintaxis es: 

 -   XPML: Las siguientes dos opciones:

    <Longitud Valor="" />
    <Longitud>
     <Longitud.Valor></Longitud.Valor>
    </Longitud>

 -  Valor: Valor del que se va obtener la longitud.

### 4.9 EsNumero 
Función para validar si el valor representa un número, la sintaxis es: 

 -   XPML: Las siguientes dos opciones:

    <EsNumero Valor="" ValorVerdadero="" ValorFalso="" />
    <EsNumero>
     <EsNumero.Valor></EsNumero.Valor>
     <EsNumero.ValorVerdadero></EsNumero.ValorVerdadero>
     <EsNumero.ValorFalso></EsNumero.ValorFalso>
    </EsNumero>
    

 -  Valor: Valor a validar. 
 
 -  ValorVerdadero: Valor a retornar si se
   valida el número correctamente.
   
 -  ValorFalso: Valor a retornar si no
   se validar correctamente.
   

 -  Configuración opcional: ValorFalso (si no
   se configura queda una cadena vacía).

### 4.10 IndiceDe
 Función para obtener la posición donde se encuentra una subcadena dentro del valor, la sintaxis es:
  
 -   XPML: Las siguientes dos opciones:

    <IndiceDe Valor="" CadenaBuscar="" Inicio="" />
    <IndiceDe>
     <IndiceDe.Valor></IndiceDe.Valor>
     <IndiceDe.CadenaBuscar></IndiceDe.CadenaBuscar>
     <IndiceDe.Inicio></IndiceDe.Inicio>
    </IndiceDe>

 -  Valor: Valor a procesar.
 -  CadenaBuscar: Subcadena que se va a
   buscar.  
   
 - Inicio: Posición desde donde se va a iniciar a buscar la
   subcadena (Si se configura un número menor a 0 o mayor a la longitud del valor a procesar genera OwlElementException).  
   
 -  Configuración opcional: Inicio (si no se configura queda 0).
 

### 4.11 EsVacio 

Función para validar si el valor está vacío, la sintaxis es: 

 -   XPML: Las siguientes dos opciones:
 
    <EsVacio Valor="" ValorVerdadero="" ValorFalso="" Limpiar="" />
    <EsVacio>
     <EsVacio.Valor></EsVacio.Valor>
     <EsVacio.ValorVerdadero></EsVacio.ValorVerdadero>
     <EsVacio.ValorFalso></EsVacio.ValorFalso>
     <EsVacio.Limpiar></EsVacio.Limpiar>
    </EsVacio>
    
 -  Valor: Valor a validar. 
 
 - ValorVerdadero: Valor a retornar si el
   valor es vacío.
  
  - ValorFalso: Valor a retornar si el valor no es vacío.
  
  - Limpiar (Si, No): Indica si quitan los espacios en blanco al inicio y fin del valor antes de validar.
   
  -  Configuración opcional: ValorFalso (si no se configura queda una cadena vacía), Limpiar (si no se configura queda false).


### 4.12 LimpiarEspacios
 Función para quitar los espacios en blanco al valor, la sintaxis es: 
 
 -   XPML: Las siguientes dos opciones:

    <LimpiarEspacios Valor="" Lado="" />
    <LimpiarEspacios>
     <LimpiarEspacios.Valor></LimpiarEspacios.Valor>
     <LimpiarEspacios.Lado></LimpiarEspacios.Lado>
    </LimpiarEspacios>

 -  Valor: Valor a procesar.
 -  Lado (Inicio, Fin): Indica el lado donde se limpian los espacios, si no se envía se limpian en ambos lados.
-  Configuración opcional: Lado.


### 4.13 Si

 Función para hacer una validación entre dos valores, la sintaxis es: 
 
 -   XPML: Las siguientes dos opciones:

    <Si Valor1="" Valor2="" TipoComparacion="" ValorVerdadero="" ValorFalso="" />
    <Si>
     <Si.Valor1></Si.Valor1>
     <Si.Valor2></Si.Valor2>
     <Si.TipoComparacion></Si.TipoComparacion>
     <Si.ValorVerdadero></Si.ValorVerdadero>
     <Si.ValorFalso></Si.ValorFalso>
    </Si>

 -  Valor1: Primer valor a comparar. 
 
 -  Valor2: Segundo valor a comparar.  

 -  TipoComparacion (Igual, Diferente, Mayor, Menor, MayorIgual, MenorIgual): Tipo de comparación a realizar entre los dos valores 
   
 -  ValorVerdadero: Valor a retornar si se cumple la validación
   

 -  ValorFalso: Valor a retornar si no se cumple la validación.
  
 -  Configuración opcional: ValorFalso (si no se configura queda una cadena vacía).


## 5. Manejo de Excepciones

 Owl valida la sintaxis del archivo de configuración y el acceso a datos, dentro de estas validaciones se pueden generar 3 tipos de excepciones:
  
 -  OwlException  
 -  OwlSeccionException    
 -  OwlElementException   
 -  OwlKeywordException

   Las excepciones dan información detallada de en qué parte del proceso se generó el error para su pronta corrección dentro del archivo de configuración. 

### 5.1 OwlException

 Excepción general de Owl de la cual se puede obtener la sección donde ocurrió el error y un mensaje. 

### 5.2 OwlSeccionException

 Excepción producida dentro del procesamiento de una seccion, de ella se puede obtener el bloque XML donde ocurrió el error, el nodo padre, la sección y un mensaje. 

### 5.3 OwlElementException

 Excepción producida dentro del procesamiento de un elemento, de ella se puede obtener el bloque XML donde ocurrió el error, el nodo padre, la sección y un mensaje.

### 5.4 OwlKeywordException 

Excepción producida dentro de la ejecución de alguna palabra clave, de ella se puede obtener la palabra clave que generó la excepción.

 OwlSeccionException y OwlElementException pueden contener como InnerException excepciones de los demás tipos de Owl, por lo tanto, si usted requiere información más detallada puede verificar si la excepción interna es de alguno de los tipos de Owl para ver la información interna.


## 6. Formatos Owl 

Owl provee algunos formatos de entrada y salida para el procesamiento de datos, sin embargo, usted puede crear sus propios tipos implementando la interfaz IFormatInput o IFormatOutput. 


### 6.1 XMLInput

 Define una estructura XML de entrada.

 Para crear un objeto de este tipo tenemos varias opciones:

 - XmlDocument: El constructor recibe un parámetro del tipo con el
   contenido del XML a procesar.  
   
 - Archivo: El constructor recibe la
   ruta del archivo a procesar.

Adicional se provee el método “AddNamespace” para agregar los namespaces que utiliza el XML, sin embargo, si no se agregan Owl los detecta y los agrega automáticamente. La forma de acceder a los datos se hace por medio de XPath. 

Ejemplo: 
Se tiene el siguiente XML:

    <?xml version="1.0" encoding="utf-8"?>
    <EFACT_D98B_ORDERS>
     <BGM BGM03="9">
     <C002 C00201="220" />
     <C106 C10601="251422" />
     </BGM>
     <DTM>
     <C507 C50701="137" C50702="20101109000000" C50703="204" />
     </DTM>
     <DTM>
     <C507 C50701="64" C50702="20101129000000" C50703="204" />
     </DTM>
     <LINLoop1>
     <LIN LIN01="1">
     <C212 C21201="7797679401230" C21202="EN" />
     </LIN>
     <IMD IMD01="F">
     <C273 C27304="SOPORTE LATERAL 300MMBCO.." />
     </IMD>
     </LINLoop1>
     <LINLoop1>
     <LIN LIN01="2">
     <C212 C21201="7797679444121" C21202="EN" />
     </LIN>
     <IMD IMD01="F">
     <C273 C27304="SOPORTE BRACKET 250X300MMBC" />
     </IMD>
     </LINLoop1>
    </EFACT_D98B_ORDERS>

Para obtener los datos en la estructura de salida se debe iterar por el nodo principal:

    <Salida Tipo="EDI">
     <Estructura Itera="EFACT_D98B_ORDERS">
     ...
     </Estructura>
    </Salida>

Para obtener datos del encabezado se crean XPath dentro de una sección para acceder a los datos:



    <Seccion Nombre="BGM" Requeridos="BGM_1004">
     <Elemento Nombre="BGM_C002_1001" Buscar="BGM[@BGM03='9']/C002/@C00201"/>
     <Elemento Nombre="BGM_C002_3055" Predeterminado="9"/>
     <Elemento Nombre="BGM_1004" Buscar="BGM[@BGM03='9']/C106/@C10601"/>
    </Seccion>

Para iterar sobre el detalle se crea una sección que recorra el LINLoop (Los XPath que se creen dentro de esta
iteración inician a partir de la etiqueta LINLoop1):

    <Seccion Nombre="LIN" Itera="LINLoop1">
     <Elemento Nombre="LIN_1082" Reservada="CONSECUTIVO_ITERA"/>
     <Elemento Nombre="LIN_C212_7140" Buscar="LIN/C212/@C21201"/>
    </Seccion>


### 6.2 GenericOutput 

Clase abstracta la cual define un documento de salida con atributos comunes (todos los formatos de Salida del Owl heredan de esta clase). 

**Nota: Al ser una clase abstracta no se puede instanciar, para ello utilice uno de los formatos específicos que provee Owl.** 

Configuración a nivel de Estructura:

| **Propiedad** | **Tipo Dato** |**Atributo** |**Etiqueta**|**Descripción**|
|--|--|--|--|--|
|Itera  |String  |X|X|Expresión a evaluar para hacer la iteración|
|Linea |String  |X|X|Etiqueta donde se encuentra el contenido de la línea|
|NumeroLinea |String  |X|X|Etiqueta donde se encuentra el número de la línea|
|SeccionEvento |Boolean  |X|X|Indica si se genera el evento final de las secciones (afecta todas las secciones)|
|SeccionEventoPrevio |Boolean  |X|X|ndica si se genera el evento previo de las secciones (afecta todas las secciones)|
|SeccionEventoGrupo|Boolean| X|X|Indica si se genera el evento de las secciones que tiene itera o más de 1 repetición (afecta todas las secciones)|
|ElementoEvento|Boolean|X|X|Indica si se genera el evento de los elementos (afecta todos los elementos)|
|ElementoEventoSeccion|Boolean |X|X|Indica si los elementos son incluidos cuando se genere el evento final de las secciones (afecta todos los elementos)|
|SeparadorDecimalesEntrada|Char|X|X|Carácter usado como separador de decimales en el contenido de entrada (afecta todos los elementos)|
|SeparadorDecimalesSalida|Char|X|X|Carácter usado como separador de decimales en el contenido de salida (afecta todos los elementos)|
|Id|String|X||Id de la sección|
|Base|String|X||Id de la estructura base|


Configuración a nivel de Sección:

| **Propiedad** | **Tipo Dato** |**Atributo** |**Etiqueta**|**Descripción**|
|--|--|--|--|--|
|Nombre |String  |X||Nombre de la sección|
|Nodo |String  |X|X|Etiqueta de donde se toma la línea y contenido de la línea|
|Itera |String  |X|X|Expresión a evaluar para hacer una iteración|
|Evento|Boolean  |X|X|Indica si se genera el evento final para la sección (afecta solo la sección)|
|EventoPrevio |Boolean  |X|X|Indica si se genera el evento previo para la sección (afecta solo la sección)|
|EventoGrupo|Boolean| X|X|Indica si se genera el evento cuando la sección tiene itera o tiene más de 1 repetición (afecta solo la sección)|
|Descripcion|String|X|X|Descripción de la sección|
|Repeticiones|Int |X|X|Cantidad de veces que se debe repetir la sección|
|Id|String|X||Id de la sección|
|MostrarContenido|Boolean|X|X|Indica si el contenido de la sección se muestra en el evento|
|Sobreescribir|Boolean|X|X|(Herencia) Indica si se sobrescribe la sección|
|Base|String|X||Id de la estructura base|
|AntesDe|String|X||(Herencia) Nombre de la sección que va después de la actual|
|DespuesDe|String|X||(Herencia) Nombre de la sección que va antes de la actual|
|Requeridos|String (Atributo) XPML (Etiqueta)|X|X|Lista de elementos requeridos para que se genere la sección|
|RequeridosGrupo|String (Atributo) XPML (Etiqueta)|X|X|(Herencia) Nombre de la sección que va antes de la actual|
|Si|XPML)||X|Condición que se debe cumplir para que se genere la sección|

 - Requeridos y RequeridosGrupo: Se utilizan cuando se requiere
   validar que exista algún(os) elemento(s) o algún contenido para que la sección se genere.

Ejemplo como atributo para RequeridosGrupo: En este caso es necesario que se genere el QTY para que pueda generarse el PIA

    <Seccion Nombre="PIA" RequeridosGrupo="QTY">
     <Elemento Nombre="PIA_4347" />
     <Elemento Nombre="PIA_C212_7140" />
     <Elemento Nombre="PIA_C212_7143" />
     <Seccion Nombre="QTY">
     <Elemento Nombre="QTY_C186_6063" />
     <Elemento Nombre="QTY_C186_6060" />
     <Elemento Nombre="QTY_C186_6411" />
     </Seccion>
    </Seccion>

Ejemplo como atributo para Requeridos: En este caso es necesario que el elemento PIA_4347 tenga valor para que se pueda generar el PIA.

    <Seccion Nombre="PIA" Requeridos="PIA_4347">
     <Elemento Nombre="PIA_4347" />
     <Elemento Nombre="PIA_C212_7140" />
     <Elemento Nombre="PIA_C212_7143" />
     <Seccion Nombre="QTY">
     <Elemento Nombre="QTY_C186_6063" />
     <Elemento Nombre="QTY_C186_6060" />
     <Elemento Nombre="QTY_C186_6411" />
     </Seccion>
    </Seccion>

Ejemplo como etiqueta para RequeridosGrupo:

    <Seccion.RequeridosGrupo>
     <Campo>seccion1</Campo>
     <Campo OperadorLogico="Y">
     <Requeridos>
     <Campo>seccion2</Campo>
     <Campo OperadorLogico="O">seccion3</Campo>
     </Requeridos>
     </Campo>
    </ Seccion.Requeridos>

Ejemplo como etiqueta para Requeridos:

    <Seccion.Requeridos>
     <Campo>campo1</Campo>
     <Campo OperadorLogico="Y">
     <Requeridos>
     <Campo>campo2</Campo>
     <Campo OperadorLogico="O">campo3</Campo>
     </Requeridos>
     </Campo>
    </ Seccion.Requeridos>


 -       
    - La etiqueta Campo puede colocarse n veces de acuerdo a los campos que se vayan a configurar.   
    
    - El atributo OperadorLogico recibe los valores “O” o “Y” y se configura a partir del segundo campo.
    
    -  La etiqueta Campo puede llevar el nombre del elemento, el valor de la sección o una nueva configuración de Requeridos. Cuando un campo lleva una etiqueta de Requeridos primero se evalúa el contenido interno.
          
 - Si: Se usa esta etiqueta para crear un condicional que indica si la sección se genera o no. 
 
La sintaxis es:

    <Seccion.Si Previo="Si">
     <Condicion>
     <Valor1>
     <!--VALOR EN XPML-->
     </Valor1>
     <Valor2 TipoComparacion="Igual">
     <!--VALOR EN XPML-->
     </Valor2>
     </Condicion>
     <Condicion OperadorLogico="O">
     <Si>
     <Condicion>
     <Valor1>
     <!--VALOR EN XPML-->
     </Valor1>
     <Valor2 TipoComparacion="Igual">
     <!--VALOR EN XPML-->
     </Valor2>
     </Condicion>
     <Condicion OperadorLogico="Y">
     <Valor1>
     <!--VALOR EN XPML-->
     </Valor1>
     <Valor2 TipoComparacion="Igual">
     <!--VALOR EN XPML-->
     </Valor2>
     </Condicion>
     </Si>
     </Condicion>
    </Seccion.Si>


 -        
      -  El atributo Previo indica si la validación se realiza antes o después de procesar los elementos de la sección, se puede dar lo siguiente: 
            
              ▪ Previo=“Si”: Se valida si se cumple lo configurado en el “Si”, en caso de cumplirse se continua el procesamiento de los elementos, en caso de NO cumplirse se cancela el procesamiento de la sección y se ignoran los elementos. 
              ▪ Previo=“No”: Primero se procesan los elementos y posteriormente se valida si se cumple lo configurado en el “Si”, en caso de cumplirse se deja el segmento, en caso de NO cumplirse se elimina el segmento del contenido de salida.
              
                             
        - La etiqueta Condicion puede colocarse n veces de acuerdo a las condiciones que se vayan a configurar.
        
        -   El atributo OperadorLogico recibe los valores “O” o “Y” y se configura a partir de la segunda condición.
        
        - La etiqueta Condición puede llevar los valores a comparar o una nueva configuración de Si, cuando un campo lleva una etiqueta Si, primero se evalúa el contenido interno.
        
        - Las etiquetas Valor1 y Valor2 son los valores a comparar en la condición, el valor se de cada etiqueta se obtiene a partir de una palabra clave en XPML.
        
        - La etiqueta Valor2 lleva el tipo de comparación a realizar, son soportados “Igual” y “Diferente” para valores alfanuméricos y “Mayor”, “Menor”, “MayorIgual” y “MenorIgual” para valores numéricos.

Configuración a nivel de Elemento:
| **Propiedad** | **Tipo Dato** |**Atributo** |**Etiqueta**|**Descripción**|
|--|--|--|--|--|
|Nombre |String  |X||Nombre del Elemento|
|VariableGlobal |String  |X|X|Nombre de la variable global a crear|
|Evento|Boolean  |X|X|Indica si se genera el evento para el elemento (afecta solo el elemento)|
|EventoSeccion |Boolean |X|X|Indica si el elemento es incluido en el evento final de la sección (afecta solo el elemento)|
|Descripcion|String |X|X|Descripción del Elemento|
|Oculto|Boolean |X|X|Indica si el campo es oculto (no se pinta en la salida)|
|SeparadorDecimalesEntrada|Char |X|X|Carácter usado como separador de decimales en el contenido de entrada (afecta solo el elemento)|
|SeparadorDecimalesSalida|Char |X|X|Carácter usado como separador de decimales en el contenido de salida (afecta solo el elemento)|
|Sobreescribir|Boolean |X||(Herencia) Indica si se sobrescribe la sección|
|AntesDe|String |X||(Herencia) Nombre de la sección que va después de la actual|
|DespuesDe|String |X||(Herencia) Nombre de la sección que va antes de la actual|
|Id|String |X||Id del Elemento|
|Nodo|String |X|X|Etiqueta de donde se toma la línea y contenido de la línea|
|TipoDato|Enum (Alfanumerico y Numerico) |X|X|Tipo de dato del Elemento|
|Requerido|Boolean |X|X|Indica si el elemento es requerido|
|Longitud |String |X|X|Configuración de la longitud y relleno del Elemento|
|Formato |String |X|X|Formato del valor|
|Contador |String |X|X|Configuración para la creación de un contador|
|Valor* |String |X|X|Valor del elemento * Si se configura como atributo se debe hacer colocando alguna de las siguiente palabras clave: Predeterminado, Variable, Reservada o Buscar|


• Longitud: Define la longitud de un campo. 
Ejemplo:

    <Elemento Nombre="Identificador"
    Longitud=”CANTIDAD_CARACTERES/ALINEACION/CARÁCTER_RELLENO/SIN_VALOR” />

 - 
     - CANTIDAD_CARACTERES: Indica la cantidad máxima de caracteres, cuando la cantidad sea de un dato numérico se coloca (ENTERO,DECIMAL).
     
        Si se requiere que coloque todos los enteros o todos los decimales se coloca * 
        
        Ejemplo:
         (* ,2) deja todos los enteros y dos decimales.
      (2, *) deja dos enteros y todos los decimales. 
      
      Cuando se coloca * los demás parámetros no se toman en cuenta.
      
      
    - ALINEACION: Recibe el valor que indica hacia donde se alinea el campo (Izquierda, Derecha, Numero y Numero-S), Numero centra el valor y coloca relleno al lado entero y al lado decimal con el separador de decimales y Numero-S centra igual que Numero pero no coloca separador de decimales.

    - CARÁCTER_RELLENO: Carácter para rellenar el campo, cuando la cantidad sea de un dato numérico y la alineación sea Numero o Numero-S se puede colocar (ENTERO,DECIMAL).

 - SIN_VALOR: Indica que hacer con la longitud si el elemento no tiene valor (Completar, NoCompletar y Completar-S). La configuración “Completar-S” es utilizada solo cuando en Alineación se coloque “Numero” y se utiliza para que no se coloque el separador cuando el elemento no tenga valor.
 
 

 -  Formato: Le da formato al valor final del elemento, los posibles valores son:

      - Texto: Formato para valores alfanuméricos, los formatos soportados son:
             - A: Mayúsculas.
             - a: Minúsculas.
             - C: Capital.
             - T: Limpiar Espacios.
             - TS: Limpiar Espacios al Inicio. 
             - TE: Limpiar Espacios al Final.
         
     - Numero: Formato para valores numéricos, los formatos soportados son:
             - #Formato: El “Formato” son los soportados por .Net
             
     
 Si se requiere aplicar varios formatos se separan por pipe “|” 
 Ejemplo:    
 

    <Elemento Formato="A|T" />

 - Contador: Crea una variable para ser usada como un contador en Owl.
 
  Ejemplo:

    <Elemento Contador=" Nombre/Incremento" />

 - - Nombre: Nombre del contador, se crea una variable Owl con ese nombre.
 
   - Incremento: Valor con el que se incrementa el contador, puede ser el valor del elemento y se colocaría “ValorActual”.

### 6.2.1 Eventos

 - **StructureInitiated:** Evento generado cuando se va a iniciar una estructura. En este evento podemos modificar las propiedades configurables en el Owl Output Config.
 
 - **SectionGroupInitiated:** Este evento se produce cuando se empieza a procesar una sección que tiene Itera o se genera más de una vez. En este evento tenemos acceso a la configuración de la sección y a la cantidad de veces que se va a generar, también podemos modificar las propiedades configurables en el Owl Output Config.
 
 - **SectionInitiated:** Este evento se produce cuando se va a iniciar a procesar una sección. En este evento tenemos acceso a la configuración de la sección, también podemos modificar las propiedades configurables en el Owl Output Config.
 
 - **SectionProcessed:** Este evento se produce cuando se termina de procesar una sección. En este evento tenemos acceso a la configuración de la sección, los elementos de la sección y el contenido de la sección, también podemos modificar las propiedades configurables en el Owl Output Config.

También desde el método podemos cancelar la escritura de la sección, se hace de la siguiente forma:

    static void Output_SectionGroupInitiated(object sender, OutputSectionEventArgs e)
    {
     e.XPMLConfig.CancelWriteSection = true;
    }

Por defecto ninguna sección genera evento, pero si se requiere modificar el comportamiento se puede hacer de la siguiente forma:

    <Estructura SeccionEvento="Si">

Si se requiere solo modificar el comportamiento de una sola sección se realiza de la siguiente forma:

    <Seccion Nombre="Campos" Evento="Si">

 - **AlphanumericElementProcessed:** Este evento se produce cuando se termina de procesar un elemento de tipo Alfanumerico y en él podemos modificar las propiedades configurables en el Owl Output Config y marcar el elemento con error.
 
 - **NumericElementProcessed:** Este evento se produce cuando se termina de procesar un elemento de tipo Numerico y en él podemos modificar las propiedades configurables en el Owl Output Config y marcar el elemento con error.

También desde el método podemos establecer si el campo tiene error o modificar su valor, se hace de la siguiente forma:

    static void Output_AlphanumericElementProcessed(object sender, OutputElementEventArgs e) { e.XPMLConfig.ElementWithError = true; e.XPMLConfig.Valor = "nuevovalor"; }
 
 Al colocarlo en error el campo queda en blanco

Por defecto ningún elemento en la configuración genera evento, pero si se requiere modificar el comportamiento se puede hacer de la siguiente forma:

    <Estructura ElementoEvento="No" ElementoEventoSeccion="No">

Si se requiere modificar solo el comportamiento de un solo elemento se configura de la siguiente forma:

    <Elemento Nombre="Campo" Evento="No" EventoSeccion="Si" />


### 6.3 FlatFileOutput

 Define un archivo plano de salida, adicional a la configuración estándar soporta los siguientes atributos. Configuración a nivel de Sección:
   
   | **Propiedad** | **Tipo Dato** |**Atributo** |**Etiqueta**|**Descripción**|
|--|--|--|--|--|
|Separador |String  |X|X|Separador utilizado para separar los elementos|
|QuitarSeparadoresFinal |Boolean |X|X|Indica si se quitan los separadores al final de la sección|

### 6.4 EDIOutput 

Define un EDI de salida, adicional a la configuración estándar soporta los siguientes atributos. 

Configuración a nivel de Estructura:
   
  
 | **Propiedad** | **Tipo Dato** |**Atributo** |**Etiqueta**|**Descripción**|
|--|--|--|--|--|
|SeparadorSubElementos|Char  |X|X|Separador de subelementos|
|SeparadorElementos |Char |X|X|Separador de elementos|
|TerminadorSegmento |Char |X|X|Terminador de segmentos|
|CaracterEscape |Char |X|X|Carácter de escape|

Para EDIOutput son de suma importancia los nombres que se coloquen a las Secciones y Elementos de la estructura de salida dentro del Owl Output Config.

**NOTA: Owl no tiene un orden de generación de documentos EDI, el EDI se genera de acuerdo a como esta en el Owl Output Config.**


### 6.5 ANSIOutput 

Define un ANSI de salida, adicional a la configuración estándar soporta los siguientes atributos.

Configuración a nivel de Estructura:
  | **Propiedad** | **Tipo Dato** |**Atributo** |**Etiqueta**|**Descripción**|
|--|--|--|--|--|
|SeparadorElementos|Char|X|X|Separador de elementos|
|TerminadorSegmento|Char |X|X|Terminador de segmentos|
|CaracterEscape |Char |X|X|Carácter de escape|

Para ANSISalida son de suma importancia los nombres que se coloquen a las Secciones y Elementos de la estructura de salida dentro del Owl Output Config.

**NOTA: Owl no tiene un orden de generación de documentos ANSI, el ANSI se genera de acuerdo a como esta en el Owl Output Config.**


### 6.6 XMLOutput

Define un XML de salida, adicional a la configuración estándar soporta los siguientes atributos.

Configuración a nivel de Estructura:

 | **Propiedad** | **Tipo Dato** |**Atributo** |**Etiqueta**|**Descripción**|
|--|--|--|--|--|
|XMLValor|String|X|X|Nombre del elemento que contiene el valor interno de las etiquetas|
|EscaparCaracteres|Boolean |X|X|Indica si se escapan los caracteres Xml reservados en los valores de los elemento, por defecto es “Si” (Afecta todos los elementos)|

Configuración a nivel de Sección:

 | **Propiedad** | **Tipo Dato** |**Atributo** |**Etiqueta**|**Descripción**|
|--|--|--|--|--|
|TipoEtiqueta|Enum (Compuesta, Simple, Apertura, Cierre, Comentario y CData)|X|X|Tipo de etiqueta Xml que se va a generar, por defecto es “Compuesta”|

Configuración a nivel de Elemento:
 | **Propiedad** | **Tipo Dato** |**Atributo** |**Etiqueta**|**Descripción**|
|--|--|--|--|--|
|TipoElementoXml|Enum (Comentario y CData)|X|X|Tipo de etiqueta Xml que se va a generar, por defecto es “Compuesta”|
|EscaparCaracteres|Boolean|X|X|Indica si se escapan los caracteres Xml reservados en el valor del elemento (Afecta solo ese elemento)|
|ValorRequerido|Boolean|X|X|Indica si, en caso de no tener valor el elemento, no se genera el atributo|

### 6.7 SQLOutput 

Define un SQL de salida, adicional a la configuración estándar soporta los siguientes atributos.

Configuración a nivel de Sección:
 | **Propiedad** | **Tipo Dato** |**Atributo** |**Etiqueta**|**Descripción**|
|--|--|--|--|--|
|TipoSQL|Enum (Select, Insert, Update y Delete)|X|X|Tipo de SQL que se va a generar, por defecto es “Select”|

Configuración a nivel de Elemento:
 | **Propiedad** | **Tipo Dato** |**Atributo** |**Etiqueta**|**Descripción**|
|--|--|--|--|--|
|TipoElementoSQL|Enum (Select y Where)|X|X|Indica si el elemento hace parte del Select o del Where, no apica para Insert. - Para Delete todos los elementos son Where - Para Update solo aplica Where, los demás campos son los que se actualicen.|
|TipoWhere|Enum (Equal, Different, Like, In, Between y NotBetween|X|X|Cuando el TipoElementoSQL es de tipo Where indica el operador lógico con el que queda el valor.|
|ValorRequerido|Boolean|X|X|Indica si, en caso de no tener valor el elemento, se genera como NULL.|


### 6.8 JsonOutput
Define un Json de salida, no tiene configuración adicional a la estándar. 
El Json se define de la siguiente forma:

 - Seccion: Objeto Json.
 - Seccion con Iteración o Repetición: Array Json.
 - Elemento; Propiedad.

                           
## 7. Herencia de Configuraciones.

Teniendo en cuenta que muchas configuraciones son muy similares y muchas veces solo se cambia el valor de un campo o una sección, Owl brinda la posibilidad de que una estructura de salida pueda heredar de otra estructura de salida, para realizar esto se debe hacer lo siguiente:  

 -  Tener una configuración Base.
 - 

    <Estructura Id="Principal" Itera="EFACT_D98B_ORDERS">
     <Seccion Nombre="Encabezado" Separador="|">
     <Elemento Nombre="c1" Predeterminado="1" Longitud="2" />
     <Elemento Nombre="emisor" Buscar="NADLoop1/NAD[@NAD='BY']/C082/@C08201" />
     <Elemento Nombre="receptor" Buscar="NADLoop11/NAD1[@NAD01='SU']/C0591/@C05901" />
     </Seccion>
     <Seccion Nombre="Encabezado2" Separador="|">
     <Elemento Nombre="c1" Predeterminado="1" />
     <Elemento Nombre="emisor" Buscar="NADLoop1/NAD[@NAD='BY']/C082/@C08201" />
     </Seccion>
     <Seccion Nombre="Detalle" Separador="|" Itera="LINLoop1">
     <Elemento Nombre="c1" Predeterminado="2" />
     <Elemento Nombre="c2" Reservada="CONSECUTIVO_ITERA" />
     <Elemento Nombre="c3" Buscar="LIN/C212/@C21201" />
     </Seccion>
    </Estructura>

Es obligatorio que la estructura que va a servir como Base tenga configurado el atributo “Id” de lo contrario el Owl genera una excepción.

 - Crear la estructura Hija
 - 
 
     <Estructura Id="Secundario" Base="Principal">
     <Seccion Nombre="Encabezado" Separador="@">
     <Elemento Nombre="c1" Predeterminado="nuevovalor" Oculto="Si" />
     </Seccion>
    </Estructura>

Es obligatorio que la estructura hija tenga configurado los atributos “Id” y “Base”, en la Base se debe colocar el Id de la estructura Base, la estructura base se configura por medio de OwlSettings con las propiedades PathConfigBase o XmlConfigBase.

 - En la estructura hija se pueden modificar los atributos de la etiqueta Estructura de la Base.
 
 -  En la estructura hija se deben configurar solo las secciones y elementos de la base que se van a modificar, para hacer la relación de que sección o elemento es el que se está modificando se usa el id o el nombre.
 
 - Si se configuran Requeridos siempre se dejan los de la configuración hija.

Las opciones para reemplazar secciones y elementos en la configuración hija son las siguientes:

 - Si solo se quieren agregar o actualizar atributos se coloca el elemento o sección con los nuevos valores
 - Si se desea cambiar la ubicación de la sección o un elemento se hace por medio de los atributos AntesDe y DespuesDe, donde se coloca el id de la sección o elemento que servirá como referencia para dar la nueva ubicación.
 - Si se agrega una nueva sección por defecto se coloca al final, en caso de requerir dar ubicación se utilizan los atributos AntesDe o DespuesDe
 - Si se desea sobreescribir la sección o el elemento completo en la configuración hija se coloca el atributo “Sobreescribir” con valor “Si” Ejemplo:
 - 

      <Elemento Nombre="c1" Sobreescribir=”Si” />


## 8. Hola Mundo

A partir de un ORDEN se requiere generar un plano de salida que tiene la siguiente estructura (Los campos se separan por pipe):

|**ENCABEZADO**  |
|NOMBRE  | SEGMENTO |
|--|--|
|ID  |“1”  |
|EMISOR |NAD+BY|
|No. ORDEN |BGM 220|
|FECHA ORDEN |DTM 137|
|FECHA ENTREGA |DTM 2|
|CODIGO PROVEEDOR |NAD+SU|

|**DETALLE**  |
|NOMBRE  | SEGMENTO |
|--|--|
|ID  |“2”|
|EAN PRODUCTO |LIN|
|Valor|MOA+203|
|CANTIDAD ORDENADA |QTY+21|


|**RESUMEN**  |
|NOMBRE  | SEGMENTO |
|--|--|
|ID  |“3”|
|TOTAL |MOA|

Los pasos para generar la salida son:

 -  Crear XML de configuración:
 - 
 
     <?xml version="1.0" encoding="utf-8" ?>
    <Configuracion Version="1.0">
     <Documento>
     <Salida Tipo="Plano">
     <Estructura Itera="ORDERS_EDI">
     <Seccion Nombre="Encabezado" Separador="|">
     <Elemento Nombre="ID" Predeterminado="1" />
     <Elemento Nombre="Emisor"
    Buscar="NADs/NAD[CalificadorParte='BY']/IdentificacionParte" />
     <Elemento Nombre="NoOrden" Buscar="BGM/NumeroDocumento" />
     <Elemento Nombre="Fecha_Orden"
    Buscar="DTMs/DTM[CalificadorFechaHoraPeriodo='137']/FechaHoraPeriodo" />
     <Elemento Nombre="Fecha_Entrega"
    Buscar="DTMs/DTM[CalificadorFechaHoraPeriodo='2']/FechaHoraPeriodo" />
     <Elemento Nombre="Receptor"
    Buscar="NADs/NAD[CalificadorParte='SU']/IdentificacionParte" />
     </Seccion>
     <Seccion Nombre="ValoresProducto" Separador="|" Itera="LINs/LIN">
     <Elemento Nombre="ID" Predeterminado="2"/>
     <Elemento Nombre="EAN" Buscar="NumeroItem"/>
     <Elemento Nombre="Valor"
    Buscar="MOAsINVOIC/MOA[TipoCantidadMonetaria='203']/CantidadMonetaria"/>
     <Elemento Nombre="Cantidad"
    Buscar="QTYsPRICAT/QTY[CalificadorCantidad='21']/Cantidad"/>
     </Seccion>
     <Seccion Nombre="Resumen" Separador="|">
     <Elemento Nombre="ID" Predeterminado="3" />
     <Elemento Nombre="Total"
    Buscar="UNS/MOAs/MOA[TipoCantidadMonetaria='116']/CantidadMonetaria" />
     </Seccion>
     </Estructura>
     </Salida>
     </Documento>
    </Configuracion>
    
 -  Definimos la configuración de mapeo e invocamos a OwlHandler:
 -   
 

        OwlSettings config = new OwlSettings();
            config.PathConfig = @"XML_EDI\config.xml";
            OwlHandler handler = new OwlHandler(config);
            try
            {
             handler.LoadConfigMap();
             handler.LoadInput(entrada);
            }
            catch (OwlSectionException e)
            {
            }
            catch (OwlException e)
            {
            }
    
    
     -  Creamos el objeto de salida y llamamos al método que escribe los  datos:
    FlatFileOutput salida = new FlatFileOutput();
    try
    {
     int intContador = 1;
     foreach (string strContenido in handler.WriteOutput(salida))
     {
    
     }
    }
    catch (OwlElementException e)
    {
    }
    catch (OwlSectionException e)
    {
    }
    catch (OwlException e)
    {
    }
	
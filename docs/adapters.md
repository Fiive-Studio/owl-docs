# < PGA Mapper>

## 1. Introduccion

PGA Mapper es una herramienta capaz de cargar documentos planos en objetos .Net mediante un archivo de configuración (PGA Input Config), también provee los métodos necesarios para una vez  teniendo los objetos .Net retornar a la cadena original del documento plano.
Es importante mencionar que PGA Mapper es sensible a mayúsculas y minúsculas por lo cual usted deberá tener cuidado cuando realice configuraciones.

## 2. Estructura del PGA

Para poder trabajar con PGA Mapper se requiere lo siguiente: 

 - ***PGA Input Config***: En este archivo se define la estructura de
   entrada, este archivo se configura con la sintaxis XPML (explicada
   más adelante).
   
 - ***InputConfiguration***: Clase donde se carga el PGA Input Config.  
 
 - ***Document***: Clase base para los documentos, se debe seleccionar de
   acuerdo al tipo de documento de entrada (EDIDocument, ANSIDocument).

### 2.1 PGA  Input Config
  
En este archivo es donde se define la estructura del archivo a leer, el siguiente ejemplo muestra un archivo de configuración:

    <?xml version="1.0" encoding="utf-8" ?>
    <Configuracion Version="5.0">
     <Documento>
     <Entrada Tipo="EDI">
     <Estructura>
     <Seccion Nombre="UNB" />
     <Seccion Nombre="UNH" />
     <Seccion Nombre="BGM" />
     <Seccion Nombre="DTM" />
     <Seccion Nombre="RFF" />
     <Seccion Nombre="NAD" />
     <Seccion Nombre="ERC">
     <Seccion Nombre="FTX" />
     </Seccion>
     <Seccion Nombre="UNT" />
     <Seccion Nombre="UNZ" />
     </Estructura>
     </Entrada>
     </Documento>
    </Configuracion>

 
### 2.2 InputConfiguration

Clase que representa la configuración en objetos .Net

    XmlDocument xml = new XmlDocument();
    xml.Load(@"C:\ruta\config.xml");
    InputConfiguration config = XmlExtension.Deserialize<InputConfiguration>(xml);

---
### 2.3 Document

Clase base para los documentos, se debe seleccionar de acuerdo al tipo de documento de entrada (EDIDocument, ANSIDocument), con estas clases se pueden cargar los documentos planos en objetos y retornar a la cadena original, adicional es posible al modificar los objetos generar la nueva cadena del documento con los cambios realizados.

# 3. XPML

El XPML (eXtensible PGA Markup Language / Lenguaje Extensible de Marcado para PGA) es un lenguaje de PGA para configurar los PGA Config permitiendo realizar configuraciones más robustas y con más opciones, se basa en XML. El lenguaje fue introducido a partir de la versión 2.0 de PGA. 

La sintaxis de XPML se utiliza para configurar las propiedades de un determinado objeto y se puede realizar de la siguiente forma para los PGA Input Config:

|TIPO|EJEMPLO|
|--|--|
|Atributo: Se coloca la propiedad como atributo Xml y se asigna directamente el valor de la propiedad| < Seccion Nombre=”UNB” /> |

# 4. Manejo de Excepciones

PGA Mapper valida el archivo a cargar con el PGA Input Config, dentro de estas validaciones se puede generar la siguiente excepción: 

 - PGAMapperException.

 Las excepción da información detallada de en qué parte del proceso se generó el error para su pronta corrección.
 
 ### 4.1 PGAMapperException 
 
 Excepción que se produce cuando se presenta algún error en la carga del documento, en caso de que el error sea porque el documento no cumple con la estructura configurada, en el mensaje se puede obtener la posición donde se produjo el error.

# 5. Formatos PGA Mapper

PGA Mapper provee algunos formatos de entrada para la carga de los datos.

### 5.1 Configuración General 

A continuación se detalla la configuración general para todos los formatos.

Configuración a nivel de Sección:
| **Propieda** |**Tipo Dato** | **Descripcion** |
|--|--|--|
|Nombre  |String  | Nombre de la sección |
|Descripcion|String|Descripcion de la sección|

Cuando se requiera configurar grupos de secciones, se deben incluir las secciones hijas como subetiquetas de la sección principal 

Configuración a nivel de Elemento:
| **Propieda** |**Tipo Dato** | **Descripcion** |
|--|--|--|
|Nombre  |String  | Nombre del elemento |

Una vez se carga el documento en los objetos .Net se disponen de las siguientes opciones comunes: 

Para convertir el objeto de nuevo a la cadena se realiza de la siguiente forma:

    string content = document.ToString();
 **Nota: Si al objeto se le han realizado cambios en los datos, esto se verá reflejado en el nuevo contenido** 

Para acceder a las secciones cargadas dentro del objeto se puede realizar de las siguientes formas: 

 - Índice: Se utiliza para obtener una sola sección, recibe como
   parámetro el nombre de la sección, ejemplo: document ["UNH"] 
   
 - Método: Se utiliza para obtener secciones que pueden tener varias
   ocurrencias, recibe como parámetro el nombre de la sección, ejemplo:
   document.GetSegments("LIN").**Nota: En caso de obtener una sección que se repite por medio del índice, siempre retorna la primera ocurrencia**

 • Para acceder a las secciones que se configuran como grupos, se realiza como en los dos puntos anteriores pero anidando la sección padre con la hija 
        o Índice: document ["NAD"] ["RFF"] 
        o Método: ocument["NAD"].GetSegments ("RFF")

**Nota: Se debe tener en cuenta que si la sección configurada no está dentro del documento cargado, el valor retornado es Null para el índice y Array vacío para el método**

 ### 5.2 EDI
 
 Nombre dll: Carvajal.SI.PGA.Mapper.EDI.dll 
Define un archivo con formato EDI, la configuración se realiza de la siguiente forma: 

 - Seccion: El nombre de la sección debe ser el nombre del segmento
   EDI.
   
 - Elemento: No aplica para EDI, **si se configuran no se tienen en
   cuenta**

 
EDISegmentBase: Clase base de todos los segmentos EDI, una vez se carga el documento todos los segmentos quedan de este tipo, en caso de requerir trabajar directamente con el tipo específico se debe hacer el CAST, ejemplo: UNH unh = (UNH)document["UNH"]

Ejemplo de la configuración:

    <!--Configuración de un segmentos-->
    <Seccion Nombre="ISA" />
    <!--Configuración de un grupo de segmentos-->
    <Seccion Nombre="LIN">
     <Seccion Nombre="REF" />
    </Seccion>

### 5.3 ANSI

Nombre dll: Carvajal.SI.PGA.Mapper.ANSI.dll Define un archivo con formato ANSI, la configuración se realiza de la siguiente forma: 

 - Seccion: El nombre de la sección debe ser el nombre del segmento
   ANSI. 
   
 - Elemento: No aplica para ANSI, **si se configuran no se tienen en
   cuenta**.

 ANSISegmentBase: Clase base de todos los segmentos ANSI, una vez se carga el documento todos los segmentos quedan de este tipo, en caso de requerir trabajar directamente con el tipo específico se debe hacer el CAST, ejemplo: ISA isa = (ISA)document["ISA"]

Ejemplo de la configuración

    <!--Configuración de un segmentos-->
    <Seccion Nombre="ISA" />
    <!--Configuración de un grupo de segmentos-->
    <Seccion Nombre="LIN">
     <Seccion Nombre="REF" />
    </Seccion>

Para carga un documento se realiza de la siguiente forma:

    ANSIDocument document = new ANSIDocument();
    document.LoadContent(config, ansiContent);

### 5.4 Generic Flat File Nombre dll:

 Carvajal.SI.PGA.Mapper.GenericFlatFile.dll Define un archivo genérico con formato plano, la configuración se realiza de la siguiente forma:
 
 - Seccion: El nombre de la sección debe ser el identificador de la
   sección, es decir el valor del primer campo.
   
 - Elemento: Define  los campos de la sección.
    
FlatFileSegmentBase: Clase base de todos los segmentos del archivo plano, una vez se carga el documento todos los segmentos quedan de este tipo.
 
Actualmente se soportan 2 tipos de archivos planos: 
- Con separador de campos
-   De ancho fijo

Ejemplo de la configuración con separador

    <Seccion Nombre="ENC" Separador=",">
     <Elemento Nombre="ENC_0" />
     <Elemento Nombre="ENC_1" />
     <Elemento Nombre="ENC_2" />
     <Elemento Nombre="ENC_3" />
    </Seccion>
    <Seccion Nombre="DET" Separador=",">
     <Elemento Nombre="DET_0" />
     <Elemento Nombre="DET_1" />
     <Elemento Nombre="DET_2" />
     <Elemento Nombre="DET_3" />
     <Seccion Nombre="LOC" Separador=",">
     <Elemento Nombre="LOC_0" />
     <Elemento Nombre="LOC_1" />
     </Seccion>
    </Seccion>

Ejemplo de la configuración con ancho fijo

    <Seccion Nombre="ENC">
     <Elemento Nombre="ENC_0" PosInicial="0" Longitud="3" />
     <Elemento Nombre="ENC_1" PosInicial="3" Longitud="5" />
     <Elemento Nombre="ENC_2" PosInicial="8" Longitud="5" />
     <Elemento Nombre="ENC_3" PosInicial="13" Longitud="8" />
    </Seccion>
    <Seccion Nombre="DET">
     <Elemento Nombre="DET_0" PosInicial="0" Longitud="3" />
     <Elemento Nombre="DET_1" PosInicial="3" Longitud="3" />
     <Elemento Nombre="DET_2" PosInicial="6" Longitud="12" />
     <Elemento Nombre="DET_3" PosInicial="18" Longitud="2" />
     <Seccion Nombre="LOC">
     <Elemento Nombre="LOC_0" PosInicial="0" Longitud="3" />
     <Elemento Nombre="LOC_1" PosInicial="3" Longitud="3" />
     </Seccion>
    </Seccion>

Para carga un documento se realiza de la siguiente forma:

    FlatFileDocument document = new FlatFileDocument();
    document.LoadContent(config, flatFileContent);

## 6. Splitters 

PGA Mapper provee métodos para dividir el archivo cuando contiene múltiples documentos.

 ### 6.1 EDI 

Teniendo en cuenta la estructura de un documento EDI se cuentan con los siguientes métodos para dividirlo: 

 -  SplitEDIByUNB: Divide el EDI cuando vienen múltiples UNB.
 -  SplitEDIByUNH: Divide el EDI cuando vienen múltiples UNH.
 -  SplitEDIByUNBAndUNH: Divide el EDI por UNH y por UNB.

Ejemplo:

    List<string> lst = EDIExtension.SplitEDIByUNBAndUNH(ediContent, new
    EDISegmentProperties());
    
### 6.2 ANSI

 Teniendo en cuenta la estructura de un documento ANSI se cuentan con los siguientes métodos para dividirlo: 

 -  SplitANSIByISA: Divide el ANSI cuando vienen múltiples ISA.
 -  SplitANSIByST: Divide el ANSI cuando vienen múltiples ST. 
 -  SplitANSIByISAAndST: Divide el ANSI por ST y por ISA.

Ejemplo:

    List<string> lst = ANSIExtension.SplitANSIByISAAndST(ansiContent, new
    ANSISegmentProperties());


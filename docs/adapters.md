# Owl Adapters

## 1. Introduccion

Owl Adapter es un conjunto de librerias que permite cargar documentos en objetos .Net mediante un archivo de configuración (Owl Input Config), también provee los métodos para obtener y modificar los valores de los objetos.

## 2. Estructura de Owl Adapter

Para poder trabajar con Owl Adapter se requiere lo siguiente: 

 - **Owl Input Config**: En este archivo se configura la estructura de entrada en sintaxis XOML.
   
 - **OwlConfig**: Clase donde se carga el Owl Input Config. Se debe usar la clase específica de cada Adapter 
 
 - **Document**: Se debe usar la clase específica de cada tipo de Adapter.

### 2.1 Owl Input Config
  
En este archivo es donde se define la estructura del archivo a cargar, el siguiente ejemplo muestra un archivo de configuración:

    <?xml version="1.0" encoding="utf-8" ?>
    <owl version="1.0">
      <section id="ENC" name="ENC">
        <element name="enc_0" start-position="0" length="3" />
        <element name="enc_1" start-position="3" length="5" />
        <element name="enc_2" start-position="8" length="5" />
        <element name="enc_3" start-position="13" length="8" />
      </section>

      <section id="DET" name="DET">
        <element name="det_0" start-position="0" length="3" />
        <element name="det_1" start-position="3" length="3" />
        <element name="det_2" start-position="6" length="12" />
        <element name="det_3" start-position="18" length="2" />

        <section id="LOC" name="LOC">
          <element name="loc_0" start-position="0" length="3" />
          <element name="loc_1" start-position="3" length="3" />
        </section>
      </section>
    </owl>

### 2.2 OwlConfig

Clase que representa la configuración en objetos .Net, cada adaptador implementa su propia clase ()

    XmlDocument xml = new XmlDocument();
    xml.Load(@"C:\path\owl.xml");
    OwlConfig owlConfig = XmlExtension.Deserialize<OwlConfig>(xml);

### 2.3 Document

Cada Owl Adapter expone su propia clase para la carga del documento (FlatFileDocument, EDIDocument, ANSIDocument), con estas clases se pueden cargar los documentos en objetos y retornar a la cadena original, adicional es posible al modificar los objetos y generar la nueva cadena del documento con los cambios realizados.

## 3. Manejo de Excepciones

Owl Adapter valida el archivo a cargar con el Owl Input Config, dentro de estas validaciones se pueden generar las siguientes excepción: 

 - OwlAdapterException
 - OwlContentException
 - OwlDataTypeException
 - OwlLengthException
 - OwlRequiredException

Las excepciónes dan información detallada de en qué parte del proceso se generó el error.
 
### 3.1 OwlAdapterException 
 
Excepción que se produce cuando se presenta algún error de configuración en el adaptador.

### 3.2 OwlContentException 
 
Excepción que se produce cuando se presenta algún error en la carga del contenido del documento, en caso de que el error sea porque el documento no cumple con la estructura configurada, en el mensaje se puede obtener la posición donde se produjo el error.

### 3.3 OwlDataTypeException 
 
Excepción que se produce cuando un campo no cumple con el tipo de dato configurado.

### 3.4 OwlLengthException 
 
Excepción que se produce cuando un campo excede la longitud configurada.

### 3.5 OwlRequiredException 
 
Excepción que se produce cuando un campo obligatorio no se envia en el documento de entrada.

## 4. Formatos Owl Adapter

Owl Adapter provee algunos formatos de entrada para la carga de los datos.

### 4.1 Configuración General 

A continuación se detalla la configuración general para todos los formatos.

Configuración a nivel de Sección:

| **Propiedad** |**Tipo Dato** |**Obligatoriedad**| **Descripcion** |
|--|--|--|--|
|id  |String |Opcional| Id de la sección |
|name  |String  |Obligatorio| Nombre de la sección |
|description |String|Opcional|Descripcion de la sección|

Cuando se requiera configurar grupos de secciones, se deben incluir las secciones hijas como subetiquetas de la sección principal 

Una vez se carga el documento en los objetos .Net se disponen de las siguientes opciones comunes: 

Para convertir el objeto de nuevo a la cadena se realiza de la siguiente forma:

    string content = document.ToString();

 **Nota: Si al objeto se le han realizado cambios en los datos, esto se verá reflejado en el nuevo contenido** 

Para acceder a las secciones cargadas dentro del objeto se puede realizar de las siguientes formas: 

 - Índice: Se utiliza para obtener una sola sección, recibe como parámetro el nombre de la sección, ejemplo: document ["section"] 

 - Método: Se utiliza para obtener secciones que pueden tener varias ocurrencias, recibe como parámetro el nombre de la sección, ejemplo: document.GetSegments("section").
 
 **Nota: En caso de obtener una sección que se repite por medio del índice, siempre retorna la primera ocurrencia**

 - Para acceder a las secciones que se configuran como grupos, se realiza como en los dos puntos anteriores pero anidando la sección padre con la hija

   - Índice: document["section_parent"]["section_child"]
   - Método: document["section_parent"].GetSegments("section_child")

**Nota: Se debe tener en cuenta que si la sección configurada no está dentro del documento cargado, el valor retornado es Null para el índice y Array vacío para el método**

### 4.2 EDI
 
Nombre dll: Fiive.Owl.EDI.dll

Define un archivo con formato EDI, la configuración se realiza de la siguiente forma: 

 - Seccion: El nombre de la sección debe ser el nombre del segmento EDI.

 - Elemento: No aplica para los documentos EDI, si se configura se ignora.
 
EDISegmentBase: Clase base de todos los segmentos EDI, una vez se carga el documento todos los segmentos se instancian con este tipo, en caso de requerir trabajar directamente con el tipo específico se debe hacer el CAST, ejemplo: UNH unh = (UNH)document["UNH"]

Ejemplo de la configuración:

    <!--Configuración de un segmentos-->
    <Seccion Nombre="ISA" />
    <!--Configuración de un grupo de segmentos-->
    <Seccion Nombre="LIN">
     <Seccion Nombre="REF" />
    </Seccion>

Para carga un documento se realiza de la siguiente forma:

    EDIDocument document = new EDIDocument();
    document.LoadContent(config, ediContent);

### 4.3 ANSI

Nombre dll: Fiive.Owl.ANSI.dll

Define un archivo con formato ANSI, la configuración se realiza de la siguiente forma: 

 - Seccion: El nombre de la sección debe ser el nombre del segmento ANSI.

 - Elemento: No aplica para los documentos ANSI, si se configura se ignora.

 ANSISegmentBase: Clase base de todos los segmentos ANSI, una vez se carga el documento todos los segmentos se instancian con este tipo, en caso de requerir trabajar directamente con el tipo específico se debe hacer el CAST, ejemplo: ISA isa = (ISA)document["ISA"]

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

### 4.4 Flat File

Nombre dll: Fiive.Owl.FlatFile.dll

Define un archivo genérico con formato plano, la configuración se realiza de la siguiente forma:
 
 - Seccion: El nombre de la sección debe ser el identificador de la sección, es decir el valor del primer campo.
   
 - Elemento: Define los campos de la sección.
    
Section: Clase base de todos los segmentos del archivo plano, una vez se carga el documento todos los segmentos se instancian con este tipo.

Adicional a la configuración estándar soporta los siguientes atributos. 
Configuración a nivel de Sección:

| **Propiedad** |**Tipo Dato** |**Obligatoriedad**| **Descripcion** |
|--|--|--|--|
|separator|String|Opcional|Separador de campos|

Configuración a nivel de Elemento:

| **Propiedad** |**Tipo Dato** |**Obligatoriedad**| **Descripcion** |
|--|--|--|--|
|name|String|Obligatorio|Nombre del elemento|
|start-position|Int|Opcional|Posición inicial del valor, aplica para archivos de ancho fijo|
|length|Int|Opcional|Largo del valor, si se configura el separator a nivel de section, se valida que el valor no supere la longitud configurada, si no se configura el separator es la longitud a tomar a partir del valor configurado en start-position|
|data-type|Enum (int, decimal y string)|Opcional|Tipo de dato del campo|
|required|Boolean|Opcional|Indica si el campo es obligatorio|
 
Actualmente se soportan 2 tipos de archivos planos: 

- Con separador de campos
- De ancho fijo

Ejemplo de la configuración con separador

    <?xml version="1.0" encoding="utf-8" ?>
    <owl version="1.0">
      <section id="ENC" name="ENC" separator=";">
        <element name="enc_0" length="3" />
        <element name="enc_1" length="5" />
        <element name="enc_2" length="5" />
        <element name="enc_3" length="8" />
      </section>

      <section id="DET" name="DET" separator=";">
        <element name="det_0" length="3" />
        <element name="det_1" length="3" />
        <element name="det_2" length="12" />
        <element name="det_3" length="2" />

        <section id="LOC" name="LOC" separator=";">
          <element name="loc_0" length="3" />
          <element name="loc_1" length="2" />
        </section>
      </section>
    </owl>

Ejemplo de la configuración con ancho fijo

    <?xml version="1.0" encoding="utf-8" ?>
    <owl version="1.0">
      <section id="ENC" name="ENC">
        <element name="enc_0" start-position="0" length="3" />
        <element name="enc_1" start-position="3" length="5" data-type="int" />
        <element name="enc_2" start-position="8" length="5" />
        <element name="enc_3" start-position="13" length="8" />
      </section>

      <section id="DET" name="DET">
        <element name="det_0" start-position="0" length="3" />
        <element name="det_1" start-position="3" length="3" />
        <element name="det_2" start-position="6" length="12" />
        <element name="det_3" start-position="18" length="2" />

        <section id="LOC" name="LOC">
          <element name="loc_0" start-position="0" length="3" />
          <element name="loc_1" start-position="3" length="3" />
        </section>
      </section>
    </owl>

Para carga un documento se realiza de la siguiente forma:

    FlatFileDocument document = new FlatFileDocument();
    document.LoadContent(config, flatFileContent);

# API-REST

El API REST, provee la funcionalidad completa para la administración del proceso de
envío de mensajes ya que permite administrar sus contactos, grupo y mensajes de forma
integral con la aplicación que se desee implementar.

# Autenticación
El primer proceso fundamental dentro del proceso de configuración de la aplicación
consiste en la “Autenticación de las interfaces REST”, para lo cual se requiere que de
forma obligatoria cada solicitud realizada a la plataforma se encuentre validada o
“firmada”, utilizando para el efecto el API KEY y el API SECRET correspondiente a la
cuenta.

Es importante resaltar que dichas aplicaciones (API KEY y API SECRET) deben mantenerse
siempre en secreto y bajo ninguna circunstancia deben hacerse públicas. Sólo la
plataforma y el cliente contratante del servicio tienen acceso y conocimiento de esta
información por lo que la misma será utilizada siempre para verificar la identidad del
usuario que hace las solicitudes al API.

Para poder realizar el proceso de autenticación y autorización de las solicitudes
efectuadas al API, cada solicitud debe contener, en los encabezados del protocolo HTTP
una firma generada con información específica para dicha solicitud. Esta firma debe ser
generada utilizando para el efecto una combinación del API Key, el API Secret,
información de la solicitud, fecha y hora del envío, y el algoritmo de cifrado HMACSHA1.

# Generación De La Firma
Como hemos mencionado la firma es requerida de forma obligatoria para determinar la
autenticidad de una solicitud. Se debe tener siempre en cuenta que, debido a que la
firma incluye información de la fecha y hora y los datos en la solicitud, una misma firma
nunca podrá ser utilizada para hacer diferentes llamadas. Con esta característica
evitamos que un tercero pueda interceptar un paquete en la red de forma anómala y
utilizar la firma para hacer otros envíos.

A continuación se describen los pasos a seguir de forma ordenada para poder generar la
firma de forma exitosa:

1. Ordenar los parámetros de la solicitud en orden alfabético.
2. Concatenar los parámetros utilizando el formato llave=valor. Si la operación es de
tipo POST o PUT también se deberá concatenar el contenido del cuerpo de la
solicitud, tal y como van a ser enviados. Ejemplo: limit=10 start=0 {"data":"del
post"}
3. Generar una cadena con la fecha utilizado para el efecto el estándar para el
protocolo HTTP, con formato RFC 1123. Ejemplo: Thu, 12 Jul 2014 19:55:55 GMT

4. Concatenar el API Key, la fecha y los parámetros generados del paso 2, de manera
que la estructura resultante quede de la siguiente forma: <API Key><fecha><parámetros>.

5. Ejemplo

         1d4e705080edec039fe580dd26fd0027Thu, 12 Jul 2014 19:55:55 GMTlimit=10start=0{"data":"del post"}

6. Cifrar esta cadena utilizando el algoritmo de encriptación HMAC-SHA1. La base
para el cifrado será el API Secret.
7. El cifrado binario resultante, debe ser convertido a una cadena BASE64. Esta
cadena será la firma para dicha solicitud.

# Encabezado De Autenticación HTTP
El siguiente paso consiste en realizar la autenticación de forma exitosa, para lo cual
debe incluirse un encabezado a la consulta HTTP a realizar. Para ello se debe incluir la
fecha y el encabezado de autorización con la firma generada para dicha consulta. Todas
las solicitudes deben llevar el encabezado con la firma de autenticación.

El encabezado HTTP lleva la llave con la etiqueta “Authorization” y como valor debe
llevar una cadena que incluye el API Key y la firma generada con el formato IM <API
Key>:<firma>
  
Nota: La fecha debe ser la misma fecha utilizada en la generación de la firma.
El encabezado final de HTTP se verá de la siguiente manera:

        Date: Thu, 12 Jul 2014 19:55:55 GMT
        Authorization: IM1D4E705080EDEC039FE580DD26FD0027:WM/16HWXG46H9WEVFNWJRE2F2

# Manejo de Errores
Todas las solicitudes efectuadas a la plataforma retornan un código de estado como
parte del protocolo HTTP. En la interfaz REST, de igual manera, todas las llamadas
retornan un estado HTTP que indica el resultado de la operación.

# Solicitud Exitosa
Si la solicitud enviada fue exitosa se retornará el código “200 OK”. Adicionalmente, de
acuerdo a la operación realizada, la respuesta podrá contener o no los datos, en el
cuerpo de la misma. Estos pueden o no ser procesados, según la necesidad del cliente.

# Solicitud Con Error
Todas las solicitudes enviadas a la plataforma y que tuvieron algún error, retornarán un
código diferente a “200 OK” como estado genérico HTTP. Adicionalmente el cuerpo de la
respuesta contendrá un JSON con el código específico y una descripción del error
generado. El JSON de respuesta tendrá el siguiente formato:

{ code: '40301', error: {% trans 'Ha excedido el número de mensajes contratado' % }
En caso de recibir un estado HTTP de error como respuesta a la consulta generada, se
recomienda procesar el cuerpo el mensaje para obtener más información sobre la causa
de la operación fallida.

# Contactos
Una de las ventajas de la interfaz REST es que provee la funcionalidad para administrar
los contactos de manera remota, lo cual permite que se mantenga en sincronía con su
aplicación. Se puede agregar, modificar, eliminar y listar los contactos de la cuenta.
Nota: Para todos los recursos de la interfaz REST, debe incluirse la firma de
autenticación.

# Obtener Un Listado de Contactos

GET /contacts
Esta operación se utiliza para obtener el listado de contactos asociados a la cuenta. Si
desea listar los contactos asociados a un grupo deberá referirse a las operaciones de
grupo.

Solicitud
Parámetros: limit | Tipo: Numérico (Opcional)
Descripción: Límite de registros a retornar en la consulta. Valor por defecto: 50. Valor máximo: 1000.

Parámetros: start | Tipo: Numérico (Opcional)
Descripción: Parámetro de corrimiento para la cuenta de registros a retornar en la consulta. El valor inicial es cero.

Parámetros: query | Tipo: Texto (Opcional)
Descripción: Filtro general para la búsqueda. La consulta retornará cualquier registro que cuente con este valor en el msisdn o en el primer y último nombre.

Parámetros: status | Tipo: Texto (Opcional)
Descripción: Filtro para el estado en el que se encuentran los contactos. Posible valores: SUSCRIBED, CONFIRMED, CANCELLED, INVITED. Valor por defecto: SUSCRIBED, CONFIRMED.

Parámetros: Booleano (Opcional) | Tipo: Texto (Opcional)
Descripción: Si esta opción es verdadera el retorno de la llamada contendrá una versión resumida de los contactos. Esta versión sólo contiene el teléfono y nombre, dejando fuera los demás campos. Posibles valores: 1 o 0. Valor por defecto: 0.

----->  Respuesta

La consulta retornará un listado de objetos tipo “contactos”. Si no existieran resultados
para los criterios especificados, se retornará una lista vacía.

----->  Ejemplo de solicitud

         Utilizando HTTP:
         GET /api/rest/contacts?query=jose&limit=10 HTTP/1.1
         Accept-Encoding: identity
         Content-Length: 0
         Connection: close
         Date: Thu, 07 Aug 2014 20:47:07 GMT
         Content-Type: application/x-www-form-urlencoded
         Authorization: IM1d4e705080edec039fe580dd26fd0027:WM/16HwXg46H9WevfnWjre2F2

         Utilizando JAVA:
         List contactStatuses = null;
         String query = "";
         Integer start = 0;
         Integer limit = 10;
         boolean shortResults = true;
         Contacts instance = new Contacts(
         "api key",
         "api secret",
         "http://apps01-tigo-csms.im.local:8101/");
         ApiResponse> result = instance.getList(contactStatuses, query, start, limit,
         shortResults);
         if (result.isOk()) {
         List list = result.getResponse();
         } else {
         System.out.print("http code: " + result.getHttpCode());
         System.out.print("api code: " + result.getErrorCode());
         System.out.print("description: " + result.getErrorDescription());
         }

----->  Respuesta

    Utilizando HTTP:
    HTTP/1.1 200 OK
    Date: Thu, 07 Aug 2014 20:47:07 GMT
    Connection: Keep-Alive
    Transfer-Encoding: chunked
    Content-Type: application/json

    Utilizando JAVA:
    [{"msisdn":"50212345678","status":"SUSCRIBED","tags":
    [""],"phone_number":"12345678","country_code":"502","first_name":"Jose","last_na
    me":"Perez","full_name":"Joseperez","added_from":"WEB_FORM","custom_field_1":
    "Guatemala","custom_field_2":"","custom_field_3":"","custom_field_4":"","custom_f
    ield_5":""}]

# Obtener Un Contacto Con El MSIDSN
GET /contacts/:msisdn
Esta operación se utiliza para obtener directamente un contacto en base al MSISDN
(Número de teléfono con código internacional de país).

----->  Solicitud

Parámetros :msisdn | Tipo: Numérico | Descripción: Número de teléfono en formato internacional.
Incluye el código de país (Ejemplo: 502123435678)

----->  Respuesta

De existir un contacto con el número de teléfono enviado, la consulta retornará un
objeto de tipo “contacto”. Si no existiera resultado para el contacto enviado se
retornará un código de error.

----->  Ejemplo de solicitud Utilizando HTTP:

        GET /api/rest/contacts/50212345678 HTTP/1.1
        Accept-Encoding: identity
        Content-Length: 0
        Connection: close
        Date: Thu, 07 Aug 2014 20:47:07 GMT
        Content-Type: application/x-www-form-urlencoded
        Authorization: IM1d4e705080edec039fe580dd26fd0027:WM/16HwXg46H9WevfnWjre2F2

        Utilizando el SDK Java:
        String msisdn = "50212345678";
        Contacts instance = new Contacts(
        "api key",
        "api secret",
        "http://apps01-tigo-csms.im.local:8101/");
        ApiResponse result = instance.getByMsisdn(msisdn);
        if (result.isOk()) {
        ContactJsonObject contact = result.getResponse();
        } else {
        System.out.print("http code: " + result.getHttpCode());
        System.out.print("api code: " + result.getErrorCode());
        System.out.print("description: " + result.getErrorDescription());
        }

----->  Respuesta

        Parámetros Tipo Descripción
        :msisdn Numérico Número de teléfono en formato internacional.
        Incluye el código de país ( Ejemplo:502123435678)
        Utilizando HTTP:
        HTTP/1.1 200 OK
        Date: Thu, 07 Aug 2014 20:47:07 GMT
        Connection: Keep-Alive
        Transfer-Encoding: chunked
        Content-Type: application/json
        Utilizando el SDK Java:
        {"msisdn":"50212345678","status":"SUSCRIBED","tags":
        [""],"phone_number":"12345678","country_code":"502","first_name":"Jose","last_na
        me":"Perez","full_name":"JosePerez","added_from":"WEB_FORM","custom_field_1":
        "Guatemala","custom_field_2":"","custom_field_3":"","custom_field_4":"","custom_f
        ield_5":""}




# Crear Un Contacto Nuevo
POST/contacts/:msisdn
Esta operación se utiliza para crear un contacto nuevo.

----->  Solicitud

Parámetros :msisdn | Tipo: Numérico | Descripción: Número de teléfono en formato internacional.
Incluye el código de país (Ejemplo:502123435678)

----->  Post-Data

Se debe enviar el JSON correspondiente a un objeto de tipo contacto.

----->  Respuesta

Si la operación de agregar el contacto se realiza con éxito, se retornará status 200 OK y
el JSON del objeto tipo contacto. En caso contrario se responderá un status de error.

----->  Solicitud

----->  Utilizando HTTP:


    POST /api/rest/contacts/50212345678 HTTP/1.1
    Accept-Encoding: identity
    Content-Length: 220
    Connection: close
    Date: Thu, 07 Aug 2014 20:47:07 GMT
    Content-Type: application/x-www-form-urlencoded
    
----->  Utilizando SDK JAVA:


    String countryCode = "502";
    String msisdn = "50212345678";
    String firstName = "Jose";
    String lastName = "Perez";
    Contacts instance = new Contacts(
    "api key",
    "api secret",
    "http://apps01-tigo-csms.im.local:8101/");
    ApiResponse> result = instance.add(countryCode, msisdn, firstName, lastName);
    if (result.isOk()) {
    ContactJsonObject contact = result.getResponse();
    } else {
    System.out.print("http code: " + result.getHttpCode());
    System.out.print("api code: " + result.getErrorCode());
    System.out.print("description: " + result.getErrorDescription());
    }

----->  Respuesta 

    :msisdn Numérico Número de teléfono en formato internacional.
    Incluye el código de país ( Ejemplo:502123435678)
    
----->  Utilizando HTTP:


    HTTP/1.1 200 OK
    Date: Thu, 07 Aug 2014 20:47:07 GMT
    Connection: Keep-Alive
    Transfer-Encoding: chunked
    Content-Type: application/json
   
----->  Utilizando SDK JAVA:


    Authorization:IM1d4e705080edec039fe580dd26fd0027:WM/
    16HwXg46H9WevfnWjre2F21o={"msisdn":"50212345678","phone_number":"1234567
    8","country_code":"502","first_name":"Jose","last_name":"Perez","custom_field_1":
    "Guatemala","custom_field_2":"","custom_field_3":"","custom_field_4":"","custom_f
    ield_5":""}
    
    
   # Actualiza un contacto
PUT /contacts/:msisdn
   
Esta operación se utiliza para actualizar un contacto existente. El contacto se identifica
con el :MSISDN (Número de teléfono con código internacional de país).

-----> Solicitud

Parámetros :msisdn | Tipo: Numérico | Descripción: Número de teléfono en formato internacional.
Incluye el código de país (Ejemplo:502123435678)

-----> Post-Data

Se debe enviar el JSON correspondiente a un objeto de tipo contacto.

-----> Respuesta

Si la operación de actualización es realizada de forma exitosa, se retornará status 200
OK y el JSON del objeto tipo contacto. En caso contrario se responderá un status de
error.

# Ejemplo de solicitud

-----> Utilizando HTTP:
         
         
         PUT /api/rest/contacts/50212345678 HTTP/1.1
         Accept-Encoding: identity
         Content-Length: 220
         Connection: close
         Date: Thu, 07 Aug 2014 20:47:07 GMT
         Content-Type: application/x-www-form-urlencoded
         
         
-----> Utilizando SDK JAVA:
         
         String countryCode = "502";
         String msisdn = "50212345678";
         String firstName = "Jose";
         String lastName = "Perez";
         Contacts instance = new Contacts(
         "api key",
         "api secret",
         "http://apps01-tigo-csms.im.local:8101/");
         ApiResponse> result = instance.update(countryCode, msisdn, firstName,
         lastName);
         if (result.isOk()) {
         ContactJsonObject contact = result.getResponse();
         } else {
         System.out.print("http code: " + result.getHttpCode());
         System.out.print("api code: " + result.getErrorCode());
         System.out.print("description: " + result.getErrorDescription()); }

-----> Respuesta Utilizando HTTP:

         HTTP/1.1 200 OK
         Date: Thu, 07 Aug 2014 20:47:07 GMT
         Connection: Keep-Alive
         Transfer-Encoding: chunked
         Content-Type: application/json

-----> Respuesta Utilizando el SDK JAVA:

         Authorization:IM1d4e705080edec039fe580dd26fd0027:WM/
         16HwXg46H9WevfnWjre2F21o={"msisdn":"50212345678","phone_number":"1234567
         8","country_code":"502","first_name":"Jose","last_name":"Perez","custom_field_1":
         "Guatemala","custom_field_2":"","custom_field_3":"","custom_field_4":"","custom_f
         ield_5":""}
         
         
# Elimina Un Contacto
DELETE /contacts/:msisdn

Esta operación se utiliza para dar de baja un contacto existente. El contacto se
identifica con el :MSISDN (Número de teléfono con código internacional de país).

-----> Solicitud

Parámetros :msisdn | Tipo: Numérico | Descripción: Número de teléfono en formato internacional.
Incluye el código de país (Ejemplo: 502123435678)

-----> Respuesta

Si la operación de eliminación es realizada de forma exitosa, se retornará status 200 OK
y el JSON del objeto tipo contacto. En caso contrario se responderá con status de error.

# Ejemplo de solicitud

-----> Utilizando HTTP:

         DELETE /api/rest/contacts/50212345678 HTTP/1.1
         Accept-Encoding: identity
         Content-Length: 0
         Connection: close
         Date: Thu, 07 Aug 2014 20:47:07 GMT
         Content-Type: application/x-www-form-urlencoded
         
-----> Utilizando SDK JAVA:

         String msisdn = "50212345678";
         Contacts instance = new Contacts(
         "api key",
         "api secret",
         "http://apps01-tigo-csms.im.local:8101/");
         ApiResponse> result = instance.delete(msisdn);
         if (result.isOk()) {
         ContactJsonObject contact = result.getResponse();
         } else {
         System.out.print("http code: " + result.getHttpCode());
         System.out.print("api code: " + result.getErrorCode());
         System.out.print("description: " + result.getErrorDescription());
         }
         
-----> Respuesta Utilizando HTTP:

         HTTP/1.1 200 OK
         Date: Thu, 07 Aug 2014 20:47:07 GMT
         Connection: Keep-Alive
         Transfer-Encoding: chunked
         Content-Type: application/json

-----> Respuesta Utilizando SDK JAVA

         Authorization:IM1d4e705080edec039fe580dd26fd0027:WM/
         16HwXg46H9WevfnWjre2F2
         
         
# Obtiene el Listado de Grupos de un Contacto
GET /contacts/:msisdn/groups

Esta operación se utiliza para obtener el listado de grupos a los que pertenece un
contacto

-----> Solicitud

Parámetros :msisdn | Tipo: Numérico | Descripción: Número de teléfono en formato internacional.
Incluye el código de país (Ejemplo: 502123435678)

-----> Respuesta

La consulta retornará un arreglo de objetos tipo Grupo. Si el contacto no tiene grupos
asociados, se retornará un arreglo vacío.

# Ejemplo de solicitud

-----> Utilizando HTTP:

         GET /api/rest/contacts/50212345678/groups HTTP/1.1
         Accept-Encoding: identity
         Content-Length: 0
         Connection: close
         Date: Thu, 07 Aug 2014 20:47:07 GMT
         Content-Type: application/x-www-form-urlencoded

-----> Utilizando SDK JAVA:

         String msisdn = "50212345678";
         Contacts instance = new Contacts(
         "api key",
         "api secret",
         "http://apps01-tigo-csms.im.local:8101/");
         ApiResponse> result = instance.getGroupList(msisdn);
         if (result.isOk()) {
         List list = result.getResponse();
         } else {
         System.out.print("http code: " + result.getHttpCode());
         System.out.print("api code: " + result.getErrorCode());
         System.out.print("description: " + result.getErrorDescription());
         }

-----> Respuesta

Parámetros Tipo Descripción
:msisdn Numérico Número de teléfono en formato internacional.
Incluye el código de país (Ejemplo:502123435678)

-----> Utilizando HTTP

         HTTP/1.1 200 OK
         Date: Thu, 07 Aug 2014 20:47:07 GMT
         Connection: Keep-Alive
         Transfer-Encoding: chunked
         Content-Type: application/json

-----> Utilizando SDK JAVA

         Authorization:IM1d4e705080edec039fe580dd26fd0027:WM/
         16HwXg46H9WevfnWjre2F2
         [
         { short_name: 'ventas', name: 'Ventas', description: 'Grupo de Ventas', members: {
         total: 5, pending: 1, confirmed: 4} },
         ...
         ]

# Grupos

La interfaz REST permite la administración de grupos de contactos de manera remota,
permitiendo organizar sus contactos de forma eficiente para facilitar los envíos de
mensajes. Se pueden agregar, modificar, eliminar y listar los grupos de contactos de la
cuenta.

Nota: Para todos los recursos de la interfaz REST, debe incluirse la firma de
autenticación.

# Obtiene El Listado de Grupos
GET /groups

Esta operación se utiliza para obtener el listado de grupos de contactos de la cuenta.

-----> Solicitud

Parámetros: limit
Tipo: Numérico (Opcional)
Descripción: Límite de registros a retornar en la consulta. Valor por defecto: 50. Valor máximo: 1000.

Parámetros: start
Tipo: Numérico (Opcional)
Descripción: Parámetro de corrimiento para la cuenta de registros a retornar en la consulta. El valor inicial es cero.

Parámetros: Query
Tipo: Texto (Opcional)
Descripción: Filtro general para la búsqueda. La consulta retornará cualquier registro que cuente con este valor en el nombre o nombre corto

Parámetros: shortResults
Tipo: Booleano (Opcional)
Descripción: Si esta opción es verdadera el retorno de la llamada contendrá una versión resumida de los contactos. Esta versión sólo contiene el teléfono y nombre, dejando fuera los demás campos. Posibles valores: 1 o 0. Valor por defecto: 0.

-----> Respuesta

La consulta retornará un listado de objetos tipo grupo. Si no existen resultados para los
criterios especificados, se retornará una lista vacía.

# Ejemplo de solicitud

-----> Utilizando HTTP:

         GET /api/rest/groups?query=ventas&limit=10 HTTP/1.1
         Accept-Encoding: identity
         Content-Length: 0
         Connection: close
         Date: Thu, 07 Aug 2014 20:47:07 GMT
         Content-Type: application/x-www-form-urlencoded

-----> Utilizando SDK JAVA:

         String query = "";
         Integer start = 0;
         Integer limit = 10;
         boolean shortResults = false;
         Groups instance = new Groups(
         "api key",
         "api secret",
         "http://apps01-tigo-csms.im.local:8101/");
         ApiResponse<List<GroupJsonObject>> result = instance.getList(query, start,
         limit, shortResults);
         if (result.isOk()) {
         List<GroupJsonObject> list = result.getResponse();
         } else {
         System.out.print("http code: " + result.getHttpCode());
         System.out.print("api code: " + result.getErrorCode());
         System.out.print("description: " + result.getErrorDescription());
         }

-----> Respuesta Utilizando HTTP:

         HTTP/1.1 200 OK
         Date: Thu, 07 Aug 2014 20:47:07 GMT
         Connection: Keep-Alive
         Transfer-Encoding: chunked
         Content-Type: application/json

-----> Utilizando SDK JAVA:

         Authorization:IM1d4e705080edec039fe580dd26fd0027:WM/
         16HwXg46H9WevfnWjre2F2
         [{ short_name: 'ventas', name: 'Ventas', description: 'Grupo de Ventas', members:
         { total: 5, pending: 1, confirmed: 4} }]


# Crea un Grupo Nuevo
POST /groups/:short_name

Esta operación crea un grupo de contactos nuevo.

-----> Solicitud

Parámetros :short_name
Tipo: Texto
Descripción: El nombre corto asignado al grupo

-----> Post-Data

Se debe enviar el JSON correspondiente a un objeto tipo grupo.

-----> Respuesta

Si la operación de creación es finalizada con éxito se retornará status 200 OK y el JSON
del objeto de tipo grupo. En caso contrario se responderá con status de error.

# Ejemplo de solicitud

-----> Utilizando HTTP:

         POST /api/rest/groups/ventas HTTP/1.1
         Accept-Encoding: identity
         Content-Length: 220
         Connection: close
         Date: Thu, 07 Aug 2014 20:47:07 GMT
         Content-Type: application/x-www-form-urlencoded

-----> Utilizando SKD JAVA:

         String shortName = "ventas";
         String name = "Ventas";
         String description = "Grupo de Ventas";
         Groups instance = new Groups(
         "api key",
         "api secret",
         "http://apps01-tigo-csms.im.local:8101/");
         ApiResponse<GroupJsonObject> result = instance.add(shortName, name,
         description);
         if (result.isOk()) {
         GroupJsonObject group = result.getResponse();
         } else {
         System.out.print("http code: " + result.getHttpCode());
         System.out.print("api code: " + result.getErrorCode());
         System.out.print("description: " + result.getErrorDescription());
         }
         

-----> RESPUESTA

Parámetros : short_name  | Tipo : Texto | Descripción: El nombre corto asignado al grupo

-----> Utilizando HTTP:

         HTTP/1.1 200 OK
         Date: Thu, 07 Aug 2014 20:47:07 GMT
         Connection: Keep-Alive
         Transfer-Encoding: chunked
         Content-Type: application/json

-----> Utilizando SDK Java:

         Authorization:IM1d4e705080edec039fe580dd26fd0027:WM/
         16HwXg46H9WevfnWjre2F2
         { short_name: 'ventas', name: 'Ventas', description: 'Grupo de Ventas', members: {
         total: 0, pending: 0, confirmed: 0} }
         

# Actualiza Un Grupo

PUT /groups/:short_name
Esta operación actualiza los datos de un grupo existente. El grupo se identifica con el
nombre corto: short_name

-----> Solicitud

Parámetros :short_name | Tipo: Texto | Descripción: El nombre corto asignado al grupo

-----> Post-Data

Se debe enviar el JSON correspondiente a un objeto de tipo grupo.

-----> Respuesta

Si la operación de actualización es finalizada con éxito, se retornará status 200 OK y el JSON del objeto tipo grupo. En caso contrario se responderá con status de error.

# Ejemplo de solicitud

-----> Utilizando HTTP:

         PUT /api/rest/groups/ventas HTTP/1.1
         Accept-Encoding: identity
         Content-Length: 220
         Connection: close
         Date: Thu, 07 Aug 2014 20:47:07 GMT
         Content-Type: application/x-www-form-urlencoded

-----> Utilizando SDK JAVA:

         String shortName = "ventas";
         String name = "Ventas";
         String description = "Grupo de Ventas";
         String newShortName = "ventas";
         Groups instance = new Groups(
         "api key",
         "api secret",
         "http://apps01-tigo-csms.im.local:8101/");
         ApiResponse<GroupJsonObject> result = instance.update(shortName, name,
         description, newShortName);
         if (result.isOk()) {
         GroupJsonObject group = result.getResponse();
         } else {
         System.out.print("http code: " + result.getHttpCode());
         System.out.print("api code: " + result.getErrorCode());
         System.out.print("description: " + result.getErrorDescription());} 

-----> Respuesta

-----> Utilizando HTTP:

         HTTP/1.1 200 OK
         Parámetros Tipo Descripción
         :short_name Texto El nombre corto asignado al grupo
         Date: Thu, 07 Aug 2014 20:47:07 GMT
         Connection: Keep-Alive
         Transfer-Encoding: chunked
         Content-Type: application/json

-----> Utilizando SDK JAVA:

         Authorization:IM1d4e705080edec039fe580dd26fd0027:WM/
         16HwXg46H9WevfnWjre2F2
         { short_name: 'ventas', name: 'Ventas', description: 'Grupo de Ventas', members: {
         total: 0, pending: 0, confirmed: 0} }
         
# Elimina un Grupo
DELETE /groups/:short_name

Esta operación da de baja un grupo existente. El grupo se identifica con el nombre corto: short_name

-----> Solicitud

Parámetros :short_name | Tipo: Texto | Descripción: El nombre corto asignado al grupo

-----> Respuesta

Si la operación de eliminación es realizada con éxito, se retornará status 200 OK y el JSON del objeto tipo grupo. En caso contrario se responderá un status de error.

# Ejemplo de solicitud

-----> Utilizando HTTP:

         DELETE /api/rest/groups/ventas HTTP/1.1
         Accept-Encoding: identity
         Content-Length: 0
         Connection: close
         Date: Thu, 07 Aug 2014 20:47:07 GMT
         Content-Type: application/x-www-form-urlencoded

-----> Utilizando SKD JAVA:

         String shortName = "ventas";
         Groups instance = new Groups(
         "api key",
         "api secret",
         "http://apps01-tigo-csms.im.local:8101/");
         ApiResponse<GroupJsonObject> result = instance.delete(shortName);
         if (result.isOk()) {
         GroupJsonObject group = result.getResponse();
         } else {
         System.out.print("http code: " + result.getHttpCode());
         System.out.print("api code: " + result.getErrorCode());
         System.out.print("description: " + result.getErrorDescription());
         }

-----> Respuesta

-----> Utilizando HTTP:

         HTTP/1.1 200 OK
         Parámetros Tipo Descripción
         :short_name Texto El nombre corto asignado al grupo
         Date: Thu, 07 Aug 2014 20:47:07 GMT
         Connection: Keep-Alive
         Transfer-Encoding: chunked
         Content-Type: application/json

-----> Utilizando SDK JAVA:

         Authorization:IM1d4e705080edec039fe580dd26fd0027:WM/
         16HwXg46H9WevfnWjre2F2
         { short_name: 'ventas', name: 'Ventas', description: 'Grupo de Ventas', members: {
         total: 0, pending: 0, confirmed: 0} }

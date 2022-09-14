# Pruebas Ingeniería Levo

Elegirás al menos uno de los proyectos de modo que nos permita validar tu experiencia y estilo de trabajo.

Para cualquiera que elijas, los features que implementes deben estar cubiertos por pruebas unitarias y/o de integración, si el proyecto requiere un setup previo, te sugerimos documentarlo en el README de tu proyecto para realizarlo por cuenta propia o utilizar un pipeline de Github Actions demostrando una configuración básica de pruebas que podemos reproducir también de forma local.

Recuerda no _commitear_ secretos como claves o llaves en el código fuente. Una opción es hacer deployment del proyecto en un sitio público, pero esto no obvia el requerimiento de las pruebas automatizadas del proyecto.

### Ejercicios propuestos

#### Frontend (autenticación)

Te sugerimos utilizar cypress para las pruebas de punto a punto.

El ejercicio consiste en un aplicativo sencillo realizado en Angular que conste de:

1. un simple formulario de ingreso (opcionalmente integrando al menos un login de terceros, como Google, Twitter, OAuth, etc.).
2. en caso de que el usuario no esté autenticado o su JWT haya expirado, indicar de manera muy visible al usuario que debe autenticarse de nuevo y al hacer click en "Aceptar" el usuario es llevado de vuelta a la página de ingreso
3. un área privada que valide automáticamente el JWT, haciendo polling cada pocos segundos mientras el usuario visita la página, para validar el JWT
4. un área pública (tipo FAQ), en esta no realizamos ningún polling ni envío de token
5. enlaces entre las áreas públicas y privada (si se visita el área privada y el token ya expiró, mostrar la advertencia indicada antes (punto 2)

Para poder hacer pruebas interactivas, te recomendamos configurar la vigencia del token en muy poco tiempo, apenas de unos pocos minutos.

##### Opciones de servicio que verifica la autenticación

Puedes implementarlo por cuenta propia o con otro proveedor, lo único que requerimos es lo siguiente:

- un endpoint que emita un token para credenciales válidas (estas pueden estar hardcodeadas ya que no estamos requiriendo un registro previo que almacene nuevos usuarios),
  - un endpoint que valide la vigencia del token y en caso de que ya expiró devolver un error 401/403 según se considere apropiado (y justificar la decisión)

Opciones sugeridas:

- Auth0 (SDK ya preparado, opciones de login con terceros, plan gratuito)
- Servicio propio (te sugerimos pipedream) que solo requiere los endpoints antes indicados, puedes usar el código de una implementación muy simple de un servidor de autorización de OAuth en https://livebook.manning.com/book/oauth-2-in-action/chapter-5/

#### Backend (lógica transaccional)

Para backend en este caso utilizaríamos *PHPUnit sobre Laravel*, tomar en cuenta que un proyecto creado como indica en la documentación ya incluye tests ejecutables, y generadores de pruebas funcionales.

##### Funcionalidad esencial

Requerimos una API que permita registrar "transacciones" sobre una cuenta bancaria, mantener el registro de los eventos (depositar, retiros) y lanzar notificaciones en caso de que se detecte:

- repetidos intentos de realizar un retiro que dejaría un saldo deudor (esto ya está implementado en el tutorial referido)
- flujo de efectivo superior a los $10,000 **en 24 horas**

En caso de incurrir en 3 o más intentos de retirar dinero sin fondos suficientes, emitir un correo de oferta de préstamo al cliente del banco.

Para un flujo de efectivo[^1] superior a los $10,000 **en 24 horas**[^2]:
1. bloquear la transacción si se trata de un retiro[^3]
2. registrando *tanto el intento de retiro como la causa de rechazo*
3. enviar una alerta a un "administrador" del sitio con un registro de las transacciones de las últimas 48 horas.

Advertencias:

Un flujo que rebasa el umbral fuera del plazo dado, digamos 10 mil pesos en 48 horas pero que apenas promediaría los 5 mil por día, no dispararía ninguna alerta según esta regla tan simple. Pero esto es lo esperado: la detección del flujo en un período dado.

El plazo de 24 horas debe manejarse de forma natural, ejemplo: Una operación de las 2:00 a.m. se valida contra todas las ocurridas en las 24 horas previas hasta las 2:00 a.m. del día anterior. No solo con las de su día calendario.

###### Acerca del requerimiento de detectar transacciones en un período determinado.

Sabemos que el tema del control del tiempo en pruebas _dependientes del entorno_ (acá está la clave: las dependencias)
es un poco truculento, pero precisamente nos requiere un dominio del ciclo de vida de pruebas y en qué puntos podemos
extender el runner y nuestros objetos para jugar tanto con tiempos como otras "entradas externas" como APIs de terceros o un puerto de red.

Consejo: Laravel provee un mecanismo extremadamente simple de simulación de tiempos, y el de su librería base de manejo del tiempo `Carbon` tiene una funcionalidad muy sencilla de user para congelar el tiempo en una fecha dada para pruebas.

##### Sugerencias de implementación

Puesto que en este caso el ejercicio no es de seguridad sino solo de funcionalidad, puedes elegir cualquier mecanismo que consideres conveniente para distinguir a un cliente, no requires ni un sistema de login/registro completo, sino puede ser cualquier campo del body o un header en que distingas las transacciones de unos clientes de otros.

Para la implementación puedes elegir apoyarte, solo en los eventos y notificaciones nativas de Laravel o integrar un paquete de manejo de modelos con características más afines a los features de registro de transacciones y reacción ante eventos e historial de eventos previos sobre una misma cuenta.

En este sentido te aconsejamos revisar el paquete de "Event Sourcing" para Laravel:

- Introducción al Event Sourcing con Laravel: https://www.youtube.com/watch?v=9tbxl_I1EGE
- Documentación del paquete de Spatie: https://spatie.be/docs/laravel-event-sourcing/v7/introduction
- Documentación de Testing y dobles de prueba https://laravel.com/docs/9.x/mocking#notification-fake
- Repositorio con el ejemplo de Event Sourcing que explica en el video: https://github.com/spatie/larabank-aggregates

##### Sobre la verificación automática de la funcionalidad

Además de las necesarias pruebas interactivas de la funcionalidad que presentas, esperamos alguna forma simple de verificar la lógica desarrollada, con pruebas repetibles.

En nuestra práctica cotidiana, además del happy path requerimos cubrir casos de fallo o alternativas. Configurar el escenario base para cada una de las pruebas, de forma manual consume mucho más tiempo que el que acumulativamente se ahorra escribiendo pruebas, que en el caso de métodos con entradas y salidas previsibles, no toma demasiado tiempo escribir.

Sobre todo la principal ganancia es que las pruebas se pueden reproducir sin preparar un entorno de pruebas dedicado por cada ejercicio de tests.

Lamentablemente las pruebas interactivas exigen una preparación del entorno y datos de prueba, así como la recolección de los datos y reporte, no nos facilita iterar rápidamente en detectar defectos e incluso regresiones. Por eso como mínimo para el desarrollo del core del business es fundamental para nosotros el dominio de PHPUnit/Jest y otras herramientas de pruebas de los respectivos ecosistemas.

[^1]: Por flujo de efectivo, se entiende tanto depósitos como retiros, cualquier entrada o salida suma hacia la cantidad que se está monitoreando.
[^2]: Esta es la funcionalidad principal, detectar el umbral en un periodo dado. No la notificación. El registro tanto del intento como del rechazo son parte importante pero secundaria. No así manejar correctamente el plazo de 24 horas, esto es el core de todo el ejercicio.
[^3]: Si es retiro, se bloquea la transacción. No bloqueamos depósitos pero si monitoreamos si excede el umbral, y disparamos alerta.

# Automatización E2E de una app híbrida (iOS y Android) — caso de estudio

> **Nota:** el código pertenece al cliente y no es público. Este repositorio documenta el enfoque, las decisiones y los resultados. Los nombres del cliente, de la app y cualquier dato interno se han omitido.

## Contexto

- **Producto:** app de entrenamiento y comunidad deportiva de una marca de retail, en iOS y Android. Gran parte de sus pantallas son **webviews** (páginas web incrustadas en la app nativa).
- **Mi papel:** único QA del proyecto, responsable de la calidad de principio a fin.
- **Problema:** cada release exigía repetir a mano la misma regresión en dos plataformas. Además, había bugs que solo aparecían en la frontera entre lo nativo y lo web, como un botón escondido detrás de la barra inferior de la app.

## Enfoque: tres capas de pruebas

| Capa | Herramienta | Para qué |
|---|---|---|
| **Smoke de webviews** | Playwright (Chromium simulando un Pixel 7 y WebKit simulando un iPhone) | Comprobar en segundos que las páginas cargan, sin app ni dispositivo |
| **App Android** | Appium + WebdriverIO (UiAutomator2), en emulador o móvil real | Usar la app como una usuaria real: toques, formularios, navegación |
| **App iOS** | Appium + WebdriverIO (XCUITest), en iPhone real | Las mismas pruebas que en Android, con las diferencias de iOS resueltas |

La capa de Playwright es rápida pero no ve nada nativo. Las pruebas de Appium son las que validan una versión.

## Qué cubren

- **Login** con la cuenta de pruebas, incluido el paso por el navegador del sistema.
- **Mi cuenta:** carga de datos y edición de datos personales, con validación de campos obligatorios y guardado real.
- **Newsletter:** validación del consentimiento, alta y baja.
- **Comunidad:** recorrido de todas las pestañas y de sus páginas internas hasta 3 niveles, botón atrás, enlaces externos (mapas) y vuelta a la app.
- **Integraciones OAuth con 4 proveedores de actividad deportiva:** vincular, cancelar la desvinculación y desvincular, en las dos plataformas.
- **Detección de elementos tapados:** en cada pantalla se lee la jerarquía nativa y se comprueba que ningún botón o enlace de la web queda debajo de un componente nativo (barra inferior, botones flotantes, avisos) ni de otro elemento de la página.
  - Un solape de 4 a 10 px se acepta con aviso y captura de evidencia.
  - Más de 10 px es fallo.

## Decisiones de diseño

- **Pruebas idempotentes:** cada prueba detecta el estado de la app (onboarding, sin sesión, sesión de otra cuenta) y lo resuelve antes de empezar. Reutiliza la sesión cuando es segura, así que en una ronda completa solo se inicia sesión una vez.
- **La cuenta siempre queda limpia:** si una prueba cambia algo (alta en la newsletter, cuenta vinculada), siempre lo deshace al terminar, aunque una ejecución anterior se cortara a medias.
- **Seguras en producción:** las webviews cargan el entorno real, así que las pruebas **nunca** confirman reservas ni envían códigos de verificación. De esos botones solo comprueban que se pueden pulsar.
- **Toques nativos frente a clics web:** en iOS, los botones que llaman al servidor se pulsan con toques reales, como un dedo, porque así es como los usa una persona.
- **Modo asistido con una persona en el bucle:** cuando un proveedor pide un captcha o un código por email, la prueba espera a que una persona lo complete. No se intenta saltar ninguna protección.
- **Fallar ante lo inesperado:** si aparece una pestaña o un proveedor nuevo que la prueba no conoce, falla a propósito. Así nadie da por probado algo que no se ha automatizado.
- **Pensadas para el equipo:** mensajes de error en español, capturas y logs de cada fallo, y una guía paso a paso para que alguien sin experiencia con la Terminal pueda lanzarlas.

## Resultados

- **Regresión de las webviews en iOS y Android en minutos**, sin tocar el móvil. Por ejemplo, el recorrido completo de la Comunidad tarda 4–5 minutos por plataforma.
- **Bugs detectados por la automatización y reportados al equipo,** entre otros:
  - un botón de la lista de espera escondido detrás de la barra inferior (el bug que dio origen a la prueba);
  - un fallo silencioso al vincular un proveedor, sin ningún mensaje de error para la usuaria;
  - un botón en iOS que no responde al primer toque;
  - una hoja emergente que tapa el acceso al perfil al abrir la app;
  - un botón de cerrar que queda a 2 px del borde de la zona visible.
- También se detectaron **hallazgos de seguridad**, que se reportaron al equipo por el canal adecuado y no se detallan aquí.

## Stack

TypeScript · Node.js · Appium (UiAutomator2 y XCUITest) · WebdriverIO · Playwright · Android Emulator (API 34) · iPhone real · Xcode · Git/GitHub

## Cómo se construyó

Desarrollado con **asistencia de IA generativa**. Yo defino qué hay que probar, los criterios de aceptación y los casos límite, reviso cada resultado y decido qué es un bug y qué no. La IA acelera la escritura y el mantenimiento del código de las pruebas.

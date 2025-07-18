Taller Práctico: Análisis y Corrección de Vulnerabilidades Web
¡Bienvenido a este taller práctico! El objetivo es analizar una aplicación bancaria deliberadamente vulnerable, explotar sus fallos de seguridad y, lo más importante, aprender a corregirlos paso a paso.

🚀 Configuración del Entorno
Clona el Repositorio:



git clone https://github.com/JhonnyCCM/Taller-seguridad.git
Abre el Proyecto: Abre la carpeta en Visual Studio Code.

Inicia el Servidor: Haz clic derecho en el archivo demo_banco_vulnerable.html y selecciona "Open with Live Server".

😈 Fase 1: Explotando las Vulnerabilidades
Vamos a actuar como un atacante para encontrar y explotar los fallos de la aplicación.

Ataque #1: CSRF (Cross-Site Request Forgery)
En la aplicación del banco, inicia sesión.

Abre una nueva pestaña y navega al archivo pagina_maliciosa.html.

Haz clic en el botón rojo "¡RECLAMAR MI PREMIO AHORA!".

Observa: Serás redirigido al banco, donde se habrá ejecutado una transferencia de $1000 sin tu permiso.

Ataque #2: XSS y Robo de Token de localStorage
Con la sesión iniciada, abre las Herramientas de Desarrollador (F12) y en la pestaña "Aplicación", verás el user_session guardado en localStorage.

En la aplicación, ve a la sección "Editar Mensaje de Bienvenida" e introduce el siguiente script de ataque (payload):


<img src="x" onerror="alert('¡XSS! Tu token robado es: ' + localStorage.getItem('user_session'))">
Haz clic en "Actualizar Mensaje".

Observa: Aparecerá una alerta mostrando el token secreto que estaba guardado en localStorage.

🛡️ Fase 2: Corrección del Código
Ahora, vamos a solucionar cada una de las vulnerabilidades aplicando los cambios correctos.

Corrección #1: Mitigar el Ataque CSRF
Añadiremos un Token Anti-CSRF para validar que cada transferencia sea legítima.

Paso 1.1: Generar y Añadir el Token al Formulario
Primero, generamos el token al iniciar sesión y lo insertamos de forma oculta en el formulario de transferencia.

En la función login(), añade la línea que crea el token:

function login() {
    localStorage.setItem('isLoggedIn', 'true');
    localStorage.setItem('balance', '5000');
    // AÑADIR ESTA LÍNEA para crear el token
    localStorage.setItem('csrf_token', "token_secreto_" + Math.random());
    displayDashboard();
}
En la función displayDashboard(), añade el código para insertar el token en el formulario:


function displayDashboard() {
    // ...código existente para mostrar el panel...

    // AÑADIR ESTAS LÍNEAS para poner el token en el formulario
    const transferForm = document.getElementById('transfer-form');
    // Evita añadir tokens duplicados si la función se llama de nuevo
    if (!transferForm.querySelector('input[name="csrf_token"]')) {
        const csrfInput = document.createElement('input');
        csrfInput.type = 'hidden';
        csrfInput.name = 'csrf_token';
        csrfInput.value = localStorage.getItem('csrf_token');
        transferForm.appendChild(csrfInput);
    }
}
Paso 1.2: Validar el Token antes de Transferir
Ahora, modificamos la lógica del formulario para que verifique el token antes de procesar cualquier transferencia.

Reemplaza todo el manejador onsubmit por este código corregido:



document.getElementById('transfer-form').onsubmit = function(e) {
    e.preventDefault();
    
    // 1. Obtenemos el token guardado en nuestro almacenamiento
    const storedToken = localStorage.getItem('csrf_token');
    
    // 2. Obtenemos el token que se envió con el formulario desde el campo oculto
    const receivedToken = e.target.csrf_token ? e.target.csrf_token.value : null;

    // 3. Comparamos. Si no coinciden o no hay token, bloqueamos la acción.
    if (receivedToken !== storedToken || !receivedToken) {
        alert("ALERTA DE SEGURIDAD: Petición no válida (Token CSRF incorrecto).");
        return;
    }

    // Si la validación pasa, la transferencia se procesa de forma segura
    processTransfer(parseFloat(document.getElementById('amount').value), document.getElementById('recipient').value);
};
¿Por qué funciona? La página maliciosa no conoce el csrf_token secreto, por lo que su petición llega sin él. Nuestra nueva lógica detecta la ausencia del token correcto y bloquea el ataque.

Corrección #2: Neutralizar el Ataque XSS
Este cambio es simple pero crucial. Le diremos al navegador que trate la entrada del usuario como texto plano y no como código.

En la función updateWelcomeMessage(), reemplaza .innerHTML por .innerText.

JavaScript

// CÓDIGO VULNERABLE
// document.getElementById('welcome-message').innerHTML = document.getElementById('welcome-input').value;

// CÓDIGO CORREGIDO
document.getElementById('welcome-message').innerText = document.getElementById('welcome-input').value;
¿Por qué funciona? .innerText escribe el contenido literalmente en la pantalla, ignorando cualquier etiqueta <script>. El código del atacante se muestra, pero no se ejecuta.

Corrección #3: Dejar de Exponer el Token de Sesión
La solución definitiva es nunca guardar el token de sesión en localStorage.

En la función login(), elimina o comenta la línea que guarda el user_session.


function login() {
    localStorage.setItem('isLoggedIn', 'true');
    localStorage.setItem('balance', '5000');

    // ELIMINAR O COMENTAR ESTA LÍNEA
    // const fakeSessionToken = "session_token_super_secreto_" + Date.now();
    // localStorage.setItem('user_session', fakeSessionToken);

    localStorage.setItem('csrf_token', "token_secreto_" + Math.random());
    displayDashboard();
}
¿Por qué funciona? Este cambio representa la decisión de usar Cookies HttpOnly desde el backend. Al hacer el token invisible para JavaScript, ningún ataque XSS futuro podrá robarlo.

¡Felicidades! Has analizado, explotado y corregido exitosamente las vulnerabilidades de la aplicación.
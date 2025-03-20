# robandoPDFS
Google Drive how to install PDF restricted

## Script
```javascript
function navegarPagina(selectorInput, numeroPagina) {
    const input = document.querySelector(selectorInput);
    if (!input) {
        console.error("No se encontró el input con el selector:", selectorInput);
        return;
    }

    input.focus();
    input.value = numeroPagina;
    input.dispatchEvent(new Event('input', { bubbles: true, cancelable: true }));
    input.dispatchEvent(new KeyboardEvent('keydown', {
        key: 'Enter',
        code: 'Enter',
        which: 13,
        keyCode: 13,
        bubbles: true,
        cancelable: true
    }));
}

function navegarAutomaticamenteEnDocumento(selectorPaginacion, selectorInput, tiempoEspera, callbackDocumentoCompleto) { // callbackDocumentoCompleto se mantiene para la estructura, pero no se usa directamente para el límite aquí
    const elementoPaginacion = document.querySelector(selectorPaginacion);
    if (!elementoPaginacion) {
        console.error("No se encontró el elemento de paginación.");
        if (callbackDocumentoCompleto) {
            callbackDocumentoCompleto();
        }
        return;
    }

    const totalPaginasDocumento = parseInt(elementoPaginacion.querySelector('.a-b-La-yc-Hh').textContent, 10);
    let paginaActualDocumento = 1;

    function siguientePagina() {
        if (paginaActualDocumento <= totalPaginasDocumento) {
            console.log(`Navegando a la página ${paginaActualDocumento} del documento.`);
            navegarPagina(selectorInput, paginaActualDocumento);
            paginaActualDocumento++;
            setTimeout(siguientePagina, tiempoEspera);
        } else {
            console.log("Se llegó al final de las páginas del documento.");
            if (callbackDocumentoCompleto) {
                callbackDocumentoCompleto();
            }
        }
    }

    siguientePagina();
}


function navegarPaginasAcumuladasConLimite(selectorPaginacion, selectorInputPagina, botonSiguienteDocumentoSelector, maxPaginasTotal = 600, tiempoEsperaPagina = 700, tiempoEsperaDocumento = 300) {
    let paginasTotalesNavegadas = 0; // Contador simple para páginas totales
    let documentoActual = 0; // Contador de documento actual (para logs)

    function navegarSiguienteDocumento() { // Esta función ahora maneja "Siguiente Página Acumulada"
        documentoActual++; // Incrementa el contador de documento para logs, aunque ahora estamos acumulando páginas

        function documentoCompletoCallback() { // Callback simplificado, solo para pasar al "Siguiente" o finalizar
            if (paginasTotalesNavegadas < maxPaginasTotal) { // **Verificación SIMPLE del límite de páginas acumuladas**
                const botonSiguienteAcumulado = document.querySelector(botonSiguienteDocumentoSelector); // Usamos el mismo selector, asumiendo que es "Siguiente Página Acumulada"

                if (botonSiguienteAcumulado) {
                    console.log(`Pasando a la siguiente página acumulada (Documento ${documentoActual}). Páginas totales navegadas: ${paginasTotalesNavegadas}`);
                    // Simular clic en el botón "Siguiente Página Acumulada"
                    botonSiguienteAcumulado.dispatchEvent(new MouseEvent('mousedown', { bubbles: true, cancelable: true, view: window }));
                    botonSiguienteAcumulado.dispatchEvent(new MouseEvent('mouseup', { bubbles: true, cancelable: true, view: window }));
                    botonSiguienteAcumulado.dispatchEvent(new MouseEvent('click', { bubbles: true, cancelable: true, view: window }));
                    document.body.dispatchEvent(new FocusEvent('focus', { bubbles: true, cancelable: true, view: window }));

                    setTimeout(navegarSiguienteDocumento, tiempoEsperaDocumento); // Llamar recursivamente para la siguiente página acumulada
                } else {
                    console.log("No se encontró el botón 'Siguiente Página Acumulada'. Finalizando navegación.");
                    console.log(`Se completó la navegación. Páginas totales navegadas: ${paginasTotalesNavegadas}`);
                    ejecutarScriptJsPDF(); // Ejecutar jsPDF si no hay más páginas "Siguiente"
                }
            } else {
                console.log(`Límite máximo de páginas acumuladas (${maxPaginasTotal}) alcanzado. Finalizando navegación.`);
                console.log(`Se completó la navegación. Páginas totales navegadas: ${paginasTotalesNavegadas}`);
                ejecutarScriptJsPDF(); // Ejecutar jsPDF si se alcanza el límite
            }
        }


        if (paginasTotalesNavegadas < maxPaginasTotal) { // Verificar ANTES de navegar en el documento
            console.log(`Iniciando navegación del documento ${documentoActual}. Páginas totales navegadas hasta ahora: ${paginasTotalesNavegadas}`);
            navegarAutomaticamenteEnDocumento(selectorPaginacion, selectorInputPagina, tiempoEsperaPagina, documentoCompletoCallback); // Navegar por las páginas del documento actual
            // `documentoCompletoCallback` se llamará cuando termine `navegarAutomaticamenteEnDocumento`
            // y dentro de `documentoCompletoCallback` se decide si se pasa a la siguiente página acumulada o se finaliza.

        } else {
             console.log(`Límite máximo de páginas acumuladas (${maxPaginasTotal}) ya alcanzado. Finalizando navegación.`);
             console.log(`Se completó la navegación. Páginas totales navegadas: ${paginasTotalesNavegadas}`);
             ejecutarScriptJsPDF(); // Ejecutar jsPDF si el límite ya se alcanzó antes de iniciar este documento
        }
        paginasTotalesNavegadas += parseInt(document.querySelector('.a-b-La-yc-Hh').textContent, 10); // **IMPORTANTE**: Sumar las páginas del documento *después* de navegarlo para contar correctamente
    }


    function ejecutarScriptJsPDF() {
        let jspdf = document.createElement( "script" );
        jspdf.onload = function () {
            let pdf = new jsPDF();
            let elements = document.getElementsByTagName( "img" );
            for ( let i in elements) {
                let img = elements[i];
                if (!/^blob:/.test(img.src)) {
                    continue ;
                }
                let canvasElement = document.createElement( 'canvas' );
                let con = canvasElement.getContext( "2d" );
                canvasElement.width = img.width;
                canvasElement.height = img.height;
                con.drawImage(img, 0, 0,img.width, img.height);
                let imgData = canvasElement.toDataURL( "image/jpeg" , 1.0);
                pdf.addImage(imgData, 'JPEG' , 0, 0);
                pdf.addPage();
            }
            pdf.save( "download.pdf" );
        };
        jspdf.src = 'https://cdnjs.cloudflare.com/ajax/libs/jspdf/1.3.2/jspdf.min.js' ;
        document.body.appendChild(jspdf);
    }

    // Iniciar la navegación
    navegarSiguienteDocumento();
}


// Uso:
const selectorPaginacion = '.a-b-vo';  // Selector del contenedor de paginación
const selectorInputPagina = '.a-b-La-su-vb'; // Selector del input de número de página
const botonSiguienteDocumentoSelector = 'div[aria-label="Siguiente"]'; // Selector del botón "Siguiente" (ahora para páginas acumuladas)
const tiempoEsperaEntrePaginas = 500; // Tiempo de espera entre páginas (para navegarAutomaticamenteEnDocumento)
const tiempoEsperaEntreDocumentos = 300; // Tiempo de espera después de hacer clic en "Siguiente Página Acumulada"
const maxPaginasTotalNavegar = 600; // Límite máximo de páginas totales a navegar

navegarPaginasAcumuladasConLimite(selectorPaginacion, selectorInputPagina, botonSiguienteDocumentoSelector, maxPaginasTotalNavegar, tiempoEsperaEntrePaginas, tiempoEsperaEntreDocumentos);
```

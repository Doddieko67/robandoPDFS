# robandoPDFS
Google Drive how to install PDF restricted

## Script
```javascript
// Arreglo para almacenar la información de cada documento: título y número de páginas
let documentosInfo = [];

// Función para navegar a una página específica ingresando un número en un input
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

// Función para navegar automáticamente por todas las páginas de un documento
function navegarAutomaticamenteEnDocumento(selectorPaginacion, selectorInput, tiempoEspera, callbackDocumentoCompleto) {
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

// Función para navegar por todos los documentos y sus páginas, acumulando información
function navegarPaginasAcumuladasConLimite(selectorPaginacion, selectorInputPagina, botonSiguienteDocumentoSelector, maxPaginasTotal = 600, tiempoEsperaPagina = 1000, tiempoEsperaDocumento = 500) {
    let paginasTotalesNavegadas = 0; // Contador de páginas totales navegadas
    let documentoActual = 0; // Contador de documentos para logs

    function navegarSiguienteDocumento() {
        documentoActual++;

        function documentoCompletoCallback() {
            if (paginasTotalesNavegadas < maxPaginasTotal) {
                const botonSiguienteAcumulado = document.querySelector(botonSiguienteDocumentoSelector);
                if (botonSiguienteAcumulado) {
                    console.log(`Pasando al siguiente documento (${documentoActual}). Páginas totales navegadas: ${paginasTotalesNavegadas}`);
                    // Simular clic en el botón "Siguiente"
                    botonSiguienteAcumulado.dispatchEvent(new MouseEvent('mousedown', { bubbles: true, cancelable: true, view: window }));
                    botonSiguienteAcumulado.dispatchEvent(new MouseEvent('mouseup', { bubbles: true, cancelable: true, view: window }));
                    botonSiguienteAcumulado.dispatchEvent(new MouseEvent('click', { bubbles: true, cancelable: true, view: window }));
                    document.body.dispatchEvent(new FocusEvent('focus', { bubbles: true, cancelable: true, view: window }));

                    setTimeout(navegarSiguienteDocumento, tiempoEsperaDocumento);
                } else {
                    console.log("No se encontró el botón 'Siguiente'. Finalizando navegación.");
                    console.log(`Navegación completada. Páginas totales navegadas: ${paginasTotalesNavegadas}`);
                    setTimeout(ejecutarScriptJsPDF, 5000); // Espera adicional de 5 segundos
                }
            } else {
                console.log(`Límite máximo de páginas (${maxPaginasTotal}) alcanzado. Finalizando navegación.`);
                console.log(`Navegación completada. Páginas totales navegadas: ${paginasTotalesNavegadas}`);
                setTimeout(ejecutarScriptJsPDF, 5000); // Espera adicional de 5 segundos
            }
        }

        if (paginasTotalesNavegadas < maxPaginasTotal) {
            console.log(`Iniciando navegación del documento ${documentoActual}. Páginas totales navegadas hasta ahora: ${paginasTotalesNavegadas}`);
            // Obtener título y número de páginas del documento actual
            const tituloDocumento = document.querySelector('.a-b-K-T.a-b-cg-Zf').textContent;
            const totalPaginasDocumento = parseInt(document.querySelector('.a-b-La-yc-Hh').textContent, 10);
            documentosInfo.push({ titulo: tituloDocumento, paginas: totalPaginasDocumento });
            navegarAutomaticamenteEnDocumento(selectorPaginacion, selectorInputPagina, tiempoEsperaPagina, documentoCompletoCallback);
            paginasTotalesNavegadas += totalPaginasDocumento;
        } else {
            console.log(`Límite máximo de páginas (${maxPaginasTotal}) alcanzado. Finalizando navegación.`);
            console.log(`Navegación completada. Páginas totales navegadas: ${paginasTotalesNavegadas}`);
            setTimeout(ejecutarScriptJsPDF, 5000); // Espera adicional de 5 segundos
        }
    }

    navegarSiguienteDocumento();
}

// Función para generar PDFs separados por documento
function ejecutarScriptJsPDF() {
    let jspdf = document.createElement("script");
    jspdf.onload = function () {
        let elements = document.getElementsByTagName("img");
        let imgIndex = 0;
        console.log(documentosInfo);
        documentosInfo.forEach((docInfo, index) => {
            let pdf = new jsPDF();
            for (let i = 0; i < docInfo.paginas; i++) {
                if (imgIndex >= elements.length) {
                    console.error("No hay suficientes imágenes para el documento:", docInfo.titulo);
                    break;
                }
                let img = elements[imgIndex];
                while (!/^blob:/.test(img.src)) {
                    imgIndex++;                    
                    img = elements[imgIndex];
                    console.log(img.src)     
                }
                let canvasElement = document.createElement('canvas');
                let con = canvasElement.getContext("2d");
                canvasElement.width = img.width;
                canvasElement.height = img.height;
                con.drawImage(img, 0, 0, img.width, img.height);
                let imgData = canvasElement.toDataURL("image/jpeg", 1.0);
                pdf.addImage(imgData, 'JPEG', 0, 0);
                if (i < docInfo.paginas - 1) {
                    pdf.addPage();
                }
                imgIndex++;
            }
            pdf.save(`${docInfo.titulo}.pdf`);
            console.log(`PDF generado: ${docInfo.titulo}.pdf`);
        });
    };
    jspdf.src = 'https://cdnjs.cloudflare.com/ajax/libs/jspdf/1.3.2/jspdf.min.js';
    document.body.appendChild(jspdf);
}

// Configuración y ejecución
const selectorPaginacion = '.a-b-vo'; // Selector del contenedor de paginación
const selectorInputPagina = '.a-b-La-su-vb'; // Selector del input de número de página
const botonSiguienteDocumentoSelector = 'div[aria-label="Siguiente"]'; // Selector del botón "Siguiente"
const tiempoEsperaEntrePaginas = 1000; // 1000 ms entre páginas
const tiempoEsperaEntreDocumentos = 500; // 500 ms entre documentos
const maxPaginasTotalNavegar = 10; // Límite máximo de páginas totales

navegarPaginasAcumuladasConLimite(
    selectorPaginacion,
    selectorInputPagina,
    botonSiguienteDocumentoSelector,
    maxPaginasTotalNavegar,
    tiempoEsperaEntrePaginas,
    tiempoEsperaEntreDocumentos
);
```

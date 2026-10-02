# OptiScaler AMD Pre-SR — 1.9.9.1

Paquete de integración para Windows mantenido por m7668. Combina OptiScaler con el trabajo comunitario AMD Pre-SR para ofrecer instalador, componentes de escalado y opciones DLSS-NR Pre-SR y XeFG/XeMFG para juegos compatibles. Es un paquete comunitario, no una publicación ni respaldo oficial de los proyectos originales.

## Proyectos base

Este proyecto combina y se basa en:

- [OptiScaler](https://github.com/optiscaler/OptiScaler): framework de proxy gráfico, escalado y generación de fotogramas.
- [DLSS-NR-on-AMD](https://github.com/danielblnc/DLSS-NR-on-AMD): backend AMD DLSS-NR de Daniel; el instalador y la configuración ofrecen una vía de integración para versiones compatibles, incluida la 0.5.0.
- [dlss-5-amd-project](https://github.com/TheAutomatic/dlss-5-amd-project): base para la integración AMD Pre-SR y la planificación relacionada.

La ruta XeFG/XeMFG puede usar el plugin ASI XeFGUnlock, un componente independiente de la comunidad OptiScaler.

## Contenido de 1.9.9.1

Proxy OptiScaler, configuración, instalador/desinstalador; dependencias AMD FidelityFX, Intel XeSS y DirectX Agility con sus avisos; runtime NR lmxxf, módulos gfx1200/gfx1201, shaders, manifiestos y documentación en chino, inglés y español.

Esta versión conserva los valores suministrados: DLSS-NR está desactivado ([DlssNr] Enabled=false) y el backend seleccionado es lmxxf. Para usar Daniel 0.5.0, obtén e instala su runtime externo desde una fuente oficial; este paquete no activa ese backend por sí solo.

## Dependencias externas y redistribución

El ZIP público no incluye archivos proxy/runtime, instalador ni pesos de modelo de Daniel. Su licencia prohíbe redistribuir el software total o parcialmente, incluso volver a subirlo o incluirlo en otra herramienta. Obtén los archivos en las [publicaciones oficiales](https://github.com/danielblnc/DLSS-NR-on-AMD/releases) y respeta la [licencia](https://github.com/danielblnc/DLSS-NR-on-AMD/blob/master/LICENSE). La documentación del instalador requiere que el usuario proporcione version.dll de Daniel; los pesos se pueden generar o proporcionar según las instrucciones originales.

El ZIP público también omite nvngx_dlssnr.dll de NVIDIA y los archivos ASI/INI de XeFGUnlock. Este proyecto no incluye una licencia independiente de redistribución para XeFGUnlock; obtén el plugin del autor o de una fuente autorizada de la comunidad OptiScaler.

## Instalación y configuración

1. Descarga y extrae OptiScaler-AMD-PreSR-1.9.9.1.zip de esta publicación.
2. Ejecuta Setup.bat. Sin argumentos, abre un selector de carpeta de juego y un menú de selección de DLL proxy.
3. Para activar Daniel NR o XeMFG, obtén los archivos externos de sus titulares o canales autorizados y sigue las instrucciones originales y del instalador. No vuelvas a subirlos aquí.

Ejemplo XeFGUnlock (empieza probando 2×):

    [Plugins]
    LoadAsiPlugins=true

    [FrameGen]
    External=false
    Enabled=true
    FGInput=dlssg
    FGOutput=xefg

    [XeFG]
    InterpolationCount=1

InterpolationCount=1 corresponde a 2×. La compatibilidad depende del juego, el controlador, el hardware y la versión del plugin. Consulta [TheAutomatic/dlss-5-amd-project](https://github.com/TheAutomatic/dlss-5-amd-project) para notas de integración.

## Licencias y aviso

SHA256SUMS.txt enumera los hashes SHA-256 de los archivos del ZIP público. Los avisos de licencia y atribución están en Licenses/. Cada componente conserva sus propias condiciones; este paquete no concede una licencia única para todos los componentes ni está respaldado por AMD, Intel, Microsoft, NVIDIA o los mantenedores originales.

Se proporciona tal cual. Haz una copia de seguridad de los archivos del juego y de la configuración antes de instalar. Al informar de un problema, incluye el juego, la GPU, el controlador y los registros pertinentes después de quitar la información personal.
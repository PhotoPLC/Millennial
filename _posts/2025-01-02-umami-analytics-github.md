---
layout: post
title: "Umami Analytics"
date: 2026-01-13
author: "Pedro López"
categories: documentation
tags: [documentation,sample]
image: umami2.jpg
description: "Cómo solucioné el bloqueo de Umami Analytics debido al firewall educativo usando Jekyll."
---



¿Alguna vez has notado que tu web carga instantáneamente en casa, pero al abrirla en la escuela o la oficina parece que ha vuelto a la época del módem?

Eso me pasó recientemente. Tengo mi portafolio alojado en **GitHub Pages** con **Jekyll**. En mi casa vuela. Pero el otro día, intentando enseñárselo a un profesor usando el WiFi del centro, la páginas donde querías acceder tardaban en ejecutarse **4 segundos exactos** antes de mostrar nada.

Como desarrollador junior, 4 segundos es una eternidad. Es la diferencia entre un usuario feliz y uno que cierra la pestaña pensando que la web está rota. Así que decidí investigar.

## 🕵️‍♂️ El culpable: Un Timeout silencioso

Abrí las herramientas de desarrollador (`F12`), fui a la pestaña de **Network** y recargué. Ahí estaba el problema: una petición pendiente que bloqueaba todo el proceso de carga.

Era mi script de analíticas: **Umami**.

El firewall del centro educativo tiene una lista negra de dominios y palabras clave como `analytics`, `tracking` o `metrics`. Pero aquí está el problema técnico real:

1.  Si el firewall rechazara la conexión inmediatamente (un error 403 o RST), la web cargaría rápido, aunque sin analíticas.
2.  Pero este firewall hace **Packet Dropping**. Simplemente ignora la petición.
3.  Mi navegador se queda esperando una respuesta ("handshake") que nunca llega.
4.  Tras **4 segundos**, el navegador se rinde (Timeout) y continúa cargando el resto de la web.

Esos 4 segundos de "pantalla esperando" eran culpa de una mala gestión de la conexión bloqueada.

## 🛠️ La Solución en Jekyll

Al estar en un entorno estático (GitHub Pages), no tengo un servidor backend para configurar proxies complejos, pero la solución fue más sencilla de lo esperado: **Camuflaje y Atributos**.

### 1. El Camuflaje (Archivo Script en el propio repositorio)
El firewall bloqueaba la petición `https://cloud.umami.is/script` por defecto. La solución ideal y facil es guardar el archivo `script` en el repositorio para que la URL del script no parezca sospechosa.

### 2. La implementación en el código (Defer)
Lo más importante fue asegurarme de que, incluso si el firewall decide bloquearme de nuevo en el futuro, mi web no se congele.

Modifiqué mi archivo `_includes/head.html` (o donde tengas tus scripts) para añadir el atributo `defer`. Esto le dice al navegador: *"Sigue pintando la web, accede este script ubicado dentro de assets (o cualquier carpeta) y ejecútalo "*.

Así es como quedó mi código final:

```<script defer 
  src="{{ site.github.url }}/assets/script.js" 
  data-website-id="xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
  data-host-url="https://cloud.umami.is">
</script>
```

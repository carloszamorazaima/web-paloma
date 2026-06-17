---
name: scroll-video
description: >
  Usa esta skill cuando el usuario quiera añadir el efecto de vídeo controlado por scroll al hero de la web.
  Se activa con frases como: "usa la skill scroll-video", "añade el efecto scroll al vídeo",
  "quiero el efecto Apple en el hero", "que el vídeo avance con el scroll".
---

# Scroll Video — Instrucciones para Claude

## Objetivo
Añadir un efecto cinematográfico al hero de index.html donde el vídeo avanza fotograma a fotograma mientras el usuario hace scroll, igual que Apple hace con el iPhone en su web.

---

## Cómo funciona el efecto

1. El usuario llega a la página — el vídeo está parado en el primer fotograma
2. El usuario empieza a hacer scroll hacia abajo
3. El vídeo avanza fotograma a fotograma sincronizado con el scroll
4. Cuando el vídeo termina, el scroll normal de la página continúa
5. En móvil: el vídeo se reproduce automáticamente (autoplay silenciado) porque el scroll-driven no funciona bien en táctil

---

## Paso 1 — Localizar la sección del hero

Busca el comentario `<!-- SCROLL-VIDEO-SECTION -->` en index.html. Ahí es donde está el vídeo hero que hay que modificar.

---

## Paso 2 — Estructura HTML necesaria

Reemplaza o modifica la sección hero para que tenga esta estructura:

```html
<!-- SCROLL-VIDEO-SECTION -->
<section class="scroll-video-hero" id="scroll-hero">
  <div class="scroll-video-sticky">
    <video
      id="heroVideo"
      src="assets/hero.mp4"
      poster="assets/hero-poster.jpg"
      playsinline
      muted
      preload="auto"
    ></video>
    <div class="hero-content">
      <!-- El contenido del hero (título, subtítulo, botones) va aquí -->
    </div>
    <div class="scroll-progress-bar">
      <div class="scroll-progress-fill" id="progressFill"></div>
    </div>
  </div>
</section>
```

---

## Paso 3 — CSS necesario

Añade este CSS dentro del `<style>` de index.html:

```css
/* Scroll Video Hero */
.scroll-video-hero {
  height: 500vh; /* Controla la velocidad: más altura = scroll más lento */
  position: relative;
}

.scroll-video-sticky {
  position: sticky;
  top: 0;
  height: 100vh;
  width: 100%;
  overflow: hidden;
}

.scroll-video-sticky video {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.hero-content {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  z-index: 2;
  background: rgba(0, 0, 0, 0.55);
  color: white;
  text-align: center;
  padding: 0 20px;
}

/* Barra de progreso */
.scroll-progress-bar {
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  height: 4px;
  background: rgba(255, 255, 255, 0.2);
  z-index: 3;
}

.scroll-progress-fill {
  height: 100%;
  width: 0%;
  background: var(--color-primary, #2978fd);
  transition: width 0.05s linear;
}

/* Móvil — autoplay en lugar de scroll-driven */
@media (max-width: 768px) {
  .scroll-video-hero {
    height: 100vh;
  }
  .scroll-video-sticky {
    position: relative;
  }
}

/* Respeta preferencia de movimiento reducido */
@media (prefers-reduced-motion: reduce) {
  .scroll-video-hero {
    height: 100vh;
  }
}
```

---

## Paso 4 — JavaScript necesario

Añade este script antes del cierre de `</body>` en index.html:

```javascript
(function() {
  const video = document.getElementById('heroVideo');
  const hero = document.getElementById('scroll-hero');
  const progressFill = document.getElementById('progressFill');

  if (!video || !hero) return;

  // Detectar móvil
  const isMobile = window.matchMedia('(max-width: 768px)').matches;
  const prefersReduced = window.matchMedia('(prefers-reduced-motion: reduce)').matches;

  if (isMobile || prefersReduced) {
    // En móvil: autoplay normal
    video.autoplay = true;
    video.loop = true;
    video.play().catch(() => {});
    return;
  }

  // Desktop: scroll-driven
  video.pause();
  video.currentTime = 0;

  let isVideoReady = false;

  video.addEventListener('loadedmetadata', () => {
    isVideoReady = true;
  });

  // Forzar carga
  video.load();

  function updateVideo() {
    if (!isVideoReady || !video.duration) return;

    const heroRect = hero.getBoundingClientRect();
    const heroTop = heroRect.top;
    const heroHeight = hero.offsetHeight;
    const windowHeight = window.innerHeight;

    // Progreso del scroll dentro de la sección
    const scrolled = -heroTop;
    const scrollable = heroHeight - windowHeight;

    if (scrolled <= 0) {
      video.currentTime = 0;
      if (progressFill) progressFill.style.width = '0%';
      return;
    }

    if (scrolled >= scrollable) {
      video.currentTime = video.duration;
      if (progressFill) progressFill.style.width = '100%';
      return;
    }

    const progress = scrolled / scrollable;
    video.currentTime = progress * video.duration;
    if (progressFill) progressFill.style.width = (progress * 100) + '%';
  }

  // Usar requestAnimationFrame para suavidad
  let ticking = false;
  window.addEventListener('scroll', () => {
    if (!ticking) {
      requestAnimationFrame(() => {
        updateVideo();
        ticking = false;
      });
      ticking = true;
    }
  }, { passive: true });

  // Actualizar también al redimensionar
  window.addEventListener('resize', updateVideo);

})();
```

---

## Paso 5 — Ajustar la velocidad

La velocidad del efecto se controla con la propiedad `height` de `.scroll-video-hero`:

- `300vh` → efecto rápido (poco scroll para ver el vídeo completo)
- `500vh` → velocidad media (recomendada)
- `700vh` → efecto lento y cinematográfico

Si el usuario dice que va demasiado rápido, aumenta el valor. Si dice que va demasiado lento, redúcelo.

---

## Paso 6 — Al terminar

Confirma al usuario:
- Que el efecto está instalado en index.html
- Que abra la web con Live Server (no abrir el archivo directamente)
- Que haga scroll lento sobre el hero para ver el efecto
- Que si quiere ajustar la velocidad te lo diga

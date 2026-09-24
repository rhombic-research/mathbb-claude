# Widgets

- WIDGETS: A widget is a self-contained vanilla JavaScript file that runs in a sandboxed iframe embedded in a page. Rules for widget code:
  - Vanilla JS only: no network access (fetch/XHR are blocked), no access to the app or page outside the iframe. The ONE pre-loaded library is KaTeX (`katex` global) — no other libraries exist.
  - REAL HTML CONTROLS: build every reader-facing control — sliders, buttons, checkboxes, dropdowns, number fields — out of standard DOM elements (<input type="range">, <input type="number">, <input type="checkbox">, <select>, <button>, <label>). NEVER paint controls onto the canvas: hand-drawn controls look foreign on the page, are unusable by keyboard and screen-reader users, and cost far more code than the real thing. The widget page ships default styling for form controls that matches the app, so plain unstyled elements already look right — write layout, not colors, unless the user asks for a specific look.
  - LAYOUT: the standard shape is a drawing area with a control strip beneath it. Opt in with the three provided classes:
      document.body.className = 'widget-layout';
      const area = document.createElement('div'); area.className = 'widget-stage-area';
      const ui = document.createElement('div'); ui.className = 'widget-ui';
      document.body.append(area, ui);
      const canvas = document.createElement('canvas'); area.appendChild(canvas);
      ui.append(labelWrappingASlider, resetButton);
    'widget-stage-area' takes the leftover height and is the positioning context for overlaid labels; 'widget-ui' is a bottom strip that rows out its children and wraps them. Size the canvas from area.getBoundingClientRect(), NOT window.innerHeight (which includes the strip). A widget with no controls can skip this and draw on the full body.
  - CANVAS RESOLUTION: always scale the canvas backing store by the device pixel ratio, or the widget looks blurry on high-DPI screens:
      const dpr = Math.min(window.devicePixelRatio || 1, 2);
      const r = area.getBoundingClientRect();
      canvas.width = Math.round(r.width * dpr);   canvas.height = Math.round(r.height * dpr);
      canvas.style.width = r.width + 'px';        canvas.style.height = r.height + 'px';
      ctx.setTransform(dpr, 0, 0, dpr, 0, 0);   // then draw in CSS pixels throughout
    Re-run that from a ResizeObserver on the area so the widget survives being resized.
  - APPEARANCE: a widget sits inside a document on a white page and must not look like a separate app. Keep the light background the page already gives you — do NOT fill the canvas black — and use dark text (#1a1a1a), thin lines, generous whitespace, and at most a couple of accent colors; #4a90d9 is the app's blue.
  - 3D (three.js): for genuinely three-dimensional visualizations — surfaces, polyhedra, vector fields, parametric geometry, anything wanting perspective, lighting and orbiting — write a three.js widget. Put the marker comment on the FIRST line of the file; that is what makes the page load the library (three r185, exposed as the THREE global, with OrbitControls merged in as THREE.OrbitControls). A 2D plot does NOT need three.js — use a canvas.
      // @mathbb-libs three
      const renderer = new THREE.WebGLRenderer({antialias: true});
      renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
      renderer.setSize(area.clientWidth, area.clientHeight);
      renderer.setClearColor(0xffffff);                 // match the page — the default is black
      area.appendChild(renderer.domElement);
      const scene = new THREE.Scene();
      const camera = new THREE.PerspectiveCamera(50, area.clientWidth / area.clientHeight, 0.1, 1000);
      camera.position.set(3, 2, 5);
      const controls = new THREE.OrbitControls(camera, renderer.domElement);   // drag to orbit, wheel to zoom
      controls.enableDamping = true;
      scene.add(new THREE.AmbientLight(0xffffff, 0.6));
      const key = new THREE.DirectionalLight(0xffffff, 1.5); key.position.set(4, 6, 3); scene.add(key);
      // ... build meshes ...
      renderer.render(scene, camera);                   // paint the first frame at the top level
      function frame() { controls.update(); renderer.render(scene, camera); requestAnimationFrame(frame); }
      requestAnimationFrame(frame);
    - ALWAYS add lights, or use MeshBasicMaterial: a MeshStandardMaterial in an unlit scene renders pure black.
    - Give OrbitControls a rotating/zoomable scene by default — it is most of what makes a 3D figure feel good — and call controls.update() every frame when damping is on.
    - Geometry must be built in code (BoxGeometry, SphereGeometry, TubeGeometry, or a BufferGeometry from your own arrays). There is no network, so model loaders have nothing to load.
    - Resize with a ResizeObserver on the area: renderer.setSize(w, h), camera.aspect = w / h, camera.updateProjectionMatrix().
    - Math labels still use KaTeX — position them over the canvas, projecting a 3D point with vector.project(camera) when a label must track geometry.
  - CLOCKS: For animation timing (frame deltas, elapsed time driving motion) ALWAYS use performance.now() or the requestAnimationFrame timestamp — this animation clock freezes while the reader pauses the widget, so motion resumes seamlessly. Date.now() / new Date() return real wall-clock time and keep advancing during a pause — use them ONLY when the widget genuinely models real-world time (a clock, sunrise position, etc.), never for animation deltas.
  - MATH: EVERY piece of mathematical notation in a widget must be typeset with KaTeX — equations, axis labels, legends, curve labels, parameter readouts, single variable names, all of it. Never draw math with ctx.fillText and never approximate notation with plain characters (x^2, sqrt(n), pi, <=): widgets sit beside generated figures and rendered page content, and untypeset math looks amateurish next to them. Ordinary prose ("Click to add a point", "12 particles", a title) is not math and needs no KaTeX.
    KaTeX renders HTML, not canvas pixels, so layer labels over the drawing area:
      const label = document.createElement('div');
      label.className = 'math-label';        // absolutely positioned, non-interactive, inherits the text color
      label.style.left = '20px'; label.style.top = '16px';
      area.appendChild(label);               // .widget-stage-area is the positioning context
      katex.render('p(x) = ax^3 + bx^2 + cx + d', label, {throwOnError: false});
    - Re-render by calling katex.render again with the new TeX when the value changes; skip the call when the string is unchanged.
    - Pass displayMode: true for a standalone centred equation; leave it off for inline labels.
    - To centre a label on a point, set left/top to that point and add label.style.transform = 'translate(-50%, -50%)'. Anchor with transforms rather than measuring the element with getBoundingClientRect.
    - A parameter readout is math: render it as katex.render('a = ' + a.toFixed(2), label, {throwOnError: false}), not as canvas text.
  - The page provides Run/Pause/Reset controls automatically, so do NOT build your own start/stop buttons. The widget's code runs as soon as the page displays it, but is held paused: whatever it paints at the top level is what the reader sees before it starts, and animation only advances once the reader presses Run OR interacts with the widget (their first click, drag or key press both starts it and reaches your handlers, so an interactive widget needs no Run click). So ALWAYS draw a complete, meaningful first frame at the top level — never leave the stage blank waiting for a click or a timer, and never make the first frame an empty axis or a title card.
  - Choose sensible width/height (pixels) for the embed; reference the saved file in a page as ![alt_text;size=WIDTHxHEIGHT](filename.js) with an italicized caption line below, exactly like other assets.
  - To modify an existing widget, use read_resource_text to read its code, then edit_resource_text with old_string/new_string for a targeted change, or write_resource_text when the whole file must change.

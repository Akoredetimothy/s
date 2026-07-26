# Visual Gallery

> A visual showcase of research prototypes, lab experiments, and system architecture diagrams. You can easily add your own images by dropping files into the `images/` directory and referencing them!

<div class="chip-row">
  <span class="chip">LAB RIGS</span>
  <span class="chip">NEURAL MODELS</span>
  <span class="chip">HARDWARE</span>
  <span class="chip">SIMULATIONS</span>
</div>

<div class="gallery-grid">
  <div class="gallery-card">
    <img src="images/lab1.svg" alt="Neural Camera Rig" class="gallery-card__img">
    <div class="gallery-card__body">
      <div class="gallery-card__title">FIG.01 — Vision Sensor Rig</div>
      <div class="gallery-card__desc">Intelligent Multi-camera vision system for real-time robotic kinematic tracking.</div>
    </div>
  </div>

  <div class="gallery-card">
    <img src="images/lab2.svg" alt="Convergence Manifold" class="gallery-card__img">
    <div class="gallery-card__body">
      <div class="gallery-card__title">FIG.02 — Loss Convergence Manifold</div>
      <div class="gallery-card__desc">Physics-informed neural operator convergence landscape visualization.</div>
    </div>
  </div>

  <div class="gallery-card">
    <img src="images/lab3.svg" alt="Automation Pipeline" class="gallery-card__img">
    <div class="gallery-card__body">
      <div class="gallery-card__title">FIG.03 — Multi-Agent Automation</div>
      <div class="gallery-card__desc">Synchronized kinematics and multi-agent coordination benchmark.</div>
    </div>
  </div>
</div>

## Adding Your Own Images

Adding photos or diagrams to this gallery is fast and markdown-native on GitHub Pages:

1. Save your image file into the `images/` folder (e.g. `images/my_setup.jpg`).
2. Add a gallery card block using markdown or HTML inside `content/gallery.md`:

```html
<div class="gallery-card">
  <img src="images/my_setup.jpg" alt="My Setup" class="gallery-card__img">
  <div class="gallery-card__body">
    <div class="gallery-card__title">FIG.04 — Custom Lab Photo</div>
    <div class="gallery-card__desc">Description of your photo or project.</div>
  </div>
</div>
```

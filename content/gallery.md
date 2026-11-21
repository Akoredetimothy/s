# Visual Gallery

> A visual showcase of research prototypes, lab experiments, and system architecture diagrams. Click any image to expand it.

<div class="chip-row">
  <span class="chip">LAB RIGS</span>
  <span class="chip">NEURAL MODELS</span>
  <span class="chip">HARDWARE</span>
  <span class="chip">SIMULATIONS</span>
  <span class="chip">POINT CLOUDS</span>
  <span class="chip">EDGE AI</span>
</div>

<div class="gallery-grid">
  <div class="gallery-card" tabindex="0" style="--ratio:1.6">
    <img src="images/lab1.svg" alt="Neural Camera Rig" class="gallery-card__img">
    <div class="gallery-card__body">
      <div class="gallery-card__title">FIG.01 — Vision Sensor Rig</div>
    </div>
  </div>

  <div class="gallery-card" tabindex="0" style="--ratio:1.6">
    <img src="images/lab2.svg" alt="Convergence Manifold" class="gallery-card__img">
    <div class="gallery-card__body">
      <div class="gallery-card__title">FIG.02 — Loss Convergence Manifold</div>
    </div>
  </div>

  <div class="gallery-card" tabindex="0" style="--ratio:1.6">
    <img src="images/lab3.svg" alt="Automation Pipeline" class="gallery-card__img">
    <div class="gallery-card__body">
      <div class="gallery-card__title">FIG.03 — Multi-Agent Automation</div>
    </div>
  </div>

  <div class="gallery-card" tabindex="0" style="--ratio:1.6">
    <img src="images/lab4.svg" alt="Optical Flow Vector Dynamics" class="gallery-card__img">
    <div class="gallery-card__body">
      <div class="gallery-card__title">FIG.04 — Optical Flow Dynamics</div>
    </div>
  </div>

  <div class="gallery-card" tabindex="0" style="--ratio:1.6">
    <img src="images/lab5.svg" alt="Quantum State Operator" class="gallery-card__img">
    <div class="gallery-card__body">
      <div class="gallery-card__title">FIG.05 — Quantum State Mesh</div>
    </div>
  </div>

  <div class="gallery-card" tabindex="0" style="--ratio:1.6">
    <img src="images/lab6.svg" alt="Autonomous Kinematics Rig" class="gallery-card__img">
    <div class="gallery-card__body">
      <div class="gallery-card__title">FIG.06 — Kinematics Rig</div>
    </div>
  </div>

  <div class="gallery-card" tabindex="0" style="--ratio:1.6">
    <img src="images/lab7.svg" alt="Edge AI Inference Cluster" class="gallery-card__img">
    <div class="gallery-card__body">
      <div class="gallery-card__title">FIG.07 — Edge AI Cluster</div>
    </div>
  </div>

  <div class="gallery-card" tabindex="0" style="--ratio:1.6">
    <img src="images/lab8.svg" alt="Spatial 3D Point Cloud Mesh" class="gallery-card__img">
    <div class="gallery-card__body">
      <div class="gallery-card__title">FIG.08 — Spatial 3D Point Cloud</div>
    </div>
  </div>
</div>

## Adding Your Own Images

Adding photos or diagrams to this gallery is fast and markdown-native on GitHub Pages:

1. Save your image file into the `images/` folder (e.g. `images/my_setup.jpg`).
2. Add a gallery card block inside `content/gallery.md`. Set `--ratio` to the image's width divided by its height so the layout matches its proportions:

```html
<div class="gallery-card" tabindex="0" style="--ratio:1.6">
  <img src="images/my_setup.jpg" alt="My Setup" class="gallery-card__img">
  <div class="gallery-card__body">
    <div class="gallery-card__title">FIG.09 — Custom Lab Photo</div>
  </div>
</div>
```

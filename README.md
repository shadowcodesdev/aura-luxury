# AURA LUXURY HAUTE

<p align="center">
  <strong>Photorealistic textile. Procedural craft. Real-time performance.</strong>
</p>

<p align="center">
  <a href="https://aura-luxury-haute-3d.vercel.app"><strong>✦ Open the live configurator →</strong></a>
</p>

<p align="center">
  <a href="https://aura-luxury-haute-3d.vercel.app"><img src="https://img.shields.io/badge/Live%20Preview-aura--luxury--haute--3d.vercel.app-c9a46c?style=for-the-badge&logo=vercel&logoColor=white" alt="Live preview" /></a>
  <img src="https://img.shields.io/badge/Three.js-3D%20Rendering-black?style=for-the-badge&logo=threedotjs&logoColor=white" alt="Three.js" />
  <img src="https://img.shields.io/badge/GLSL%20ES%203.0-Procedural%20Shaders-8b6f47?style=for-the-badge" alt="GLSL ES 3.0" />
</p>

---

## The experience

**AURA LUXURY HAUTE** is an ultra-high-fidelity 3D fashion configurator built around a digital twin of draped luxury textile. Instead of relying on heavyweight 8K texture stacks, the surface is composed in real time with custom GLSL: woven micro-relief, directional microfiber sheen, fold transmission, and tactile friction audio.

> **Reference ALH-PBR** · Anisotropic Sheen Model · Procedural Shader Graph

<div align="center">
  <a href="https://aura-luxury-haute-3d.vercel.app"><strong>Experience the live preview</strong></a><br />
  <sub>Best experienced with sound enabled and a modern WebGL 2 browser.</sub>
</div>

## Performance, by design

| Target | Budget |
| --- | --- |
| **Frame rate** | Locked 60 FPS on mobile silicon · 120 FPS on discrete desktop GPUs |
| **GPU payload** | ≤ 25 MB total texture memory through procedural shading |
| **Time to interact** | ≤ 1.2 s on 4G networks |
| **Geometry** | 45k-vertex draped textile mesh + 1.2k-vertex brutalist pedestal |
| **Thermals** | Sustained interactive orbit without thermal throttling |

## Architecture

```text
┌──────────────────────────┐       ┌─────────────────────────────┐
│ Touch / Pointer Inertia  │──────▶│ Dynamic Resolution Control  │
└──────────────────────────┘       └──────────────┬──────────────┘
                                                  │
                 ┌────────────────────────────────┴────────────────────────┐
                 ▼                                                         ▼
      ┌──────────────────────┐                              ┌────────────────────────┐
      │ WebAudio Spatial     │                              │ Three.js Scene Graph   │
      │ Friction Synthesizer │                              │ Custom PBR Material    │
      └──────────────────────┘                              └────────────┬───────────┘
                                                                        │
                                     ┌──────────────────────────────────┴──────────────────────────────────┐
                                     ▼                                                                     ▼
                          ┌────────────────────────┐                                      ┌────────────────────────┐
                          │ Procedural Weave Graph │                                      │ Anisotropic Sheen BRDF │
                          │ FBM relief + yarn twist│                                      │ GGX + directional fiber│
                          └────────────────────────┘                                      └────────────────────────┘
```

## What makes it feel real

- **Procedural twill shader graph** — A mathematical cellular weave evaluator replaces multi-layer diffuse, normal, and roughness maps.
- **Anisotropic GGX highlights** — Explicit tangent and bitangent vectors reproduce the directional sheen of twill and wool fibers.
- **Subsurface transmission wrap** — Fold-aware edge lighting suggests micro-light transmission through thin textile surfaces.
- **Procedural friction audio** — Native WebAudio buffers generate tactile contact sound without downloading audio assets.
- **Editorial chiaroscuro** — A 50/50 split-frame presentation pairs Didone-inspired typography with a pitch-black viewport.
- **Adaptive rendering** — Dynamic resolution scaling protects the frame-rate target across mobile and desktop hardware.

## Technology

| Layer | Implementation |
| --- | --- |
| Framework | Next.js · App Router |
| 3D engine | Three.js · `RawShaderMaterial` · custom shader pipeline |
| Shading | GLSL ES 3.0 · procedural cellular noise · anisotropic BRDF |
| Audio | WebAudio API · zero-asset procedural band-pass noise engine |
| Deployment | Vercel Edge Network |

## Shader core

The fragment shader calculates woven surface relief, anisotropic microfacet distribution, and subsurface transmission directly on the GPU:

```glsl
float getProceduralWeave(vec2 uv) {
    vec2 p = uv * 350.0;
    float threadA = cos(p.x) * sin(p.y);
    float threadB = sin(p.x) * cos(p.y);
    return mix(threadA, threadB,
        step(0.0, sin(p.x * 0.5 + p.y * 0.5))) * 0.08;
}

float D_GGX_Anisotropic(
    float NoH, float ToH, float BoH, float ax, float ay
) {
    float a2 = ax * ay;
    vec3 v = vec3(ay * ToH, ax * BoH, a2 * NoH);
    float v2 = dot(v, v);
    float w2 = a2 / v2;
    return a2 * w2 * w2 * (1.0 / PI);
}
```

## Try it live

<a href="https://aura-luxury-haute-3d.vercel.app">
  <img src="https://img.shields.io/badge/LAUNCH%20AURA%20LUXURY%20HAUTE-c9a46c?style=for-the-badge&logo=vercel&logoColor=white" alt="Launch AURA LUXURY HAUTE" />
</a>

The live experience is available at **[aura-luxury-haute-3d.vercel.app](https://aura-luxury-haute-3d.vercel.app)**.

---

<p align="center">
  <sub>ALH-PBR · Procedural textile rendering for the web.</sub>
</p>

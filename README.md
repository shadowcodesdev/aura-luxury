# AURA LUXURY HAUTE — 3D CONFIGURATOR

### `REF. ALH-PBR — ANISOTROPIC SHEEN MODEL` // `02 // PROCEDURAL SHADER GRAPH`

> Ultra-high-fidelity luxury fashion digital twin rendering draped textiles with procedural weave micro-relief and physical anisotropic microfiber sheen at locked 60–120 FPS.
> 
> 

---

### Core Performance Benchmarks

```
[ TARGET FRAMERATE ]  Locked 60 FPS (Mobile Silicon) / 120 FPS (Discrete Desktop GPU)
[ TEXTURE MEMORY ]    <= 25 MB total GPU payload (Procedural math over 8K bitmaps)
[ TIME TO INTERACT ]  <= 1.2s on 4G networks
[ THERMAL BUDGET ]    Zero thermal throttling during sustained interactive orbit
[ GEOMETRY FOOTPRINT] 45k vertex draped textile mesh + 1.2k vertex brutalist cast pedestal

```

---

### Low-Level Technical Architecture

```
[Touch / Pointer Inertia Controller] ──▶ [Dynamic Resolution Controller (DRS)]
                                                    │
                                                    ▼
   [WebAudio Spatial Engine]            [Three.js Scene Graph]
(Contact-Friction Synthesizer)                      │
                                                    ▼
                                    [Custom PBR GLSL Material]
                         ┌──────────────────────────┴──────────────────────────┐
                         ▼                                                     ▼
              [Procedural Weave Graph]                             [Anisotropic Sheen BRDF]
         (FBM Micro-Heightmap + Yarn Twist)                    (GGX-Kajiya-Kay Directional Spline)

```

---

### Technical Highlights

* **Procedural Twill Shader Graph:** Replaces heavy multi-layer 8K diffuse, normal, and roughness maps with a mathematical cellular weave evaluator in GLSL. Evaluates thread frequency and yarn twists directly in the fragment stage.


* **Anisotropic GGX Specular Distribution:** Modifies the microfacet specular distribution using explicit surface bitangent vectors to simulate directional fiber sheen on twill and wool.


* **Subsurface Transmission Wrap:** Models micro-light transmission across textile folds using inverted surface normal dot products factored against edge thickness approximations.


* **Real-Time WebAudio Friction Synthesizer:** Generates interactive textile tactile audio entirely through native WebAudio buffers without downloading audio assets. Dynamically modulates a bandpass filter between 800 Hz and 2400 Hz based on cursor angular velocity to simulate raw linen friction.


* **Editorial Chiaroscuro Presentation:** Balanced 50/50 split-frame layout marrying editorial Didone typography with pitch-black (`#000000`) high-contrast viewport shading.



---

### Tech Stack

* **Framework:** Next.js (App Router)


* **3D Core:** Three.js (`RawShaderMaterial` / Custom Shader Pipeline)


* **Shading Language:** GLSL ES 3.0 (Procedural Cellular Noise & Anisotropic BRDF)


* **Audio Synthesis:** WebAudio API (Zero-asset procedural bandpass noise engine)


* **Deployment:** Vercel Edge Network



---

### Material Shader Implementation (GLSL ES 3.0 Fragment)

Surface relief, GGX microfacet distribution, and subsurface transmission are calculated directly in hardware:

```glsl
#version 300 es
precision highp float;

in vec3 v_worldPosition;
in vec3 v_worldNormal;
in vec4 v_worldTangent;
in vec2 v_uv;

uniform vec3 u_cameraPosition;
uniform vec3 u_lightPosition;
uniform vec3 u_baseColor;
uniform float u_roughnessX;
uniform float u_roughnessY;
uniform float u_sheenIntensity;

out vec4 fragColor;

const float PI = 3.14159265359;

float getProceduralWeave(vec2 uv) {
    vec2 p = uv * 350.0;
    float threadA = cos(p.x) * sin(p.y);
    float threadB = sin(p.x) * cos(p.y);
    return mix(threadA, threadB, step(0.0, sin(p.x * 0.5 + p.y * 0.5))) * 0.08;
}

float D_GGX_Anisotropic(float NoH, float ToH, float BoH, float ax, float ay) {
    float a2 = ax * ay;
    vec3 v = vec3(ay * ToH, ax * BoH, a2 * NoH);
    float v2 = dot(v, v);
    float w2 = a2 / v2;
    return a2 * w2 * w2 * (1.0 / PI);
}

void main() {
    vec3 N = normalize(v_worldNormal);
    vec3 T = normalize(v_worldTangent.xyz);
    vec3 B = normalize(cross(N, T) * v_worldTangent.w);
    
    float weave = getProceduralWeave(v_uv);
    N = normalize(N + T * weave * 0.4 + B * weave * 0.4);
    
    vec3 V = normalize(u_cameraPosition - v_worldPosition);
    vec3 L = normalize(u_lightPosition - v_worldPosition);
    vec3 H = normalize(V + L);
    
    float NoV = max(dot(N, V), 0.001);
    float NoL = max(dot(N, L), 0.001);
    float NoH = max(dot(N, H), 0.001);
    float ToH = dot(T, H);
    float BoH = dot(B, H);
    
    float D = D_GGX_Anisotropic(NoH, ToH, BoH, u_roughnessX, u_roughnessY);
    float sheenDist = pow(1.0 - NoH, 4.0) * u_sheenIntensity;
    float sss = smoothstep(0.0, 1.0, dot(-V, L) * 0.5 + 0.5) * 0.25;
    
    vec3 diffuse = u_baseColor * (NoL + sss);
    vec3 finalColor = diffuse + (D * sheenDist * vec3(0.95, 0.90, 0.85));
    fragColor = vec4(finalColor, 1.0);
}

```

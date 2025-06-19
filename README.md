# GLTF-Compression

## GLTF Transform

Docs:
https://gltf-transform.dev/cli

1. gltf-transform optimize statue.glb compress.glb: DONT USE THIS
2.  gltf-transform draco statue.glb compress.glb: DONT USE THIS
3. Texture size, ex: gltf-transform resize -- width 1024 --height 1024 statue.glb compress.glb -> Can be skipped and done in Blender
4. KTX, ex: gltf-transform uastc statue.glb compress.glb

*Use GTLF Report to check VRAM and Draw calls and adjust as needed.*

We can target specific textured if needed and keep textures small in resolution as possible.

Best to combine many materials into oone texture:
https://www.youtube.com/watch?v=wG6ON8wZYLc&t=4s

## GLTFJSX

Docs:
https://github.com/pmndrs/gltfjsx 
For model optimization use GLTFJSX instead

$ npx gltfjsx [Model.glb] [options]

1. npx gltfjsx model.glb --transform

-------------------------------------------------------------------------------------------------------

More clear - CHATGPT response:

✅ Recommended Optimization Order for GLTF Files
1. Optimize Textures First (e.g., using gltf-transform)
Why:

Textures usually take up most of the file size.

Compressing them with ktx2 (Basis Universal or UASTC) can significantly reduce file size and improve runtime performance.

Tools:

gltf-transform – Use commands like:<br>
gltf-transform uastc input.glb output.glb<br>
gltf-transform prune output.glb output-compressed.glb<br>

This compresses textures without modifying geometry or scene structure.

2. Optimize Geometry / Convert to JSX with gltfjsx
Why:

After you reduce texture size, converting the model into JSX makes the Three.js integration easier and can further reduce runtime overhead by:

Decimating geometry.

Removing unused nodes/materials.

Splitting into reusable components.

Tool:

gltfjsx

npx gltfjsx output-compressed.glb
This creates a React/Three.js component from your optimized .glb.

🚫 If You Reverse the Order:
If you run gltfjsx first, and then optimize with gltf-transform, you might end up:

Repeating compression on already converted assets.

Missing optimizations (gltf-transform prune, draco, etc.) that affect internal GLTF structure and reduce size.

Not getting the benefit of compressed textures in your JSX file unless you re-import or modify the JSX manually.

🔄 Optional Final Step:
After both steps, you can re-run gltf-transform prune to remove any unused elements introduced during conversion.

✅ Summary of Best Order:

[Original GLB/GLTF]
      ↓
[Compress textures + optimize via gltf-transform]
      ↓
[Convert optimized GLB to JSX via gltfjsx]
Let me know if you're using Draco, Meshopt, or UASTC and I can tailor a command sequence for your use case.








You said:
## What if I dont want a react component?


ChatGPT said:
If you don’t want a React component, and you're working with vanilla Three.js, then you likely don’t need gltfjsx at all — that tool is specifically for converting GLTF files into React Three Fiber components.

Instead, your optimization pipeline should look like this:

✅ Recommended Optimization Workflow for Vanilla Three.js (no React):

1. Start with Texture Compression (using gltf-transform)<br>
This is your most important optimization for real-world web performance.

Use UASTC or ETC1S with KTX2 to compress textures.

gltf-transform uastc input.glb output.glb<br>

You can also run:

gltf-transform prune output.glb output-optimized.glb<br>  

Optional: Use draco or meshopt for geometry compression.
gltf-transform draco output-optimized.glb output-final.glb<br>  


2. Load in Vanilla Three.js
You now just load the final .glb file using GLTFLoader in Three.js:

JS:<br>
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js';

const loader = new GLTFLoader();
loader.load('models/output-final.glb', (gltf) => {
  scene.add(gltf.scene);
});
Optional Geometry Simplification:
If you're also trying to reduce polygon count (not just texture size), you can:

Use Blender to decimate the mesh and export a new .glb.

Use a CLI tool like meshoptimizer or gltf-transform simplify.

Example:

bash
Copy
Edit
gltf-transform simplify input.glb output-simplified.glb --ratio 0.5
✅ Summary for Non-React Three.js Workflow:
csharp
Copy
Edit
[Original GLB/GLTF]
      ↓
[Compress Textures → gltf-transform (uastc, prune, etc.)]
      ↓
[Optional: Simplify geometry with gltf-transform or Blender]
      ↓
[Load in Three.js via GLTFLoader]


--

## Reducing draw calls:

✅ 1. Merge Meshes with the Same Material
Each mesh with a unique material = 1 draw call.

Solution: Merge meshes that share the same material into a single mesh.

Tools:
Blender:

Join objects (Ctrl + J) that use the same material.

gltf-transform:
gltf-transform weld input.glb output.glb
gltf-transform merge output.glb merged.glb

merge combines compatible nodes/meshes.
weld removes duplicated vertices.

2. Use a Single Material or Texture Atlas
Problem: More materials = more draw calls.

Solution 1: Use one material per model.
Combine all texture maps into a single texture atlas (e.g., in Blender or with tools like TexturePacker).

Map UVs accordingly.

Solution 2: Use gltf-transform instance to reuse geometry without duplicating draw calls (ideal for trees, props, etc.)


✅ 3. Use Instancing (for repeated objects)
If you have many identical objects (like trees, rocks, tiles), draw them with InstancedMesh instead of adding each one individually.

Example:
js
Copy
Edit
const geometry = new BoxGeometry();
const material = new MeshStandardMaterial();
const mesh = new InstancedMesh(geometry, material, 100);

for (let i = 0; i < 100; i++) {
  const matrix = new Matrix4().makeTranslation(i, 0, 0);
  mesh.setMatrixAt(i, matrix);
}
scene.add(mesh);
Result:
All 100 boxes = 1 draw call.

✅ 4. Avoid Too Many Skinned Meshes or Lights
Each skinned mesh or light increases shader complexity and draw calls.

Consider baking lighting and animations into textures or morph targets.

✅ 5. Bake Materials and Lighting
Instead of using real-time lighting and shadows:

Bake everything into textures (lightmaps, AO).

Use unlit or simple materials.

Reduces shaders and draw calls.

✅ 6. Remove Unused Nodes and Materials
Use:

bash
Copy
Edit
gltf-transform prune input.glb output.glb
This will delete:

Unused meshes

Unused materials

Unused nodes

🔍 Use gltf-transform inspect or tools like gltf.report to analyze draw calls and scene structure.

✅ Summary: How to Reduce Draw Calls
| Strategy                            | Tool/Method                     |
| ----------------------------------- | ------------------------------- |
| Merge meshes with same material     | Blender, `gltf-transform merge` |
| Use fewer materials / texture atlas | Blender, TexturePacker          |
| Use instancing for repeated items   | Three.js `InstancedMesh`        |
| Prune unused nodes/materials        | `gltf-transform prune`          |
| Bake lighting into textures         | Blender light baking            |

Baking materials into one texture map:
https://www.youtube.com/watch?v=wG6ON8wZYLc&t=4s

--

## So there should be a new .glb file for say when we use uastc and then prune have  a new .glb file too?

Yes — each time you apply an optimization step like uastc, prune, simplify, or merge, you should generate a new .glb file as the output. This gives you:
A clean chain of transformation (easier to debug or rollback), <br>
Full control over each step’s result, <br>
The ability to analyze size/performance gains per step. <br>

✅ Example Workflow with .glb Versions
Let’s say your original file is:

model-original.glb
You would run steps like this:

1. Compress textures using UASTC

gltf-transform uastc model-original.glb model-uastc.glb<br>
model-uastc.glb now has GPU-compressed textures (KTX2).

2. Prune unused data

gltf-transform prune model-uastc.glb model-uastc-pruned.glb<br>
model-uastc-pruned.glb removes unused materials, nodes, etc.

3. Optional: Merge geometries or weld vertices

gltf-transform merge model-uastc-pruned.glb model-merged.glb<br>
model-merged.glb now has fewer draw calls from merged meshes.

4. Optional: Geometry simplification

gltf-transform simplify model-merged.glb model-simplified.glb --ratio 0.7<br>
model-simplified.glb now has lower poly count.

🎯 Final File:
Use the last one (model-simplified.glb, or whichever is final in your chain) in Three.js or React Three Fiber.

✅ Benefits of Creating New Files for Each Step:
Reason	                                Benefit
Debug step-by-step	                    See what broke and where
Analyze file size reductions	              Optimize each step separately
Easier backups / rollbacks	              Restore from any step
Safer CLI use	                          Avoid overwriting accidentally


# GLB Optimization Workflow for Three.js (Vanilla)

This workflow shows how to optimize a `.glb` file step-by-step using [gltf-transform](https://gltf-transform.donmccurdy.com/) and [Draco compression](https://google.github.io/draco/), producing smaller files with fewer draw calls and better runtime performance.

---

## 🛠️ Optimization Steps

| Step | Command Example | Input File | Output File | Purpose |
|------|------------------|------------|-------------|---------|
| 1. Compress Textures (UASTC) | `gltf-transform uastc model-original.glb model-uastc.glb` | `model-original.glb` | `model-uastc.glb` | GPU-friendly texture compression (KTX2 + UASTC) |
| 2. Prune Unused Data | `gltf-transform prune model-uastc.glb model-uastc-pruned.glb` | `model-uastc.glb` | `model-uastc-pruned.glb` | Remove unused nodes, materials, textures |
| 3. Merge Meshes | `gltf-transform merge model-uastc-pruned.glb model-merged.glb` | `model-uastc-pruned.glb` | `model-merged.glb` | Merge meshes to reduce draw calls |
| 4. Simplify Geometry (Optional) | `gltf-transform simplify model-merged.glb model-simplified.glb --ratio 0.7` | `model-merged.glb` | `model-simplified.glb` | Reduce polygon count (optional) |
| 5. Compress Geometry (Draco) | `gltf-transform draco model-simplified.glb model-draco.glb` | `model-simplified.glb` | `model-draco.glb` | Compress geometry using Draco to reduce file size |

---

## ✅ Final Output

Use `model-draco.glb` (or the latest step you completed) in your Three.js project:

```js
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js';
import { DRACOLoader } from 'three/examples/jsm/loaders/DRACOLoader.js';

const loader = new GLTFLoader();

const dracoLoader = new DRACOLoader();
dracoLoader.setDecoderPath('/path/to/draco/'); // e.g., './draco/' folder with decoder files
loader.setDRACOLoader(dracoLoader);

loader.load('model-draco.glb', (gltf) => {
  scene.add(gltf.scene);
});


---

## ✅ Final Output

Use `model-draco.glb` (or the latest step you completed) in your Three.js project:

```js
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js';
import { DRACOLoader } from 'three/examples/jsm/loaders/DRACOLoader.js';

const loader = new GLTFLoader();

const dracoLoader = new DRACOLoader();
dracoLoader.setDecoderPath('/path/to/draco/'); // e.g., './draco/' folder with decoder files
loader.setDRACOLoader(dracoLoader);

loader.load('model-draco.glb', (gltf) => {
  scene.add(gltf.scene);
});

## 🔍 Optional Tools for Analysis

| Tool | Purpose |
|------|---------|
| [`gltf-transform inspect`](https://gltf-transform.donmccurdy.com/) | CLI tool to inspect mesh/material count, draw calls |
| [gltf.report](https://gltf.report) | Online viewer to inspect performance & validation |
| [gltf-viewer.donmccurdy.com](https://gltf-viewer.donmccurdy.com/) | Web viewer for checking compression & rendering |

📝 Notes

- Draco compression requires the decoder (JavaScript or WASM) to be included in your project.
- Texture compression (UASTC) requires `KTX2Loader`, Basis Universal transcoder, and proper `KTX2Loader` setup (optional for basic GLB loading).
- This workflow keeps all steps modular and reversible.




  


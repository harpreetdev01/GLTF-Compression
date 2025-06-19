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



  


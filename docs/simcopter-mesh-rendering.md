Rendering a .MAX mesh

## Introduction

Geometry files (geo/*.MAX) contain the vertex data. Textures are packed (mostly) into BMP/sim3d.bmp. Geometry file either uses plain colors or has a reference to a single texture in a texture atlas in sim3d.bmp. 

## Details

### Texturing

Each face has a 'tex_file' parameter. This is id of the texture atlas to use. Each face also has a 'texnum' parameter which is the index to the texture in the texture atlas.
The 'texnum' index starts from bottom left corner of the atlas. Hence, texnum '0' is the bottom-left texture, '1' the texture on its right side, '8' the texture in the first column on the second-bottommost row and so on.

Texture coordinates seem be to mapped from the 'face vertex flags'. 
2nd flag seems to be the x- texcoord. For instance: (-1 at left side, 0 at right side means repeat texture horizontally once)
4th (last) flag seems to be the y- texcoord. For instance: (-11 at ground level, 0 at ceiling means repeat texture vertically 12 times)
For calculating texture repeat counts what seems to matter is the difference between x or y flag between vertices rather than the absolute value.
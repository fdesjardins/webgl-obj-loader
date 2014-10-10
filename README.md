# webgl-obj-loader
A simple script to help bring OBJ models to your WebGL world. I originally
wrote this script for my CS Graphics class so that we didn't have to only have
cubes and spheres for models in order to learn WebGL. At the time, the only
sort of model loader for WebGL was [Mr. Doob's ThreeJS](http://threejs.org/). And in order to use the
loaders you had to use the entire framework (or do some very serious hacking
and duct-taping in order get the model information). My main focus in creating
this loader was to easily allow importing models without having to have special
knowledge of a 3D graphics program (like Blender) while keeping it low-level
enough so that the focus was on learning WebGL rather than learning some
framework.

## Mesh(objStr)

The main Mesh class. The constructor will parse through the OBJ file data
and collect the vertex, vertex normal, texture, and face information. This
information can then be used later on when creating your VBOs. Look at the
`initMeshBuffers` source for an example of how to use the newly created Mesh

### Attributes:
* **vertices:** an array containing the vertex values that correspond to each unique face index. The array is flat in that each vertex's component is an element of the array. For example: with `verts = [1, -1, 1, ...]`, `verts[0] is x`, `verts[1] is y`, and `verts[2] is z`. Continuing on, `verts[3]` would be the beginning of the next vertex: its x component. This is in preparation for using `gl.ELEMENT_ARRAY_BUFFER` for the `gl.drawElements` call.
* **vertexNormals:** an array containing the vertex normals that correspond to each unique face index. It is flat, just like `vertices`.
* **textures:** an array containing the `s` and `t` (or `u` and `v`) coordinates for this mesh's texture. It is flat just like `vertices` except it goes by groups of 2 instead of 3.
* **indices:** an array containing the indicies to be used in conjunction with the above three arrays in order to draw the triangles that make up faces. For example, `indices[42]` could contain the number `38`. This would then be used internally by WebGL on all three of the above arrays telling which vertex, normal, and texture coords to use: `vertices[38]`, `vertexNormals[38]`, and `textures[38]`.

### Params:

* **objStr** a string representation of an OBJ file with newlines preserved.

A simple example:

In your `index.html` file:
```html
<html>
    <head>
        <script type="text/plain" id="my_cube.obj">
            ####
            #
            #	OBJ File Generated by Blender
            #
            ####
            o my_cube.obj
            v 1 1 1
            v -1 1 1
            v -1 -1 1
            v 1 -1 1
            v 1 1 -1
            v -1 1 -1
            v -1 -1 -1
            v 1 -1 -1
            vn 0 0 1
            vn 1 0 0
            vn -1 0 0
            vn 0 0 -1
            vn 0 1 0
            vn 0 -1 0
            f 1//1 2//1 3//1
            f 3//1 4//1 1//1
            f 5//2 1//2 4//2
            f 4//2 8//2 5//2
            f 2//3 6//3 7//3
            f 7//3 3//3 2//3
            f 7//4 8//4 5//4
            f 5//4 6//4 7//4
            f 5//5 6//5 2//5
            f 2//5 1//5 5//5
            f 8//6 4//6 3//6
            f 3//6 7//6 8//6
        </script>
    </head>
</html>
```
And in your `app.js`:

```javascript
var gl = canvas.getContext('webgl');
var objStr = document.getElementById('my_cube.obj').innerHTML;
var mesh = new OBJ.Mesh(objStr);

// use the included helper function to initialize the VBOs
// if you don't want to use this function, have a look at its
// source to see how to use the Mesh instance.
OBJ.initMeshBuffers(gl, mesh);
// have a look at the initMeshBuffers docs for an exmample of how to
// render the model at this point

```

## Some helper functions

### downloadMeshes(nameAndURLs, completionCallback, meshes)

Takes in a JS Object of `mesh_name`, `'/url/to/OBJ/file'` pairs and a callback
function. Each OBJ file will be ajaxed in and automatically converted to
an OBJ.Mesh. When all files have successfully downloaded the callback
function provided will be called and passed in an object containing
the newly created meshes.

**Note:** In order to use this function as a way to download meshes, a
webserver of some sort must be used.

#### Params:

* **nameAndURLs:** an object where the key is the name of the mesh and the value is the url to that mesh&#39;s OBJ file

* **completionCallback:** should contain a function that will take one parameter: an object array where the keys will be the unique object name and the value will be a Mesh object

* **meshes:** In case other meshes are loaded separately or if a previously declared variable is desired to be used, pass in a (possibly empty) json object of the pattern: `{ 'mesh_name': OBJ.Mesh }`

A simple example:


```javascript
var app = {};
    app.meshes = {};

var gl = document.getElementById('mycanvas').getContext('webgl');

function webGLStart(meshes){
  app.meshes = meshes;
  // initialize the VBOs
  OBJ.initMeshBuffers(gl, app.meshes.suzanne);
  OBJ.initMeshBuffers(gl, app.meshes.sphere);
  ... other cool stuff ...
  // refer to the initMeshBuffers docs for an example of
  // how to render the mesh to the screen after calling
  // initMeshBuffers
}

window.onload = function(){
  OBJ.downloadMeshes({
    'suzanne': 'models/suzanne.obj', // located in the models folder on the server
    'sphere': 'models/sphere.obj'
  }, webGLStart);
}
```

### initMeshBuffers(gl, mesh)

Takes in the WebGL context and a Mesh, then creates and appends the buffers
to the mesh object as attributes.

#### Params:

* **gl** *WebGLRenderingContext* the `canvas.getContext('webgl')` context instance

* **mesh** *Mesh* a single `OBJ.Mesh` instance

The newly created mesh attributes are:

Attrbute | Description
:--- | ---
**normalBuffer**       |contains the model&#39;s Vertex Normals
normalBuffer.itemSize  |set to 3 items
normalBuffer.numItems  |the total number of vertex normals
**textureBuffer**      |contains the model&#39;s Texture Coordinates
textureBuffer.itemSize |set to 2 items
textureBuffer.numItems |the number of texture coordinates
**vertexBuffer**       |contains the model&#39;s Vertex Position Coordinates (does not include w)
vertexBuffer.itemSize  |set to 3 items
vertexBuffer.numItems  |the total number of vertices
**indexBuffer**        |contains the indices of the faces
indexBuffer.itemSize   |is set to 1
indexBuffer.numItems   |the total number of indices

A simple example (a lot of steps are missing, so don't copy and paste):
```javascript
var gl   = canvas.getContext('webgl'),
var mesh = new OBJ.Mesh(obj_file_data);
// compile the shaders and create a shader program
var shaderProgram = gl.createProgram();
// compilation stuff here
...
// make sure you have vertex, vertex normal, and texture coordinate
// attributes located in your shaders and attach them to the shader program
shaderProgram.vertexPositionAttribute = gl.getAttribLocation(shaderProgram, "aVertexPosition");
gl.enableVertexAttribArray(shaderProgram.vertexPositionAttribute);

shaderProgram.vertexNormalAttribute = gl.getAttribLocation(shaderProgram, "aVertexNormal");
gl.enableVertexAttribArray(shaderProgram.vertexNormalAttribute);

shaderProgram.textureCoordAttribute = gl.getAttribLocation(shaderProgram, "aTextureCoord");
gl.enableVertexAttribArray(shaderProgram.textureCoordAttribute);

// create and initialize the vertex, vertex normal, and texture coordinate buffers
// and save on to the mesh object
OBJ.initMeshBuffers(gl, mesh);

// now to render the mesh
gl.bindBuffer(gl.ARRAY_BUFFER, mesh.vertexBuffer);
gl.vertexAttribPointer(shaderProgram.vertexPositionAttribute, mesh.vertexBuffer.itemSize, gl.FLOAT, false, 0, 0);

gl.bindBuffer(gl.ARRAY_BUFFER, mesh.textureBuffer);
gl.vertexAttribPointer(shaderProgram.textureCoordAttribute, mesh.textureBuffer.itemSize, gl.FLOAT, false, 0, 0);

gl.bindBuffer(gl.ARRAY_BUFFER, mesh.normalBuffer);
gl.vertexAttribPointer(shaderProgram.vertexNormalAttribute, mesh.normalBuffer.itemSize, gl.FLOAT, false, 0, 0);

gl.bindBuffer(gl.ELEMENT_ARRAY_BUFFER, model.mesh.indexBuffer);
gl.drawElements(gl.TRIANGLES, model.mesh.indexBuffer.numItems, gl.UNSIGNED_SHORT, 0);
```

### deleteMeshBuffers(gl, mesh)
Deletes the mesh's buffers, which you would do when deleting an object from a
scene so that you don't leak video memory.  Excessive buffer creation and
deletion leads to video memory fragmentation.  Beware.

## Node.js
`npm install webgl-obj-loader`

```javascript
var fs = require('fs');
var OBJ = require('webgl-obj-loader');

var meshPath = './development/models/sphere.obj';
var opt = { encoding: 'utf8' };

fs.readFile(meshPath, opt, function (err, data){
  if (err) return console.error(err);
  var mesh = new OBJ.Mesh(data);
});
```

## Demo
http://frenchtoast747.github.com/webgl-obj-loader/
This demo is the same thing inside of the gh-pages branch. Do a `git checkout gh-pages` inside of the webgl-obj-loader directory to see how the OBJ loader is used in a project.

## ChangeLog
**0.1.0**
* Dropped jQuery dependency: `downloadMeshes` no longer requires jQuery to ajax in the OBJ files.
* changed namespace to something a little shorter: `OBJ`
* Updated documentation

**0.0.3**
* Initial support for Quad models

**0.0.2**
* Texture Coordinates are now loaded into mesh.textures

**0.0.1**
* Vertex Normals are now loaded into mesh.vertexNormals

[![Bitdeli Badge](https://d2weczhvl823v0.cloudfront.net/frenchtoast747/webgl-obj-loader/trend.png)](https://bitdeli.com/free "Bitdeli Badge")

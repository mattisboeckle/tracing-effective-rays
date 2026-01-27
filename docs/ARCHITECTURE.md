## Table of contents

- [Project structure](#project-structure)
  - [Main](#main)
  - [Test](#test)
  - [Lib](#lib)
    - [Common](#common)
    - [PPM](#ppm)
    - [Vec3](#vec3)
    - [Camera](#camera)
    - [Ray](#ray)


## Project Structure

### Main

`src/main.effekt` contains the entrypoint for the program and the definitions of what the scene contains, how the camera is configured and how to render it

### Test

`src/test.effekt` is the main entrypoint for testing. Calls test suites in `src/test/`

Currently tests
- `vec3`
- end-to-end rendering of an image without writing files

### Lib

Contains most of the code for this project and is split into different modules

#### Common
[common.effekt](../src/lib/common.effekt)

Contains type definitions and functions that are needed in other files and is not dependent on anything else in lib.
This file mostly exists to prevent circular dependency errors.

#### PPM
[ppm.effekt](../src/lib/ppm.effekt)

Handles writing images in the `.ppm` format. 
The format is written in plaintext and consists of a header
```
P3      # means RGB image in ASCII
400 255 # width & height
255     # maximum value for color 
```
followed by RGB triplets
```
255 0 0 # r g b
```
This is a very simple format that is not very disk space efficient but works for our simple purposes.

We also convert Vec3 ranging from 0.0 to 1.0 to our color space of 0 to 255 

#### Vec3
[vec3.effekt](../src/lib/vec3.effekt)

3-dimensional vector of doubles. 

Features
- basic arithmetic operations
- dot-/ and cross-product
- random generation 
- reflection & refraction

#### Camera
[camera.effekt](../src/lib/camera.effekt)

Calculates the viewport of the camera base on its input parameters.

Can generate random sampling Rays for a pixel.

Generates an image based on reading the colors those sampling Rays return.
Reading the colors is modeled as the effect `readColor` which can be handled in various ways.
Example handlers are in [Ray](#ray).

#### Ray
[ray.effekt](../src/lib/ray.effekt)

Types:
- Ray
- HitRecord
- Material
- Hittable

Effects:
- readColor
- Raytrace
    - hit
    - miss

Contains most of the math of shooting rays into a scene as well as what it means to hit something.

We have different handlers that either work independently like `constantColor`, `linearGradient` or `randomColor` and handlers that interact with the scene or "world" which consists of `Hittable` objects. In this program that is spheres.

These hittable objects can have a material that is either
- Lambertian (meaning one constant color)
- Metal (reflect rays instead of bouncing back towards the normal direction)
- Dielectric (Materials like glass or water that refract & reflect based on the vector of entry)

We calculate `Raytrace` effects for hittable objects that either `hit` the object at a some position and result in a `HitRecord` with information about the hit or `miss` it.

Calculating hits in a scene means returning the `HitRecord` for the closest object in the scene.

The "normal" raytracing handler `scatterRays` calculates hits in the scene up to a certain recursion depth. Which has the following steps if not at the depth limit.

1. Calculate the closest hit within the Scene which either
    1. We hit an object
        1. Get the attenuation of the material
        2. Scatter the ray based on the material of the hit
        3. Recurse with the scattered ray and reduced depth
        4. Resume with the aggregated result
    2. We missed all objects
        1. Resume with a gradient background of the scene 
     
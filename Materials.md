# Materials

See also:

[Textures](https://www.notion.so/Textures-5f2cd441f2614496986e12b7faecb367?pvs=21)

[Look Development Process](https://www.notion.so/Look-Development-Process-83217af900ee42b4ab8f5c2de48b6b48?pvs=21)

[Gradient Mapping](https://www.notion.so/Gradient-Mapping-2feb77a5d47b408c9bd7f67e12507748?pvs=21)

[Lighting](https://www.notion.so/Lighting-836bd696b3d946d48eb6f84835ab71a6?pvs=21)

# What are materials

A material is an asset that can be applied to geometry to define its shading. This shading is usually only visible when a light hits the surface, so lighting plays a huge role in materials. The exception is in shader models such as unlit. 

In a material you can define the value of parameters that are available from the shader, such as the color, roughness, and so on. Most of the time when we talk about a shader, it’s a BSDF based on the work Brian Karis. You can see the work that Brian Karis did in this paper:

[2013_Siggraph_UE4_BrianKaris.pdf](/img/2013_Siggraph_UE4_BrianKaris.pdf)

This collection of papers does a great job at explaining the ground work of real-time PBR: [https://blog.selfshadow.com/publications/s2012-shading-course/](https://blog.selfshadow.com/publications/s2012-shading-course/) 

Mind you, not all shaders are BSDFs! [https://www.shadertoy.com/](https://www.shadertoy.com/) has some great examples of crazy shaders written in HLSL. 

![shader_vs_material.png](/img/shader_vs_material.png)

![brick_wall.png](/img/brick_wall.png)

### BSDF vs BRDF vs BTDF

BSDF is a superset and the generalization of the BRDF and BTDF. The concept behind all BxDF functions could be described as a 4-dimensional function: a function of light direction (l) and view direction (v), the normal of the surface (n), and surface tangent (t). 

![brdf_vs_btdf.png](/img/brdf_vs_btdf.png)

![brdf_function.png](/img/brdf_function.png)

The Base Color, Roughness, or Normal inputs of a BSDF are all BRDFs in their own right, with their own calculations. These inputs are always numbers, but the way we describe those numbers can be using math, flat values, or textures. Textures are really just 2 or 3 dimensional collection of numbers. 

The bidirectional scattering distribution function (BSDF) radiometrically characterizes the scatter of optical radiation from a surface as a function of the angular positions of the incident and scattered beams. By definition, it is the ratio of the scattered radiance to the incident irradiance: the unit is inverse steradian. The term bidirectional reflectance distribution function (BRDF) is used when specifically referring to reflected scatter. Likewise, bidirectional transmittance distribution function (BTDF) refers to scatter transmitted through a material. Read more [here](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4930030/). 

![diffuse_and_specular.png](/img/diffuse_and_specular.png)

### Shader vs Material

A shader is a small program that tells the GPU how to *shade* an object on the screen and the calculations required to get there. 

Shaders have the following prerequisites:

- A shader needs to be attached to a material used
- A scripting language used to author shaders, such as HLSL and GLSL

A shader and a material can be used in different ways, depending on the game engine. In Unreal Engine, the shader used can be based on the target platform selected.

There are generally two ways to do materials; PBR (physically based rendering/shading) and NPR (non-photoreal rendering/shading). The type of shader you use doesn’t imply one or the other, because there are plenty of ways to “break” a PBR shader by giving it “wrong” inputs (more on that later). 

# PBS (physically based shading)

## PBS vs PBR

> Physically Based Rendering (PBR) is a method of shading and rendering that provides a more accurate representation of how light interacts with surfaces. It can be referred to as Physically Based Rendering (PBR) or Physically Based Shading (PBS). Depending on what aspect of the pipeline is being discussed, PBS is usually specific to shading concepts and PBR specific to rendering and lighting. However, both terms describe on a whole, the process of representing assets from a physically accurate standpoint. - Wes McDermott
> 

[the-pbr-guide](https://learndownload.adobe.com/pub/learn/substance-3d-designer/the-pbr-guide.pdf)

## Why is it important?

Less guessing, more out-of-the-box, more guidelines online, self-diagnosable. 

## Getting started

![is_surface_metal.png](/img/is_surface_metal.png)

## Physically based values

Educate yourself with some good values. 

[https://seblagarde.wordpress.com/2014/04/14/dontnod-physically-based-rendering-chart-for-unreal-engine-4/](https://seblagarde.wordpress.com/2014/04/14/dontnod-physically-based-rendering-chart-for-unreal-engine-4/) the original. 

[https://google.github.io/filament/Material Properties.pdf](https://google.github.io/filament/Material%20Properties.pdf) & [https://google.github.io/filament/images/material_chart.jpg](https://google.github.io/filament/images/material_chart.jpg) very good image explaining all the parameters. 

[https://physicallybased.info/](https://physicallybased.info/) has slightly more numbers. 

[https://gbaran.gumroad.com/l/rjthlk](https://gbaran.gumroad.com/l/rjthlk) has a lot of numbers. 

[https://docs.unrealengine.com/5.2/en-US/physically-based-materials-in-unreal-engine/](https://docs.unrealengine.com/5.2/en-US/physically-based-materials-in-unreal-engine/) the Unreal documentation also has numbers. 

I found another source that notes this:

- Red: Dielectric materials must be inside [30-240] sRGB
- Orange: Metallic materials must be inside [186-255] sRGB
- Blue: Glass materials must have a fresnel reflectance inside [52-114] sRGB
- Yellow: Metallic materials must have a value near 0 or near 1, in-between values are often an error

[https://physicallybased.info/tools/](https://physicallybased.info/tools/) Tool to transform between color encodings. 

![srgb_notation.png](/img/srgb_notation.png)

![dontnodgraphicchartforunrealengine4.png](/img/dontnodgraphicchartforunrealengine4.png)

![filament_pbr_chart.jpeg](/img/filament_pbr_chart.jpeg)

### Diffuse (sometimes referred to as base color or albedo)

Correct values should generally fall within the darkest and brightest natural element found on earth, coal and fresh snow. For dielectric (non-metals) this is:

- sRGB 50 - 243
- Linear 0.0319 - 0.89627

For non-dielectric (metals) this range is:

- sRGB 180 - 255
- Linear 0.45641 - 1

You can test these values in Substance Painter/Designer using the PBR Validate nodes. In Unreal you can build a post-process material that can validate it for you:

[https://80.lv/articles/validating-pbr-materials-in-ue4/](https://80.lv/articles/validating-pbr-materials-in-ue4/)

The name *diffuse* comes from the fact that they are responsible for *diffuse reflection* (see Diffuse reflection vs Specular reflection), which is the phenomena responsible for giving materials a color in real life. In general, diffuse values are much brighter then you’d think, which makes authoring them a little bit challenging. If your diffuse maps are too dark, you will see much less light bouncing around the scene than you would expect (see Roughness vs Specular). 

![diffuse_only_remember_me.png](/img/diffuse_only_remember_me.png)

### Diffuse in dieletric (metallic) materials

If a shader’s metallic input is above a certain value (near 1.0), the color of the diffuse map is used to color the specular response. That is also why diffuse maps for metals should be authored very brightly (>180 sRGB). 

### Specular (and cavity)

Specular should generally be kept at .5 which equals to about 4% reflectance, which is good for the vast majority of materials. However, we can use Cavity Maps (small scale ambient occlusion) and multiply with .5 to help the rendering engine occlude more on small scale. This can be helpful because real-time renderers can’t get so granular as is sometimes required. 

### Specular resources

[https://campi3d.com/External/MariExtensionPack/userGuide5R8/IORtoSpecularLeveldielectric.html](https://campi3d.com/External/MariExtensionPack/userGuide5R8/IORtoSpecularLeveldielectric.html) 

[https://mssiphiwemoyo.com/blog/a-z-ior-values-complete-guide/](https://mssiphiwemoyo.com/blog/a-z-ior-values-complete-guide/) 

[https://physicallybased.info/tools/](https://physicallybased.info/tools/) Calculator for translating IOR values into Specular ones. 

[https://pixelandpoly.com/ior.html](https://pixelandpoly.com/ior.html) 

### Roughness

In real life, the phenomena of roughness is the result of light scattering inside the material into many different directions due to micro-detail in the material (on the micro-meter scale). The light that is exiting the material again is considered “bounce light”. 

The surface shown here on the right is too small scale to approach in geometry, so in rendering engines we lean on roughness maps, but also micro-normal maps to emulate it.

In materials, the roughness input controls how rough or smooth a surface is. In the material this manifests as how sharp or blurry reflections appear on the material. 

*Hard* materials have very dense structures and therefore do not allow the light to scatter inside their structures a lot. *Soft* materials (like skin) have a lot of scattering going on, which we can replicate using *subsurface scattering*. 

![light_scattering.png](/img/light_scattering.png)

![smoothness_roughness_surface.jpg](/img/smoothness_roughness_surface.jpg)

Not only roughness influences the sharpness of the reflection, also a micro-normal emulating tiny geometric details influences the sharpness of the reflection. 

### Diffuse reflection vs specular reflection

It’s important to note the difference between the specular reflection and the diffuse reflection of a material. 

The diffuse reflection is responsible for scattering light *around*, most of which appears to us via surrounding surfaces. Specular reflection is the light that is reflected directly into our eyes. 

In the case of colored objects (absorbent), diffused rays will lose some wavelengths during their walk in the material, and will emerge colored. And, when a colored object has both diffuse and specular reflection, usually only the diffuse component is colored. 

A cherry reflects diffusely red light, absorbs all other colors and has a specular reflection which is essentially white (if the incident light is white light). This is quite general, because, except for metals, the reflectivity of most materials depends on their refractive index, which varies little with the wavelength, so that all colors are reflected nearly with the same intensity.

![diffuse_and_specular_reflection.png](/img/diffuse_and_specular_reflection.png)

[Diffuse reflection](https://en.wikipedia.org/wiki/Diffuse_reflection)

### Roughness vs specular

In shaders, roughness and specular (reflectivity) are not the same thing and nor are they tied together. Roughness controls the microsurface detail which means how blurry or sharp the refection is. Specular controls how much light a surface reflects and the color of the reflection.

Specularity refers to the amount of specular light reflected by a surface. This value is inherent to the Material type, and usually the default value of 0.5 is accurate. The Specular input is not used for reflection/specularity maps or to add surface variation. These should be handled in the Roughness map. Unreal Engine uses a default Specular of 0.5, which represents approximately 4% specular reflection. This value is accurate for a vast majority of materials.

Contrary to instinct, the roughness of a material has a minimal impact on the diffuse reflection but a great impact on the specular reflection. 

![impact_of_roughness_on_bounce_lighting_v2.jpg](/img/impact_of_roughness_on_bounce_lighting_v2.jpg)

A white light shines perpendicular on a flat plane. The resulting light scattered around is the result of diffuse reflections (light bounces). 

![impact_of_roughness_on_bounce_lighting_v3.jpg](/img/impact_of_roughness_on_bounce_lighting_v3.jpg)

Reducing the value of the color of the object directly reduces the amount of light being scattered around (light bounce). 

### Specular vs Roughness workflows

Most engines assume a roughness workflow, with most artist working in the roughness workflow and leaving specular at default (0.5 == 4% specular). Notable difference being the CryEngine, which only uses specular based materials. This was done because when they had to decide between roughness or specular, they were building RYSE Son of Rome and that project required a lot of good metallic transitions, something that causes artifacts in the roughness workflow. 

Roughness is a parameter that is easily understood by artists, whereas specular is a bit more involved and requires more knowledge. Before the BSDF shader models, everything was done via specular. 

### Specular workflow

In the Specular workflow, a **gloss map** is used to define the smoothness of a surface as opposed to a **roughness map**. **Specular maps** are special because in some cases they should be colored. Most non-metals reflect 2% to 5% of the light as specular and the highlight is monochrome/gray. As the variation is so little, it is often enough to use a constant specular value instead of a specular texture map. However, if metal and non-metal are mixed in a single texture, it is required to use a specular map, as metal has a much brighter specular color than non-metal. 

![shader_values.jpg](/img/shader_values.jpg)

![SPEC_Range_new_copy.png](/img/SPEC_Range_new_copy.png)

[Creating Textures for Physically Based Shading - CRYENGINE 3 Manual - Documentation](https://docs.cryengine.com/display/SDKDOC2/Creating+Textures+for+Physically+Based+Shading)

More resources for PBS

2014: [https://blog.selfshadow.com/publications/s2014-shading-course/](https://blog.selfshadow.com/publications/s2014-shading-course/) 

2015: [https://blog.selfshadow.com/publications/s2015-shading-course/](https://blog.selfshadow.com/publications/s2015-shading-course/)

2016: [https://blog.selfshadow.com/publications/s2016-shading-course/](https://blog.selfshadow.com/publications/s2016-shading-course/) 

2017: [https://blog.selfshadow.com/publications/s2017-shading-course/](https://blog.selfshadow.com/publications/s2017-shading-course/) 

# Optimizing materials

The amount of instructions and samplers in a material are generally the two most important statistics. Texture resolution 

## Channel packing

Channel packing is a method where we store multiple unrelated 2D maps into a single 3D map, like Metallic, Roughness, Occlusion (MRO/ORM) or Base Color plus Alpha or Base Color plus Roughness (BcA/BcR). This is useful to reduce the amount of texture samplers in a material, increasing performance. 

Be aware that Texture Samplers NOT sharing the same UV set have their cost increased, especially for Virtual Texture Samplers. 

### Channel packing overview

There’s 4 channels to a texture. The Green channel has 1 more bit-size so if you’re packing gradients, that’s the channel to do it in. The alpha channel has its own compression setting so if you’re looking for crisp results across mips, put it there (high detail/frequency). 

### Channel packing with normals

Generally speaking it’s bad practice to channel pack things together with a normal map. 

The Coalition channel packs the Normal X and Y together with a Roughness and Metallic into a BC7. I didn’t know this was possible and bears looking in to. See here: 

[The Making of “Alpha Point”—UE5 Technical Demo | GDC 2021](https://youtu.be/X2FBFFBDJf0?t=1078)

They didn’t do this for Gears 4, but for Gears 5 they started doing this to keep the size down, which was apparently necessary. Deathrey mentions one problem with this: 

![deathrey_1.png](/img/deathrey_1.png)

![deathrey_2.png](/img/deathrey_2.png)

## Reducing shader instruction count

Unreal offers an out of the box Bake Materials function that basically does what Substance Painter does. This is great for one-offs, but at a larger scale this involves too much manual labor. It reduces texture instructions and texture samplers.  

![unreal_bake_materials.png](/img/unreal_bake_materials.png)

You should also try and prevent Static Switches as much as possible, especially turn on or off further up a chain of instances. 

> A Static Switch will only create 2 versions of the material internally. This stacks up to maximum possible combinations between them (i.e. 3 switches already creates 9 materials, 4 switches creates 16, etc), which in the end defeats all the advantages of the Material Instance system.
> 

> A new version of the Material must be compiled for every used combination of static parameters in a Material, which can lead to a massive increase in shader permutations if abused. Try to minimize the number of static parameters in the Material and the number of permutations of those static parameters that are actually used.
> 

It’s a bit unintuitive so here’s an illustration:

![shader_permutations.png](/img/shader_permutations.png)

You’d think the instance down the line with only one switch different shares the same master as its neighbor, but instead it creates a new master material. 

# Advanced material techniques

## UV/UDIM

Using the UV space of an object beyond 0,0 to 1,1 space we can employ some really cool tricks. In the Dead Space Remake they’ve used that space to color an asset. 

![uv_index.png](/img/uv_index.png)

![uv_index_tool.png](/img/uv_index_tool.png)

[DEAD SPACE REMAKE Official Art Deep-Dive (2023) 4K](https://youtu.be/4jlRuA4bJFI?t=259)

For The Ascent they used a similar idea. 

[Building the World of 'The Ascent'](https://youtu.be/FodXp5BkENk?t=784)

## Custom Primitive Data (UE only)

[Storing Custom Data in a Material Per Primitive](https://docs.unrealengine.com/4.27/en-US/RenderingAndGraphics/Materials/CustomPrimitiveData/)

[Optimize with Custom Data | 5-Minute Materials [UE4/UE5]](https://youtu.be/QQ-VzeGjLMo)

## Triplanar mapping

### World aligned

[Triplanar, Dithered Triplanar, and Biplanar Mapping in Unreal](https://ryandowlingsoka.com/unreal/triplanar-dither-biplanar/)

[https://iquilezles.org/articles/biplanar/](https://iquilezles.org/articles/biplanar/) 

[Your Triplanar is wrong. Here's how to make one that works. [UE5]](https://www.youtube.com/watch?v=Cq5H59G-DHI)

### Object aligned

![object_aligned_world_aligned_texture.png](/img/object_aligned_world_aligned_texture.png)

# Naming conventions

## Epic’s recommendations

```
[AssetTypePrefix]_[AssetName]_[Descriptor]_[OptionalVariantLetterOrNumber]
```

[Recommended Asset Naming Conventions](https://docs.unrealengine.com/5.2/en-US/recommended-asset-naming-conventions-in-unreal-engine-projects/)

## Ready at Dawn

For The Order: 1886 they use a hierarchical layering system which has some really nice naming conventions that are way better than most I’ve seen. They derive a Common material (cmn) from one or multiple Templates (mtt), blending between them using various alpha masks. 

```
[material hierarchy indication]_[material type]_[primary tuning variable]_[intensity of variable]
```

- mtt_brass_polished_025
- cmn_brass_a_tile_spotted_dark
- cmn_steel_a_tile_dark
- wpn_thermitegun_source

![rad_mat_naming.png](/img/rad_mat_naming.png)

![rad_mat_naming_2.png](/img/rad_mat_naming_2.png)

![rad_mat_naming_3.png](/img/rad_mat_naming_3.png)

# Resources, see also

Sebastien Lagarde has some fantastic resources. His PDF about the art direction of Remember Me is great and shows a lot of the systemic thinking of how the studio prepared artists for the PBR workflow

[https://seblagarde.files.wordpress.com/2013/08/gdce13_lagarde_harduin_light.pdf](https://seblagarde.files.wordpress.com/2013/08/gdce13_lagarde_harduin_light.pdf) 

PBR value guides which are referenced all over the place

[https://seblagarde.files.wordpress.com/2014/04/dontnodgraphicchartforunrealengine4.png](https://seblagarde.files.wordpress.com/2014/04/dontnodgraphicchartforunrealengine4.png) 

[https://physicallybased.info/](https://physicallybased.info/) 

Henry Labounta’s notes on Art Direction Tools for Photo Real Games
[https://web.archive.org/web/20110424200123/https://stachmo.wordpress.com/2011/03/12/the-epic-gdc11-adventure-day-5/](https://web.archive.org/web/20110424200123/https://stachmo.wordpress.com/2011/03/12/the-epic-gdc11-adventure-day-5/)

Ready at Dawn has some great notes on how they managed materials for The Order: 1886 which looks fucking amazing

[https://blog.selfshadow.com/publications/s2013-shading-course/rad/s2013_pbs_rad_slides.pdf](https://blog.selfshadow.com/publications/s2013-shading-course/rad/s2013_pbs_rad_slides.pdf)

Vincent Derozier has a lot of tutorials and resources related to materials

[https://www.artstation.com/vincentderozier](https://www.artstation.com/vincentderozier)

Playground Games did some insane lookdev on car paint shaders which is quite interesting

[Physically-Based Calibration: Accurate Material Production in 'Forza Horizon 4'](https://www.gdcvault.com/browse/gdc-19/play/1025994)

- Notes
    
    - Macbeth Munsell Color Checker for shooting your own reference.
    
    - best time to get proper reference for a surface texture or some large piece of detail is on an overcast day.
    
    - use reference daily, in comparison to what you’re making, in every review, *always.* 
    
    - Don’t just tack reference to a wall and say it’s there for when you need it.  Look at it constantly.
    
    - black contact shadows to ground objects to surfaces.
    
    - when reviewing a scene, check it in all viewport viewing modes.  The 
    scene should read properly always, whether it is a grey surfaced view, 
    flat view, texture only view, or lit view.
    

### Material & texture artists

Free nodes and other setups: 

[Maxime Guyard-Morin](https://www.artstation.com/nova__12)

Breakdowns and deep dives:

[Malte Resenberger-Loosmann](https://www.artstation.com/iammalte)

Varied and long experience:

[Rogelio Olguin](https://www.artstation.com/rogelio)

High quality stuff:

[Stan Brown](https://www.artstation.com/stanbrown)

Naughty Dog artist:

[Jared Sobotta](https://www.artstation.com/jaredsobotta)

Amazing quality realistic materials:

[Joe Taylor](https://www.artstation.com/joetaylorart)

Nicely stylized realism materials:

[Civilization 7 Wall Materials, Clark Coots](https://www.artstation.com/artwork/8BPKKQ)
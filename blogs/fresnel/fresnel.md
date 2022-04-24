# Fresnel

- an conclusion of some resources of fresnel in rendering

---

# Basics

The interaction of light with a planar interface between two substance follows the Fresnal Equations. The surface is assumed to not have any irregulariies between 1 light wavelength and 100 wavelengths in size.

ight incident on a flat surface split into reflected part and refrated part, we define the angle between surface normal and reflected light direction is <img src="https://latex.codecogs.com/svg.latex?\theta_i" />, the angle between surface normal and refracted light direction is <img src="https://latex.codecogs.com/svg.latex?\theta_t" />, the Index of Refraction(IOR) of two substance is <img src="https://latex.codecogs.com/svg.latex?n_{i}" />, <img src="https://latex.codecogs.com/svg.latex?n_{t}" />
(incident, transmitted).

> the formula of IOR is  <img src="https://latex.codecogs.com/svg.latex?n = \eta + ik" />, for dieletric, kappa is zero, we use 

<img src="fresnel1.png" width="500"/>

(light is an electromagnetic wave, and the electric field of this wave oscillates perpendicularly to the direction of propagation)

S polarization is pependicular to the plance of incidence
P polarization is parallel to the plance of incidence

<img src="fresnel2.png" width="500"/>

We don't derive the Fresnal Equation but just give it out. you can find the derivation [here](https://www.brown.edu/research/labs/mittleman/sites/brown.edu.research.labs.mittleman/files/uploads/lecture13_0.pdf)

the reflection and refraction coefficient of S polarization and P polarization

<img src="https://latex.codecogs.com/svg.latex?R_s = (\frac{n_i\cos\theta_i-n_t\cos\theta_t}{n_i\cos\theta_i+n_t\cos\theta_t})^2" />  <img src="https://latex.codecogs.com/svg.latex?T_s = (\frac{2n_i\cos\theta_i}{n_i\cos\theta_i+n_t\cos\theta_t})^2" />

<img src="https://latex.codecogs.com/svg.latex?R_p = (\frac{n_i\cos\theta_t-n_t\cos\theta_i}{n_i\cos\theta_t+n_t\cos\theta_i})^2" /> <img src="https://latex.codecogs.com/svg.latex?T_p = (\frac{2n_i\cos\theta_i}{n_i\cos\theta_t+n_t\cos\theta_i})^2" />

for both polarizations, <img src="https://latex.codecogs.com/svg.latex?n_{i} \sin\theta_i = n_{t} \sin\theta_t" /> , so we can replace <img src="https://latex.codecogs.com/svg.latex?\cos\theta_t" /> with <img src="https://latex.codecogs.com/svg.latex?\sqrt{1-\frac{n_i}{n_t}(\sin\theta_i)^2}" /> and get the equation only about <img src="https://latex.codecogs.com/svg.latex?\theta_i" />

in real-time rendering, we consider no polarization of S and P, so the reflection is <img src="https://latex.codecogs.com/svg.latex?\frac{R_s+R_p}{2}" />, and in this blog we don't care about refraction, <img src="https://latex.codecogs.com/svg.latex?T_s" />, <img src="https://latex.codecogs.com/svg.latex?T_p" /> are ignored.


<img src="fresnel3.jpg" width="500"/>

the following figure demonstrate the reflection ratio of light shoot to different material from air.(glass copper aluminum)

<img src="fresnel4.jpg" width="500"/>


#### total internal reflection

if the Index of Refraction of substance <img src="https://latex.codecogs.com/svg.latex?n_i<n_t" />, according to snell's law, <img src="https://latex.codecogs.com/svg.latex?\theta_t>\theta_i" />, if <img src="https://latex.codecogs.com/svg.latex?\sin\theta_i = \sin\theta_c=\frac{n_t}{n_i}" />, no transmission occurs, that is total internal reflection.  

## Make It Real-Time

####  Shilick Approximation

calculating the Fresnal Equation in real-time is very expensive, Schilick gives an easier way to approximate the fresnel reflection.

the fresnel reflection can be see as an interpolation between <img src="https://latex.codecogs.com/svg.latex?F_0" /> and <img src="https://latex.codecogs.com/svg.latex?1" />, <img src="https://latex.codecogs.com/svg.latex?F_0" /> is the fresnel reflection when <img src="https://latex.codecogs.com/svg.latex?\theta_i = 0" />. 

<img src="https://latex.codecogs.com/svg.latex?F_0 =\frac{R_s+R_p}{2}=(\frac{n_i-n_t}{n_i+n_t})^2" />

the F0 value of some dialetric materials

<img src="f0.jpg" width="500"/>


<img src="https://latex.codecogs.com/svg.latex?F_0" /> requires Index of Refraction which is not easy to understand, in rtr, we just use an RGB color instead.

here we got the entire schilick approximation:

<img src="https://latex.codecogs.com/svg.latex?F_r = F_0 + (1-F_0)(1-\cos\theta_i)^5" />

here is an figure of result of schilick approx compared with fresnel IOR(glass, copper, aluminum, chromium, iron, zinc). The reflection of dialetric material like glass can be approxed correctly, but for metal, it's not good enough.

<img src="http://www.realtimerendering.com/figures/thumb/RTR4.09.22.jpg" width="500"/>

the reflection of metal varies with the wavelenth of light, lets recall the IOR, the IOR of metal(especially colored metal) varies from with light wavelength.

<img src="ior.png" width="500"/>

that means for RGB light, the reflection is different, we just assign RGB with different  <img src="https://latex.codecogs.com/svg.latex?F_0" /> to solve this.

from the figure, at grazing angle, the reflection have a small decrease for some metals(aluminum, chromium, iron, zinc). An easy way is to modify the approx to 

<img src="https://latex.codecogs.com/svg.latex?F_r = F_0+(F_{90}-F_0)(1-\cos\theta_i)^{\frac{1}{p}}" /> 

## What's more

[Gulbrandsen14](https://jcgt.org/published/0003/04/03/) gaves an artist friendly approxmiztion for metallic fresnel. He made an remap of IOR to reflectivity <img src="https://latex.codecogs.com/svg.latex?r" /> and edgetint <img src="https://latex.codecogs.com/svg.latex?g" /> derived from Fresnal Equation. But it is RGB based rendering, didn't account the spectual distribution.

![](https://jcgt.org/published/0003/04/03/icon.png)

## TBD


[Hoffman19](http://renderwonk.com/publications/mam2019/)

[Hoffman20](https://blog.selfshadow.com/publications/s2020-shading-course/hoffman/s2020_pbs_hoffman_slides.pdf)

[Belcour20](https://blog.selfshadow.com/publications/s2020-shading-course/belcour/slides/index.html)

## Other thoughts

**why almost all render engine use a fresnel term to mix specular and diffuse color?**

>the fresnel equation describes the light amount of reflection and refraction. The reflected light contributes to the specular brdf, and the refracted light under the surface will be absorbed and scatterd outside the surface. the scattered light is the diffuse part we calculated in shader.

# Ref

https://jcgt.org/published/0003/04/03/
https://zhuanlan.zhihu.com/p/158025828
https://learnopengl.com/PBR/Theory
https://hal.inria.fr/hal-02883680v2/document
https://blog.selfshadow.com/publications/s2020-shading-course/hoffman/s2020_pbs_hoffman_slides.pdf
https://www.brown.edu/research/labs/mittleman/sites/brown.edeu.research.labs.mittleman/files/uploads/lecture13_0.pdf
https://www.electrical4u.com/fresnel-equations/
https://zhuanlan.zhihu.com/p/31534769
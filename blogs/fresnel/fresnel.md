# Fresnel

- an conclusion of some resources of fresnel in rendering

---

# Basics

The interaction of light with a planar interface between two substance follows the Fresnal Equations. The surface is assumed to not have any irregulariies between 1 light wavelength and 100 wavelengths in size.

ight incident on a flat surface split into reflected part and refrated part, we define the angle between surface normal and reflected light direction is $\theta_i$, the angle between surface normal and refracted light direction is $\theta_t$, the Index of Refraction(IOR) of two substance is $n_{i}$, $n_{t}$
(incident, transmitted).

> the formula of IOR is  $n = \eta + ik$, for dieletric, kappa is zero, we use 

![](https://tse1-mm.cn.bing.net/th/id/R-C.9883e023774b495ecde1d8928472f6b7?rik=HxVxdNRUbkCddA&riu=http%3a%2f%2fem.geosci.xyz%2f_images%2fsnellslaw_setup.png&ehk=wAAT5nbPwQUaROoEqYC6xZLtkAyMphC7MMldZfRR2x8%3d&risl=&pid=ImgRaw&r=0)

(light is an electromagnetic wave, and the electric field of this wave oscillates perpendicularly to the direction of propagation)

S polarization is pependicular to the plance of incidence
P polarization is parallel to the plance of incidence

![](https://www.electrical4u.com/wp-content/uploads/What-are-the-Fresnel-Equations.png)

We don't derive the Fresnal Equation but just give it out. you can find the derivation [here](https://www.brown.edu/research/labs/mittleman/sites/brown.edu.research.labs.mittleman/files/uploads/lecture13_0.pdf)

the reflection and refraction coefficient of S polarization and P polarization

$R_s = (\frac{n_i\cos\theta_i-n_t\cos\theta_t}{n_i\cos\theta_i+n_t\cos\theta_t})^2$ $T_s = (\frac{2n_i\cos\theta_i}{n_i\cos\theta_i+n_t\cos\theta_t})^2$ 

$R_p = (\frac{n_i\cos\theta_t-n_t\cos\theta_i}{n_i\cos\theta_t+n_t\cos\theta_i})^2$ $T_p = (\frac{2n_i\cos\theta_i}{n_i\cos\theta_t+n_t\cos\theta_i})^2$

for both polarizations, $n_{i} \sin\theta_i = n_{t} \sin\theta_t$, so we can replace $\cos\theta_t$ with $\sqrt{1-\frac{n_i}{n_t}(\sin\theta_i)^2}$ and get the equation only about $\theta_i$

in real-time rendering, we consider no polarization of S and P, so the reflection is $\frac{R_s+R_p}{2}$, and in this blog we don't care about refraction, $T_s$, $T_p$ are ignored.

![](https://pic1.zhimg.com/80/v2-e371ae15bd4a56044adb40d22699e2d8_720w.jpg)

the following figure demonstrate the reflection ratio of light shoot to different material from air.(glass copper aluminum)

![](http://www.realtimerendering.com/figures/thumb/RTR4.09.20.jpg)


#### total internal reflection

if the Index of Refraction of substance $n_i<n_t$, according to snell's law, $\theta_t>\theta_i$, if $\sin\theta_i = \sin\theta_c=\frac{n_t}{n_i}$($\sin\theta_t=1$), no transmission occurs, that is total internal reflection.  

## Make It Real-Time

####  Shilick Approximation

calculating the Fresnal Equation in real-time is very expensive, Schilick gives an easier way to approximate the fresnel reflection.

the fresnel reflection can be see as an interpolation between $F_0$ and $1$, $F_0$ is the fresnel reflection when $\theta_i = 0$. 

$F_0 =\frac{R_s+R_p}{2}=(\frac{n_i-n_t}{n_i+n_t})^2$

![](https://pic2.zhimg.com/80/v2-7717785a1ae3287626fba3a9dde91405_720w.jpg)

$F_0$ requires Index of Refraction which is not easy to understand, in rtr, we just use an RGB color instead.

here we got the entire schilick approximation:

$F_r = F_0 + (1-F_0)(1-\cos\theta_i)^5$

here is an figure of result of schilick approx compared with GT(glass, copper, aluminum, chromium, iron, zinc). The reflection of dialetric material like glass can be approxed correctly, but for metal, it's not good enough.

![](http://www.realtimerendering.com/figures/thumb/RTR4.09.22.jpg)

the reflection of metal varies with the wavelenth of light, lets recall the IOR, the IOR of metal(especially colored metal) varies from with light wavelength.

![](ior.png)

that means for RGB light, the reflection is different, we just assign RGB with different value of $F_0$ to solve this.

from the figure, at grazing angle, the reflection have a small decrease for some metals(aluminum, chromium, iron, zinc). An easy way is to modify the approx to 

$F_r = F_0+(F_{90}-F_0)(1-\cos\theta_i)^{\frac{1}{p}}$ 

## What's more

[Gulbrandsen14](https://jcgt.org/published/0003/04/03/) gaves an artist friendly approxmiztion for metallic fresnel. He made an remap of IOR to reflectivity $r$ and edgetint $g$ derived from Fresnal Equation. But it is RGB based rendering, didn't account the spectual IOR data.

![](https://jcgt.org/published/0003/04/03/icon.png)

## TBD


[Hoffman19](http://renderwonk.com/publications/mam2019/)

[Hoffman20](https://blog.selfshadow.com/publications/s2020-shading-course/hoffman/s2020_pbs_hoffman_slides.pdf)

[Belcour20](https://blog.selfshadow.com/publications/s2020-shading-course/belcour/slides/index.html)

## Other thoughts

**why almost all render engine use a fresnel term to mix specular and diffuse color?**

>the fresnel equation describes the light amount of reflection and refraction. The reflected light contributes to the specular brdf, and the refracted light under the surface will be absorbed and scatterd outside the surface. the scattered light is the diffuse part we calculated in shader.

# Ref

https://zhuanlan.zhihu.com/p/158025828
https://learnopengl.com/PBR/Theory
https://hal.inria.fr/hal-02883680v2/document
https://blog.selfshadow.com/publications/s2020-shading-course/hoffman/s2020_pbs_hoffman_slides.pdf
https://www.brown.edu/research/labs/mittleman/sites/brown.edeu.research.labs.mittleman/files/uploads/lecture13_0.pdf
https://www.electrical4u.com/fresnel-equations/#:~:text=What%20are%20the%20Fresnel%20Equations%3F%20The%20Fresnel%20Equations,as%20well%20as%20phase%20shifts%20between%20the%20waves.
https://zhuanlan.zhihu.com/p/31534769#:~:text=%E8%8F%B2%E6%B6%85%E8%80%B3%E6%96%B9%E7%A8%8B%20%E8%8F%B2%E6%B6%85%E8%80%B3%E6%96%B9%E7%A8%8B%EF%BC%88Fresnel,equation%EF%BC%89%E6%8F%8F%E8%BF%B0%E4%BA%86%E5%85%89%E7%BA%BF%E7%BB%8F%E8%BF%87%E4%B8%A4%E4%B8%AA%E4%BB%8B%E8%B4%A8%E7%9A%84%E7%95%8C%E9%9D%A2%E6%97%B6%EF%BC%8C%E5%8F%8D%E5%B0%84%E5%92%8C%E9%80%8F%E5%B0%84%E7%9A%84%E5%85%89%E5%BC%BA%E6%AF%94%E9%87%8D%E3%80%82%20%E5%8F%82%E8%80%83%E4%B8%8B%E5%9B%BE%E7%9A%84%E7%AC%A6%E5%8F%B7%EF%BC%9A
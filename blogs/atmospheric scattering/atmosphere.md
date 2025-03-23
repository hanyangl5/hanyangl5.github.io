
## Real Time Atmosphere Rendering

unlike hard surfaces we only consider reflection in real-time rendering, some paticipants can reflect, refract, and absorb light in the same time. like cloud, smoke, sky. we use techs called volume rendering for those medias.

![08f4a18b438b1fc5c2533e6fb5a8c11f.png](:/7b490cbbe18b479b954306a31029ed3e)

VRE

https://graphics.pixar.com/library/ProductionVolumeRendering/paper.pdf

\# Precompute Atomspheric Scattering EB08

https://ebruneton.github.io/precomputed_atmospheric_scattering/

# Precompute Atomspheric Scattering Explaination

$\sigma_s$ and $\sigma_t$ are only related to the density of the volume. In atmospheric scattering, we assume the planet is perfect sphere, and the density of the particles follows $\rho(h) = \exp({-\frac{h}{H}})$, then $\sigma_t$ is a function of h. and can be represented by $\sigma_t(0)\cdot \rho(h)$

* * *

The volumetric scattering equation calculates the radiance L(p,o) seen at point p in direction o:

```
              in-lights  ...       ...
               /         /         /      
eye(p) <----- x1 <----- x2 <----- x3 <----- opaque surface(p_s) 
               \         \         \
               out-scat  ...       ... 
```

$L(p, o) = T(p_{s}, p)\cdot L(p_{s}, o) + S(p_s, x)$

$T(a, b) = e^{-\int_{x=a}^b\sigma_t{(x)}dt}$

$S(p,p_s , o) = \int_{x=p_{s}}^{p}T(p_{s}, x)\cdot L_s(x, o)\sigma_sdt$

$L_s(x, o) = \sum_{i=0}^{lights}L_{in}(x, i)\cdot Phase(i, o)\cdot Vis(x, i)$

- $T(p_s,p)$ is the transmittance between $p$ and opaque surface $p_s$, accounting for attenuation.
- The first term is light reflected from the surface, attenuated, also called ground reflection.
- The second term is in-scattering from all points x between the $p_s$ and $p$.
- $L_s(x,o)$ is the source radiance at x coming from all light sources (sun, sky) in direction o.



volume rendering equation

![alt text](image-1.png)

$$L(x,\omega)=\int_{t=0}^dT_{t}\left[\sigma_a(x_t)L_e(x_t,\omega)+\sigma_s(x_t)L_s(x_t,\omega)+L_d(x_d,\omega)\right]dt$$

omit emission, 

$$L(x,\omega)=\int_{t=0}^dT_{t}\sigma_s(x_t)L_s(x_t,\omega)dt + T_{d}L_d(x_d,\omega)dt$$


$$
L_{s} ( {x_t}, \omega)=\int_{S^{2}} f_{p} ( {x_t}, \omega, \omega^{\prime} ) L ( {x_t}, \omega^{\prime} ) d \omega^{\prime}. 
$$

Single Scattering


<!-- for single light source, ex, sunlight $L(s)$

$$
L_s(x_t, \omega) = f_{p} ( {x_t}, \omega, \omega^{\prime} ) T(x, s) L ( s ). 
$$ -->

## Scattering Coefficient

$$
\sigma_{s_{Ray}}(h)=\frac{8 \pi^{2} ( n^{2}-1 )^{2}} {3 N \lambda^{4}} e^{-\frac{h}{H_{Ray}}}
$$

$$
\sigma_{s_{Mie}}(h)=\sigma_{s_{Mie}}(0)
e^{-\frac{h}{H_{Mie}}}
$$

with

$$\sigma_{s_{Ray}} = \sigma_{t_{Ray}}$$


$$\sigma_{t_{Mie}}(h)=1.11 \sigma_{s_{Mie}}(h)$$

## Precompute Transmittance

for atmosphere rendering, Transmittance between any point a, b $T(a, b)$ can be computed using 
$$\frac{T(a, i)}{T(b, i)} = \frac{e^{-\int_{x=a}^i\sigma_t{(x)}dt}}{e^{-\int_{x=b}^i\sigma_t{(x)}dt}}$$


$i$ is the intersection point of $L$ and atmosphere upper bound.

we denote direction of a and b $v = \frac{a-b}{||a-b||}$

Transmittance of a point $x$ on direction $v$ can be represent by $\hat{T}(r, \mu)$, $r$ is altitude of $x$, $r=||x||$, $\mu$ is cosine angle between $v$ and zenith, $\mu=\frac{v.x}{r}$.

implementation detail of how to map $[0,1]^2$ to $r, \mu$

```c++
// get r, mu from transmitance lut uv
float2 RMuFromTransmittanceLutUv(AtmosphereParam atm, float2 uv) {
    float x_mu = uv.y;
    float x_r = uv.x;
    
    float H = sqrt(atm.top_radius2 - atm.bottom_radius2);
    float rho = H * x_r;
    // r^2 = rho^2 + bottom_radius^2  
    float r = sqrt(rho * rho + atm.bottom_radius2);
    
    // d is range (from atm.top_radius - r, rho + H)
    float d = lerp(atm.top_radius - r, rho + H, x_mu);
    
    // cosine law
    // top_radius^2 = d^2 + r^2 - 2rd * -mu 
    // note: cos(pi - theta) = -cos(theta)
    // mu = top_radius^2 - d^2 - r^2 / 2rd  
    if (d == 0.0f)
    {
        return 1.0f;
    }
    float mu = (H * H - rho * rho - d * d) / (2.0 * r * d);
    mu = clamp(mu, -1.0f, 1.0f);
    return float2(r, mu);
}

float2 TransmittanceLutUvFromRMu(AtmosphereParam atm, float R, float Mu) {
	float H = sqrt(max(0.0f, atm.top_radius2 - atm.bottom_radius2));
	float Rho = sqrt(max(0.0f, R * R - atm.bottom_radius2));

	float Discriminant = R * R * (Mu * Mu - 1.0f) + atm.top_radius2;
	float D = max(0.0f, (-R * Mu + sqrt(Discriminant))); // Distance to atmosphere boundary

	float Dmin = atm.top_radius - R;
	float Dmax = Rho + H;
	float Xmu = (D - Dmin) / (Dmax - Dmin);
	float Xr = Rho / H;

	UV = float2(Xmu, Xr);
}
```


## Precompute Direct Irradiance

For single scattering, $L(p_{s}, o)$ is only affected by sun light,

$L(p_{s}) = L_{s}$

## Precompute In-Scattering?

$S(p,i)$ represent the in scattering radiance reaching p from q.

we denote $o = \overline{pq}$

If we only account one light source $in$, $S(p, i) = \int_{x=i}^{p}T(q, x)\cdot L_{sun}(x, in)\cdot Phase(in, o)\cdot Vis(x, in)\sigma_s(x)dt$

$L_{sun}(p, in) = L_{sun}\cdot T(p, in)$

$S$ is a function of $p, i, in$

we can parameterized these into $r, \mu, \mu_s, \nu$  
$r$ is altitude of $p$, $\mu$ is cosine angle between $r$ and $o$, $\mu_s$ is the angle between $r$ with $in$, $\nu$ is angle between $o$ and $in$.

when rendering the scene, we need to account for occlusion, we use $\hat{S}$ to represent the in-scattering without occlusion, $p_o$ represent to occluder point, the formula can be represented by.

$S(p, o, s) = \hat{S}(p, o, s) - T(p, p_p)\cdot\hat{S}(p_o, o, s)$

## Multi-Scattering

$L_{s_{ms}}$ is affected by the single scattering $L_s$ we computed in a 4D lut.

we denote L we computed before as $L_0 = R_0 + S_0$  
$L = L_0 + S_0 + S_1 + S_2 + ... +Sn$

$S_1(p,p_s , o) = \int_{x=p_{s}}^{p}T(p_{s}, x)\sum_{i=0}^{S_0}S_0{in}(x, i)\cdot Phase(i, o)\cdot Vis(x, i)\sigma_sdt$

$S_2 = ...$

Accounting for occlusion for multiscattering is difficult.

Delte Irradiance

$\Epsilon=\int_{2\pi}\overline{L}$
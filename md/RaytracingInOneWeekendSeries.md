
2020.1.16-2020.1.31的关于Raytracing In One Weekend系列的notes

# 输出图片

原文中使用的图片是ppm格式， 是一种未压缩的格式，一些设备不支持直接查看ppm格式图片，因此我使用了一个开源库svpng，可以将rgb颜色信息直接保存到png文件中

项目地址: [svpng](https://github.com/miloyip/svpng)

将颜色信息写入png文件的的C++代码如下
~~~
#include "svpng.inc"

int main() {

    int nx=200,ny=150;// width & height
    unsigned char rgb[nx * ny * 3], *p = rgb;
    FILE *fp = fopen("test.png", "wb");
    for (int j = ny-1; j >= 0; j--)
        for (int i = 0; i < nx; i++) {
            float r = float(i) / float(nx);
            float g = float(j) / float(ny);
            float b = 0.2;
            *p++ = int(255.99*r);    /* R */
            *p++ = int(255.99*g);    /* G */
            *p++ = int(255.99*b);    /* B */
        }
    svpng(fp, nx, ny, rgb, 0);
    fclose(fp);
    return 0;
}
~~~

渲染出的图片如下

![](https://pic.downk.cc/item/5e21457f2fb38b8c3c30944c.png)

图片中像素从左到右，从上到下输出

可以看出，从左到右，红色通道的值越来越大，图像越来越红，从上到下，绿色通道的值越来越大，图像越来越绿

# 向量类 vec3.h

我们构造的向量类如下

~~~
#include <iostream>
#include <cmath>
#include <cstdlib>

class vec3 {
public:
    vec3() {}
    vec3(float e0, float e1, float e2) { e[0] = e0; e[1] = e1; e[2] = e2; }
    inline float x() const { return e[0]; }
    inline float y() const { return e[1]; }
    inline float z() const { return e[2]; }
    inline float r() const { return e[0]; }
    inline float g() const { return e[1]; }
    inline float b() const { return e[2]; }

    inline const vec3& operator+() const { return *this; }
    inline vec3 operator-() const { return vec3(-e[0], -e[1], -e[2]); }
    inline float operator[](int i) const { return e[i]; }
    inline float& operator[](int i) { return e[i]; }

    inline vec3& operator+=(const vec3 &v2);
    inline vec3& operator-=(const vec3 &v2);
    inline vec3& operator*=(const vec3 &v2);
    inline vec3& operator/=(const vec3 &v2);
    inline vec3& operator*=(const float t);
    inline vec3& operator/=(const float t);

    inline float length() const { return sqrt(e[0]*e[0] + e[1]*e[1] + e[2]*e[2]); }
    inline float squared_length() const { return e[0]*e[0] + e[1]*e[1] + e[2]*e[2]; }
    inline void make_unit_vector();

    float e[3];
};
~~~

类成员函数的实例化
~~~
inline std::istream& operator>>(std::istream &is, vec3 &t) {
    is >> t.e[0] >> t.e[1] >> t.e[2];
    return is;
}

inline std::ostream& operator<<(std::ostream &os, const vec3 &t) {
    os << t.e[0] << " " << t.e[1] << " " << t.e[2];
    return os;
}

inline void vec3::make_unit_vector() {
    float k = 1.0 / sqrt(e[0]*e[0] + e[1]*e[1] + e[2]*e[2]);
    e[0] *= k; e[1] *= k; e[2] *= k;
}

inline vec3 operator+(const vec3 &v1, const vec3 &v2) {
    return vec3(v1.e[0] + v2.e[0], v1.e[1] + v2.e[1], v1.e[2] + v2.e[2]);
}

inline vec3 operator-(const vec3 &v1, const vec3 &v2) {
    return vec3(v1.e[0] - v2.e[0], v1.e[1] - v2.e[1], v1.e[2] - v2.e[2]);
}

inline vec3 operator*(const vec3 &v1, const vec3 &v2) {
    return vec3(v1.e[0] * v2.e[0], v1.e[1] * v2.e[1], v1.e[2] * v2.e[2]);
}

inline vec3 operator*(float t, const vec3 &v) {
    return vec3(t*v.e[0], t*v.e[1], t*v.e[2]);
}

inline vec3 operator*(const vec3 &v, float t) {
    return vec3(t*v.e[0], t*v.e[1], t*v.e[2]);
}

inline vec3 operator/(const vec3 &v1, const vec3 &v2) {
    return vec3(v1.e[0] / v2.e[0], v1.e[1] / v2.e[1], v1.e[2] / v2.e[2]);
}

inline vec3 operator/(vec3 v, float t) {
    return vec3(v.e[0]/t, v.e[1]/t, v.e[2]/t);
}

inline float dot(const vec3 &v1, const vec3 &v2) {
    return v1.e[0]*v2.e[0]
         + v1.e[1]*v2.e[1]
         + v1.e[2]*v2.e[2];
}

inline vec3 cross(const vec3 &v1, const vec3 &v2) {
    return vec3(v1.e[1] * v2.e[2] - v1.e[2] * v2.e[1],
                v1.e[2] * v2.e[0] - v1.e[0] * v2.e[2],
                v1.e[0] * v2.e[1] - v1.e[1] * v2.e[0]);
}

inline vec3& vec3::operator+=(const vec3 &v) {
    e[0] += v.e[0];
    e[1] += v.e[1];
    e[2] += v.e[2];
    return *this;
}

inline vec3& vec3::operator-=(const vec3& v) {
    e[0] -= v.e[0];
    e[1] -= v.e[1];
    e[2] -= v.e[2];
    return *this;
}

inline vec3& vec3::operator*=(const vec3 &v) {
    e[0] *= v.e[0];
    e[1] *= v.e[1];
    e[2] *= v.e[2];
    return *this;
}

inline vec3& vec3::operator*=(const float t) {
    e[0] *= t;
    e[1] *= t;
    e[2] *= t;
    return *this;
}

inline vec3& vec3::operator/=(const vec3 &v) {
    e[0] /= v.e[0];
    e[1] /= v.e[1];
    e[2] /= v.e[2];
    return *this;
}

inline vec3& vec3::operator/=(const float t) {
    float k = 1.0/t;

    e[0] *= k;
    e[1] *= k;
    e[2] *= k;
    return *this;
}

inline vec3 unit_vector(vec3 v) {
    return v / v.length();
}
~~~

vec3类中的成员变量e[3]可以表示三维空间中的xyz坐标和rgb三个通道

类中成员函数有向量长度length，长度平方squared_length，单位化make_unit_vector

我们更改main函数中的代码来测试一下

~~~
#include "svpng.inc"
#include "vec3.h"
int main() {

    int nx=200,ny=150; // width & height
    unsigned char rgb[nx * ny * 3], *p = rgb;
    FILE *fp = fopen("rgb.png", "wb");
    for (int j = ny-1; j >= 0; j--)
        for (int i = 0; i < nx; i++) {
            vec3 col(float(i) / float(nx), float(j) / float(ny), 0.2);
            *p++ = int(255.99*col[0]);    /* R */
            *p++ = int(255.99*col[1]);    /* G */
            *p++ = int(255.99*col[2]);    /* B */
        }

    svpng(fp, nx, ny, rgb, 0);
    fclose(fp); 
    return 0;
}
~~~

会得到与上一章相同的图像

# 光线 相机 背景


### 光线类ray.h

我们习惯用直线来表示光线

直线方程是p(t)=A+t∗B，其中，A，B都是vec3类型，A是光线的起点，B是光线的方向，t为一实数（代码中用浮点数表示），p(t)对应t不同取值时直线上的点

![](https://raytracing.github.io/images/fig-1-04-1.jpg)

ray类的代码如下

~~~
#ifndef RAYH
#define RAYH
#include "vec3.h"

class ray
{
    public:
        ray() {}
        ray(const vec3& a, const vec3& b) { A = a; B = b; }
        vec3 origin() const       { return A; }
        vec3 direction() const    { return B; }
        vec3 point_at_parameter(float t) const { return A + t*B; }

        vec3 A;
        vec3 B;
};

#endif
~~~

### color函数

~~~
vec3 color(const ray& r) {
    vec3 unit_direction = unit_vector(r.direction());
    float t = 0.5*(unit_direction.y() + 1.0);
    return (1.0-t)*vec3(1.0, 1.0, 1.0) + t*vec3(0.5, 0.7, 1.0);
}
~~~

color函数接收一个ray类型变量r，并将r的方向向量单位化，t是一个(0,1)之间的浮点数，光线y分量越小，t越小，反之，t越大
函数返回(1,1,1)，(0.5,0.7,1.0)颜色的插值作为背景颜色


### 坐标系统

使用右手坐标系，我们光线的起点设为(0,0,0)，屏幕的坐下角为(-2,-1,-1)，屏幕的宽，高分别为4，2

~~~
    vec3 lower_left_corner(-2.0, -1.0, -1.0);
    vec3 horizontal(4.0, 0.0, 0.0);
    vec3 vertical(0.0, 2.0, 0.0);
    vec3 origin(0.0, 0.0, 0.0);
~~~

如下图所示

![](https://raytracing.github.io/images/fig-1.03-cam-geom.jpg)


修改后的main函数如下

~~~
#include "svpng.inc"
#include "ray.h"

vec3 color(const ray& r) {
    vec3 unit_direction = unit_vector(r.direction());
    float t = 0.5*(unit_direction.y() + 1.0);
    return (1.0-t)*vec3(1.0, 1.0, 1.0) + t*vec3(0.5, 0.7, 1.0);
}

int main() {

    int nx=600,ny=300;
    unsigned char rgb[nx * ny * 3], *p = rgb;
    FILE *fp = fopen("test.png", "wb");

    vec3 lower_left_corner(-2.0, -1.0, -1.0);
    vec3 horizontal(4.0, 0.0, 0.0);
    vec3 vertical(0.0, 2.0, 0.0);
    vec3 origin(0.0, 0.0, 0.0);

    for (int j = ny-1; j >= 0; j--)
        for (int i = 0; i < nx; i++) {
            float u = float(i) / float(nx);
            float v = float(j) / float(ny);
            ray r(origin, lower_left_corner + u*horizontal + v*vertical);
            vec3 col = color(r);

            *p++ = int(255.99*col[0]);    /* R */
            *p++ = int(255.99*col[1]);    /* G */
            *p++ = int(255.99*col[2]);    /* B */
        }
    svpng(fp, nx, ny, rgb, 0);
    fclose(fp); 
    return 0;
}
~~~

渲染出的图像如下

![](https://pic.downk.cc/item/5e2174ab2fb38b8c3c384130.png)

在竖直方向，图像的颜色由白色(0,0,0)渐变到蓝色(0.5,0.8,1.0)，在水平方向，由于光线向量的y分量相同，颜色不变化


# 球体 表面法线 多物体


### hittable类
~~~
#ifndef HITTABLEH
#define HITTABLEH

#include "ray.h"

struct hit_record {
    float t;
    vec3 p;
    vec3 normal;
};

class hittable  {
    public:
        virtual bool hit(
            const ray& r, float t_min, float t_max, hit_record& rec) const = 0;
};

#endif
~~~

hittable是一个抽象类，其中定义了数据类型hit_record，包含光线与表面的交点p，交点的法线方向normal，t用于记录离观察者最近的点

hit函数接收ray类型变量r，t_min，t_max，并将hit信息保存在rec中

### 球体类 sphere.h

~~~
#ifndef SPHEREH
#define SPHEREH

#include "hittable.h"

class sphere: public hittable  {
    public:
        sphere() {}
        sphere(vec3 cen, float r) : center(cen), radius(r)  {};
        virtual bool hit(const ray& r, float tmin, float tmax, hit_record& rec) const;
        vec3 center;
        float radius;
};

bool sphere::hit(const ray& r, float t_min, float t_max, hit_record& rec) const {
    vec3 oc = r.origin() - center;
    float a = dot(r.direction(), r.direction());
    float b = dot(oc, r.direction());
    float c = dot(oc, oc) - radius*radius;
    float discriminant = b*b - a*c;

    if (discriminant > 0) {
        float temp = (-b - sqrt(discriminant))/a;
        if (temp < t_max && temp > t_min) {
            rec.t = temp;
            rec.p = r.point_at_parameter(rec.t);
            rec.normal = (rec.p - center) / radius;
            return true;
        }
        temp = (-b + sqrt(discriminant)) / a;
        if (temp < t_max && temp > t_min) {
            rec.t = temp;
            rec.p = r.point_at_parameter(rec.t);
            rec.normal = (rec.p - center) / radius;
            return true;
        }
    }
    //no roots
    return false;
}


#endif
~~~

sphere类继承自hittable类的初始化需要球心cent和半径r两个变量

hit函数用于计算光线是否与球体相交，数学推导如下:

我们通常用 $(x−Cx)^2+(y−Cy)^2+(z−Cz)^2=R^2$ 表示球心为(Cx,Cy,Cz)的球体

我们记 $C=(Cx,Cy,Cz) P=(x,y,z)$

那么 $dot(P-C,P-C)=(x−Cx)^2+(y−Cy)^2+(z−Cz)^2=R^2$

点P的坐标可以用ray类中的point_at_parameter函数得到，即 $p(t)=A+t∗B$

改写方程如下

$dot((p(t)−C),(p(t)−C))=dot((A+t∗B−C),(A+t∗B−C))=R^2$

$t^2⋅dot(B,B)+2t⋅dot(B,A−C)+dot(A−C,A−C)−R^2=0$

我们要做的就是对不同的t求出此方程的解（代码中在方程的系数上稍有变化，但不影响结果的正负）

![](https://raytracing.github.io/images/fig-1.04-ray-sphere.jpg)

判别式$(b^2-4ac)<0$，方程无实数解时：光线与球体无交点，输出背景色

判别式>0时，方程有两个解r1，r2，我们首先考察距离观察点角较近的交点，即方程解中较小的一个，因为光线不会穿过物体，只会照射到球体上较近的点（我们假设物体是完全不透明的），记录交点的t值，坐标p，和法线方向n

---

### hittablelist类 多物体

~~~
#ifndef HITTABLELISTH
#define HITTABLELISTH

#include "hittable.h"

class hittable_list: public hittable {
    public:
        hittable_list() {}
        hittable_list(hittable **l, int n) {list = l; list_size = n; }
        virtual bool hit(
            const ray& r, float tmin, float tmax, hit_record& rec) const;
        hittable **list;
        int list_size;
};

bool hittable_list::hit(const ray& r, float t_min, float t_max,
                        hit_record& rec) const {

    hit_record temp_rec;
    bool hit_anything = false;
    double closest_so_far = t_max;
    for (int i = 0; i < list_size; i++) {
        if (list[i]->hit(r, t_min, closest_so_far, temp_rec)) {
            hit_anything = true;
            closest_so_far = temp_rec.t;
            rec = temp_rec;
        }
    }
    return hit_anything;
}

#endif
~~~

hittablelist类会计算出光线与多个球体最近的交点


修改后的main函数

~~~
#include "svpng.inc"
#include "sphere.h"
#include "hittablelist.h"
#include<cfloat>

vec3 color(const ray& r, hittable *world) {
    hit_record rec;
    if (world->hit(r, 0.0, FLT_MAX, rec)) {
        return 0.5*vec3(rec.normal.x()+1, rec.normal.y()+1, rec.normal.z()+1);
    }
    else{
    vec3 unit_direction = unit_vector(r.direction());
    float t = 0.5*(unit_direction.y() + 1.0);
    return (1.0-t)*vec3(1.0, 1.0, 1.0) + t*vec3(0.5, 0.7, 1.0);   
    }
}

int main() {

    int nx=600,ny=300;
    unsigned char rgb[nx * ny * 3], *p = rgb;
    FILE *fp = fopen("test.png", "wb");

    vec3 lower_left_corner(-2.0, -1.0, -1.0);
    vec3 horizontal(4.0, 0.0, 0.0);
    vec3 vertical(0.0, 2.0, 0.0);
    vec3 origin(0.0, 0.0, 0.0);

    hittable *list[2];
    list[0] = new sphere(vec3(0,0,-1), 0.5);
    list[1] = new sphere(vec3(0,-100.5,-1), 100);
    hittable *world = new hittable_list(list,2);
    
    for (int j = ny-1; j >= 0; j--)
        for (int i = 0; i < nx; i++) {

            float u = float(i) / float(nx);
            float v = float(j) / float(ny);
            ray r(origin, lower_left_corner + u*horizontal + v*vertical);
            vec3 pap = r.point_at_parameter(2.0);
            vec3 col = color(r, world);
            *p++ = int(255.99*col[0]);    /* R */
            *p++ = int(255.99*col[1]);    /* G */
            *p++ = int(255.99*col[2]);    /* B */
        }
    svpng(fp, nx, ny, rgb, 0);
    fclose(fp); 
    return 0;
}
~~~

在color函数中，如果光线照射到了球体上，会根据交点的法线方向，改变颜色的值

---

产生的图片如下

![](https://pic.downk.cc/item/5e2180132fb38b8c3c398a66.png)

---
layout: post
title: Ray Tracing in One Weekend Summary V - Antialiasing
date: 2020-01-19

---

# 抗锯齿


### random类

先引用我们上一章所生成的图像

![](https://pic.downk.cc/item/5e2180132fb38b8c3c398a66.png)

可以看出，在球体的边缘会有明显的锯齿感，减少锯齿感的方法是，在计算某一个像素的颜色时，计算它邻近的区域内多个采样点的颜色，并求平均值，因此我们引入一个random类

~~~
#ifndef RANDOMH
#define RANDOMH

#include <cstdlib>

inline double random_double() {
    return rand() / (RAND_MAX + 1.0);
}
#endif
~~~

random_double函数生成一个(0,1)之间的浮点数

---

### camera类

~~~
#ifndef CAMERAH
#define CAMERAH

#include "ray.h"

class camera {
    public:
        camera() {
            lower_left_corner = vec3(-2.0, -1.0, -1.0);
            horizontal = vec3(4.0, 0.0, 0.0);
            vertical = vec3(0.0, 2.0, 0.0);
            origin = vec3(0.0, 0.0, 0.0);
        }
        ray get_ray(float u, float v) {
            return ray(origin,
                       lower_left_corner + u*horizontal + v*vertical - origin);
        }

        vec3 origin;
        vec3 lower_left_corner;
        vec3 horizontal;
        vec3 vertical;
};
#endif
~~~

amera类用于发射不同的光线，用于采样

---

修改后的main函数
~~~
#include "svpng.inc"
#include "sphere.h"
#include "hittablelist.h"
#include "camera.h"
#include "random.h"
#include<cfloat>

vec3 color(const ray& r, hittable *world) {
    hit_record rec;
    if (world->hit(r, 0.0, FLT_MAX, rec)) {
        return 0.5*vec3(rec.normal.x()+1, rec.normal.y()+1, rec.normal.z()+1);
    }
    else{
    vec3 unit_direction = unit_vector(r.direction());
    float t = 0.5*(unit_direction.y() + 1.0);
    return (1.0-t)*vec3(1.0, 1.0, 1.0) + t*vec3(0.5, 0.7, 1.0);   
    }
 
}

int main() {

    int nx=600,ny=300,ns=100;
    unsigned char rgb[nx * ny * 3], *p = rgb;
    FILE *fp = fopen("test.png", "wb");

    vec3 lower_left_corner(-2.0, -1.0, -1.0);
    vec3 horizontal(4.0, 0.0, 0.0);
    vec3 vertical(0.0, 2.0, 0.0);
    vec3 origin(0.0, 0.0, 0.0);
    camera cam;
    hittable *list[2];
    list[0] = new sphere(vec3(0,0,-1), 0.5);
    list[1] = new sphere(vec3(0,-100.5,-1), 100);
    hittable *world = new hittable_list(list,2);

    for (int j = ny-1; j >= 0; j--){
        for (int i = 0; i < nx; i++) {
            vec3 col(0,0,0);
            for(int s = 0; s < ns ; s++){
                float u = float(i + random_double()) / float(nx);
                float v = float(j + random_double()) / float(ny);
                ray r = cam.get_ray(u,v);               
                col += color(r, world);           
            }
            col /= float(ns);
            *p++ = int(255.99*col[0]);    /* R */
            *p++ = int(255.99*col[1]);    /* G */
            *p++ = int(255.99*col[2]);    /* B */   
        }
    }
    svpng(fp, nx, ny, rgb, 0);
    fclose(fp); 
    return 0;
}
~~~

我们以采样100次为例（ns=100）

---

渲染出的图片如下

![](https://pic.downk.cc/item/5e226b362fb38b8c3c4eef5c.png)

球体边缘处的像素锯齿感明显降低


# 材质

---

### 材质类

~~~
class material{
public:
    virtual bool scatter(
        const ray& r_in,
        const hit_record& rec,
        vec3& attenuation,
        ray& scattered) const = 0;
};
~~~

scatter函数接受一条光线r_in，碰撞记录rec，衰减attenuation，和散射后的光线scattered

同时要以下代码中稍做修改

sphere.h

~~~
#ifndef SPHEREH
#define SPHEREH

#include "hittable.h"

class sphere: public hittable  {
    public:
        sphere() {}
        sphere(vec3 cen, float r, material *m) : center(cen), radius(r), mat_ptr(m)  {};
        virtual bool hit(const ray& r, float tmin, float tmax, hit_record& rec) const;
        vec3 center;
        float radius;
        material *mat_ptr; /* NEW */
};

bool sphere::hit(const ray& r, float t_min, float t_max, hit_record& rec) const {
    vec3 oc = r.origin() - center;
    float a = dot(r.direction(), r.direction());
    float b = dot(oc, r.direction());
    float c = dot(oc, oc) - radius*radius;
    float discriminant = b*b - a*c;
    if (discriminant > 0) {
        float temp = (-b - sqrt(discriminant))/a;
        if (temp < t_max && temp > t_min) {
            rec.t = temp;
            rec.p = r.point_at_parameter(rec.t);
            rec.normal = (rec.p - center) / radius;
            rec.mat_ptr = mat_ptr; /* NEW */
            return true;
        }
        temp = (-b + sqrt(discriminant)) / a;
        if (temp < t_max && temp > t_min) {
            rec.t = temp;
            rec.p = r.point_at_parameter(rec.t);
            rec.normal = (rec.p - center) / radius;
            rec.mat_ptr = mat_ptr; /* NEW */
            return true;
        }
    }
    return false;
}
#endif
~~~

hittable.h

~~~
#ifndef HITTABLEH
#define HITTABLEH
#include "ray.h"

class material;

struct hit_record {
    float t;
    vec3 p;
    vec3 normal;
    material *mat_ptr;
};

class hittable  {
    public:
        virtual bool hit(
            const ray& r, float t_min, float t_max, hit_record& rec) const = 0;
};

#endif
~~~

添加类成员mat_ptr用于在球体初始化时指定材质

---

### 漫反射 diffuse

我们用lambert光照模型模拟漫反射，lambert模型用于纯粹的漫反射表面的物体，光源照射到物体表面后，向四面八方反射，产生漫反射效果。

![](https://raytracing.github.io/images/fig-1-08-1.jpg)

如图所示，光线在照射到平面上时反射的方向是随机的

random.h
~~~
vec3 random_in_unit_sphere() {
    vec3 p;
    do {
        p = 2.0*vec3(random_double(), random_double(), random_double()) - vec3(1,1,1);
    } while (p.squared_length() >= 1.0);
    return p;
}
~~~
random_in_unit_sphere生成一个长度为1的随机向量

![](https://raytracing.github.io/images/fig-1-08-2.jpg)

表面法线为 **n**，光线与表面交点 **p**，随机向量 **random**，**p+n+random**就是漫反射的方向

添加material的子类lambertian

~~~
class lambertian : public material {
    public:
        lambertian(const vec3& a) : albedo(a) {}
        virtual bool scatter(const ray& r_in, const hit_record& rec, vec3& attenuation, ray& scattered) const {
            vec3 target = rec.p + rec.normal + random_in_unit_sphere();
            scattered = ray(rec.p, target-rec.p);
            attenuation = albedo;
            return true;
        }
        vec3 albedo;
};
~~~

返回值为true，scattered为反射光线

---

### 高光反射 specular

计算反射方向，数学推导如下

![](https://raytracing.github.io/images/fig-1-09-1.jpg)

我们已知入射方向 **v**，表面法线方向 **n**且长度为1，v在n方向上的投影为 **dot(n,v)**，反射方向即为 **v-2·n·dot(v,n)**

计算反射方向的函数reflect

~~~
vec3 reflect(const vec3& v, const vec3& n) {
    return v - 2*dot(v,n)*n;
}
~~~

添加material的自类metal

~~~
class metal : public material {
    public:
        metal(const vec3& a) : albedo(a) {}
        virtual bool scatter(const ray& r_in, const hit_record& rec,
                             vec3& attenuation, ray& scattered) const {
            vec3 reflected = reflect(unit_vector(r_in.direction()), rec.normal);
            scattered = ray(rec.p, reflected);
            attenuation = albedo;
            return (dot(scattered.direction(), rec.normal) > 0);
        }
        vec3 albedo;
};
~~~

---

修改后的main函数

~~~
#include "svpng.inc"
#include "sphere.h"
#include "hittablelist.h"
#include "camera.h"
#include "random.h"
#include "material.h"
#include<cfloat>

vec3 color(const ray& r, hittable *world, int depth) {
    hit_record rec;
    if (world->hit(r, 0.001, FLT_MAX, rec)) {
        ray scattered;
        vec3 attenuation;
        if (depth < 50 && rec.mat_ptr->scatter(r, rec, attenuation, scattered)) {
            return attenuation*color(scattered, world, depth+1);
        }
        else {
            return vec3(0,0,0);
        }
    }
    else{
    vec3 unit_direction = unit_vector(r.direction());
    float t = 0.5*(unit_direction.y() + 1.0);
    return (1.0-t)*vec3(1.0, 1.0, 1.0) + t*vec3(0.5, 0.7, 1.0);   
    }
}

int main() {

    int nx=600,ny=300,ns=100;
    unsigned char rgb[nx * ny * 3], *p = rgb;
    FILE *fp = fopen("test.png", "wb");

    vec3 lower_left_corner(-2.0, -1.0, -1.0);
    vec3 horizontal(4.0, 0.0, 0.0);
    vec3 vertical(0.0, 2.0, 0.0);
    vec3 origin(0.0, 0.0, 0.0);
    camera cam;
    hittable *list[4];
    list[0] = new sphere(vec3(0,0,-1), 0.5, new lambertian(vec3(0.8, 0.3, 0.3)));
    list[1] = new sphere(vec3(0,-100.5,-1), 100, new lambertian(vec3(0.8, 0.8, 0.0)));
    list[2] = new sphere(vec3(1,0,-1), 0.5, new lambertian(vec3(0.2,0.2,0.8)));
    list[3] = new sphere(vec3(-1,0,-1), 0.5, new metal(vec3(0.8, 0.8, 0.8)));
    hittable *world = new hittable_list(list,4);

    for (int j = ny-1; j >= 0; j--){
        for (int i = 0; i < nx; i++) {
            vec3 col(0,0,0);
            for(int s = 0; s < ns ; s++){
                float u = float(i + random_double()) / float(nx);
                float v = float(j + random_double()) / float(ny);
                ray r = cam.get_ray(u,v);               
                col += color(r, world, 0);           
            }
            col /= float(ns);
            col = vec3( sqrt(col[0]), sqrt(col[1]), sqrt(col[2]) );
            *p++ = int(255.99*col[0]);    /* R */
            *p++ = int(255.99*col[1]);    /* G */
            *p++ = int(255.99*col[2]);    /* B */   
        }
    }
    svpng(fp, nx, ny, rgb, 0);
    fclose(fp); 
    return 0;
}
~~~

在color函数中，depth的值为光线反射的次数

depth=0

![](https://pic.downk.cc/item/5e22f3e52fb38b8c3c5c4570.png)

depth=1

![](https://pic.downk.cc/item/5e22f3e52fb38b8c3c5c4572.png)

depth=2

![](https://pic.downk.cc/item/5e22f3e52fb38b8c3c5c4574.png)

depth=1000

![](https://pic.downk.cc/item/5e22de9e2fb38b8c3c59f22f.png)

反射次数越多，图像越理想

在迭代过程中，我们假设光线无衰减，attenuation值不变，为物体表面的颜色，因为attenuation<0，反射次数越多，光线颜色的rgb值越来越小，趋近与0，这也是为什么在球体的交界处产生阴影的原因

---

### gamma correction

[一篇对gamma矫正的介绍](https://learnopengl-cn.github.io/05%20Advanced%20Lighting/02%20Gamma%20Correction/#gamma)

人眼对灰阶的感知不是线性的，而是类似1/α的曲线，所以我们生成的图片在某些地方看起来会过暗或过亮

~~~
col = vec3( sqrt(col[0]), sqrt(col[1]), sqrt(col[2]) );
~~~

以上代码的作用是对颜色信息进行一个非线性的映射

---

### shadow acne problem

一些关于此问题的描述和解释

[zhihu](https://www.zhihu.com/question/49090321)

[cause-of-shadow-acne -stackoverflow](https://computergraphics.stackexchange.com/questions/2192/cause-of-shadow-acne)

~~~
if (world->hit(r, 0.001, MAXFLOAT, rec)) {...}
~~~

解决方法是，在上面语句中，将t_min加上一个bias（0.001）

渲染出的图像对比

![](https://pic.downk.cc/item/5e22fcee2fb38b8c3c5d2905.png)

![](https://pic.downk.cc/item/5e22de9e2fb38b8c3c59f22f.png)

---

### fuzz

~~~
class metal : public material {
    public:
        metal(const vec3& a, float f) : albedo(a) { /*NEW*/
            if (f < 1) fuzz = f; else fuzz = 1;
        }

        virtual bool scatter(const ray& r_in, const hit_record& rec,
                             vec3& attenuation, ray& scattered) const {
            vec3 reflected = reflect(unit_vector(r_in.direction()), rec.normal);
            scattered = ray(rec.p, reflected + fuzz*random_in_unit_sphere()); /*NEW*/
            attenuation = albedo;
            return (dot(scattered.direction(), rec.normal) > 0);
        }
        vec3 albedo;
        float fuzz; /*NEW*/
};
~~~

为metal材质添加模糊效果

效果如下：

![](https://pic.downk.cc/item/5e2306652fb38b8c3c5e1354.png)

---
layout: post
title: Ray Tracing in One Weekend Summary VII - Dielectrics
date: 2020-01-21

---

# Dielectrics

### 折射向量

光线照射在水，玻璃等物体时，会发生折射现象，如图所示

![](https://raytracing.github.io/images/fig-1-10-1.jpg)

求解折射方向的步骤如下

[用 C 语言画光（五）：折射](https://zhuanlan.zhihu.com/p/31127076)

计算折射方向的代码如下

~~~
bool refract(const vec3& v, const vec3& n, float ni_over_nt, vec3& refracted) {
    vec3 uv = unit_vector(v);
    float dt = dot(uv, n);
    float discriminant = 1.0 - ni_over_nt*ni_over_nt*(1-dt*dt);
    if (discriminant > 0) {
        refracted = ni_over_nt*(uv - n*dt) - n*sqrt(discriminant);
        return true;
    }
    else
        return false;
}
~~~

ni_over_nt为两种介质折射率之比n1/n2

---

### 反射系数

照射到物体上的光并不会全部发生折射，有一部分光会发生反射，计算反射稀疏的代码如下

~~~
float schlick(float cosine, float ref_idx) {
    float r0 = (1-ref_idx) / (1+ref_idx);
    r0 = r0*r0;
    return r0 + (1-r0)*pow((1 - cosine),5);
}
~~~

参考

[从Phong光照模型到 BRDF](https://zhuanlan.zhihu.com/p/75360639)

[怎么模拟ray tracing图形中介质材料的颜色](https://blog.csdn.net/libing_zeng/article/details/54428732)中24.1.2 介质界面的反射系数

---

### 电解质类deilectric

~~~
class dielectric : public material {
    public:
        dielectric(float ri) : ref_idx(ri) {}
        virtual bool scatter(const ray& r_in, const hit_record& rec,
                             vec3& attenuation, ray& scattered) const {
            vec3 outward_normal;
            vec3 reflected = reflect(r_in.direction(), rec.normal);
            float ni_over_nt;
            attenuation = vec3(1.0, 1.0, 1.0);
            vec3 refracted;

            float reflect_prob;
            float cosine;

            if (dot(r_in.direction(), rec.normal) > 0) {
                 outward_normal = -rec.normal;
                 ni_over_nt = ref_idx;
                 cosine = ref_idx * dot(r_in.direction(), rec.normal)
                        / r_in.direction().length();
            }
            else {
                 outward_normal = rec.normal;
                 ni_over_nt = 1.0 / ref_idx;
                 cosine = -dot(r_in.direction(), rec.normal)
                        / r_in.direction().length();
            }

            if (refract(r_in.direction(), outward_normal, ni_over_nt, refracted)) {
               reflect_prob = schlick(cosine, ref_idx);
            }
            else {
               reflect_prob = 1.0;
            }

            if (random_double() < reflect_prob) {
               scattered = ray(rec.p, reflected);
            }
            else {
               scattered = ray(rec.p, refracted);
            }

            return true;
        }

        float ref_idx;
};
~~~

---

修改main函数

~~~    
list[0] = new sphere(vec3(0,0,-1), 0.5, new metal(vec3(0.1, 0.2, 0.5),1));
list[1] = new sphere(vec3(0,-100.5,-1), 100, new lambertian(vec3(0.8, 0.8, 0.0)));
list[2] = new sphere(vec3(1,0,-1), 0.5, new metal(vec3(0.8, 0.6, 0.2), 0.3));
list[3] = new sphere(vec3(-1,0,-1), 0.5, new dielectric(1.5));
~~~

---

渲染出的图片如下

![](https://pic.downk.cc/item/5e2316052fb38b8c3c5fdd85.png)

可以看出左侧的球体中像是颠倒的，下面这副图可以给出一个解释(图源https://blog.csdn.net/libing_zeng/article/details/54428732)

![](https://img-blog.csdn.net/20170114205419889?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGliaW5nX3plbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

---
layout: post
title: Ray Tracing in One Weekend Summary VIII - Camera
date: 2020-01-22

---

# 相机

---

### 投影

![](https://raytracing.github.io/images/fig-1-11-1.jpg)

θ是可视角度，图像的高度 **h=tan(θ/2)**

我们想要我们的相机处于任意位置并可以指向任意位置，即从 **lookfrom**指向**lookat**

![](https://raytracing.github.io/images/fig-1-11-2.jpg)

我们要求得垂直于 **lookfrom-lookat**向量的平面上的单位向量 **u**，**v**

![](https://raytracing.github.io/images/fig-1-11-3.jpg)

已知有 **vup**，**lookfrom-lookat**两个向量，求 **u**，**v**向量的方式如下：

**u=cross(vup,lookfrom)**

**v=cross(u,lookfrom)**

**u**，**v**是投影平面水平和竖直方向的步长


---

### 景深

关于景深的介绍笔者不再赘述，用代码模拟这种效果的基本思想如下：

![](https://raytracing.github.io/images/fig-1-12-2.jpg)

关于成像位置，引入参数focus_dist之后，成像位置将会是-focus_dist*w。图片的高宽都需要乘以参数focus_dist。光线从镜头内的随机一点射出，打到焦平面上

---

### 相机类

~~~
#ifndef CAMERAH
#define CAMERAH

#include "random.h"
#include "ray.h"

vec3 random_in_unit_disk() {
    vec3 p;
    do {
        p = 2.0*vec3(random_double(),random_double(),0) - vec3(1,1,0);
    } while (dot(p,p) >= 1.0);
    return p;
}

class camera {
    public:
        camera(vec3 lookfrom, vec3 lookat, vec3 vup, float vfov, float aspect,
               float aperture, float focus_dist) {
            lens_radius = aperture / 2;
            float theta = vfov*M_PI/180;
            float half_height = tan(theta/2);
            float half_width = aspect * half_height;
            origin = lookfrom;
            w = unit_vector(lookfrom - lookat);
            u = unit_vector(cross(vup, w));
            v = cross(w, u);
            lower_left_corner = origin
                              - half_width * focus_dist * u
                              - half_height * focus_dist * v
                              - focus_dist * w;
            horizontal = 2*half_width*focus_dist*u;
            vertical = 2*half_height*focus_dist*v;
        }

        ray get_ray(float s, float t) {
            vec3 rd = lens_radius*random_in_unit_disk();
            vec3 offset = u * rd.x() + v * rd.y();
            return ray(origin + offset,
                       lower_left_corner + s*horizontal + t*vertical
                           - origin - offset);
        }

        vec3 origin;
        vec3 lower_left_corner;
        vec3 horizontal;
        vec3 vertical;
        vec3 u, v, w;
        float lens_radius;
};
#endif
~~~

camera类的构造函数中

lookfrom：观察点

lookat：目标点，即视野中心

vup：摄像机竖直上方

vfov：可视角度

aspect：长宽比

focus_dist：焦距

aperture：光圈大小

---

修改main函数中下面语句

~~~
vec3 lookfrom(3,3,2);
vec3 lookat(0,0,-1);
float dist_to_focus = (lookfrom-lookat).length();
float aperture = 2.0;

camera cam(lookfrom, lookat, vec3(0,1,0), 50,
           float(nx)/float(ny), aperture, dist_to_focus);
~~~

将观察点放置在(-2,2,1)，指向(0,0,-1)（中间蓝球的中心），y轴为上方，可视角度为90度...

---

结果如下

![](https://pic.downk.cc/item/5e25db982fb38b8c3ca6713c.png)


---
layout: post
title: Ray Tracing in One Weekend Summary IX - Next
date: 2020-01-22

---

# 封面图的渲染

---

一系列随机的球体

~~~
hittable *random_scene() {
    int n = 500;
    hittable **list = new hittable*[n+1];
    list[0] =  new sphere(vec3(0,-1000,0), 1000, new lambertian(vec3(0.5, 0.5, 0.5)));
    int i = 1;
    for (int a = -11; a < 11; a++) {
        for (int b = -11; b < 11; b++) {
            float choose_mat = random_double();
            vec3 center(a+0.9*random_double(),0.2,b+0.9*random_double());
            if ((center-vec3(4,0.2,0)).length() > 0.9) {
                if (choose_mat < 0.8) {  // diffuse
                    list[i++] = new sphere(center, 0.2,
                        new lambertian(vec3(random_double()*random_double(),
                                            random_double()*random_double(),
                                            random_double()*random_double())
                        )
                    );
                }
                else if (choose_mat < 0.95) { // metal
                    list[i++] = new sphere(center, 0.2,
                            new metal(vec3(0.5*(1 + random_double()),
                                           0.5*(1 + random_double()),
                                           0.5*(1 + random_double())),
                                      0.5*random_double()));
                }
                else {  // glass
                    list[i++] = new sphere(center, 0.2, new dielectric(1.5));
                }
            }
        }
    }

    list[i++] = new sphere(vec3(0, 1, 0), 1.0, new dielectric(1.5));
    list[i++] = new sphere(vec3(-4, 1, 0), 1.0, new lambertian(vec3(0.4, 0.2, 0.1)));
    list[i++] = new sphere(vec3(4, 1, 0), 1.0, new metal(vec3(0.7, 0.6, 0.5), 0.0));

    return new hittable_list(list,i);
}
~~~

---

修改main函数

~~~

int main() {

    int nx=600,ny=300,ns=100;
    unsigned char rgb[nx * ny * 3], *p = rgb;
    FILE *fp = fopen("test.png", "wb");

    vec3 lookfrom(0,10,0);
    vec3 lookat(0,0,-1);
    float dist_to_focus = (lookfrom-lookat).length();
    float aperture = 0.01;

    camera cam(lookfrom, lookat, vec3(0,1,0), 50,
           float(nx)/float(ny), aperture, dist_to_focus);
    hittable *world = random_scene();

    for (int j = ny-1; j >= 0; j--){
        for (int i = 0; i < nx; i++) {
            vec3 col(0,0,0);
            for(int s = 0; s < ns ; s++){
                float u = float(i + random_double()) / float(nx);
                float v = float(j + random_double()) / float(ny);
                ray r = cam.get_ray(u,v);               
                col += color(r, world, 0);           
            }
            col /= float(ns);
            col = vec3( sqrt(col[0]), sqrt(col[1]), sqrt(col[2]) );
            *p++ = int(255.99*col[0]);    /* R */
            *p++ = int(255.99*col[1]);    /* G */
            *p++ = int(255.99*col[2]);    /* B */   
        }
    }
    svpng(fp, nx, ny, rgb, 0);
    fclose(fp); 
    return 0;
}
~~~

---

### output

![](https://pic.downk.cc/item/5e26bdf52fb38b8c3cb88f3f.png)

---

### next

- 光源

- 纹理

- 多边形

-并行计算

---
layout: post
title: Ray Tracing in One Weekend Summary X - Bounding Volume Hierarchies
date: 2020-01-23

---

# 层次包围盒

---

本章我们重构hittable类，可以使得代码运行的更快，在之前的光线追踪器中，计算光线-物体相交是最费时的步骤，运算的时间和物体的数量是线性相关的，而且，我们用二分搜索的思想

首先把模型分为两类：1）划分空间 2）划分对象

然后，每条光线的相交就是一个次线性搜索

对多个物体，用bounding volume的关键就是，找到一个包围了所有物体的包围盒，如果光线和包围盒没有交点，那光线和这些物体都不相交，判断碰撞的伪代码如下

~~~
if (ray hits bounding object)
    return whether ray hits bounded objects
else
    return false
~~~

如上图，每一个物体都有一个bounding volume，多个物体的bounding volume相互叠加，就形成了一个有层级关系的bounding volume，

![](https://raytracing.github.io/images/fig-2-3-01.jpg)

对此层级检测碰撞的伪代码为

~~~
if(hit purple) //紫色
    hit0 = hits blue enclosed objects
    hit1 = hits red enclosed objects
    if(hit0 or hit1)
        return true and info of closer hit //返回hit的信息
else
    return false
~~~

在实际操作中，我们采用与坐标轴平行的长方体（axis-aligned bounding rectangular parallelepiped）作为包围盒，或叫做aabbs，我们只用检测光线是否与包围盒相交，而不用得知交点的具体信息

以2D平面为例

**p(t)= a+t*b**

x轴上

**x0=ax0+t0∗bx0**
 
**x1=ax1+t1∗bx1**

解得：

**t0=(x0-ax0)/bx0**

**t1=(x1-ax1)/bx1**

同理，在y轴上

**t0=(y0-ay0)/by0**

**t1=(y1-ay1)/by1**

x，y是分开计算的，接下来，判断x，y轴的(t0,t1)有没有相交的部分即可



判断的方法如下：

![](https://raytracing.github.io/images/fig-2-3-04.jpg)

    两个区间的左端点最大值小于右端点的最小值
    　　有交点
    反之
    　　无交点

注意：

1）直接计算出来的t0，t1大小不确定，需要先判断，调整大小使t0 < t1

2）如果除数Bx=0，或者分子是0，求出来的解也许无意义，需要提前检测

### aabb.h

~~~
#ifndef AABB
#define AABB 

#include "hittable.h"
#include "ray.h"

//faster than fmin and fmax because it doesn’t worry about NaNs and other exceptions
inline float ffmin(float a, float b) { return a < b ? a : b; }
inline float ffmax(float a, float b) { return a > b ? a : b; }

class aabb {
    public:
        aabb() {}
        aabb(const vec3& a, const vec3& b) { _min = a; _max = b;}
        vec3 min() const {return _min; }
        vec3 max() const {return _max; }
        virtual bool hit(const ray& r, float tmin, float tmax) const;

        vec3 _min;
        vec3 _max;
};

inline bool aabb::hit(const ray& r, float tmin, float tmax) const {
    for (int a = 0; a < 3; a++) {
        float invD = 1.0f / r.direction()[a];
        float t0 = (min()[a] - r.origin()[a]) * invD;
        float t1 = (max()[a] - r.origin()[a]) * invD;
        if (invD < 0.0f)
            std::swap(t0, t1);
        tmin = t0 > tmin ? t0 : tmin;
        tmax = t1 < tmax ? t1 : tmax;
        if (tmax <= tmin)
            return false;
    }
    return true;
}
#endif
~~~

接下来我们在hittable.h中添加bounding_box纯虚函数，这样，在每个物体在计算时都会有一个包围盒了

~~~
class hittable {
    public:
        virtual bool hit(
            const ray& r, float t_min, float t_max, hit_record& rec) const = 0;
        virtual bool bounding_box(float t0, float t1, aabb& box) const = 0;
};
~~~

sphere的包围盒：球心加半径

~~~
bool sphere::bounding_box(float t0, float t1, aabb& box) const {
    box = aabb(center - vec3(radius, radius, radius),
               center + vec3(radius, radius, radius));
    return true;
}
~~~

moving sphere的包围盒：对t0时刻的box和t1时刻的box，取一个更大的boundingbox

~~~
bool moving_sphere::bounding_box(float t0, float t1, aabb& box) const {
    aabb box0(center(t0) - vec3(radius, radius, radius),
              center(t0) + vec3(radius, radius, radius));
    aabb box1(center(t1) - vec3(radius, radius, radius),
              center(t1) + vec3(radius, radius, radius));
    box = surrounding_box(box0, box1);
    return true;
}
~~~

hittable_list的包围盒：对list的每个物体计算surrounding_box，并叠加，最终返回一个包含所有物体的包围盒

~~~
bool hittable_list::bounding_box(float t0, float t1, aabb& box) const {
    if (list_size < 1) return false;
    aabb temp_box;
    bool first_true = list[0]->bounding_box(t0, t1, temp_box);
    if (!first_true)
        return false;
    else
        box = temp_box;
    for (int i = 1; i < list_size; i++) {
        if(list[i]->bounding_box(t0, t1, temp_box)) {
            box = surrounding_box(box, temp_box);
        }
        else
            return false;
    }
    return true;
}
~~~

~~~
//两个包围盒叠加生成的包围盒
aabb surrounding_box(aabb box0, aabb box1) {
    vec3 small( ffmin(box0.min().x(), box1.min().x()),
                ffmin(box0.min().y(), box1.min().y()),
                ffmin(box0.min().z(), box1.min().z()));
    vec3 big  ( ffmax(box0.max().x(), box1.max().x()),
                ffmax(box0.max().y(), box1.max().y()),
                ffmax(box0.max().z(), box1.max().z()));
    return aabb(small,big);
}
~~~

### bvh.h



~~~
class bvh_node : public hittable {
    public:
        bvh_node() {}
        bvh_node(hittable **l, int n, float time0, float time1);
        virtual bool hit(const ray& r, float tmin, float tmax, hit_record& rec) const;
        virtual bool bounding_box(float t0, float t1, aabb& box) const;

        hittable *left;
        hittable *right;
        aabb box;
};

bool bvh_node::bounding_box(float t0, float t1, aabb& b) const {
    b = box;
    return true;
}

~~~

hit函数中，如果光线击中了节点的包围盒，对于左右子树进行递归操作，直到射到叶子节点，击中重叠的部分，击中的数据用引用rec传出去。

~~~
bool bvh_node::hit(const ray& r, float t_min, float t_max, hit_record& rec) const {
    if (box.hit(r, t_min, t_max)) {
        hit_record left_rec, right_rec;
        bool hit_left = left->hit(r, t_min, t_max, left_rec);
        bool hit_right = right->hit(r, t_min, t_max, right_rec);
        if (hit_left && hit_right) {
            if (left_rec.t < right_rec.t)
                rec = left_rec;
            else
                rec = right_rec;
            return true;
        }
        else if (hit_left) {
            rec = left_rec;
            return true;
        }
        else if (hit_right) {
            rec = right_rec;
            return true;
        }
        else
            return false;
    }
    else return false;
}

~~~

我们要将对象列表bvh_node分为两个子列表，如果划分得很好，代码的运行速度就会快很多，最佳是满二叉树的情况，步骤如下：

    随机选择一个轴
    使用qsort对物体进行排序
    在每个子树中放一半物体

    
~~~
bvh_node::bvh_node(hittable **l, int n, float time0, float time1) {
    int axis = int(3*random_double());

    if (axis == 0)
       qsort(l, n, sizeof(hittable *), box_x_compare);
    else if (axis == 1)
       qsort(l, n, sizeof(hittable *), box_y_compare);
    else
       qsort(l, n, sizeof(hittable *), box_z_compare);

    if (n == 1) {
        left = right = l[0];
    }
    else if (n == 2) {
        left = l[0];
        right = l[1];
    }
    else {
        left = new bvh_node(l, n/2, time0, time1);
        right = new bvh_node(l + n/2, n - n/2, time0, time1);
    }

    aabb box_left, box_right;

    if (!left->bounding_box(time0, time1, box_left) ||
        !right->bounding_box(time0, time1, box_right)) {

        std::cerr << "no bounding box in bvh_node constructor\n";
    }

    box = surrounding_box(box_left, box_right);
}
~~~

compare函数

~~~
int box_x_compare (const void * a, const void * b) {
    aabb box_left, box_right;
    hittable *ah = *(hittable**)a;
    hittable *bh = *(hittable**)b;

    if (!ah->bounding_box(0,0, box_left) || !bh->bounding_box(0,0, box_right))
        std::cerr << "no bounding box in bvh_node constructor\n";

    if (box_left.min().x() - box_right.min().x() < 0.0)
        return -1;
    else
        return 1;
}

int box_y_compare (const void * a, const void * b) {
    aabb box_left, box_right;
    hittable *ah = *(hittable**)a;
    hittable *bh = *(hittable**)b;

    if (!ah->bounding_box(0,0, box_left) || !bh->bounding_box(0,0, box_right))
        std::cerr << "no bounding box in bvh_node constructor\n";

    if (box_left.min().y() - box_right.min().y() < 0.0)
        return -1;
    else
        return 1;
}

int box_z_compare (const void * a, const void * b) {
    aabb box_left, box_right;
    hittable *ah = *(hittable**)a;
    hittable *bh = *(hittable**)b;

    if (!ah->bounding_box(0,0, box_left) || !bh->bounding_box(0,0, box_right))
        std::cerr << "no bounding box in bvh_node constructor\n";

    if (box_left.min().z() - box_right.min().z() < 0.0)
        return -1;
    else
        return 1;
}
~~~

---
layout: post
title: Ray Tracing in One Weekend Summary XI - Textures
date: 2020-01-24

---

# 纹理

---

### 纹理类

~~~
#ifndef TEXTURE
#define TEXTURE 


class texture {
    public:
        virtual vec3 value(float u, float v, const vec3& p) const = 0;
};

//纯色纹理
class constant_texture : public texture {
    public:
        constant_texture() {}
        constant_texture(vec3 c) : color(c) {}
        virtual vec3 value(float u, float v, const vec3& p) const {
            return color;
        }
        vec3 color;
};

//棋盘纹理
class checker_texture : public texture {
    public:
        checker_texture() {}
        checker_texture(texture *t0, texture *t1): even(t0), odd(t1) {}
        virtual vec3 value(float u, float v, const vec3& p) const {
            float sines = sin(10*p.x())*sin(10*p.y())*sin(10*p.z());
            if (sines < 0)
                return odd->value(u, v, p);
            else
                return even->value(u, v, p);
        }
        texture *odd;
        texture *even;
};

#endif
~~~

修改材质类中相关代码

~~~
class lambertian : public material {
    public:
        lambertian(texture *a) : albedo(a) {}
        virtual bool scatter(const ray& r_in, const hit_record& rec,
                             vec3& attenuation, ray& scattered) const {
            vec3 target = rec.p + rec.normal + random_in_unit_sphere();
            scattered = ray(rec.p, target - rec.p);
            attenuation = albedo->value(0, 0, rec.p);
            return true;
        }
        texture *albedo;
};
~~~

在纯色纹理中，value函数直接返回所接受的vec3 p的颜色值

在棋盘纹理中，value函数根据p的xyz坐标返回一个周期性的二值颜色，用三角函数实现是一个不错的选择

---

### Examples

以下代码定义了纯色和棋盘两种不同的材质

~~~
hittable *two_spheres() {
    texture *checker = new checker_texture(
        new constant_texture(vec3(0.2, 0.3, 0.1)),
        new constant_texture(vec3(0.9, 0.9, 0.9))
    );
    texture* simple=new constant_texture(vec3(0.5,0.5,0.5));


    int n = 50;
    hittable **list = new hittable*[n+1];
    list[0] = new sphere(vec3(0,-10, 0), 10, new lambertian(checker));
    list[1] = new sphere(vec3(0, 10, 0), 10, new lambertian(simple));
    return new hittable_list(list,2);
}
~~~

---

### 结果如下

![](https://pic.downk.cc/item/5e2a655a2fb38b8c3c5a145a.png)

---

### 渐变色纹理

顺手写了一个根据交点坐标渲染渐变颜色的纹理

~~~
class rainbow_texture:public texture
{
public:
    rainbow_texture(){};
    virtual vec3 value(float u,float v,const vec3& p)const{
        return vec3(fabs(sin(p.x())),fabs(sin(p.y())),fabs(sin(p.z())));
    }
    ~rainbow_texture();
};
~~~

---

渲染  结果如下

![](https://pic.downk.cc/item/5e2a86032fb38b8c3c5c4176.png)

---
layout: post
title: Ray Tracing in One Weekend Summary XII - Perlin Noise
date: 2020-01-25

---

由程序产生噪声的方法大致可以分为两类：

- 基于晶格的方法（Lattice based）又可细分为两种：
    第一种是梯度噪声（Gradient noise），包括Perlin noise，Simplex noise，Wavelet noise等
    第二种是Value noise。

- 基于点的方法（Point based）
    Worley noise

---

下图是我们常见的白噪声图像

![](https://raytracing.github.io/images/img-2-5-01.jpg)

梯度噪声产生的纹理具有连续性，因为在其噪声与相邻点噪声的加权有关，所以经常用来模拟山脉、云朵等具有连续性的物质，perlin noise，其它梯度噪声还有Simplex Noise和Wavelet Noise，它们也是由Perlin Noise演变而来

Perlin Noise返回如下所示的图像，是一个具有模糊效果的噪声图

![](https://raytracing.github.io/images/img-2-5-02.jpg)


---

### 算法步骤

生成perlin noise的步骤如下：

- 给定点输入点p

- 对于每一个与点p相邻的方格端点（二维的噪声就是4个点，三维噪声就是8个点）

- 挑选一个伪随机梯度向量

- 计算随机向量和距离的点积

- 每个维度上，均采用缓和曲线，计算出加权平均值，缓和曲线可以选用3*t^3-2*t^3

具体推导可见下面几篇文章以及pen perlin的论文

[](https://blog.csdn.net/qq_34302921/article/details/80849139)

[](http://www.twinklingstar.cn/2015/2581/classical-perlin-noise/#2_Perlin)



---

### perlin.h

~~~
static vec3* perlin_generate() {
    vec3 *p = new vec3[256];
    for (int i = 0; i < 256; ++i) {
        double x_random = 2*random_double() - 1;
        double y_random = 2*random_double() - 1;
        double z_random = 2*random_double() - 1;
        p[i] = unit_vector(vec3(x_random, y_random, z_random));
    }
    return p;
}
~~~

perlin_generate返回一个长度为1的随机vec3数组

~~~
void permute(int *p, int n) {
    for (int i = n-1; i > 0; i--) {
        int target = int(random_double()*(i+1));
        int tmp = p[i];
        p[i] = p[target];
        p[target] = tmp;
    }
    return;
}
~~~

打乱数组p的顺序，经测试，std::random_shuffle也可以达到同样的效果

~~~
static int* perlin_generate_perm() {
    int * p = new int[256];
    for (int i = 0; i < 256; i++)
        p[i] = i;
    permute(p, 256);
    return p;
}
~~~

perlin_generate_perm返回一个乱序无重复的从0~255的数组

~~~
inline float perlin_interp(vec3 c[2][2][2], float u, float v, float w) {
    //缓和曲线
    float uu = u*u*(3-2*u);
    float vv = v*v*(3-2*v);
    float ww = w*w*(3-2*w);
    float accum = 0;
    for (int i=0; i < 2; i++)
        for (int j=0; j < 2; j++)
            for (int k=0; k < 2; k++) {
                vec3 weight_v(u-i, v-j, w-k);
                //三维线性插值*权重*梯度
                accum += (i*uu + (1-i)*(1-uu))*
                    (j*vv + (1-j)*(1-vv))*
                    (k*ww + (1-k)*(1-ww))*dot(c[i][j][k], weight_v);
            }
    return accum;
}
~~~

perlin类

~~~
class perlin {
    public:
        float noise(const vec3& p) const;
        static vec3 *ranvec;
        static int *perm_x;
        static int *perm_y;
        static int *perm_z;
};

vec3* perlin::ranvec = perlin_generate();//随机向量
int *perlin::perm_x = perlin_generate_perm();//随机数组
int *perlin::perm_y = perlin_generate_perm();//随机数组
int *perlin::perm_z = perlin_generate_perm();//随机数组

float perlin::noise(const vec3& p) const {
    float u = p.x() - floor(p.x());
    float v = p.y() - floor(p.y());
    float w = p.z() - floor(p.z());

    int i = floor(p.x());
    int j = floor(p.y());
    int k = floor(p.z());

    vec3 c[2][2][2];
    //挑选p临近8个点的随机向量
    for (int di=0; di < 2; di++)
        for (int dj=0; dj < 2; dj++)
            for (int dk=0; dk < 2; dk++)
                //只保留后8位
                c[di][dj][dk] = ranvec[perm_x[(i+di) & 255] ^ perm_y[(j+dj) & 255] ^ perm_z[(k+dk) & 255] ];

    return perlin_interp(c, u, v, w);
}
~~~

perlin_noise材质

~~~
class noise_texture : public texture {
    public:
        noise_texture() {}
        noise_texture(float sc) : scale(sc) {}
        virtual vec3 value(float u, float v, const vec3& p) const {
            return vec3(1,1,1) * noise.noise(scale*p);
        }

        perlin noise;
        float scale;
};
~~~

noise_texture接受一个scale值，scale越大，噪声纹理越密

---

### test

~~~
hittable *two_perlin_spheres() {
    texture* pertext = new noise_texture(10);
    texture* simple=new constant_texture(vec3(0.0,0.0,0.0));
    hittable **list = new hittable*[2];
    list[0] = new sphere(vec3(0,-1000, 0), 1000, new lambertian(pertext));
    list[1] = new sphere(vec3(0, 2, 0), 1, new lambertian(pertext));
    return new hittable_list(list, 2);
}
~~~

~~~
vec3 lookfrom(13,4,3);
vec3 lookat(0,2,0);
float dist_to_focus = 10.0;
float aperture = 0.0;
float vfov = 20.0;

camera cam(lookfrom, lookat, vec3(0,1,0), vfov, float(nx)/float(ny),aperture, dist_to_focus, 0.0, 1.0);
hittable* world=two_perlin_spheres();
~~~

渲染出的图片如下

![](https://pic.downk.cc/item/5e3036a92fb38b8c3ccb988b.png)

---

### turb

~~~
float turb(const vec3 &p, int depth = 7) const {
    float accum = 0;
    vec3 temp_p = p;
    float weight = 1.0;

    for (int i = 0; i < depth; i++) {
      accum += weight * noise(temp_p);
      weight *= 0.5;
      temp_p *= 2;
    }
    return fabs(accum);
}
~~~

~~~
class noise_texture : public texture {
    public:
        noise_texture() {}
        noise_texture(float sc) : scale(sc) {}
        virtual vec3 value(float u, float v, const vec3& p) const {
            return vec3(1,1,1) * 0.5 * (1 + sin(scale*p.z() + 10*noise.turb(p)));
        }
        perlin noise;
        float scale;
};
~~~

加上turbulence后渲染出的大理石材质

![](https://pic.downk.cc/item/5e303a112fb38b8c3ccbe11a.png)

---
layout: post
title: Ray Tracing in One Weekend Summary XIII - Image Texture Mapping
date: 2020-01-26

---

# 纹理映射

纹理映射，就是通过读取一张图片，使用uv映射的方法，直接将一张图片的纹理绘制在物体表面。

直接的方法是缩放uv，uv是[0,1]之间的float。而像素肯定大于这个区间，所以需要进行缩放，用(i,j)表示当前像素，nx和ny表示纹理的分辨率，所以对于任意像素(i,j)位置，对应的uv坐标就是

**u = i / (nx - 1)**

**v = j / (ny - 1)**

---

### 读取纹理

[](https://github.com/nothings/stb/blob/master/stb_image.h)

我们用stb_image这个库来读取图片纹理，在使用前，需要进行初始化

~~~
#define STB_IMAGE_IMPLEMENTATION
#include "stb_image.h"
~~~

可以用stbi_load函数将纹理数据存入data中

~~~
// Basic usage
//    int x,y,n;
//    unsigned char *data = stbi_load(filename, &x, &y, &n, 0);
//    // ... process data if not NULL ...
//    // ... x = width, y = height, n = # 8-bit components per pixel ...
//    // ... replace '0' with '1'..'4' to force that many components per pixel
//    // ... but 'n' will always be the number that it would have been if you said 0
~~~

x，y是图片的宽，高

---

### 投影函数

使用球的顶点来求球面的纹理坐标，示意图如下

![](https://pic.downk.cc/item/5e3111562fb38b8c3cde3bf4.jpg)

对交点P(x,y,z)和半径r，映射成球坐标的θ和ϕ。有以下公式

**x=r*cosθcosϕ**

**y=r*sinθ**

**z=r*cosθsinϕ**

设 **r=1**，则

~~~
float theta = asin(p.y());
~~~

~~~
float phi = atan2(p.z(), p.x());
~~~

接下来我们要将ϕ，θ映射到(0,1)内

纹理坐标的u对应ϕ，ϕ的范围是从[-π,π]，映射到[0,1]之间，即：

~~~
u = float(1 - (phi + M_PI) / (2 * M_PI));
~~~

上面除了个1是颠倒一下，以让球面图片向上。

v值对应θ，θ的范围是从[-π/2,π/2],映射到[0,1]之间，即：

~~~
v = float((theta + M_PI / 2) / M_PI);
~~~

由上整理成一个根据交点p求uv的函数为：

~~~
void get_sphere_uv(const vec3& p, float& u, float& v)
{
    float phi = atan2(p.z(), p.x());
    float theta = asin(p.y());
    u = float(1 - (phi + M_PI) / (2 * M_PI));
    v = float((theta + M_PI / 2) / M_PI);
}
~~~

**对于简单的几何形状，如球形、圆柱投影函数是可以用数学推导的，在常规情况下，投影函数通常在美术建模阶段使用，并将投影结果存储于顶点数据中。也就是说，在软件开发过程中，我们一般不会去用投影函数去计算得到投影结果，而是直接使用在美术建模过程中，已经存储在模型顶点数据中的投影结果 — Real-Time Rendering 3rd**

---

### 图片纹理类

~~~
#ifndef IMAGETEXTURE
#define IMAGETEXTURE

#define STB_IMAGE_IMPLEMENTATION
#include "stb_image.h"
#include "texture.h"

class image_texture : public texture {
    public:
        image_texture() {}
        image_texture(unsigned char *pixels, int A, int B)
            : data(pixels), nx(A), ny(B) {}
        virtual vec3 value(float u, float v, const vec3& p) const;
        unsigned char *data; //纹理数据
        int nx, ny; //纹理的长，宽
};

//提取data数据中指定位置的rgb值
vec3 image_texture::value(float u, float v, const vec3& p) const {
     int i = (u) * nx;
     int j = (1-v) * ny - 0.001;
     if (i < 0) i = 0;
     if (j < 0) j = 0;
     if (i > nx-1) i = nx-1;
     if (j > ny-1) j = ny-1;
     float r = int(data[3*i + 3*nx*j]  ) / 255.0;
     float g = int(data[3*i + 3*nx*j+1]) / 255.0;
     float b = int(data[3*i + 3*nx*j+2]) / 255.0;
     return vec3(r, g, b);
}
#endif
~~~

---

修改hittable.h

~~~
void get_sphere_uv(const vec3& p, float& u, float& v) {
    float phi = atan2(p.z(), p.x());
    float theta = asin(p.y());
    u = 1-(phi + M_PI) / (2*M_PI);
    v = (theta + M_PI/2) / M_PI;
}

struct hit_record {
    float t;  
    float u;
    float v;
    vec3 p;
    vec3 normal; 
    material *mat_ptr;
};
~~~

sphere.h

get_sphere_uv获取撞击点的坐标

~~~
bool sphere::hit(const ray& r, float t_min, float t_max, hit_record& rec) const {
    vec3 oc = r.origin() - center;
    float a = dot(r.direction(), r.direction());
    float b = dot(oc, r.direction());
    float c = dot(oc, oc) - radius*radius;
    float discriminant = b*b - a*c;
    if (discriminant > 0) {
        
        float temp = (-b - sqrt(discriminant))/a;
        if (temp < t_max && temp > t_min) {
            rec.t = temp;
            rec.p = r.point_at_parameter(rec.t);
            get_sphere_uv((rec.p-center)/radius, rec.u, rec.v);
            rec.normal = (rec.p - center) / radius;
            rec.mat_ptr = mat_ptr; 
            return true;
        }
        
         temp = (-b + sqrt(discriminant)) / a;
        if (temp < t_max && temp > t_min) {
            rec.t = temp;
            rec.p = r.point_at_parameter(rec.t);
            get_sphere_uv((rec.p-center)/radius, rec.u, rec.v);
            rec.normal = (rec.p - center) / radius;
            rec.mat_ptr = mat_ptr; 
            return true;
        }
    }
    return false;
}
~~~

material.h

反射求取图片对应出的像素值

~~~
class lambertian : public material {
    public:
        lambertian(texture *a) : albedo(a) {}
        virtual bool scatter(const ray& r_in, const hit_record& rec, vec3& attenuation, ray& scattered) const  {
             vec3 target = rec.p + rec.normal + random_in_unit_sphere();
             scattered = ray(rec.p, target-rec.p, r_in.time());
             attenuation = albedo->value(rec.u, rec.v, rec.p);
             return true;
        }
        texture *albedo;
};
~~~

main.cpp

读取并生成纹理

~~~
hittable* texture_spheres(){
    int nx, ny, nn;
    unsigned char *tex_data = stbi_load("earthmap.jpg", &nx, &ny, &nn, 0);
    material *mat = new lambertian(new image_texture(tex_data, nx, ny));
    int n = 50;
    hittable **list = new hittable*[n+1];
    list[0] = new sphere(vec3(0,0, 0), 1, mat);
    return new hittable_list(list,1);
}
~~~

---

最终渲染的图片如下

![](https://pic.downk.cc/item/5e2e71af2fb38b8c3ca808d3.png)

---
layout: post
title: Ray Tracing in One Weekend Summary XIV - Rectangles & Lights & Translation
date: 2020-01-27

---

# 矩形 光照 变换
 
---

矩形和光照。如何做一个自发光的材质，首先须要在hit_record里面加一个 emitted的方法。比如说背景如果是纯黑的话，就相当于光线来了的时候，他不反射任何光线。

### 自发光

在material类中添加虚函数emitted，默认返回黑色
 
~~~
virtual vec3 emitted(float u, float v, const vec3& p) const {
    return vec3(0,0,0);
}
~~~

diffuse_light类

~~~
class diffuse_light : public material {
    public:
        diffuse_light(texture *a) : emit(a) {}
        virtual bool scatter(const ray& r_in, const hit_record& rec,
            vec3& attenuation, ray& scattered) const { return false; }
        virtual vec3 emitted(float u, float v, const vec3& p) const {
            return emit->value(u, v, p);
        }
        texture *emit;
};
~~~

color函数

~~~
vec3 color(const ray& r, hittable *world, int depth) {
    hit_record rec;
    if (world->hit(r, 0.001, MAXFLOAT, rec)) {
        ray scattered;
        vec3 attenuation;
        vec3 emitted = rec.mat_ptr->emitted(rec.u, rec.v, rec.p);
        if (depth < 50 && rec.mat_ptr->scatter(r, rec, attenuation, scattered))
             return emitted + attenuation*color(scattered, world, depth+1);
        else
            return emitted;
    }
    else
        return vec3(0,0,0);
}
~~~

### 矩形


以xy平面上的矩形为例，在 **z=k**时，**x0**，**x1**，**y0**，**y1**可以定义一个矩形平面区域

![](https://raytracing.github.io/images/fig-2-7-01.jpg)

计算光线和矩形相交的方程如下

**p(t)=a+t*b**

**z(t)=az+t*bz**

**z=k**

解得：

**t=(k−az)/bz**

**x=ax+t∗bx**
 
**y=ay+t∗by**

如果同时x在区间[x0,x1]，y在[y0,y1]上，光线就和矩形相交

### xyz_rec.h

~~~
#ifndef XYZRECT
#define XYZRECT 

#include "hittable.h"

class xy_rect:public hittable
{
    public:
        xy_rect();
        xy_rect(float _x0, float _x1, float _y0, float _y1, float _k, material *mat)
                : x0(_x0), x1(_x1), y0(_y0), y1(_y1), k(_k), mp(mat) {};
        ~xy_rect();
        virtual bool hit(const ray& r,float t_min,float t_max,hit_record& rec)const;
        virtual bool bounding_box(float t_min,float t_max,aabb& box)const;

        float x0,x1,y0,y1,k;
        material *mp;
};

class xz_rect: public hittable {
    public:
        xz_rect() {}
        xz_rect(float _x0, float _x1, float _z0, float _z1, float _k, material *mat)
            : x0(_x0), x1(_x1), z0(_z0), z1(_z1), k(_k), mp(mat) {};
        virtual bool hit(const ray& r, float t_min, float t_max, hit_record& rec) const;
        virtual bool bounding_box(float t_min, float t_max, aabb& box) const;
        
        material *mp;
        float x0, x1, z0, z1, k;
};

class yz_rect: public hittable {
    public:
        yz_rect() {}
        yz_rect(float _y0, float _y1, float _z0, float _z1, float _k, material *mat)
            : y0(_y0), y1(_y1), z0(_z0), z1(_z1), k(_k), mp(mat) {};
        virtual bool hit(const ray& r, float t_min, float t_max, hit_record& rec) const;
        virtual bool bounding_box(float t_min, float t_max, aabb& box) const;

        material  *mp;
        float y0, y1, z0, z1, k;
};
bool xy_rect::hit(const ray& r, float t_min, float t_max, hit_record& rec) const {
    float t = (k-r.origin().z()) / r.direction().z();
    if (t < t_min || t > t_max)
        return false;
    float x = r.origin().x() + t*r.direction().x();
    float y = r.origin().y() + t*r.direction().y();
    if (x < x0 || x > x1 || y < y0 || y > y1)
        return false;
    rec.u = (x-x0)/(x1-x0);
    rec.v = (y-y0)/(y1-y0);
    rec.t = t;
    rec.mat_ptr = mp;
    rec.p = r.point_at_parameter(t);
    rec.normal = vec3(0, 0, 1);
    return true;
}
bool xy_rect::bounding_box(float t_min,float t_max,aabb& box)const{
    box =  aabb(vec3(x0,y0, k-0.0001), vec3(x1, y1, k+0.0001));
    return true;
}

bool xz_rect::hit(const ray& r, float t_min, float t_max, hit_record& rec) const {
    float t = (k-r.origin().y()) / r.direction().y();
    if (t < t_min || t > t_max)
        return false;
    float x = r.origin().x() + t*r.direction().x();
    float z = r.origin().z() + t*r.direction().z();
    if (x < x0 || x > x1 || z < z0 || z > z1)
        return false;
    rec.u = (x-x0)/(x1-x0);
    rec.v = (z-z0)/(z1-z0);
    rec.t = t;
    rec.mat_ptr = mp;
    rec.p = r.point_at_parameter(t);
    rec.normal = vec3(0, 1, 0);
    return true;
}

bool xz_rect::bounding_box(float t_min, float t_max, aabb& box) const {
    box =  aabb(vec3(x0,k-0.0001,z0), vec3(x1, k+0.0001, z1));
    return true;
}

bool yz_rect::hit(const ray& r, float t_min, float t_max, hit_record& rec) const {
    float t = (k-r.origin().x()) / r.direction().x();
    if (t < t_min || t > t_max)
        return false;
    float y = r.origin().y() + t*r.direction().y();
    float z = r.origin().z() + t*r.direction().z();
    if (y < y0 || y > y1 || z < z0 || z > z1)
        return false;
    rec.u = (y-y0)/(y1-y0);
    rec.v = (z-z0)/(z1-z0);
    rec.t = t;
    rec.mat_ptr = mp;
    rec.p = r.point_at_parameter(t);
    rec.normal = vec3(1, 0, 0);
    return true;
}

bool yz_rect::bounding_box(float t_min, float t_max, aabb& box) const {
    box =  aabb(vec3(k-0.0001, y0, z0), vec3(k+0.0001, y1, z1));
    return true;
}
#endif
~~~

类中有hit和bounding_box两个成员函数

###flip_normals.h

在前一步做完后，渲染出的cornell box有一个bug，我们所建立的矩形是有朝向的，光照射到矩形背面时只会渲染出黑色，因此，我们要对一些矩形进行法向量的翻转

flip_normals.h

~~~
class flip_normals : public hittable {
    public:
        flip_normals(hittable *p) : ptr(p) {}

        virtual bool hit(
            const ray& r, float t_min, float t_max, hit_record& rec) const {

            if (ptr->hit(r, t_min, t_max, rec)) {
                rec.normal = -rec.normal;
                return true;
            }
            else
                return false;
        }

        virtual bool bounding_box(float t0, float t1, aabb& box) const {
            return ptr->bounding_box(t0, t1, box);
        }

        hittable *ptr;
};
~~~

### test

接下来渲染一个cornell_box

~~~
hittable *cornell_box() {
    hittable **list = new hittable*[50];
    int i = 0;
    material *red = new lambertian(new constant_texture(vec3(0.65, 0.05, 0.05)));
    material *white = new lambertian(new constant_texture(vec3(0.73, 0.73, 0.73)));
    material *green = new lambertian(new constant_texture(vec3(0.12, 0.45, 0.15)));
    material *light = new diffuse_light(new constant_texture(vec3(15, 15, 15)));
    material *spec = new metal(vec3(1,1,1),0);

    list[i++] = new flip_normals(new yz_rect(0, 555, 0, 555, 555, green));
    list[i++] = new yz_rect(0, 555, 0, 555, 0, red);
    list[i++] = new xz_rect(213, 343, 227, 332, 554, light);
    list[i++] = new flip_normals(new xz_rect(0, 555, 0, 555, 555, white));
    list[i++] = new xz_rect(0, 555, 0, 555, 0, white);
    list[i++] = new flip_normals(new xy_rect(0, 555, 0, 555, 555, white));

    list[i++] = new sphere(vec3(150, 150, 350), 100, spec);
    list[i++] = new sphere(vec3(350, 100, 500), 100, green);

    return new hittable_list(list,i);
}
~~~

~~~
vec3 lookfrom(278, 278, -800);
vec3 lookat(278,278,0);
float dist_to_focus = 10.0;
float aperture = 0.0;
float vfov = 40.0;
camera cam(lookfrom, lookat, vec3(0,1,0), vfov, float(nx)/float(ny),aperture, dist_to_focus, 0.0, 1.0);
~~~

渲染结果如下

![](https://pic.downk.cc/item/5e31999e2fb38b8c3ce9b208.png)


### 长方体

长方体类的定义只需要定义8个相邻的矩形即可

~~~
class box: public hittable {
    public:
        box() {}
        box(const vec3& p0, const vec3& p1, material *ptr);
        virtual bool hit(const ray& r, float t0, float t1, hit_record& rec) const;
        virtual bool bounding_box(float t0, float t1, aabb& box) const {
            box =  aabb(pmin, pmax);
            return true;
        }
        vec3 pmin, pmax;
        hittable *list_ptr;
};

box::box(const vec3& p0, const vec3& p1, material *ptr) {
    pmin = p0;
    pmax = p1;
    hittable **list = new hittable*[6];
    list[0] = new xy_rect(p0.x(), p1.x(), p0.y(), p1.y(), p1.z(), ptr);
    list[1] = new flip_normals(
        new xy_rect(p0.x(), p1.x(), p0.y(), p1.y(), p0.z(), ptr));
    list[2] = new xz_rect(p0.x(), p1.x(), p0.z(), p1.z(), p1.y(), ptr);
    list[3] = new flip_normals(
        new xz_rect(p0.x(), p1.x(), p0.z(), p1.z(), p0.y(), ptr));
    list[4] = new yz_rect(p0.y(), p1.y(), p0.z(), p1.z(), p1.x(), ptr);
    list[5] = new flip_normals(
        new yz_rect(p0.y(), p1.y(), p0.z(), p1.z(), p0.x(), ptr));
    list_ptr = new hittable_list(list,6);
}

bool box::hit(const ray& r, float t0, float t1, hit_record& rec) const {
    return list_ptr->hit(r, t0, t1, rec);
}
~~~

### 变换

#### 平移

translate.h

~~~
#ifndef TRANSLATE
#define TRANSLATE 

class translate : public hittable {
    public:
        translate(hittable *p, const vec3& displacement) : ptr(p), offset(displacement) {}
        virtual bool hit(const ray& r, float t_min, float t_max, hit_record& rec) const;
        virtual bool bounding_box(float t0, float t1, aabb& box) const;
        hittable *ptr;
        vec3 offset;
};

bool translate::hit(const ray& r, float t_min, float t_max, hit_record& rec) const {
    ray moved_r(r.origin() - offset, r.direction(), r.time());//平移光线
    if (ptr->hit(moved_r, t_min, t_max, rec)) {
        //平移交点
        rec.p += offset;
        return true;
    }
    else
        return false;
}

bool translate::bounding_box(float t0, float t1, aabb& box) const {
    if (ptr->bounding_box(t0, t1, box)) {
        box = aabb(box.min() + offset, box.max() + offset);
        return true;
    }
    else
        return false;
}
#endif
~~~

#### 旋转

物体绕z轴旋转的示意图如下

![](https://pic1.zhimg.com/80/v2-72fae3a355859660cdf42881b335d30c_hd.jpg)

旋转角度为θ

~~~
x'=cos(θ)*x-sin(θ)*y
y'=sin(θ)*x+cos(θ)*y
~~~

rotate.h

~~~
#ifndef ROTATE
#define ROTATE 

class rotate_y : public hittable {
    public:
        rotate_y(hittable *p, float angle);
        virtual bool hit(const ray& r, float t_min, float t_max, hit_record& rec) const;
        virtual bool bounding_box(float t0, float t1, aabb& box) const {
            box = bbox; return hasbox;
        }
        hittable *ptr;
        float sin_theta;
        float cos_theta;
        bool hasbox;
        aabb bbox;
};

rotate_y::rotate_y(hittable *p, float angle) : ptr(p) {
    float radians = (M_PI / 180.) * angle;
    sin_theta = sin(radians);
    cos_theta = cos(radians);
    hasbox = ptr->bounding_box(0, 1, bbox);
    vec3 min(FLT_MAX, FLT_MAX, FLT_MAX);
    vec3 max(-FLT_MAX, -FLT_MAX, -FLT_MAX);
    for (int i = 0; i < 2; i++) {
        for (int j = 0; j < 2; j++) {
            for (int k = 0; k < 2; k++) {
                float x = i*bbox.max().x() + (1-i)*bbox.min().x();
                float y = j*bbox.max().y() + (1-j)*bbox.min().y();
                float z = k*bbox.max().z() + (1-k)*bbox.min().z();
                float newx = cos_theta*x + sin_theta*z;
                float newz = -sin_theta*x + cos_theta*z;
                vec3 tester(newx, y, newz);
                //重新计算bounding_box
                for ( int c = 0; c < 3; c++ )
                {
                    if ( tester[c] > max[c] )
                        max[c] = tester[c];
                    if ( tester[c] < min[c] )
                        min[c] = tester[c];
                }
            }
        }
    }
    bbox = aabb(min, max);
}

//交点，法线，入射光线也做同样的旋转
bool rotate_y::hit(const ray& r, float t_min, float t_max, hit_record& rec) const {
    vec3 origin = r.origin();
    vec3 direction = r.direction();
    origin[0] = cos_theta*r.origin()[0] - sin_theta*r.origin()[2];
    origin[2] =  sin_theta*r.origin()[0] + cos_theta*r.origin()[2];
    direction[0] = cos_theta*r.direction()[0] - sin_theta*r.direction()[2];
    direction[2] = sin_theta*r.direction()[0] + cos_theta*r.direction()[2];
    ray rotated_r(origin, direction, r.time());
    if (ptr->hit(rotated_r, t_min, t_max, rec)) {
        vec3 p = rec.p;
        vec3 normal = rec.normal;
        p[0] = cos_theta*rec.p[0] + sin_theta*rec.p[2];
        p[2] = -sin_theta*rec.p[0] + cos_theta*rec.p[2];
        normal[0] = cos_theta*rec.normal[0] + sin_theta*rec.normal[2];
        normal[2] = -sin_theta*rec.normal[0] + cos_theta*rec.normal[2];
        rec.p = p;
        rec.normal = normal;
        return true;
    }
    else
        return false;
}
#endif
~~~

---

### test

~~~
hittable *cornell_box() {
    hittable **list = new hittable*[50];
    int i = 0;
    material *red = new lambertian(new constant_texture(vec3(0.65, 0.05, 0.05)));
    material *white = new lambertian(new constant_texture(vec3(0.73, 0.73, 0.73)));
    material *green = new lambertian(new constant_texture(vec3(0.12, 0.45, 0.15)));
    material *light = new diffuse_light(new constant_texture(vec3(15, 15, 15)));
    material *spec = new metal(vec3(1,1,1),0);

    list[i++] = new flip_normals(new yz_rect(0, 555, 0, 555, 555, green));
    list[i++] = new yz_rect(0, 555, 0, 555, 0, red);
    list[i++] = new xz_rect(213, 343, 227, 332, 554, light);
    list[i++] = new flip_normals(new xz_rect(0, 555, 0, 555, 555, white));
    list[i++] = new xz_rect(0, 555, 0, 555, 0, white);
    list[i++] = new flip_normals(new xy_rect(0, 555, 0, 555, 555, white));

    list[i++] = new translate(new rotate_y(new box(vec3(0,0,0), vec3(165,165,165), white), -18),vec3(130,0,65));
    list[i++] = new translate(new rotate_y(new box(vec3(0,0,0), vec3(165,330,165), white), 15),vec3(265,0,295));

    return new hittable_list(list,i);
}
~~~

渲染出的图像如下

![](https://pic.downk.cc/item/5e31ad492fb38b8c3ceb61cf.png)

---
layout: post
title: Ray Tracing in One Weekend Summary XV - Volumes
date: 2020-01-29

---

# 烟雾

光照射到volume上时，既有可能直接穿过，又可能发生反射，折射。光线在烟雾体中能传播多远，是由volume的密度决定的，密度越高，光线穿透性越差，光线传播的距离也越短。

我们在volume的内部添加一些随机方向的表面来实现这种效果

当光线通过体积时，它可能在任何点散射。 光线在任何小距离dL中散射的概率为：

概率 **p=C*dL**，其中C与volume密度成正比。

---

iostropic.h

~~~
class isotropic : public material {
    public:
        isotropic(texture *a) : albedo(a) {}
        virtual bool scatter(
            const ray& r_in,
            const hit_record& rec,
            vec3& attenuation,
            ray& scattered) const {

            scattered = ray(rec.p, random_in_unit_sphere());
            attenuation = albedo->value(rec.u, rec.v, rec.p);
            return true;
        }
        texture *albedo;
};
~~~

在isotropic.h中，volume中的散射方向是随机方向，与漫反射不同的是，漫反射的散射光线不可能指到物体内部，它一定是散射到表面外，isotropic材质的散射光线可以沿原来的方向一往前，以此实现透光性

---

constant_medium.h

~~~
class constant_medium : public hittable {
    public:
        constant_medium(hittable *b, float d, texture *a) : boundary(b), density(d) {
            phase_function = new isotropic(a);
        }
        virtual bool hit(
            const ray& r, float t_min, float t_max, hit_record& rec) const;
        virtual bool bounding_box(float t0, float t1, aabb& box) const {
            return boundary->bounding_box(t0, t1, box);
        }
        hittable *boundary;
        float density;
        material *phase_function;
};

bool constant_medium::hit(const ray& r, float t_min, float t_max, hit_record& rec)
const {
    const bool enableDebug = false;//用于debug
    bool debugging = enableDebug && random_double() < 0.00001;

    hit_record rec1, rec2;

    if (boundary->hit(r, -FLT_MAX, FLT_MAX, rec1)) {
        if (boundary->hit(r, rec1.t+0.0001, FLT_MAX, rec2)) {

            if (debugging) std::cerr << "\nt0 t1 " << rec1.t << " " << rec2.t << '\n';

            if (rec1.t < t_min)
                rec1.t = t_min;

            if (rec2.t > t_max)
                rec2.t = t_max;

            if (rec1.t >= rec2.t)
                return false;

            if (rec1.t < 0)
                rec1.t = 0;

            float distance_inside_boundary = (rec2.t - rec1.t)*r.direction().length();
            float hit_distance = -(1/density) * log(random_double());

            if (hit_distance < distance_inside_boundary) {

                rec.t = rec1.t + hit_distance / r.direction().length();
                rec.p = r.point_at_parameter(rec.t);

                if (debugging) {
                    std::cerr << "hit_distance = " <<  hit_distance << '\n'
                              << "rec.t = " <<  rec.t << '\n'
                              << "rec.p = " <<  rec.p << '\n';
                }

                rec.normal = vec3(1,0,0);  // arbitrary
                rec.mat_ptr = phase_function;
                return true;
            }
        }
    }
    return false;
}
~~~

hit函数里面是一些边界合法性检测

### test

~~~
hittable *cornell_smoke() {
    hittable **list = new hittable*[8];
    int i = 0;
    material *red = new lambertian(new constant_texture(vec3(0.65, 0.05, 0.05)));
    material *white = new lambertian(new constant_texture(vec3(0.73, 0.73, 0.73)));
    material *green = new lambertian(new constant_texture(vec3(0.12, 0.45, 0.15)));
    material *light = new diffuse_light(new constant_texture(vec3(15, 15, 15)));

    list[i++] = new flip_normals(new yz_rect(0, 555, 0, 555, 555, green));
    list[i++] = new yz_rect(0, 555, 0, 555, 0, red);
    list[i++] = new xz_rect(213, 343, 227, 332, 554, light);
    list[i++] = new flip_normals(new xz_rect(0, 555, 0, 555, 555, white));
    list[i++] = new xz_rect(0, 555, 0, 555, 0, white);
    list[i++] = new flip_normals(new xy_rect(0, 555, 0, 555, 555, white));

    hittable *b1 = new translate(new rotate_y(new box(vec3(0, 0, 0), vec3(165, 165, 165), white), -18),vec3(130,0,65));
    hittable *b2 = new translate(new rotate_y(new box(vec3(0, 0, 0), vec3(165, 330, 165), white),  15),vec3(265,0,295));

    list[i++] = new constant_medium(b1, 0.01, new constant_texture(vec3(1.0, 1.0, 1.0)));
    list[i++] = new constant_medium(b2, 0.01, new constant_texture(vec3(0.0, 0.0, 0.0)));
        
    return new hittable_list(list,i);
}
~~~

渲染结果如下

![](https://pic.downk.cc/item/5e3291772fb38b8c3cfd8743.png)

---
layout: post
title: Ray Tracing in One Weekend Summary XVI - Testing all features
date: 2020-01-30

---

#封面图的渲染



~~~
hittable *final() {
    int nb = 20;
    hittable **list = new hittable*[30];
    hittable **boxlist = new hittable*[10000];
    hittable **boxlist2 = new hittable*[10000];
    material *white = new lambertian( new constant_texture(vec3(0.73, 0.73, 0.73)));
    material *ground = new lambertian( new constant_texture(vec3(0.48, 0.83, 0.53)));
    int b = 0;
    for (int i = 0; i < nb; i++) {
        for (int j = 0; j < nb; j++) {
            float w = 100;
            float x0 = -1000 + i*w;
            float z0 = -1000 + j*w;
            float y0 = 0;
            float x1 = x0 + w;
            float y1 = 100*(random_double()+0.01);
            float z1 = z0 + w;
            boxlist[b++] = new box(vec3(x0,y0,z0), vec3(x1,y1,z1), ground);
        }
    }
    int l = 0;
    list[l++] = new bvh_node(boxlist, b, 0, 1);
    material *light = new diffuse_light( new constant_texture(vec3(7, 7, 7)));
    list[l++] = new xz_rect(123, 423, 147, 412, 554, light);
    vec3 center(400, 400, 200);
    list[l++] = new moving_sphere(center, center+vec3(30, 0, 0),
        0, 1, 50, new lambertian(new constant_texture(vec3(0.7, 0.3, 0.1))));
    list[l++] = new sphere(vec3(260, 150, 45), 50, new dielectric(1.5));
    list[l++] = new sphere(vec3(0, 150, 145), 50,
        new metal(vec3(0.8, 0.8, 0.9), 10.0));
    hittable *boundary = new sphere(vec3(360, 150, 145), 70, new dielectric(1.5));
    list[l++] = boundary;
    list[l++] = new constant_medium(boundary, 0.2,
        new constant_texture(vec3(0.2, 0.4, 0.9)));
    boundary = new sphere(vec3(0, 0, 0), 5000, new dielectric(1.5));
    list[l++] = new constant_medium(boundary, 0.0001,
        new constant_texture(vec3(1.0, 1.0, 1.0)));
    int nx, ny, nn;
    unsigned char *tex_data = stbi_load("earthmap.jpg", &nx, &ny, &nn, 0);
    material *emat =  new lambertian(new image_texture(tex_data, nx, ny));
    list[l++] = new sphere(vec3(400, 200, 400), 100, emat);
    texture *pertext = new noise_texture(0.1);
    list[l++] =  new sphere(vec3(220, 280, 300), 80, new lambertian( pertext ));
    int ns = 1000;
    for (int j = 0; j < ns; j++) {
        boxlist2[j] = new sphere(
            vec3(165*random_double(), 165*random_double(), 165*random_double()),
            10, white);
    }
    list[l++] = new translate(new rotate_y(
        new bvh_node(boxlist2, ns, 0.0, 1.0), 15), vec3(-100,270,395));
    return new hittable_list(list,l);
}
~~~

分辨率600x600

sample:500

---

渲染结果如下

![](https://pic.downk.cc/item/5e32cbe82fb38b8c3c03af36.png)




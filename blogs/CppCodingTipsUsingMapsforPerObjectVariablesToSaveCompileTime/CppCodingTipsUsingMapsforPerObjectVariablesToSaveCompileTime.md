# C++ Coding Tips: Using Maps for Per-Object Variables To Save Compile Time

## The Problem

When working on large C++ projects, compilation time can become a significant bottleneck in the development process. When you need to add a new variable to a class, the typical approach is to declare the variable in the class definition within the header file. However, this approach has a drawback. Modifying the header file triggers recompilation of all the source files that include that header, even if they don't use the newly added variable. This can significantly increase the overall compilation time, especially in large projects with numerous source files.. so if you want to quickly test something and don't want to waste too time on code compiling, you can try the method below.

## The Solution

To solve this issue, we can leverage the power of maps. Instead of adding the variable directly to the class definition in the header file, we can create a map that associates the object's address with the corresponding variable value. This map can be defined and managed within the source file.

## Code Example

For an example class A

```c++
// A.h
class A {
public:
    A();
    ~A();
    void foo();
};
```

In the source file, declare a map that maps the object's address to the variable value. For example:

```c++
std::unordered_map<A*, int> variables_map;
```
When creating an object of the class, insert an entry into the map with the object's address as the key and the initial value of the variable.

```c++
A::A() {
    // variable can be initilized in ctor or other places
    variables_map[this] = rand();
}
```

To access or modify the variable value for a specific object, use the object's address as the key to retrieve or update the corresponding value in the map.

```c++
int value = variables_map[obj];
variables_map[obj] = new_value;
```

**IMPORTANT IMPORTANT IMPORTANT**
When the object is destroyed, remove the corresponding entry from the map to avoid memory leaks. For example:

```c++
A::~A() {
    // don't forget to remove object from map when dtor/delete called
    variables_map.erase(this);
}
```

By using this approach, you can effectively add per-object variables without modifying the header file. The map serves as a storage mechanism for the variable values, associating them with the respective objects. This technique helps reduce compilation time, as changes to the source file do not require recompilation of other files that include the header.

[the complete code](https://godbolt.org/z/rz7MxWz6j)

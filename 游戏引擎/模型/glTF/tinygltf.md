添加`std_image.h`，`std_image_write.h`，`json.hpp`，`tiny_gltf.h`；

```c++
// Define these only in *one* .cc file.
#define TINYGLTF_IMPLEMENTATION
#define STB_IMAGE_IMPLEMENTATION
#define STB_IMAGE_WRITE_IMPLEMENTATION
// #define TINYGLTF_NOEXCEPTION // optional. disable exception handling.
#include "tiny_gltf.h"

using namespace tinygltf;

Model model;
TinyGLTF loader;
std::string err;
std::string warn;

bool ret = loader.LoadASCIIFromFile(&model, &err, &warn, argv[1]);
//bool ret = loader.LoadBinaryFromFile(&model, &err, &warn, argv[1]); // for binary glTF(.glb)

if (!warn.empty()) {
  printf("Warn: %s\n", warn.c_str());
}

if (!err.empty()) {
  printf("Err: %s\n", err.c_str());
}

if (!ret) {
  printf("Failed to parse glTF\n");
  return -1;
}
```


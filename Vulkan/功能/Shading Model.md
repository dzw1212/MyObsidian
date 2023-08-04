
一般来说，着色计算主要在Fragment Shader中进行。这是因为Fragment Shader为每个像素运行一次，能够为每个像素提供更精确的光照和颜色计算。这对于实现复杂的光照模型，如Phong或者Physically-Based Rendering (PBR)，以及详细的纹理和材质效果非常重要。

相比之下，Vertex Shader主要负责处理顶点数据，包括顶点的位置、颜色、法线、纹理坐标等，并将这些数据传递给Fragment Shader。在某些简单的或者性能敏感的情况下，你可能会在Vertex Shader中进行一些基础的光照计算，但这通常会导致质量较低，因为Vertex Shader只为每个顶点运行一次，然后在这些顶点之间插值得到像素的值。


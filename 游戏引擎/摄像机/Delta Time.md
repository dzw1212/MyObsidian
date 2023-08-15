为了实现顺滑的摄像机移动并且独立于帧率，可以使用一个概念叫做"delta time"（Δ时间或者叫帧间隔时间）；Delta time是指上一帧到当前帧的时间间隔；
可以用这个时间间隔来调整摄像机的移动速度，使得摄像机的移动速度不受帧率的影响；

具体来说，你可以在每一帧中计算delta time，然后用这个delta time来乘以摄像机的移动速度；
这样，即使帧率变化，摄像机的移动速度也会保持一致；

```cpp
float speed = 10.0f; // 摄像机的移动速度
float deltaTime = 0.0f; // delta time
float lastFrame = 0.0f; // 上一帧的时间

void update() 
{
    float currentFrame = glfwGetTime();
    deltaTime = currentFrame - lastFrame;
    lastFrame = currentFrame;

    if (glfwGetKey(window, GLFW_KEY_W) == GLFW_PRESS)
        cameraPos += speed * deltaTime * cameraFront;
    if (glfwGetKey(window, GLFW_KEY_S) == GLFW_PRESS)
        cameraPos -= speed * deltaTime * cameraFront;
    if (glfwGetKey(window, GLFW_KEY_A) == GLFW_PRESS)
        cameraPos -= glm::normalize(glm::cross(cameraFront, cameraUp)) * speed * deltaTime;
    if (glfwGetKey(window, GLFW_KEY_D) == GLFW_PRESS)
        cameraPos += glm::normalize(glm::cross(cameraFront, cameraUp)) * speed * deltaTime;
}
```
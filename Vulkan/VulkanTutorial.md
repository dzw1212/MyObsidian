# 初始化Window

## GLFW实现

```cpp
void VulkanRenderer::initWindow()
{
    glfwInit();
    glfwWindowHint(GLFW_CLIENT_API, GLFW_NO_API);
    glfwWindowHint(GLFW_RESIZABLE, GLFW_TRUE);

    m_pWindow = glfwCreateWindow(m_uiWindowWidth, m_uiWindowHeight, m_strWindowTitle.c_str(), nullptr, nullptr);
    glfwMakeContextCurrent(m_pWindow);

    glfwSetWindowUserPointer(m_pWindow, this);
    glfwSetFramebufferSizeCallback(m_pWindow, frameBufferResizeCallBack);
}

void VulkanRenderer::frameBufferResizeCallBack(GLFWwindow* pWindow, int nWidth, int nHeight)
{
    auto app = reinterpret_cast<VulkanRenderer*>(glfwGetWindowUserPointer(pWindow));
    if (app)
        app->m_bFrameBufferResized = true;
}
```

# 初始化Vulkan

```cpp
void VulkanRenderer::initVulkan()
{
    setupCamera();
    createInstance();
    setupDebugMessenger();
    createWindowSurface();
    pickPhysicalDevice();
    createLogicalDevice();
    createSwapChain();
    createImageViews();
    createRenderPass();
    createDescriptorSetLayout();
    createGraphicsPipeline();
    createCommandPool();
    createColorResources();
    createDepthResources();
    createFrameBuffers();
    createTextureImage();
    createTextureImageView();
    createTextureSampler();
    loadModel(MODEL_LOAD_TYPE_TINY_OBJ_LOADER);
    createVertexBuffer();
    createIndexBuffer();
    createUniformBuffers();
    createDescriptorPool();
    createDescriptorSets();
    createCommandBuffers();
    createSyncObjects();
}
```

## 创建VK Instance

[[Vulkan/概念#vk Instance|vk Instance]]

### 获取GLFW所需的Extension

```cpp
void VulkanRenderer::getGLFWExtensions()
{
    m_glfwExtensions = glfwGetRequiredInstanceExtensions(&m_glfwExtensionCount);

    m_vecChosedExtensions = std::vector<const char*>(m_glfwExtensions, m_glfwExtensions + m_glfwExtensionCount);
}
```

### 获取Validation Layer所需的Extension（可选）

```cpp
void VulkanRenderer::getValidationLayerMessageCallbackExtension()
{
    m_vecChosedExtensions.push_back(VK_EXT_DEBUG_UTILS_EXTENSION_NAME);
}
```

### 获取所有可用的Extension

```cpp
bool VulkanRenderer::getAllValidExtensions()
{
    uint32_t uiExtensionCount = 0;
    vkEnumerateInstanceExtensionProperties(nullptr, &uiExtensionCount, nullptr);

    if (uiExtensionCount == 0)
        return false;

    m_vecExtensions.resize(uiExtensionCount);

    vkEnumerateInstanceExtensionProperties(nullptr, &uiExtensionCount, m_vecExtensions.data());

#ifdef SHOW_ALL_EXTENSION_INFO
    std::cout << "Available Extensions:" << std::endl;
    for (const auto& ext : m_vecExtensions)
    {
        std::cout << "\tName : " << ext.extensionName << std::endl;
        std::cout << "\t\tVersion : " << ext.specVersion << std::endl;
    }
#endif

    return true;
}
```

### 逐个对比判断所需的Extension是否可用

```cpp
bool VulkanRenderer::checkChosedExtensionValid()
{
    for (const auto& chosedExt : m_vecChosedExtensions)
    {
        bool bFound = false;
        for (const auto& extProperties : m_vecExtensions)
        {
            if (strcmp(extProperties.extensionName, chosedExt))
            {
                bFound = true;
                break;
            }
        }
        if (!bFound)
            return false;
    }
    return true;
}
```

### 获取所有可用的Validation Layer（可选）

```cpp
bool VulkanRenderer::getAllValidValidationLayers()
{
    uint32_t uiLayerCount = 0;
    vkEnumerateInstanceLayerProperties(&uiLayerCount, nullptr);

    if (uiLayerCount == 0)
        return false;

    m_vecValidationLayers.resize(uiLayerCount);
    vkEnumerateInstanceLayerProperties(&uiLayerCount, m_vecValidationLayers.data());

#ifdef SHOW_ALL_VALIDATION_LAYER_INFO
    std::cout << "Available Validation Layers:" << std::endl;
    for (const auto& layer : m_vecValidationLayers)
    {
        std::cout << "\tName : " << layer.layerName << std::endl;
        std::cout << "\t\tVersion : " << layer.specVersion << std::endl;
        std::cout << "\t\tImpVersion : " << layer.implementationVersion << std::endl;
        std::cout << "\t\tDescription : " << layer.description << std::endl;
    }
#endif

    return true;
}
```

### 逐个对比判断所需的Validation Layer是否可用（可选）

```cpp
const std::vector<const char*> m_vecChosedValidationLayers = {
	"VK_LAYER_KHRONOS_validation",
};

bool VulkanRenderer::checkChosedValidationLayerValid()
{
    for (const char* chosedLayer : m_vecChosedValidationLayers)
    {
        bool bFound = false;
        for (const auto& layerProperties : m_vecValidationLayers)
        {
            if (strcmp(layerProperties.layerName, chosedLayer))
            {
                bFound = true;
                break;
            }
        }

        if (!bFound)
            return false;
    }
    return true;
}
```

### 创建VkApplicationInfo

```cpp
VkApplicationInfo appInfo{};
appInfo.sType = VK_STRUCTURE_TYPE_APPLICATION_INFO;
appInfo.pApplicationName = "Hello Triangle";
appInfo.applicationVersion = VK_MAKE_VERSION(1, 0, 0);
appInfo.pEngineName = "No Engine";
appInfo.engineVersion = VK_MAKE_VERSION(1, 0, 0);
appInfo.apiVersion = VK_API_VERSION_1_0;
```

### 创建VkInstanceCreateInfo

```cpp
VkInstanceCreateInfo createInfo{};
createInfo.sType = VK_STRUCTURE_TYPE_INSTANCE_CREATE_INFO;
createInfo.pApplicationInfo = &appInfo;
//设置Extensions
createInfo.enabledExtensionCount = static_cast<uint32_t>(m_vecChosedExtensions.size());
createInfo.ppEnabledExtensionNames = m_vecChosedExtensions.data();
//设置Validation Layer
VkDebugUtilsMessengerCreateInfoEXT debugMessengerCreateInfo{};
if (g_bEnableValidationLayer)
{
	createInfo.enabledLayerCount = static_cast<int>(m_vecChosedValidationLayers.size());
	createInfo.ppEnabledLayerNames = m_vecChosedValidationLayers.data();
	populateDebugMessengerCreateInfo(debugMessengerCreateInfo);
	createInfo.pNext = &debugMessengerCreateInfo;
}
else
{
	createInfo.enabledLayerCount = 0;
	createInfo.pNext = nullptr;
}
```

```cpp
void VulkanRenderer::populateDebugMessengerCreateInfo(VkDebugUtilsMessengerCreateInfoEXT& messengerCreateInfo)
{
	messengerCreateInfo = {};
	messengerCreateInfo.sType =
		VK_STRUCTURE_TYPE_DEBUG_UTILS_MESSENGER_CREATE_INFO_EXT;
	messengerCreateInfo.messageSeverity =
		VK_DEBUG_UTILS_MESSAGE_SEVERITY_ERROR_BIT_EXT;/* |
		VK_DEBUG_UTILS_MESSAGE_SEVERITY_WARNING_BIT_EXT |
		VK_DEBUG_UTILS_MESSAGE_SEVERITY_INFO_BIT_EXT |
		VK_DEBUG_UTILS_MESSAGE_SEVERITY_VERBOSE_BIT_EXT;*/
	messengerCreateInfo.messageType =
		VK_DEBUG_UTILS_MESSAGE_TYPE_GENERAL_BIT_EXT |
		VK_DEBUG_UTILS_MESSAGE_TYPE_PERFORMANCE_BIT_EXT |
		VK_DEBUG_UTILS_MESSAGE_TYPE_VALIDATION_BIT_EXT;
	messengerCreateInfo.pfnUserCallback = debugCallBack;
}
```


### 创建vkInstance

```cpp
VkResult res = vkCreateInstance(&createInfo, nullptr, &m_Instance);
if (res != VK_SUCCESS)
	throw std::runtime_error("VK create instance failed");
```

## 设置DebugMessenger（可选）

```cpp
void VulkanRenderer::setupDebugMessenger()
{
    if (!g_bEnableValidationLayer)
        return;

    VkDebugUtilsMessengerCreateInfoEXT messengerCreateInfo{};
    populateDebugMessengerCreateInfo(messengerCreateInfo);

    //由于vkCreateDebugUtilsMessengerEXT属于Extension的函数，不能直接使用
    //因此需要先封装一层，使用vkGetInstanceProcAddr判断该函数是否可用
    VkResult res = CreateDebugUtilsMessengerEXT(m_Instance, &messengerCreateInfo, nullptr, &m_DebugMessenger);
    if (res != VK_SUCCESS)
        throw std::runtime_error("Setup Debug Messenger Failed");
}
```

```cpp
VkResult VulkanRenderer::CreateDebugUtilsMessengerEXT(
    VkInstance instance,
    const VkDebugUtilsMessengerCreateInfoEXT* pCreateInfo,
    const VkAllocationCallbacks* pAllocator,
    VkDebugUtilsMessengerEXT* pDebugMessenger)
{
    auto func = (PFN_vkCreateDebugUtilsMessengerEXT)vkGetInstanceProcAddr(
        instance, "vkCreateDebugUtilsMessengerEXT");
        
    if (func != nullptr)
        return func(instance, pCreateInfo, pAllocator, pDebugMessenger);
    else
        return VK_ERROR_EXTENSION_NOT_PRESENT;
}
```

## 创建Window Surface

```cpp
void VulkanRenderer::createWindowSurface()
{
    //手动创建
    //VkWin32SurfaceCreateInfoKHR createInfo{};
    //createInfo.sType = VK_STRUCTURE_TYPE_WIN32_SURFACE_CREATE_INFO_KHR;
    //createInfo.hwnd = glfwGetWin32Window(m_pWindow);
    //createInfo.hinstance = GetModuleHandle(nullptr);
    //VkResult res = vkCreateWin32SurfaceKHR(m_Instance, &createInfo, nullptr, &m_Surface);
    //if (res != VK_SUCCESS)
    //  throw std::runtime_error("Create Win32 Surface By User Failed");

    //使用glfw提供的接口创建
    VkResult res = glfwCreateWindowSurface(m_Instance, m_pWindow, nullptr, &m_Surface);
    if (res != VK_SUCCESS)
        throw std::runtime_error("Create Window Surface Failed");
}
```

## 选择物理设备

### 列出所有可用的物理设备

```cpp
vkEnumeratePhysicalDevices(m_Instance, &m_PhysicalDeviceCount, nullptr);
if (m_PhysicalDeviceCount == 0)
	throw std::runtime_error("Find Physical Device Failed");

m_vecValidPhysicalDevices.resize(m_PhysicalDeviceCount);
vkEnumeratePhysicalDevices(m_Instance, &m_PhysicalDeviceCount, m_vecValidPhysicalDevices.data());

#ifdef SHOW_ALL_PHYSICAL_DEVICE_INFO
    std::cout << "Physical Device Properties:" << std::endl;
    for (const auto& device : m_vecValidPhysicalDevices)
    {
        VkPhysicalDeviceProperties deviceProperties;
        vkGetPhysicalDeviceProperties(device, &deviceProperties);
        std::cout << "\tID : " << deviceProperties.deviceID << std::endl;
        std::cout << "\tName : " << deviceProperties.deviceName << std::endl;
        std::cout << "\tType : " << deviceProperties.deviceType << std::endl;
    }
#endif
```


### 选择最合适的物理设备或默认第一个

```cpp
if (g_bChooseBestPhysicalDevice)
{
	//选择最适合的显卡作为目标显卡
	std::multimap<int, VkPhysicalDevice> candidates;
	for (const auto& device : m_vecValidPhysicalDevices)
	{
		int nScore = ratePhysicalDeviceSuitability(device);
		candidates.insert(std::make_pair(nScore, device));
	}
	//multimap有序，分数最高的在末尾
	if (candidates.crbegin()->first > 0)
		m_PhysicalDevice = candidates.crbegin()->second;
}
else
{
	//选择第一个可用的显卡作为目标显卡
	for (const auto& device : m_vecValidPhysicalDevices)
	{
		if (isPhysicalDeviceSuitable(device))
		{
			m_PhysicalDevice = device;
			break;
		}
	}
}
```

```cpp
//给显卡评分
int VulkanRenderer::ratePhysicalDeviceSuitability(VkPhysicalDevice device)
{
    int nScore = 0;

    //Basic
    VkPhysicalDeviceProperties deviceProperties;
    vkGetPhysicalDeviceProperties(device, &deviceProperties);
    //Feature
    VkPhysicalDeviceFeatures deviceFeatures;
    vkGetPhysicalDeviceFeatures(device, &deviceFeatures);
    
    //检查是否是独立显卡
    if (deviceProperties.deviceType == VK_PHYSICAL_DEVICE_TYPE_DISCRETE_GPU)
        nScore += 1000;

    //检查支持的最大图像尺寸,越大越好
    nScore += deviceProperties.limits.maxImageDimension2D;

    //检查是否满足其他要求
    if (!isPhysicalDeviceSuitable(device))
        return 0;

    return nScore;
}
```

```cpp

```

## 创建逻辑设备

## 创建Swap Chain

## 创建Image View

## 创建Render Pass

# 主循环

# 清理Vulkan


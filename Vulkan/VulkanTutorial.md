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

### 设置App相关信息

```cpp
VkApplicationInfo appInfo{};
appInfo.sType = VK_STRUCTURE_TYPE_APPLICATION_INFO;
appInfo.pApplicationName = "Hello Triangle";
appInfo.applicationVersion = VK_MAKE_VERSION(1, 0, 0);
appInfo.pEngineName = "No Engine";
appInfo.engineVersion = VK_MAKE_VERSION(1, 0, 0);
appInfo.apiVersion = VK_API_VERSION_1_0;
```

### 填充VkInstanceCreateInfo

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

[[Vulkan/概念#Window Surface|Window Surface]]

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

## 创建逻辑设备

### 获取Graphic和Present队列族

```cpp
struct QueueFamilyIndices
{
	std::optional<uint32_t> graphicFamily;
	std::optional<uint32_t> presentFamily;

	bool isComplete()
	{
		return graphicFamily.has_value() && presentFamily.has_value();
	}
};
```

```cpp
VulkanRenderer::QueueFamilyIndices VulkanRenderer::findQueueFamilies(VkPhysicalDevice device)
{
    //使用二段式方法获取所用支持的Queue Family
    vkGetPhysicalDeviceQueueFamilyProperties(device, &m_QueueFamilyCount, nullptr);
    if (m_QueueFamilyCount == 0)
        throw std::runtime_error("Find Valid Queue Family Failed");

    m_vecQueueFamilies.resize(m_QueueFamilyCount);
    vkGetPhysicalDeviceQueueFamilyProperties(device, &m_QueueFamilyCount, m_vecQueueFamilies.data());
    
    QueueFamilyIndices indices;

    //获取Graphic Queue的Idx
    int nIdx = 0;
    for (const auto& queueFamily : m_vecQueueFamilies)
    {
        if (queueFamily.queueFlags & VK_QUEUE_GRAPHICS_BIT)
        {
            indices.graphicFamily = nIdx;
            break;
        }
        nIdx++;
    }

    //获取Present Queue的Idx
    nIdx = 0;
    VkBool32 bPresentSupport = false;
    vkGetPhysicalDeviceSurfaceSupportKHR(device, nIdx, m_Surface, &bPresentSupport);
    if (bPresentSupport)
        indices.presentFamily = nIdx;

    //graphicFamily和presentFamily通常是同一个QueueFamily
    return indices;
}
```

```cpp
QueueFamilyIndices indices = findQueueFamilies(m_PhysicalDevice);

//graphicFamily和presentFamily通常是同一个QueueFamily
//因此使用set避免不同的queueFamily的Idx相同的情况
std::set<uint32_t> setUniqueQueueFamily = {
	indices.graphicFamily.value(),
	indices.presentFamily.value(),
};
```

### 填充VkDeviceQueueCreateInfo

```cpp
float queuePriority = 1.f;
for (const auto queueFamily : setUniqueQueueFamily)
{
	//创建Queue的createInfo
	VkDeviceQueueCreateInfo queueCreateInfo{};
	queueCreateInfo.sType = VK_STRUCTURE_TYPE_DEVICE_QUEUE_CREATE_INFO;
	queueCreateInfo.queueFamilyIndex = indices.graphicFamily.value();
	queueCreateInfo.queueCount = 1;	//Queue的优先级，范围[0.f, 1.f]，控制CommandBuffer的执行顺序
	queueCreateInfo.pQueuePriorities = &queuePriority;
	vecQueueCreateInfo.push_back(queueCreateInfo);
}
```

### 设置硬件相关Feature

```cpp
VkPhysicalDeviceFeatures deviceFeatures{};
deviceFeatures.samplerAnisotropy = VK_TRUE; //启用各向异性过滤，用于纹理采样
deviceFeatures.sampleRateShading = VK_TRUE;	//启用Sample Rate Shaing，用于MSAA抗锯齿
```

### 填充VkDeviceCreateInfo

```cpp
VkDeviceCreateInfo createInfo{};
createInfo.sType = VK_STRUCTURE_TYPE_DEVICE_CREATE_INFO;
createInfo.queueCreateInfoCount = static_cast<int>(vecQueueCreateInfo.size());
createInfo.pQueueCreateInfos = vecQueueCreateInfo.data();
createInfo.pEnabledFeatures = &deviceFeatures;
createInfo.enabledExtensionCount = static_cast<int>(m_vecDeviceExtensions.size());
createInfo.ppEnabledExtensionNames = m_vecDeviceExtensions.data();

if (g_bEnableValidationLayer)
{
	createInfo.enabledLayerCount = static_cast<int>(m_vecChosedValidationLayers.size());
	createInfo.ppEnabledLayerNames = m_vecChosedValidationLayers.data();
}
else
	createInfo.enabledExtensionCount = 0;
```

### 创建VkDevice

```cpp
VkResult res = vkCreateDevice(m_PhysicalDevice, &createInfo, nullptr, &m_LogicalDevice);
if (res != VK_SUCCESS)
	throw std::runtime_error("Create Logical Device Failed");
```

## 创建Swap Chain

[[Vulkan/概念#Swap Chain|Swap Chain]]

### 获取硬件支持的Swap Chain信息

```cpp
struct SwapChainSupportDetails
{
	VkSurfaceCapabilitiesKHR capabilities;	//SwapChain容量相关信息
	std::vector<VkSurfaceFormatKHR> vecSurfaceFormats;	//支持哪些图像格式
	std::vector<VkPresentModeKHR> vecPresentModes;	//支持哪些表现模式，如二缓、三缓等
};
```

```cpp
VulkanRenderer::SwapChainSupportDetails VulkanRenderer::querySwapChainSupport(VkPhysicalDevice device)
{
	SwapChainSupportDetails details;

	//获取硬件支持的capability，包含Image数量上下限等信息
	vkGetPhysicalDeviceSurfaceCapabilitiesKHR(device, m_Surface, &details.capabilities);

	//获取硬件支持的Surface Format列表
	uint32_t uiFormatCount = 0;
	vkGetPhysicalDeviceSurfaceFormatsKHR(device, m_Surface, &uiFormatCount, nullptr);
	if (uiFormatCount > 0)
	{
		details.vecSurfaceFormats.resize(uiFormatCount);
		vkGetPhysicalDeviceSurfaceFormatsKHR(device, m_Surface, &uiFormatCount, details.vecSurfaceFormats.data());
	}

	//获取硬件支持的Present Mode列表
	uint32_t uiPresentModeCount = 0;
	vkGetPhysicalDeviceSurfacePresentModesKHR(device, m_Surface, &uiPresentModeCount, nullptr);
	if (uiPresentModeCount > 0)
	{
		details.vecPresentModes.resize(uiPresentModeCount);
		vkGetPhysicalDeviceSurfacePresentModesKHR(device, m_Surface, &uiPresentModeCount, details.vecPresentModes.data());
	}

	return details;
}
```

### 设置Swap Chain的格式

#### 图片格式

一般选择32位非线性sRGB；

```cpp
VkSurfaceFormatKHR VulkanRenderer::chooseSwapChainSurfaceFormat(const std::vector<VkSurfaceFormatKHR>& vecAvailableFormats)
{
	if (vecAvailableFormats.size() == 0)
		throw std::runtime_error("No Available Surface Format");

	//找到支持sRGB格式的format
	for (const auto& format : vecAvailableFormats)
	{
		if (format.format == VK_FORMAT_B8G8R8A8_SRGB
			&& format.colorSpace == VK_COLOR_SPACE_SRGB_NONLINEAR_KHR)
		{
			return format;
		}
	}

	//如果找不到，索性直接返回第一个
	return vecAvailableFormats[0];
}
```

#### 表现模式

游戏推荐MAILBOX（延迟低），电影推荐FIFO；

```cpp
VkPresentModeKHR VulkanRenderer::chooseSwapChainPresentMode(const std::vector<VkPresentModeKHR>& vecAvailableModes)
{
	if (vecAvailableModes.size() == 0)
		throw std::runtime_error("No Available Present Mode");

	for (const auto& mode : vecAvailableModes)
	{
		if (mode == VK_PRESENT_MODE_MAILBOX_KHR)
			return mode;
	}
	//如果不支持MAILBOX，就使用FIFO
	return VK_PRESENT_MODE_FIFO_KHR;
	//return VK_PRESENT_MODE_IMMEDIATE_KHR;
}
```

#### 图像尺寸

一般为窗口的大小；

```cpp
VkExtent2D VulkanRenderer::chooseSwapChainSwapExtent(const VkSurfaceCapabilitiesKHR& capabilities)
{
	//如果已经设置过，直接返回
	if (capabilities.currentExtent.width != std::numeric_limits<uint32_t>::max())
		return capabilities.currentExtent;
	else
	{
		int nWidth = 0;
		int nHeight = 0;
		glfwGetFramebufferSize(m_pWindow, &nWidth, &nHeight);

		VkExtent2D actualExtent = {
			static_cast<uint32_t>(nWidth),
			static_cast<uint32_t>(nHeight),
		};

		actualExtent.width = std::clamp(actualExtent.width,
			capabilities.minImageExtent.width,
			capabilities.maxImageExtent.width);
		actualExtent.height = std::clamp(actualExtent.height,
			capabilities.minImageExtent.height,
			capabilities.maxImageExtent.height);

		return actualExtent;
	}
}
```

#### 图片数量

一般为最小值+1；

```cpp
uint32_t uiImageCount = details.capabilities.minImageCount + 1;
if (details.capabilities.maxImageCount > 0)
{
	uiImageCount = std::clamp(uiImageCount,
		details.capabilities.minImageCount,
		details.capabilities.maxImageCount);
}
```

### 填充VkSwapchainCreateInfoKHR

```cpp
VkSwapchainCreateInfoKHR createInfo{};
createInfo.sType = VK_STRUCTURE_TYPE_SWAPCHAIN_CREATE_INFO_KHR;
createInfo.surface = m_Surface;
createInfo.minImageCount = uiImageCount;
createInfo.imageFormat = surfaceFormat.format;
createInfo.imageColorSpace = surfaceFormat.colorSpace;
createInfo.imageExtent = extent;
createInfo.imageArrayLayers = 1;
createInfo.imageUsage = VK_IMAGE_USAGE_COLOR_ATTACHMENT_BIT;
createInfo.presentMode = presentMode;

//设置graphicQueue和presentQueue
QueueFamilyIndices indices = findQueueFamilies(m_PhysicalDevice);
uint32_t queueFamilyIndices[] = {
	indices.graphicFamily.value(),
	indices.presentFamily.value()
};
if (indices.graphicFamily != indices.presentFamily)
{
	createInfo.imageSharingMode = VK_SHARING_MODE_CONCURRENT;
	createInfo.queueFamilyIndexCount = 2;
	createInfo.pQueueFamilyIndices = queueFamilyIndices;
}
else
{
	createInfo.imageSharingMode = VK_SHARING_MODE_EXCLUSIVE;
	createInfo.queueFamilyIndexCount = 0;
	createInfo.pQueueFamilyIndices = nullptr;
}

//设置如何进行Transform
createInfo.preTransform = details.capabilities.currentTransform; //不做任何Transform

//设置是否启用Alpha通道
createInfo.compositeAlpha = VK_COMPOSITE_ALPHA_OPAQUE_BIT_KHR; //不启用Alpha通道

//设置是否启用隐藏面剔除
createInfo.clipped = VK_TRUE;

//oldSwapChain用于window resize时
createInfo.oldSwapchain = VK_NULL_HANDLE;
```

### 创建VkSwapchainKHR

```cpp
VkResult res = vkCreateSwapchainKHR(m_LogicalDevice, &createInfo, nullptr, &m_SwapChain);
if (res != VK_SUCCESS)
	throw std::runtime_error("Create Swap Chain Failed");
```

## 创建Image View

[[Vulkan/概念#Image View|Image View]]

```cpp
//为每个Image创建ImageView
int nIdx = 0;
m_vecSwapChainImageViews.resize(m_vecSwapChainImages.size());
for (size_t i = 0; i < m_vecSwapChainImages.size(); ++i)
{
	m_vecSwapChainImageViews[i] = createImageView(m_vecSwapChainImages[i], m_SwapChainFormat, VK_IMAGE_ASPECT_COLOR_BIT, 1);
}
```

```cpp
VkImageView VulkanRenderer::createImageView(VkImage image, VkFormat format, 
	VkImageAspectFlags aspectFlags, uint32_t uiMipLevel)
{
	VkImageViewCreateInfo createInfo{};
	createInfo.sType = VK_STRUCTURE_TYPE_IMAGE_VIEW_CREATE_INFO;
	createInfo.image = image;
	createInfo.viewType = VK_IMAGE_VIEW_TYPE_2D;
	createInfo.format = format;
	createInfo.subresourceRange.aspectMask = aspectFlags;
	createInfo.subresourceRange.baseMipLevel = 0;
	createInfo.subresourceRange.levelCount = uiMipLevel;
	createInfo.subresourceRange.baseArrayLayer = 0;
	createInfo.subresourceRange.layerCount = 1;

	VkImageView imageView;

	VkResult res = vkCreateImageView(m_LogicalDevice, &createInfo, nullptr, &imageView);
	if (res != VK_SUCCESS)
		throw std::runtime_error("Create Texture Image View Failed");

	return imageView;
}
```

## 创建Render Pass

[[Vulkan/概念#Render Pass|Render Pass]]

### 设置Color Attachment Descriptor 及 Reference

```cpp
//设置Color Attachment Description, Reference
VkAttachmentDescription colorAttachment{};
colorAttachment.format = m_SwapChainFormat;
colorAttachment.samples = m_MSAASamples;
colorAttachment.loadOp = VK_ATTACHMENT_LOAD_OP_CLEAR;
colorAttachment.storeOp = VK_ATTACHMENT_STORE_OP_STORE;
colorAttachment.stencilLoadOp = VK_ATTACHMENT_LOAD_OP_DONT_CARE;
colorAttachment.stencilStoreOp = VK_ATTACHMENT_STORE_OP_DONT_CARE;
colorAttachment.initialLayout = VK_IMAGE_LAYOUT_UNDEFINED;
//如果直接显示，则为VK_IMAGE_LAYOUT_PRESENT_SRC_KHR 
//如果使用MSAA，则不能直接显示，需要改为VK_IMAGE_LAYOUT_COLOR_ATTACHMENT_OPTIMAL
colorAttachment.finalLayout = VK_IMAGE_LAYOUT_COLOR_ATTACHMENT_OPTIMAL;

VkAttachmentReference colorAttachmentRef{};
colorAttachmentRef.attachment = 0; //0表示第一个attachment，即上面定义的那个
colorAttachmentRef.layout = VK_IMAGE_LAYOUT_COLOR_ATTACHMENT_OPTIMAL;
```

### 设置Depth Attachment Descriptor 及 Reference

```cpp
//设置Depth Attachment Description, Reference
VkAttachmentDescription depthAttachment{};
depthAttachment.format = findDepthFormat();
depthAttachment.samples = m_MSAASamples;
depthAttachment.loadOp = VK_ATTACHMENT_LOAD_OP_CLEAR;
depthAttachment.storeOp = VK_ATTACHMENT_STORE_OP_DONT_CARE;
depthAttachment.stencilLoadOp = VK_ATTACHMENT_LOAD_OP_DONT_CARE;
depthAttachment.stencilStoreOp = VK_ATTACHMENT_STORE_OP_DONT_CARE;
depthAttachment.initialLayout = VK_IMAGE_LAYOUT_UNDEFINED;
depthAttachment.finalLayout = VK_IMAGE_LAYOUT_DEPTH_STENCIL_ATTACHMENT_OPTIMAL;

VkAttachmentReference depthAttachmentRef{};
depthAttachmentRef.attachment = 1;
depthAttachmentRef.layout = VK_IMAGE_LAYOUT_DEPTH_STENCIL_ATTACHMENT_OPTIMAL;
```

### 设置Subpass

```cpp
//设置Subpass
VkSubpassDescription subpass{};
subpass.pipelineBindPoint = VK_PIPELINE_BIND_POINT_GRAPHICS;
subpass.colorAttachmentCount = 1;
subpass.pColorAttachments = &colorAttachmentRef;
//一个Subpass只能指定一个depthAttachment
subpass.pDepthStencilAttachment = &depthAttachmentRef;
//指定MSAA之后的Color Attachment
subpass.pResolveAttachments = &colorAttachmentResolveRef;
```

### 设置Dependency

```cpp
//设置Dependency
VkSubpassDependency dependency{};
dependency.srcSubpass = VK_SUBPASS_EXTERNAL;
dependency.dstSubpass = 0;
dependency.srcStageMask = VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT
	| VK_PIPELINE_STAGE_EARLY_FRAGMENT_TESTS_BIT;
dependency.srcAccessMask = 0;
dependency.dstStageMask = VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT
	| VK_PIPELINE_STAGE_EARLY_FRAGMENT_TESTS_BIT;
dependency.dstAccessMask = VK_ACCESS_COLOR_ATTACHMENT_WRITE_BIT
	| VK_ACCESS_DEPTH_STENCIL_ATTACHMENT_WRITE_BIT;
```

### 填充VkRenderPassCreateInfo

```cpp
std::vector<VkAttachmentDescription>  vecAttachments = {
	colorAttachment,
	depthAttachment,
	colorAttachmentResolve,
};

VkRenderPassCreateInfo renderPassCreateInfo{};
renderPassCreateInfo.sType = VK_STRUCTURE_TYPE_RENDER_PASS_CREATE_INFO;
renderPassCreateInfo.attachmentCount = static_cast<uint32_t>(vecAttachments.size());
renderPassCreateInfo.pAttachments = vecAttachments.data();
renderPassCreateInfo.subpassCount = 1;
renderPassCreateInfo.pSubpasses = &subpass;
renderPassCreateInfo.dependencyCount = 1;
renderPassCreateInfo.pDependencies = &dependency;
```

### 创建VkRenderPass

```cpp
VkResult res = vkCreateRenderPass(m_LogicalDevice, &renderPassCreateInfo, nullptr, &m_RenderPass);
if (res != VK_SUCCESS)
	throw std::runtime_error("Create Render Pass Failed");
```

## 创建Descriptor Set Layout

[[Vulkan/概念#Descriptor|Descriptor]]

### 绑定UBO

```cpp
//UniformBufferObject Binding
VkDescriptorSetLayoutBinding uboLayoutBinding{};
uboLayoutBinding.binding = 0; //对应Vertex Shader中的layout(binding=0)
uboLayoutBinding.descriptorCount = 1;
uboLayoutBinding.descriptorType = VK_DESCRIPTOR_TYPE_UNIFORM_BUFFER;	//类型为Uniform Buffer
uboLayoutBinding.stageFlags = VK_SHADER_STAGE_VERTEX_BIT; //只需要在vertex stage生效
uboLayoutBinding.pImmutableSamplers = nullptr;
```

### 绑定图片采样器

```cpp
//CombinedImageSampler Binding
VkDescriptorSetLayoutBinding samplerLayoutBinding{};
samplerLayoutBinding.binding = 1;
samplerLayoutBinding.descriptorCount = 1;
samplerLayoutBinding.descriptorType = VK_DESCRIPTOR_TYPE_COMBINED_IMAGE_SAMPLER;
samplerLayoutBinding.stageFlags = VK_SHADER_STAGE_FRAGMENT_BIT; //只用于fragment stage
samplerLayoutBinding.pImmutableSamplers = nullptr;
```

### 填充VkDescriptorSetLayoutCreateInfo

```cpp
std::vector<VkDescriptorSetLayoutBinding> vecDescriptorLayoutBinding = {
	uboLayoutBinding,
	samplerLayoutBinding,
};

VkDescriptorSetLayoutCreateInfo createInfo{};
createInfo.sType = VK_STRUCTURE_TYPE_DESCRIPTOR_SET_LAYOUT_CREATE_INFO;
createInfo.bindingCount = static_cast<uint32_t>(vecDescriptorLayoutBinding.size());
createInfo.pBindings = vecDescriptorLayoutBinding.data();
```

### 创建VkDescriptorSetLayout

```cpp
VkResult res = vkCreateDescriptorSetLayout(m_LogicalDevice, &createInfo, nullptr, &m_DescriptorSetLayout);
if (res != VK_SUCCESS)
	throw std::runtime_error("Create Descriptor Set Layout Failed");
```


## 创建Graphic Pipeline

### 可编程管线部分

#### Vertex与Fragment

[[Vulkan/概念#Shader Module|Shader Module]]

```cpp
/*******************************可编程管线部分*********************************/
auto vertShaderCode = readFile("shader/vert.spv");
auto fragShaderCode = readFile("shader/frag.spv");

VkShaderModule vertShaderModule = createShaderModule(vertShaderCode);
VkShaderModule fragShaderModule = createShaderModule(fragShaderCode);

VkPipelineShaderStageCreateInfo vertShaderStageCreateInfo{};
vertShaderStageCreateInfo.sType = VK_STRUCTURE_TYPE_PIPELINE_SHADER_STAGE_CREATE_INFO;
vertShaderStageCreateInfo.stage = VK_SHADER_STAGE_VERTEX_BIT;
vertShaderStageCreateInfo.module = vertShaderModule; //Bytecode
vertShaderStageCreateInfo.pName = "main"; //要invoke的函数

VkPipelineShaderStageCreateInfo fragShaderStageCreateInfo{};
fragShaderStageCreateInfo.sType = VK_STRUCTURE_TYPE_PIPELINE_SHADER_STAGE_CREATE_INFO;
fragShaderStageCreateInfo.stage = VK_SHADER_STAGE_FRAGMENT_BIT;
fragShaderStageCreateInfo.module = fragShaderModule; //Bytecode
fragShaderStageCreateInfo.pName = "main"; //要invoke的函数

VkPipelineShaderStageCreateInfo shaderStageCreateInfos[] = {
	vertShaderStageCreateInfo,
	fragShaderStageCreateInfo,
};
```

```cpp
//pipeline创建之后记得删除shaderModule
vkDestroyShaderModule(m_LogicalDevice, vertShaderModule, nullptr);
vkDestroyShaderModule(m_LogicalDevice, fragShaderModule, nullptr);
```

### 固定管线部分

#### 设置Dynamic State

[[Vulkan/概念#Dynamic State|Dynamic State]]

```cpp
//设置Dynamic State
//一般会将Viewport和Scissor设为dynamic，以方便随时修改
bool bViewportAndScissorIsDynamic = false;
std::vector<VkDynamicState> vecDynamicStates = {
	VK_DYNAMIC_STATE_VIEWPORT,
	VK_DYNAMIC_STATE_SCISSOR,
};

VkPipelineDynamicStateCreateInfo dynamicStateCreateInfo{};
dynamicStateCreateInfo.sType = VK_STRUCTURE_TYPE_PIPELINE_DYNAMIC_STATE_CREATE_INFO;
dynamicStateCreateInfo.dynamicStateCount = static_cast<uint32_t>(vecDynamicStates.size());
dynamicStateCreateInfo.pDynamicStates = vecDynamicStates.data();
bViewportAndScissorIsDynamic = true;
```

#### 关联Vertex数据与VertexShaderModule

[[Vulkan/概念#Vertex Input State|Vertex Input State]]
[[Vulkan/概念#Vertex Input Binding Description|Vertex Input Binding Description]]
[[Vulkan/概念#Vertex Input Attribute Description|Vertex Input Attribute Description]]

```cpp
//将Vertex数据与VertexShaderModule关联
auto bindingDescription = Vertex3D::getBindingDescription();
auto attributeDescriptions = Vertex3D::getAttributeDescriptions();
VkPipelineVertexInputStateCreateInfo vertexInputCreateInfo{};
vertexInputCreateInfo.sType = VK_STRUCTURE_TYPE_PIPELINE_VERTEX_INPUT_STATE_CREATE_INFO;
vertexInputCreateInfo.vertexBindingDescriptionCount = 1;
vertexInputCreateInfo.pVertexBindingDescriptions = &bindingDescription;
vertexInputCreateInfo.vertexAttributeDescriptionCount = static_cast<int>(attributeDescriptions.size());
vertexInputCreateInfo.pVertexAttributeDescriptions = attributeDescriptions.data();
```

```cpp
//per-vertex的数据会打包在一个array中，只需要一个binding即可
static VkVertexInputBindingDescription getBindingDescription()
{
	VkVertexInputBindingDescription bindingDescription{};
	bindingDescription.binding = 0; //在binding array中的Idx
	bindingDescription.stride = sizeof(Vertex3D);	//所占字节数
	bindingDescription.inputRate = VK_VERTEX_INPUT_RATE_VERTEX; //传输速率，分为vertex或instance

	return bindingDescription;
}
```

```cpp
//pos，color和texcoord，需要三个attributeDescription
static std::array<VkVertexInputAttributeDescription, 3> getAttributeDescriptions()
{
	std::array<VkVertexInputAttributeDescription, 3> attributeDescriptions{};
	attributeDescriptions[0].binding = 0;
	attributeDescriptions[0].location = 0;
	attributeDescriptions[0].format = VK_FORMAT_R32G32B32_SFLOAT;
	attributeDescriptions[0].offset = offsetof(Vertex3D, pos);

	attributeDescriptions[1].binding = 0;
	attributeDescriptions[1].location = 1;
	attributeDescriptions[1].format = VK_FORMAT_R32G32B32_SFLOAT;
	attributeDescriptions[1].offset = offsetof(Vertex3D, color);

	attributeDescriptions[2].binding = 0;
	attributeDescriptions[2].location = 2;
	attributeDescriptions[2].format = VK_FORMAT_R32G32_SFLOAT;
	attributeDescriptions[2].offset = offsetof(Vertex3D, texCoord);

	return attributeDescriptions;
}
```

#### 设置图元

[[Vulkan/概念#Input Assembly State|Input Assembly State]]

```cpp
//Input Assembly描述要绘制的几何形体，是否启用primitive restart
VkPipelineInputAssemblyStateCreateInfo inputAssemblyCreateInfo{};
inputAssemblyCreateInfo.sType = VK_STRUCTURE_TYPE_PIPELINE_INPUT_ASSEMBLY_STATE_CREATE_INFO;
inputAssemblyCreateInfo.topology = VK_PRIMITIVE_TOPOLOGY_TRIANGLE_LIST;
inputAssemblyCreateInfo.primitiveRestartEnable = VK_FALSE;
```

#### 设置视口与裁剪区域

[[Vulkan/概念#Viewport State|Viewport State]]

```cpp
//设置Viewport，范围为[0,0]到[width,height]
VkViewport viewport{};
viewport.x = 0.f;
viewport.y = 0.f;
viewport.width = static_cast<float>(m_SwapChainExtent.width);
viewport.height = static_cast<float>(m_SwapChainExtent.height);
viewport.minDepth = 0.f;
viewport.maxDepth = 1.f;
```

```cpp
//设置裁剪区域Scissor Rectangle
VkRect2D scissor{};
scissor.offset = { 0,0 };
VkExtent2D ScissorExtent;
ScissorExtent.width = m_SwapChainExtent.width;
ScissorExtent.height = m_SwapChainExtent.height;
scissor.extent = ScissorExtent;
```

```cpp
//设置Viewport与Scissor的数量
VkPipelineViewportStateCreateInfo viewportStateCreateInfo{};
viewportStateCreateInfo.sType = VK_STRUCTURE_TYPE_PIPELINE_VIEWPORT_STATE_CREATE_INFO;
viewportStateCreateInfo.viewportCount = 1;
viewportStateCreateInfo.scissorCount = 1;
//如果没有把Viewport和Scissor设为dynamic state，则需要在此处指定
//使用这种设置方法，设置的Pipeline是不可变的
if (!bViewportAndScissorIsDynamic)
{
	viewportStateCreateInfo.pViewports = &viewport;
	viewportStateCreateInfo.pScissors = &scissor;
}
```

#### 设置光栅化

[[Vulkan/概念#Rasterization State|Rasterization State]]

```cpp
//设置光栅化
VkPipelineRasterizationStateCreateInfo rasterizationStateCreateInfo{};
rasterizationStateCreateInfo.sType = VK_STRUCTURE_TYPE_PIPELINE_RASTERIZATION_STATE_CREATE_INFO;
rasterizationStateCreateInfo.depthClampEnable = VK_FALSE;	//开启后，超过远近平面的部分会被截断在远近平面上，而不是丢弃
rasterizationStateCreateInfo.rasterizerDiscardEnable = VK_FALSE;	//开启后，禁止所有图元经过光栅化器
rasterizationStateCreateInfo.polygonMode = VK_POLYGON_MODE_FILL;	//图元模式，可以是FILL、LINE、POINT
rasterizationStateCreateInfo.lineWidth = 1.f;	//指定光栅化后的线段宽度
rasterizationStateCreateInfo.cullMode = VK_CULL_MODE_BACK_BIT;	//剔除模式，可以是NONE、FRONT、BACK、FRONT_AND_BACK
rasterizationStateCreateInfo.frontFace = VK_FRONT_FACE_COUNTER_CLOCKWISE; //顶点序，可以是顺时针cw或逆时针ccw
rasterizationStateCreateInfo.depthBiasEnable = VK_FALSE; //深度偏移，一般用于Shaodw Map中避免阴影痤疮
rasterizationStateCreateInfo.depthBiasConstantFactor = 0.f;
rasterizationStateCreateInfo.depthBiasClamp = 0.f;
rasterizationStateCreateInfo.depthBiasSlopeFactor = 0.f;
```

#### 设置多重采样

[[Vulkan/概念#Multisample State|Multisample State]]

```cpp
//设置多重采样，抗锯齿
VkPipelineMultisampleStateCreateInfo multisamplingStateCreateInfo{};
multisamplingStateCreateInfo.sType = VK_STRUCTURE_TYPE_PIPELINE_MULTISAMPLE_STATE_CREATE_INFO;
multisamplingStateCreateInfo.sampleShadingEnable = VK_TRUE;
multisamplingStateCreateInfo.minSampleShading = 0.8f;
multisamplingStateCreateInfo.rasterizationSamples = m_MSAASamples;
multisamplingStateCreateInfo.minSampleShading = 1.f;
multisamplingStateCreateInfo.pSampleMask = nullptr;
multisamplingStateCreateInfo.alphaToCoverageEnable = VK_FALSE;
multisamplingStateCreateInfo.alphaToOneEnable = VK_FALSE;
```


#### 设置深度与模板测试

[[Vulkan/概念#Depth Stencil State|Depth Stencil State]]

```cpp
//设置Depth和Stencil Testing
VkPipelineDepthStencilStateCreateInfo depthStencilStateCreateInfo{};
depthStencilStateCreateInfo.sType = VK_STRUCTURE_TYPE_PIPELINE_DEPTH_STENCIL_STATE_CREATE_INFO;
depthStencilStateCreateInfo.depthTestEnable = VK_TRUE;
depthStencilStateCreateInfo.depthWriteEnable = VK_TRUE;
depthStencilStateCreateInfo.depthCompareOp = VK_COMPARE_OP_LESS;
depthStencilStateCreateInfo.depthBoundsTestEnable = VK_FALSE;
depthStencilStateCreateInfo.minDepthBounds = 0.f;
depthStencilStateCreateInfo.maxDepthBounds = 1.f;
depthStencilStateCreateInfo.stencilTestEnable = VK_FALSE;
depthStencilStateCreateInfo.front = {};
depthStencilStateCreateInfo.back = {};
```


#### 设置颜色混合

[[Vulkan/概念#Color Blend State|Color Blend State]]

```cpp
//设置颜色混合
VkPipelineColorBlendAttachmentState colorBlendAttachment{};
colorBlendAttachment.colorWriteMask =
	VK_COLOR_COMPONENT_R_BIT
	| VK_COLOR_COMPONENT_G_BIT
	| VK_COLOR_COMPONENT_B_BIT
	| VK_COLOR_COMPONENT_A_BIT;
colorBlendAttachment.blendEnable = VK_FALSE;
colorBlendAttachment.srcColorBlendFactor = VK_BLEND_FACTOR_ONE;
colorBlendAttachment.dstColorBlendFactor = VK_BLEND_FACTOR_ZERO;
colorBlendAttachment.colorBlendOp = VK_BLEND_OP_ADD;
colorBlendAttachment.srcAlphaBlendFactor = VK_BLEND_FACTOR_ONE;
colorBlendAttachment.dstAlphaBlendFactor = VK_BLEND_FACTOR_ZERO;
colorBlendAttachment.alphaBlendOp = VK_BLEND_OP_ADD;

VkPipelineColorBlendStateCreateInfo colorBlendStateCreateInfo{};
colorBlendStateCreateInfo.sType = VK_STRUCTURE_TYPE_PIPELINE_COLOR_BLEND_STATE_CREATE_INFO;
colorBlendStateCreateInfo.logicOpEnable = VK_FALSE;
colorBlendStateCreateInfo.logicOp = VK_LOGIC_OP_COPY;
colorBlendStateCreateInfo.attachmentCount = 1;
colorBlendStateCreateInfo.pAttachments = &colorBlendAttachment;
colorBlendStateCreateInfo.blendConstants[0] = 0.f;
colorBlendStateCreateInfo.blendConstants[1] = 0.f;
colorBlendStateCreateInfo.blendConstants[2] = 0.f;
colorBlendStateCreateInfo.blendConstants[3] = 0.f;
```


#### 设置管线布局

[[Vulkan/概念#Pipeline Layout|Pipeline Layout]]

```cpp
//设置Pipeline Layout
VkPipelineLayoutCreateInfo pipelineLayoutCreateInfo{};
pipelineLayoutCreateInfo.sType = VK_STRUCTURE_TYPE_PIPELINE_LAYOUT_CREATE_INFO;
pipelineLayoutCreateInfo.setLayoutCount = 1;
pipelineLayoutCreateInfo.pSetLayouts = &m_DescriptorSetLayout;
pipelineLayoutCreateInfo.pushConstantRangeCount = 0;
pipelineLayoutCreateInfo.pPushConstantRanges = nullptr;

VkResult res = vkCreatePipelineLayout(m_LogicalDevice, &pipelineLayoutCreateInfo, nullptr, &m_PipelineLayout);
if (res != VK_SUCCESS)
	throw std::runtime_error("Create Pipeline Layout Failed");
```

### 填充VkGraphicsPipelineCreateInfo

```cpp
/*******************************创建管线*********************************/
VkGraphicsPipelineCreateInfo pipelineCreateInfo{};
pipelineCreateInfo.sType = VK_STRUCTURE_TYPE_GRAPHICS_PIPELINE_CREATE_INFO;
pipelineCreateInfo.stageCount = 2;
pipelineCreateInfo.pStages = shaderStageCreateInfos;
pipelineCreateInfo.pVertexInputState = &vertexInputCreateInfo;
pipelineCreateInfo.pInputAssemblyState = &inputAssemblyCreateInfo;
pipelineCreateInfo.pViewportState = &viewportStateCreateInfo;
pipelineCreateInfo.pRasterizationState = &rasterizationStateCreateInfo;
pipelineCreateInfo.pMultisampleState = &multisamplingStateCreateInfo;
pipelineCreateInfo.pDepthStencilState = &depthStencilStateCreateInfo;
pipelineCreateInfo.pColorBlendState = &colorBlendStateCreateInfo;
//pipelineCreateInfo.pDynamicState = &dynamicStateCreateInfo;
pipelineCreateInfo.layout = m_PipelineLayout;
pipelineCreateInfo.renderPass = m_RenderPass;
pipelineCreateInfo.subpass = 0;
pipelineCreateInfo.basePipelineHandle = VK_NULL_HANDLE;
pipelineCreateInfo.basePipelineIndex = -1;
```

### 创建vkPipeline

```cpp
res = vkCreateGraphicsPipelines(m_LogicalDevice, VK_NULL_HANDLE, 1, &pipelineCreateInfo, nullptr, &m_Pipeline);
if (res != VK_SUCCESS)
	throw std::runtime_error("Create Graphics Pipeline Failed");
```


## 创建Command Pool

[[Vulkan/概念#Command Pool|Command Pool]]

```cpp
void VulkanRenderer::createCommandPool()
{
	QueueFamilyIndices indices = findQueueFamilies(m_PhysicalDevice);

	VkCommandPoolCreateInfo commandPoolCreateInfo{};
	commandPoolCreateInfo.sType = VK_STRUCTURE_TYPE_COMMAND_POOL_CREATE_INFO;
	commandPoolCreateInfo.flags = VK_COMMAND_POOL_CREATE_RESET_COMMAND_BUFFER_BIT;
	commandPoolCreateInfo.queueFamilyIndex = indices.graphicFamily.value();

	VkResult res = vkCreateCommandPool(m_LogicalDevice, &commandPoolCreateInfo, nullptr, &m_CommandPool);
	if (res != VK_SUCCESS)
		throw std::runtime_error("Create Command Pool Failed");

	//创建用于分配单次提交CommandBuffer的Pool
	commandPoolCreateInfo.flags = VK_COMMAND_POOL_CREATE_TRANSIENT_BIT;

	res = vkCreateCommandPool(m_LogicalDevice, &commandPoolCreateInfo, nullptr, &m_TransferCommandPool);
	if (res != VK_SUCCESS)
		throw std::runtime_error("Create Transfer Command Pool Failed");
}
```

## 创建Frame Buffer

### 创建Depth Image View

```cpp
void VulkanRenderer::createFrameBuffers()
{
	//为每个ImageView创建FrameBuffer
	m_vecSwapChainFrameBuffers.resize(m_vecSwapChainImageViews.size());
	for (size_t i = 0; i < m_vecSwapChainImageViews.size(); ++i)
	{
		std::vector<VkImageView> vecImageViewAttachments = { 
			m_MSAAColorImageView,
			m_DepthImageView,
			m_vecSwapChainImageViews[i],
		};

		VkFramebufferCreateInfo frameBufferCreateInfo{};
		frameBufferCreateInfo.sType = VK_STRUCTURE_TYPE_FRAMEBUFFER_CREATE_INFO;
		frameBufferCreateInfo.renderPass = m_RenderPass;
		frameBufferCreateInfo.attachmentCount = static_cast<uint32_t>(vecImageViewAttachments.size());
		frameBufferCreateInfo.pAttachments = vecImageViewAttachments.data();
		frameBufferCreateInfo.width = m_SwapChainExtent.width;
		frameBufferCreateInfo.height = m_SwapChainExtent.height;
		frameBufferCreateInfo.layers = 1;

		if (vkCreateFramebuffer(m_LogicalDevice, &frameBufferCreateInfo, nullptr, &m_vecSwapChainFrameBuffers[i]) != VK_SUCCESS)
			throw std::runtime_error("Create Frame Buffer Failed");
	}
}
```

## 创建Texture Image


## 创建Texture Image View

## 创建Texture Sampler

## 载入模型

## 创建Vertex Buffer

```cpp
void VulkanRenderer::createVertexBuffer()
{
	if (m_Vertices.size() == 0)
		throw std::runtime_error("No Vertex Data");

	VkDeviceSize verticesSize = sizeof(m_Vertices[0]) * m_Vertices.size();

	//创建stageBufferMemory作为transfer的src
	VkBuffer stagingBuffer;
	VkDeviceMemory stagingBufferMemory;
	createBuffer(verticesSize, 
		VK_BUFFER_USAGE_TRANSFER_SRC_BIT,	//用途是作为transfer的src
		VK_MEMORY_PROPERTY_HOST_VISIBLE_BIT | VK_MEMORY_PROPERTY_HOST_COHERENT_BIT, //具有CPU可见的显存类型
		stagingBuffer, stagingBufferMemory);

	//把Vertex数据传输到stageBufferMemory
	void* vertexData;
	vkMapMemory(m_LogicalDevice, stagingBufferMemory, 0, verticesSize, 0, &vertexData);
	memcpy(vertexData, m_Vertices.data(), static_cast<size_t>(verticesSize));
	vkUnmapMemory(m_LogicalDevice, stagingBufferMemory);

	//创建device local的vertexBufferMemory作为transfer的dst
	createBuffer(verticesSize, 
		VK_BUFFER_USAGE_VERTEX_BUFFER_BIT | VK_BUFFER_USAGE_TRANSFER_DST_BIT, //用途是作为transfer的dst、以及vertexBuffer
		VK_MEMORY_PROPERTY_DEVICE_LOCAL_BIT,	//具有性能最佳的DeviceLocal显存类型
		m_VertexBuffer, m_VertexBufferMemory);

	copyBuffer(stagingBuffer, m_VertexBuffer, verticesSize);

	//清理stage buffer及其memory
	vkDestroyBuffer(m_LogicalDevice, stagingBuffer, nullptr);
	vkFreeMemory(m_LogicalDevice, stagingBufferMemory, nullptr);
}
```


## 创建Index Buffer

## 创建Uniform Buffer



# 主循环

# 清理Vulkan


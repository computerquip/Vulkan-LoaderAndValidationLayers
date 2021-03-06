# Layer Description and Status

## Overview

Layer libraries can be written to intercept or hook VK entry points for various
debug and validation purposes.  One or more VK entry points can be defined in your Layer
library.  Undefined entrypoints in the Layer library will be passed to the next Layer which
may be the driver.  Multiple layer libraries can be chained (actually a hierarchy) together.
vkEnumerateInstanceLayerProperties and vkEnumerateDeviceLayerProperties can be called to list the
available layers and their properties. Layers can intercept Vulkan instance level entry points
in which case they are called an Instance Layer.  Layers can intercept device entry  points
in which case they are called a Device Layer. Instance level entry points are those with VkInstance
or VkPhysicalDevice as first parameter.  Device level entry points are those with VkDevice, VkCommandBuffer,
or VkQueue as the first parameter. Layers that want to intercept both instance and device
level entrypoints are called Global Layers. vkXXXXGetProcAddr is used internally by the Layers and
Loader to initialize dispatch tables. Device Layers are activated at vkCreateDevice time. Instance
Layers are activated at vkCreateInstance time.  Layers can also be activated via environment variables
(VK_INSTANCE_LAYERS or VK_DEVICE_LAYERS).

All validation layers work with the DEBUG_REPORT extension to provide the application or user with
validation feedback. When a validation layer is enabled, it will look at the vk_layer_settings.txt
file to determine its behavior. Such as outputing to a file, stdout or debug output (Windows). An
application can also register callback functions via the DEBUG_REPORT extension to receive callbacks
when the requested validation events happen. Application callbacks happen regardless of the
settings in vk_layer_settings.txt

### Layer library example code

Note that some layers are code-generated and will therefore exist in the directory (build_dir)/layers

-include/vkLayer.h  - header file for layer code.

### Layer Details
For complete details of current validation layers, including all of the validation checks that they perform, please refer to the document layers/vk_validation_layer_details.md. Below is a brief overview of each layer.

### Print Object Stats
(build dir)/layers/object_tracker.cpp (name=VK_LAYER_LUNARG_object_tracker) - Track object creation, use, and destruction. As objects are created, they're stored in a map. As objects are used, the layer verifies they exist in the map, flagging errors for unknown objects. As objects are destroyed, they're removed from the map. At vkDestroyDevice() and vkDestroyInstance() times, if any objects have not been destroyed, they are reported as leaked objects. If a Dbg callback function is registered, this layer will use callback function(s) for reporting, otherwise uses stdout.

### Validate Draw State and Shaders
layers/draw\_state.cpp (name=VK_LAYER_LUNARG_draw_state) - The draw\_state layer tracks the Descriptor Set, Pipeline State, Shaders, and dynamic state performing some point validation as states are created and used, and further validation at each Draw call. Of primary interest is making sure that the resources bound to Descriptor Sets correctly align with the layout specified for the Set. Also, all of the image and buffer layouts are validated to make sure explicit layout transitions are properly managed. Additionally draw\_state include shader validation (formerly separate shader\_checker layer) that inspects the SPIR-V shader images and fixed function pipeline stages at PSO creation time. It flags errors when inconsistencies are found across interfaces between shader stages. The exact behavior of the checks depends on the pair of pipeline stages involved. If a Dbg callback function is registered, this layer will use callback function(s) for reporting, otherwise uses stdout.

### Track GPU Memory
layers/mem\_tracker.cpp (name=VK_LAYER_LUNARG_mem_tracker) - The mem\_tracker layer tracks memory objects and references and validates that they are managed correctly by the application.  This includes tracking object bindings, memory hazards, and memory object lifetimes. mem\_tracker validates several other hazard-related issues related to command buffers, fences, and memory mapping. If a Dbg callback function is registered, this layer will use callback function(s) for reporting, otherwise uses stdout.

### Check parameters
layers/param_checker.cpp (name=VK_LAYER_LUNARG_param_checker) - Check the input parameters to API calls for validity. If a Dbg callback function is registered, this layer will use callback function(s) for reporting, otherwise uses stdout.

### Image parameters
layers/image.cpp (name=VK_LAYER_LUNARG_image) - The image layer is intended to validate image parameters, formats, and correct use. Images are a significant enough area that they were given a separate layer. If a Dbg callback function is registered, this layer will use callback function(s) for reporting, otherwise uses stdout.

### Check threading
<build dir>/layers/threading.cpp (name=VK_LAYER_GOOGLE_threading) - Check multithreading of API calls for validity. Currently this checks that only one thread at a time uses an object in free-threaded API calls. If a Dbg callback function is registered, this layer will use callback function(s) for reporting, otherwise uses stdout.

### Swapchain
<build dir>/layer/swapchain.cpp (name=VK_LAYER_LUNARG_swapchain) - Check that WSI extensions are being used correctly.

### Device Limitations
layers/device_limits.cpp (name=VK_LAYER_LUNARG_device_limits) - This layer is intended to capture underlying device features and limitations and then flag errors if an app makes requests for unsupported features or exceeding limitations. This layer is a work in progress and currently only flags some high-level errors without flagging errors on specific feature and limitation. If a Dbg callback function is registered, this layer will use callback function(s) for reporting, otherwise uses stdout.

## Using Layers

1. Build VK loader using normal steps (cmake and make)
2. Place libVkLayer_<name>.so in the same directory as your VK test or app:

    cp build/layer/libVkLayer_threading.so  build/tests

    This is required for the Loader to be able to scan and enumerate your library.
    Alternatively, use the VK\_LAYER\_PATH environment variable to specify where the layer libraries reside.

3. Create a vk_layer_settings.txt file in the same directory to specify how your layers should behave.

    Model it after the following example:  [*vk_layer_settings.txt*](vk_layer_settings.txt)

4. Specify which Layers to activate by using
vkCreateDevice and/or vkCreateInstance or environment variables.

    export VK\_INSTANCE\_LAYERS=VK_LAYER_LUNARG_param_checker:VK_LAYER_LUNARG_draw_state
    export VK\_DEVICE\_LAYERS=VK_LAYER_LUNARG_param_checker:VK_LAYER_LUNARG_draw_state
    cd build/tests; ./vkinfo


## Status


### Current known issues


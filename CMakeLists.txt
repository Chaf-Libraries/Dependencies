set(third_party_folder "Dependencies")

set(BUILD_SHARED_LIBS OFF CACHE BOOL "" FORCE)

if(NOT TARGET glfw)
    option(GLFW_BUILD_EXAMPLES "" OFF)
    option(GLFW_BUILD_TESTS "" OFF)
    option(GLFW_BUILD_DOCS "" OFF)
    option(GLFW_INSTALL "" OFF)
    add_subdirectory(glfw)
    set_target_properties(glfw PROPERTIES FOLDER ${third_party_folder}/glfw)
    set_target_properties(update_mappings PROPERTIES FOLDER ${third_party_folder}/glfw)
endif()

if(NOT TARGET spdlog)
    option(SPDLOG_BUILD_EXAMPLE "" OFF)
    option(SPDLOG_INSTALL "" OFF)
    add_subdirectory(spdlog)
    set_target_properties(spdlog PROPERTIES FOLDER ${third_party_folder}/spdlog)
endif()

if(NOT TARGET glm)
    file(GLOB_RECURSE GLM_FILES "${CMAKE_CURRENT_SOURCE_DIR}/glm/glm/*.hpp" "${CMAKE_CURRENT_SOURCE_DIR}/glm/glm/detail/glm.cpp")
    add_library(glm ${GLM_FILES})
    target_include_directories(glm PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/glm)
    set_target_properties(glm PROPERTIES FOLDER ${third_party_folder}/glm)
endif()

if(NOT TARGET vulkan)
    add_library(vulkan INTERFACE)
    set(VULKAN_INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/vulkan/include)
    target_sources(vulkan INTERFACE ${VULKAN_INCLUDE_DIR}/vulkan/vulkan.h)
    target_include_directories(vulkan INTERFACE ${VULKAN_INCLUDE_DIR})
    target_compile_definitions(vulkan INTERFACE VK_NO_PROTOTYPES)

    if(ANDROID)
        target_compile_definitions(vulkan INTERFACE VK_USE_PLATFORM_ANDROID_KHR)
    elseif(WIN32)
        target_compile_definitions(vulkan INTERFACE VK_USE_PLATFORM_WIN32_KHR)
    elseif(APPLE)
        target_compile_definitions(vulkan INTERFACE VK_USE_PLATFORM_MACOS_MVK)
    elseif(UNIX)
        # See whether X11 is available. If not, fall back to direct-to-display mode.
        find_package(X11 QUIET)
        if (X11_FOUND)
            target_compile_definitions(vulkan INTERFACE VK_USE_PLATFORM_XCB_KHR)
        else()
            set(DIRECT_TO_DISPLAY TRUE)
            set(DIRECT_TO_DISPLAY TRUE PARENT_SCOPE)
            target_compile_definitions(vulkan INTERFACE VK_USE_PLATFORM_DISPLAY_KHR)
        endif()
    endif() 
endif()

if(NOT TARGET vma)
    add_library(vma INTERFACE)
    set(VMA_DIR ${CMAKE_CURRENT_SOURCE_DIR}/vma/include)
    target_sources(vma INTERFACE ${VMA_DIR}/vk_mem_alloc.h)
    target_include_directories(vma INTERFACE ${VMA_DIR})
    target_link_libraries(vma INTERFACE vulkan)
    target_compile_definitions(vma INTERFACE VMA_STATIC_VULKAN_FUNCTIONS)
endif()

if(NOT TARGET volk)
    set(VOLK_DIR "${CMAKE_CURRENT_SOURCE_DIR}/volk")
    set(VOLK_FILES
        "${VOLK_DIR}/volk.c"
        "${VOLK_DIR}/volk.h")
    add_library(volk STATIC ${VOLK_FILES})
    target_link_libraries(volk PUBLIC vulkan)
    target_include_directories(volk PUBLIC ${VOLK_DIR})
    set_target_properties(volk PROPERTIES FOLDER ${third_party_folder}/volk)
endif()

if(NOT TARGET stb)
    file(GLOB_RECURSE stb_sources CONFIGURE_DEPENDS  "${CMAKE_CURRENT_SOURCE_DIR}/stb/*.h")
    add_library(stb INTERFACE ${stb_sources})
    target_include_directories(stb INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}/stb)
    set_target_properties(stb PROPERTIES FOLDER ${third_party_folder}/stb)
endif()

if(NOT TARGET D3D12MemoryAllocator)
    add_subdirectory(D3D12MA)
    target_include_directories(D3D12MemoryAllocator INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}/D3D12MA/include)
    set_target_properties(D3D12MemoryAllocator PROPERTIES FOLDER ${third_party_folder}/D3D12MA)
endif()

if(NOT TARGET glslang)
    option(ENABLE_HLSL "" OFF)
    option(ENABLE_SPVREMAPPER "" OFF)
    option(ENABLE_GLSLANG_BINARIES "" OFF)
    option(ENABLE_CTEST "" OFF)
    add_subdirectory(glslang)
    set_target_properties(glslang PROPERTIES FOLDER ${third_party_folder}/glslang)
    set_target_properties(GenericCodeGen PROPERTIES FOLDER ${third_party_folder}/glslang)
    set_target_properties(MachineIndependent PROPERTIES FOLDER ${third_party_folder}/glslang)
    set_target_properties(OGLCompiler PROPERTIES FOLDER ${third_party_folder}/glslang)
    set_target_properties(OSDependent PROPERTIES FOLDER ${third_party_folder}/glslang)
    set_target_properties(SPIRV PROPERTIES FOLDER ${third_party_folder}/glslang)
endif()

if(NOT TARGET dxc)
    add_library(dxc INTERFACE)
    target_include_directories(dxc INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}/dxc/inc)
    target_link_libraries(dxc INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}/dxc/lib/x64/dxcompiler.lib)
    set_target_properties(dxc PROPERTIES FOLDER ${third_party_folder}/dxc)
endif()

# spirv-cross
option(SPIRV_CROSS_CLI "" OFF)
option(SPIRV_CROSS_ENABLE_CPP "" OFF)
option(SPIRV_CROSS_ENABLE_C_API "" OFF)
option(SPIRV_CROSS_ENABLE_MSL "" OFF)
option(SPIRV_CROSS_ENABLE_TESTS "" OFF)
option(SPIRV_CROSS_ENABLE_UTIL "" OFF)
add_subdirectory(spirv_cross)
set_target_properties(spirv-cross-core PROPERTIES FOLDER ${third_party_folder}/spirv_cross)
set_target_properties(spirv-cross-glsl PROPERTIES FOLDER ${third_party_folder}/spirv_cross)
set_target_properties(spirv-cross-hlsl PROPERTIES FOLDER ${third_party_folder}/spirv_cross)
set_target_properties(spirv-cross-reflect PROPERTIES FOLDER ${third_party_folder}/spirv_cross)

# spirv-reflect
option(SPIRV_REFLECT_EXAMPLES "" OFF)
option(SPIRV_REFLECT_EXECUTABLE "" OFF)
option(SPIRV_REFLECT_STATIC_LIB "" ON)
add_subdirectory(spirv_reflect)
set_target_properties(spirv-reflect-static PROPERTIES FOLDER ${third_party_folder}/spirv_reflect)

if(NOT TARGET imgui)
    set(IMGUI_DIR "${CMAKE_CURRENT_SOURCE_DIR}/imgui")
    set(IMGUI_FILES
        "${IMGUI_DIR}/imgui.cpp"
        "${IMGUI_DIR}/imgui_demo.cpp"
        "${IMGUI_DIR}/imgui_draw.cpp"
        "${IMGUI_DIR}/imgui_widgets.cpp"
        "${IMGUI_DIR}/imgui_tables.cpp"
        "${IMGUI_DIR}/imconfig.h"
        "${IMGUI_DIR}/imgui.h"
        "${IMGUI_DIR}/imgui_internal.h"
        "${IMGUI_DIR}/imstb_rectpack.h"
        "${IMGUI_DIR}/imstb_textedit.h"
        "${IMGUI_DIR}/imstb_truetype.h"
        "${IMGUI_DIR}/backends/imgui_impl_vulkan.h"
        "${IMGUI_DIR}/backends/imgui_impl_vulkan.cpp"
        "${IMGUI_DIR}/backends/imgui_impl_glfw.h"
        "${IMGUI_DIR}/backends/imgui_impl_glfw.cpp")
    add_library(imgui STATIC ${IMGUI_FILES})
    target_include_directories(imgui PUBLIC ${IMGUI_DIR} ${IMGUI_DIR}/backends)
    target_link_libraries(imgui glfw volk)
    set_target_properties(imgui PROPERTIES FOLDER ${third_party_folder}/imgui)
endif()

if(NOT TARGET imnode)
    file(GLOB_RECURSE imnode_sources CONFIGURE_DEPENDS  "${CMAKE_CURRENT_SOURCE_DIR}/imnode/*")
    add_library(imnode STATIC ${imnode_sources})
    target_include_directories(imnode INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}/imnode)
    target_link_libraries(imnode imgui)
    set_target_properties(imnode PROPERTIES FOLDER ${third_party_folder}/imnode)
endif()

if(NOT TARGET implot)
    file(GLOB implot_sources CONFIGURE_DEPENDS  "${CMAKE_CURRENT_SOURCE_DIR}/implot/*")
    add_library(implot STATIC ${implot_sources})
    target_include_directories(implot INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}/implot)
    target_link_libraries(implot imgui)
    set_target_properties(implot PROPERTIES FOLDER ${third_party_folder}/implot)
endif()

if(NOT TARGET rttr)
    option(BUILD_EXAMPLES "" OFF)
    option(BUILD_DOCUMENTATION "" OFF)
    option(BUILD_EXTERNAL "" OFF)
    option(BUILD_INSTALLER "" OFF)
    option(BUILD_PACKAGE "" OFF)
    option(BUILD_RTTR_DYNAMIC "" OFF)
    option(BUILD_UNIT_TESTS "" OFF)
    option(BUILD_STATIC "" ON)
    add_subdirectory(rttr)
    set_target_properties(rttr_core_lib PROPERTIES FOLDER ${third_party_folder}/rttr)
endif()

if(NOT TARGET cereal)
    add_library(cereal INTERFACE)
    target_include_directories(cereal INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}/cereal/include)
    set_target_properties(cereal PROPERTIES FOLDER ${third_party_folder}/cereal)
endif()

if(NOT TARGET entt)
    add_library(entt INTERFACE)
    target_include_directories(entt INTERFACE "${CMAKE_CURRENT_SOURCE_DIR}/entt/single_include/entt")
    set_target_properties(entt PROPERTIES FOLDER ${third_party_folder}/entt)
endif()

if(NOT TARGET cgltf)
    add_library(cgltf INTERFACE)
    target_include_directories(cgltf INTERFACE "${CMAKE_CURRENT_SOURCE_DIR}/cgltf")
    set_target_properties(cgltf PROPERTIES FOLDER ${third_party_folder}/cgltf)
endif()

if(NOT TARGET meshoptimizer)
    add_subdirectory(meshoptimizer)
    set_target_properties(meshoptimizer PROPERTIES FOLDER ${third_party_folder}/meshoptimizer)
endif()

if(NOT TARGET ImGuizmo)
    add_library(ImGuizmo STATIC "${CMAKE_CURRENT_SOURCE_DIR}/ImGuizmo/ImGuizmo.h" "${CMAKE_CURRENT_SOURCE_DIR}/ImGuizmo/ImGuizmo.cpp")
    target_include_directories(ImGuizmo INTERFACE "${CMAKE_CURRENT_SOURCE_DIR}/ImGuizmo")
    target_link_libraries(ImGuizmo imgui)
    set_target_properties(ImGuizmo PROPERTIES FOLDER ${third_party_folder}/ImGuizmo)
endif()

if(NOT TARGET sol2)
file(GLOB_RECURSE SOL_FILES "${CMAKE_CURRENT_SOURCE_DIR}/sol2/*")
add_library(sol2 STATIC "${SOL_FILES}")
target_include_directories(sol2 INTERFACE "${CMAKE_CURRENT_SOURCE_DIR}/sol2/lua")
target_include_directories(sol2 INTERFACE "${CMAKE_CURRENT_SOURCE_DIR}/sol2")
set_target_properties(sol2 PROPERTIES FOLDER ${third_party_folder}/sol2)
endif()

if(NOT TARGET nativefiledialog)
    file(GLOB_RECURSE nativefiledialog_sources CONFIGURE_DEPENDS  "${CMAKE_CURRENT_SOURCE_DIR}/nativefiledialog/src/*.h" "${CMAKE_CURRENT_SOURCE_DIR}/nativefiledialog/src/*.cpp" "${CMAKE_CURRENT_SOURCE_DIR}/nativefiledialog/src/nfd_common.c")
    add_library(nativefiledialog STATIC ${nativefiledialog_sources})
    target_include_directories(nativefiledialog PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/nativefiledialog/src/include)
    target_include_directories(nativefiledialog PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/nativefiledialog/src)
    set_target_properties(nativefiledialog PROPERTIES FOLDER ${third_party_folder}/nativefiledialog)
endif()

add_subdirectory(antlr)
set_target_properties(ANTLR_Runtime PROPERTIES FOLDER ${third_party_folder}/ANTLR)
set_target_properties(ANTLR_CPP14Parser PROPERTIES FOLDER ${third_party_folder}/ANTLR)

add_library(glad STATIC "${CMAKE_CURRENT_SOURCE_DIR}/glad/src/glad.c")
target_include_directories(glad PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/glad/include)
set_target_properties(glad PROPERTIES FOLDER ${third_party_folder}/glad)

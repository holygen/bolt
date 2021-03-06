cmake_minimum_required(VERSION 3.2)

project(tests)

set_policy()

SET(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${BOLT_ROOT}/cmakes")
find_package(Uni)
find_package(ModelTools)
find_package(Image)
find_package(TensorComputing)
find_package(Inference)
find_package(jpeg)
if(USE_MALI)
    find_package(Gcl)
endif(USE_MALI)

set_project_install_directory()
set_c_cxx_flags()
set_test_c_cxx_flags()


function(blas_enhance name)
    add_executable(${name} ${name}.cpp)
    add_dependencies(${name} blas-enhance)
    add_dependencies(${name} blas-enhance_static)
    target_link_libraries(${name} ${BLAS_ENHANCE_LIBRARY})
endfunction()

function(tensor_computing name)
    add_executable(${name} ${name}.cpp)
    add_dependencies(${name} tensor_computing)
    add_dependencies(${name} tensor_computing_static)
    target_link_libraries(${name} ${TENSOR_COMPUTING_LIBRARIES})
    if(USE_MALI)
        target_link_libraries(${name} ${OPENCL_LIBRARIES})
    endif(USE_MALI)
endfunction()

function(image name)
    add_executable(${name} ${name}.cpp)
    add_dependencies(${name} image)
    add_dependencies(${name} image_static)
    target_link_libraries(${name} ${IMAGE_LIBRARIES})
endfunction()

function(model_tools name)
    add_executable(${name} ${name}.cpp)
    if (USE_CAFFE)
        add_dependencies(${name} model-tools)
        add_dependencies(${name} model-tools_static)
        add_dependencies(${name} model-tools_caffe)
        add_dependencies(${name} model-tools_caffe_static)
        TARGET_LINK_LIBRARIES(${name} ${MODEL_TOOLS_LIBRARIES})
    endif (USE_CAFFE)

    if (USE_ONNX)
        add_dependencies(${name} model-tools)
        add_dependencies(${name} model-tools_static)
        add_dependencies(${name} model-tools_onnx)
        add_dependencies(${name} model-tools_onnx_static)
        TARGET_LINK_LIBRARIES(${name} ${MODEL_TOOLS_LIBRARIES})
    endif (USE_ONNX)

    if (USE_TFLITE)
        add_dependencies(${name} model-tools)
        add_dependencies(${name} model-tools_static)
        add_dependencies(${name} model-tools_tflite)
        add_dependencies(${name} model-tools_tflite_static)
        TARGET_LINK_LIBRARIES(${name} ${MODEL_TOOLS_LIBRARIES})
    endif (USE_TFLITE)
endfunction()

function(inference name)
    file(GLOB srcs ${name}.c ${name}.cpp)
    add_executable(${name} ${srcs})
    if (USE_DYNAMIC_LIBRARY)
        TARGET_LINK_LIBRARIES(${name} inference)
    else (USE_DYNAMIC_LIBRARY)
        TARGET_LINK_LIBRARIES(${name} inference_static)
    endif (USE_DYNAMIC_LIBRARY)
    TARGET_LINK_LIBRARIES(${name} ${INFERENCE_LIBRARIES} ${JPEG_LIBRARY})
    if (USE_MALI)
        TARGET_LINK_LIBRARIES(${name} ${KERNELBIN_LIBRARIES} ${OPENCL_LIBRARIES})
    endif (USE_MALI)
endfunction()

tensor_computing(tensor_computing_library_search)
if (BUILD_TEST)
    if (USE_NEON)
        blas_enhance(test_mmm)
        blas_enhance(test_mvm)
        tensor_computing(test_activation)
        tensor_computing(test_attention)
        tensor_computing(test_clip)
        tensor_computing(test_concat)
        tensor_computing(test_convolution)
        tensor_computing(test_deconvolution)
        tensor_computing(test_depthwise_convolution)
        tensor_computing(test_dilated_convolution)
        tensor_computing(test_eltwise)
        tensor_computing(test_fully_connected)
        tensor_computing(test_lstm)
        tensor_computing(test_multiply)
        tensor_computing(test_axis_mean)
        tensor_computing(test_pooling)
        tensor_computing(test_reshape)
        tensor_computing(test_softmax)
        tensor_computing(test_split)
        tensor_computing(test_slice)
        tensor_computing(test_scale)
        tensor_computing(test_transpose)
        
        if (USE_INT8)
            blas_enhance(test_mmm_int8)
            blas_enhance(test_mvm_int8)
            tensor_computing(test_convolution_int8)
            tensor_computing(test_depthwise_convolution_int8)
            tensor_computing(test_concat_int8)
            tensor_computing(test_pooling_int8)
        endif(USE_INT8)
        
        tensor_computing(test_convolution_bnn)
    endif (USE_NEON)
    
    image(test_image_processing)
    image(test_image_resize)
    
    inference(lenet)
    inference(bert)
    inference(tinybert)
    inference(classification)
    inference(classification_bin)
    inference(nmt)
    inference(test_api_c)
    if (USE_MALI)
        if (USE_FP16)
        inference(test_pipeline_ocl)
        inference(hdr_ocl)
        endif (USE_FP16)
    endif (USE_MALI)
endif (BUILD_TEST)

if (USE_CAFFE)
    model_tools(caffe2bolt)
endif (USE_CAFFE)
if (USE_ONNX)
    model_tools(onnx2bolt)
endif (USE_ONNX)
if (USE_TFLITE)
    model_tools(tflite2bolt)
endif (USE_TFLITE)

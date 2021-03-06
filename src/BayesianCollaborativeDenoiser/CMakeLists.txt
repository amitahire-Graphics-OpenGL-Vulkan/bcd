# --------------------------------------------------------------------------------------------------
# Source code for the paper no.36 "Bayesian Collaborative Denoising
# for Monte-Carlo Rendering", submitted to the ACM SIGGRAPH 2016
# conference technical paper program.
#
# For review only. Please, do not distribute outside the review context.
#
# Final open source licence and comments to be inserted if published.
#
# Copyright(C) 2014-2016
# The Authors
#
# All rights reserved.
# ---------------------------------------------------------------------------------------------------


SET(target_name bcd)


# First we check whether optional and required libraries have been found

SET(required_library_is_missing FALSE)
SET(missing_required_libraries "")

SET(optional_library_is_missing FALSE)
SET(missing_optional_libraries "")


if (NOT OPENMP_FOUND)
	SET(optional_library_is_missing TRUE)
	SET(missing_optional_libraries ${missing_optional_libraries} OpenMP)
endif(NOT OPENMP_FOUND) 

if (NOT CUDA_FOUND)
	SET(optional_library_is_missing TRUE)
	SET(missing_optional_libraries ${missing_optional_libraries} "CUDA")
endif(NOT CUDA_FOUND)

if(optional_library_is_missing)
	message(WARNING "missing optional libraries ${missing_optional_libraries} for target ${target_name}")
endif(optional_library_is_missing)

if(required_library_is_missing)
	message(WARNING "disabling target ${target_name} which requires libraries ${missing_required_libraries}")
	return()
endif(required_library_is_missing)





SET(src_files
	Denoiser.cpp
	DenoisingUnit.cpp
	MultiscaleDenoiser.cpp
	SpikeRemovalFilter.cpp
	ParametersIO.cpp
	SamplesAccumulator.cpp
	${common_folder}/Chronometer.cpp
	${common_folder}/ImageIO.cpp
	${common_folder}/Utils.cpp
	${common_folder}/CovarianceMatrix.cpp
	${exr_folder}/io_exr.cpp
)
SET(header_files
	IDenoiser.h
	Denoiser.h
	DenoisingUnit.h
	MultiscaleDenoiser.h
	SpikeRemovalFilter.h
	ParametersIO.h
	SamplesAccumulator.h
	${common_folder}/DeepImage.h
	${common_folder}/Chronometer.h
	${common_folder}/ImageIO.h
	${common_folder}/Utils.h
	${common_folder}/CovarianceMatrix.h
	${exr_folder}/io_exr.h
)
SET(header_implementation_files
	${common_folder}/DeepImage.hpp
)

IF(CUDA_FOUND)
	MESSAGE(STATUS "Using cuda's nvcc for generating ${target_name}")
	list(APPEND CUDA_NVCC_FLAGS "-arch=sm_20")
	SET(src_files ${src_files} CudaHistogramDistance.cu)
	SET(header_files ${header_files} CudaHistogramDistance.h CudaUtils.h)
	CUDA_ADD_LIBRARY(${target_name} ${src_files} ${header_files} ${header_implementation_files} STATIC)
	TARGET_COMPILE_DEFINITIONS(${target_name} PRIVATE FOUND_CUDA)
ELSE(CUDA_FOUND)
	ADD_LIBRARY(${target_name} STATIC ${src_files} ${header_files} ${header_implementation_files})
ENDIF(CUDA_FOUND)


TARGET_INCLUDE_DIRECTORIES(${target_name} PUBLIC ${common_folder})
TARGET_INCLUDE_DIRECTORIES(${target_name} PRIVATE ${exr_folder})
TARGET_INCLUDE_DIRECTORIES(${target_name} PUBLIC ${bcd_folder})



SET_TARGET_PROPERTIES(${target_name} PROPERTIES CXX_STANDARD 11)
TARGET_LINK_LIBRARIES(${target_name} PUBLIC OpenMP)
TARGET_LINK_LIBRARIES(${target_name} PRIVATE OpenExr)
TARGET_LINK_LIBRARIES(${target_name} PRIVATE eigen)
TARGET_LINK_LIBRARIES(${target_name} PRIVATE Json)

INSTALL(TARGETS ${target_name} ARCHIVE DESTINATION lib)

SET(bcd_library ${target_name} PARENT_SCOPE)


cmake_minimum_required(VERSION 2.8.8)

if (BUILD_FOR_CPU)
	add_library(tridcpu_sp OBJECT ./trid_cpu.cpp ./trid_mpi_cpu.cpp)
	add_library(tridcpu_dp OBJECT ./trid_cpu.cpp ./trid_mpi_cpu.cpp)
	add_library(tridcpu    SHARED $<TARGET_OBJECTS:tridcpu_sp> $<TARGET_OBJECTS:tridcpu_dp>)
#	add_library(tridcpu    SHARED $<TARGET_OBJECTS:tridcpu_dp>)

	target_include_directories(tridcpu_sp PRIVATE ${PROJECT_SOURCE_DIR}/include ./ PRIVATE ../ )
	target_include_directories(tridcpu_dp PRIVATE ${PROJECT_SOURCE_DIR}/include ./ PRIVATE ../ )

	#/opt/intel/composer_xe_2015.3.187/bin/intel64/icpc -O3 -xAVX -ipo -restrict -parallel -fopenmp -qopt-report=2 -qopt-report-phase=vec -qopt-report-phase=par -offload-attribute-target=mic -mkl -offload-option,mic,ld,"-L/opt/intel/composer_xe_2015.3.187/mkl/lib/mic -lmkl_intel_ilp64 -lmkl_intel_thread -lmkl_core -L/opt/intel/composer_xe_2015.3.187/compiler/lib/mic -limf -lintlc -lsvml -lirng -liomp5 -loffload -lcilkrts" -DFPPREC=0  -DN_MAX=1024  -I./include -I./libtrid  -D__OFFLOAD__ src/adi_cpu.cpp -L./libtrid/lib -limf -lintlc -lsvml -lirng  -o adi_phi_offload

	#set_property(TARGET tridcpu_sp PROPERTY POSITION_INDEPENDENT_CODE ON)
	#set_property(TARGET tridcpu_dp PROPERTY POSITION_INDEPENDENT_CODE ON)

	#set_target_properties(tridcpu_sp PROPERTIES COMPILE_FLAGS "-fPIC -DFPPREC=0")
	#set_target_properties(tridcpu_dp PROPERTIES COMPILE_FLAGS "-fPIC -DFPPREC=1")
	
	target_compile_options(tridcpu_sp PRIVATE -fPIC) 
	target_compile_options(tridcpu_dp PRIVATE -fPIC)
	
	target_compile_definitions(tridcpu_sp PRIVATE -DFPPREC=0)
	target_compile_definitions(tridcpu_dp PRIVATE -DFPPREC=1)

	install(TARGETS tridcpu    
		LIBRARY DESTINATION ${CMAKE_BINARY_DIR}/lib
		ARCHIVE DESTINATION ${CMAKE_BINARY_DIR}/lib)
	install(FILES ${PROJECT_SOURCE_DIR}/include/trid_cpu.h 
		            ${PROJECT_SOURCE_DIR}/include/trid_common.h 
		            ${PROJECT_SOURCE_DIR}/include/trid_util.h
		            ${PROJECT_SOURCE_DIR}/include/trid_simd.h
		            ${PROJECT_SOURCE_DIR}/include/trid_mpi_cpu.h
                ${PROJECT_SOURCE_DIR}/src/cpu/trid_mpi_cpu.hpp
                ${PROJECT_SOURCE_DIR}/include/trid_mpi_solver_params.hpp
		      DESTINATION ${CMAKE_BINARY_DIR}/include)
endif (BUILD_FOR_CPU)

if (BUILD_FOR_MIC AND INTEL_CC)
	add_library(tridmic_offload_sp OBJECT ./trid_cpu.cpp)
	add_library(tridmic_offload_dp OBJECT ./trid_cpu.cpp)

	add_library(tridmic_native_sp OBJECT ./trid_cpu.cpp)
	add_library(tridmic_native_dp OBJECT ./trid_cpu.cpp)

	set(MICINCLUDES /software/mpss-3.5.2/src/glibc-2.14.1+mpss3.5.2/assert/ /software/mpss-3.5.2/src/glibc-2.14.1+mpss3.5.2/misc/sys/ /software/mpss-3.5.2/src/glibc-2.14.1+mpss3.5.2/include/ /software/mpss-3.5.2/src/glibc-2.14.1+mpss3.5.2/sys/ )

	target_include_directories(tridmic_offload_sp PUBLIC ${PROJECT_SOURCE_DIR}/include ./ PRIVATE ../ ${MICINCLUDES} )
	target_include_directories(tridmic_offload_dp PUBLIC ${PROJECT_SOURCE_DIR}/include ./ PRIVATE ../ ${MICINCLUDES} )

	target_include_directories(tridmic_native_sp  PUBLIC ${PROJECT_SOURCE_DIR}/include ./ PRIVATE ../ ${MICINCLUDES} )
	target_include_directories(tridmic_native_dp  PUBLIC ${PROJECT_SOURCE_DIR}/include ./ PRIVATE ../ ${MICINCLUDES} )

	target_compile_definitions(tridmic_offload_sp PRIVATE -DFPPREC=0)
	target_compile_definitions(tridmic_offload_dp PRIVATE -DFPPREC=1)
	#set_target_properties(tridmic_offload_sp PROPERTIES COMPILE_FLAGS "-fPIC -DFPPREC=0")
	#set_target_properties(tridmic_offload_dp PROPERTIES COMPILE_FLAGS "-fPIC -DFPPREC=1")

	target_compile_options(tridmic_offload_sp PRIVATE -xAVX -m64 -parallel -qopenmp -fPIC -offload -qopt-report-phase=par -opt-report-phase=offload -offload-attribute-target=mic -mkl -offload-option,mic,ld,"-L${INTEL_PATH}/mkl/lib/mic -lmkl_intel_ilp64 -lmkl_intel_thread -lmkl_core -L${INTEL_PATH}/compiler/lib/mic -limf -lintlc -lsvml -lirng -liomp5 -loffload -lcilkrts" -I./include -I./libtrid  -D__OFFLOAD__ -L./libtrid/lib -limf -lintlc -lsvml -lirng)
	target_compile_options(tridmic_offload_dp PRIVATE -xAVX -m64 -parallel -qopenmp -fPIC -offload -qopt-report-phase=par -opt-report-phase=offload -offload-attribute-target=mic -mkl -offload-option,mic,ld,"-L${INTEL_PATH}/mkl/lib/mic -lmkl_intel_ilp64 -lmkl_intel_thread -lmkl_core -L${INTEL_PATH}/compiler/lib/mic -limf -lintlc -lsvml -lirng -liomp5 -loffload -lcilkrts" -I./include -I./libtrid  -D__OFFLOAD__ -L./libtrid/lib -limf -lintlc -lsvml -lirng)


	#set_target_properties(tridcpu_sp PROPERTIES COMPILE_FLAGS "-DFPPREC=0")
	#set_target_properties(tridcpu_dp PROPERTIES COMPILE_FLAGS "-DFPPREC=1")

	target_compile_definitions(tridmic_native_sp PRIVATE -DFPPREC=0)
	target_compile_definitions(tridmic_native_dp PRIVATE -DFPPREC=1)
	target_compile_options(tridmic_native_sp PRIVATE -mmic -parallel -qopenmp -fPIC)
	target_compile_options(tridmic_native_dp PRIVATE -mmic -parallel -qopenmp -fPIC)

	add_library(tridmic_offload SHARED $<TARGET_OBJECTS:tridmic_offload_sp> $<TARGET_OBJECTS:tridmic_offload_dp>)
	set_target_properties(tridmic_offload PROPERTIES LINK_FLAGS -L./libtrid/lib -limf -lintlc -lsvml -lirng)

	add_library(tridmic_native SHARED $<TARGET_OBJECTS:tridmic_native_sp> $<TARGET_OBJECTS:tridmic_native_dp>)

	install(TARGETS tridmic_offload 
		LIBRARY DESTINATION ${CMAKE_BINARY_DIR}/lib
		ARCHIVE DESTINATION ${CMAKE_BINARY_DIR}/lib)
	install(TARGETS tridmic_native  
		LIBRARY DESTINATION ${CMAKE_BINARY_DIR}/lib
		ARCHIVE DESTINATION ${CMAKE_BINARY_DIR}/lib)
	install(FILES ${PROJECT_SOURCE_DIR}/include/trid_cpu.h 
					${PROJECT_SOURCE_DIR}/include/trid_common.h 
					${PROJECT_SOURCE_DIR}/include/trid_simd.h
					${PROJECT_SOURCE_DIR}/src/cpu/trid_mpi_cpu.hpp
					${PROJECT_SOURCE_DIR}/include/trid_mpi_solver_params.hpp
		DESTINATION ${CMAKE_BINARY_DIR}/include)
endif (BUILD_FOR_MIC AND INTEL_CC)



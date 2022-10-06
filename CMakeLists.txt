cmake_minimum_required(VERSION 3.10)
project(constraintControllerLite VERSION 1.0.0)

add_subdirectory(ros)

set(REQUIRED_CISST_LIBRARIES
    cisstCommon
    cisstVector
    cisstOSAbstraction
    cisstMultiTask
    cisstParameterTypes
    cisstRobot
    cisstNumerical
)

find_package(cisst 1.1.0 COMPONENTS ${REQUIRED_CISST_LIBRARIES})

if(cisst_FOUND_AS_REQUIRED)
    # load cisst configuration
    include(${CISST_USE_FILE})

    cisst_find_saw_component (sawConstraintController REQUIRED)
    find_package(sawConstraintController REQUIRED)
    include_directories(${sawConstraintController_INCLUDE_DIR})
    include_directories(${sawConstraintController_HEADER_DIR})
    link_directories(${sawConstraintController_LIBRARY_DIR})

    # catkin/ROS paths
    cisst_is_catkin_build(constraintControllerLite_IS_CATKIN_BUILT)

    if(constraintControllerLite_IS_CATKIN_BUILT)
        set(constraintControllerLite_CONFIG_FILE_DIR "${CATKIN_DEVEL_PREFIX}/share/constraintControllerLite/cmake")
        set(LIBRARY_OUTPUT_PATH "${constraintControllerLite_BINARY_DIR}/lib")
    else()
        set(constraintControllerLite_CONFIG_FILE_DIR "${constraintControllerLite_BINARY_DIR}")
    endif()

    # create/configure file for find_package (sawConstraintController)
    set(constraintControllerLite_INCLUDE_DIR
        "${constraintControllerLite_SOURCE_DIR}/include"
        "${constraintControllerLite_BINARY_DIR}/include"
    )
    set(constraintControllerLite_LIBRARY_DIR "${LIBRARY_OUTPUT_PATH}")
    set(constraintControllerLite_LIBRARIES constraintControllerLite)

    # Set the version number
    set(constraintControllerLite_VERSION_MAJOR "1")
    set(constraintControllerLite_VERSION_MINOR "0")
    set(constraintControllerLite_VERSION_PATCH "0")
    set(constraintControllerLite_VERSION "${constraintControllerLite_VERSION_MAJOR}.${constraintControllerLite_VERSION_MINOR}.${constraintControllerLite_VERSION_PATCH}")

    # Generate constraintControllerLiteRevision.h
    configure_file("${constraintControllerLite_SOURCE_DIR}/code/constraintControllerLiteRevision.h.in"
        "${constraintControllerLite_BINARY_DIR}/include/constraintControllerLite/constraintControllerLiteRevision.h")

    include_directories(${constraintControllerLite_INCLUDE_DIR})
    set(constraintControllerLite_HEADER_DIR "include/constraintControllerLite")

    set(HEADER_FILES
        ${constraintControllerLite_HEADER_DIR}/ccl_helper.h
        ${constraintControllerLite_HEADER_DIR}/mtsConstraintBase.h
        ${constraintControllerLite_HEADER_DIR}/constraintControllerLite.h
        ${constraintControllerLite_HEADER_DIR}/mtsConstraintFollowPositionTraj.h
        ${constraintControllerLite_HEADER_DIR}/mtsConstraintAbsoluteJointLimits.h
        ${constraintControllerLite_HEADER_DIR}/mtsConstraintFixOrientation.h
        ${constraintControllerLite_HEADER_DIR}/mtsConstraintJointVelLimits.h
        ${constraintControllerLite_HEADER_DIR}/mtsConstraintStayOnAxis.h
    )

    set(SOURCE_FILES
        code/ccl_helper.cpp
        code/constraintControllerLite.cpp
        code/mtsConstraintFollowPositionTraj.cpp
        code/mtsConstraintAbsoluteJointLimits.cpp
        code/mtsConstraintFixOrientation.cpp
        code/mtsConstraintJointVelLimits.cpp
        code/mtsConstraintStayOnAxis.cpp
    )

    add_library(constraintControllerLite
        ${HEADER_FILES}
        ${SOURCE_FILES}
    )

    set_property(TARGET constraintControllerLite PROPERTY FOLDER "constraintControllerLite")
    cisst_target_link_libraries(constraintControllerLite ${REQUIRED_CISST_LIBRARIES} ${cisstConstraintController_LIBRARIES})

    set(constraintControllerLite_CMAKE_CONFIG_FILE "${constraintControllerLite_CONFIG_FILE_DIR}/constraintControllerLiteConfig.cmake")
    configure_file("${CMAKE_CURRENT_SOURCE_DIR}/cmake/constraintControllerLiteConfig.cmake.in"
        "${constraintControllerLite_CMAKE_CONFIG_FILE}"
        @ONLY)

    cisst_add_config_version("${constraintControllerLite_CONFIG_FILE_DIR}/constraintControllerLiteConfigVersion.cmake"
        VERSION ${constraintControllerLite_VERSION}
        DESTINATION "share/constraintControllerLite"
        COMPONENT constraintControllerLite)

    # Install targets for headers and library
    install(DIRECTORY
        "${constraintControllerLite_SOURCE_DIR}/include/constraintControllerLite"
        "${constraintControllerLite_BINARY_DIR}/include/constraintControllerLite"
        DESTINATION include
        PATTERN .svn EXCLUDE)

    install(TARGETS constraintControllerLite
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib)

    install(FILES "${constraintControllerLite_CMAKE_CONFIG_FILE}"
        DESTINATION "share/constraintControllerLite")

else(cisst_FOUND_AS_REQUIRED)
    message("Information: code in ${CMAKE_CURRENT_SOURCE_DIR} will not be compiled, it requires ${REQUIRED_CISST_LIBRARIES}")
endif(cisst_FOUND_AS_REQUIRED)
cmake_minimum_required(VERSION 3.4)

if(${CMAKE_SOURCE_DIR} STREQUAL ${CMAKE_BINARY_DIR})
    message(FATAL_ERROR "Prevented in-tree built. Please create a build directory outside of the source code and call cmake from there. Thank you.")
endif()

project(qt_cpackifw VERSION 0.0.1)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -O0 -g -Wall -DDEBUG")
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -O2 -DRELEASE")

set(CMAKE_INCLUDE_CURRENT_DIR ON)

add_executable(qt_cpackifw WIN32 MACOSX_BUNDLE src/main.cpp)

install(
    TARGETS qt_cpackifw
    RUNTIME DESTINATION qt_cpackifw
    COMPONENT qt_cpackifw_installer
    BUNDLE DESTINATION qt_cpackifw
    COMPONENT qt_cpackifw_installer
)

function(CPACKIFW_COMMON)
    set(CPACK_PACKAGE_NAME qt_cpackifw)
    set(CPACK_PACKAGE_FILE_NAME installer)
    set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Installation Tool")
    set(CPACK_PACKAGE_VERSION "1.0.0") # Version of installer
    set(CPACK_COMPONENTS_ALL qt_cpackifw_installer)
    set(CPACK_IFW_PACKAGE_START_MENU_DIRECTORY Qt_CPackIFW)
    set(CPACK_GENERATOR IFW)
    set(CPACK_IFW_VERBOSE ON)

    include(CPack REQUIRED)
    include(CPackIFW REQUIRED)

    cpack_add_component(
        qt_cpackifw_installer
        DISPLAY_NAME "Qt CPackIFW"
        DESCRIPTION "Install me"
        REQUIRED
    )

    cpack_ifw_configure_component(
        qt_cpackifw_installer
        FORCED_INSTALLATION
        NAME qt.cpackifw.installer
        VERSION ${PROJECT_VERSION} # Version of component
        LICENSES License ${qt_cpackifw_SOURCE_DIR}/LICENSE
        DEFAULT TRUE
    )
endfunction()

if(CPACK_IFW_ROOT OR DEFINED ENV{QTIFWDIR})
    if(DEFINED ENV{QTDIR})
        if(APPLE)
            if(EXISTS $ENV{QTDIR}/bin/macdeployqt)
                message("*")
                message("* Note")
                message("*")
                message("* Because of a bug in CPackIFW, it doesn't manage correctly the package created on OSX.")
                message("* Unfortunately CPackIFW doesn't forward the .dmg extension to the binarycreator (see IFW documentation for more details).")
                message("* Therefore it creates a .app directory on OSX, that isn't properly a file and cannot be treated the same way .run and .exe files are.")
                message("* As a result, make package generates an empty directory within the build directory and leaves the bundle somewhere within the _CPack_Packages directory.")
                message("*")
                message("* I strongly suggest to run binarycreator through an external script and use the .dmg extension directly. At least until the bug in CPackIFW isn't fixed.")
                message("* From within the build directory run:")
                message("*")
                message("*     binarycreator -f -c _CPack_Packages/Darwin/IFW/installer/config/config.xml -p _CPack_Packages/Darwin/IFW/installer/packages/ package/installer.dmg")
                message("*")
                message("* The bundle and the dmg file will be created within the package directory as:")
                message("*")
                message("*     package/installer.app [bundle]")
                message("*     package/installer.dmg [dmg]")
                message("*")

                add_custom_command(
                    TARGET qt_cpackifw POST_BUILD
                    COMMAND $ENV{QTDIR}/bin/macdeployqt ${PROJECT_NAME}.app
                )

                CPACKIFW_COMMON()
            else()
                message("Unable to find executable QTDIR/bin/macdeployqt.")
            endif()
        elseif(WIN32)
            if(EXISTS $ENV{QTDIR}/bin/windeployqt.exe)
                add_custom_command(
                    TARGET qt_cpackifw POST_BUILD
                    COMMAND ${CMAKE_COMMAND} -E remove_directory ${CMAKE_BINARY_DIR}/windeployqt_stuff
                    COMMAND $ENV{QTDIR}/bin/windeployqt.exe --compiler-runtime --dir ${CMAKE_BINARY_DIR}/windeployqt_stuff $<TARGET_FILE:qt_cpackifw>
                )

                install(
                    DIRECTORY ${CMAKE_BINARY_DIR}/windeployqt_stuff/
                    DESTINATION ${PROJECT_NAME}
                    COMPONENT qt_cpackifw_installer
                )

                CPACKIFW_COMMON()
            else()
                message("Unable to find executable QTDIR/bin/windeployqt.")
            endif()
        elseif(UNIX)
            CPACKIFW_COMMON()
        endif()
    else()
        message("Set properly environment variable QTDIR to be able to create a package.")
    endif()
else()
    message("If you want to enable target package you can:")
    message("\t* Either pass -DCPACK_IFW_ROOT=<path> to cmake")
    message("\t* Or set the environment variable QTIFWDIR")
    message("To specify the location of the QtIFW tool suite.")
    message("The specified path should not contain bin at the end (for example: D:\\DevTools\\QtIFW2.0.5).")
endif()

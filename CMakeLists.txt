#
# Project Definition
#
cmake_minimum_required(VERSION 3.8.2)
project(CEFProxy)

#
# Find dependencies
#
find_package(ThreadComms 0.1 REQUIRED)
find_package(FixedJSON 0.1 REQUIRED)
find_package(DevToolsLog 0.1 REQUIRED)
find_package(OSDevTools 0.1 REQUIRED)
find_package(UtilTime 0.1 REQUIRED)
find_package(GTKUtils 0.1 REQUIRED)
find_package(CPPWebSocketResponseRequest 0.1 REQUIRED)
find_package(Boost REQUIRED COMPONENTS system thread)
find_package(GTK REQUIRED gtk)
find_package(WEBSOCKETPP 0.8.2 REQUIRED)
find_package(OpenSSL REQUIRED)
find_package(CEF REQUIRED)
find_package(CEFBaseApp REQUIRED)


add_library(CEFProxyLib STATIC
    include/ProxyBrowserApp.h
    include/ProxyRESTEndPoint.h
    include/ReqNavigate.h
    include/ReqResource.h
    include/SubOnResourceLoaded.h
    ProxyLib/ProxyBrowserApp.cpp
    ProxyLib/ProxyRESTEndPoint.cpp
    ProxyLib/ReqNavigate.cpp
    ProxyLib/ReqResource.cpp
    ProxyLib/SubOnResourceLoaded.cpp
        include/ReqCookieJar.h ProxyLib/ReqCookieJar.cpp)

target_link_libraries(CEFProxyLib
        FixedJSON::FixedJSON
        ThreadComms::ThreadComms
        DevToolsLog::Log
        OSDevTools::OSDevTools
        Boost::system
        Boost::thread
        Boost::boost
        OpenSSL::SSL
        OpenSSL::Crypto
        CEFBaseApp::CEFBaseAppUtils
        CEFBaseApp::CEFBaseAppCore
)

target_include_directories(CEFProxyLib
    PUBLIC
        $<BUILD_INTERFACE:${CEFProxy_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>
    PRIVATE
        ${CEF_INCLUDE_PATH}
)

set_property(TARGET CEFProxyLib PROPERTY PUBLIC_HEADER
    ${CEFProxy_SOURCE_DIR}/include/ProxyBrowserApp.h
    ${CEFProxy_SOURCE_DIR}/include/ProxyRESTEndPoint.h
    ${CEFProxy_SOURCE_DIR}/include/ReqNavigate.h
    ${CEFProxy_SOURCE_DIR}/include/ReqResource.h
    ${CEFProxy_SOURCE_DIR}/include/SubOnResourceLoaded.h
)



add_executable(proxy proxy/proxy.cpp)
#TODO: This config need cleaning up - see how much we can drag back into the BaseApp project

#
# Test Configuration
#
find_package(GTest REQUIRED)

add_executable(cookie test/cookie.cpp)
COPY_FILES(cookie "${CEF_BINARY_FILES}" "${CEF_BINARY_DIR_RELEASE}" ${CEFProxy_BINARY_DIR})
COPY_FILES(cookie "${CEF_RESOURCE_FILES}" "${CEF_BINARY_DIR_RELEASE}" ${CEFProxy_BINARY_DIR})
COPY_FILES(cookie "index.html" ${CEFProxy_SOURCE_DIR}/test ${CEFProxy_BINARY_DIR})

add_executable(navigate test/loaded.cpp)
COPY_FILES(navigate "redirect.html" ${CEFProxy_SOURCE_DIR}/test ${CEFProxy_BINARY_DIR})
COPY_FILES(navigate "dummy.html" ${CEFProxy_SOURCE_DIR}/test
        ${CEFProxy_BINARY_DIR})

enable_testing()
add_test(cookie cookie)
add_test(navigate navigate)

target_link_libraries(cookie
    PUBLIC
    CEFProxyLib
    CPPWebSocketResponseRequest::CPPWebSocketResponseRequest
    CEFBaseApp::CefTestApp
    ${CEF_BINARY_DIR_RELEASE}/libcef_dll_wrapper.a
    ${CEF_LIB_RELEASE}
    ${CEF_STANDARD_LIBS}
    GTest::GTest
)

target_include_directories(cookie
    PUBLIC
    $<BUILD_INTERFACE:${CEFBaseApp_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
    PRIVATE
    ${CEF_INCLUDE_PATH}
)

target_link_libraries(navigate
        PUBLIC
        CEFProxyLib
        CPPWebSocketResponseRequest::CPPWebSocketResponseRequest
        CEFBaseApp::CefTestApp
        ${CEF_BINARY_DIR_RELEASE}/libcef_dll_wrapper.a
        ${CEF_LIB_RELEASE}
        ${CEF_STANDARD_LIBS}
        GTest::GTest
        )


target_include_directories(navigate
        PUBLIC
        $<BUILD_INTERFACE:${CEFBaseApp_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>
        PRIVATE
        ${CEF_INCLUDE_PATH}
        )




# Determine the target output directory.
SET_CEF_TARGET_OUT_DIR()

target_link_libraries(proxy
        CEFProxyLib
        CEFBaseApp::CEFBaseAppUtils
        CEFBaseApp::CEFBaseAppCore
        CEFBaseApp::CEFBaseAppDefaults
        GTKUtils::GTKUtil
        CPPWebSocketResponseRequest::CPPWebSocketResponseRequest
        ${CEF_BINARY_DIR_RELEASE}/libcef_dll_wrapper.a
        ${CEF_LIB_RELEASE}
        ${CEF_STANDARD_LIBS}
)

target_include_directories(proxy
    PRIVATE
        ${CEF_INCLUDE_PATH}
)

#TODO: Now this links we've done a lot of the hard work. We still need to do
#      the file copy etc to allow the binary to run + the above todo around
#       makin it easier for client libraries to link...

set(PROJECT_NAME ext_server_kiwi)

project(${PROJECT_NAME} C)

include(${CMAKE_CURRENT_SOURCE_DIR}/../CMakeListsFuncs.txt)

add_definitions(
    -D_USRDLL
    -DUNICODE
    -D_UNICODE
    -D_POWERKATZ
)

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} /MP")

include_directories(../../source/common)
include_directories(../../source/ReflectiveDllInjection/common)
include_directories(../../source/extensions/kiwi/mimikatz/inc)

set(SRC_DIR ../../source/extensions/kiwi)

# Unfortunately we have to be a little judicious with what we include
file(GLOB SRC_FILES
    ${SRC_DIR}/*.c
    ${SRC_DIR}/mimikatz/modules/*.c
    ${SRC_DIR}/mimikatz/modules/rpc/*.c
    ${SRC_DIR}/mimikatz/mimikatz/*.c
    ${SRC_DIR}/mimikatz/mimikatz/modules/*.c
    ${SRC_DIR}/mimikatz/mimikatz/modules/crypto/*.c
    ${SRC_DIR}/mimikatz/mimikatz/modules/dpapi/*.c
    ${SRC_DIR}/mimikatz/mimikatz/modules/dpapi/packages/*.c
    ${SRC_DIR}/mimikatz/mimikatz/modules/kerberos/*.c
    ${SRC_DIR}/mimikatz/mimikatz/modules/lsadump/*.c
    ${SRC_DIR}/mimikatz/mimikatz/modules/lsadump/packages/*.c
    ${SRC_DIR}/mimikatz/mimikatz/modules/sekurlsa/*.c
    ${SRC_DIR}/mimikatz/mimikatz/modules/sekurlsa/crypto/*.c
    ${SRC_DIR}/mimikatz/mimikatz/modules/sekurlsa/packages/*.c
)

# Ditch some old stuff that shouldn't even be here in the mimikatz repo!
list(REMOVE_ITEM SRC_FILES ${CMAKE_CURRENT_SOURCE_DIR}/${SRC_DIR}/mimikatz/mimikatz/modules/kuhl_m_dpapi.c)

add_library(${PROJECT_NAME} SHARED ${SRC_FILES})
set_target_properties(${PROJECT_NAME} PROPERTIES OUTPUT_NAME ${PROJECT_NAME}.${TARGET_ARCH})

if(IS_X86)
    set(KIWI_LIB_DIR ${WORKSPACE_ROOT_DIR}/../source/extensions/kiwi/mimikatz/lib/Win32)
else()
    set(KIWI_LIB_DIR ${WORKSPACE_ROOT_DIR}/../source/extensions/kiwi/mimikatz/lib/x64)
endif()

set(LINK_LIBS
    psapi
    crypt32
    Shlwapi
    Secur32
    Wtsapi32
    winscard
    wldap32
    Netapi32
    Rpcrt4
    SetupAPI
    Userenv
    Cabinet
    Dnsapi
    version
    msxml2
)
# Add all the custom libs that come with the mimikatz source
set(LINK_LIBS
    ${LINK_LIBS}
    ${KIWI_LIB_DIR}/advapi32.hash.lib
    ${KIWI_LIB_DIR}/cryptdll.lib
    ${KIWI_LIB_DIR}/fltlib.lib
    ${KIWI_LIB_DIR}/hid.lib
    ${KIWI_LIB_DIR}/msasn1.min.lib
    ${KIWI_LIB_DIR}/ncrypt.lib
    ${KIWI_LIB_DIR}/netapi32.min.lib
    ${KIWI_LIB_DIR}/ntdll.min.lib
    ${KIWI_LIB_DIR}/samlib.lib
    ${KIWI_LIB_DIR}/winsta.lib
)

if(IS_X86)
    set(LINK_LIBS ${LINK_LIBS} backcompat)
endif()
target_link_libraries(${PROJECT_NAME} ${LINK_LIBS})

# Post processing (required for all Meterpreter DLLs)
editbin(${PROJECT_NAME} ${BIN_SUBSYSTEM})
copyoutput(${PROJECT_NAME} ${BIN_OUTPUT_DIR})

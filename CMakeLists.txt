project(PunalWebScketBoostBeast)

cmake_minimum_required(VERSION 3.0.0)

include(ExternalProject)
#set(simple_example)

#Macros Start
MACRO(SUBDIRLIST result curdir)
  FILE(GLOB children RELATIVE ${curdir} ${curdir}/*)
  SET(dirlist "")
  FOREACH(child ${children})
    IF(IS_DIRECTORY ${curdir}/${child})
      LIST(APPEND dirlist ${child})
    ENDIF()
  ENDFOREACH()
  SET(${result} ${dirlist})
ENDMACRO()
#
MACRO(SUBFOLDEREXITS doesexist curdir foldertosearch)  
  set(${doesexist} 0)

  if(EXISTS ${curdir}/${foldertosearch})
    if(IS_DIRECTORY ${curdir}/${foldertosearch})
        set(${doesexist} 1)
    endif()
  endif()
ENDMACRO()
#Macros End

#Global Vars Start
    #
    ##Boost Specific Start
        ##
        set(P_Boost_Root_Dir ${PROJECT_SOURCE_DIR}/boost/boost)
        set(P_Boost_Simplified_Include False)#Example All the Include Files are Arranged in Direct Subdirectory instead of Inner Subdirectory
        ##
    ##Boost Specific End
    #
    ##OpenSSL Specific Start
        ##
        set(P_OpenSSL_Root_Dir ${PROJECT_SOURCE_DIR}/openssl)
        set(P_OpenSSL_Lib_Dir ${P_OpenSSL_Root_Dir})
        set(P_OpenSSL_Sould_Link_crypt32 False)
        set(P_OpenSSL_Lib_Name_1 libcrypto)
        set(P_OpenSSL_Lib_Name_2 libssl)
        set(P_OpenSSL_Should_Build False)
        ##
    ##OpenSSL Specific End
    #
#Global Vars End

#CMake Build_Compile Vars Start
    #set(CMAKE_CONFIGURATION_TYPES "Debug;Release;MinSizeRel;RelWithDebInfo" CACHE STRING "Supported configs" FORCE)
    add_compile_options(/bigobj)#In Case it Complains fatal error C1128: number of sections exceeded object file format limit: compile with /bigobj 
#CMake Build_Compile Vars End

#External Library_Project Vars Start
    #
    ##Boost Specific Start
        ##
        #set(Boost_DEBUG 1)
        set(BOOST_ROOT ${P_Boost_Root_Dir})
        #set(Boost_USE_STATIC_LIBS OFF) 
        #set(Boost_USE_MULTITHREADED ON)  
        #set(Boost_USE_STATIC_RUNTIME OFF)
        ##
    ##Boost Specific End
    #
    ##OpenSSL Specific Start
        ##
        set(OPENSSL_ROOT_DIR ${P_OpenSSL_Root_Dir}) # either set it here or from the command line
        set(OPENSSL_INCLUDE_DIR ${OPENSSL_ROOT_DIR}/include)
        #set(OPENSSL_ROOT_DIR ${OPENSSL_ROOT})
        ##
    ##OpenSSL Specific End
    #
#External Library_Project Vars End

#External Library_Project Additonal Setup Start
    #
    ##Boost Specific Start
        ##
        ###Adding All Boost Sud Directories Start
            ###
                if(NOT P_Boost_Simplified_Include)
                    SUBDIRLIST(P_BOOST_SUB_MODULES ${BOOST_ROOT}/libs)
                    FOREACH(subdir ${P_BOOST_SUB_MODULES})  
                        set(P_Local_SubDir_Exists False)
                        SUBFOLDEREXITS(P_Local_SubDir_Exists ${BOOST_ROOT}/libs/${subdir} "include")

                        if (P_Local_SubDir_Exists EQUAL 1)
                            include_directories(${BOOST_ROOT}/libs/${subdir}/include)
                        else()
                            SUBDIRLIST(P_Local_Boost_SubDirectories ${BOOST_ROOT}/libs/${subdir})
        
                            FOREACH(Local_subdir ${P_Local_Boost_SubDirectories})  
                                set(P_Local_SubDir_Exists 0)
                                SUBFOLDEREXITS(P_Local_SubDir_Exists ${BOOST_ROOT}/libs/${subdir}/${Local_subdir} include)
                                if (P_Local_SubDir_Exists EQUAL 1)
                                    include_directories(${BOOST_ROOT}/libs/${subdir}/${Local_subdir}/include)
                                endif()
                            ENDFOREACH()
                        endif()
                    ENDFOREACH()
                else()
                    SUBDIRLIST(P_BOOST_SUB_MODULES ${BOOST_ROOT})
                    FOREACH(subdir ${P_BOOST_SUB_MODULES})  
                        include_directories(${BOOST_ROOT}/${subdir})
                    ENDFOREACH()
                endif()
            ###
        ###Adding All Boost Sud Directories End
        ##
    ##Boost Specific End
    #
    ##OpenSSL Specific Start
        ##
        ###Adding All OpenSSL Includes Start
            ###
                include_directories(${OPENSSL_INCLUDE_DIR})
            ###
        ###Adding All OpenSSL Includes End
        ##
        ###OpenSSL Build Start
            ###
                if(P_OpenSSL_Should_Build)
                    if(CMAKE_HOST_WIN32)   
                        set(P_Visual_Studio_Pre_Build_Command_Target)
                        if(NOT CMAKE_CL_64)
                            set(P_OPENSSL_BUILD_COMMAND_1 perl ${OPENSSL_ROOT_DIR}/Configure VC-WIN32)
                        else()
                            set(P_OPENSSL_BUILD_COMMAND_1 perl ${OPENSSL_ROOT_DIR}/Configure VC-WIN64A)
                        endif()
                        set(P_OPENSSL_BUILD_COMMAND_2 nmake clean)
                        set(P_OPENSSL_BUILD_COMMAND_3 nmake)
                    elseif(CMAKE_HOST_APPLE)
                        set(P_OPENSSL_BUILD_COMMAND_1 perl ${OPENSSL_ROOT_DIR}/Configure)
                        set(P_OPENSSL_BUILD_COMMAND_2 make clean)
                        set(P_OPENSSL_BUILD_COMMAND_3 make)
                    else()
                        message(FATAL_ERROR "OpenSSL Pre-Build Failed As Host OS Is Invalid/Unspecified")
                    endif()
    
                    set(P_OPENSSL_BUILD_COMMAND_VAR)  
                    if(WIN32)
                        add_custom_target(
                            P_OPENSSL_BUILD_COMMAND_VAR ALL
                            WORKING_DIRECTORY ${OPENSSL_ROOT_DIR}
                            COMMAND ${P_OPENSSL_BUILD_COMMAND_1}
                            COMMAND ${P_OPENSSL_BUILD_COMMAND_2}
                            COMMAND ${P_OPENSSL_BUILD_COMMAND_3}
                            BYPRODUCTS ${P_OpenSSL_Lib_Dir}/${P_OpenSSL_Lib_Name_1}.lib ${P_OpenSSL_Lib_Dir}/${P_OpenSSL_Lib_Name_2}.lib
                            USES_TERMINAL
                        )
                    endif()
                    if(APPLE)
                        add_custom_target(
                            P_OPENSSL_BUILD_COMMAND_VAR ALL
                            WORKING_DIRECTORY ${OPENSSL_ROOT_DIR}
                            COMMAND ${P_OPENSSL_BUILD_COMMAND_1}
                            COMMAND ${P_OPENSSL_BUILD_COMMAND_2}
                            COMMAND ${P_OPENSSL_BUILD_COMMAND_3}
                            BYPRODUCTS ${P_OpenSSL_Lib_Dir}/${P_OpenSSL_Lib_Name_1}.a ${P_OpenSSL_Lib_Dir}/${P_OpenSSL_Lib_Name_2}.a
                            USES_TERMINAL
                        )
                    endif()
                endif()
            ###
        ###OpenSSL Build End
        ##
    ##OpenSSL Specific End
    #
#External Library_Project Additonal Setup End

#Example Files ONLY FOR TESTING Start
    #
    include_directories(${BOOST_ROOT}/libs/beast/example)
    add_executable(simple_example main.cpp)
    if(P_OpenSSL_Should_Build)
        add_dependencies(simple_example P_OPENSSL_BUILD_COMMAND_VAR)
    endif()
    #
#Example Files ONLY FOR TESTING End

#Library Linking Start
    #
    ##OpenSSL Specific Start
        ##
        IF(True)
            if(WIN32)
                target_link_libraries(simple_example ${P_OpenSSL_Lib_Dir}/${P_OpenSSL_Lib_Name_1}.lib)
                target_link_libraries(simple_example ${P_OpenSSL_Lib_Dir}/${P_OpenSSL_Lib_Name_2}.lib)
                if(P_OpenSSL_Sould_Link_crypt32)
                    target_link_libraries(simple_example crypt32.lib)
                endif()
            endif()
            if(APPLE)
                target_link_libraries(simple_example ${P_OpenSSL_Lib_Dir}/${P_OpenSSL_Lib_Name_1}.a)
                target_link_libraries(simple_example ${P_OpenSSL_Lib_Dir}/${P_OpenSSL_Lib_Name_2}.a)
            endif()
        ENDIF()
        ##
    ##OpenSSL Specific End
    #
#Library Linking End

SET(VERSION_MAJOR 0)
SET(VERSION_MINOR 1)

IF("${PARAVIEW_VERSION_MAJOR}.${PARAVIEW_VERSION_MINOR}" VERSION_LESS "3.10")
  SET(SERVER_MANAGER_XML_FILE FbxExporter_3_8_1.xml)
ELSE()
  SET(SERVER_MANAGER_XML_FILE FbxExporter.xml)
ENDIF() # PARAVIEW_VERSION

# Initing and exiting of Fbx
IF(PARAVIEW_QT_VERSION VERSION_GREATER "4")
  QT5_WRAP_CPP(MOC_SRCS pqFbxStarter.h)
ELSE()
  QT4_WRAP_CPP(MOC_SRCS pqFbxStarter.h)
ENDIF()
ADD_PARAVIEW_AUTO_START(IFACES IFACE_SRCS CLASS_NAME pqFbxStarter
                        STARTUP onStartup
                        SHUTDOWN onShutdown)

ADD_PARAVIEW_PLUGIN(FbxExporter "${VERSION_MAJOR}.${VERSION_MINOR}"
	GUI_INTERFACES ${IFACES}
	SERVER_MANAGER_XML ${SERVER_MANAGER_XML_FILE}
	SERVER_MANAGER_SOURCES vtkFbxExporter.cxx
	SERVER_SOURCES ${CMAKE_SOURCE_DIR}/vtkFbxConverter.cxx
	SOURCES pqFbxStarter.cxx ${MOC_SRCS} ${IFACE_SRCS}
	)

TARGET_LINK_LIBRARIES(FbxExporter PUBLIC VtkFbxConverter ${LINK_LIBRARIES})

### INSTALL and PACKAGING ###

# Install targets with an absolute destination path will not be packaged at all!!!
OPTION(PACKAGING "This will package everything in a zip file." OFF)
OPTION(INSTALL_IN_PARAVIEW "This will install the plugin in ParaView." OFF)
IF(APPLE)
  OPTION(INSTALL_IN_APP "" ON)
  SET(MAC_APP_LOCATION ${ParaView_DIR}/../../../install/Applications/paraview.app CACHE PATH "")
ENDIF()
SET(INSTALL_DIRECTORY ${CMAKE_INSTALL_PREFIX})
IF(INSTALL_IN_PARAVIEW OR INSTALL_IN_APP)
  IF(APPLE)
    IF(USE_INSTALL_PREFIX)
      SET(INSTALL_DIRECTORY ${CMAKE_INSTALL_PREFIX}/bin/paraview.app/Contents/MacOS/plugins)
    ELSEIF(INSTALL_IN_APP)
      SET(INSTALL_DIRECTORY ${MAC_APP_LOCATION}/Contents/MacOS/plugins)
    ELSE()
      SET(INSTALL_DIRECTORY ${ParaView_DIR}/bin/paraview.app/Contents/MacOS/plugins)
    ENDIF()
  ELSE()
    IF(USE_INSTALL_PREFIX)
      SET(INSTALL_DIRECTORY ${CMAKE_INSTALL_PREFIX}/bin/plugins)
    ELSE()
      SET(INSTALL_DIRECTORY ${ParaView_DIR}/bin/plugins)
    ENDIF()
  ENDIF() # APPLE
ENDIF() # INSTALL_IN_PARAVIEW

IF(WIN32)
	INSTALL(TARGETS FbxExporter RUNTIME DESTINATION ${INSTALL_DIRECTORY})
	INSTALL(FILES
	  ${CMAKE_SOURCE_DIR}/README.md
	  DESTINATION ${INSTALL_DIRECTORY})

	IF(PACKAGING)
	  SET(CPACK_GENERATOR ZIP)
	  SET(CPACK_PACKAGE_VERSION_MAJOR "${VERSION_MAJOR}")
    SET(CPACK_PACKAGE_VERSION_MINOR "${VERSION_MINOR}")
    SET(CPACK_PACKAGE_VERSION_PATCH "0")
    SET(CPACK_PACKAGE_DESCRIPTION_SUMMARY "ParaView Fbx-Exporter Plugin")
    SET(CPACK_PACKAGE_VENDOR "Lars Bilke")
    SET(CPACK_PACKAGE_INSTALL_DIRECTORY ".")
    IF(${CMAKE_SIZEOF_VOID_P} EQUAL 8)
      SET(WIN_VERSION "win64")
    ELSE()
      SET(WIN_VERSION "win32")
    ENDIF()
    SET(CPACK_PACKAGE_FILE_NAME "pv_fbx_plugin-${VERSION_MAJOR}.${VERSION_MINOR}-${WIN_VERSION}-ParaView-${PARAVIEW_VERSION_MAJOR}.${PARAVIEW_VERSION_MINOR}.${PARAVIEW_VERSION_PATCH}")
    SET(CPACK_PACKAGE_DESCRIPTION_FILE "${CMAKE_SOURCE_DIR}/README.md")
    INCLUDE(CPack)
  ENDIF() # PACKAGING
ELSE()
	INSTALL(TARGETS FbxExporter LIBRARY DESTINATION ${INSTALL_DIRECTORY})
ENDIF() # WIN32

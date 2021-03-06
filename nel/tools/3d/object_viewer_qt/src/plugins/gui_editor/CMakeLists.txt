IF(WITH_LUA51)
	FIND_PACKAGE(Lua51 REQUIRED)
ELSE(WITH_LUA51)
	FIND_PACKAGE(Lua50 REQUIRED)
ENDIF(WITH_LUA51)
	FIND_PACKAGE(Luabind REQUIRED)
	FIND_PACKAGE(CURL REQUIRED)
	
IF(WIN32 OR CURL_LIBRARIES MATCHES "\\.a")
	SET(CURL_STATIC ON)
ENDIF(WIN32 OR CURL_LIBRARIES MATCHES "\\.a")

IF(CURL_STATIC)
	SET(CURL_DEFINITIONS -DCURL_STATICLIB)
	FIND_PACKAGE(OpenSSL QUIET)
	
	IF(OPENSSL_FOUND)
		SET(CURL_INCLUDE_DIRS ${CURL_INCLUDE_DIRS} ${OPENSSL_INCLUDE_DIR})
		SET(CURL_LIBRARIES ${CURL_LIBRARIES} ${OPENSSL_LIBRARIES})
	ENDIF(OPENSSL_FOUND)
	
	# CURL Macports version depends on libidn, libintl and libiconv too
	IF(APPLE)
		FIND_LIBRARY(IDN_LIBRARY idn)
		FIND_LIBRARY(INTL_LIBRARY intl)
		SET(CURL_LIBRARIES ${CURL_LIBRARIES} ${IDN_LIBRARY} ${INTL_LIBRARY})
	ENDIF(APPLE)
ENDIF(CURL_STATIC)

FIND_PACKAGE(Libwww REQUIRED)


INCLUDE_DIRECTORIES(	${CMAKE_CURRENT_BINARY_DIR}
			${CMAKE_CURRENT_SOURCE_DIR}
			${LIBXML2_INCLUDE_DIR}
			${LUA_INCLUDE_DIR}
			${QT_INCLUDES})

FILE(GLOB SRC *.cpp *.h)

SET(OVQT_EXT_SYS_SRC	${CMAKE_CURRENT_SOURCE_DIR}/../../extension_system/iplugin.h
			${CMAKE_CURRENT_SOURCE_DIR}/../../extension_system/iplugin_manager.h
			${CMAKE_CURRENT_SOURCE_DIR}/../../extension_system/iplugin_spec.h)

SET(OVQT_PLUGIN_GUI_EDITOR_HDR
	gui_editor_plugin.h
	gui_editor_window.h
	gui_editor_context.h
	gui_editor_core_listener.h
	widget_properties.h
	widget_hierarchy.h
	link_list.h
	link_editor.h
	proc_list.h
	proc_editor.h
	action_editor.h
	property_browser_ctrl.h
	project_window.h
	nel3d_widget.h
	nelgui_widget.h
	new_property_widget.h
	new_widget_widget.h
	)
	
SET(OVQT_PLUGIN_GUI_EDITOR_UIS
	gui_editor_window.ui
	widget_properties.ui
	widget_hierarchy.ui
	link_list.ui
	link_editor.ui
	proc_list.ui
	proc_editor.ui
	action_editor.ui
	project_window.ui
	new_property_widget.ui
	new_widget_widget.ui
	)

SET(QT_USE_QTGUI TRUE)
SET(QT_USE_QTOPENGL TRUE)

QT4_ADD_RESOURCES(OVQT_PLUGIN_GUI_EDITOR_RC_SRCS ${OVQT_PLUGIN_GUI_EDITOR_RCS})
QT4_WRAP_CPP(OVQT_PLUGIN_GUI_EDITOR_MOC_SRC ${OVQT_PLUGIN_GUI_EDITOR_HDR})
QT4_WRAP_UI(OVQT_PLUGIN_GUI_EDITOR_UI_HDRS ${OVQT_PLUGIN_GUI_EDITOR_UIS})

SOURCE_GROUP(QtResources FILES ${OVQT_PLUGIN_GUI_EDITOR_UIS})
SOURCE_GROUP(QtGeneratedUiHdr FILES ${OVQT_PLUGIN_GUI_EDITOR_UI_HDRS})
SOURCE_GROUP(QtGeneratedMocQrcSrc FILES ${OVQT_PLUGIN_GUI_EDITOR_MOC_SRC} OVQT_PLUGIN_GUI_EDITOR_RC_SRCS)
SOURCE_GROUP("GUI Editor Plugin" FILES ${SRC})
SOURCE_GROUP("OVQT Extension System" FILES ${OVQT_EXT_SYS_SRC})

ADD_LIBRARY(ovqt_plugin_gui_editor MODULE ${SRC}
            ${OVQT_PLUGIN_GUI_EDITOR_MOC_SRC}
            ${OVQT_EXT_SYS_SRC}
            ${OVQT_PLUGIN_GUI_EDITOR_UI_HDRS}
            ${OVQT_PLUGIN_GUI_EDITOR_RC_SRCS}
            )

TARGET_LINK_LIBRARIES(
						ovqt_plugin_gui_editor
						ovqt_plugin_core
						nelmisc
						nel3d
						nelgui
						${QT_LIBRARIES}
						${QT_QTOPENGL_LIBRARY}
						qt_property_browser
						${LUA_LIBRARIES}
						${LUABIND_LIBRARIES}
						${CURL_LIBRARIES}
						${LIBWWW_LIBRARIES}
						${LIBXML2_LIBRARIES}
						)

NL_DEFAULT_PROPS(ovqt_plugin_gui_editor "NeL, Tools, 3D: Object Viewer Qt Plugin: GUI Editor")
NL_ADD_RUNTIME_FLAGS(ovqt_plugin_gui_editor)
NL_ADD_LIB_SUFFIX(ovqt_plugin_gui_editor)

ADD_DEFINITIONS(-DGUI_EDITOR_LIBRARY ${LIBXML2_DEFINITIONS} -DQT_PLUGIN -DQT_SHARED ${QT_DEFINITIONS})

INSTALL(TARGETS ovqt_plugin_gui_editor LIBRARY DESTINATION lib RUNTIME DESTINATION bin ARCHIVE DESTINATION lib COMPONENT tools3d)

cmake_minimum_required(VERSION 3.24)
project(MicroTeX LANGUAGES CXX)

find_package(tinyxml2 REQUIRED)

# TODO MSVC /utf-8 flag?

add_library(Core
	# atom folder
	src/atom/atom_basic.cpp
	src/atom/atom_char.cpp
	src/atom/atom_impl.cpp
	src/atom/atom_matrix.cpp
	src/atom/atom_row.cpp
	src/atom/atom_space.cpp
	src/atom/colors_def.cpp
	src/atom/unit_conversion.cpp
	# box folder
	src/box/box.cpp
	src/box/box_factory.cpp
	src/box/box_group.cpp
	src/box/box_single.cpp
	# core folder
	src/core/core.cpp
	src/core/formula.cpp
	src/core/formula_def.cpp
	src/core/glue.cpp
	src/core/localized_num.cpp
	src/core/macro.cpp
	src/core/macro_def.cpp
	src/core/macro_impl.cpp
	src/core/parser.cpp
	# fonts folder
	src/fonts/alphabet.cpp
	src/fonts/font_basic.cpp
	src/fonts/font_info.cpp
	src/fonts/fonts.cpp
	# utils folder
	src/utils/string_utils.cpp
	src/utils/utf.cpp
	src/utils/utils.cpp
	# res folder
	src/res/builtin/formula_mappings.res.cpp
	src/res/builtin/symbol_mapping.res.cpp
	src/res/builtin/tex_param.res.cpp
	src/res/builtin/tex_symbols.res.cpp
	src/res/font/bi10.def.cpp
	src/res/font/bx10.def.cpp
	src/res/font/cmbsy10.def.cpp
	src/res/font/cmbx10.def.cpp
	src/res/font/cmbxti10.def.cpp
	src/res/font/cmex10.def.cpp
	src/res/font/cmmi10.def.cpp
	src/res/font/cmmi10_unchanged.def.cpp
	src/res/font/cmmib10.def.cpp
	src/res/font/cmmib10_unchanged.def.cpp
	src/res/font/cmr10.def.cpp
	src/res/font/cmss10.def.cpp
	src/res/font/cmssbx10.def.cpp
	src/res/font/cmssi10.def.cpp
	src/res/font/cmsy10.def.cpp
	src/res/font/cmti10.def.cpp
	src/res/font/cmti10_unchanged.def.cpp
	src/res/font/cmtt10.def.cpp
	src/res/font/dsrom10.def.cpp
	src/res/font/eufb10.def.cpp
	src/res/font/eufm10.def.cpp
	src/res/font/i10.def.cpp
	src/res/font/moustache.def.cpp
	src/res/font/msam10.def.cpp
	src/res/font/msbm10.def.cpp
	src/res/font/r10.def.cpp
	src/res/font/r10_unchanged.def.cpp
	src/res/font/rsfs10.def.cpp
	src/res/font/sb10.def.cpp
	src/res/font/sbi10.def.cpp
	src/res/font/si10.def.cpp
	src/res/font/special.def.cpp
	src/res/font/ss10.def.cpp
	src/res/font/stmary10.def.cpp
	src/res/font/tt10.def.cpp
	src/res/parser/font_parser.cpp
	src/res/parser/formula_parser.cpp
	src/res/reg/builtin_font_reg.cpp
	src/res/reg/builtin_syms_reg.cpp
	src/res/sym/amsfonts.def.cpp
	src/res/sym/amssymb.def.cpp
	src/res/sym/base.def.cpp
	src/res/sym/stmaryrd.def.cpp
	src/res/sym/symspecial.def.cpp

	src/latex.cpp
	src/render.cpp
)
add_library(MicroTeX::Core ALIAS Core)
list(APPEND TARGETS_TO_INSTALL Core)

target_include_directories(Core PUBLIC
	src
)

target_link_libraries(Core PUBLIC
	tinyxml2::tinyxml2
)

set_target_properties(Core PROPERTIES
	CXX_STANDARD 17
	CXX_STANDARD_REQUIRED ON
	CXX_EXTENSIONS OFF
)

option(HAVE_LOG "If enable log" OFF)
if(HAVE_LOG)
	target_compile_definitions(Core PRIVATE -DHAVE_LOG)
endif()

option(GRAPHICS_DEBUG "If enable graphics debug" OFF)
if(GRAPHICS_DEBUG)
	target_compile_definitions(Core PRIVATE -DGRAPHICS_DEBUG)
endif()

option(MEM_CHECK "If compile for memory check only" OFF)
if (MEM_CHECK)
	target_compile_definitions(Core PRIVATE -DMEM_CHECK)
endif()

# TODO optional gtk dependencies?

# TODO install

option(BUILD_GTK_CAIRO_INTEGRATION "Build GTK/Cairo integration" ON)
option(BUILD_GTK_EXAMPLE "Build GTK example, only affective if GTK/Cairo integration is enabled" ON)
if(BUILD_GTK_CAIRO_INTEGRATION)
	find_package(Fontconfig REQUIRED)
	find_package(PkgConfig REQUIRED)
	pkg_check_modules(CairoMM REQUIRED IMPORTED_TARGET cairomm-1.0)
	pkg_check_modules(GTKMM REQUIRED IMPORTED_TARGET gtkmm-3.0)

	add_library(CairoIntegration
		src/platform/cairo/graphic_cairo.cpp
	)
	add_library(MicroTeX::CairoIntegration ALIAS CairoIntegration)
	list(APPEND TARGETS_TO_INSTALL CairoIntegration)
	target_link_libraries(CairoIntegration PUBLIC
		Core
		Fontconfig::Fontconfig
		PkgConfig::CairoMM
		PkgConfig::GTKMM
	)
	target_compile_definitions(CairoIntegration PRIVATE -DBUILD_GTK=1)

	if(BUILD_GTK_EXAMPLE)
		pkg_check_modules(GSVMM REQUIRED IMPORTED_TARGET gtksourceviewmm-3.0)
		add_executable(MicroTeXGTKExample
			src/samples/gtkmm_main.cpp
		)
		target_link_libraries(MicroTeXGTKExample PRIVATE
			CairoIntegration
			PkgConfig::GSVMM
		)
		target_compile_definitions(MicroTeXGTKExample PRIVATE -DBUILD_GTK=1)
	endif()
endif()

option(BUILD_QT_INTEGRATION "Build Qt integration" ON)
option(BUILD_QT_EXAMPLE "Build Qt example, only affective if Qt integration is enabled" ON)
if(BUILD_QT_INTEGRATION)
	set(NEEDED_QT_COMPONENTS Gui)
	if(BUILD_QT_EXAMPLE)
		list(APPEND NEEDED_QT_COMPONENTS Widgets)
	endif()

	find_package(QT NAMES Qt6 Qt5 COMPONENTS ${NEEDED_QT_COMPONENTS} REQUIRED)
	find_package(Qt${QT_VERSION_MAJOR} COMPONENTS ${NEEDED_QT_COMPONENTS} REQUIRED)

	add_library(QtIntegration
		src/platform/qt/graphic_qt.cpp
	)
	add_library(MicroTeX::QtIntegration ALIAS QtIntegration)
	list(APPEND TARGETS_TO_INSTALL QtIntegration)
	target_link_libraries(QtIntegration PUBLIC
		Core
		Qt${QT_VERSION_MAJOR}::Gui
	)
	target_compile_definitions(QtIntegration PRIVATE -DBUILD_QT=1)

	if(BUILD_QT_EXAMPLE)
		# TODO there is a qml widget provided in source, build that too?
		add_executable(MicroTeXQtExample
			src/samples/qt_texwidget.cpp
			src/samples/qt_mainwindow.cpp
			src/samples/qt_main.cpp
		)
		target_link_libraries(MicroTeXQtExample PRIVATE
			QtIntegration
			Qt${QT_VERSION_MAJOR}::Widgets
		)
		target_compile_definitions(MicroTeXQtExample PRIVATE -DBUILD_QT=1)
		set_target_properties(MicroTeXQtExample PROPERTIES
			AUTOMOC ON
		)
	endif()
endif()

# TODO Skia integration
# the project's code is currently coupled to QString, so it's actually not that useful at all

if(WIN32)
	option(BUILD_GDI_INTEGRATION "Build GDI integration" ON)
	option(BUILD_WIN32_EXAMPLE "Build win32 example, only affective if and GDI integration is enabled")
	# TODO
endif()

add_library(MicroTeX INTERFACE)
add_library(MicroTeX::MicroTeX ALIAS MicroTeX)
target_link_libraries(MicroTeX INTERFACE ${TARGETS_TO_INSTALL})
list(APPEND TARGETS_TO_INSTALL MicroTeX)

# TODO install


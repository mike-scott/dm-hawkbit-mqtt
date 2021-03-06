cmake_minimum_required(VERSION 3.8.2)

# This application has its own Kconfig options.
set(KCONFIG_ROOT ${CMAKE_CURRENT_SOURCE_DIR}/Kconfig)

# CONF_FILE is prj.conf, plus (in addition to, not or) the following, if they
# exist:
#   - local.conf
#   - boards/${BOARD}.conf
#   - boards/${BOARD}-local.conf
#
# The possibly nonexistent files need to be absolute paths; otherwise, the
# behavior of EXISTS is not well-defined.
set(CONF_FILE prj.conf)
if(EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/local.conf)
  set(CONF_FILE ${CONF_FILE} " local.conf")
endif()
if(EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/boards/${BOARD}.conf)
  set(CONF_FILE ${CONF_FILE} " boards/${BOARD}.conf")
endif()
if(EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/boards/${BOARD}-local.conf)
  set(CONF_FILE ${CONF_FILE} " boards/${BOARD}-local.conf")
endif()

# DTC overlays are split into a part that is specific to the
# application image (common.overlay) and a part that is shared by both
# the application and MCUboot builds (mcuboot.overlay).
set(DTC_OVERLAY_FILE
  "${CMAKE_CURRENT_SOURCE_DIR}/common.overlay ${CMAKE_CURRENT_SOURCE_DIR}/mcuboot.overlay")
# Individual boards may also define DTS overlays.
if(EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/boards/${BOARD}.overlay)
  set(DTC_OVERLAY_FILE
    ${DTC_OVERLAY_FILE} " ${CMAKE_CURRENT_SOURCE_DIR}/boards/${BOARD}.overlay")
endif()

# Mandatory Zephyr boilerplate.
include($ENV{ZEPHYR_BASE}/cmake/app/boilerplate.cmake NO_POLICY_SCOPE)
project(NONE)

# Application "library" build configuration. TODO: move these out of this tree.
target_include_directories(app PRIVATE $ENV{ZEPHYR_BASE}/lib)

target_sources(app PRIVATE src/lib/hawkbit.c)
target_sources(app PRIVATE src/lib/product_id.c)

# Application build configuration.
target_include_directories(app PRIVATE $ENV{ZEPHYR_BASE}/tests/include)
target_include_directories(app PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/src/lib)

target_sources(app PRIVATE src/main.c)
target_sources(app PRIVATE src/blink_led.c)
target_sources(app PRIVATE src/app_work_queue.c)
target_sources(app PRIVATE src/mqtt_temperature.c)
target_sources_ifdef(CONFIG_BT               app PRIVATE src/bluetooth.c)

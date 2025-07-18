cmake_minimum_required(VERSION 3.0 FATAL_ERROR)
project(sw_daq_tofpet2)
if (NOT EXISTS ${CMAKE_BINARY_DIR}/CMakeCache.txt)
	if (NOT CMAKE_BUILD_TYPE)
		set(CMAKE_BUILD_TYPE "Release" CACHE STRING "" FORCE)
	endif()
endif()


find_program(LSB_RELEASE_EXEC lsb_release REQUIRED)
execute_process(COMMAND ${LSB_RELEASE_EXEC} -is
	OUTPUT_VARIABLE LSB_ID_SHORT
	OUTPUT_STRIP_TRAILING_WHITESPACE
)
execute_process(COMMAND ${LSB_RELEASE_EXEC} -rs
	OUTPUT_VARIABLE LSB_RELEASE_SHORT
	OUTPUT_STRIP_TRAILING_WHITESPACE
)

find_package(PythonLibs 3.6 REQUIRED)
if((${LSB_ID_SHORT} STREQUAL "Ubuntu") AND (${LSB_RELEASE_SHORT} VERSION_LESS_EQUAL "20.04"))
	find_package(Boost REQUIRED python regex)
elseif((EXISTS "/etc/redhat-release") AND (${LSB_RELEASE_SHORT} VERSION_GREATER_EQUAL "9"))
	find_package(Boost REQUIRED python39 regex)
else()
	find_package(Boost REQUIRED python3 regex)
endif()

include(CheckIncludeFiles)

include_directories(${Boost_INCLUDE_DIRS} ${PYTHON_INCLUDE_DIRS})
include_directories("/usr/include/iniparser")

link_libraries("rt" "pthread" "aio")

include_directories("src/raw_data")
include_directories("src/base")
include_directories("src/online_monitor")

if(CMAKE_BUILD_TYPE STREQUAL "Release")
set( CMAKE_C_FLAGS "${CMAKE_CXX_FLAGS} -DLINUX -march=native -O3" )
set( CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DLINUX  -march=native -O3" )
endif()

set (DAQD_SOURCES "src/daqd/daqd.cpp" "src/daqd/Client.cpp" "src/daqd/FrameServer.cpp" "src/daqd/UDPFrameServer.cpp" "src/daqd/DAQFrameServer.cpp" "src/daqd/PFP_KX7.cpp"  "src/raw_data/shm_raw.cpp" )

add_executable("daqd" ${DAQD_SOURCES})
add_executable("write_raw" "src/raw_data/write_raw.cpp" "src/raw_data/shm_raw.cpp" "src/raw_data/AsyncWriter.cpp")

execute_process(COMMAND root-config --incdir OUTPUT_VARIABLE ROOT_INCDIR)
string(STRIP ${ROOT_INCDIR} ROOT_INCDIR)
execute_process(COMMAND root-config --libs OUTPUT_VARIABLE ROOT_LIBS)
string(STRIP ${ROOT_LIBS} ROOT_LIBS)
execute_process(COMMAND root-config --cflags OUTPUT_VARIABLE ROOT_CXX_FLAGS)
string(STRIP ${ROOT_CXX_FLAGS} ROOT_CXX_FLAGS)

link_libraries(${Boost_LIBRARIES} ${PYTHON_LIBRARIES} "-liniparser" "-laio")
add_library(shm_raw_py MODULE "src/raw_data/shm_raw.cpp" "src/raw_data/shm_raw_py.cpp")
set_target_properties(shm_raw_py PROPERTIES LIBRARY_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/petsys OUTPUT_NAME shm_raw PREFIX "")

link_libraries(${Boost_LIBRARIES} ${PYTHON_LIBRARIES} "-liniparser" "-laio")
add_library(monitor_py MODULE "src/online_monitor/Monitor.cpp" "src/online_monitor/SingleValue.cpp" "src/online_monitor/Histogram1D.cpp" "src/online_monitor/monitor_py.cpp")
set_target_properties(monitor_py PROPERTIES LIBRARY_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/petsys OUTPUT_NAME monitor PREFIX "")


configure_file("src/petsys_py_lib/__init__.py" "petsys/__init__.py" COPYONLY)
configure_file("src/petsys_py_lib/tofpet2b.py" "petsys/tofpet2b.py" COPYONLY)
configure_file("src/petsys_py_lib/tofpet2c.py" "petsys/tofpet2c.py" COPYONLY)
configure_file("src/petsys_py_lib/info.py" "petsys/info.py" COPYONLY)
configure_file("src/petsys_py_lib/daqd.py" "petsys/daqd.py" COPYONLY)
configure_file("src/petsys_py_lib/bitarray_utils.py" "petsys/bitarray_utils.py" COPYONLY)
configure_file("src/petsys_py_lib/config.py" "petsys/config.py" COPYONLY)
configure_file("src/petsys_py_lib/spi.py" "petsys/spi.py" COPYONLY)
configure_file("src/petsys_py_lib/i2c.py" "petsys/i2c.py" COPYONLY)
configure_file("src/petsys_py_lib/bias.py" "petsys/bias.py" COPYONLY)
configure_file("src/petsys_py_lib/boot.py" "petsys/boot.py" COPYONLY)
configure_file("src/petsys_py_lib/fe_temperature.py" "petsys/fe_temperature.py" COPYONLY)
configure_file("src/petsys_py_lib/fe_eeprom.py" "petsys/fe_eeprom.py" COPYONLY)
configure_file("src/petsys_py_lib/fe_power.py" "petsys/fe_power.py" COPYONLY)
configure_file("src/petsys_py_lib/fe_power_8k.py" "petsys/fe_power_8k.py" COPYONLY)
configure_file("src/petsys_util/setSI53xx.py" "setSI53xx.py" COPYONLY)
configure_file("src/petsys_util/SI5326_config.txt" "SI5326_config.txt" COPYONLY)
configure_file("src/petsys_util/acquire_tdc_calibration" "acquire_tdc_calibration" COPYONLY)
configure_file("src/petsys_util/acquire_threshold_calibration" "acquire_threshold_calibration" COPYONLY)
configure_file("src/petsys_util/process_threshold_calibration" "process_threshold_calibration" COPYONLY)
configure_file("src/petsys_util/acquire_qdc_calibration" "acquire_qdc_calibration" COPYONLY)
configure_file("src/petsys_util/acquire_sipm_data" "acquire_sipm_data" COPYONLY)
configure_file("src/petsys_util/acquire_sipm_data_basic" "acquire_sipm_data_basic" COPYONLY)
configure_file("src/petsys_util/make_asic_mask" "make_asic_mask" COPYONLY)
configure_file("src/petsys_util/make_bias_calibration_table" "make_bias_calibration_table" COPYONLY)
configure_file("src/petsys_util/make_simple_bias_settings_table" "make_simple_bias_settings_table" COPYONLY)
configure_file("src/petsys_util/make_simple_channel_map" "make_simple_channel_map" COPYONLY)
configure_file("src/petsys_util/make_simple_energy_calibration_table" "make_simple_energy_calibration_table" COPYONLY)
configure_file("src/petsys_util/make_simple_disc_settings_table" "make_simple_disc_settings_table" COPYONLY)
configure_file("src/petsys_util/init_system" "init_system" COPYONLY)
configure_file("src/petsys_util/set_fem_power" "set_fem_power" COPYONLY)
configure_file("src/petsys_util/set_bias" "set_bias" COPYONLY)
configure_file("src/petsys_util/set_si53xx" "set_si53xx" COPYONLY)
configure_file("src/petsys_util/update_prom" "update_prom" COPYONLY)
configure_file("src/petsys_util/boot_sw" "boot_sw" COPYONLY)
configure_file("src/petsys_util/read_temperature_sensors" "read_temperature_sensors" COPYONLY)
configure_file("src/petsys_util/read_communication_counters" "read_communication_counters" COPYONLY)
configure_file("src/petsys_util/check_asic_links" "check_asic_links" COPYONLY)
configure_file("src/petsys_util/change_calibration_topology" "change_calibration_topology" COPYONLY)
configure_file("src/petsys_util/psqc_fetp.C" "psqc_fetp.C" COPYONLY)
configure_file("src/petsys_util/eeprom_write" "eeprom_write" COPYONLY)
configure_file("src/petsys_util/stop_acquisition" "stop_acquisition" COPYONLY)
configure_file("src/gui/start_gui" "start_gui" COPYONLY)
if((${LSB_ID_SHORT} STREQUAL "Ubuntu"))
  configure_file("src/gui/gui_layout_ubuntu.glade" "gui/gui_layout.glade" COPYONLY)
else()
  configure_file("src/gui/gui_layout.glade" "gui/gui_layout.glade" COPYONLY)
endif() 
configure_file("src/gui/logo_PETsys.png" "gui/logo_PETsys.png" COPYONLY)
configure_file("src/gui/logo_PETsys2.png" "gui/logo_PETsys2.png" COPYONLY)
configure_file("src/gui/logo_PETsys3.png" "gui/logo_PETsys3.png" COPYONLY)
configure_file("src/online_monitor/launch_monitor" "launch_monitor" COPYONLY)
configure_file("src/petsys_util/acquire_sipm_data_with_monitoring" "acquire_sipm_data_with_monitoring" COPYONLY)
configure_file("config.ini" "config.ini" COPYONLY)

include_directories( ${ROOT_INCDIR} )
link_libraries(${ROOT_LIBS})
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${ROOT_CXX_FLAGS}")

add_library ( common STATIC
	"src/base/ThreadPool.cpp"
	"src/base/SystemConfig.cpp"
	"src/raw_data/RawReader.cpp"
	"src/raw_data/shm_raw.cpp"
	"src/raw_data/AsyncWriter.cpp"
	"src/base/Instrumentation.cpp"
	"src/base/CoarseSorter.cpp"
	"src/base/ProcessHit.cpp"
	"src/base/HwTriggerSimulator.cpp"
	"src/base/SimpleGrouper.cpp"
	"src/base/CoincidenceGrouper.cpp"
	"src/base/DataFileWriter.cpp"
	"src/online_monitor/Monitor.cpp"
	"src/online_monitor/SingleValue.cpp"
	"src/online_monitor/Histogram1D.cpp"
)
add_executable("process_tdc_calibration" "src/petsys_util/process_tdc_calibration.cpp" )
target_link_libraries("process_tdc_calibration" common)

add_executable("process_qdc_calibration" "src/petsys_util/process_qdc_calibration.cpp" )
target_link_libraries("process_qdc_calibration" common)

add_executable("process_hwtrigger_empirical_calibration" "src/petsys_util/process_hwtrigger_empirical_calibration.cpp" )
target_link_libraries("process_hwtrigger_empirical_calibration" common)

add_executable("convert_raw_to_raw" "src/petsys_util/convert_raw_to_raw.cpp" )
target_link_libraries("convert_raw_to_raw" common)

add_executable("convert_raw_to_group" "src/petsys_util/convert_raw_to_group.cpp" )
target_link_libraries("convert_raw_to_group" common)

add_executable("convert_raw_to_coincidence" "src/petsys_util/convert_raw_to_coincidence.cpp" )
target_link_libraries("convert_raw_to_coincidence" common)

add_executable("convert_raw_to_singles" "src/petsys_util/convert_raw_to_singles.cpp" )
target_link_libraries("convert_raw_to_singles" common)

add_executable("print_raw" "src/petsys_util/print_raw.cpp" )
target_link_libraries("print_raw" common)

add_executable("merge_raw" "src/petsys_util/merge_raw.cpp" )
target_link_libraries("merge_raw" common)

add_executable("online_monitor" "src/online_monitor/online_monitor.cpp")
target_link_libraries("online_monitor" common)

add_executable("online_process" "src/online_monitor/online_process.cpp")
target_link_libraries("online_process" common)

# Port headers (Fault.h, IT_Client.h). Header-only: FaultHandler() is defined
# by DelegateMQ's fault port.
add_library(PortLib INTERFACE)

# Include directories for the library
target_include_directories(PortLib INTERFACE "${CMAKE_CURRENT_SOURCE_DIR}")

# IT_Client.h includes DelegateMQ.h
target_link_libraries(PortLib INTERFACE DelegateMQ::DelegateMQ)

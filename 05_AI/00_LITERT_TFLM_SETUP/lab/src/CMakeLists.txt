# main/*.c and main/*.cc are picked up automatically once you copy the
# hello_world example here in Step 4 of the KR doc (see idf_component_register
# below, which globs whatever source files exist in this directory).

idf_component_register(
    SRC_DIRS "."
    INCLUDE_DIRS "."
)

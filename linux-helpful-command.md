

### Check for function is exported

You can use `nm` command to check if symbol is exported in the library - `nm -C libname.so | grep "gpiod_line_set_direction_input"`. If there is no output for this function, then check for some other function of your library - if there is output for other function, then the function isn't exported.

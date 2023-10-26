# How to debug GPGPU-Sim

## Use GDB

When debuging GPGPU-Sim, the first thing is compile the whole project under `debug` environment, this could be done at the beginning using `source setup_environment debug`. Then compile the targeted application program using `nvcc -g` flag. Now use gdb to debug the program, by simply `gdb prog`. 

As for making break point, one method is break at file line level, like `b gpgpusim_entrypoint.cc:199`, which is the `struct _cuda_device_id *gpgpu_context::GPGPUSim_Init()` function, it's the logical entry point of the whole simualtion.

Inside gdb, we could use `info frame` to check the current stack and previous stack, use `up` could jumped to previous stack frame, and `domn` could jump to the next. It's useful if we want to track the function calls.

## Add Log library

It's also possible to add a log module into the GPGPU-Sim project. For example, if we would like to integrate the simple [`log.c`](https://github.com/rxi/log.c), first we could create a new directory inside the `src`, named     `liblog`. Then clone the `log.c` project into that folder. To simplyfy the makefile structure, just move all the `*.c` and `.h` files in the `liblog` folder. Then copy a makefile from any same level folder, like `gpgpu-sim`, change the following lines:

```shell
OUTPUT_DIR=$(SIM_OBJ_FILES_DIR)/liblog

liblog.a:$(OBJS)
	ar rcs  $(OUTPUT_DIR)/liblog.a $(OBJS)
```

remove other unrelated `.o` genegration, later this make file could be called from `src` level makefile. Now edit the `src` level Makefile, add these lines:

```shell
gpu_uarch_simlib:
	make   -C ./gpgpu-sim

# for the new module
liblog:
	make -C ./liblog
```

Then we could modify the highest level Makefile, add some lines:

```shell
# link to the final .libcudart.so
$(SIM_LIB_DIR)/libcudart.so: makedirs $(LIBS) cudalib
    # ...
	$(SIM_OBJ_FILES_DIR)/liblog/*.o \
    #...
    -o $(SIM_LIB_DIR)/libcudart.so

# create corresponding build directory
liblig: makedirs 
	$(MAKE) -C ./src/liblig/ depend
	$(MAKE) -C ./src/liblig/

makedirs:
	# ...
    if [ ! -d $(SIM_OBJ_FILES_DIR)/liblog ]; then mkdir -p $(SIM_OBJ_FILES_DIR)/liblog; fi;
	# ...
```

Now if we want to use the log module in `cuda-sim/cuda-sim.cc`, first include the coresponding header file,
and then modify the makefile to include the log module object file, by adding these lines:

```shell
OBJS	:= ... $(OUTPUT_DIR)/cuda_device_runtime.o $(OUTPUT_DIR)/../liblog/log.o
#...
$(OUTPUT_DIR)/../liblog/log.o:
	make -C ../liblog
```

Then we could re-compile the simulator, and the log module could work.
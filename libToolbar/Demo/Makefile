# Application Name
NAME = Demo
# Resources
RSRCS= AppResource.rsrc Icon.rsrc
# Libraries
LIBS= be root ../libToolbar.a
# Library path
LIBPATHS= 
# System include path
SYSTEM_INCLUDE_PATHS =
# Local include path 
LOCAL_INCLUDE_PATHS = ../
# OPTIMIZE
OPTIMIZE= FULL
# Defines
DEFINES=
# Warnings
WARNINGS = ALL
# Sources
SRCS= HApp.cpp\
		HWindow.cpp\
		main.cpp
#--------------------------------------------------------
#	determine wheather running on x86 or ppc
MACHINE=$(shell uname -m)
ifeq ($(MACHINE), BePC)
	CPU = x86
else
	CPU = ppc
endif
#	set the directory where object files and binaries will be created
	OBJ_DIR		:= obj.$(CPU)

#	create some default settings
ifeq ($(NAME), )
	NAME = NameThisApp
endif
# 	specify that the binary should be created in the object directory
	TARGET		:= "$(NAME)"

#	specify the mimeset tool
	MIMESET		:= mimeset

#	specify the tools for adding and removing resources
	XRES		= xres
# 	platform specific settings

#	x86 Settings
ifeq ($(CPU), x86)
#	set the compiler and compiler flags
	CC		=	gcc

#	SETTING: set the CFLAGS for each binary type	
	ifeq ($(TYPE), DRIVER)
		CFLAGS	+= -D_KERNEL_MODE=1 -no-fpic
	else
		CFLAGS +=
	endif

#	SETTING: set the proper optimization level
	ifeq ($(OPTIMIZE), FULL)
		OPTIMIZER	= -O3
	else
	ifeq ($(OPTIMIZE), SOME)
		OPTIMIZER	= -O1
	else
	ifeq ($(OPTIMIZE), NONE)
		OPTIMIZER	= -O0
	else
#		OPTIMIZE not set so set to full
		OPTIMIZER	= -O3
	endif
	endif
	endif
		

	
#	SETTING: set proper debugger flags
	ifeq ($(DEBUGGER), TRUE)
		DEBUG += -g
		OPTIMIZER = -O0
	endif	
	
	CFLAGS += $(OPTIMIZER) $(DEBUG)

#	SETTING: set warning level
	ifeq ($(WARNINGS), ALL)
		CFLAGS += -Wall -Wno-multichar -Wno-ctor-dtor-privacy
	else
	ifeq ($(WARNINGS), NONE)
		CFLAGS += -w
	endif
	endif

#	set the linker and linker flags
	LD			= gcc
	LDFLAGS		+= $(DEBUG)

#	SETTING: set linker flags for each binary type
	ifeq ($(TYPE), APP)
		LDFLAGS += -Xlinker -soname=_APP_
	else
	ifeq ($(TYPE), SHARED)
		LDFLAGS += -nostart -Xlinker -soname=$(NAME)
	else
	ifeq ($(TYPE), DRIVER)
		LDFLAGS += -nostdlib /boot/develop/lib/x86/_KERNEL_
	endif 
	endif 
	endif 

	
else

#	ppc Settings
ifeq ($(CPU), ppc)
#	set the compiler and compiler flags
	CC		=	mwcc
	CFLAGS	+=	

#	SETTING: set the proper optimization level
	ifeq ($(OPTIMIZE), FULL)
		OPTIMIZER	= -O7
	else
	ifeq ($(OPTIMIZE), SOME)
		OPTIMIZER	= -O3
	else
	ifeq ($(OPTIMIZE), NONE)
		OPTIMIZER	= -O0
	else
#		OPTIMIZE not set so set to full
		OPTIMIZER	= -O7
	endif
	endif
	endif

	#set the proper debugger settings
	ifeq ($(DEBUGGER), TRUE)
		DEBUG += -g
	endif	

	CFLAGS += $(OPTIMIZER) $(DEBUG)

#	SETTING: set warning level
	ifeq ($(WARNINGS), ALL)
		CFLAGS += -w on -requireprotos
	else
	ifeq ($(WARNINGS), NONE)
		CFLAGS += -w off
	endif
	endif

	# clear the standard environment variable
	# now there are no standard libraries to link against
	BELIBFILES=

#	set the linker and linker flags
	LD			= mwldppc

#	SETTING: set linker flags for each binary type
	ifeq ($(TYPE), APP)
		LDFLAGS += 
	else
	ifeq ($(TYPE), SHARED)
		LDFLAGS += 	-xms 
	endif
	endif

	ifeq ($(TYPE), DRIVER)
		LDFLAGS += -nodefaults \
					-export all \
					-G \
					/boot/develop/lib/ppc/glue-noinit.a \
					/boot/develop/lib/ppc/_KERNEL_
	else
		LDFLAGS +=	-export pragma \
					-init _init_routine_ \
					-term _term_routine_ \
					-lroot \
					/boot/develop/lib/ppc/glue-noinit.a \
					/boot/develop/lib/ppc/init_term_dyn.o \
					/boot/develop/lib/ppc/start_dyn.o 
	endif			
	
	
#	SETTING: output symbols in an xMAP file
	ifeq ($(SYMBOLS), TRUE)
		LDFLAGS += -map $(TARGET).xMAP
	endif

#	SETTING: output debugging info to a .SYM file
	ifeq ($(DEBUGGER), TRUE)
		LDFLAGS += -g -osym $(TARGET).SYM
	endif

endif
endif


# psuedo-function for converting a list of source files in SRCS variable
# to a corresponding list of object files in $(OBJ_DIR)/xxx.o
# The "function" strips off the src file suffix (.ccp or .c or whatever)
# and then strips of the directory name, leaving just the root file name.
# It then appends the .o suffix and prepends the $(OBJ_DIR)/ path
define SRCS_LIST_TO_OBJS
	$(addprefix $(OBJ_DIR)/, $(addsuffix .o, $(foreach file, $(SRCS), \
	$(basename $(notdir $(file))))))
endef

OBJS = $(SRCS_LIST_TO_OBJS)

# create a unique list of paths to our sourcefiles
SRC_PATHS += $(sort $(foreach file, $(SRCS), $(dir $(file))))

# add source paths to VPATH if not already present
VPATH :=
VPATH += $(addprefix :, $(subst  ,:, $(filter-out $($(subst, :, ,$(VPATH))), $(SRC_PATHS))))

#	SETTING: build the local and system include paths
ifeq ($(CPU), x86)
	LOC_INCLUDES = $(foreach path, $(SRC_PATHS) $(LOCAL_INCLUDE_PATHS), $(addprefix -I, $(path)))
	SYS_INCLUDES += -I-
	SYS_INCLUDES += $(foreach path, $(SYSTEM_INCLUDE_PATHS), $(addprefix -I, $(path)))
else
ifeq ($(CPU), ppc)
	LOC_INCLUDES = $(foreach path, $(SRC_PATHS) $(LOCAL_INCLUDE_PATHS), $(addprefix -I, $(path)))
	SYS_INCLUDES += -i-
	SYS_INCLUDES += $(foreach path, $(SYSTEM_INCLUDE_PATHS), $(addprefix -i , $(path)))
endif
endif

INCLUDES = $(LOC_INCLUDES) $(SYS_INCLUDES)

# SETTING: add the -L prefix to all library paths to search
LINK_PATHS = $(foreach path, $(SRC_PATHS) $(LIBPATHS), \
	$(addprefix -L, $(path)))

#	SETTING: specify the additional libraries to link against
#	if the libraries have a .so or .a prefix, or if they are _APP_ or _KERNEL_
#	simply add them to the list
LINK_LIBS += $(filter %.so %.a _APP_ _KERNEL_, $(LIBS))
#	if the libraries do not have suffixes and are not _APP_ or _KERNEL_
#	prepend -l to each name: be becomes -lbe
LINK_LIBS += $(foreach lib, $(filter-out %.so %.a _APP_ _KERNEL_, $(LIBS)), $(addprefix -l, $(lib)))

# add to the linker flags 
LDFLAGS += $(LINK_PATHS)  $(LINK_LIBS)

#	SETTING: add the defines to the compiler flags
CFLAGS += $(foreach define, $(DEFINES), $(addprefix -D, $(define)))

#	SETTING: add the additional compiler flags
CFLAGS += $(COMPILER_FLAGS)

#	SETTING: add the additional linker flags
LDFLAGS += $(LINKER_FLAGS)

#	SETTING: use the archive tools if building a static library
#	otherwise use the linker
ifeq ($(TYPE), STATIC)
	BUILD_LINE = ar -cru "$(TARGET)" $(OBJS)
else
	BUILD_LINE = $(LD) -o $@ $(OBJS) $(LDFLAGS)
endif

#	create the resource instruction
	ifeq ($(RSRCS), )
		DO_RSRCS :=
	else
		DO_RSRCS := $(XRES) -o "$(TARGET)" $(RSRCS)
	endif


#	define the actual work to be done	
default: $(TARGET)

$(TARGET):	$(OBJ_DIR) $(OBJS) $(RSRCS)
		$(BUILD_LINE)
		$(DO_RSRCS)
		$(MIMESET) -f $@


#	rule to create the object file directory if needed
$(OBJ_DIR)::
	@[ -d $(OBJ_DIR) ] || mkdir $(OBJ_DIR) > /dev/null 2>&1

$(OBJ_DIR)/%.o : %.c
	$(CC) -c $< $(INCLUDES) $(CFLAGS) -o $@
$(OBJ_DIR)/%.o : %.cpp
	$(CC) -c $< $(INCLUDES) $(CFLAGS) -o $@
$(OBJ_DIR)/%.o : %.cp
	$(CC) -c $< $(INCLUDES) $(CFLAGS) -o $@
$(OBJ_DIR)/%.o : %.cc
	$(CC) -c $< $(INCLUDES) $(CFLAGS) -o $@
$(OBJ_DIR)/%.o : %.C
	$(CC) -c $< $(INCLUDES) $(CFLAGS) -o $@
$(OBJ_DIR)/%.o : %.CC
	$(CC) -c $< $(INCLUDES) $(CFLAGS) -o $@
$(OBJ_DIR)/%.o : %.CPP
	$(CC) -c $< $(INCLUDES) $(CFLAGS) -o $@

#	rules to handle lex/flex and yacc/bison files

$(OBJ_DIR)/%.o: %.l
	flex $<
	$(CC) -c $(INCLUDES) $(CFLAGS) lex.yy.c -o $@
$(OBJ_DIR)/%.o: %.y
	bison -d -y $<
	$(CC) -c $(INCLUDES) $(CFLAGS) y.tab.c -o $@

#	empty rule. Things that depend on this rule will always get triggered
FORCE:

#	The generic clean command. Delete everything in the object folder.
clean :: FORCE
	-rm -rf $(OBJ_DIR)

#	remove just the application from the object folder
rmapp ::
	-rm -f "$(TARGET)"

# dependency generation

ifeq ($(CPU), ppc)
DEPFLAG = -make
SEDSCRIPT = sed -e'/\/boot\/develop\/headers\/.*\\/d' -e's%^.*/boot/develop/headers/.*% %g' -e's%^.*/\([^/]*\.o\)%$$(OBJ_DIR)/\1%' -e"s%`pwd`/%%g"
else
DEPFLAG = -MM
SEDSCRIPT = sed -e's%^\(.*\.o\)%$$(OBJ_DIR)/\1%'
endif

depend :: FORCE
	$(CC) $(INCLUDES) $(CFLAGS) $(DEPFLAG) $(SRCS) | $(SEDSCRIPT) > .dependencies

cleandepend :: FORCE
	-rm -f .dependencies

# make it easy to install drivers for testing
USER_BIN_PATH = /boot/home/config/add-ons/kernel/drivers/bin
USER_DEV_PATH = /boot/home/config/add-ons/kernel/drivers/dev

driverinstall ::
ifeq ($(TYPE), DRIVER)
	copyattr --data $(OBJ_DIR)/$(NAME) $(USER_BIN_PATH)/$(NAME)
	mkdir -p $(USER_DEV_PATH)/$(DRIVER_PATH)
	ln -sf $(USER_BIN_PATH)/$(NAME) $(USER_DEV_PATH)/$(DRIVER_PATH)/$(NAME)
endif

-include .dependencies

---
title: Makefile简记
categories: Makefile C Linux
tags: Makefile C Linux
description: Makefile简记
---
# 使h文件变动，重新编译
- 生成.d文件
```
DEPS := $(patsubst %.o,%.d, $(OBJS))
-include $(DEPS)
.c.o:
    $(CC) $(CFLAGS)$(LDFLAGS) -c $< $(INCS)  -MMD -MP -o $@
```
# 所有.o .d文件放到指定文件夹下
- 增加VPATH变量
```
VPATH :=$(dir $(SRCS))
OBJS += $(addprefix $(OBJ_PATH)/,$(notdir $(addsuffix .o, $(basename  $(SRCS)))))
$(OBJ_PATH)/%.o:%.c
	$(CC) $(CFLAGS)$(LDFLAGS) -c $< $(INCS)  -MMD -MP -o $@
```
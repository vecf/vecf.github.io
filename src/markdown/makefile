.PHONY: all clean

SRCS := $(wildcard *.md)
OBJS := $(SRCS:.md=.html)
CFLAGS := -s -M pagetitle="title" -c template.css
CC := pandoc

all: $(OBJS) 

# compilation of individual source files
%.html : %.md
	$(CC) $(CFLAGS) -o $@ $<

clean:
	rm $(OBJS)

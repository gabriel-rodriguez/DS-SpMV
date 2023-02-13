ifndef POLYTEST
POLYTEST=-DPOLYBENCH_PAPI
endif
POLYFLAGS=${POLYTEST} -DPOLYBENCH_CACHE_SIZE="(65*1024)"

CC=gcc
CFLAGS=-Ofast -g -march=native -mtune=native -ftree-vectorize -ftree-loop-vectorize -fvect-cost-model=unlimited -fsimd-cost-model=unlimited -fprefetch-loop-arrays -D_POSIX_C_SOURCE=200112L -Wno-incompatible-pointer-types -I /home/gabriel/workspace/intel/oneapi/mkl/2022.0.2/include -I /home/gabriel/workspace/intel/oneapi/vtune/2023.0.0/include/ -DREPS=100
#CFLAGS=-O3 -fno-tree-vectorize -march=native
#CFLAGS=-O0 -g -march=native
MKLROOT=/home/gabriel/workspace/intel/oneapi/mkl/2022.0.2/

MACVETH=/home/gabriel/workspace/MACVETH-test4/build/macveth
MACVETH_OPTS=--mvmarch=alderlake --misa=avx2 --simd-cost-model=unlimited --min-redux-size=4 --redux-win-size=-1

.PRECIOUS: %.S %.o %.mv.c
.SUFFIXES:

BINARIES=spmv_regular

all: ${BINARIES}

%: %.c spmv.h polybench.c
#	${CC} ${CFLAGS} ${POLYFLAGS} -I . -B/usr/share/libhugetlbfs -Wl,--hugetlbfs-align -o $@ $^ -lrbio -lm -L/home/gabriel/workspace/papi-install/lib -lpapi
	${CC} ${CFLAGS} ${POLYFLAGS} -I . -B/usr/share/libhugetlbfs -Wl,--hugetlbfs-align -o $@ $^ -lrbio -lm -L/home/gabriel/workspace/papi-install/lib -lpapi -L${MKLROOT}/lib/intel64 -lmkl_intel_lp64 -lmkl_sequential -lmkl_core -lpthread -lm -ldl

%.S: %.c
	${CC} ${CFLAGS} ${POLYFLAGS} -I . -S $< -o $@

%.o: %.S
	${CC} ${CFLAGS} ${POLYFLAGS} -c -o $@ $<
	objdump -d $@ > $@.dump
	PYTHONOPTIMIZE=2 python2 smart_fix_asm.py $< $@.dump $<.tmpfix.S
	${CC} ${CFLAGS} ${POLYFLAGS} -c -o $@ $<.tmpfix.S
	rm $<.tmpfix.S $@.dump
#	gcc -c -o $@ $<

.SECONDEXPANSION:
%.ast: %.ast.c spmv_poly.c spmv.h polybench.c $$(subst .ast.c,.ast.o,$$(wildcard %.fragments-*.ast.c))
	${CC} ${CFLAGS} ${POLYFLAGS} -I . -B/usr/share/libhugetlbfs -Wl,--hugetlbfs-align -o $@ $^ -lrbio -lm -lpapi
	$(eval $@_TMP := $(shell mktemp -d))
	$$(nm $@ |grep kernel_spmv|cut -d " " -f 1|awk '{val="0x"$$1;print strtonum(val);}'|sort -n > $@_TMP)
	TEXT_START=$$(head -n 1 < $@_TMP)&& \
	TEXT_END=$$(tail -n 1 < $@_TMP)&& \
	TEXT_SEGMENT_SIZE=$$(bc <<< $${TEXT_END}-$${TEXT_START})&& \
	echo "Recompiling with -DTEXT_SEGMENT_SIZE=$${TEXT_SEGMENT_SIZE}"&& \
	${CC} ${CFLAGS} ${POLYFLAGS} -I . -B/usr/share/libhugetlbfs -Wl,--hugetlbfs-align -DTEXT_SEGMENT_SIZE=$${TEXT_SEGMENT_SIZE} -o $@ $^ -lrbio -lm -L/home/gabriel/workspace/papi-install/lib -lpapi
	$$(rm $@_TMP)

.SECONDEXPANSION:
%.mv: %.ast.c spmv_poly.c polybench.c $$(subst .ast.c,.mv.o,$$(wildcard %.fragments-*.ast.c)) /home/gabriel/workspace/intel/oneapi/vtune/2023.0.0/lib64/libittnotify.a
	${CC} ${CFLAGS} ${POLYFLAGS} -I . -B/usr/share/libhugetlbfs -Wl,--hugetlbfs-align -o $@ $^ -lrbio -lm -lpapi
	$(eval $@_TMP := $(shell mktemp -d))
	$$(nm $@ |grep kernel_spmv|cut -d " " -f 1|awk '{val="0x"$$1;print strtonum(val);}'|sort -n > $@_TMP)
	TEXT_START=$$(head -n 1 < $@_TMP)&& \
	TEXT_END=$$(tail -n 1 < $@_TMP)&& \
	TEXT_SEGMENT_SIZE=$$(bc <<< $${TEXT_END}-$${TEXT_START})&& \
	echo "Recompiling with -DTEXT_SEGMENT_SIZE=$${TEXT_SEGMENT_SIZE}"&& \
	${CC} ${CFLAGS} ${POLYFLAGS} -I . -B/usr/share/libhugetlbfs -Wl,--hugetlbfs-align -DTEXT_SEGMENT_SIZE=$${TEXT_SEGMENT_SIZE} -o $@ $^ -lrbio -lm -L/home/gabriel/workspace/papi-install/lib -lpapi
	$$(rm $@_TMP)

%.mv.c: %.ast.c
	${MACVETH} ${MACVETH_OPTS} $^ -o $@ -- -I/usr/include

clean:
	rm -f ${BINARIES}

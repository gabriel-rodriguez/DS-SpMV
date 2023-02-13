CC=gcc
CFLAGS=-O3 -fno-tree-vectorize
CFLAGS=-O3 -ftree-vectorize -ftree-loop-vectorize -fvect-cost-model=unlimited -fsimd-cost-model=unlimited -fprefetch-loop-arrays -march=native -std=c99 ${POLYFLAGS}
#CFLAGS=-O3 -fno-tree-vectorize -march=native
#CFLAGS=-O3 -fopenmp -ftree-vectorize -ftree-loop-vectorize
#CFLAGS=-O0 -g
CC=icc
#CFLAGS=-O3 -qopt-report-phase=vec -D_Float128=__float128
##CFLAGS=-O3 -no-vec -D_Float128=__float128
POLYFLAGS=-DEXTRALARGE_DATASET ${POLYTEST}
#CFLAGS=-O3 -xSKYLAKE -D_Float128=__float128 -mkl=sequential -qopt-report=5 -qoverride-limits -vec-threshold0 ${POLYFLAGS}
#CFLAGS=-O2 -xSKYLAKE -D_Float128=__float128 -mkl=sequential -qopt-report=5 -qoverride-limits -vec-threshold0 ${POLYFLAGS}
CFLAGS=-O2 -xSKYLAKE -D_Float128=__float128 -mkl=sequential -qoverride-limits -vec-threshold0 ${POLYFLAGS}
#CFLAGS=-O2 -xBROADWELL -Ofast -axAVX2 -qoverride-limits -vec-threshold0 ${POLYFLAGS}
#CFLAGS=-O3 -qopenmp -D_Float128=__float128 -mkl
#CFLAGS=-g -O0 -D_Float128=__float128 -xSKYLAKE -mkl=sequential -qopt-report=5 -qoverride-limits -vec-threshold0 ${POLYFLAGS}

POCC=~/workspace/pocc-devel/bin/pocc
POCC_OPTS=--no-candl --vectorizer --pragmatizer --past-hoist-lb
#POCC_OPTS=--vectorizer --pragmatizer --past-hoist-lb
#POCC_OPTS=--pragmatizer --past-hoist-lb
#POCC_OPTS=--pluto --pluto-tile --pluto-parallel --pluto-prevector --pragmatizer
#POCC_OPTS=--ponos

.PRECIOUS: %.c %.cookies.S %.cookies.o
.SUFFIXES:

BINARIES=spmv_regular spmv_mkl spmv_ie_mkl

all: ${BINARIES}

%: %.c spmv.h
	${CC} ${CFLAGS} ${POLYFLAGS} -I . polybench.c $< -o $@ -lrbio -lm -lpapi

#%.o: %.c
#	${CC} ${CFLAGS} ${POLYFLAGS} -I . -S $< -o orig.S
#	python2 fix_asm.py orig.S fixed.S
#	gcc -c -o $@ fixed.S
#	rm orig.S fixed.S

%.cookies.S: %.cookies.c
	${CC} ${CFLAGS} ${POLYFLAGS} -I . -S $< -o $@

%.cookies.o: %.cookies.S
	PYTHONOPTIMIZE=2 python2 fix_asm.py $< fixed.S
	gcc -c -o $@ fixed.S
	rm fixed.S

%.debug: %.cookies.o spmv_poly.c spmv.h polybench.c
	${CC} ${CFLAGS} -g ${POLYFLAGS} -I . ${TMPFILE} -o $@ $^ -lrbio -lm -lpapi

%.cookies: %.cookies.o spmv_poly.c spmv.h polybench.c
	${CC} ${CFLAGS} -B/usr/share/libhugetlbfs ${POLYFLAGS} -I . -Wl,--hugetlbfs-link=BDT ${TMPFILE} -o $@ $^ -lrbio -lm -lpapi -lhugetlbfs
#	${CC} ${CFLAGS} -B/usr/share/libhugetlbfs ${POLYFLAGS} -I . -Wl,--hugetlbfs-align ${TMPFILE} -o $@ $^ -lrbio -lm -lpapi -lhugetlbfs

%.pocc.c: ../ufl_matrices/%.scop
	${POCC} --read-scop ${POCC_OPTS} $< -o $@

clean:
	rm -f ${BINARIES} *.pocc.c

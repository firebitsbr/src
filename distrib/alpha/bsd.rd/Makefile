#	$OpenBSD: Makefile,v 1.23 2017/01/27 17:59:09 natano Exp $

TOP=		${.CURDIR}/..

.include "${TOP}/Makefile.inc"
IMAGE=		ramdisk${REV}.fs

STRIP?=		strip
GZIPCMD?=	gzip
GZIPFLAGS?=	-9fn

RAMDISK=	RAMDISKBIG

.include "${TOP}/inst-common/Makefile.inc"

LISTS+=		${.CURDIR}/list.local

all: bsd.rd

DISKTYPE=	rdroot
MAKEFSARGS_RD=	-o disklabel=${DISKTYPE},minfree=0,density=8192

bsd.rd:	bsd.rd_unz
	objcopy -Sg -R .comment bsd.rd_unz bsd.strip
	${STRIP} bsd.strip
	${GZIPCMD} -c ${GZIPFLAGS} bsd.strip > bsd.rd

bsd.rd_unz: bsd ${IMAGE} rdsetroot
	cp bsd bsd.rd_unz
	${.OBJDIR}/rdsetroot bsd.rd_unz ${IMAGE}

bsd:
	cd ${.CURDIR}/../../../sys/arch/alpha/compile/${RAMDISK} && \
	    su ${BUILDUSER} -c '${MAKE} config && ${MAKE} clean && env COPTS=-Os ${MAKE}'
	cp -p ${.CURDIR}/../../../sys/arch/alpha/compile/${RAMDISK}/obj/bsd bsd

${IMAGE}: ${CBIN}
	rm -rf $@.d
	install -d -o root -g wheel $@.d
	mtree -def ${MTREE} -p $@.d -u
	TOPDIR=${TOP} CURDIR=${.CURDIR} OBJDIR=${.OBJDIR} REV=${REV} \
	    TARGDIR=$@.d UTILS=${UTILS} RELEASEDIR=${RELEASEDIR} \
	    sh ${UTILS}/runlist.sh ${LISTS}
	rm $@.d/${CBIN}
	makefs ${MAKEFSARGS_RD} $@ $@.d

rdsetroot: ${TOP}/../common/elfrdsetroot.c
	${HOSTCC} ${HOSTCFLAGS} -o rdsetroot \
	    ${TOP}/../common/elfrdsetroot.c ${TOP}/../common/elf32.c \
	    ${TOP}/../common/elf64.c

.ifdef RELEASEDIR
beforeinstall:
	cp bsd.rd ${RELEASEDIR}
.endif

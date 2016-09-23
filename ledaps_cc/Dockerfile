#docker run --rm -v <path to directory with ancilliary data>:/opt/ledaps -v <path to local directory with data>:/data -v <path to directory for results>:/results name_image <path to source data in docker/L*.*> <path to ancillary data in docker>
  
#Example:
#docker run -v /Users/ledaps_anc:/opt/ledaps -v /Users/data:/data -v /Users/results:/results name_image /opt/ledaps /data/LE70210492014004EDC00.tar.bz /results
#FROM ubuntu:14.04
FROM geodata/gdal:1.11.2
MAINTAINER Erick Palacios <epalacios@conabio.gob.mx>
USER root
RUN apt-get update -y && \
	apt-get install -y \
	zlib1g zlib1g-dev libtiff5 libtiff5-dev \
	libgeotiff2 libgeotiff-dev libxml2 libxml2-dev \
	g++ gfortran bison byacc flex csh make \
	nano subversion git curl

ENV USERNAME=root
RUN mkdir /usr/local/hdf4
RUN chown $USERNAME /usr/local/hdf4

RUN cd /usr/local/hdf4 && curl http://www.hdfgroup.org/ftp/HDF/HDF_Current/src/hdf-4.2.12.tar.gz -O -L && tar -xzvf hdf-4.2.12.tar.gz
#RUN cd /usr/local/hdf4 && curl https://www.hdfgroup.org/ftp/HDF/previous/src/hdf-4.2.11.tar.gz -O && tar -xzvf hdf-4.2.11.tar.gz
RUN cd /usr/local/hdf4/hdf-4.2.12 && ./configure && make check install && ldconfig

RUN mkdir /usr/local/hdf-eos
RUN chown $USERNAME /usr/local/hdf-eos
RUN cd /usr/local/hdf-eos && curl ftp://edhs1.gsfc.nasa.gov/edhs/hdfeos/latest_release/HDF-EOS2.19v1.00.tar.Z -O && tar -xzvf HDF-EOS2.19v1.00.tar.Z
RUN cd /usr/local/hdf-eos/hdfeos && ./configure -enable-install-include CC=/usr/local/hdf4/hdf-4.2.12/hdf4/bin/h4cc && make check install && ldconfig

ENV HDFEOS_GCTPINC="/usr/local/hdf-eos/hdfeos/hdfeos2/include"
ENV HDFEOS_GCTPLIB="/usr/local/hdf-eos/hdfeos/hdfeos2/lib"
ENV TIFFINC="/usr/include/x86_64-linux-gnu/"
ENV TIFFLIB="/usr/lib/x86_64-linux-gnu/"
ENV GEOTIFF_INC="/usr/include/geotiff/"
ENV GEOTIFF_LIB="/usr/lib/"
ENV HDFINC="/usr/local/hdf4/hdf-4.2.12/hdf4/include/"
ENV HDFLIB="/usr/local/hdf4/hdf-4.2.12/hdf4/lib/"
ENV HDFEOS_INC="/usr/local/hdf-eos/hdfeos/include/"
ENV HDFEOS_LIB="/usr/local/hdf-eos/hdfeos/hdfeos2/lib"
ENV JPEGINC="/usr/include/"
ENV JPEGLIB="/usr/lib/x86_64-linux-gnu/"
ENV XML2INC="/usr/include/libxml2/"
ENV XML2LIB="/usr/lib/x86_64-linux-gnu/"
ENV ZLIBINC="/usr/include/"
ENV ZLIBLIB="/usr/lib/x86_64-linux-gnu/"
ENV JBIGINC="/usr/include/"
ENV JBIGLIB="/usr/lib/x86_64-linux-gnu/"


RUN cd /usr/local/ && git clone https://github.com/USGS-EROS/espa-product-formatter.git espa-common
RUN mkdir -p /usr/local/espa-tools
ENV PREFIX="/usr/local/espa-tools/"


#RUN cd /usr/local/espa-tools && mkdir static_data && cd static_data && curl http://espa.cr.usgs.gov/downloads/auxiliaries/land_water_poly/land_no_buf.ply.gz -O && gunzip land_no_buf.ply.gz
RUN cd /usr/local/espa-tools && mkdir static_data && cd static_data && curl http://edclpdsftp.cr.usgs.gov/downloads/auxiliaries//land_water_poly/land_no_buf.ply.gz -O && gunzip land_no_buf.ply.gz
ENV ESPA_LAND_MASS_POLYGON="$PREFIX/static_data/land_no_buf.ply"

ENV ESPAINC="/usr/local/espa-tools/include"
ENV ESPALIB="/usr/local/espa-tools/lib"
RUN cd /usr/local/espa-common/ && make install && ldconfig


#Surface reflectances...

RUN cd /usr/local && mkdir espa-surface && git clone https://github.com/USGS-EROS/espa-surface-reflectance.git espa-surface/
#ENV LEDAPS_AUX_DIR="/opt/ledaps"
ADD Makefile /usr/local/espa-surface/ledaps/ledapsSrc/src/lndsr/
RUN cd /usr/local/espa-surface/ledaps/ledapsSrc/src && make && make install
ENV BIN="/usr/local/espa-tools/bin"
RUN cp /usr/local/espa-surface/ledaps/ledapsSrc/scripts/do_ledaps.csh $BIN
RUN cp /usr/local/espa-surface/ledaps/ledapsSrc/scripts/do_ledaps.py $BIN
RUN cp /usr/local/espa-surface/ledaps/ledapsSrc/scripts/ncdump $BIN

#Auxilliary files...

ENV NCDF4INC="/usr/include/"

ENV NCDF4LIB="/usr/lib"

ENV ESPA_SCHEMA=/usr/local/espa-common/schema/espa_internal_metadata_v2_0.xsd

RUN cp -r /usr/include/hdf/* /usr/local/espa-surface/ledaps/ledapsAncSrc

RUN cd /usr/local/espa-surface/ledaps/ledapsAncSrc && make && make install

VOLUME ["/opt/ledaps", "/data", "/results"]

#ADD shell_script /opt/shell_script

WORKDIR /results

#ENTRYPOINT ["/opt/shell_script"]


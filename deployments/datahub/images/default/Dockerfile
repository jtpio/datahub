FROM buildpack-deps:focal-scm

ENV CONDA_DIR /opt/conda

# Set up common env variables
ENV TZ=America/Los_Angeles
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

ENV LC_ALL en_US.UTF-8
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US.UTF-8
ENV DEBIAN_FRONTEND=noninteractive
ENV NB_USER jovyan
ENV NB_UID 1000


RUN adduser --disabled-password --gecos "Default Jupyter user" ${NB_USER}

# Create user owned R libs dir
# This lets users temporarily install packages
ENV R_LIBS_USER /opt/r
RUN install -d -o ${NB_USER} -g ${NB_USER} ${R_LIBS_USER}

RUN apt-get -qq update --yes && \
    apt-get -qq install --yes \
            tar \
            vim \
            micro \
            mc \
            tini \
# For jupyter-tree-download. Ref: https://github.com/berkeley-dsep-infra/datahub/issues/3979			
			zip \
            locales > /dev/null

RUN echo "en_US.UTF-8 UTF-8" > /etc/locale.gen && \
    locale-gen

# for nbconvert
# FIXME: Understand what exactly we want
# texlive-plain-generic is new name of texlive-generic-recommended
RUN apt-get update > /dev/null && \
    apt-get -qq install --yes \
            pandoc \
            texlive-xetex \
            texlive-fonts-recommended \
            # provides FandolSong-Regular.otf for issue #2714
            texlive-lang-chinese \
            texlive-plain-generic > /dev/null

RUN apt-get update > /dev/null && \
    apt-get -qq install --yes \
            # for LS88-5 and modules basemap
            libspatialindex-dev \
            # for R sf packages
            libgeos-dev \
            libproj-dev \
            proj-data \
            proj-bin \
            # For L&S22
            graphviz \
            # for phys 151
            gfortran \
            # for eps 109; fall 2019
            ffmpeg  \
            # for data100
            libpq-dev \
            # for issue #2695, fall 2021?
            libarmadillo-dev \
            postgresql-client > /dev/null

# Install packages needed by notebook-as-pdf
# Default fonts seem ok, we just install an emoji font
RUN apt-get update && \
    apt-get install --yes \
            libx11-xcb1 \
            libxtst6 \
            libxrandr2 \
            libasound2 \
            libpangocairo-1.0-0 \
            libatk1.0-0 \
            libatk-bridge2.0-0 \
            libgtk-3-0 \
            libnss3 \
            libxss1 \
            fonts-noto-color-emoji > /dev/null && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Install R packages
# Our pre-built R packages from rspm are built against system libs in focal
# rstan takes forever to compile from source, and needs libnodejs
ENV R_VERSION=4.2.1-2.2004.0
RUN apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E298A3A825C0D65DFD57CBB651716619E084DAB9
RUN echo "deb https://cloud.r-project.org/bin/linux/ubuntu focal-cran40/" > /etc/apt/sources.list.d/cran.list
RUN apt-get update -qq --yes > /dev/null && \
    apt-get install --yes -qq \
    r-base-core=${R_VERSION} \
    r-base-dev=${R_VERSION} \
    r-cran-littler=0.3.15-1.2004.0 > /dev/null

# Needed by RStudio
RUN apt-get update -qq --yes && \
    apt-get install --yes --no-install-recommends -qq \
        psmisc \
        sudo \
        libapparmor1 \
        lsb-release \
        libclang-dev  > /dev/null

# apt packages needed for R packages
RUN apt update --yes > /dev/null && \
    apt install --no-install-recommends --yes \
    # R package qpdf
    libpoppler-cpp-dev \
    # R package imager
    libx11-dev libglpk-dev libgmp3-dev libxml2-dev \
    # R package units
    libudunits2-dev \
    # R package sf
    libgdal-dev libproj-dev gdal-bin \
    # R package magick
    libmagick++-dev imagemagick > /dev/null

ENV RSTUDIO_URL https://download2.rstudio.org/server/bionic/amd64/rstudio-server-2022.07.1-554-amd64.deb
RUN curl --silent --location --fail ${RSTUDIO_URL} > /tmp/rstudio.deb && \
#    dpkg -i /tmp/rstudio.deb && \
    apt install --no-install-recommends --yes /tmp/rstudio.deb && \
    rm /tmp/rstudio.deb

#ENV SHINY_SERVER_URL https://download3.rstudio.org/ubuntu-14.04/x86_64/shiny-server-1.5.17.973-amd64.deb
ENV SHINY_SERVER_URL https://download3.rstudio.org/ubuntu-18.04/x86_64/shiny-server-1.5.19.995-amd64.deb
RUN curl --silent --location --fail ${SHINY_SERVER_URL} > /tmp/shiny-server.deb && \
#    dpkg -i /tmp/shiny-server.deb && \
    apt install --no-install-recommends --yes /tmp/shiny-server.deb && \
    rm /tmp/shiny-server.deb

# For 2022 Fall ESPM 157, but probably others will use it later
ENV QUARTO_VERSION="0.9.522"
RUN mkdir -p /opt/quarto/${QUARTO_VERSION} && \
    curl -L -o /tmp/quarto.tar.gz "https://github.com/quarto-dev/quarto-cli/releases/download/v${QUARTO_VERSION}/quarto-${QUARTO_VERSION}-linux-amd64.tar.gz" && \
    tar xzf /tmp/quarto.tar.gz -C "/opt/quarto/${QUARTO_VERSION}" --strip-components=1 && \
	rm /tmp/quarto.tar.gz && \
	ln -s /opt/quarto/${QUARTO_VERSION}/bin/quarto /usr/local/bin/quarto

# Set CRAN mirror to rspm before we install anything
COPY Rprofile.site /usr/lib/R/etc/Rprofile.site
# RStudio needs its own config
COPY rsession.conf /etc/rstudio/rsession.conf

# R_LIBS_USER is set by default in /etc/R/Renviron, which RStudio loads.
# We uncomment the default, and set what we wanna - so it picks up
# the packages we install. Without this, RStudio doesn't see the packages
# that R does.
# Stolen from https://github.com/jupyterhub/repo2docker/blob/6a07a48b2df48168685bb0f993d2a12bd86e23bf/repo2docker/buildpacks/r.py
# To try fight https://community.rstudio.com/t/timedatectl-had-status-1/72060,
# which shows up sometimes when trying to install packages that want the TZ
# timedatectl expects systemd running, which isn't true in our containers
RUN sed -i -e '/^R_LIBS_USER=/s/^/#/' /etc/R/Renviron && \
    echo "R_LIBS_USER=${R_LIBS_USER}" >> /etc/R/Renviron && \
    echo "TZ=${TZ}" >> /etc/R/Renviron

# Install R libraries as our user
USER ${NB_USER}

COPY class-libs.R /tmp/class-libs.R
RUN mkdir -p /tmp/r-packages

# Install all our base R packages
COPY install.R  /tmp/install.R
RUN /tmp/install.R && rm -rf /tmp/downloaded_packages

# pdftools
COPY r-packages/dlab-ctawg.r /tmp/r-packages/
RUN r /tmp/r-packages/dlab-ctawg.r && rm -rf /tmp/downloaded_packages

COPY r-packages/2019-fall-stat-131a.r /tmp/r-packages
RUN r /tmp/r-packages/2019-fall-stat-131a.r && rm -rf /tmp/downloaded_packages

COPY r-packages/econ-140.r /tmp/r-packages
RUN r /tmp/r-packages/econ-140.r && rm -rf /tmp/downloaded_packages

COPY r-packages/eep-1118.r /tmp/r-packages
RUN r /tmp/r-packages/eep-1118.r && rm -rf /tmp/downloaded_packages

COPY r-packages/ias-c188.r /tmp/r-packages
RUN r /tmp/r-packages/ias-c188.r && rm -rf /tmp/downloaded_packages

COPY r-packages/orphaned-ph-142.r /tmp/r-packages
RUN r /tmp/r-packages/orphaned-ph-142.r && rm -rf /tmp/downloaded_packages

COPY r-packages/2020-spring-envecon-c118.r /tmp/r-packages/
RUN r /tmp/r-packages/2020-spring-envecon-c118.r && rm -rf /tmp/downloaded_packages

COPY r-packages/orphaned-ph-290.r /tmp/r-packages/
RUN r /tmp/r-packages/orphaned-ph-290.r && rm -rf /tmp/downloaded_packages

# remove after Spring '22 semester
COPY r-packages/orphaned-2021-spring-stat-20.r /tmp/r-packages/
RUN r /tmp/r-packages/orphaned-2021-spring-stat-20.r && rm -rf /tmp/downloaded_packages

COPY r-packages/2021-spring-espm-288.r /tmp/r-packages/
RUN r /tmp/r-packages/2021-spring-espm-288.r && rm -rf /tmp/downloaded_packages

COPY r-packages/ib161.r /tmp/r-packages/
RUN r /tmp/r-packages/ib161.r && rm -rf /tmp/downloaded_packages

COPY r-packages/ps-3.r /tmp/r-packages/
RUN r /tmp/r-packages/ps-3.r && rm -rf /tmp/downloaded_packages

# remove after Spring '22 semester
COPY r-packages/orphaned-stat-20.r /tmp/r-packages/
RUN r /tmp/r-packages/orphaned-stat-20.r && rm -rf /tmp/downloaded_packages

COPY r-packages/2023-spring-stat-135.r /tmp/r-packages/
RUN r /tmp/r-packages/2023-spring-stat-135.r && rm -rf /tmp/downloaded_packages

ENV PATH ${CONDA_DIR}/bin:$PATH:/usr/lib/rstudio-server/bin

# Set this to be on container storage, rather than under $HOME ENV IPYTHONDIR ${CONDA_DIR}/etc/ipython

WORKDIR /home/${NB_USER}

# Install mambaforge as root
USER root
COPY install-mambaforge.bash /tmp/install-mambaforge.bash
RUN /tmp/install-mambaforge.bash

# Install conda environment as our user
USER ${NB_USER}

COPY environment.yml /tmp/environment.yml

RUN mamba env update -p ${CONDA_DIR} -f /tmp/environment.yml && mamba clean -afy

COPY infra-requirements.txt /tmp/infra-requirements.txt
RUN pip install --no-cache -r /tmp/infra-requirements.txt
RUN jupyter contrib nbextensions install --sys-prefix --symlink && \
    jupyter nbextensions_configurator enable --sys-prefix

COPY requirements.txt /tmp/requirements.txt
RUN pip install --no-cache -r /tmp/requirements.txt

# Set up nbpdf dependencies
ENV PYPPETEER_HOME ${CONDA_DIR}
RUN pyppeteer-install

# Install IR kernelspec
RUN r -e "IRkernel::installspec(user = FALSE, prefix='${CONDA_DIR}')"

COPY d8extension.bash /usr/local/sbin/d8extension.bash
RUN /usr/local/sbin/d8extension.bash

ENV NLTK_DATA ${CONDA_DIR}/nltk_data
COPY connectors/text.bash /usr/local/sbin/connector-text.bash
RUN /usr/local/sbin/connector-text.bash

COPY connectors/2021-fall-phys-188-288.bash /usr/local/sbin/
RUN /usr/local/sbin/2021-fall-phys-188-288.bash

# Used by MCB32, but incompatible with ipywidgets 8.x
RUN pip install --no-cache qgrid==1.3.1
RUN jupyter nbextension enable --py --sys-prefix qgrid

# clear out /tmp
USER root
RUN rm -rf /tmp/*

USER ${NB_USER}

EXPOSE 8888

ENTRYPOINT ["tini", "--"]

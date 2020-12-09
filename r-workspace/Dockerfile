## Commit from circa mid 11/2020
## Base image based on ubuntu:focal with py3.8
FROM jupyter/minimal-notebook:42f4c82a07ff

## Declares build arguments
ARG NB_USER=jovyan
ARG NB_UID=1000
ARG NB_GID=100

ENV NB_USER=jovyan
ENV NB_UID="${NB_UID}"
ENV NB_GID="${NB_GID}"
ENV NB_USER="${NB_USER}"
ENV HOME="/home/${NB_USER}"

ENV JUPYTER_CONFIG_DIR="${NB_USER}/.jupyter"
ENV JUPYTER_DATA_DIR="${NB_USER}/.local/share/jupyter"
ENV JUPYTER_PATH="${NB_USER}/.local/share/jupyter"
ENV JUPYTER_RUNTIME_DIR="${NB_USER}/.local/share/jupyter/runtime"

# Via https://rstudio.com/products/rstudio/download-server/debian-ubuntu/
ENV RSTUDIO_URL="https://download2.rstudio.org/server/bionic/amd64/rstudio-server-1.2.5001-amd64.deb"
# This is MD5, because that is what RStudio download page provides!
ENV RSTUDIO_CHECKSUM="d33881b9ab786c09556c410e7dc477de"

# Via https://www.rstudio.com/products/shiny/download-server/
ENV SHINY_URL="https://download3.rstudio.org/ubuntu-14.04/x86_64/shiny-server-1.5.12.933-amd64.deb"
ENV SHINY_CHECKSUM="9aeef6613e7f58f21c97a4600921340e"

# Version of MRAN to pull devtools from.
ENV DEVTOOLS_VERSION="2018-02-01"

ENV SHINY_DEB="/tmp/shiny.deb"

USER root

# R pre-requisites
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    ca-certificates \
    fonts-dejavu \
    fonts-liberation \
    locales \
    gfortran \
    gcc \
    r-cran-rodbc \
    run-one \
    sudo \
    unixodbc \
    unixodbc-dev \
    wget \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/*

# Fix for devtools https://github.com/conda-forge/r-devtools-feedstock/issues/4
RUN ln -s /bin/tar /bin/gtar

USER "${NB_UID}"

# R packages
RUN conda install --quiet --yes \
    'r-base=3.6.*' \
    'r-irkernel=1.1*' \
 && conda clean --all -f -y \
 && fix-permissions "${CONDA_DIR}"

# Install e1071 R package (dependency of the caret R package)
RUN conda install --quiet --yes r-e1071

# Install pip packages with pip
COPY requirements.txt /tmp/
RUN pip install -r /tmp/requirements.txt

# Install R packages
COPY install.R /tmp/install.R
RUN if [ -f /tmp/install.R ]; then R --quiet -f /tmp/install.R; fi

USER root

# Install RStudio
RUN curl --silent --location --fail "${RSTUDIO_URL}" > /tmp/rstudio.deb \
 && echo "${RSTUDIO_CHECKSUM} /tmp/rstudio.deb" | md5sum -c - \
 && apt-get update \
 && apt install -y /tmp/rstudio.deb \
 && rm /tmp/rstudio.deb \
 && apt-get -qq purge \
 && apt-get -qq clean \
 && rm -rf /var/lib/apt/lists/*

# Install Shiny
RUN curl --silent --location --fail "${SHINY_URL}" > "${SHINY_DEB}" \
 && echo "${SHINY_CHECKSUM}" "${SHINY_DEB}" | md5sum -c - \
 && dpkg -i "${SHINY_DEB}" \
 && rm "${SHINY_DEB}"

# Ensure permissions are set for Shiny
RUN install -o "${NB_UID}" -g "${NB_GID}" -d /var/log/shiny-server \
 && install -o "${NB_UID}" -g "${NB_GID}" -d /var/lib/shiny-server \
 && install -o "${NB_UID}" -g "${NB_GID}" /dev/null /var/log/shiny-server.log \
 && install -o "${NB_UID}" -g "${NB_GID}" /dev/null /var/run/shiny-server.pid

# Install and enable RStudio/Shiny extensions
RUN pip install --no-cache-dir jupyter-server-proxy==1.5.0 \
 && pip install --no-cache-dir https://github.com/jupyterhub/jupyter-rsession-proxy/archive/d5efed5455870556fc414f30871d0feca675a4b4.zip \
 && pip install --no-cache-dir https://github.com/ryanlovett/jupyter-shiny-proxy/archive/47557dc47e2aeeab490eb5f3eeae414cdde4a6a9.zip \
 && jupyter serverextension enable jupyter_server_proxy --sys-prefix \
 && jupyter nbextension install --py jupyter_server_proxy --sys-prefix \
 && jupyter nbextension enable --py jupyter_server_proxy --sys-prefix

# Install nbgrader and then disable all extensions by default
RUN jupyter nbextension install --symlink --sys-prefix --py nbgrader --overwrite \
 && jupyter nbextension disable --sys-prefix --py nbgrader \
 && jupyter serverextension disable --sys-prefix --py nbgrader

# Everyone gets the nbgrader validate extension
RUN jupyter nbextension enable --sys-prefix validate_assignment/main --section=notebook \
 && jupyter serverextension enable --sys-prefix nbgrader.server_extensions.validate_assignment

# Everyone gets the assignment extension
RUN jupyter serverextension enable --sys-prefix nbgrader.server_extensions.assignment_list \
 && jupyter nbextension enable --sys-prefix assignment_list/main --section=tree

## Copy nbgrader and notebook configs
COPY global_nbgrader_config.py /etc/jupyter/nbgrader_config.py
RUN cp /etc/jupyter/jupyter_notebook_config.py /etc/jupyter/jupyter_notebook_config_base.py 
COPY jupyter_notebook_config.py /etc/jupyter/

# Fix permissions as root
RUN fix-permissions "${HOME}" \
 && fix-permissions /usr/local/lib/R/ \
 && fix-permissions "${CONDA_DIR}" \
 && fix-permissions /etc/jupyter \
 && fix-permissions /var/log/shiny-server

RUN chown -R "${NB_UID}" "${HOME}"

CMD ["start-notebook.sh"]

USER "${NB_USER}"

WORKDIR "${HOME}"

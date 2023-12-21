# Copyright (c) Jupyter Development Team.
# Distributed under the terms of the Modified BSD License.
# ARG REGISTRY=quay.io
# ARG OWNER=jupyter
# ARG BASE_CONTAINER=$REGISTRY/$OWNER/scipy-notebook
# FROM $BASE_CONTAINER

# LABEL maintainer="Jupyter Project <jupyter@googlegroups.com>"

ARG REGISTRY=docker.io
ARG OWNER=yifengdev
ARG BASE_IMAGE=scipy-jupyter
ARG BASE_CONTAINER=$REGISTRY/$OWNER/$BASE_IMAGE
FROM $BASE_CONTAINER

LABEL maintainer="Liu <932048437@qq.com>"

# Fix: https://github.com/hadolint/hadolint/wiki/DL4006
# Fix: https://github.com/koalaman/shellcheck/wiki/SC3014
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

USER ${NB_UID}

# Add setup scripts that may be used by downstream images or inherited images
COPY setup-scripts/ /opt/setup-scripts/

# R packages including IRKernel which gets installed globally.
# r-e1071: dependency of the caret R package
RUN conda create -n r-env && \
    # conda run -n jupyter conda install -y \
    source /opt/conda/bin/activate r-env && \
    conda install --yes \
    r-base \
    r-caret \
    r-crayon \
    r-devtools \
    r-e1071 \
    r-forecast \
    r-hexbin \
    r-htmltools \
    r-htmlwidgets \
    r-irkernel \
    r-nycflights13 \
    r-randomforest \
    r-rcurl \
    r-rmarkdown \
    r-rodbc \
    r-rsqlite \
    r-shiny \
    r-tidymodels \
    r-tidyverse \
    rpy2 \
    unixodbc && \
    conda clean --all -f -y && \
    ln -s /opt/conda/envs/jupyter/bin/jupyter /opt/conda/envs/r-env/bin/jupyter && \
    fix-permissions "${CONDA_DIR}" && \
    fix-permissions "/home/${NB_USER}"

# Add an R mimetype option to specify how the plot returns from R to the browser
COPY --chown=${NB_UID}:${NB_GID} Rprofile.site /opt/conda/envs/r-env/lib/R/etc/

USER root

# Julia dependencies
# install Julia packages in /opt/julia instead of ${HOME}
ENV JULIA_DEPOT_PATH=/opt/conda/envs/julia-env/share/julia \
    JULIA_PKGDIR=/opt/conda/envs/julia-env/share/julia

# R pre-requisites
RUN apt-get update --yes && \
    apt-get install --yes --no-install-recommends \
    fonts-dejavu \
    gfortran \
    gcc && \
    apt-get clean && rm -rf /var/lib/apt/lists/* && \
    conda create -n julia-env && \
    source /opt/conda/bin/activate julia-env && \
    conda clean --all -f -y && \
    /opt/setup-scripts/setup_julia.py && \
    fix-permissions "${CONDA_DIR}" && \
    fix-permissions "/home/${NB_USER}"

RUN source /opt/conda/bin/activate r-env && \
    R --slave -e "IRkernel::installspec(name = 'r-env', displayname = 'R (r-env)', user = FALSE)" && \
    mv /usr/local/share/jupyter/kernels/r-env /opt/conda/envs/jupyter/share/jupyter/kernels && \
    rm -rf /usr/local/share/jupyter && \
    rm -rf /root/.local && \
    chown -R ${NB_UID}:${NB_GID} "${CONDA_DIR}/envs/jupyter/share/jupyter/kernels/" && \
    fix-permissions "${CONDA_DIR}" && \
    fix-permissions "/home/${NB_USER}" "${CONDA_DIR}/envs/jupyter/share/jupyter/kernels/"
# Setup Julia

USER ${NB_UID}

# Setup IJulia kernel & other packages
RUN /opt/setup-scripts/setup-julia-packages.bash




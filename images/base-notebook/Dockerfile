# Copyright (c) Jupyter Development Team.
# Distributed under the terms of the Modified BSD License.
ARG REGISTRY=docker.io
ARG OWNER=yifengdev
ARG BASE_IMAGE=base-conda
ARG BASE_CONTAINER=$REGISTRY/$OWNER/$BASE_IMAGE
FROM $BASE_CONTAINER

LABEL maintainer="Liu <932048437@qq.com>"

# Fix: https://github.com/hadolint/hadolint/wiki/DL4006
# Fix: https://github.com/koalaman/shellcheck/wiki/SC3014
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

USER root

# Install all OS dependencies for the Server that starts but lacks all
# features (e.g., download as all possible file formats)
RUN apt-get update --yes && \
    apt-get install --yes --no-install-recommends \
    fonts-liberation \
    # - pandoc is used to convert notebooks to html files
    #   it's not present in the aarch64 Ubuntu image, so we install it here
    pandoc \
    # - run-one - a wrapper script that runs no more
    #   than one unique  instance  of  some  command with a unique set of arguments,
    #   we use `run-one-constantly` to support the `RESTARTABLE` option
    run-one && \
    apt-get clean && rm -rf /var/lib/apt/lists/* && \
    sed -i '/^# Execute the command!/i \
    if [[ "${cmd[0]}" == "jupyter" ]]; then\n    source /opt/conda/bin/activate jupyter\nfi\n' /usr/local/bin/start.sh 

USER ${NB_UID}

# Install JupyterLab, Jupyter Notebook, JupyterHub and NBClassic
# Generate a Jupyter Server config
# Cleanup temporary files
# Correct permissions
# Do all this in a single RUN command to avoid duplicating all of the
# files across image layers when the permissions change
WORKDIR /tmp
RUN conda create -n jupyter && \
    # conda run -n jupyter conda install -y \
    source /opt/conda/bin/activate jupyter && \
    conda install -y \
    jupyter_core \
    jupyterlab \
    notebook \
    jupyterhub \
    nbclassic && \
    # conda run -n jupyter jupyter server --generate-config && \
    # conda run -n jupyter conda clean --all -f -y && \
    # conda run -n jupyter npm cache clean --force && \
    # conda run -n jupyter jupyter lab clean && \
    jupyter server --generate-config && \
    conda clean --all -f -y && \
    npm cache clean --force && \
    jupyter lab clean && \
    # conda create --name keras-torch --clone keras-tf && \
    rm -rf "/home/${NB_USER}/.cache/yarn" && \
    fix-permissions "${CONDA_DIR}" && \
    fix-permissions "/home/${NB_USER}"

ENV JUPYTER_PORT=8888
EXPOSE $JUPYTER_PORT

# Configure container startup
CMD ["start-notebook.py"]

# Copy local files as late as possible to avoid cache busting
COPY start-notebook.py start-notebook.sh start-singleuser.py start-singleuser.sh /usr/local/bin/
COPY jupyter_server_config.py docker_healthcheck.py /etc/jupyter/

# Fix permissions on /etc/jupyter as root
USER root
RUN fix-permissions /etc/jupyter/

# HEALTHCHECK documentation: https://docs.docker.com/engine/reference/builder/#healthcheck
# This healtcheck works well for `lab`, `notebook`, `nbclassic`, `server`, and `retro` jupyter commands
# https://github.com/jupyter/docker-stacks/issues/915#issuecomment-1068528799
HEALTHCHECK --interval=5s --timeout=3s --start-period=5s --retries=3 \
    CMD /etc/jupyter/docker_healthcheck.py || exit 1

# Switch back to jovyan to avoid accidental container runs as root
USER ${NB_UID}

WORKDIR "${HOME}"

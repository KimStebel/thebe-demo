FROM jupyter/notebook

MAINTAINER Kim Stebel <kim.stebel@gmail.com>

USER root

WORKDIR /

RUN apt-get update && apt-get install -yq --no-install-recommends wget

# Configure environment
ENV CONDA_DIR /opt/conda
ENV PATH $CONDA_DIR/bin:$PATH
ENV SHELL /bin/bash
ENV NB_USER root
ENV NB_UID 0

# Install conda
RUN mkdir -p $CONDA_DIR && \
    echo export PATH=$CONDA_DIR/bin:'$PATH' > /etc/profile.d/conda.sh && \
    wget --quiet https://repo.continuum.io/miniconda/Miniconda3-3.9.1-Linux-x86_64.sh && \
    echo "6c6b44acdd0bc4229377ee10d52c8ac6160c336d9cdd669db7371aa9344e1ac3 *Miniconda3-3.9.1-Linux-x86_64.sh" | sha256sum -c - && \
    /bin/bash /Miniconda3-3.9.1-Linux-x86_64.sh -f -b -p $CONDA_DIR && \
    rm Miniconda3-3.9.1-Linux-x86_64.sh && \
    $CONDA_DIR/bin/conda install --yes conda==3.14.1

RUN mkdir -p /root/work && \
    mkdir -p /root/.jupyter && \
    mkdir -p /root/.local

ENV PATH /opt/happy/1.19.4/bin:/opt/alex/3.1.3/bin:$PATH

# Python packages
RUN conda install --yes numpy pandas scikit-learn scikit-image matplotlib scipy seaborn sympy cython patsy statsmodels cloudpickle dill numba bokeh beautiful-soup && conda clean -yt

# Now for a python2 environment
RUN conda create -p $CONDA_DIR/envs/python2 python=2.7 ipykernel numpy pandas scikit-learn scikit-image matplotlib scipy seaborn sympy cython patsy statsmodels cloudpickle dill numba bokeh && conda clean -yt
RUN $CONDA_DIR/envs/python2/bin/python \
    $CONDA_DIR/envs/python2/bin/ipython \
    kernelspec install-self --user

# install python cloudant library
RUN pip3 install --no-cache-dir cloudant
RUN $CONDA_DIR/bin/pip install --no-cache-dir cloudant

# Expose our custom setup to the installed ipython (for mounting by nginx)
COPY resources/custom.js /opt/conda/lib/python3.4/site-packages/notebook/static/custom/

# Add the templates
COPY resources/templates/ /srv/templates/
RUN chmod a+rX /srv/templates

# Append tmpnb specific options to the base config
COPY resources/jupyter_notebook_config.partial.py /tmp/
RUN cat /tmp/jupyter_notebook_config.partial.py >> /root/.jupyter/jupyter_notebook_config.py && \
    rm /tmp/jupyter_notebook_config.partial.py

# install bash kernel, curl, jq, netcat
RUN apt-get update && apt-get install -y --force-yes --no-install-recommends \
    curl \
    netcat \
    jq && \
    pip3 install ipywidgets metakernel_bash

# javascript, node.js, ibm-graph-client for node.js
ENV NODE_PATH /usr/local/lib/node_modules/
RUN apt-get update && apt-get install -y --force-yes --no-install-recommends \
    nodejs-legacy \
    npm && \
    npm install -g cloudant && \
    npm install -g ijavascript && \
    npm install -g 'ibm-graph-client@0.3.4' && \
    ijs --ijs-install=global

# install cloudant library for python 2
RUN $CONDA_DIR/envs/python2/bin/pip install --no-cache-dir cloudant

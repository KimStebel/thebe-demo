FROM jupyter/notebook

MAINTAINER Kim Stebel <kim.stebel@gmail.com>

USER root

WORKDIR /

RUN apt-get update && apt-get install -yq --no-install-recommends \
    wget 

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


# Ruby dependencies
#RUN apt-get install -y --no-install-recommends ruby ruby-dev libtool autoconf automake gnuplot-nox libsqlite3-dev libatlas-base-dev libgsl0-dev libmagick++-dev imagemagick && \
#    ln -s /usr/bin/libtoolize /usr/bin/libtool && \
#    apt-get clean
#RUN gem install --no-rdoc --no-ri sciruby-full

ENV PATH /root/.cabal/bin:/opt/cabal/1.22/bin:/opt/ghc/7.8.4/bin:/opt/happy/1.19.4/bin:/opt/alex/3.1.3/bin:$PATH
ENV PYTHONPATH $SPARK_HOME/python:$SPARK_HOME/python/lib/py4j-0.8.2.1-src.zip

# Python packages
RUN conda install --yes numpy pandas scikit-learn scikit-image matplotlib scipy seaborn sympy cython patsy statsmodels cloudpickle dill numba bokeh beautiful-soup && conda clean -yt

# Now for a python2 environment
RUN conda create -p $CONDA_DIR/envs/python2 python=2.7 ipykernel numpy pandas scikit-learn scikit-image matplotlib scipy seaborn sympy cython patsy statsmodels cloudpickle dill numba bokeh && conda clean -yt
RUN $CONDA_DIR/envs/python2/bin/python \
    $CONDA_DIR/envs/python2/bin/ipython \
    kernelspec install-self --user

# R packages
#RUN conda config --add channels r
#RUN conda install --yes r-irkernel r-plyr r-devtools r-rcurl r-dplyr r-ggplot2 r-caret rpy2 r-tidyr r-shiny r-rmarkdown r-forecast r-stringr r-rsqlite r-reshape2 r-nycflights13 r-randomforest && conda clean -yt

# Extra Kernels
#RUN pip install --user --no-cache-dir bash_kernel && \
#    python -m bash_kernel.install

RUN pip3 install --no-cache-dir cloudant
RUN $CONDA_DIR/bin/pip install --pre --no-cache-dir cloudant

# Convert notebooks to the current format and trust them
RUN find /root/work -name '*.ipynb' -exec ipython nbconvert --to notebook {} --output {} \; && \
    env "PATH=$PATH" find /root/work -name '*.ipynb' -exec ipython trust {} \;

# Finally, add the site specific tmpnb.org / try.jupyter.org configuration.
# These should probably be split off into a separate docker image so that others
# can reuse the very expensive build of all the above with their own site 
# customization.

# Expose our custom setup to the installed ipython (for mounting by nginx)
COPY resources/custom.js /opt/conda/lib/python3.4/site-packages/notebook/static/custom/

# Add the templates
COPY resources/templates/ /srv/templates/
RUN chmod a+rX /srv/templates

# Append tmpnb specific options to the base config
COPY resources/jupyter_notebook_config.partial.py /tmp/
RUN cat /tmp/jupyter_notebook_config.partial.py >> /root/.jupyter/jupyter_notebook_config.py && \
    rm /tmp/jupyter_notebook_config.partial.py

#javascript / node
RUN apt-get update && apt-get install -y --force-yes --no-install-recommends \
    nodejs-legacy \
    npm && \
    npm install -g cloudant && \
    npm install -g ijavascript && \
    npm install -g git+https://github.com/ibm-cds-labs/gds-wrapper.git && \
    ijs --ijs-install=global

ENV NODE_PATH /usr/local/lib/node_modules/

#bash, curl, jq, netcat
RUN apt-get update && apt-get install -y --force-yes --no-install-recommends \
    curl \
    netcat \
    jq && \
    pip3 install ipywidgets metakernel_bash

#add datasets
COPY datasets/ /datasets/

#add pyspark kernel spec
RUN mkdir -p /usr/local/share/jupyter/kernels/pyspark
COPY resources/pyspark/kernel.json /usr/local/share/jupyter/kernels/pyspark/

# Spark dependencies
ENV APACHE_SPARK_VERSION 1.5.1
RUN wget -qO - http://d3kbcqa49mib13.cloudfront.net/spark-${APACHE_SPARK_VERSION}-bin-hadoop2.3.tgz | tar -xz -C /usr/local/ && \
    cd /usr/local && \
    ln -s spark-${APACHE_SPARK_VERSION}-bin-hadoop2.3 spark

# add Scala Spark kernel
RUN mkdir -p /opt/sparkkernel/bin /opt/sparkkernel/lib
COPY resources/spark-kernel/bin/spark-kernel /opt/sparkkernel/bin/
COPY resources/spark-kernel/lib/kernel-assembly-* /opt/sparkkernel/lib/
COPY resources/spark-kernel/VERSION /opt/sparkkernel/
RUN chmod +x /opt/sparkkernel

# Add Scala Spark kernel spec
RUN mkdir -p /usr/local/share/jupyter/kernels/scala
COPY resources/spark-kernel.json /usr/local/share/jupyter/kernels/scala/kernel.json

ENV SPARK_HOME /usr/local/spark

# install java 7
RUN apt-get update && \
    apt-get install -y --no-install-recommends openjdk-7-jre-headless && \
    apt-get clean

# install leiningen
RUN wget https://raw.githubusercontent.com/technomancy/leiningen/stable/bin/lein && chmod +x lein && mv lein /usr/bin/ && lein

# install clojupyter
RUN git clone https://github.com/kimstebel/clojupyter && \
    cd clojupyter && \
    make && \
    make install && \
    cp -r /root/.ipython/kernels/clojure /usr/local/share/jupyter/kernels/


ARG CENTOS_VER
FROM centos:${CENTOS_VER}

LABEL maintainer="Maksim Rakitin <mrakitin@bnl.gov>"

ARG CENTOS_VER
ENV CENTOS_VER=${CENTOS_VER}

# Set an encoding to make things work smoothly.
ENV LANG en_US.UTF-8

# Add qemu in here so that we can use this image on regular linux hosts with qemu user installed
# ADD qemu-aarch64-static /usr/bin/qemu-aarch64-static
# ADD qemu-arm-static /usr/bin/qemu-arm-static
# ADD qemu-ppc64le-static /usr/bin/qemu-ppc64le-static
# ADD qemu-s390x-static /usr/bin/qemu-s390x-static

# bust the docker cache so that we always rerun the installs below
ADD https://loripsum.net/api /opt/docker/etc/gibberish

# Resolves a nasty NOKEY warning that appears when using yum.
RUN rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-${CENTOS_VER}

# Install basic requirements.
COPY scripts/yum_clean_all /opt/docker/bin/
RUN yum update -y && \
    yum install -y \
        bzip2 \
        sudo \
        tar \
        which && \
    /opt/docker/bin/yum_clean_all

# Run common commands
COPY scripts/run_commands /opt/docker/bin/run_commands
RUN /opt/docker/bin/run_commands

# Download and cache new compiler packages.
# Should speedup installation of them on CIs.
RUN source /opt/conda/etc/profile.d/conda.sh && \
    conda activate && \
    conda create -n test --yes --quiet --download-only \
        conda-forge::binutils_impl_linux-64 \
        conda-forge::binutils_linux-64 \
        conda-forge::gcc_impl_linux-64 \
        conda-forge::gcc_linux-64 \
        conda-forge::gfortran_impl_linux-64 \
        conda-forge::gfortran_linux-64 \
        conda-forge::gxx_impl_linux-64 \
        conda-forge::gxx_linux-64 \
        conda-forge::libgcc-ng \
        conda-forge::libgfortran-ng \
        conda-forge::libstdcxx-ng && \
    conda remove --yes --quiet -n test --all && \
    conda clean -tiy && \
    chgrp -R nsls2 /opt/conda && \
    chmod -R g=u /opt/conda

# Add a file for users to source to activate the `conda`
# environment `base`. Also add a file that wraps that for
# use with the `ENTRYPOINT`.
COPY linux-anvil-cos7-x86_64/entrypoint_source /opt/docker/bin/entrypoint_source
COPY scripts/entrypoint /opt/docker/bin/entrypoint

# Ensure that all containers start with tini and the user selected process.
# Activate the `conda` environment `base` and the devtoolset compiler.
# Provide a default command (`bash`), which will start if the user doesn't specify one.
ENTRYPOINT [ "/opt/conda/bin/tini", "--", "/opt/docker/bin/entrypoint" ]
CMD [ "/bin/bash" ]

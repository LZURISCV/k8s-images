FROM riscv64/debian:testing-20250317-slim AS build

# 禁用 debconf 的交互式提示
ENV DEBIAN_FRONTEND=noninteractive

# 安装构建工具和 Go 语言环境
RUN apt-get update && apt-get install -y \
    make \
    gcc \
    g++ \
    git \
    curl \
    golang \
    && rm -rf /var/lib/apt/lists/*

# 设置 mpi-operator 版本
# 默认为 v2
ARG VERSION=v2
ARG RELEASE_VERSION

ADD . /go/src/github.com/kubeflow/mpi-operator
WORKDIR /go/src/github.com/kubeflow/mpi-operator
RUN make RELEASE_VERSION=${RELEASE_VERSION} mpi-operator.$VERSION
RUN ln -s mpi-operator.${VERSION} _output/cmd/bin/mpi-operator

FROM riscv64/debian:testing-20250317-slim

# 安装 Bash、OpenMPI 环境以及 openssh-server
RUN apt-get update && apt-get install -y \
    bash \
    openmpi-bin \
    libopenmpi-dev \
    openssh-server \
    && rm -rf /var/lib/apt/lists/*

# 可选：配置 openssh-server（例如创建 sshd 运行目录）
RUN mkdir -p /var/run/sshd

ENV CONTROLLER_VERSION=$VERSION
COPY --from=build /go/src/github.com/kubeflow/mpi-operator/_output/cmd/bin/* /opt/
COPY third_party/library/license.txt /opt/license.txt

# 默认 ENTRYPOINT 指向 mpi-operator，可根据需要覆盖
ENTRYPOINT ["/opt/mpi-operator"]
# 默认 CMD 为 Bash 交互环境，如需使用 mpi-operator，可通过 docker run 参数覆盖
CMD ["/bin/bash"]



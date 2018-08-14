# Copyright 2017 The Kubernetes Authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

FROM gcr.io/k8s-testimages/dind-test-base-amd64:0.2

# Get our short startup scripts first, because they're least likely to change.
COPY init-wrapper.sh /init-wrapper.sh
COPY start.sh /start.sh

# Addon config changes infrequently, so get them next.
COPY cluster /

# Get the debs
COPY kubectl.deb /
COPY kubelet.deb /
COPY kubernetes-cni.deb /
COPY kubeadm.deb /

# Grab a file with the version info (one for kubeadm, one for docker).
COPY source_version /
COPY docker_version /

# Install the binaries before getting the component images, because if they've
# changed, we're already redoing everything.

# kubernetes-cni.deb needs this dir
RUN mkdir -p /opt/cni/bin

# kubeadm depends on kubelet depends on kubernetes-cni
RUN dpkg -i kubernetes-cni.deb
RUN dpkg -i kubectl.deb
RUN dpkg -i kubelet.deb
RUN apt update
RUN apt install wget
ENV CRI_VERSION="v1.11.1"
RUN wget https://github.com/kubernetes-incubator/cri-tools/releases/download/$CRI_VERSION/crictl-$CRI_VERSION-linux-amd64.tar.gz
RUN tar zxvf crictl-*-linux-amd64.tar.gz -C /usr/local/bin
RUN rm -f crictl-*-linux-amd64.tar.gz
RUN dpkg -i --force depends kubeadm.deb

# Get the component docker images last, because they're most likely to change.
COPY kube-proxy.tar /
COPY kube-controller-manager.tar /
COPY kube-scheduler.tar /
COPY kube-apiserver.tar /

ENTRYPOINT ["/init-wrapper.sh"]

FROM ubuntu
RUN /usr/bin/apt-get update -yq && /usr/bin/apt-get install -yq iputils-ping  \
iproute2 && rm -rf /var/lib/apt/lists/*   
CMD ping -c 3 172.16.1.1

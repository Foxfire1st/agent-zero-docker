FROM nvidia/cuda:12.9.0-cudnn-devel-ubuntu24.04

SHELL ["/bin/bash", "-c"]

RUN apt update 
RUN apt upgrade -y 
RUN apt --fix-broken install -y python3 python3-pip wget git openssh-server

RUN mkdir -p ~/miniconda3 && wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda3/miniconda.sh && bash ~/miniconda3/miniconda.sh -b -u -p ~/miniconda3 && rm ~/miniconda3/miniconda.sh

ENV PATH="/root/miniconda3/bin:$PATH"
ENV PATH=/usr/local/cuda/bin:$PATH
ENV LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH

RUN cd /root && git clone https://github.com/frdel/agent-zero

WORKDIR /root/agent-zero

RUN conda create -n a0 python=3.12 -y 
RUN eval "$(conda shell.bash hook)" && \
    conda activate a0 && \
    pip install -r requirements.txt

# Generate SSH host keys
RUN ssh-keygen -A

# Configure SSH (following official agent-zero setup)
RUN mkdir -p /var/run/sshd
RUN echo 'root:temp123' | chpasswd
RUN sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
RUN sed -i 's/#PasswordAuthentication yes/PasswordAuthentication yes/' /etc/ssh/sshd_config
RUN sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config

# Configure SSH client to not prompt for host key verification for localhost
RUN echo "Host localhost" >> /root/.ssh/config
RUN echo "    StrictHostKeyChecking no" >> /root/.ssh/config
RUN echo "    UserKnownHostsFile /dev/null" >> /root/.ssh/config
RUN mkdir -p /root/.ssh && chmod 700 /root/.ssh && chmod 600 /root/.ssh/config

CMD service ssh start && sleep 3 && eval "$(conda shell.bash hook)" && conda activate a0 && python prepare.py --dockerized=true && python run_ui.py --host 0.0.0.0 --port 5000
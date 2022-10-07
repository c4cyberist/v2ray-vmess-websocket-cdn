FROM ubuntu:22.04

# Disable Prompt During Packages Installation
ARG DEBIAN_FRONTEND=noninteractive

RUN apt update && \
apt install curl gpg -y && \
echo 'deb [signed-by=/usr/share/keyrings/tor-archive-keyring.gpg] https://deb.torproject.org/torproject.org jammy main' > /etc/apt/sources.list.d/tor.list && \
curl -s https://deb.torproject.org/torproject.org/A3C4F0F979CAA22CDBA8F512EE8CBC9E886DDD89.asc | gpg --dearmor | tee /usr/share/keyrings/tor-archive-keyring.gpg >/dev/null && \
apt update && \
apt upgrade -y && \
apt install apt-transport-https tor deb.torproject.org-keyring -y && \
rm -rf /var/lib/apt/lists/* && \
apt clean

CMD /usr/bin/tor -f /etc/tor/torrc

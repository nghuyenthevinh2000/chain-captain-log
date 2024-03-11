* sudo newgroup docker
* sudo chmod 666 /var/run/docker.sock
* sudo usermod -aG docker ${USER}

# Log out and log back in so that group membership is re-evaluated or run this command
newgrp docker

docker run hello-world

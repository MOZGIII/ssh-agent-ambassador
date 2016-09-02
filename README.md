# ssh-agent-ambassador

A docker container to proxy access to `ssh-agent`.

## Usage

### Simple usage guide.

Build `ssh-agent-ambassador` container first.

```
docker build -t ssh-agent-ambassador .
```

Then run it.

```
docker run --rm -it -v $SSH_AUTH_SOCK:/ssh_auth_sock -p 1234:1234 ssh-agent-ambassador
```

Now, use your ssh agent proxy.

```
# Install socat
apt-get install -y socat

# Create unix socket to allow ssh tooling connections
socat tcp-connect:<ssh-agent-ambassador-ip>:1234 unix-listen:./local_ssh_auth_sock

# Use remote ssh agent
SSH_AUTH_SOCK=./local_ssh_auth_sock git clone <your private repo>
```

### Example for use at `docker build` phase.

Step 1. Modify you `Dcokerfile` like this (this is for `debian` OS family):

```dockerfile
# Install socat and ssh tooling
RUN apt-get update \
 && apt-get install -yqq \
      ssh \
      socat \
 && rm -rf /var/lib/apt/lists/* \
 && mkdir -p ~/.ssh/ \
 && echo "Host *\n\tStrictHostKeyChecking no\n\n" >> ~/.ssh/config

# Enable private repositories access
# via ssh during build time.
ARG SSH_AUTH_SOCK_PROXY

# Set SSH_AUTH_SOCK upfront to open unix socket to it.
ENV SSH_AUTH_SOCK=/tmp/ssh_auth_sock_proxy

# Use socat to proxify access to ssh-agent.
RUN socat -t 1000000 "unix-listen:$SSH_AUTH_SOCK,fork" "tcp-connect:$SSH_AUTH_SOCK_PROXY" & sleep 1 \
 && ssh-add -l

# Connect ssh-agent proxy at before you use it at every RUN command.
RUN socat -t 1000000 "unix-listen:$SSH_AUTH_SOCK,fork" "tcp-connect:$SSH_AUTH_SOCK_PROXY" & sleep 1 \
 && ssh-add -l
```

Step 2. Run `ssh-agent-ambassador` container and keep it running.

Step 3. Execute `docker build`.

Pass `SSH_AUTH_SOCK_PROXY` build arg.

```
docker build --build-arg='SSH_AUTH_SOCK_PROXY=<ambassador-container-ip>:1234' .
```

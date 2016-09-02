# ssh-agent-ambassador

A docker container to proxy access to `ssh-agent`.

## Usage

Build `ssh-agent-ambassador` container first.

```
docker build -t ssh-agent-ambassador .
```

Then run it:

```
docker run --rm -it -v $SSH_AUTH_SOCK:/ssh_auth_sock -p 1234:1234 ssh-agent-ambassador
```

Now, use your ssh agent in other container (even during build phase):

```
# Install socat
apt-get install -y socat

# Create unix socket to allow ssh tooling connections
socat tcp-connect:<ssh-agent-ambassador-ip>:1234 unix-listen:./local_ssh_auth_sock

# Use remote ssh agent
SSH_AUTH_SOCK=./local_ssh_auth_sock git clone <your private repo>
```

<h1 align="center">
    SSH Proxy
</h1>

<p align="center">
    <strong>Dockerized SSH bastion to proxy SSH connections to arbitrary containers.</strong>
</p>

<p align="center">
    <a href="https://hub.docker.com/r/mltooling/ssh-proxy" title="Docker Image Version"><img src="https://images.microbadger.com/badges/version/mltooling/ssh-proxy.svg"></a>
    <a href="https://hub.docker.com/r/mltooling/ssh-proxy" title="Docker Pulls"><img src="https://img.shields.io/docker/pulls/mltooling/ssh-proxy.svg"></a>
    <a href="https://hub.docker.com/r/mltooling/ssh-proxy" title="Docker Image Metadata"><img src="https://images.microbadger.com/badges/image/mltooling/ssh-proxy.svg"></a>
    <a href="https://github.com/ml-tooling/ssh-proxy/blob/develop/LICENSE" title="SSH Proxy License"><img src="https://img.shields.io/badge/License-Apache%202.0-green.svg"></a>
    <a href="https://gitter.im/ml-tooling/community" title="Chat on Gitter"><img src="https://badges.gitter.im/ml-tooling/community.svg"></a>
    <a href="https://twitter.com/mltooling" title="ML Tooling on Twitter"><img src="https://img.shields.io/twitter/follow/mltooling.svg?style=social"></a>
</p>

<p align="center">
  <a href="#getting-started">Getting Started</a> •
  <a href="#highlights">Highlights</a> •
  <a href="#where-to-ask-questions">Support</a> •
  <a href="https://github.com/ml-tooling/ssh-proxy/issues/new?labels=bug&template=01_bug-report.md">Report a Bug</a> •
  <a href="#contribution">Contribution</a>
</p>

This SSH proxy can be deployed as a standalone docker container that allows to proxy any user SSH connection to arbitrary unexposed containers. This enables users to securely access any container via SSH within a cluster only via a single exposed port and provides full SSH compatibility (e.g. port tunneling, scp, sftp, rsync, sshfs, X11 forwarding). This proxy has a few security features built-in to make sure that users can only access target containers that they are allowed to.

## Highlights

- 🛡 SSH access to behind-firewall clusters via a single port.
- 🐳 Easy to deploy via Docker and Kubernetes.
- 🔐 Restrict target containers based on port and DNS pattern.
- 🛠 Full SSH compatibility (port tunneling, scp, sftp, rsync, sshfs).
- 📄 Basic access logging based on user logins.

## Getting Started

### Prerequisites

- The target container names must start with the prefix defined via `$SSH_PERMIT_TARGET_HOST`.
- The SSH target containers must have a valid public key that can be found under `$SSH_TARGET_KEY_PATH` (default: `~/.ssh/id_ed25519.pub`).

> ℹ️ _The SSH proxy accepts an incoming key, if it belongs to one of the targets key, in other words the proxy/bastion server authorizes all target public keys. It is still not possible to login to the bastion directly. The authorization happens only for creating and tunneling the final connection._

- In Kubernetes mode, the SSH proxy and the SSH targets must be in the same namespace.

> ℹ️ _The implemented behavior can be slow for big clusters, as `kubectl exec` is a quite slow command._

You can avoid those requirements by setting `$MANUAL_AUTH_FILE=true` and maintaing the bastion's `/etc/ssh/authorized_keys_cache` file yourself (e.g. by mounting a file at the same location). In this case, you don't have to mount the Docker socket / Kubernetes config into the container. The `authorized_keys_cache` file has the same format as the standard ssh authorized_keys file.

### Start SSH Proxy

#### Docker

```bash
docker run -d \
    -p 8091:22 \
    -v /var/run/docker.sock:/var/run/docker.sock \
    --env SSH_PERMIT_TARGET_HOST=<some-name> \
    mltooling/ssh-proxy
```

#### Kubernetes

_WIP_

```bash
docker run -d \
    -p 8091:22 \
    -v /root/.kube/config:/root/.kube/config \
    --env SSH_PERMIT_TARGET_HOST=<some-name-prefix> \
    mltooling/ssh-proxy
```

### Connect to Target

```bash
ssh \
    -o "ProxyCommand=ssh -W %h:%p -p 8091 -i ~/.ssh/<target-key> limited-user@<ssh-proxy-host>" \
    -p <target-port> \
    -i ~/.ssh/<target-key> \
    root@<target-host>
```

Doing this way, the connection from client to target is end-to-end encrypted.

> ℹ️ _The "\<target-host\>" host can be the Docker container name or Kubernetes service name. In that case, the bastion has to be in the same Docker network or the connection must be allowed in case of existing Networkpolicies in Kubernetes, respectively._

### Configuration

The container can be configured with the following environment variables (`--env`):

<table>
    <tr>
        <th>Variable</th>
        <th>Description</th>
        <th>Default</th>
    </tr>
    <tr>
        <td>SSH_PERMIT_TARGET_HOST</td>
        <td>Defines which other containers can be ssh targets. The container names must start with the prefix. The ssh connection to the target can only be made for targets where the name starts with the same prefix. The '*' character can be used as wildcards, e.g. 'workspace-*' would allow connecting to target containers/services which names start with 'workspace-'.
        </td>
        <td>*</td>
    </tr>
    <tr>
        <td>SSH_PERMIT_SERVICE_PORT</td>
        <td>Defines on which port the other containers can be reached via ssh. The ssh connection to the target can only be made via this port then.</td>
        <td>22</td>
    </tr>
    <tr>
        <td>SSH_TARGET_KEY_PATH</td>
        <td>The path inside the target containers where the manager looks for a valid public key.
        Consider that `~` will be resolved to the target container's home.</td>
        <td>~/.ssh/id_ed25519.pub</td>
    </tr>
    <tr>
        <td>MANUAL_AUTH_FILE</td>
        <td>Disables the bastion's public key fetching method and you have to maintain the /etc/ssh/authorized_keys_cache file yourself (e.g. by mounting a respective file there)</td>
        <td>false</td>
    </tr>
</table>

## Features

### Access Logging

Logins are logged at `/etc/ssh/access.log`

# Where to ask questions

The SSH Proxy project is maintained by [@raethlein](https://twitter.com/raethlein) and [@LukasMasuch](https://twitter.com/LukasMasuch). Please understand that we won't be able
to provide individual support via email. We also believe that help is much more
valuable if it's shared publicly so that more people can benefit from it.

| Type                     | Channel                                              |
| ------------------------ | ------------------------------------------------------ |
| 🚨 **Bug Reports**       | <a href="https://github.com/ml-tooling/ssh-proxy/issues?utf8=%E2%9C%93&q=is%3Aopen+is%3Aissue+label%3Abug+sort%3Areactions-%2B1-desc+" title="Open Bug Report"><img src="https://img.shields.io/github/issues/ml-tooling/ssh-proxy/bug.svg"></a>                                 |
| 🎁 **Feature Requests**  | <a href="https://github.com/ml-tooling/ssh-proxy/issues?q=is%3Aopen+is%3Aissue+label%3Afeature-request+sort%3Areactions-%2B1-desc" title="Open Feature Request"><img src="https://img.shields.io/github/issues/ml-tooling/ssh-proxy/feature-request.svg?label=feature%20requests"></a>                                 |
| 👩‍💻 **Usage Questions**   |  <a href="https://stackoverflow.com/questions/tagged/ml-tooling" title="Open Question on Stackoverflow"><img src="https://img.shields.io/badge/stackoverflow-ml--tooling-orange.svg"></a> <a href="https://gitter.im/ml-tooling/community" title="Chat on Gitter"><img src="https://badges.gitter.im/ml-tooling/community.svg"></a> |
| 🗯 **General Discussion** | <a href="https://gitter.im/ml-tooling/community" title="Chat on Gitter"><img src="https://badges.gitter.im/ml-tooling/community.svg"></a>  <a href="https://twitter.com/mltooling" title="ML Tooling on Twitter"><img src="https://img.shields.io/twitter/follow/mltooling.svg?style=social"></a>                  |

## Contribution

- Pull requests are encouraged and always welcome. Read [`CONTRIBUTING.md`](https://github.com/ml-tooling/ssh-proxy/tree/master/CONTRIBUTING.md) and check out [help-wanted](https://github.com/ml-tooling/ssh-proxy/issues?utf8=%E2%9C%93&q=is%3Aopen+is%3Aissue+label%3A"help+wanted"+sort%3Areactions-%2B1-desc+) issues.
- Submit github issues for any [feature enhancements](https://github.com/ml-tooling/ssh-proxy/issues/new?assignees=&labels=feature-request&template=02_feature-request.md&title=), [bugs](https://github.com/ml-tooling/ssh-proxy/issues/new?assignees=&labels=bug&template=01_bug-report.md&title=), or [documentation](https://github.com/ml-tooling/ssh-proxy/issues/new?assignees=&labels=enhancement%2C+docs&template=03_documentation.md&title=) problems.
- By participating in this project you agree to abide by its [Code of Conduct](https://github.com/ml-tooling/ssh-proxy/tree/master/CODE_OF_CONDUCT.md).

---

Licensed **Apache 2.0**. Created and maintained with ❤️ by developers from SAP in Berlin. 
Production Deployment
=====================

Example how to deploy redis for single-instance production environment.

Clone repositories
------------------

.. code-block:: bash

    mkdir ~/extra2000
    cd ~/extra2000
    git clone https://github.com/extra2000/redis-podman.git

Then, ``cd`` into project root directory:

.. code-block:: bash

    cd redis-podman

Deploy redis
------------

From the project root directory, ``cd`` into ``deployment/production/redis``:

.. code-block:: bash

    cd deployment/production/redis

Create config file and fix permissions:

.. code-block:: bash

    cp -v configmaps/redis.yaml{.example,}
    cp -v configs/redis.conf{.example,}
    chcon -v -t container_file_t configs/redis.conf

Create pod file:

.. code-block:: bash

    cp -v redis-pod.yaml{.example,}

Create SELinux security policy:

.. code-block:: bash

    cp -v selinux/redis_podman.cil{.example,}

Load SELinux security policy:

.. code-block:: bash

    sudo semodule -i selinux/redis_podman.cil /usr/share/udica/templates/base_container.cil

Verify that the SELinux module exists:

.. code-block:: bash

    sudo semodule --list | grep -e "redis_podman"

Deploy redis:

.. code-block:: bash

    podman play kube --configmap configmaps/redis.yaml --seccomp-profile-root ./seccomp redis-pod.yaml

Test redis via TCP connection:

.. code-block:: bash

    podman run -it --network=host --rm docker.io/redis:6.2-alpine redis-cli -h 127.0.0.1 -p 6379 ping

Test redis via Unix socket:

.. code-block:: bash

    podman run -it --network=host --rm -v redis-socket-dir:/data/run/:ro docker.io/redis:6.2-alpine redis-cli -s /data/run/redis.sock ping

Create systemd files to run at startup:

.. code-block:: bash

    mkdir -pv ~/.config/systemd/user
    cd ~/.config/systemd/user
    podman generate systemd --files --name redis-pod-srv01
    systemctl --user enable container-redis-pod-srv01.service
